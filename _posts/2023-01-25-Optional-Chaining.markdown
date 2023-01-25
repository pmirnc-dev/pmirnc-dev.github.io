---
layout: post
title:  "JavaScript Optional Chaining"
date:   2023-01-25 00:00:00 +0900
author: dhjeon
categories: javascript
---
<hr/>

### 체이닝 (.)

객체에 연쇄적으로 메소드를 호출할 수 있도록 하는 패턴

```jsx
const obj = {
	value: 1,
	increment: () => {
		this.value += 1;
		return this;
	},
	add: (number) => {
		this.value += number;
		return this;
	},
	close: () => {
		console.log(this.value)
	}
}
// 체이닝 호출
obj.increment().add(2).close(); // return 4

// 체이닝을 하지 않으면 아래와 같이 호출하여 사용
obj.increment()
obj.add(2)
obj.close() // return 4 
```

- this

    <aside>
    💡 자바스크립트에서의 this

  함수의 현재 실행 문맥 (context)
  자바스크립트에서 함수를 호출할 때 기존 매개변수로 전달되는 인자값에 더해
  arguments 객체 및 this 인자가 함수 내부로 암묵적으로 전달됩니다.

    </aside>

    1. 객체의 메소드 호출할 때 this 바인딩
        - 객체의 프로퍼티가 함수일 경우, 함수 내부에서 사용된 this는 해당 함수를 호출한 객체로 바인딩 됩니다.

        ```jsx
        const obj = {
            name: '전다훈',
            getName: function (){
                console.log(this === obj) // return true
                console.log(this.name)    // return "전다훈"
            }
        }
        ```

    2. 함수를 호출할 때 this 바인딩
       - 함수를 호출할 경우, 해당 함수에서 사용된 this는 전역 객체에 바인딩 됩니다.

         브라우저에서의 전역 객체는 window

       ```jsx
           const str = "Hello World";
           console.log(window.str) // "Hello World"
           const testFunction = function () {
               console.log(this.str) // "Hello World" 
           }
       ```


### 체이닝을 통한 속성값 접근의 문제점

```jsx
const user = { name: {first: 'jeon', last: 'dahoon' } }

console.log(user.name.first) // return 'jeon'
console.log(user.phone) // error
// Uncaught TypeError: Cannot read property 'phone' of undefined
```

&& 연산자를 사용하거나 속성이 존재하는지를 확인하여 Error 발생을 방지할 수 있었습니다.

```jsx
const nestedProp = user.phone && user.phone.type;

if (user.hasOwnProperty('phone')) { ... }
```

### 옵셔널 체이닝 (?.)

옵셔널 체이닝 이란 ES11에 추가된 새로운 문법

말 그대로 특정 값에 대한 참조에 대해서 값이 존재하지 않을 때 선택적으로 체이닝을 할 수 있는 문법

`?.` 는 좌항의 피연산자가 undefined 또는 null 일 경우 undefined를 반환하고 그렇지 않다면 우항의 프로퍼티 참조를 이어서 진행합니다.

옵셔널 체이닝의 경우 Falsy 값이라도 null 또는 undefined가 아니라면 참조를 이어서 진행합니다.

<aside>
💡 Falsy 값이란?

false undefined, null, 0, -0, NaN, ‘’ 와 같은 값을 의미합니다.

</aside>

```jsx
const str = '';

// 문자열 길이를 참조, 이때 좌항 피연사자가 false로 평가되는 Falsy값이라도
// null 또는 undefined가 아니라면 우항의 프로퍼티 참조를 이거간다.
const length = str?.length;
console.log(length); // return 0
```

체이닝으로 속성값에 접근했을 때 해당 속성이 존재하지 않다면 TypeError 을 발생시켰다면

옵셔널 체이닝을 사용하여 같은 속성값에 접근할 경우 존재하지 않는다면 TypeError가 아닌 undefined을 반환하게 됩니다.

```jsx
user?.name?.first // return "jeon"

user?.phone // return undefined
```

`?.` 은 배열이나 함수에서도 사용할 수 있습니다.

```jsx
const arr = null;
arr?.[0] // return undefined;

const func = undefined
func?.() // return undefined;

const user1 = {
	name: '전다훈'
}
const user2 = null;

const key = 'name';

console.log(user1?.[key]); // '전다훈'
console.log(user2?.[key]); // undefined
console.log(user1?.[key]?.phone?.not?.existing); // undefined
```

### 주의사항

1. 옵셔널 체이닝은 **없어도 되는 값에만 사용**해야합니다.

```jsx
// phone의 경우 없어도 되기 때문에 ?. 을 사용하는데 문제가 없습니다.
user?.phone

// 단, 꼭 필요한 값이 user의 값이 없다면 ?. 을 사용하더라고 에러가 발생하게 됩니다.
// ReferenceError: user is not defined

// 반드시 ?. 의 대상이 되는 값은 선언이 되어있어야합니다.
```

1. `?.` 을 많이 사용하면 꼭 있어야 하는 값이  없어도 에러를 발생시키지 않기 때문에 주의해서 사용해야합니다.

### 요약

1. obj?.prop - obj가 존재하면 obj.prop을 반환하고, 그렇지 않다면 undefined 반환
2. obj?.[prop] - obj가 존재하면 obj.[prop]을 반환하고, 그렇지 않다면 undefined 반환
3. obj?.method() - obj가 존재하면 obj.method()을 반환하고, 그렇지 않다면 undefined 반환

`?.` 를 사용하여 체인을 만들면 중첩 프로퍼티에 좀 더 안전한게 접근할 수 있습니다.

`?.` 을 사용할 때 평가 대상이 없어도 괜찮은 경우에만 선택적으로 사용해야합니다.