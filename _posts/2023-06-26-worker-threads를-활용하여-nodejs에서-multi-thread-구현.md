---
layout: post
title:  "worker threads를 활용하여 nodejs에서 multi thread 구현"
date:   2023-06-26 09:00:00 +0900
author: hsyoo
categories: nodejs worker-threads multi-thread
---

<hr/>


# 스레드?

> 스레드(thread)는 어떠한 프로그램 내에서, 특히 프로세스 내에서 실행되는 흐름의 단위를 말한다. 일반적으로 한 프로그램은 하나의 스레드를 가지고 있지만, 프로그램 환경에 따라 둘 이상의 스레드를 동시에 실행할 수 있다. 이러한 실행 방식을 멀티스레드(multithread)라고 한다
>  [https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85)](https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85)){:target="_blank"}

컴퓨터 공학에서 스레드는 위와 같이 정의되고 있습니다.

자바스크립트는 싱글 스레드로 동작합니다.

이 점은 우리에게 꽤 유용한 상황을 만들어줍니다.

동시성 문제에 대한 걱정 없이 코드를 작성할 수 있기 때문입니다.

그럼 nodejs도 싱글 스레드일까요?

> Node.js는 Chrome V8 JavaScript 엔진으로 빌드된 자바스크립트 런타임(환경)이다.

nodejs의 이벤트 루프는 싱글 스레드가 맞습니다.

nodejs는 libuv 라이브러리의 thread pool을 이용해서 통해 멀티스레드를 지원합니다.

그래서 우리는 nodejs에서 멀티스레드로 구현이 가능하다고 볼 수 있습니다.

