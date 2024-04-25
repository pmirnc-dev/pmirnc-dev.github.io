---
layout: post
title: "리액트 vs 뷰"
date: 2024-04-24 09:00:00 +0900
author: ejsin
categories: react vue
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

```jsx
import React, { Component } from "react"

class LifecycleCounter extends Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
  }

  componentDidMount() {}

  componentDidUpdate(prevProps, prevState) {}

  componentWillUnmount() {}

  incrementCount = () => {
    this.setState((prevState) => ({
      count: prevState.count + 1,
    }))
  }

  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.incrementCount}>Increment</button>
      </div>
    )
  }
}

export default LifecycleCounter
```

예전 리액트는 클래스 형으로 작성되었으며 뷰2 와 비슷하게 **생명 주기** 개념을 기본으로 사용했습니다.

### 함수형 컴포넌트 (현재)

```jsx
import React, { useState, useEffect } from "react"

function FunctionalCounter() {
  const [count, setCount] = useState(0)
  const [items, setItems] = useState(['a', 'b', 'c'])

  useEffect(() => {
    console.log("상태 변화")

    return () => {
      console.log("컴포넌트 제거")
    }
  }, [count])

  return (
    <div>
      {count && <h1>Count: {count}</h1>}
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ul>
        {items.map(item => <li key={item}>{item}</li>)}
      </ul>
    </div>
  )
}

export default FunctionalCounter
```

위 형태가 이제 현재 함수형 컴포넌트의 전형입니다.

보시면 **_useState_** 와 **_useEffect_** 가 사용 되었습니다.

이런걸 Hooks 라고 부르는데 리액트 v16.8(2019년 2월)에 출시 되었습니다.

빌드, 배포, 최적화 작업 등을 제외하고 순수 화면 작업을 위한 리액트에서는

위 useState 와 useEffect 가 90퍼의 지분을 차지한다 해도 될 거 같습니다.

이 밖에도 useContext, useReducer, useMemo, useCallback, useRef 등

