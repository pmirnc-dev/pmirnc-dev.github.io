---
layout: post
title:  "text-format"
date:   2023-04-06 09:40:00 +0900
author: ybcho
categories: String text-format
---
<hr/>

## 텍스트서식
문자열을 일정한 형식에 맞게 정리하거나 표현하는 것
텍스트 문자열을 변경하는 데 사용되는 방법

## javascript 문자열의 특징

- 자바스크립트에서는 글자 하나만 저장할 수 있는 별도의 자료형(ex: JAVA의 char)이 없습니다.
- 텍스트 형식의 데이터는 길이에 상관없이 문자열은 변경 불가능한 배열과 같은 객체 형태로 저장됩니다.
- UTF-16 형식을 따릅니다.

문자열 리터럴은 3가지 방식으로 표현할 수 있습니다.

- 작은 따옴표 '작은 따옴표'

- 큰 따옴표 "큰 따옴표"

- 백틱 \`백틱\`

### backtick
백틱은 표현식을 ${...}로 감싸고 백틱으로 다시 감싸면 표현식을 간단하게 문자열에 삽입할 수 있습니다.
```javascript
function sum(a,b){
    return a+b;
}
const num1=10;
const num2=20;
console.log(`두 수 ${num1},${num2}의 합은 ${sum(num1,num2)}`); // "두 수 10, 20의 합은 30"
```
백틱을 사용하면 문자열을 여러 줄에 걸쳐 작성할 수도 있습니다.

```javascript
console.log(`PMI
개발부
화이팅
`)
```

### escape sequence
문자나 키보드에서 직접 입력할 수 없는 문자를 표현하기 위한 방법입니다.

백슬래쉬(\) + 코드를 사용하여 표현합니다.

#### 종류
- \t 탭
- \n 개행
- \x 16진수 문자 표현
- \u UTF-16 문자 표현 (UTF-32는 중괄호{}사용)
- \\`'" 따옴표
```javascript
console.log( "\"PMI\"" );
console.log( "\u2700" );
console.log( "\u{1F60D}" );
```

### String 객체
String 객체는 문자열 생성자입니다. new 키워드 없이 호출한 String은 일반적으로 사용하는 리터럴 표기법(문자열 원형)과 같은 역할을 합니다.
```javascript
console.log(String('PMI')); //PMI
console.log('PMI'); //PMI

console.log(String(`${1+2}`)); //3
console.log(`${1+2}`); //3
```
하지만 new 키워드를 사용하여 생성한 String 객체는 리터럴 표기법 문자열 차이점이 있습니다.
```javascript
const str1 = "2+2";
const str2 = String(str1);
const str3 = new String(str1);

console.log(str1); // "2+2"
console.log(str2); // "2+2"
console.log(str3); // {"2+2"}

console.log(typeof str1); // "string"
console.log(typeof str2); // "string"
console.log(typeof str3); // "object"

console.log(eval(str1));  // 4
console.log(eval(str3)); // "2 + 2"
```




***UTF-16이란?***

UTF-16은 유니코드 문자를 나타내기 위한 인코딩 방식 중 하나입니다.

UTF-16은 16비트로 문자를 표현하며, 각 문자마다 2바이트 또는 4바이트(surrogate pair,utf-32)를 사용합니다.

따라서 기본적으론 2^16=65536 개의 문자를 표현할 수 있습니다.

javascript는 UTF-16형식을 따르기 때문에 UTF-16에 포함된 문자 1개는 length를 1로 표현합니다. 때문에 한글과 영어 동일하게 길이가 1로 표현됩니다. 

UTF-16에 포함되지 않는 문자들은(emoji 등)2개의 코드를 조합하여(surrogate pair) 표현하거나 utf-32로 표현하기 때문에 length는 2입니다. ex) 😀의 utf-16코드는 "\uD83D\uDE00" u{1F600}


### 문자열 메소드

