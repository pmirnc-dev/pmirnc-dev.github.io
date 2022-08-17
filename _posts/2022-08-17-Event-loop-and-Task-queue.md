---
layout: post
title:  "이벤트 루프와 태스크 큐"
date:   2022-08-17 09:00:00 +0900
author: hsyoo
categories: javascript eventloop
---
<hr/>

## 개요 aka 떡밥
우리가 사랑하고 사랑해야만 하는 자바스크립트는 어떻게 동작할까요?  
흔히 자바스크립트는 "싱글 스레드(single thread)"로 작동한다고 한다고 합니다.  
싱글스레드라 함은 말 그대로 한 번에 하나에 작업을 수행한다는 말인 것 같은데  
어떻게 브라우저는 동시에 입/출력 등의 복잡한 작업을 처리하고   
node.js express에서는 수많은 http처리를 할 수 있는 것일까요? 

## 자바스크립트는 정말 싱글 쓰레드인가

자바스크립트의 메인 스레드인 "**이벤트 루프**"가 싱글 스레드이기 때문에 싱글 스레드가 맞습니다  
하지만 실제 자바스크립트가 런타임되는 브라우저나 node.js에서는 멀티 스레드"처럼" 작동됩니다.  
이런 런타임 환경이 싱글 스레드를 사용하는 자바 스크립트 엔진과 상호 연동하기 위해 사용되는 장치가  
"**이벤트 루프**"입니다  
그럼 뭔가 무시무시해 보이는 "**이벤트 루프**"는 도덕책 무엇..?

## 이벤트 루프(event loop)

이벤트 루프를 설명하는 아주 유명한 영상이 하나 있습니다.  

[![이벤트 루프](https://img.youtube.com/vi/8aGhZQkoFbQ/0.jpg)](https://www.youtube.com/watch?v=8aGhZQkoFbQ){:target="_blank"}
<br/>
<br/>
<br/>
스코틀랜드 출신의 개발자인 필립 로버츠의 키노트는 이벤트 루프를 설명하는 게 아주 유명한 영상입니다..  
완벽히 정리된 영상 같습니다. 시각적으로 구현한 위 영상에서  
콜스택(call stack), 이벤트 루프(event loop), 태스크 큐(task queue)가 모두 설명이 되었군요  
우리는 setTimeout은 web api 영역에서 구동이 된다는 것도 알았습니다.

###_이벤트 루프는 콜스택이 빈 것을 확인하면 태스크 큐의 작업을 하나씩 콜스택으로 가져와 처리를 한다_###
로 정리될 것 같군요


위 영상에서 빠진 게 있습니다 (위 영상은 2014년 키노트입니다)
태스크 큐에는 종류가 있다는 겁니다  

## 마이크로태스크(Microtasks)와 매크로태스크(Macrotasks)

위 영상에서 setTimeout이 태스트 큐에 들어갔지만 더 자세히 들여다보면  
매크로 태스크 큐에 들어가게 됩니다  

여기 킹받는 예제를 봅시다

```javascript

const log = console.log;

log('script start'); // A

setTimeout(function () { // B
    log('setTimeout');
}, 0);

Promise.resolve()
    .then(function () { // C
        log('promise1');
    })
    .then(function () { // D
        log('promise2');
    });

log('script end'); // E
```
출처: [링크](https://velog.io/@yejineee/%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%ED%83%9C%EC%8A%A4%ED%81%AC-%ED%81%90-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-%EB%A7%A4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-g6f0joxx)

위 코드를 봤을 때 어떤 순서로 실행이 될까요?

{% include codepen.html hash="oNqQpVa" title="마이크로 태스크 큐와 매크로 태스크큐" %}

결과는 다음과 같게 됩니다

```
script start A
script end E
promise1 C
promise2 D
setTimeout B

```

저는 처음에 A -> E -> B -> C- > D 순서로 실행될 거라고 생각했는데  
태스크 큐에는 종류가 있고, 그 종류들 사이에 우선 순위가 있다는 걸 알게 되었습니다.  

|구분|함수|
|---|---|
|매크로 태스크 큐<br/>(Macro task queue)|setTimeout, setInterval, setImmediate, requestAnimationFrame, I/O, UI 렌더링|
|마이크로태스크 큐<br/>(Micro task queue)|process.nextTick, **Promise**, Object.observe, MutationObserver|

위와 같은 태스크 큐의 종류가 있고,
**마이크로 태스크 큐**는 **매크로 태스크 큐**보다 우선 순위를 갖게 됩니다.

## 마치며
javascript개발자라고 자신있게 얘기하기 위해서는 동작 원리는 정확히 이해를 해야된다고 생각되서  
저도 어렴풋한 기억을 다시 더듬어 해당 내용들을 작성해봤습니다.  
재미없고 와닿을 수 없는 내용들일 수 있지만 fe / be에 모두 javascript를 사용하는  
피앰아이 개발자들에겐 꼭 알려주고 공유하고 싶었던 내용이었기에 시간을 내서 포스팅을 했습니다
그럼 20000

### 참고 자료
 - [https://velog.io/@yejineee](https://velog.io/@yejineee/%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%ED%83%9C%EC%8A%A4%ED%81%AC-%ED%81%90-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-%EB%A7%A4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-g6f0joxx)
 - [https://dkrnfls.tistory.com/362](https://dkrnfls.tistory.com/362)
 - [https://developer.mozilla.org/ko/docs/Web/API/HTML_DOM_API/Microtask_guide](https://developer.mozilla.org/ko/docs/Web/API/HTML_DOM_API/Microtask_guide)
 - [https://meetup.toast.com/posts/89](https://meetup.toast.com/posts/89)
