---
layout: post
title:  "useReducer"
date:   2025-01-02 09:30:00 +0900
author: ybcho
categories: useReducer, react
published: true
---
<hr/>



## useReducer란?

React의 내장 훅 중 하나로, 복잡한 상태 업데이트 로직을 useState와 다르게 reducer 함수로 분리하여 관리할 수 있습니다.

### 언제 사용해야 할까?
- 단일 컴포넌트 내에 복잡한 상태 관리에 적합합니다.
- 상태 객체가 여러 개의 하위 값을 갖거나, 상태 업데이트 로직이 복잡한 경우에 사용하기 좋습니다.
- reducer 함수는 컴포넌트 외부로 분리할 수 있기 때문에, 동일한 상태 업데이트 로직을 재사용이 필요할때 사용하기 졸습니다.
- Redux와 유사한 패턴을 사용하기 때문에, Redux로 전환할 계획이 있을 때 사용하기 좋습니다.


## useReducer 구성 요소
### State
현재 상태 값입니다. state는 직접 수정할 수 없으며 오직 액션(Action)에 의해서만 변경 되기 때문에, 상태 변화를 예측하고 추적하기 쉽습니다.

```javascript
const initialState = {
  cart: [
    { id: 1, name: "상품 A", price: 10000, quantity: 2 },
    { id: 2, name: "상품 B", price: 5000, quantity: 1 },
  ],
  user: {
    isLoggedIn: false,
    name: null,
  },
  selectedProduct: null,
};
```

### Action
action은 무슨 일이 일어났는지를 설명하는 단순한 객체입니다. 
<br/>상태를 변경하는 유일한 방법은 action을 발생시키는 것입니다. 

- type 속성(필수): action의 종류를 나타내는 문자열입니다. 일반적으로 대문자와 _를 사용하여 정의합니다 (예: ADD_TO_CART, USER_LOGIN).
- payload 속성(선택): action과 관련된 추가 데이터가 포함됩니다. 예를 들어, 장바구니에 추가할 상품 정보, 로그인한 사용자 정보 등이 payload에 포함될 수 있습니다.

### Reducer
Reducer는 현재 state와 action을 받아서 새로운 state를 반환하는 함수 입니다.
<br/>기존 상태를 직접 수정하지 않고, 항상 새로운 상태 객체를 생성하여 반환합니다.
<br/>일반적으로 switch 문을 사용하여 action.type에 따라 다른 상태 업데이트 로직을 처리합니다.

```javascript
function cartReducer(state = [], action) {
    switch (action.type) {
        case 'ADD_TO_CART':
            return [...state, action.payload];
        case 'REMOVE_FROM_CART':
            return state.filter(item => item.id !== action.payload.id);
        default:
            return state;
    }
}
```

### Dispatch
Dispatch는 action을 발생시키는 함수입니다. dispatch 함수에 action 객체를 전달하면, reducer가 호출되고 상태가 업데이트됩니다.

```javascript
import React, {useReducer} from 'react';

const [state, dispatch] = useReducer(cartReducer, initialState); //첫번째는 reducer, 두번째는 state 전달

dispatch({
  type: 'ADD_TO_CART',
  payload: { id: 3, name: '상품 C', price: 15000, quantity: 1 }
});
```

### 장점

복잡한 상태 로직을 효율적으로 관리

- reducer 함수는 action 타입에 따라 상태를 업데이트하는 로직을 정의하기 때문에, 상태 변화의 흐름을 쉽게 파악할 수 있습니다.

불필요한 리렌더링 방지 (최적화 가능성)

- dispatch 함수를 사용해 여러 상태를 한 번에 업데이트할 수 있어 불필요한 리렌더링을 줄일 수 있습니다.

예측 가능한 상태 변화

- useReducer는 action 기반으로 상태를 업데이트하기 때문에, 상태 변화의 원인과 결과를 명확하게 파악할 수 있습니다.

### 단점

보일러플레이트 코드

- useState에 비해 더 많은 코드를 작성해야 하고, 간단한 상태 변경을 위해서도 action 타입을 정의하고, reducer 함수에서 해당 action을 처리하는 로직을 작성해야 합니다.

컴포넌트 외부 상태 관리의 한계

- 컴포넌트 내부의 상태를 관리하는 데는 효과적이지만, 여러 컴포넌트 간에 상태를 공유하는 데는 적합하지 않습니다.
<br/>이의 경우 Redux 전역 상태 관리 라이브러리를 사용하거나 Context API를 사용하는 것이 좋습니다


<hr />
그 동안 useReducer의 필요성을 느끼지 못하고 얼마 전 견적 관리 페이지 작업을 하면서 많이 복잡한 상태 관리를 해야했는데 useState로 관리 했기 때문에 예상치 못한 업데이트가 일어나거나
기능 수정을 할 때 군데군데 수정할 부분을 찾기 어려웠습니다. 이를 계기로 useReducer를 알아봤고, 상태 업데이트 로직을 분리하여 직관적으로 볼 수 있는 useReducer를 사용하고 컴포넌트를 더 분리하였더라면
좀 더 수월하게 작업을 할 수 있었을 것을 깨달았습니다. 앞으로는 상황에 따라 useState, useReducer 혹은 필요에 따라 redux를 사용할지 잘 판단해야겠습니다    

### 참고
[https://youtu.be/tdORpiegLg0?si=Y295iYAmv64tICEE](https://youtu.be/tdORpiegLg0?si=Y295iYAmv64tICEE)
