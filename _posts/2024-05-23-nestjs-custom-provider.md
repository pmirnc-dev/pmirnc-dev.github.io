---
layout: post
title:  "nestjs custom provider"
date:   2024-05-23 09:00:00 +0900
author: ybcho
categories: ['nestjs']
---
<hr/>
### nestjs provider란?
> DI(종속성 주입)를 통해 다른 컴포넌트에 재사용 가능한 기능 또는 서비스를 제공하는 기본 구성 요소입니다.<br/>
> 앱이 제공하고자 하는 핵심 기능을 수행하는 역할을 합니다.
> 
> provider를 등록하면 NestJS IoC 컨테이너에 의해 관리됩니다.
 
<br/>

### provider 종속성 주입 일반적인 방법

다음 코드는 일반적으로 종속성 주입 하는 방법입니다.

* 서비스 만들기

다른 클래스에 주입 받을수 있도록 @injectable()데코레이터 선언

```ts
@injectable() 
export class CatService{
    //로직 구현
}
```

<br/>

* 모듈에 등록

```ts
@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class CatsModule{}
```

<br/>

* provider를 사용할 클래스에 주입

```ts
@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}
}
```

### custom provider
custom provider는 기본적으로 제공되는 provider 외에 사용자가 직접 정의하는 provider를 의미합니다.

custom provider는 다음과 같을 때 사용할 수 있습니다.

* 외부 라이브러리를 provider로 등록하고 싶을 때
* 상황에 맞게 동적으로 인스턴스를 생성해야 할 때
* 테스트를 위해 모의 버전으로 클래스를 재정의하고 싶을 때

custom provider를 등록하는 방법은 4가지 방법이 있습니다.

1. value provider
2. class provider
3. factory provider
4. Alias provider

하나씩 방법과 특징을 소개하겠습니다.


### value provider (useValue)
value provider는 상수 값을 주입해야하거나, 외부 라이브러리를 네스트 컨테이너에 주입해야하거나, 실제 구현을 mock 객체로 대체해야할 때 사용합니다.
```ts
@Module({
    controllers: [AnimalController],
    providers:[
        {
            provide: 'MAX', //useValue로 상수 값 등록하여 사용
            useValue: 5000,
        },
        {
            provide: 'AXIOS', //useValue로 axios 객체를 등록하여 사용  
            useValue: axios,
        },
    ]
})

export class AnimalModule {}
```

```ts
@Injectable()
export class AnimalService {
    constructor( // provide에 설정한 토큰값으로 종속성 주입
        @Inject('AXIOS') private readonly axios: Axios,
        @Inject('MAX') private readonly max: number
    ) {}
}
```

### class provider (useClass)
class provider는 일반적으로 provider를 등록할때 사용하는 방법으로, 인스턴스를 동적으로 생성할 수 있고 특정 조건에 따라 주입해야할 인스턴스를 달리 해줄 수 있습니다.

```ts
@Module({
    controllers: [AnimalController],
    providers:[
        {
            provide: 'ANIMAL',
            useClass: process.env.NODE_ENV === 'production' ? CatService : DogService, //환경에 따라 동적으로 provider 등록
        },
        {
            provide: 'DOG',
            useClass: DogService,
        },
        {
            provide: CatService, //일반적으로 주입했을 때와 같은 방법
            useClass: CatService,
        },
        //CatService 위와 같음
    ]
})

export class AnimalModule {}
```

```ts
@Controller('animal')
export class AnimalController {
    constructor(
        @Inject('ANIMAL') private readonly animalService, //토큰을 string으로 설정했을 경우
        @Inject('DOG') private readonly dogService: DogService,
        @Inject(CatService) private readonly catService: CatService, // @Inject() 생략가능
    ) {}
}
```

### factory provider (useFactory)
factory provider는 factory 함수를 정의하여 동적으로 인스턴스를 생성할 때 사용합니다. inject 속성으로 factory 함수에 인자를 전달 할 수 있습니다.

> ※ Factory 함수는 인자를 받아 객체를 return 해주는 함수(인자에 따라 동적으로 객체 리턴)
```ts


@Module({
    controllers: [AnimalController],
    providers: [
        {
            provide: 'AXIOS',
            useFactory: async (configService: ConfigService): Promise<AxiosInstance> => {
                const axiosConfig = {
                    baseURL: configService.get('API_BASE_URL'), // 환경 변수 사용
                    timeout: configService.get('API_TIMEOUT'),
                    // ... 기타 Axios 설정 옵션 (headers, interceptors 등)
                };
                return axios.create(axiosConfig);
            },
            inject: [ConfigService], // ConfigService 인자 전달
        },
    ],
})
```

```ts
@Injectable()
export class AnimalService {
    constructor( // provide에 설정한 토큰값으로 종속성 주입
        @Inject('AXIOS') private readonly axios: AxiosInstance,
    ) {}
}
```

### alias provider (useExisting)
이미 존재하는 provider의 인스턴스를 새로운 토큰(provider 이름)으로 등록하여 재사용하는 데 사용됩니다.
useExisting으로 주입한 provider는 같은 인스턴스를 사용하기 때문에 코드 중복을 줄일 수 있습니다.
```ts
@Module({
    controllers: [AnimalController],
    //두개 서비스 모두 PatProvider 인터페이스 상속 구현 상태
    providers:[
        DogService,
        CatService, 
        {
            provide:"patService",
            useExisting:proccess.env.PAT ==='CAT' ? CatService : DogService
        }
    ]
})
```

```ts
@Injectable()
export class AnimalService {
    constructor( // provide에 설정한 토큰값으로 종속성 주입
        @Inject('patService') private readonly patService: PatProvider,
        private readonly catService:CatService,
        private readonly dogService:DogService
    ) {}
    
    //공통 기능
    feed(){
        this.patService.feed(); //같은 인스턴스 사용
    }
    
    //고양이만 하는 행동
    grooming(){
        this.catService.grooming();
    }

    //강아지만 하는 행동
    walk(){
        this.dogService.walk();
    }
}
```

---
### 참고

[https://docs.nestjs.com/fundamentals/custom-providers](https://docs.nestjs.com/fundamentals/custom-providers)

[https://velog.io/@mskwon/NestJS-Fundamentals-Custom-Provider](https://velog.io/@mskwon/NestJS-Fundamentals-Custom-Provider)


