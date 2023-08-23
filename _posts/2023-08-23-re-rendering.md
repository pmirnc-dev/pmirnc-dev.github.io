---
layout: post
title:  "re-rendering"
date:   2023-08-23 09:25:07 +0900
author: tkkim
categories: Vue
---
<hr/>


# re-rendering 해야 할 상황이 온다면?
<hr/>
## re-rendering

### 리랜더링(re-rendering)

![Untitled](/assets/images/tkkim/20230823/1.png)

뷰는 데이터가 변경되었을 경우 화면이 갱신되는 **반응형 시스템**을 가지고 있다.

컴포넌트 인스턴스에 있는 `watcher` 가 데이터의 수정을 감지하면 재 랜더링이 되는 시스템

뷰는 배열과 객체의 수정 시 재 랜더링이 되지 않는 경우가 있고 퀵폴 업데이트 작업 중 해당 이슈를 맞이하게 되었다.

### 배열의 경우

- 인덱스로 배열에 있는 항목을 직접 설정 `vm.items[index] = newValue`
- 배열 길이를 수정 `vm.items.length = newLength`

⇒  변경사항 감지 불가능

- 변경 감지를 위해 해당 메소드를 사용해 갱신하면 된다.
    - `push()`
    - `pop()`
    - `shift()`
    - `unshift()`
    - `splice()`
    - `sort()`
    - `reverse()`



### 객체의 경우

Vue는 속성의 추가, 제거 감지 불가능

변경 감지를 위해서는 다음과 같은 방법이 있다.

- 전체 페이지 재 랜더링
- 본인 포함 하위 컴포넌트만 재 랜더링
    - `this.$forceUpdate();`
    - 랜더링 되어야하는 컴포넌트의 `:key` 키값을 설정해 업데이트
- Watch 사용

## 변경 감지 방법

### forceUpdate

```jsx
	function(){		
        //강제 랜더링
        this.$forceUpdate();
        
        // 렌더링 이후에 필요한 작업 수행
        this.$nextTick(function(){}
    }
```

과도하게 사용할 시 애플리케이션의 성능하락 주의!

### `:key` 키값 설정  ★

```jsx
<template>
		<custom-footer
            @register="validateForm"
            @goList="goList"
            @temporarySave="temporarySave"
            @preview="preview"
          ></custom-footer>
    
          <v-dialog v-model="formSample" width="540" persistent scrollable>
            <form-sample @hide="hideDialog" :key="componentKey"> </form-sample>
          </v-dialog>

</template>
<script lang="ts">
@Component({
  components: {
    FormSample,
  },
})
export default class HomeView extends Vue {
 
  preview() {
    this.componentKey += 1;
  }
</script>

```

key 값을 컴포넌트에 매핑해 props로 내려주기

→ key값이 변할 경우 예전 component를 지우고 새 component 생성

미리 보기 클릭 시 preview function 내에 있는 key값이 변하면서 재 랜더링 된다.

### Watch

객체의 내용이 변경되어도 객체 컨테이너는 변경되지 않을 때

즉 watch가 속성 내부가 변경되었다고 생각하지 않을 경우

속성 내부를 검사할 수 있게 알려줘야 한다. - deep: true 설정, handler 메소드 재배치

```jsx
//vue-property-decorator 문법
@Component
export default class Home extend Vue {
  public watchExample: string = '예시임';

	//감시할 속성명, 
  @Watch("watchExample", {immediate: true, deep: true}) 
  public exampleMethod(새로운 값을 넣을 변수, 이전 값을 넣을 변수) {
    console.log(새로운 값을 넣을 변수, 이전 값을 넣을 변수)
  }
}
```

deep - 변경 감지가 필요할 경우

immediate - 변경 & 페이지 읽어들였을 때의 처리가 필요할 경우

[참조]

[https://michaelnthiessen.com/force-re-render](https://michaelnthiessen.com/force-re-render)

[https://hyeonyeee.tistory.com/97](https://hyeonyeee.tistory.com/97)

[https://y-chyachya.tistory.com/11](https://y-chyachya.tistory.com/11)

[https://minho-jang.github.io/development/5/](https://minho-jang.github.io/development/5/)

[https://ui.toast.com/weekly-pick/ko_20190307](https://ui.toast.com/weekly-pick/ko_20190307)

[https://kyounghwan01.github.io/blog/TS/Vue/grammer/#watch](https://kyounghwan01.github.io/blog/TS/Vue/grammer/#watch)

[https://engineer-mole.tistory.com/335](https://engineer-mole.tistory.com/335)