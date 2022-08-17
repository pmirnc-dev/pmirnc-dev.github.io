---
layout: post
title:  "javascript 기초"
date:   2022-08-17 20:15:00 +0900
author: ybcho
categories: javascript
---
<hr/>

### 목차

---

1. javascript
2. 변수와 상수
3. 연산자
4. IF, ?(조건부 연산자)
5. 함수

   <br />

---

## JavaScript란?

자바스크립트는 ‘웹페이지에 생동감을 불어넣기 위해’ 만들어진 프로그래밍 언어입니다.

쉽게 웹페이지 요소들을 조작하는 언어라고 생각하시면 되겠습니다

javascript는 원래 frontend에서 사용하는 언어이지만 node.js의 등장으로 backend에서도 사용합니다.

<br />

---

## 변수와 상수

---

javascript는 다른 언어와 다르게 자료형(int, string, float 등등)과 함께 선언하지 않습니다.   
변수냐 상수냐에 따라 var, let, const 키워드와 함께 선언합니다.

변수와 상수를 아래와 같이 선언할 수 있습니다.
```jsx
var item; //변수
let item =""; //변수
const item="cookie"; //상수
```

먼저 변수는 (variable) 변할 수 있는 값이라는 뜻으로 이름을 붙인 저장공간입니다.   
var와 let 모두 변수 선언 키워드지만 차이점이 있습니다. 차이점은 아래에 다시 설명하겠습니다.

변수는 언제든지 값을 변경, 대입할 수 있습니다.
```jsx
let item = "cookie";
let food;
item = "cake";
food = item;
```

상수는 절대적인 값으로 한번 할당한 값은 변경 할 수 없으며 선언과 할당이 같이 이루어져야 합니다.
```jsx
const item = "cookie";
item = "food" // error: Assignment to constant variable.
```
<br>
### 변수 선언 var와 let의 차이점

```jsx
var name // 변수 선언
name = 'gil-dong' // 값의 할당

var name = 'gil-dong' // 위에서 선언했지만 다시 선언할 수 있음
```
<br>

```jsx
console.log(name) // output: undefined

var name = 'gil-dong'
console.log(name) // output: gil-dong
```
<br>

```jsx
var a = 1

if (true) {
    var a = 5
}

console.log(a) // output: 5
```
다른 언어를 해봤다면 javascript는 이상한 언어라고 생각 될겁니다.   
선언도 하지 않았는데 값을 할당 할 수 있고, 선언을 했는데 다시 선언을 해도 에러가 나지 않고, if문 스코프에서 할당한 값이 밖에서도 적용됩니다.

이는 변수 선언이 런타임에서 되는 것이 아니라, 그 이전 단계에서 먼저 실행되기 때문입니다.   
자바스크립트 엔진은 소스코드를 한 줄씩 순차적으로 실행하기에 앞서, 선언문을 찾아내 먼저 실행합니다.   
변수 선언이 어디에 있든 상관없이 다른 코드보다 먼저 실행되는 특징을 호이스팅(hoisting)이라 합니다.

위처럼 중복 선언과 스코프 문제 등 불확실성 때문에 ES6에서 const와 let이 추가되었습니다.   
let은 var의 문제점들을 해결해줬습니다. 다른 언어의 변수 선언과 같다고 보시면 되겠습니다.

1. 변수 중복 선언 불가
2. 블록 레벨 스코프
3. 변수 호이스팅 문제 해결

<br />

---

## 연산자

---
### 기본 연산자
javascript의 기본 연산자는 다음과 같습니다.

+ 덧셈 연산자 +,
+ 뺄셈 연산자 -,
+ 곱셈 연산자 *,
+ 나눗셈 연산자 /,
+ 나머지 연산자 %,
+ 거듭제곱 연산자 **

사칙연산자는 수학과 동일하기 때문에 설명에 제외하겠습니다.

#### 나머지 연산자 %

나머지 연산자는 피연산자를 나눈 후 나머지를 정수로 반환합니다.

```jsx
console.log( 5 % 2 ); // 5를 2로 나눈 후의 나머지인 1을 출력
console.log( 8 % 3 ); // 8을 3으로 나눈 후의 나머지인 2를 출력
```

#### 거듭제곱 연산자 **

거듭제곱 연산자는 왼쪽 피연산자를 오른쪽 피연산자만큼 곱한 값이 반환됩니다.

