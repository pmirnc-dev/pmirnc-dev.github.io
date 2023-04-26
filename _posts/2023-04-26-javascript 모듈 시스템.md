---
layout: post
title:  "자바스크립트 모듈 시스템"
date:   2023-04-26 14:00:00 +0900
author: ihhwang
categories: javascript module cjs amd umd esm
---
<hr/>

# 자바스크립트 모듈 시스템

---

# 개요

초기 자바스크립트는 모듈을 가져오거나 내보내는 방법이 없어, 하나의 파일에 모든 기능들을 입력했습니다.

그러나 시스템이 커지면 코드의 양도 방대해 지기 때문에 가독성도 떨어지고 유지보수도 힘든 상황이 발생 하게 됩니다.

이에 따라 프로그램을 모듈로 개발하고 배포할 수 있게 CJS, AMD, UMD, ESM등 모듈 패턴이 등장하게 됐습니다.  

모듈 시스템은 다음과 같은 4가지 형태로 제공되고 있습니다.

- CJS(CommonJS): 서버 사이드에서 사용. 동기적으로 작동
- AMD(Asynchronous Module Definition): 클라이언트 사이드에서 주로 사용. 비동기 적으로 작동
- UMD(Universal Module Definition): CJS와 AMD 둘 다 사용하기 위함.
- ESM(ECMAScript Module): 언어 자체에서 모듈을 탑재함.

## 모듈이란?

전체 애플리케이션의 일부를 독립된 코드로 분리한 것 입니다.

이러한 코드를 각각의 파일로 나누어서 관리하는 것을 모듈화 라고 합니다.

## 모듈화

모듈화는 아래와 같이 세 부분으로 이루어 집니다.

- 스코프(scope): 모든 모듈은 자신만의 독립적인 실행 영역이 있어햐 합니다.
- 정의(Definition): 모듈 정의는 exports 객체를 이용합니다.
- 사용(Usage): 모듈 사용은 require 함수를 이용합니다.

## 모듈의 장점

- 기능별 코드를 재사용할 수 있다.
- 코드가 캡슐화 되기 때문에 무분별하게 코드가 변경되는 것을 방지할 수 있다.

## 모듈의 한계

프로그램이 커질수록 세분화된 파일이 많아지므로, 각 변수들의 스코프나 호출시 발생하는 네트워크 비용에 신경을 써야 하는 상황이 발생 합니다.

## 번들링

모듈의 문제점을 보완하기 위해 기능별로 모듈화된 파일을 다시 하나로 묶어줍니다.

번들링은 빌드 시 파일이 하나로 묶이게 됩니다. 대표적으로 webpack이 있습니다.


# CJS(CommonJS)

---

CommonJS는 Javascript 브라우저에서 뿐만 아니라, 서버사이드 애플리케이션이나 데스크톱 애플리케이션에서도 사용하려고 조직한 자발적 워킹 그룹 입니다.

CJS 방식으로 브라우저에서 사용하기 위해서는 browserify, webpack, rollup 등 모듈 번들러를 통한 모듈 번들링이 필요 합니다.

add.js

```jsx
// add.js
module.exports.add = (x, y) => x + y;
```

index.js

```jsx
// main.js
const { add } = require('./add'); // require 로 모듈을 가져올 수 있습니다.
const result = add(1, 2);
console.log('result: ', result); // result: 3
```

예제: https://replit.com/@hwangyear/cmstest

하지만 이런 방식은 브라우저에서 결정적인 단점이 존재합니다.

필요한 모듈을 모두 내려받을 때 까지 아무것도 할 수 없게 됩니다.

그렇기에 CJS가 꼭 서버사이드에 국한된 것이 아니지만, 

목적은 서버사이드에서 JS를 사용하는 것이기 때문에 서버사이드용으로 사용할 때에 장점이 많습니다.


# AMD(Asynchronous Module Definition)

---

AMD 그룹은 비동기 상황에서도 Javascript 모듈을 쓰기 위해 CJS에서 함께 논의하다 합의점을 이루지 못하고 독립한 그룹입니다.

AMD가 목표로 하는 것은 필요한 모듈을 네트워크를 이용해 내려받아야 하는 브라우저 환경에서도 모듈을 사용할 수 있도록 표준을 만드는 일 입니다.

대표적인 함수로 define()이 있습니다.

## define() 함수

