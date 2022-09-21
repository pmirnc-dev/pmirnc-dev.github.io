---
layout: post
title: "배열 메서드중 map, forEach"
date: 2022-09-21 17:00:00 +0900
author: mclee
categories: javascript
---
<hr/>

### 1. 원인
MDN 사이트에서 배열 관련 메소드중 매개변수를 보고 비교하여 사용을 하였습니다
callback 대한 설명을 보지도 않고 매개변수와  callback 함수 설명 내용 중 특정부분만 보고 사용했던 것 같습니다.

- 매개 변수  

|map|forEach|설명|
|---|---|---|
|currentValue|currentValue|처리할 현재 요소| 
|index|index|처리할 현재 요소의 인덱스|
|array|array|호출한 배열|

- callback 함수

|----------| map                | forEach        |
|----------|--------------------|----------------|
| callback | 새로운 배열 요소를 생성하는 함수 | 각 요소에 대해 실행 함수 |

- map 설명  
  각각의 요소에 대해 오름차순으로 값을 불러옵니다.
  그 함수의 반환값으로 새로운 배열을 만듭니다.
- forEach 설명  
  forEach()는 주어진 callback을 배열에 있는 각 요소에 대해 오름차순으로 한 번씩 실행합니다. 
  삭제했거나 초기화하지 않은 인덱스 속성에 대해서는 실행하지 않습니다.


### 2.예제  
- map
```javascript
//map 메서드_1
var array1 = [1, 2, 3, 4, 5];
var reArray1 = array1.map((item, index) => {
	return item*2;
});
console.log(reArray1);

//map 메서드_2
var array1 = [1, 2, 3, 4, 5];
var reArray1 = array1.map(item => item*2 );
console.log(reArray1);
```
- map을 잘못 사용한 예)  
```javascript
//map 메서드
var array1 = [1, 2, 3, 4, 5];
var reArray1 = [];
array1.map((item, index) => {
	console.log(index);
	reArray1.push(item*2)
});
console.log(reArray1);
```
- 잘못된 예시에서 map -> forEach 변경 목적에 맞게 수정  
```javascript
//forEach 메서드
var array2 = [1, 2, 3, 4, 5];
var reArray2 = [];
array2.forEach((item, index) => {
	console.log(index);
	reArray2.push(item*2)
	//return item * 2;
});
console.log(reArray2);
```
### 3. 결론
사용해야 되는 메서드의 목적에 맞게 사용을 하자!

#### 참조
- map 메서드  
  https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map
- forEach 메서드  
  https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach