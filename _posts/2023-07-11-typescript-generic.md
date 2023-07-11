---
layout: post
title:  "typescript Generic"
date:   2023-07-11 11:00:00 +0900
author: ybcho
categories: typescript generic
---
<hr/>

## Generic 이란
TypeScript에서 제네릭(Generic)은 타입 안정성을 유지하면서 여러 종류의 타입에 대해 작업할 수 있도록 도와주는 기능입니다.

제네릭을 사용하면 특정 타입이 아닌 타입 매개변수를 사용하여 코드를 작성할 수 있습니다.

선언 시점에 타입을 명시하는 것이 아닌 사용 시점에 타입을 명시하여 하나의 타입만이 아닌 다양한 타입을 사용할 수 있도록 하는 기법입니다.

한번의 선언으로 다양한 타입에 재사용이 가능하다는 장점이 있습니다.

## 제네릭 사용법

```ts
function 함수이름<T>(매개변수: T): T {
  // 함수 본문
}

class 클래스1 {
  메소드이름1<T>(T:매개변수){

  }
  
  메소드이름2<T>(매개변수:T):T{

  }
}

class 클래스2<T> {
    // 클래스 멤버
}

함수이름<string>("PMI");
const cls1 = new 클래스1();
cls1.메소드이름1<number>(1577);
cls1.메소드이름2<string>('1577');

const cls2 = new 클래스2<{name:string, no:number}>();

```

## 제네릭 제약조건
원하지 않는 속성에 접근하는 것을 막기 위해 Generic에 제약조건을 사용합니다. 제약조건을 사용하여 특정 타입들로만 동작하도록 할 수 있습니다.

* 제약 조건을 걸기 위해선 <T>에 extends 사용

* 제약조건에 벗어나는 타입은 에러 발생

```ts
//number와 string만 허용
function setMessage<T extends number|string>(msg: T):string{
    return "msg의 타입은 "+ typeof msg +", 메세지는 "+msg+" 입니다."
}
setMessage<string>("PMI");
setMessage<number>(12345);
setMessage<object>([1,2,3,4,5]); //type Error
```
인터페이스를 상속하여 인터페이스에 정의된 프로퍼티가 있는 타입만 사용할 수도 있습니다.
```ts
interface IProperty {
    toString():string;
    length:number
}

function genericFn1<T extends IProperty>(arg: T): T {
    console.log(arg.toString());
    console.log(arg.length);
    return arg;
}

genericFn1<string>("123");
genericFn1<number>(123); //type Error
genericFn1<Array<number>>([1,2,3,4,5]);
genericFn1<{name:string; no:number}>({name:'pmi',no:1}); //type Error
```

## 애니 타입,유니온 타입과 차이점
제네릭과 유니온 타입이 둘다 여러 타입을 동시에 다룬다는 점에서 비슷한데 

위 처럼 타입 제약조건을 줄꺼면 그냥 유니온 타입을 쓰면 되지 않을까?

```ts
function anyFn(msg:any){
    return msg
}

function genericFn<T extends number|string>(msg: T):T{
    return msg;
}
function unionFn(msg: number|string){
    return msg;
}
genericFn<string>("123").toString(); 
genericFn<number>(123).toFixed();
genericFn<string>("123").toFixed(); // 타입추론되어 사전에 에러 방지
unionFn(123).toFixed(); //type Error - string 타입엔 toFixed속성이 없다고 에러
anyFn("123").toFixed() //type 체크 안함
```
애니타입은 타입 검사를 아예 하지 않아 사전에 에러를 방지할 수 없고,

유니온타입은 인자로 number가 넘어올지 string이 넘어올지 모르기 때문에 에러가 발생하지만

제네릭은 사용 시점에 타입을 명시해줬기 때문에 타입 추론이 되어 에러가 나지 않습니다.


### aixos에서 제네릭 사용하기
![axios](/assets/images/ybcho/axios.png)

![axiosResponse](/assets/images/ybcho/axiosResponse.png)

![get](/assets/images/ybcho/get.png)

![post](/assets/images/ybcho/post.png)


<br/>
<br/>
<br/>
제네릭이 참으로 유용해 보이지만 적절히 잘 활용하는건 또 다른 문제인 것 같습니다.

제네릭을 자유자재로 잘 활용하는 날까지 노력하겠습니다...

(틀린 내용이나 피드백 해주실 내용이 있어 말씀해 주시면 감사드리겠습니다.)

---
### 참고 사이트
> [https://poiemaweb.com/typescript-generic](https://poiemaweb.com/typescript-generic)

> [https://lakelouise.tistory.com/188](https://lakelouise.tistory.com/188)

> [https://www.typescriptlang.org/ko/docs/handbook/2/generics.html](https://www.typescriptlang.org/ko/docs/handbook/2/generics.html)