define() 함수는 전역함수로 다음과 같이 정의합니다.

define(id?, dependencies?, factory);

첫번째 인수 id는 모듈을 식별하는데 사용하는 인수로, id가 없으면 로더가 요청하는 ```<script>``` 태그의 src 값을 기본 id로 설정합니다. 

만약 id를 명시한다면 파일의 절대 경로를 식별자로 지정해야 합니다.

예제1

```jsx
define('alpha', ['require', 'exports', 'beta'], function (require, exports, beta) {
	exports.verb = function() {
		// 넘겨 받는 인수를 사용해도 되고
		return beta.verb();
	
		// 또는 require()를 이용해 얻어온 모듈을 사용해도 된다.
		return require("beta").verb();
	}
}) ;
```

예제2

```jsx
define(["alpha"], function (alpha) {
	return {	
		verb: function() {		
			return alpha.verb() + 2;
		}	
	}
});
```

// 위 모듈이 [http://someurl.com/js/moduleBeta.js](http://someurl.com/js/moduleBeta.js로) 로 접근 가능 할 때

```jsx
require(["/js/moduleBeta.js"], function(moduleBeta) {
	// moduleBeta를 활용하는 코드 작성
});
```

예제3

의존성이 없는 모듈 정의.

```jsx
define({
	add: function(x, y) {
		return x + y;
	}
});
```

마지막으로 CJS 형태의 모듈을 래핑할 수도 있습니다.

```jsx
define(function (require, exports, module) {
	var a = require('a') 
    var b = require('b')
	exports.action = function (){};
});
```

## AMD의 장점

- 비동기 환경에서도 매우 잘 동작할 뿐만 아니라, 서버사이드에서도 동일한 코드로 동작합니다.
- CJS의 모듈 전송 포맷보다는 확실히 간단하고 명확합니다.
- AMD 명세는 define() 함수를 이용해 모듈을 구현하므로 전역변수 문제가 없습니다.
- 해당 모듈을 필요한 시점에 로드하는 Lazy-Load 기법을 응용할 수 있습니다.

구버전 브라우저에서의 성능은 많은 이득을 볼 수 있지만, 최신 브라우저에서는 성능이 비슷합니다.


# UMD(Universial Module Definition)

---

CJS 와 AMD 두 그룹으로 나누어지다보니 서로 호환이 되지 않는 문제가 발생합니다.

이것을 해결하기 위해 UMD가 나왔습니다. UMD는 디자인 패턴에 가까운 개념 입니다.

런타임을 확인하고 분기하는 방식 입니다.

UMD는 두 부분으로 구성 됩니다.

- 모듈 로더를 확인하는 즉시 실행함수(IIFE): 이 함수는 root(전역범위)와 factory(모듈을 선언하는 함수) 2개의 파라미터를 가집니다.
- 모듈을 생성하는 익명함수: 이 함수가 즉시 실행 함수의 2번쨰 파라미터로 전달됩니다.

```jsx
(function (root, factory) {

  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['exports','b'], factory);
  } else if (typeof exports === 'object' && typeof exports.nodeName !== 'string'){
    // CJS
    factory(exports, require('b'));
  } else {
    factory((root.commonJsStrict = {}), root.b);
  }
}(this, function (exports, b) {
  // use b in some fashion.
  exports.action = function () {};
}));
```

CJS 와 AMD 둘 다 사용할 수 있다는 장점이 있지만 코드가 매우 지저분해집니다.

# ESM(ECMASctipt Module)

ES6 이후 부터는 자바스크립트 모듈 시스템 기능이 추가되었습니다. - 모듈이 비동기로 로드 됩니다.

add.js

```jsx
// add.js
export function add(x, y) {
  return x + y
}
```

index.js

import 키워드로 모듈을 가져올 수 있다.

```jsx
// index.js
import { add } from './add.js';

const result = add(3, 4);
console.log('result: ', result);
```

## 특징: 자바스크립트 자체 모듈 시스템

ESM 은 ECMAScript에서 지원하는 JS 공식 모듈 시스템 입니다.

아직 브라우저에서 import, export를 지원하지 않아 번들러를 함께 사용해야 합니다.

```jsx
<script type="module" src="index.mjs">
<script defer src="./webcam.js" type="module"></script> --> 확장자가 js 여도 작동은 합니다.
```

다음과 같이 script 태그에 type=”module”을 선언하면 js 파일은 모듈로 동작합니다.

