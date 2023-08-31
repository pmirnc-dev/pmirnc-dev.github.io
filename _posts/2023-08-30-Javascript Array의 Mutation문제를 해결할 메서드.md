---
layout: post
title:  "Javascript 배열의 Mutation문제를 해결할 메서드"
date:   2023-08-30 10:10:00 +0900
author: yrbyeon
categories: javascript
---
<hr/>




# Javascript Array Method

javascript 관련 유튜브를 찾아보던 중 해당 영상을 보게되었다.

[https://www.youtube.com/watch?v=e6WV_DXGwSg&t=297s](https://www.youtube.com/watch?v=e6WV_DXGwSg&t=297s)

이 영상 초반에는 mutation이 언급된다.

```jsx
const x = [1,2,3];
const y = x.reverse();
y.push(0);
console.log(y);
console.log(x);
```

콘솔에 찍히는 y는 [3,2,1,0]이다. 그럼 x는..?

**mutation이 가지는 문제**

• **mutation**은 디버그하기 어려운 코드를 만든다. 데이터가 값을 보존하지 않고, 어디서 변경 되었는지 알기 어렵다.

• **mutation**은 코드를 이해하기 어렵게 만든다. 언제든지 배열이나 객체가 다른 값을 가질 수 있기 때문에 가독성이 떨어진다.

• 함수 인자의 **mutation**는 함수의 동작을 예측할 수 없게 만든다.

자바스크립트의  배열에서 reverse() 메서드를 실행시켰을 때 기존의 호출된 원래 배열의 상태에 영향을 미치는 문제에 대하여 이야기 하며, 자바스크립트에서는 이런 변수의 상태를 변경(mutate)시키는 기준이 명확하지 않다고 말한다. 즉, 새로운 변수를 생성하거나 기존 변수를 재할당하지 않고 자바스크립트 객체 또는 배열을 변경할 때 예상치 못한 변이(mutation)이 생긴다. 이 현상에서 렌더링 관련 문제도 발생하게 된다. 이런 문제를 해결하기 위하여 immutable한 데이터 구조를 가져야 한다. 

그러나 자바스크립트에서는 immutable을 기본적으로 지원하지 않으며, 완벽하지 않기 때문에 코드에서 mutation이 생기는 것을 피하는 방식을 권장한다. Array.from, 스프레드 연산자 (…) 등의 방법을 사용하여 배열을 복사후 변경한다. sort, reverse, splice등의 메서드는 원본 배열을 변경하지만 contat, map, filter등의 메서드는 배열의 복사본을 만든 다음 복사본에 대해 연산한다. 

```jsx
const z = [ 1, 2, 3 ];
const w = z.filter((r)=> r > 2);
console.log(w);
console.log(z);

const x = [ 4, 5, 6 ];
const y = [ ...x ];
y.reverse().push(9);
console.log(x);
console.log(y);
```

확정된 ECMAScript 2023사양에서는 자바스크립트 프로그램을 더욱 예측 가능하고 유지보수하기 쉽게 만드는 데 도움이 되는 배열 객체에 대하여 직접 복사하여 사용하는 과정 대신 사용할 수 있는 여러가지 메서드를 소개하고 있다. 하나의 함수만 호출하면 되기 때문에 코드 작성이 쉬워지고 배열을 먼저 복사할 필요가 없기 때문에 읽기도 더 쉬워진다.

### 1. Array.prototype.toSorted

새롭게 정렬된 배열을 반환한다. sort함수에서 처럼 악센트 기호를 포함한 문자열, 숫자를 정렬 할 때에는 여전히 주의하여 원하는 결과를 생성하는 비교함수를 작성해야 한다.

```jsx
const languages = [ "Javascript", "Typescript", "Python" ];
const sorted = languages.toSorted();
console.log(sorted);
console.log(languages);

const numbers = [4, 2, 5, 10, 1];
const numberSorted = numbers.toSorted();
console.log(numberSorted );
const numberSorted Correctly = numbers.toSorted((a,b)=> a - b);
console.log(numberSortedCorrectly);

const strings = [ "abc", "äbc", "def" ];
const stringSorted = strings.toSorted();
console.log(stringSorted);
const stringSortedCorrectly = strings.toSorted((a, b) => a.localeCompare(b));
console.log(stringSortedCorrectly);
```

### 2. Array.prototype.toReversed

순서가 반전된 새 배열을 반환한다. 

```jsx
const languages = [ "Javascript", "Typescript", "Python" ];
const reversed= languages.toReversed();
console.log(reversed);
```

### 3. Array.prototype.toSpliced

splice는 제공된 인덱스 값의 요소를 삭제하고 추가하여 기존배열을 변경하고 삭제된 요소를 포함하는 배열을 반환한다.

```jsx
const languages = [ "Javascript", "Typescript", "Python" ];
const spliced= languages.splice(2, 1, "Java");
console.log(spliced);
console.log(languages);
```

toSpliced는 splice와 조금 다르게 작동한다. toSpliced는 제거된 요소 없이 추가된 요소만 포함하여 새로운 배열을 반환한다.

```jsx
const languages = [ "Javascript", "Typescript", "Python" ];
const spliced= languages.toSpliced(2, 1, "Java", "C++");
console.log(spliced);
console.log(languages);
```

### 4. Array.prototype.with

배열의 한 요소를 변경하기 위해 대괄호 표기법을 사용하는것과 같은 복사기능이다.

```jsx
const languages = ["Javascript", "Typescript", "Python"];
const updated = languages.with(2,"Java");
console.log(updated);
console.log(languages);
```

### 5. Array.prototype.findLast

find 메서드는 조건을 만족하는 요소가 배열에 있을 경우 가장 처음에 발견한 요소를 반환한다. findLast는 이름에서부터 알 수 있듯이 조건을 만족하는 첫번째 요소를 배열의 마지막 순서에서부터 찾아 반환한다.  기존에는 동일한 동작을 하기 위해 배열을 복사하고 reverse 메서드를 사용해야 했지만 findLast는 불필요한 단계 없이 필요한 동작만 수행할 수 있다.

```jsx
const targets = [ 12,34, 21, 2, 87, 599, 90];
const found = targets.find( r => r > 40 );
console.log(found);
const lastFound = targets.lastFind( r=> r > 40);
console.log(lastFound);
```

### 6. Array.prototype.findLastIndex

findLast가 배열의 뒷쪽부터 탐색하며 조건을 만족하는 요소를 반환했다면 findLastIndex는 해당 인덱스를 반환한다. 요소를 찾지 못했을 경우 -1을 반환한다.

```jsx
const targets = [ 20, 43, 1, 78, 5, 96, 18 ];
const index = targets.findLastIndex( r=> r< 10 );
console.log(index);
```