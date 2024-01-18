---
layout: post
title:  "msw로 원할하게 프론트 테스트하기"
date:   2024-01-15 00:00:00 +0900
author: ejsin
categories: [msw, testing]
---

<hr />

# MSW

**mock service worker**

클라이언트에서 서버로 보내는 HTTP 요청을 가로채 모의로 클라이언트에 응답해주는 라이브러리입니다.

실제로 서버에 응답이 가지 않고 원하는 응답을 임의로 만들어 클라이언트 단에서 **모의** api 테스트를 할 수 있습니다.

혹시나 백엔드 개발자가 바빠 api 명세를 늦게 주거나 작업이 늦어질 때 프론트단에서 네트워크 요청을 재현해 쉽게 테스트하고 작업할 수 있습니다.

### service worker 란?
[service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)

웹 브라우저에서 백그라운드에서 실행되는 스크립트입니다.

(웹 페이지와는 별도로 작동하며 백그라운드 동기화, 푸시 알림, 캐싱 등 용도로도 사용 가능)

## msw 라이브러리 설치
```
npm install msw -D
```
당연히 개발 단계에서만 사용하기 때문에 devDependencies 로 설치하는게 좋습니다.

## worker 스크립트 작성
```
npx msw init ./public
```
msw 를 사용하기 위해 service worker 를 먼저 브라우저에 설치해줘야 합니다.

위 명령어를 통해서 자동으로 스크립트를 만들어 줍니다.

![msw_script_file](/assets/images/ejsin/msw/msw_script_file.png)

## handlers

handlers 에서 실제 네트워크 요청을 가로채고 이에 대응하는 모의 응답을 정의하는 함수들을 작성합니다.

ex) '/api/user/sign-in' 로 post 요청을 보냈을 때

```javascript
// handlers.js

import { http, HttpResponse } from 'msw'

const signIn = () => {
    return http.post('/api/user/sign-in', () => {
        return HttpResponse.json({ result: true, name: '신은진' })
    })
}

export const handlers = [
    signIn()
]
```

## browser

browser 파일에 브라우저의 worker 에서 사용할 handlers 를 등록하면 됩니다.

```javascript
// browser.js

import { setupWorker } from 'msw/browser'
import { handlers } from './handlers'

export const worker = setupWorker(...handlers)
```

## worker 시작

worker.start() 로 worker 를 실행시킵니다.

```javascript
// main.ts

import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import vuetify from './plugins/vuetify'
import '@/assets/styles/global.css'
import VueAxios from "vue-axios";
import axios from "@/utils/axios";

Vue.config.productionTip = false

Vue.use(VueAxios, axios);

if (process.env === 'development') {
  (async function bootMSW() {
    const { worker } = await import('./mocks/browser.js' as any)
    return worker.start()
  })();
}

new Vue({
  router,
  store,
  vuetify,
  render: h => h(App)
}).$mount('#app');
```

#
![ex1_result](/assets/images/ejsin/msw/ex1_result.png)

handlers 에서 등록했던 응답값이 잘 나옵니다.

## 정리

1. msw npm 설치

2. msw 를 브라우저의 service worker 에 설치 (npx msw init [해당 프로젝트의 public 디렉토리])

3. src/mocks 
   - handlers.js (mock 함수들 작성)
   - browser.js (service worker 에 handlers 등록)

4. 프로젝트에서 worker 실행 ( worker.start() )

---

### Reference

https://mswjs.io/