이때, 모듈이라는 것을 명확히 알기 위해서 mjs 확장자를 사용하도록 권장합니다.

아래는 모듈 이라는 것을 명시해주지 않고 실행하면 다음과 같이 경고가 나타납니다.

![0426_1.png](/assets/images/ihhwang/0426_1.png)

.mjs 확장자 대신 package.json에 타입을 추가해 줄 수도 있습니다.

```json
{
  "name": "nodejs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  **"type": "module",**
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@types/node": "^18.0.6",
    "node-fetch": "^3.2.6"
  }
}
```

# CJS와 ESM을 쓸 때 주의점.

---

- ```CJS``` 에서는 ```import``` 를 사용할 수 없고
- 반대로 ```ESM``` 은 ```require``` 를 사용할 수 없다.


# CJS, ESM 같이 사용하기(By Toss)

---

- CJS 에서 비동기 적으로 dynamic import 를 사용하여 ESM을 사용할 수 있지만 동기적으로 require 를 사용하는 것과 비교했을 때 더 번거롭다.

토스 팀은 이 호환되지 않는 두개의 모듈 시스템을 같이 사용하는데 그 이유는

Server-side Rendering을 적극적으로 사용하고 있기 때문에, Node.js의 CJS를 지원하는 것이 중요하고 합니다.

그리고 모듈 시스템의 지원은 브라우저 환경에서의 퍼포먼스와도 관련이 있습니다.

브라우저에서 페이지 랜더링을 빠르게 하는 것이 중요한데, 

이 때 Javascript는 로딩되어 실행되는 동안 페이지 랜더링을 중단시키는 리소스들 중 하나 입니다.

따라서 Javascript 번들 사이즈를 줄여 랜더링을 빠르게 하는 것이 중요한데, 사이즈를 줄이는 것을 Tree-shaking 이라고 합니다.

Tree-shaking은 필요하지 않은 코드와 사용되지 않은 코드를 삭제하여 Javascript 번들의 크기를 가볍게 만드는 것을 말합니다.


<strong>*이 때, CJS 는 Tree-shaking이 어렵고, ESM은 쉽게 가능합니다.*</strong>


package.json

```json
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "type": "module", // ESM 일때만 추가.
  "exports": {
    ".": {
      "require": "./cjs/index.cjs",
      "import": "./esm/index.mjs"
    }
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node main.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@types/node": "^18.0.6",
    "node-fetch": "^3.2.6",
    "typescript": "^5.0.4"
  }
}
```

main.js

```jsx
// CJS
// const { add } = require('test');
// const result = add(1, 2);

// ESM - package.json 에 "type": "module",  추가
import { add } from 'test';
const result = add(3, 4);

console.log('result: ', result);
```

cjs/index.cjs

```jsx
module.exports.add = (x, y) => x + y;
```

mjs/index.mjs

```jsx
export function add(x, y) {
  return x + y
}
```

[https://replit.com/@hwangyear/dualcmsesmtest](https://replit.com/@hwangyear/dualcmsesmtest)

# 감사합니다!
![Untitled](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99C9493C5F32979520)

---

# 참조

[https://beomy.github.io/tech/javascript/cjs-amd-umd-esm/](https://beomy.github.io/tech/javascript/cjs-amd-umd-esm/)

[https://toss.tech/article/commonjs-esm-exports-field](https://toss.tech/article/commonjs-esm-exports-field)

[https://defineall.tistory.com/916#toc41](https://defineall.tistory.com/916#toc41)

[https://darrengwon.tistory.com/1424](https://darrengwon.tistory.com/1424)

[https://roseline.oopy.io/dev/translation-why-cjs-and-esm-cannot-get-along](https://roseline.oopy.io/dev/translation-why-cjs-and-esm-cannot-get-along)

[https://blog.naver.com/pjt3591oo/222574553721](https://blog.naver.com/pjt3591oo/222574553721)

[https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)

[https://blog.naver.com/pjt3591oo/222834625061](https://blog.naver.com/pjt3591oo/222834625061)

[https://www.typescriptlang.org/docs/handbook/esm-node.html](https://www.typescriptlang.org/docs/handbook/esm-node.html)

[https://nodejs.org/api/packages.html#dual-commonjses-module-packages](https://nodejs.org/api/packages.html#dual-commonjses-module-packages)