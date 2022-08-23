---
layout: post
title:  "Cartesian Product을 활용한 쿼터(Quota) 구현"
date:   2022-08-19 09:30:00 +0900
author: hsyoo
categories: javascript algorithms 곱집합 쿼터
---
<hr/>

## Cartesian Product가 뭡니까?

카테시안 프로덕스란 무엇입니까?

> 집합론에서 곱집합(곱集合, 영어: product set , product) 또는 데카르트 곱(Descartes곱, 영어: Cartesian product 카티지언 프로덕트[*])은 각 집합의 원소를 각 성분으로 하는 튜플들의 집합이다.    
> 곱집합은 집합의 다양체에서의 직접곱이며, 집합의 범주에서의 곱이다.

아래 이미지를 참고하시면 이해가 편합니다

![https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Cartesian_Product_qtl1.svg/1280px-Cartesian_Product_qtl1.svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Cartesian_Product_qtl1.svg/1280px-Cartesian_Product_qtl1.svg.png)


위키에 하나의 예제가 더 있군요  
52장의 포커 패의 집합입니다.

![https://upload.wikimedia.org/wikipedia/commons/0/02/Piatnikcards.jpg](https://upload.wikimedia.org/wikipedia/commons/0/02/Piatnikcards.jpg)

52장의 포커 패의 집합은 모양의 집합 {♠, ♥, ♣, ♦}과 숫자의 집합 {2, ..., 10, J, Q, K, A}의 곱집합이라 생각할 수 있습니다  
_출처: 위키백과_

## 우선 온라인 조사에서 쿼터가 뭔지 간단하게 살펴봅시다

온라인 설문 조사에는 **쿼터(Quota)**라는 개념이 있는데 뜻 그대로 **할당**이라는 뜻 입니다.  

"A조사"라는 설문 조사가 있다고 예를 들겠습니다.  
"A조사"에는 성별과 연령 **조건**이 존재한다고 가정하고  
남녀 성별 조건과 20-30대 연령 조건을 추가하였습니다.

해당 조건으로 **쿼터**없이 진행을 하게 된다면 어떻게 될까요?
아래와 같이 모집이 될 수도 있을 겁니다

|연령|성별|모집 수|
|:---|:---|:---|
|20대|남성|150|
|20대|여성|50|
|30대|남성|50|
|30대|여성|150|

위와 같은 경우는 성별과 연령에 일부 쏠림 현상이 벌어졌기에   
편향된 조사 결과라고 생각될 여지가 매우 크겠죠

이 문제를 해결하고자 아래와 같은 **쿼터**를 적용합니다.

|연령|성별|모집 수|
|:---|:---|:---|
|20대|남성|100|
|20대|여성|100|
|30대|남성|100|
|30대|여성|100|

## 쿼터와 catesian product가 무슨 상관?

**쿼터**를 설정에는 다양한 문항이 활용될 수 있습니다.  
위 예제에서는 성별(남,녀)x연령(20대, 30대) 2개 문항 만이 활용되어 4개의 쿼터가 생겼습니다.
javascript로 보면 아래와 같겠네요.

```javascript

const age = [ '20대', '30대' ];
const gender = ['남성', '여성'];

console.log(age.length * gender.length);
// 결과는 4

```

하지만 보기의 개수가 많은 문항이 존재할 수 있습니다.
성별(남,녀)x연령(20대, 30대)가 이미 존재하고
여기에 17개 시도의 지역을 추가해 볼까요?

```javascript

const age = [ '20대', '30대' ];
const gender = ['남성', '여성'];
const region = ["서울특별시", "부산광역시", "대구광역시", "인천광역시", "광주광역시", "대전광역시", "울산광역시", "세종특별자치시", "경기도", "강원도", "충청북도", "충청남도", "전라북도", "전라남도", "경상북도", "경상남도", "제주특별자치도"];
console.log(age.length * gender.length * region.length);
// 결과는 68
```


|연령|성별|지역|모집 수|
|:---|:---|:---|:---|
|20대|남성|서울특별시|100|
|20대|여성|서울특별시|100|
|30대|남성|서울특별시|100|
|30대|여성|서울특별시|100|
|20대|남성|부산광역시|100|
|20대|여성|부산광역시|100|
|30대|남성|부산광역시|100|
|30대|여성|부산광역시|100|

...

위 처럼 이루어지겠군요. 이정도면 오히려 다행입니다  
여기에 한가지 문항을 더 추가할 수도 있습니다

성별(남,녀)x연령(20대, 30대)x지역(17개시도)가 이미 존재하고  
사용 휴대폰 제조사(삼성, 애플, 구글, 모토로라, 화웨이, 샤오미, 레노버 7개)를 추가해보겠습니다.

```javascript

const age = [ '20대', '30대' ];
const gender = ['남성', '여성'];
const region = ["서울특별시", "부산광역시", "대구광역시", "인천광역시", "광주광역시", "대전광역시", "울산광역시", "세종특별자치시", "경기도", "강원도", "충청북도", "충청남도", "전라북도", "전라남도", "경상북도", "경상남도", "제주특별자치도"];
const manufacturers = ["삼성", "애플", "구글", "모토로라", "화웨이", "샤오미", "레노버"];
console.log(age.length * gender.length * region.length * manufacturers.length);
// 결과는476
```
  
  
|연령|성별|지역|휴대폰 제조사|모집 수|
|:---|:---|:---|:---|:---|
|20대|남성|서울특별시|삼성|100|
|20대|여성|서울특별시|삼성|100|
|30대|남성|서울특별시|삼성|100|
|30대|여성|서울특별시|삼성|100|
|20대|남성|서울특별시|애플|100|
|20대|여성|서울특별시|애플|100|
|30대|남성|서울특별시|애플|100|
|30대|여성|서울특별시|애플|100|

...

표로 그릴 수 없을 정도로 수가 많아졌습니다

정해진 개수가 없는 **보기**를 가진 **문항**들이 몇 개나 추가될 지 모르는
상황을 어떻게 javascript에서 해결할 수 있을까요?

위 예제에서 보셨다시피 쿼터는 기준이 되는 **문항(성, 연령)**과 **보기(남성, 20대)**가
**곱집합** 형태로 구성이 됨을 알 수 있습니다.

## 레츠 두잇 두잇

우버의 시니어 엔지니어인 "Oleksii Trekhleb"이 정리한 javascript 알고리즘을 참고 했습니다  
(해당 github는 자바스크립트로 구현되는 알고리즘을 잘 정리된 저장소기 때문에 많은 참고 바랍니다)

```javascript

function cartesianProduct(setA, setB) {
  // Check if input sets are not empty.
  // Otherwise return null since we can't generate Cartesian Product out of them.
  if (!setA || !setB || !setA.length || !setB.length) {
    return null;
  }

  // Init product set.
  const product = [];

  // Now, let's go through all elements of a first and second set and form all possible pairs.
  for (let indexA = 0; indexA < setA.length; indexA += 1) {
    for (let indexB = 0; indexB < setB.length; indexB += 1) {
      // Add current product pair to the product set.
      product.push([setA[indexA], setB[indexB]]);
    }
  }

  // Return cartesian product set.
  return product;
}

```

_결과를 보시려면 "Result"탭을 선택해주세요_

{% include codepen.html hash="zYWbyWx" title="cartesian product" %}


함수를 중첩해서 사용하면서 해결이 됐습니다.  
다만 다중 배열이 들어가니 좀 보기 불편하네요.
좀 array의 reduce를 활용해서 커스텀해보겠습니다.
(아래는 typescript로, codepen에선 javascript로 구현)


```typescript

  function cartesianProduct<T>(arr: T[][]): T[][] {
    return arr.reduce(
      (a, b) => {
        return a
          .map((x) => {
            return b.map((y) => {
              return x.concat(y);
            });
          })
          .reduce((c, d) => c.concat(d), []);
      },
      [[]] as T[][],
    );
  }

```


_결과를 보시려면 "Result"탭을 선택해주세요_
{% include codepen.html hash="XWEGoBJ" title="cartesian product" %}

2중 배열로 잘 구현된 결과를 보실 수 있습니다.  
해당 로직을 이용해서 fe에서 레이아웃을 그리고 입력할 수 있는 표를 쉽게 그려줄 수 있겠네요.  

위처럼 typescript를 활용하면 generic하게 구현할 수 있겠습니다.

[https://github.com/trekhleb/javascript-algorithms/tree/master/src/algorithms/sets/cartesian-product](https://github.com/trekhleb/javascript-algorithms/tree/master/src/algorithms/sets/cartesian-product){:target="_blank"}
