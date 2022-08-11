---
layout: post
title:  "자바스크립트의 메모리 관리(garbage-collection)"
date:   2022-08-11 09:30:00 +0900
author: hsyoo
categories: javascript gc memory
published: false
---
<hr/>

_* 이글은 [mdn web docs](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management)
를 참고 해서 작성했습니다._

우리가 FE(front-end)와 BE(back-end)에서 사용하는 javascript는 어떻게 메모리 관리를 할까요?
사실 우리는 javascript(이하 JS)로 코딩을 할때 메모리에 대한 걱정을 크게 하지 않습니다. (물론 걱정이 필요한 때도 분명 있습니다.)  
JS는 객체가 생성되었을 때 자동으로 메모리를 할당하고, 더 이상 필요하지 않을때 자동으로 해제합니다.  
이걸 가비지 컬렉션(이하 GC)이라고 부릅니다.
그럼 GC는 언제 더 이상 필요하지 않다고 판단할까요?

## 자바스크립트의 메모리 할당

### 값 초기화

_문서에 있는 예제가 이해가 쉽기 때문에 가져왔습니다_

```javascript

var n = 123; // 정수를 담기 위한 메모리 할당
var s = 'azerty'; // 문자열을 담기 위한 메모리 할당

var o = {
    a: 1,
    b: null
}; // 오브젝트와 그 오브젝트에 포함된 값들을 담기 위한 메모리 할당

// (오브젝트처럼) 배열과 배열에 담긴 값들을 위한 메모리 할당
var a = [1, null, 'abra'];

function f(a) {
    return a + 2;
} // 함수를 위한 할당(함수는 호출 가능한 오브젝트)

// 함수식 또한 오브젝트를 담기 위한 메모리를 할당합니다.
someElement.addEventListener('click', function(){
    someElement.style.backgroundColor = 'blue';
}, false);


```

### 함수 호출을 통한 할당

```javascript

var d = new Date(); // Date 개체를 위해 메모리를 할당

var e = document.createElement('div'); // DOM 엘리먼트를 위해 메모리를 할당

```

## 할당된 메모리가 더 이상 필요하지 않을 때 해제하기
[참고링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management#release_when_the_memory_is_not_needed_anymore) 

이 단계에서 대부분의 문제가 발생합니다.  
"할당된 메모리가 더 이상 필요없을 때"를 알아내기가 어렵기 때문입니다.

저수준 언어에서는 메모리가 필요없어질 때를 개발자가 직접 결정하고 해제하는 방식을 사용하지만  
자바스크립트와 같은 고수준 언어들은 위에서 언급한 GC로 관리를 하기 때문이죠..

가비지 컬렉터(GC를 수행하는 주체)가 "더 이상 필요하지 않다"를 판단하기 위해서 javascript는 *Mark and sweep* 알고리즘을 사용합니다.

### Mark-and-sweep

이 알고리즘에선 필요하지 않는 오브젝트를 *닿을 수 없는 오브젝트*로 정의하고 작업을 수행합니다.  
root라는 전역 객체 집합을 가진 상태에서   
- roots가 참조하는 객체
  - roots가 참조하는 객체를 참조하는 객체
     - roots가 참조하는 객체를 참조하는 객체 참조하는 객체...
  위와 같은 객체는 *닿을 수 있는 객체*이며 이와 반대 개념인 *닿을 수 없는 객체*에 대해 GC를 수행합니다.






