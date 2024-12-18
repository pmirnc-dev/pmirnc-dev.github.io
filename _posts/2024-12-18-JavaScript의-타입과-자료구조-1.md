---
layout: post
title:  "JavaScript의 타입과 자료구조 (1)"
date:   2024-12-18 09:30:00 +0900
author: hsyoo
categories: javascript Symbol
published: true
---
<hr/>

_본 포스팅은 mdn 문서를 참고하여 작성하였습니다._
_내용이 길어 1, 2로 나누어 작성하겠습니다._

## 동적 및 약타입  
**JavaScript**는 **동적 타입**이 있는 동적 언어입니다. (python, ruby, php 등도 동적 타입 언어입니다.)  
JavaScript의 변수는 어떤 특정 타입과 연결되지 않으며, 모든 타입의 값으로 할당 (및 재할당) 가능합니다.  

```javascript
let foo = 42; // foo는 이제 숫자입니다
foo = "bar"; // foo는 이제 문자열입니다
foo = true; // foo는 이제 불리언입니다
```

JavaScript는 또한 약타입 언어이기도 합니다.   
즉, 작업에 타입 오류가 발생하는 대신, 일치하지 않는 타입이 포함된 경우 암시적 타입 변환이 가능합니다.

```javascript
const foo = 42; // foo는 숫자입니다.
const result = foo + "1"; // JavaScript는 foo를 문자열로 강제 변환하므로, 다른 피연산자와 연결할 수 있습니다.
console.log(result); // 421
```

암시적 형변환은 매우 편리하지만, 개발자가 변환을 수행할 의도가 없거나,   
다른 방향으로 변환하려는 경우(예: 숫자에서 문자열 대신, 문자열에서 숫자로 변환하는 것) 잠재적인 오류(footgun)가 될 수 있습니다.   
기호 및 BigInts의 경우, JavaScript는 의도적으로 특정 암시적 타입 변환을 허용하지 않습니다.

## 원시 값(Primitive values)

Object를 제외한 모든 타입은 언어의 최하위 수준에서 직접 표현되는 불변 값을 정의합니다. 이러한 타입의 값을 '**원시 값**'이라고 합니다.

null을 제외한 모든 기본 타입은 **typeof** 연산자로 테스트할 수 있습니다. **typeof null은 "object"를 반환하므로 === null을 사용하여 null을 테스트**해야 합니다.

null 및 undefined를 제외한, 모든 원시 타입에는 원시 값으로 작업하는 데 유용한 메서드를 제공하는 해당 객체 래퍼 타입이 있습니다.   
예를 들어, Number 객체는 **toExponential()**와 같은 메서드를 제공합니다.   

```javascript
let num = 100000;
num.toExponential(); // 1e+5
```

원시 값에서 속성에 접근하면, JavaScript는 값을 해당 래퍼 객체로 자동으로 감싸는 대신 객체의 속성에 접근합니다.   
그러나 null 또는 undefined에서 속성에 접근하면, TypeError 예외가 발생하므로 optional chaining 연산자를 도입해야 합니다.

| 타입 | typeof 반환 값 | Object wrapper |
|---|---|---|
| Null 타입 | "object" | N/A |
| Undefined 타입 | "undefined" | N/A |
| Boolean 타입 | "boolean" | Boolean |
| Number 타입 | "number" | Number |
| BigInt 타입 | "bigint" | BigInt |
| String 타입 | "string" | String |
| Symbol 타입 | "symbol" | Symbol |

### ****Null 타입****
Null 타입은 null이라는 오직 하나의 값만 가질 수 있습니다.

### Undefined 타입

Undefined 타입은 undefined이라는 오직 하나의 값만 가질 수 있습니다.

개념적으로, undefined는 값이 없음을 의미하고, null은 객체가 없음을 의미합니다. (typeof null === "object"에 대한 변명이 될 수도 있습니다). 
일반적으로 값이 없는 경우 언어의 기본값은 undefined입니다.

반환 값이 없는 return문(return;)은 암시적으로 undefined를 반환합니다.

```javascript
function f() {}

console.log(f()); // undefined

```

존재하지 않는 객체 속성에 접근하면 undefined가 반환됩니다.

```javascript
const obj = {};
console.log(obj.nonExistentProperty); // undefined
```

초기화(let x;)가 없는 변수 선언은 변수를 undefined로 암시적으로 초기화합니다.

```javascript
let x;
console.log(x); // undefined
```

Array.prototype.find() 및 Map.prototype.get()와 같은 대부분의 메서드는, 요소를 찾을 수 없을 때 undefined를 반환합니다.
```javascript

const arr = [1, 2, 3, 4, 5];
console.log(arr.find(x => x > 5)); // undefined

const map = new Map();
console.log(map.get("nonExistentKey")); // undefined
```


null은 언어의 핵심적인 부분에선 덜 자주 사용됩니다. 가장 중요한 위치는 prototype chain의 끝부분입니다.   
이어서, Object.getPrototypeOf(), Object.create() 등 프로토타입과 상호 작용하는 메서드는 undefined 대신 null을 받거나 반환합니다.

null은 키워드이지만, undefined는 전역 속성인 일반적인 식별자입니다. 
실제로는 undefined가 재정의되거나 가려져서는 안 되기 때문에 그 차이는 미미합니다.  

### Boolean 타입

Boolean 타입은 논리 요소를 나타내며 true와 false 두 가지의 값을 가질 수 있습니다.

Boolean 값은 일반적으로 삼항 연산자 , if...else, while 등을 포함한, 조건부 연산에 사용됩니다.

### Number 타입 

Number 타입은 배정밀도 64비트 이진 형식 IEEE 754 값입니다. 
<img src="https://velog.velcdn.com/post-images%2Fjaden_94%2F8cc47d10-457c-11ea-80f8-830d6ce61636%2FIEEE-754.jpeg" />


2-1074 (Number.MIN_VALUE) 와 21024 (Number.MAX_VALUE) 사이의 양수 부동 소수점 뿐만 아니라,     
-2-1074 와 -21024 사이의 음수 부동 소수점 숫자도 저장할 수 있지만,   
-(253 − 1) (Number.MIN_SAFE_INTEGER) 와 253 − 1 (Number.MAX_SAFE_INTEGER) 범위의 정수만 안전하게 저장할 수 있습니다.  
```javascript
const min = Number.MIN_VALUE;
const max = Number.MAX_VALUE;

console.log(min); // 5e-324
console.log(max); // 1.7976931348623157e+308
```

```javascript

const x = Number.MAX_SAFE_INTEGER + 1;
const y = Number.MAX_SAFE_INTEGER + 2;

console.log(x === y); // true

```

이 범위를 벗어나면, JavaScript는 더 이상 정수를 안전하게 표시할 수 없습니다. 대신 배정밀도 부동 소수점 근사값으로 표시됩니다.   
Number.isSafeInteger()를 사용하여 숫자가 안전한 정수 범위 내에 있는지 확인할 수 있습니다.  

Number.MAX_VALUE보다 큰 양수값은 +Infinity로 변환됩니다.  
Number.MIN_VALUE보다 작은 양수값은 +0으로 변환됩니다.  
Number.MAX_VALUE보다 작은 음수값은 -Infinity로 변환됩니다.
Number.MIN_VALUE보다 큰 음수값은 -0으로 변환됩니다.
±(2-1074 to 21024) 범위를 벗어나는 값은 자동으로 변환됩니다. 

```javascript
const x = Number.MAX_SAFE_INTEGER + 1;
const y = Number.MAX_SAFE_INTEGER + 2;
const z = Number.MAX_SAFE_INTEGER - 3;
console.log(Number.isSafeInteger(x)); // false
console.log(Number.isSafeInteger(y)); // false
console.log(Number.isSafeInteger(z)); // true
```