| 메소드                                 | 설명                                                           |
|-------------------------------------|--------------------------------------------------------------|
| charAt, charCodeAt,<br/>codePointAt | 문자열에서 지정된 위치에 있는 문자나 문자 코드를 반환합니다.                           |
| indexOf, lastIndexOf                | 문자열에서 지정된 부분 문자열의 위치나 지정된 부분 문자열의 마지막 위치를 각각 반환합니다.          |
| startsWith, endsWith, includes      | 문자열 시작하고, 끝나고, 지정된 문자열을 포함하는지의 여부를 반환합니다.                    |
| concat                              | 두 문자열의 텍스트를 결합하고 새로운 문자열을 반환합니다.                             |
| fromCharCode, fromCodePoint (en-US) | 유니코드 값의 지정된 시퀀스로부터 문자열을 구축합니다. 문자열 인스턴스가 아닌 문자열 클래스의 메서드입니다. |
| split                               | 부분 문자열로 문자열을 분리하여 문자열 배열로 문자열 개체를 분할합니다.                     |
| slice                               | 문자열의 한 부분을 추출하고 새 문자열을 반환합니다.                                |
| substring, substr                   | 어느 시작 및 종료 인덱스 또는 시작 인덱스 및 길이를 지정하여, 문자열의 지정된 일부를 반환합니다.     |
| match, replace, search              | 정규 표현식으로 작업합니다.                                              |
| toLowerCase,toUpperCase             | 모든 소문자 또는 대문자에서 각각 문자열을 반환합니다.                               |
| normalize                           | 호출 문자열 값의 유니 코드 표준화 양식을 반환합니다.                               |
| repeat                              | 주어진 회를 반복하는 개체 요소로 이루어진 문자열을 반환합니다.                          |
| trim                                | 문자열의 시작과 끝에서 공백을 자릅니다.                                       |

<br />


## Intl
Intl 객체는 여러 가지 언어로 서비스를 할 수 있도록 각 언어에 맞는 문자비교, 숫자, 시간, 날짜비교를 제공하는, ECMAScript 국제화 API를 위한 객체입니다.

예를 들어, 12/11/21라고 표시된 날짜 데이터를 한국 사용자들은 2012년 11월 21일로 생각을 하겠지만, 미국 사용자들에게는 12월 11일 2021년으로 받아들여질 수 있으며,

다른 나라들에서는 12일 11월 2021년으로 읽혀질 수 있습니다. 이러한 포멧팅 문제를 Intl객체로 해결할 수 있습니다.

### Intl.DateTimeFormat
언어에 맞는 날짜 및 시간 서식을 적용할 수 있습니다.

```javascript
//default는 window 설정언어
//style옵션엔 full, long, medium, short 사용
console.log(new Intl.DateTimeFormat(['ja','ko'],{ dateStyle:'full',timeStyle: 'full'}).format(new Date()));
console.log(new Intl.DateTimeFormat('en-GB').format(new Date()));
console.log(new Intl.DateTimeFormat('en-US').format(new Date()));
```
### Intl.NumberFormat
언어에 맞는 통화, 백분율, 무게, 길이, 속도, 온도와 같이 단위가 있는 숫자 데이터 서식을 적용할 수 있습니다.


```javascript
//통화
console.log(new Intl.NumberFormat('ko', { style: 'currency', currency: 'KRW' }).format(50000));
console.log(new Intl.NumberFormat('ko', { style: 'currency', currency: 'USD' }).format(40.56));

//백분율
console.log(new Intl.NumberFormat('ko', { style: 'percent' }).format(0.7));
console.log(new Intl.NumberFormat('ko', { style: 'percent' }).format(1 / 4));

//숫자 표기
console.log(new Intl.NumberFormat('ko').format(15000));
console.log(new Intl.NumberFormat('de').format(15000));
console.log(new Intl.NumberFormat('ar-EG').format(15000));

//킬로그램
console.log(new Intl.NumberFormat('ko', { style: 'unit', unit: 'kilogram' }).format(50));
console.log(new Intl.NumberFormat('ar', { style: 'unit', unit: 'kilogram' }).format(50));
```



참고 사이트   
[https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Text_formatting](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Text_formatting)

[https://www.daleseo.com/js-intl-api/](https://www.daleseo.com/js-intl-api/)

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Intl](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Intl)

