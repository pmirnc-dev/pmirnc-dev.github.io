---
layout: post
title: "리액트 vs 뷰"
date: 2024-04-24 09:00:00 +0900
author: ejsin
categories: react vue2 vue3
published: false
---

<hr>

## 프론트엔드 진영

현재 프론트엔드 영역에는 많은 프레임워크 및 라이브러리가 있습니다.
![ranking](/assets/images/ejsin/react_vs_vue/ranking.png)
(출처: https://www.boardinfinity.com/blog/the-most-popular-front-end-frameworks)

위 자료는 해외 유명 개발자 커뮤니티인 스택 오버플로우의 설문을 통해 만들어진 자료입니다.

한국의 상황을 보면 아시다시피 React 와 Vue 가 양대산맥을 이루고 있습니다.

- 사람인 : 리액트 1410, 뷰 696 (제이쿼리 677)
- 원티드 : 리액트 189, 뷰 12 (제이쿼리 12)
- 프로그래머스 : 리액트 140, 뷰 68 (제이쿼리 24)

리액트와 뷰는 SPA(Single Page Application)를 위해 만들어진 라이브러리 및 프레임워크지만

Next 와 Nuxt 등 강력한 **프레임워크의 프레임워크** 들 덕분에 SPA 의 단점들도 많이 상쇄되고 간단한 웹을 만들 때조차도 오히려 더 쉽게 셋업하고 빌드 될 수 있도록 많은 진화 과정을 거쳤습니다.

## React

### 클래스형 컴포넌트 (과거)

```javascript
import React, { Component } from "react";

class LifecycleCounter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {}

  componentDidUpdate(prevProps, prevState) {}

  componentWillUnmount() {}

  incrementCount = () => {
    this.setState((prevState) => ({
      count: prevState.count + 1,
    }));
  };

  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.incrementCount}>Increment</button>
      </div>
    );
  }
}

export default LifecycleCounter;
```

예전 리액트는 클래스 형으로 작성되었으며 뷰2 와 비슷하게 **생명 주기** 개념을 기본으로 사용했습니다.

### 함수형 컴포넌트 (현재)

```js
import React, { useState, useEffect } from "react";

function FunctionalCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("상태 변화");

    return () => {
      console.log("컴포넌트 제거");
    };
  }, [count]);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

export default FunctionalCounter;
```

위 형태가 이제 현재 함수형 컴포넌트의 전형입니다.

보시면 **_useState_** 와 **_useEffect_** 가 사용 되었습니다.

이런걸 Hooks 라고 부르는데 리액트 v16.8(2019년 2월)에 출시 되었습니다.

빌드, 배포, 최적화 작업 등을 제외하고 순수 화면 작업을 위한 리액트에서는

위 useState 와 useEffect 가 90퍼의 지분을 차지한다 해도 될 거 같습니다.

이 밖에도 useContext, useReducer, useMemo, useCallback, useRef 등

개발자들이 많이 쓰는 hooks 들이지만 필요할 때 찾아서 배워서 해도 무방할 정도입니다.

## Vue

### Vue 2

vue2 는 저희 프로젝트에서 워낙 많이 사용했기 때문에 많이 익숙합니다.

```js
<template>
  <div>
    <h1>Counter: {{ counter }}</h1>
    <button @click="increment">Increment</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      counter: 0
    };
  },

  methods: {
    increment() {
      this.counter++
    }
  },

  beforeCreate() {},

  created() {},

  beforeMount() {},

  mounted() {},

  beforeUpdate() {},

  updated() {},

  beforeDestroy() {},

  destroyed() {}
};
</script>

<style></style>
```