```jsx
console.log( 2 ** 2 ); // 4  (2 * 2)
console.log( 2 ** 3 ); // 8  (2 * 2 * 2)
console.log( 2 ** 4 ); // 16 (2 * 2 * 2 * 2)
```
#### 이항연산자 +
문자와 문자 또는 문자와 숫자 연산에서의 + 는 이항연산자로 취급됩니다.   
이때의 +는 더하기가 아닌 문자열을 연결합니다.

```jsx
let s = "my" + "string";
console.log(s); // mystring

console.log( '1' + 2 ); // "12"
console.log( 2 + '1' ); // "21"
console.log( 2 + 2 + '1' ); // "41"
```

+를 제외한 나머지 연산자는 문자를 숫자로 취급합니다.

```jsx
console.log( 6 - '2' ); // 4, '2'를 숫자로 바꾼 후 연산
console.log( '6' / '2' ); // 3, 두 피연산자가 숫자로 바뀐 후 연산
```
### 논리 연산자
논리 연산자에는 ||(논리합), &&(논리곱), !(부정)이 있습니다.   
다른 언어와는 다르게 javascript에는 일반적인 논리연산자 외에 추가 기능이 있습니다.

#### || (OR 논리합)
둘 중 하나라도 true면 true를 반환합니다.

```jsx
if (1 || 0) { // 1은 true이기 때문에 if문 실행 
    alert( 'true' );
}
```
#### && (OR 논리곱)
둘 다 true면 true, 하나라도 false면 false를 반환합니다.

```jsx
if (1 && 0) { // 0은 false이기 때문에 true && false 연산으로 false 반환  
    alert( 'true' );
}else{
    alert('false');
}
```
#### ! (NOT 부정)
true면 false를 false면 true를 반환합니다.

```jsx
let result = 10;

if (!result) { // 10은 true이기 때문에 !연산으로 false 반환  
    alert( 'true' );
}else{
    alert('false');
}
```   

#### javascript에서만 제공하는 '단축평가'
단축평가란?
||(논리합), &&(논리곱) 연산자는 왼쪽부터 오른쪽으로 평가를 진행하는데,   
값을 boolean값으로 비교 후 중간에 평가 결과가 나오면 boolean값을 반환하지 않고 원래값을 반환합니다.   
중간에 true가 반환되면 오른쪽 끝까지 가지 않고 평가 결과를 반환합니다.

