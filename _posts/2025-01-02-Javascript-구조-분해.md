---
layout: post
title:  "Javascript 구조 분해"
date:   2025-01-02 10:12:00 +0900
author: yrbyeon
categories: javascript
published: true
---
<hr/>

오늘은 자주 사용하는 javascript 배열, 객체를 사용할 때 코드를 깔끔하고 편리하게 쓰고 관리하기 위한 구조분해에 대해 정리해보겠습니다.

기본적인 내용이지만 정리에 초점을 맞추어 작성하였습니다.

### 1. 배열 구조 분해

```jsx
const score = [100,95,90,80,70,65,45,20];
```

위 배열에 점수가 내림차순으로 쓰여있습니다.

이 가운데 1위와 2위의 점수를 따로 꺼내 보상을 주고 나머지는 별개 변수로 분리를 하고 싶습니다.
```jsx
const score = [100,95,90,80,70,65,45,20];

const GoldScore = score[0];
const SilverScore = score[1];

GoldScore; //100
SilverScore; //95
```

위와 같이 배열의 index를 통해 값을 가져와 선정했습니다.

틀린 방법은 아니지만 이보다 간편한 방법이 있습니다.

분해를 이용하는 것입니다.

이 배열 대 괄호 안에 첫 번째 값을 gold, 두 번째 값을 silver라고 지정하겠습니다.

```jsx
const [gold, silver] = score;

gold; //100
silver; //95
```

score의 첫 요소가 gold, 두 번째 요소는 silver에 저장되었습니다.

이 작업은 개별 변수에 점수를 복사해서 쓰는 것이기 때문에 배열의 값에 영향을 주지 않습니다.

```jsx
let [gold2, silver2] = score;

--gold2; // 99
score; // [100,95,90,80,70,65,45,20]
```

만약 첫 번째, 두 번째 이외 나머지 값을 하나의 변수로 지정하고 싶을 때는 나머지 연산자와 함께 쓸 수 있습니다.

```jsx
const [gold, silver, ...elseThings] = score;

gold; //100
silver; //95
elseThings; // [90,80,70,65,45,20]
```

### 2. 객체 구조 분해

객체 구조 분해는 현재도 작업하며 가장 많이 사용하고 있습니다.

객체 구조 분해는 순서를 따르지 않기 때문에 배열 구조 분해 보다 실용적입니다.

```jsx
const user = {
	email: 'user000@gmail.com',
	password: 'user000!',
	firstName: '길동',
	lastName: '홍',
	born: 1910,
	died: 1940,
	city: 'Seoul'
}
```

user 객체 요소를 사용하기 위해 우리는 해당 구조 분해를 사용중입니다.

```jsx
const { firstName, lastName } = user;

firstName; //'길동'
lastName; //'홍'
```

마찬가지로 꺼내서 쓰는 것이기 때문에 구조 분해 하여 가져온 값을 변형해도 원래 객체의 값에 영향을 주지 않습니다.

```jsx
let { city } = user;

city = 'Incheon';
city; // 'Incheon'
user.city; // 'Seoul'
```

객체의 디폴트 값을 추가할 수 있습니다.

또다른 객체 human을 설정해보겠습니다. 해당 객체에는 died 요소가 없습니다.

```jsx
const human = {
	email: 'human000@gmail.com',
	password: 'human000!',
	firstName: 'Che guevara',
	lastName: 'Columbus',
	born: 2011,
	city: 'Bogota'
}
```

human 객체에 없는 died를 추가하여 해당 객체를 구조 분해해보겠습니다.

```jsx
const { city, born, died } = human;

city; // 'Bogota'
born; // 2011
died; // undefined
```

해당 객체에서 died 요소를 찾지 못했기에 undefined를 출력합니다.

이 때 해당 요소에 디폴트값을 설정할 수 있습니다.

```jsx
const { city, born, died = false } = human;

city; // 'Bogota'
born; // 2011
died; // false
```

died 값이 없으면 false가 뜨도록 설정했습니다.

반면 user 객체의 경우 died값이 있기 때문에 디폴트 값을 주더라도 died값이 출력됩니다.

```jsx
const { city, born, died = false } = user;

city; // 'Seoul'
born; // 1910
died; // 1940
```

변수 이름을 바꾼 경우에도 디폴트값을 추가할 수 있습니다.

```jsx
const { city, born, died: deathYear = false } = human;

city; // 'Bogota'
born; // 2011
deathYear; // false
```

### 3. 매개변수 분해

배열과 객체의 구조 분해는 배열과 객체를 매개변수로 사용할 때에도 활용할 수 있습니다.

현재 업무에서 많이 쓰이고 있지만 다시 한 번 정리해보겠습니다.

```jsx
const station = [
  {
    name: "논현",
    lines: 2,
    line: ["7", "신분당"],
    translate: true,
    location: "서울",
  },
  {
    name: "여의도",
    lines: 2,
    line: ["5", "9"],
    translate: true,
    location: "서울",
  },
  {
    name: "상록수",
    lines: 1,
    line: ["4"],
    translate: false,
    location: "경기",
  },
  {
    name: "등촌",
    lines: 1,
    line: ["9"],
    translate: false,
    location: "서울",
  },
  {
    name: "제물포",
    lines: 1,
    line: ["1"],
    translate: false,
    location: "인천",
  },
  {
    name: "문산",
    lines: 1,
    line: ["경의중앙"],
    translate: false,
    location: "경기",
  },
  {
    name: "마두",
    lines: 1,
    line: ["3"],
    translate: false,
    location: "경기",
  },
  {
    name: "별내",
    lines: 2,
    line: ["8", "경춘"],
    translate: true,
    location: "경기",
  },
  {
    name: "선릉",
    lines: 2,
    line: ["2", "수인분당"],
    translate: true,
    location: "서울",
  },
  {
    name: "김포공항",
    lines: 5,
    line: ["5", "9", "공항철도", "김포골드라인", "서해"],
    translate: true,
    location: "서울",
  },
  {
    name: "서울",
    lines: 6,
    line: ["1", "4", "공항철도", "경의중앙", "GTX-A", "KTX"],
    translate: true,
    location: "서울",
  },
];
```

위치가 경기인 역을 분류해보겠습니다.

```jsx
station.filter((station)=> station.location === "경기");
```

이 때 매개변수 자리에 구조 분해를 이용하니다.

```jsx
station.filter(({location})=> location === "경기");
```

여러 요소를 가져와서 쓸 수 있습니다.

```jsx
station
  .filter(({ translate }) => translate)
  .map(({ name, line }) => {
    return `${name}역은 ${line.join(",")} 으로 환승할 수 있습니다.`;
  });
```

## 마치며

구조 분해를 다시 한번 정리하며 그동안 제가 작성했던 코드들이 머릿속을 스칩니다. 이렇게 다시 한번 머릿속에 새기며 좀 더 깔끔하고 정리된 코드를 작성해야겠습니다.