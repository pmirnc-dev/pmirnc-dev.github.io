---
layout: post
title:  "Iteration protocols"
date:   2022-08-22 18:30:00 +0900
author: skan
categories: javascript
---

## Iteration protocols란?

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols

ES6가 출현하면서 여러 문법뿐만 아니라 Iteration Protocol이라는 규칙이 추가되었다. 이것은 JS 자체의 독자적인 특징이라기보다는 여러 언어에서 <b>반복 동작을 수행하기 위해 정의</b>하는 방법에 가깝다. 
<br />

##Why?
프로그래밍을 하다 보면, 어떤 구조/객체의 내부에서 반복 동작을 수행하고 싶을 때가 있다. 
이때 사용할 수 있는 방법이 `for..of`문과 `spread 연산자`인데, 이 문법을 사용하기 위한 전제가 바로 Iteration Protocol이다.<br/>
현재까지 확인된 바로는, yield*와 destructuring assignment 또한 이 규칙에 의존하는 구문, 표현식이라고 한다.

- `for..of`
```jsx
const numbers = [1, 2, 3, 4];

for (const number of numbers) {
  console.log(number); // 1 2 3 4
}
```

- `spread 연산자`
```jsx
const numbers = [1, 2, 3, 4];

console.log(...numbers); // 1 2 3 4
```

어떠한 데이터를 기준으로 데이터를 공급하는 쪽(`Array`, `String`)과 데이터를 소비하는 쪽(`for..of`, `spread operator`)으로 나누어 보자. 
여기서 만약 데이터 공급처가 데이터를 제공하기 위해 각기 다른 순회 방식을 가지게 된다면 어떻게 될까? 
아마 데이터 소비자는 그 방식에 일일이 대응하여 데이터를 받아올 수 있도록 지원해야 할 것이다.

하지만 이는 효율적인 방식이 아니기에 하나의 통일된 규칙을 만들어 두 주체가 이것에만 의존하게 할 필요가 있다. 
결국, 이 과정에서 나오게 된 것이 Iteration Protocol이며, 이것은 데이터 공급자와 소비자 사이를 이어주는 인터페이스(매개체)인 셈이다.

JS의 Iteration Protocol을 이루는 하위 규칙은 Iterable Protocol과 Iterator Protocol이 있다. 지금부터 각 Protocol의 특징을 함께 알아보자.

---

## Iterable Protocol
Iterable Protocol은 객체의 반복적인 동작에 대한 정의이다. 
어떤 객체에 아래의 2가지 사항이 구현되어 있다면 그 객체는 iterable이다.

1. 객체는 `Symbol.iterator`라는 key를 가지고 있어야 한다.
2. `Symbol.iterator`의 value는 iterator를 반환하는 함수여야 한다.

```jsx
const str = 'hello';
str[Symbol.iterator]; // [Function]
```

JS 내에서 확인할 수 있는 Built-in iterable은 다음과 같다.

- `String`
- `Array`
- `Typed Array`
- `Map`
- `Set`

떄문에 `String` 타입인 문자열도 배열처럼 순회가 가능하다.

```jsx
const str = 'edit';

// for .. of
for (let w of str) {
  console.log('for ... of > ', w);
}

//spread 연산자
const word = [...str];
console.log('spread 연산자 > ', word);

// 분해대입
const [w1, w2] = str;
console.log('분해대입 > ', w1);
console.log('분해대입 > ', w2);

// Array.from는 iterable 혹은 array-like를 인수로 받음
console.log('Array.from > ', Array.from(str));
```

결과
```text
for ... of >  e
for ... of >  d
for ... of >  i
for ... of >  t
전개구문 >  (4) ['e', 'd', 'i', 't']
구조분해할당 >  e
구조분해할당 >  d
Array.from >  (4) ['e', 'd', 'i', 't']
```
하지만 도입부에서도 언급했다시피, 이 규칙은 사용자 정의가 가능하므로 built-in iterable이 아니더라도 어느 객체에서든 충분히 구현이 가능하다.
```jsx
// custom iterable 만들기.
const iterable = {};

// Symbol.iterator키의 값은 iterator를 반환하는 함수이다.
iterable[Symbol.iterator] = function () {
  return {
    current: 1,
    last: 5,
    next: function () {
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    },
  };
};

for (const i of iterable) {
  console.log(i); // 1 2 3 4 5
}
```

---
##iterator protocol

Iterator Protocol은 값의 순서를 제공하는 방법에 대한 정의이다.
next() 매서드를 통해 배열을 순회할 수 있는 객체이다.
next()는 {done: boolean, value: any} 라는 값을 반환하며, done 이 true를 반환 할 때까지 객체를 순환한다.
iterable과 마찬가지로 어떤 객체에 아래의 2가지 사항이 구현되어 있다면 그 객체는 iterator이다.

 1. 객체는 `next()` 메서드를 가지고 있어야 한다.
 2. `next()` 실행의 반환 값은 다음과 같은 key를 가진 객체여야 한다.   
    - `value` - 반복문 안의 단일 값이며 어떠한 타입이든 상관없다.  
    - `done` - 반복문 종결 여부에 대한 판단 값이며 `Boolean` 타입이다.
    
```jsx
iterable = { //이터러블
  [Symbol.iterator]() {
    return { //이터레이터 객체
      next() {
        return { value: any, done: boolean };
      }
    }
  }
}
```
위에서 다룬 iterable의 코드를 보면 `Symbol.iterator`에 할당된 함수에서 반환하는 객체가 바로 iterator의 모양인 것을 확인할 수 있다.

예시
```jsx
// 문자열은 iterable이므로 이로부터 iterator를 생성할 수 있습니다.
const strIterator = 'go'[Symbol.iterator]();
strIterator.next(); // { value: 'g', done: false }
strIterator.next(); // { value: 'o', done: false }
strIterator.next(); // { value: undefined, done: true }
```

##결론
 - for - of 문 같은 몇몇 반복가능한 기능들을 할 수 있도록 만들어진 대상을 이터러블(iterable)라고 한다.
 - 이터러블(iterable)에는 반드시 Symbol.iterator 프로퍼티가 반드시 구현되어 있어야한다.
 - Symbol.iterator 메소드의 결과를 이터레이터(Iterator)객체 라고하고, 이터레이터는 이어지는 반복 과정을 처리한다.
 - 이터레이터 객체에는 {doen: Boolean, value: any}를 반환하는 메소드인 next() 가 반드시 구현되어 있어야 한다.

이러첨 Iteration Protocol은 객체 내부의 반복 동작을 수행하기 위해 정립된 규칙이다.
그 때문에 JS에서 존재하는 대부분의 자료 구조 내부에서 연속적인 값을 얻거나 특정 행동을 실행시키기 위해서는 반드시 이 규칙을 따라야 한다.