```javascript
// 부동 소수점 예시

const x = 0.1;
const y = 0.2;
const z = 0.3;
console.log(x + y === z); // false
```

+Infinity 및 -Infinity는 수학에서의 무한대와 유사하게 작동하지만, 약간의 차이가 있습니다. 자세한 내용은 Number.POSITIVE_INFINITY 및 Number.NEGATIVE_INFINITY를 참고하세요.

```javascript
console.log(Number.POSITIVE_INFINITY); // Infinity
console.log(Number.NEGATIVE_INFINITY); // -Infinity

```

Number 타입에는 여러 표현이 있는 하나의 값만 있습니다.  
0은 -0과 +0 둘 다로 표현됩니다(여기서 0은 +0의 별칭입니다).  
실제로는, 서로 다른 표현 간에 거의 차이가 없습니다. 예를 들어, +0 === -0은 true입니다.
그러나, 0으로 나누면 다음과 같이 알 수 있습니다.  

```javascript
console.log(42 / +0); // Infinity
console.log(42 / -0); // -Infinity
```
NaN ("Not a Number")는 산술 연산의 결과를 숫자로 표현할 수 없을 때, 일반적으로 발생하는 특별한 종류의 숫자 값입니다.   
또한, "NaN"은 자기 자신과 같지 않은 JavaScript의 유일한 값이기도 합니다.   
[Why “NaN === NaN” is False?](https://velog.io/@eunbeann/Why-NaN-NaN-is-False){:target="_blank"}

```javascript

console.log(0 / 0); // NaN
console.log(NaN === NaN); // false

```

숫자는 개념적으로 "수학적인 값"이고 항상 암시적으로 부동 소수점으로 인코딩되지만, JavaScript는 비트 연산자를 제공합니다.   
비트 연산자를 적용할 때 숫자는 먼저 32비트 정수로 변환됩니다.

> 비트 마스킹을 사용해서 하나의 숫자 안에 다수의 불리언 값을 나타낼 때 비트 연산자를 사용할 수 있긴 하지만 이건 보통 나쁜 방법으로 여겨집니다.   
> JavaScript는 일련의 불리언 값들을 나타내는 방법을 (배열이나 불리언 속성 값을 할당한 객체로) 제공합니다.   
> 비트 마스킹은 코드를 읽고, 이해하고, 유지하기 힘들게 만듭니다.  

다만 로컬 저장소의 한정된 공간을 활용해야 한다거나, 네트워크의 비트 하나하나가 소중한 극한 상황처럼 제한된 환경에서는 이런 기술이 필요할 수도 있습니다. 비트 마스킹은 용량 최적화의 마지막 수단으로만 고려해야 합니다.

### BigInt 타입
BigInt 타입은 임의 정밀도로 정수를 나타낼 수 있는 JavaScript 숫자 원시 값입니다.   
BigInt로 Number의 안전한 정수 제한(Number.MAX_SAFE_INTEGER)을 넘어서는 큰 정수도 안전하게 저장하고 연산할 수 있습니다.  
BigInt는 정수 끝에 n을 추가하거나 BigInt() 함수를 호출해 생성할 수 있습니다.  

```javascript
// BigInt
const x = BigInt(Number.MAX_SAFE_INTEGER); // 9007199254740991n
console.log(x + 1n === x + 2n); // 9007199254740992n는 9007199254740993n과 같지 않아 false입니다.

// Number
console.log(Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2); // 둘 다 9007199254740992 이기 때문에 true입니다.
```

+, *, -, **, % 연산자를 BigInt에서도 사용할 수 있습니다. 금지된 연산자는 >>>뿐 입니다. BigInt는 Number와 엄격하게 같지는 않지만 느슨하게 유사합니다.  
BigInt는 소수를 나타낼 수 없지만, 큰 정수를 더 정확하게 나타낼 수 있기 때문에, BigInt 값은 숫자보다 항상 더 정확하거나 덜 정확하지 않습니다.   
어떤 타입도 다른 타입을 수반하지 않으며, 서로 대체할 수 없습니다. TypeError는 BigInt 값이 산술 표현식의 일반 숫자와 혼합되거나 서로 암시적으로 변환되는 경우 발생합니다.

### String 타입

String 타입은 텍스트 데이터를 나타내며, UTF-16 코드 단위 수를 나타내는 16비트 부호 없는 정수 값의 나열로 인코딩됩니다. 
String의 각 요소는 String 내부의 위치를 차지합니다. 첫 번째 요소는 인덱스 0에, 다음 요소는 인덱스 1에 있습니다. 
String의 길이는 UTF-16 코드 단위의 수이며, 실제 유니코드 문자 수와 일치하지 않을 수 있습니다. 자세한 내용은 String 페이지를 참조하세요.  

JavaScript String은 변경할 수 없습니다. 즉, String이 생성되면 수정할 수 없습니다. String 메서드는 현재 String의 내용을 기반으로 새 String을 만듭니다.

- substring()을 사용해 원래 String의 하위 String을 만들 수 있습니다.    
- 연결 연산자(+) 또는 concat()를 사용하여 두 문자열을 연결합니다.

### "문자열의 타입화"를 조심하라

문자열을 사용해 복잡한 데이터를 표현하는 것이 매력적으로 보일지도 모르고, 단기적으로는 다음과 같은 장점이 있습니다.

- 연결 연산자를 통해 복잡한 문자열을 쉽게 만들 수 있습니다.
- 문자열은 디버깅이 쉽습니다. (출력 내용이 항상 문자열의 값과 동일)
- 문자열은 많은 API(입력 칸, 로컬 스토리지 값, responseText와 함께 사용하는 XMLHttpRequest 등등)의 공통 분모이며 문자열로만 작업하고 싶을 수 있습니다.

규칙을 통해, 어떤 자료구조라도 문자열로 표현할 수 있습니다. 그러나 그게 좋은 방법은 아닙니다.   
예를 들어, 구분자를 사용하면 (물론 JavaScript 배열이 더 적합하겠지만) 문자열로 리스트를 흉내낼 수도 있을 것입니다.   
그러나 구분자를 리스트의 요소로 사용하는 순간 리스트가 망가지고 맙니다. 이제 구분자를 구분하기 위해 이스케이프 문자를 선택하고, 등등... 이 모든 것이 각자의 규칙을 필요로 하고 불필요한 유지보수 부담이 발생합니다.
```javascript

// 이런 방식은 좋지 않습니다.
const list = "apple, orange, banana, watermelon, kiwi";
const items = list.split(", ");
console.log(items); // apple, orange, banana, watermelon, kiwi

// 그러나 ,를 포함하는 항목이 있을 경우 문제가 발생합니다.
const list = "apple, orange, banana, water,melon, kiwi";
const items = list.split(", ");
console.log(items); // apple, orange, banana, water, melon, kiwi
```

문자열은 텍스트 데이터에만 사용하세요. 복잡한 데이터를 표현해야 할 땐 문자열을 구문 분석하고 적합한 추상화를 사용하세요.

### Symbol 타입
Symbol은 고유하고 변경 불가능한 원시 값이며 객체의 속성키로 사용할 수 있습니다.  
어떤 프로그래밍 언어들에선 "원자"라고 부르기도 합니다. 기호의 목적은 다른 코드의 키와 충돌하지 않도록 보장되는 고유한 속성 키를 만드는 것입니다.

2부에서 계속

### 출처
[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
[https://another-light.tistory.com/105](https://another-light.tistory.com/105)
[https://ko.javascript.info/symbol](https://ko.javascript.info/symbol)