(아래 링크를 참조해주세요.)  
[https://velog.io/@jaehyeon23/Javascript-%EC%99%80-%EC%8A%A4%EB%A0%88%EB%93%9CThread](https://velog.io/@jaehyeon23/Javascript-%EC%99%80-%EC%8A%A4%EB%A0%88%EB%93%9CThread){:target="_blank"}

<hr/>

# 그래서 멀티 스레드가 뭐가 좋다는 거임

nodejs에서 병렬처리를 위해 3가지 방법이 있습니다.

1. child_process
2. cluster
3. worker_threads

이번 포스팅에서는 싱글 스레드를 사용하는 경우와 worker threads를 활용한 멀티 스레드의 차이를 확인해보겠습니다.


<div style="display:flex;flex-direction: row;align-items: center;justify-content: space-around;border: 1px solid; padding: 5rem;">
    <img src="https://www.intel.co.kr/content/dam/www/central-libraries/us/en/images/2022-11/badge-ci7-10th.png.rendition.intel.web.64.64.png">
    <p>CPU: i7-10700K</p>
</div>

제 PC에서 벤치를 진행했고 i7-10700K는 8개의 코어 수를 갖고 있고 스레드 수는 16개입니다.  

왜 8개의 코어인데 스레드 수는 16개냐고 물으신다면 인텔의 [하이퍼 스레딩](https://www.intel.co.kr/content/www/kr/ko/gaming/resources/hyper-threading.html){:target="_blank"}를 참고 하세요.


```typescript
// worker.single.thread.ts

import path from 'path';
const workerJs = path.resolve(__dirname, './worker.thread.js');

const count = 10;

function singleThread() {
  console.time('single threads');
  // MARK: single threads
  for (let i = 1; i <= count; i++) {
    const worker = require(workerJs);
    worker(i);
  }
  console.timeEnd('single threads');
}

(() => {
  singleThread();
})();

```

```javascript
// worker.thread.js
// ts로 작업할 수 있지만 추후에 worker threads를 사용하기 위해 js로 작성합니다.

function printTime() {
  return new Date().getTime();
}

function computing(index) {
    console.log(printTime() + ' > got message : ' + index);
    let sum = 0;
    // MARK: 매우 무거운 코드
    for (let i = 0; i < 1e9; i++) {
        sum += i;
    }
    console.log(printTime() + ' > check message : ' + index, 'sum ->', sum);
    return index;
}

module.exports = computing;


```

worker.single.thread.ts에서 worker.thread.js를 10번 호출합니다.  
worker.thread.js는 1e9번의 연산을 수행합니다.  
결과를 보겠습니다.  

```shell

1687770145342 > got message : 1
1687770146197 > check message : 1 sum -> 499999999067109000
1687770146197 > got message : 2
1687770147049 > check message : 2 sum -> 499999999067109000
1687770147050 > got message : 3
1687770149016 > check message : 3 sum -> 499999999067109000
1687770149017 > got message : 4
1687770150984 > check message : 4 sum -> 499999999067109000
1687770150985 > got message : 5
1687770152946 > check message : 5 sum -> 499999999067109000
1687770152946 > got message : 6
1687770155007 > check message : 6 sum -> 499999999067109000
1687770155008 > got message : 7
1687770156970 > check message : 7 sum -> 499999999067109000
1687770156971 > got message : 8
1687770158932 > check message : 8 sum -> 499999999067109000
1687770158933 > got message : 9
1687770160887 > check message : 9 sum -> 499999999067109000
1687770160888 > got message : 10
1687770162912 > check message : 10 sum -> 499999999067109000
single threads: 17.573s


```

하나의 CPU로 17초가 걸렸습니다.

이제 "worker_threads"를 활용해서 멀티 스레드로 실행해보겠습니다.
약간의 코드 변경이 필요합니다.

```typescript
// worker.multi.thread.ts
import { Worker, isMainThread } from 'worker_threads';
import path from 'path';
const workerJs = path.resolve(__dirname, './worker.thread.js');

const count = 10;

function multiThread() {
  // console.log('isMainThread', isMainThread);
  let computed = 0;
  console.time('multi threads');
  for (let i = 1; i <= count; i++) {
    const worker = new Worker(workerJs);

    worker.on('message', (data) => {
      console.log(data);
      computed += 1;
      if (computed === count) {
        console.timeEnd('multi threads');
      }
    });
    worker.postMessage(i);
  }
}

(() => {
  multiThread();
})();

```

```javascript
// worker.thread.js
const { parentPort, workerData, isMainThread, threadId } = require('worker_threads');
function printTime() {
    return new Date().getTime();
}

function computing(index) {
    console.log(printTime() + ' > got message : ' + index);
    let sum = 0;
    // MARK: 매우 무거운 코드    
    for (let i = 0; i < 1e9; i++) {
        sum += i;
    }
    console.log(printTime() + ' > check message : ' + index, 'sum ->', sum);
    return index;
}

parentPort?.on('message', async (index) => {
    const value = computing(index);
    parentPort?.postMessage(value);
    parentPort?.close();
});

module.exports = computing;
```

실행결과
```shell
1687770382111 > got message : 10
1687770382117 > got message : 1
1687770382119 > got message : 5
1687770382120 > got message : 9
1687770382122 > got message : 6
1687770382124 > got message : 4
1687770382129 > got message : 8
1687770382133 > got message : 7
1687770382133 > got message : 2
1687770382136 > got message : 3

1687770383046 > check message : 5 sum -> 499999999067109000
1687770383049 > check message : 9 sum -> 499999999067109000
1687770383050 > check message : 10 sum -> 499999999067109000
1687770383070 > check message : 8 sum -> 499999999067109000
1687770383071 > check message : 6 sum -> 499999999067109000
1687770383080 > check message : 3 sum -> 499999999067109000
1687770383082 > check message : 7 sum -> 499999999067109000
1687770383083 > check message : 2 sum -> 499999999067109000
1687770383083 > check message : 4 sum -> 499999999067109000
1687770383090 > check message : 1 sum -> 499999999067109000

multi threads: 1.300s

```

worker_threads로 멀티 스레드를 활성화하여 병렬 처리된 결과 1.3초만에 결과를 냈습니다. 

<hr/>

# worker_threads?
nodejs에선 멀티스레드를 위해 "worker_threads"라는 모듈을 제공합니다.


> Workers (threads) are useful for performing CPU-intensive JavaScript operations.   
> They do not help much with I/O-intensive work. 
> The Node.js built-in asynchronous I/O operations are more efficient than Workers can be.
> [https://nodejs.org/api/worker_threads.html](https://nodejs.org/api/worker_threads.html){:target="_blank"}

nodejs 공식 문서에서 worker threads에 대해 위와 같이 설명하고 있습니다.

> 작업자(스레드)는 CPU를 많이 사용하는 JavaScript 작업을 수행하는 데 유용합니다.   
> I/O 집약적인 작업에는 그다지 도움이 되지 않습니다.  
> Node.js 내장 비동기 I/O 작업은 작업자보다 더 효율적입니다.  

이 내용을 꼭 염두하셔야 할 것이 대부분의 웹 개발에서는 

nodejs의 이벤트 루프가 싱글 스레드 논블로킹 모델로써 비동기 I/O작업으로 처리하기 때문입니다.

그렇기 때문에 CPU 연산이 많이 필요한 경우에만 worker_threads를 사용하는 것이 좋습니다.

위에서 언급했듯이 CPU 연산이 많이 필요한 경우는 결과에 아주 큰 차이를 보입니다.


<hr/> 
# 이외에..

[https://github.com/piscinajs/piscina](https://github.com/piscinajs/piscina){:target="_blank"}  
piscina는 멀티 스레드를 구현하기에 매우 편하고 인기 많은 worker pool입니다.  
추후에 기회가 된다면 사용해보고 싶네요.  

[https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/cpu-options-supported-instances-values.html](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/cpu-options-supported-instances-values.html){:target="_blank"}  
AWS의 EC2 인스턴스들의 코어와 코어당 기본 스레드는 위에서 확인하실 수 있습니다.  
클라우드로 배포할 경우 참고해서 사용하시면 좋을 것 같아 첨부합니다.

[https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API/Using_web_workers](https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API/Using_web_workers){:target="_blank"}    
또 다른 자바스크립트 런타임인 브라우저에서도 멀티 스레드를 구현할 수 있습니다.  
웹 워커는 웹 브라우저에서 멀티 스레드를 구현하기 위해 사용하는 API입니다.
콘솔창에서 [navigator.hardwareConcurrency](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/hardwareConcurrency){:target="_blank"}를 입력하면   
사용자 CPU의 논리 코어 수를 반환할 수 있습니다.

<hr/> 
# 정리하면서

이번 포스팅에선 아래와 같이 정리하겠습니다.

- nodejs의 싱글 스레드는 매우 훌륭하다.
- CPU를 많이 사용하는 작업에는 멀티 스레드를 사용하자.



nodejs로 개발한지 꽤 오래되었지만 웹 개발 이외의 것을 해본 경험은 손에 꼽는 것 같습니다.

시간이 지날수록 nodejs를 http 기반의 API개발의 범주를 넘는 경우가 많이 생기면서, 

nodejs의 다양한 영역에 대해 알아가는 것 같아 기쁘면서도 아직도 멀었다는 생각도 드네요.

도움이 된 글들
[https://blog.bitsrc.io/node-js-event-loop-and-multi-threading-e42e5fd16a77](https://blog.bitsrc.io/node-js-event-loop-and-multi-threading-e42e5fd16a77){:target="_blank"}

예제 코드
[https://github.com/pmirnc-dev/worker-threads-example](https://github.com/pmirnc-dev/worker-threads-example){:target="_blank"}
