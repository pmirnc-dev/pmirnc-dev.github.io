---
layout: post
title:  "nestjs main.ts 내에 있는 app.enableCors() 세팅방법"
date:   2023-09-21 09:00:00 +0900
author: mclee
categories: nestjs
---
<hr/>

## nestjs main.ts(js) 내에 있는 app.enableCors() 세팅

---

### 서론
최근 컨택시스템/실사관리시스템과 PNet-파일서버 연동 관련 작업을 하면서 CORS 정책 관련 이슈들이 있었습니다.

개발자 도구창에서 네트워크탭을 살펴보고 있는 도중에 특이한 점을 발견하였습니다.

응답해더에 Access-Controls-Allow-Origin의 값이 * 으로 적혀있는 것을 발견하였습니다.

모든 요청을 받기 때문에 보안상 취약한 점이 있다는 것을 알고 있습니다.

이 부분을 해결하기 위해서는 nestjs를 backend으로 사용한 모든 프로젝트의 main.ts(js)에 내용 중
app.enableCors()인 enableCors 메서드를 알아보려고 합니다.

---
### 본론

![로컬네트워크헤더정보](/assets/images/mclee/230921/local_Access_control_Allow_Origin_Status_false.JPG)

위에 있는 캡처 사진을 보시면 해당 값이 나오는 경우는

backend에서 /src/main.ts 파일 내용중에 app.enableCors()만 작성할 경우에 Access-Controls-Allow-Origin : *  로 적혀 있는걸 보이실겁니다.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  //...
  app.enableCors();
  //...
}
bootstrap();
```
왜 모든요청(*)으로 나오는 이유는?

아래 코드를 보시면 알수 있습니다

.enableCors()의 기본값으로 설정되어 있었습니다.

[cors options 보기](https://github.com/expressjs/cors#configuration-options)
```json
{
  "origin": "*",
  "methods": "GET,HEAD,PUT,PATCH,POST,DELETE",
  "preflightContinue": false,
  "optionsSuccessStatus": 204
}
```

origin 값을 설정하기 위해서 아래 코드와 같이 값을 설정 하시면 됩니다.

origin 옵션에 들어가는 구성요소들
```typescript
declare type StaticOrigin = boolean | string | RegExp | (string | RegExp)[];

export declare type CustomOrigin = (requestOrigin: string, callback: (err: Error | null, origin?: StaticOrigin) => void) => void;

```

아래의 5가지 방법중 하나를 선택하여 추가하시면 됩니다.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  //...
  app.enableCors(
    /*
    { origin: true }                                       //boolean
    { origin: 'http://example.com' }                       //string
    { origin: /example\.com$/ }                            //정규식
    { origin: ['http://example.com', /example\.com$/ ] }   //배열
    { origin: function (origin, callback) {                //함수를 추가
          if (whitelist.indexOf(origin) !== -1) {
            console.log("allowed cors for:", origin)
            callback(null, true)
          } else {
            console.log("blocked cors for:", origin)
            callback(new Error('Not allowed by CORS'))
          }
        }
    }
    */
  );
  //...
}
bootstrap();
```
1. boolean 타입
   - true   : CORS를 활성화 됩니다. 어떤 url로 진입 하듯 Access-Control-Allow-Origin: [url] 로 응답해더를 받습니다.
   - false  : CORS를 비활성화 시킵니다.
   - 배포단계에서는 Cors를 비활성화 하거나 cors 해야 되는 url를 추가 하면 됩니다.
2. string 타입
    - 단 하나의 특정 origin만 요청을 반영
3. 정규식
    - /example\.com$/ 정규식은 example.com로 끝나는 url에서 오는 모든 요청을 반영을 합니다.
4. 배열
    - 여러개 url이 접속이 필요 경우 배열로 설정 하면 됩니다.
    - ex) ["http://example1.com", /\.example2\.com$/]
5. 함수(function)
    - origin 사용자 정의 논리를 구현하는 함수입니다.
    - 들어오는 origin을 어떻게 처리할지 로직을 직접 커스텀하는 함수

앞서 말했듯이 5가지 타입에서 하나만 설정을 하신다면 모든요청(*) 기본값을 변경 및 보안 취약점을 예방할 것으로 보입니다.

### 결론

저희가 웹리포트, 기타 프로젝트 등등 backend를 구성할 nestjs로 작성하신다면 참조되셨으면 합니다.

개발시 app.enableCors()만 사용하시다가 배포전에  origin 옵션을 추가하여 불필요한 접근을 막으시면 되겠습니다.

---
참조사이트 

- [cors options](https://github.com/expressjs/cors#configuration-options)