---
layout: post
title:  "포스트맨 그만, jest를 이용하여 함수 테스트하기 with NestJS"
date:   2024-04-02 09:12:00 +0900
author: dhjeon
categories: jest NestJS
---
<hr/>

Nest Cli 로 프로젝트를 설치하면 기본적으로 Jest가 함께 설치가 되는데 이 Jest가 어떤 역할을 하고 있는지 아시나요?

[package-json](/assets/images/dhjeon/240402/package-json-jest.png)

Jest는 JavaScript로 작성된 코드를 쉽게 테스트할 수 있도록 도와주는 테스팅 프레임워크 입니다.

JavaScript로 동작하기 때문에 NestJS 뿐만 아니라, Vue, React 등 JavaScript 기반 프레임워크에서는 다 사용할 수 있습니다.

[Jest 공식 홈페이지](https://jestjs.io/)

[NPM](https://www.npmjs.com/package/jest)

Nest Cli를 통해 프로젝트를 셋업하면 별도의 설치 없이 Jest를 사용할 수 있어요.

```json
// package.json
{
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

package.json을 살펴보면 `.spec.ts` 라는 이름이 붙은 파일의 경우 테스트 파일로 읽도록 설정이 되어있습니다. 

[app.spec.ts.file](/assets/images/dhjeon/240402/app-spec.png)

그래서 Nest Cli로 프로젝트를 만들면 spec 파일이 함께 있는 것을 볼 수 있는데요.

다시 package.json으로 돌아와 test 관련 scripts를 실행시켜주면 jest는 프로젝트 내의 `.spec.ts` 파일을 읽어 실행 시키게 됩니다.

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

백엔드에서 내가 만든 함수가 제대로 동작하는지 확인을 하고 싶을 때 포스트맨으로 해당 api에 요청을 보내 테스트를 했는데 Jest를 이용하면 이러한 과정을 생략하고 원하는 함수만 테스트를 해볼 수 있습니다.

간단한 예제를 통해서 Jest를 어떻게 활용할 수 있을지 이야기해보려고 합니다.

## 1. 모듈을 사용하지 않은 함수 테스트 하기

Nest 모듈을 사용하지 않는 다는 건 일반 함수를 분리하여 이를 Nest 모듈 내에 사용하는 방식을 의미합니다. 제가 작업을 하면서 Jest를 어떻게 활용했는지 에제를 통해 보여드리려고 합니다.

[dir1](/assets/images/dhjeon/240402/dir1.png);

테스트 시나리오는 아래와 같습니다.

```typescript
describe("유니서베이 문항 순서 변환 테스트", () => {
  test("쿼터 문항의 위치를 SQ_RECODE 뒤 쪽에 위치하도록 한다.", () => {});
});
```

쉽게 말하면

```typescript
const data = [
  {
    questionIndex: 1,
    questionNumber: "DESC",
  },
  {
    questionIndex: 2,
    questionNumber: "SQ1",
  },
  {
    questionIndex: 3,
    questionNumber: "SQ2",
  },
  {
    questionIndex: 4,
    questionNumber: "SQ2_RECODE",
  },
  {
    questionIndex: 5,
    questionNumber: "SQ3",
  },
  {
    questionIndex: 6,
    questionNumber: "QUOTA_SQ1_SQ2",
  },
];
```

이런 데이터의 형태를 아래와 같이 변경해주는 함수를 만들 겁니다.

```typescript
const expectedData = [
      {
        questionIndex: 1,
        questionNumber: "DESC",
      },
      {
        questionIndex: 2,
        questionNumber: "SQ1",
      },
      {
        questionIndex: 3,
        questionNumber: "SQ2",
      },
      {
        questionIndex: 4,
        questionNumber: "SQ2_RECODE",
      },
      {
        questionIndex: 5,
        questionNumber: "QUOTA_SQ1_SQ2",
      },
      {
        questionIndex: 6,
        questionNumber: "SQ3",
      },
    ];
```

```typescript
type InsertQuestionDto = {
  questionIndex: number;
  questionNumber: string;
};

/**@description 쿼터 모듈 추가 - SQ2 문항 뒤로 순서 바꾸기
 * @param {InsertQuestionDto[]} question 설문 정보가 담긴 배열 data
 * @param {string} cursor 이동할 문항의 인덱스가 될 questionNumber
 * @param {string} target 이동시킬 문항 questionNumber
 * */
export const changeQuestionsIndex = (
    question: InsertQuestionDto[],
    cursor: string,
    target: string,
  ): InsertQuestionDto[] => {
    return [];
  };

describe("유니서베이 문항 순서 변환 테스트", () => {
  // jest Mock을 사용하여 가상의 목킹 함수를 생성
  let changeQuestionsIndexMockFn: jest.Mock<InsertQuestionDto[], [InsertQuestionDto[], string, string]>;
  // 실제 데이터가 들어갈 곳
  let data: InsertQuestionDto[] = [];
  // 내가 원하는 데이터 형태
  let expectedData: InsertQuestionDto[] = [];

  // 필요한 함수 및 데이터 셋업
  beforeEach(() => {
    // 함수 초기화
    changeQuestionsIndexMockFn = jest.fn(changeQuestionsIndex);
    // 데이터 셋업
    data = [
      {
        questionIndex: 1,
        questionNumber: "DESC",
      },
      {
        questionIndex: 2,
        questionNumber: "SQ1",
      },
      {
        questionIndex: 3,
        questionNumber: "SQ2",
      },
      {
        questionIndex: 4,
        questionNumber: "SQ2_RECODE",
      },
      {
        questionIndex: 5,
        questionNumber: "SQ3",
      },
      {
        questionIndex: 6,
        questionNumber: "QUOTA_SQ1_SQ2",
      },
    ];

    // 내가 원하는 데이터 형태
    expectedData = [
      {
        questionIndex: 1,
        questionNumber: "DESC",
      },
      {
        questionIndex: 2,
        questionNumber: "SQ1",
      },
      {
        questionIndex: 3,
        questionNumber: "SQ2",
      },
      {
        questionIndex: 4,
        questionNumber: "SQ2_RECODE2",
      },
      {
        questionIndex: 5,
        questionNumber: "QUOTA_SQ1_SQ2", // <-- 변경
      },
      {
        questionIndex: 6,
        questionNumber: "SQ3", // <-- 변경
      },
    ];
  });
  
  test("쿼터 문항의 위치를 SQ_RECODE 뒤 쪽에 위치하도록 한다. target을 cursor 위치로 이동", () => {
    // target을 cursor 위치로 이동
    const target = "QUOTA_SQ1_SQ2"; 
    const cursor = "SQ3";
    
    expect(changeQuestionsIndexMockFn(data, cursor, target)).toEqual(expectedData);
  });
});
```

위 테스트 케이스를 실행 시키면 `changeQuestionsIndex` 반환값이 제가 원하는 데이터 형태가 아니기 때문에 실패한 테스트를 반환합니다.

[test-result](/assets/images/dhjeon/240402/result1.png); 


이제 이 함수를 성공하는 테스트 케이스로 만들기 위해 `changeQuestionsIndex` 함수를 완성해보겠습니다.

```typescript
/**@description 쿼터 모듈 추가 - SQ2 문항 뒤로 순서 바꾸기
 * @param {InsertQuestionDto[]} question 설문 정보가 담긴 배열 data
 * @param {string} cursor 이동할 문항의 인덱스가 될 questionNumber
 * @param {string} target 이동시킬 문항 questionNumber
 * */
export const changeQuestionsIndex = (
  question: InsertQuestionDto[],
  cursor: string,
  target: string,
): InsertQuestionDto[] => {
  const cursorIndex = question.findIndex((v) => v.questionNumber === cursor);
  const targetIndex = question.findIndex((v) => v.questionNumber === target);

  function moveValue(
    array: InsertQuestionDto[],
    fromIndex: number,
    toIndex: number,
  ) {
    const item = array.splice(fromIndex, 1)[0];
    array.splice(toIndex, 0, item);
  }

  moveValue(question, targetIndex, cursorIndex);

  return question
    .map((value, index) => {
      return {
        ...value,
        questionIndex: index + 1,
      };
    })
    .sort((a, b) => a.questionIndex - b.questionIndex);
};
```

함수를 작성하고 테스트를 다시 해보면 제가 원하는 형태의 데이터를 반환하기 때문에 성공하는 테스트 케이스로 바뀝니다.

[test-result2](/assets/images/dhjeon/240402/result2.png);

그렇다면 제가 원하는 데이터 형태를 변경한다면?

```typescript
expectedData = [
  .
  .
      {
        questionIndex: 4,
        questionNumber: "SQ2_RECODE2", // data는 SQ2_RECODE 인데 exptededData 는 SQ2_RECODE2
      },
  .
  .
     
    ];
```

[test-result3](/assets/images/dhjeon/240402/result3.png);

테스트 케이스는 실패하면서 잘못된 곳을 알려주기 때문에 어떤 부분이 잘못됐는지 확인하여 함수를 수정할 수 있습니다.

이렇게 함수를 테스트할 때 **내가 원하는 데이터가 어떤 형태인지 정하고**, **실제로 들어가는 데이터는 어떤 형태인지 정한 후**에 내가 원하는 데이터가 나오도록 함수를 작성하면 잘못된 곳이 어딘지 확인할 수 있습니다.

만약 테스트 케이스가 여러 개라면 마찬가지로 각각의 데이터 형태를 정의하고 여러 테스트 케이스를 작성해주면 됩니다.

```typescript

describe("process.env.COLLECTOR_PROJECT_NUMBER CHECK", () => {
  let collectorSurveyCheckMockFn: jest.Mock<boolean, [number]>;
  let projectNumber = 0;
  
  beforeEach(() => {
    collectorSurveyCheckMockFn = jest.fn(collectorSurveyCheck);
    projectNumber = process.env.COLLECTOR_PROJECT_NUMBER; 
  });
  
  test.skip("process.env.COLLECTOR_PROJECT_NUMBER을 읽을 수 있는지 체크한다.", () => {
    const expectedNumber = 10000000;
    expect(projectNumber).not.toEqual(expectedNumber);
  });

  test.skip("process.env.COLLECTOR_PROJECT_NUMBER가 숫자형인지 체크한다.", () => {
    const expectedNumber = 10000000;
    expect(typeof projectNumber).toEqual(typeof expectedNumber);
  });

  test.skip("process.env.COLLECTOR_PROJECT_NUMBER에 문자열이 들어갔을 때 NaN을 반환하는지 체크한다.", () => {
    expect(String(projectNumber)).toBeNaN();
  });

  test.skip("SNUM을 넘겼을 때 boolean 값을 넘겨 받는지 체크한다.", () => {
    const SNUM = 9999999;
    expect(collectorSurveyCheckMockFn(SNUM)).toBeTruthy();
  });

  test.skip("process.env.COLLECTOR_PROJECT_NUMBER 가 NaN 일 때 DEFAULT_NUMBER를 반환하는지 확인", () => {
    const SNUM = 9999999;
    expect(collectorSurveyCheckMockFn(SNUM)).toEqual(DEFAULT_NUMBER);
  });

  test.skip("최종 함수 테스트 콜렉터는 true, 메타서베이는 false를 반환해야한다.", () => {
    const SNUM = 1000001; // 유니 설문
    expect(collectorSurveyCheckMockFn(SNUM)).toBeTruthy();
  });
});
```

## 2. Nest 모듈을 이용하여 Service 함수 테스트 하기

일반 함수 테스트의 경우 export 된 함수를 .spec.ts 파일에 import 해서 함수를 테스트 했지만 Nest Service를 테스트 하기 위해서는 **가상의 Nest Module을 만들어서 사용** 해야합니다.

Nest Cli 로 생성했을 때 생성된 app.controller.spec.ts를 보면 가상의 테스팅 모듈을 만들어서 사용한 것을 확인할 수 있습니다.

```typescript
// app.controller.spec.ts
describe('AppController', () => {
  let appController: AppController;

  beforeEach(async () => {
    const app: TestingModule = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();

    appController = app.get<AppController>(AppController);
  });

  describe('root', () => {
    it('should return "Hello World!"', () => {
      expect(appController.getHello()).toBe('Hello World!');
    });
  });
});
```

클라이언트에서 전달 받은 값에 따라 지역 정보 배열을 반환하는 테스트 서비스 함수를 테스트 해보겠습니다. (패널 허브 v3 기준)

```typescript
const locationValue1 = ['전국']; // 전국
const locationValue2 = ['서울', '인천/경기']; // 서울 / 인천 / 경기
const locationValue3 = ['서울', '지방 5대 광역시']; // 서울 / 5대광역시
const locationValue4 = ['인천/경기', '지방 5대 광역시']; // 인천 / 경기 / 5대광역시

const locationValue5 = ['서울']; // 서울 / 인천 / 경기
const locationValue6 = ['인천/경기']; // 서울 / 5대광역시
const locationValue7 = ['지방 5대 광역시']; // 인천 / 경기 / 5대

describe('전달 받은 값에 따라 지역 정보 배열을 반환합니다.', () => {
  let quotaCalculationService: QuotaCalculationService;
  let getRegionDataMockFn: jest.Mock<string[], [string[]]>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        QuotaCalculationService,
        { provide: I18nService, useValue: {} },
        { provide: TargetQuotaRepository, useValue: {} },
      ],
    }).compile();

    quotaCalculationService = module.get<QuotaCalculationService>(
      QuotaCalculationService,
    );

    getRegionDataMockFn = jest.fn(quotaCalculationService.getRegionData);
  });

  test('전국을 입력했을 경우 지역 전체 배열을 반환 - CASE 1', () => {
    expect(getRegionDataMockFn(locationValue1)).toEqual(type1);
  });

  test('서울을 입력했을 경우 서울을 반환 - CASE 2', () => {
    expect(getRegionDataMockFn(locationValue5)).toEqual(type2);
  });

  test('인천 / 경기를 입력했을 경우 인천 경기 세종을 반환 - CASE 3', () => {
    expect(getRegionDataMockFn(locationValue6)).toEqual(type3);
  });

  test('지방 5대 광역시를 입력했을 경우 CASE 2 3을 제외한 지역 반환 - CASE4 ', () => {
    expect(getRegionDataMockFn(locationValue7)).toEqual(type4);
  });

  test('서울 / 인천 / 경기 입력했을 경우 - CASE 5', () => {
    expect(getRegionDataMockFn(locationValue2)).toEqual(type5);
  });

  test('인천 / 경기 / 지방5대 광역시를 입력했을 경우 인천 경기 세종을 반환 - CASE 6', () => {
    expect(getRegionDataMockFn(locationValue4)).toEqual(type7);
  });

  test('서울 / 지방 5대 광역시를 입력했을 경우 CASE 2 3을 제외한 지역 반환 - CASE 6', () => {
    expect(getRegionDataMockFn(locationValue3)).toEqual(type6);
  });
});
```

각각의 테스트 케이스는 이렇게 됩니다. 

사용할 서비스는 QuotaCalculationService에 있는 getRegionData 함수를 테스를 하기 위해 가상의 테스팅 모듈을 생성하였습니다.

이번 함수에서는 I18n과 TargetQuotaRepository는 사용하지 않지만 의존 주입을 위해 Testing Module의 Provider에 등록해주었습니다.

```typescript
@Injectable()
export class QuotaCalculationService {
  constructor(
    private readonly i18n: I18nService,
    private readonly targetQuotaRepository: TargetQuotaRepository,
  ) {}
  
