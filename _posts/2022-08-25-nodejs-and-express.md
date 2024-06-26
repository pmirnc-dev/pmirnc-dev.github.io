---
layout: post
title:  "node.js와 express.js"
date:   2022-08-25 09:30:00 +0900
author: skan
categories: node.js express.js
---
<hr/>

## NODE.JS

---

### 개요

NODE JS 는 일반적으로 문법이 JavaScript 이며 Google Chrome V8 JavaScript 엔진을 사용해서 
JS를 실행할수 있는 런타임 실행환경 입니다.
즉 웹 브라우저 바깥에서도 누구나 V8엔진으로 Javascript를 사용할 수 있게 만들었다.
다양한 용도로 확장하기 위해 만들어진 것이 바로 Node.js입니다.

### 탄생배경

위의 개요를 이해하기 쉽게 말하자면 자바스크립트는 사실 브라우저에 내장되어 쓰이는 언어입니다.
다시 말해, 웹 브라우저에는 자바스크립트 실행을 위한 자바스크립트 엔진이 내장되어 있고 
이는 브라우저를 사용하는 모든 사용자는 자바스크립트를 언제든 실행 시킬 수 있는 프로그램을 가지고 있음을 의미합니다.

V8이 나오기 전까지 속도와 성능이 다른 언어에 비해 느리고 최적화가 안되서 그러한 프로젝트들은 인기를 끌지 못하였습니다.
그러는 와중 최적화와 성능이 좋은 크롬에 탑재된 V8 엔진이 세상에 나오자 그 엔진으로 만든 Node.js 의 인기가 높아지고 많은 사람들이 프로젝트에 기여를 해서 거대한 생태계가 구성되기 시작했습니다.

### 특징
 - 비동기 I/O 처리: Node.js 라이브러리의 모든 API는 비동기식(async)입니다, 멈추지 않는다는거죠 (Non-blocking). 
   Node.js 기반 서버는 API가 실행되었을때, 데이터를 반환할때까지 기다리지 않고 다음 API 를 실행합니다. 
   그리고 이전에 실행했던 API가 결과값을 반환할 시, Node.js의 이벤트 알림 메커니즘을 통해 결과값을 받아옵니다.
   

 - 빠른 속도: 구글 크롬(Google Chrome)의 V8 자바스크립트 엔진(JavaScript Engine)을 사용하여 빠른 코드 실행을 제공합니다.

   
 - 단일 쓰레드와 뛰어난 확장성: Node.js는 이벤트 루프와 함께 단일 쓰레드 모델을 사용합니다. 
   이벤트 메커니즘은 서버가 멈추지않고 반응하도록 해주어 서버의 확장성을 키워줍니다.

   
 - 버퍼링이 없다: Node.js 어플리케이션엔 데이터 버퍼링이 없고, 데이터를 chunk로 출력합니다.

### 장점

 - 자바스크립트를 동일하게 사용해서 서버단 로직을 처리할 수 있다는게 가장 큰 장점 FRONT 개발자는 새로운 언어를 습득하지 않고도 자바스크립트를 활용해 서버기술을 빨리 향상시킬 수 있다.


 - 이벤트 기반 비동기방식이라 서버 무리가 적다.


 - npm(node package manager)을 통한 다양한 모듈(패키지) 제공 npm을 이용해 자신이 필요한 라이브러리와 패키지를 검색해서 설치하고 사용할 수 있기 때문에 효율성이 좋다.


 - 구글이 만드는 JavaScript 엔진을 사용한다. 구글은 V8 엔진 성능 업그레이드를 계속 하고 있다.


 - C++로 개발된 V8 JavaScript 엔진이기 때문에 확장성이 좋다.

---

## Express란?

익스프레스(Express.js)는 노드(NodeJS) 상에서 동작하는 웹 개발 프레임워크입니다. 
이외에도 Hapi.js, Koa.js 등 다양한 웹프레임워크이 있지만 현재까지 가장 많이 사용하는 것이 바로 익스프레스 엔진입니다.

익스프레스는 가볍고 유연하게 웹 프레임워크을 구성할 수 있는 장점이 있습니다. 
이것은 미들웨어(Middleware) 구조 때문에 가능한 것입니다. 
자바스크립트 코드로 작성된 다양한 기능의 미들웨어는 개발자가 필요한 것만 선택하여 익스프레스와 결합해 사용할 수 있습니다. 

>미들웨어는 req(요청) 객체, res(응답) 객체, 그리고 어플리케이션 요청-응답 사이클 도중 그 다음의 미들웨어 함수에 대한 엑세스 권한을 갖는 함수이다.
>미들웨어란 간단하게 말하면 클라이언트에게 요청이 오고 그 요청을 보내기 위해 응답하려는 중간(미들)에 목적에 맞게 처리를 하는 과정을 말한다.

결국 express란 NodeJS를 사용하여 쉽게 서버를 구성할 수 있게 만든 클래스와 라이브러리의 집합체라고 보면된다.


### Express 설치
express의 설치에 앞서, 두 가지를 선행해야 한다.
첫 번째로 NodeJS의 설치, 그리고 두 번째로 npm init을 통한 프로젝트 initialize가 되겠다.
그리고, npm install express 명령어로 express를 설치해주면 되시겠다.

### Express의 사용
express 홈페이지에 방문하면, 시작방법조차 아주 친절히 나와있다.

```jsx
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```
위와 같이 작성하면, 포트 3000에 그 유명한 'Hello World' 메시지를 확인할 수 있다.

---
 