단축평가를 활용해 아래와 같은 코드를 작성할 수 있습니다.
```jsx
let result = "apple" || "banana";  // "apple"
result = "apple" && "banana";  // "banana"

result = false || "banana";  // "banana"
result = true || "banana";  // true
result = "banana" || false;  // "banana"
result = "banana" || true;  // "banana"

result = false && "banana"; // false
result = true && "banana"; // "banana"
result = "banana" && true; // true
result = "banana" && false; // false
```
> [단축평가 참고](https://curryyou.tistory.com/193)

<br />

---

## IF, ?조건부 연산자

---
### if
if문은 if(...) 괄호 안에 들어가는 조건이 true면 블록을 실행합니다.

문자열은 빈칸이 아닌 값은 모두 true, 숫자는 0이 아닌 값은 모두 true로 변환 되어 if문을 실행합니다.   
아래 코드들은 모두 true로 if문 안 명령문들을 모두 실행합니다.
```jsx
if (true) {
    명령문
}
if (item === true) {
    명령문
}

if("0"){
    명령문
}

if(-1){
    명령문
}
```

### else if
차이가 있는 조건 여러 개를 처리해야 할 때가 있습니다. 이때 else if를 사용할 수 있습니다.

```jsx
item = 2023;
if (item < 2022) {
    명령문
} else if (year > 2022) {
    명령문 //해당 블록 실행
} else {
    명령문
}
```
### 조건부 연산자 (삼항연산자)
조건부 연산자를 사용하면 경우에 따라 if문을 더 짧고 간결하게 변형할 수 있습니다.
조건이 true면 ?뒤인 값1을 아니라면 값2를 result에 리턴합니다.

```jsx
let result = 조건 ? 값1 : 값2;

const age=17;
let result = age>=19 ? "성인" : "미성년자"; // result = "미성년자"
```
else if처럼 다중으로 사용가능합니다.

```jsx
const age="17";
let result = age>=19 ? "성인" :
    age<19 ? "미성년자" : "숫자가 아닙니다." // result = "숫자가 아닙니다."
```
<br>

---

## 함수

---
프로그래밍을 하다보면 코드가 반복적이거나 여러 곳에서 필요한 경우가 있습니다. 이럴때 함수를 사용합니다.
### 함수 선언
javascript에 함수선언 방법은 아래와 같은 방법이 있습니다.
```jsx
function 함수명(파라미터1, 파라미터2, 파라미터3...) {
    함수 본문...
}

function sum(num1, num2){
    return num1 + num2;
}

//함수 표현식
const sum = function(num1, num2){
    return num1 + num2;
}

//화살표 함수
const sum = (num1, num2) => {
    return num1 + num2;
}
```
### 함수 호출
함수 호출은 함수명(매개변수)로 호출 할 수 있습니다.
리턴값이 있으면 변수로 받거나 함수의 매개변수로 받는 등 다양하게 활용할 수 있습니다.
```jsx
function sum(num1, num2){
    return num1 + num2;
}
function multiply(num1, num2){
    return num1* num2;
}
let result1 = sum(10,20)// // result1 = 30;
alert(sum(10,20) - sum(5,5)); //20
let result2 = multiply(sum(1,2), sum(3,4)); //result2 = 21
```

### 내장 함수
내장함수는 javascript에 이미 정의된 함수를 의미합니다.
alert(), confirm(), isNaN(), String(), Number() 등등 모두 이미 정의된 함수들입니다.

### 화살표 함수(arrow function)
화살표 함수는 기존 함수보다 간결한 문법으로 함수를 만들 수 있습니다.   
화살표 함수는 function 키워드를 사용하지 않아도 되고 표현식이 계산식이거나 값이면 return 키워드도 생략할 수 있습니다.   
화살표 함수는 아래와 같이 선언할 수 있습니다.

```jsx
//화살표 함수
let func = (파라미터1, 파라미터2) => 본문;

//기존 함수
let func = function(파라미터1, 파라미터2) {
    return 본문;
};

//파라미터가 1개라면 괄호 생략가능
let func = 파라미터1 => 본문;

//Example
let showMassege = str => console.log("메세지 : ",str);
showMassege("안녕"); // 메세지 : 안녕
```

<br />

하지만 일반적으로 위처럼 사용하진 않고 콜백함수에서 화살표 함수를 많이 사용합니다.

```jsx
setTimeout(() => {
    console.log("1초 뒤에 출력됩니다.")
}, 1000);

//기존 함수
setTimeout(function() {
    console.log("1초 뒤에 출력됩니다.")
}, 1000);
```

#### ※ 화살표 함수와 일반 함수 차이
1. 생성자 함수의 역할을 할 수 없다. (new로 인스턴스 생성 불가)
2. 화살표 함수는 this를 가지지않고 함수가 정의된 스코프에 존재하는 this를 가르킵니다.
3. 화살표 함수는 arguemnts 프로퍼티를 생성하지 않는다.

#### 잠시 this에 대해서...
생성자 함수에서의 this는 함수를 생성한 대상을 가르킵니다. 아래 예제에선 Account를 생성한 account를 가르키게 됩니다.
```jsx
function Account(username, balance) {
    // 속성
    this.username = username;
    this.balance = balance;
    // 메소드
    this.getBalance = function() {
        return `${this.username}님의 잔액은 ${this.balance}입니다.`;
    };
    this.deposit = function(monye) {
        this.balance = this.balance + monye;
    };
}

account = new Account("yong-bin", 20000);
console.log(account.getBalance()); // yong-bin님의 잔액은 20000입니다.
account.deposit(2000);
console.log(account.getBalance()); // yong-bin님의 잔액은 22000입니다.
```

<br />

객체에 속한 메소드에서의 this는 메소드가 속한 객체를 가리킵니다.
```jsx
const account = {
    username: "yong-bin",
    balance: 10000,
    getBalance: function() {
        //this키워드를 사용하여 객체의 속성인 username과 balance에 접근하고 있습니다.
        return `${this.username}님의 잔액은 ${this.balance}입니다.`;
    },
    deposit: function(monye) {
        this.balance = this.balance + monye;
    }
};

console.log(account.getBalance()); // yong-bin님의 잔액은 10000입니다.
account.deposit(20000);
console.log(account.getBalance()); //// yong-bin님의 잔액은 30000입니다.
```

> [this 참고](https://bohyeon-n.github.io/deploy/javascript/this.html)