  // 지역 정보 반환
  getRegionData(locations: string[]): string[] {
    return [];
  }
}
```

만약 ['전국'] 이라는 값이 getRegionData에 전달된다면 getRegionData는 ['서울','부산','대구','인천','광주','대전','울산','세종','경기','강원','충북','충남','전북','전남','경북','경남','제주'] 를 반환해야하고,

['서울', '인천/경기'] 라는 값이 전달되면 ['서울','인천','경기','세종'] 을 반환하는 함수를 작성해야합니다.

[test-result4](/assets/images/dhjeon/240402/result4.png);

각 케이스 별로 반환하는 정보를 확인할 수 있습니다. getRegionData 함수가 원하는 값을 반환하도록 함수를 작성해 줍니다.

```typescript
@Injectable()
export class QuotaCalculationService {
  constructor(
    private readonly i18n: I18nService,
    private readonly targetQuotaRepository: TargetQuotaRepository,
  ) {}
  
  // 지역 정보 반환
  getRegionData(locations: string[]): string[] {
    return locations
      .map((location) => {
        return regionRepository[location];
      })
      .filter((v) => v)
      .flat(2);
  }
}
```

[test-result5](/assets/images/dhjeon/240402/result5.png);

이렇게 각 케이스 별로 반환 값을 확인할 수 있기 때문에 혹시나 놓친 케이스가 있는지 확인하기도 쉽고 번거롭게 해당 함수를 사용하는 컨트롤러에 api를 요청하지 않고도 함수를 확인할 수 있습니다.


