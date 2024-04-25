---
layout: post
title:  "iterator protocols 그리고 generator"
date:   2024-04-25 09:00:00 +0900
author: hsyoo
categories: javascript iterator generator
published: true
---
<hr>

* 본 문서는 mdn web docs를 참고하여 작성되었습니다.

## Iteration protocols

> 순회 프로토콜은 새로운 내장 객체 또는 구문이 아닌 프로토콜입니다. 이러한 프로토콜은 몇 가지 규칙에 따라 모든 객체에서 구현될 수 있습니다.

프로토콜은 **순회 가능 프로토콜 - iterable protocol**과 **반복자 프로토콜 - iterator protocol** 2가지로 존재합니다.

### The iterable protocol(순회 가능 프로토콜)

> 순회 가능 프로토콜을 사용하면 JavaScript 객체를 for...of 구조에서 반복되는 값과 같은 순회 동작을 정의하거나 사용자 지정할 수 있습니다. 
> Object와는 달리 Array 또는 Map과 같은 일부 내장 유형은 기본 순회 동작이 있는 내장 순회 가능 항목입니다.


### [Symbol.iterator]

반복자 프로토콜을 준수하는 객체를 반환하는 인수가 없는 함수.

[Symbol.iterator] 속성을 기본적으로 가지고 있는 객체들은 for...of 구문을 사용할 수 있습니다.

```javascript

// Array
const array = [1, 2, 3];
for (const value of array) {
  console.log(value);  // 1, 2, 3
}

// String
const text = "hello";
for (const char of text) {
    console.log(char);  // h, e, l, l, o
}

// Map
const map = new Map();
map.set('a', 1);
map.set('b', 2);
for (const [key, value] of map) {
    console.log(`${key}: ${value}`);  // a: 1, b: 2
}


// Set
const set = new Set([1, 2, 3, 4, 4]);
for (const value of set) {
    console.log(value);  // 1, 2, 3, 4
}

// 타입 배열(Typed Arrays)
const typedArray = new Uint8Array([1, 2, 3]);
for (const num of typedArray) {
    console.log(num);  // 1, 2, 3
}


// 함수 내부에서 사용 가능한 arguments 객체
function example() {
    for (const arg of arguments) {
        console.log(arg);
    }
}
example(1, 2, 3);  // 1, 2, 3

```

### The iterator protocol(반복자 프로토콜)

> 반복자 프로토콜은 유한하거나 무한한 일련의 값을 생성하는 표준 방법을 정의하고, 모든 값이 생성되었을 때 잠재적으로 반환 값을 정의합니다.
> 모든 반복자 프로토콜 메서드(next(), return(), 그리고 throw())는 IteratorResult 인터페이스를 구현하는 객체를 반환해야 합니다

**next()**
반복자의 다음 값을 반환합니다. 이 메서드는 두 개의 속성을 가진 객체를 반환합니다. value는 다음 값을 나타내며, done은 반복자가 더 이상 값을 생성할 수 없음을 나타냅니다.

**done - optional**
반복이 끝났을 때 true를 반환하고, 그렇지 않으면 false를 반환합니다. 이는 반복자가 더 이상 값을 생성할 수 없음을 나타냅니다.

**value - optional**
반복자가 반환하는 값입니다. done이 true인 경우, value는 생략할 수 있습니다.

```javascript

// Array
const array =  new Array('피', '앰', '아', '이');
const values = array.values();  // Array Iterator
console.log(values.next());  // {"value":"피","done":false}
console.log(values.next());  // {"value":"앰","done":false}
console.log(values.next());  // {"value":"아","done":false}
console.log(values.next());  // {"value":"이","done":false}

// Map
const map = new Map([['1', '피'], ['2', '앰'], ['3', '아'], ['4', '이']]);
const entries = map.entries();  // MapIterator
console.log(entries.next());  // {"value":["1","피"],"done":false}
console.log(entries.next());  // {"value":["2","앰"],"done":false}
console.log(entries.next());  // {"value":["3","아"],"done":false}
console.log(entries.next());  // {"value":["4","이"],"done":false}

// Set
const set = new Set(['피', '앰', '아', '이']);
const values = set.values();  // SetIterator
console.log(values.next());  // {"value":"피","done":false}
console.log(values.next());  // {"value":"앰","done":false}
console.log(values.next());  // {"value":"아","done":false}
console.log(values.next());  // {"value":"이","done":false}

// String
const text = '피앰아이';
const iterator = text[Symbol.iterator]();
console.log(JSON.stringify(iterator.next())); // {"value":"피","done":false}
console.log(JSON.stringify(iterator.next())); // {"value":"앰","done":false}
console.log(JSON.stringify(iterator.next())); // {"value":"아","done":false}
console.log(JSON.stringify(iterator.next())); // {"value":"이","done":false}
```

## Generator

> Generator 객체는 generator function 으로부터 반환되며, 반복 가능한 프로토콜과 반복자 프로토콜을 모두 준수합니다.

```javascript

function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator(); // "Generator { }"

console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
console.log(gen.next().value); // 3


// 무한 제너레이터

function* infinite() {
    let index = 0;

    while (true) {
        yield index++;
    }
}

const generator = infinite(); // "Generator { }"

console.log(generator.next().value); // 0
console.log(generator.next().value); // 1
console.log(generator.next().value); // 2

```

참고 문서
[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Generator](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols){:target="_blank"}  
[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols){:target="_blank"}