(https://react.dev/reference/react/hooks)

개발자들이 많이 쓰는 hooks 들이지만 필요할 때 찾아서 배워서 해도 무방할 정도입니다.

## Vue

### Vue 2

vue2 는 저희 프로젝트에서 워낙 많이 사용했기 때문에 많이 익숙합니다.

```vue
<template>
  <div>
    <h1 v-if={}>Counter: {{ counter }}</h1>
    <button @click="increment">Increment</button>
    <input v-for={} v-model={counter} />
  </div>
</template>

<script>
export default {
  data() {
    return {
      counter: 0
    }
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
}
</script>

<style></style>
```

리액트와 뷰 크게 차이는 없다고 생각합니다. 리액트 하다가 뷰를 가거나 뷰를 하다가 리액트를 해도 조금 헷갈릴 뿐이지
결국 **"상태가 변하면 리렌더링 된다"** 를 염두에 두면 작업하는데 큰 문제는 없습니다.

오히려 뷰 같은 경우 v-model 을 이용한 ***양방향 데이터 바인딩*** 을 가능하게 합니다.

```vue
<template>
  <input v-model={counter} />
  <p>{{counter}}</p>
</template>

<script>
  export default {
    data() {
      return {
        counter: 0
      }
    }
  }  
</script>
```

리액트는 엄격하게 단방향 데이터 바인딩만 가능하게 만듭니다.

```jsx
const [counter, setCounter] = useState('')

const handleChange = (e) => {
  setCounter(e.target.value)
}

return (
  <>
    <input value={counter} onChange={handleChange} />
    <p>{counter}</p>
  </>
)
```

### Vue 3

```vue
<template>
  <div>
    <h1>Counter: {{ counter }}</h1>
    <button @click="increment">Increment</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const counter = ref(0)

const increment = () => {
  counter.value++
}
</script>
```

vue3 은 개인적으로 제일 편한거 같습니다.

그냥 상태값을 ref 로 정의하고 그냥 로직을 작성하기만 하면 ***반응성*** 덕분에 쉽게 바로 뷰(view)에 렌더링 됩니다.

react 가 러닝 커브가 높다고 하는데 저는 개인적으로 vue2 도 만만치 않다 생각했습니다.

근데 확실히 vue3은 진짜 쉽고 제일 괜찮았습니다.

### 리액트의 러닝 커브

리액트의 러닝 커브가 높다고 합니다. 틀린 말은 아닌거 같습니다.

보통은 jsx 와 template 문법의 차이, react 의 데이터 바인딩 등등을 얘기하면서 러닝 커브에 대해 언급하는데 그것도 맞지만

가장 큰 경우는 두 가지라고 봅니다.

1. 리액트의 클로저(closure) 활용

밑에 코드처럼 

```jsx
function TimerComponent() {
    const [seconds, setSeconds] = useState(0)

    useEffect(() => {
        const interval = setInterval(() => {
            console.log(seconds)
        }, 1000)

        return () => clearInterval(interval)
    }, [seconds])

    const incrementSeconds = () => {
        setSeconds(seconds + 1)
    }

    return (
        <div>
            <h1>{seconds}</h1>
            <button onClick={incrementSeconds}>시간 증가</button>
        </div>
    )
}
```

2. 불변성

참조값이 변하면 안되고 새로운 참조값을 생성해야 합니다.

```jsx
function BadCounter() {
  const [items, setItems] = useState([1, 2, 3])

  // 잘못된 방법 (원본 상태 변경)
  const addItemBad = () => {
    items.push(items.length + 1)
    setItems(items)
  }

  // 올바른 방법 (새 배열 생성)
  const addItemGood = () => {
    etItems([...items, items.length + 1])
  }

  return (
    <div>
      <button onClick={addItemGood}>Add Item</button>
      {items.map(item => <div key={item}>{item}</div>)}
    </div>
  )
}
```

### 뷰의 답답함

뷰는 다 좋은데 지원되는 라이브러리나 타입스크립트에서 약간 한계가 있습니다.

1. 지원되는 라이브러리 문제

하나의 예시를 들면 bootstrap vue (https://bootstrap-vue.org/) 를 보면 아주 정리가 잘 되어 있고 편리하게 사용 가능합니다.

하지만 이제 vue3을 많이 사용하는데 bootstrap vue 3 (https://www.npmjs.com/package/bootstrap-vue-3) 같은 경우

참고할 문서도 없고 bootstrap vue 의 문서를 참고해도 지원되지 않는 기능이 너무 많습니다.

bootstrap vue next (https://www.npmjs.com/package/bootstrap-vue-next) 는 그나마 낫지만 안되는 기능이 꽤 있습니다.

2. 타입스크립트 지원

뷰 같은 경우 타입스크립트 지원이 잘 되는걸로 알고 있습니다. 그러나 뷰의 프레임워크 nuxt 에 들어가보면 상황이 좀 다릅니다.

이제 nuxt 같은 경우 서버 사이드 렌더링 이 중요한데 아래처럼 클라이언트와 서버 간에 데이터 타입 불일치 문제가 많습니다.

```vue
// store.state.user 타입이 정확히 없으면 에러 발생

export default function ({ store, redirect }) {
  if (!store.state.user) {
    redirect('/login')
  }
}
```

### React vs Vue

개인적으로 Vue3 이 가장 사용하기 편했습니다.

하지만 위에서 언급한거 외에도

1. 구조화

컴포넌트를 얼마나 잘 구조화 하고 재사용하기 쉬운지에 관해서는 리액트 와 뷰 모두 나쁘지 않지만

확실히 **리액트** 가 jsx 를 적극적으로 지원하는 만큼 좀 더 컴포넌트화 친화적입니다.

2. 테스트

테스트 같은 경우는 제가 테스트 코드를 적극적으로 작성하지 않아 잘 모르지만

위 컴포넌트화에서도 보이듯이 확실히 **리액트** 가 쉽고 뷰는 그리 테스트 친화적이지는 않다고 합니다.

3. 모바일

리액트는 React Native 라는 엄청난 무기가 있습니다.

물론 이제 공식적인 지원은 거의 하지 않고 지원되는 기능도 안드로이드, ios 보다 적지만

정말 인터랙티브한 기능이 없는 경우 안드로이드, ios 를 둘 다 사용할 수 있는 좋은 선택지고

웹뷰를 이용한 방법이라면 더욱 문제 없습니다. 

물론 뷰도 NativeScript 와 Weex 가 있지만 리액트 네이티브에 비해 많은 면에서 밀린다고 들었습니다.

4. **커뮤니티**

가장 중요하다고 생각하는데 커뮤니티가 크고 사용자가 많으니 발전이 빠르고, 일자리 수도 많아서 사람 구하기도 쉽고, 많은 자료가 있고,
또 그래서 인공지능이 학습하기 좋고, 지원되는 라이브러리도 많고 등등 이게 최대의 강점 같습니다.

---

둘 다 훌륭한 프론트엔드 프레임워크라 무얼 쓰든 하나만 잘 배워도 나머지 하나를 배우는데 엄청 좋으니

`급하게 언어를 선택해야 한다면 잘하는 걸로`, `시간이 많다면 NEXT 로` 하는게 제일 좋을 거 같습니다.