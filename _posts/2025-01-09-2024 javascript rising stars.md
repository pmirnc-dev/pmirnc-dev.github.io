---
layout: post
title:  "2024 JavaScript Rising Stars"
date:   2025-01-09 11:24:00 +0900
author: ihhwang
categories: javascript
published: true
---
<hr/>

안녕하세요.

스터디 주제를 못 찾아서 최근에 본 것을 공유해보고자 합니다. 

# 2024 JavaScript Rising Stars

2016년도 부터 현재까지 이어진 한 해 동안의 javascript 트렌드 순위를 볼 수 있습니다.

## Front-end Frameworks

### 1. htmx

이거 뭔지 처음 알게 됐는데 HTML에서 직접 최신 브라우저 기능에 액세스할 수 있게 해주는 라이브러리입니다.

간단한 화면을 만들 때 좋을 것으로 보입니다.

Link => [https://htmx.org/](https://htmx.org/)

### 2. React

react가 2위인게 놀랐으나 다른 프레임워크에 비해 여전히 압도적으로 사용량이 높습니다.

vue.js 와 svelte는 이제 차이가 없어보이네요.

## Back-end/Full-stack

### Next.js

next.js로 백엔드를 할 수 있다는건 알았는데 nest.js 의 약 2배정도의 사용량을 보입니다.

근데 next.js는 front 로 많이 쓰이니 어쩔 수 없다고 생각 합니다.

이 외에 Hono, Astro 라는 것도 보입니다.

Hono Link => [https://hono.dev/](https://hono.dev/)

Astro Link => [https://astro.build/](https://astro.build/)

Chat GPT 에게 물어본 특징 및 사용사례에 대한 추천 입니다.


|특징|Next.js|Nest.js|Hono|Astro|
|---|---|---|---|---|
|**역할**|풀스택 프론트엔드 및 SSR/SSG 프레임워크|백엔드 전용 프레임워크|초경량 웹 프레임워크|정적 사이트 생성(SSG) 및 컴포넌트 중심 개발|
|**언어**|React 기반 (JavaScript/TypeScript)|TypeScript|JavaScript/TypeScript|JavaScript/TypeScript|
|**성능 초점**|다목적, 유연성|모듈화와 확장성|속도와 경량성|빌드 속도와 정적 콘텐츠 최적화|
|**렌더링 방식**|SSR, SSG, CSR|API 서버|CSR|SSG, CSR|
|**커뮤니티 크기**|대형|대형|소형|중형|
|**철학**|풀스택 솔루션 제공|백엔드 로직 처리|최소한의 코드로 서버 개발|정적 및 하이브리드 사이트 생성 최적화|


|필요 조건|추천 프레임워크|이유|
|---|---|---|
|**다양한 렌더링 방식과 SEO**|Next.js|SSR, SSG, ISR 등 다양한 렌더링 방식 지원.|
|**대규모 백엔드 시스템 개발**|Nest.js|모듈화 및 확장성이 뛰어나고 DI를 지원.|
|**초경량 서버나 서버리스 환경**|Hono|속도와 경량성 중심, 서버리스 환경에 최적화.|
|**정적 콘텐츠 중심의 사이트**|Astro|빠른 빌드 속도와 JavaScript 최소화.|
|**프론트엔드와 백엔드 모두 포함된 프로젝트**|Next.js + Nest.js|프론트엔드는 Next.js, 백엔드는 Nest.js로 분리.|
|**작고 간단한 API 서버**|Hono|경량 코딩으로 빠른 개발 가능.|


## Tools

### Biome

처음 들어보는 건데 Prettier 에 비해 97% 빠르다고 강조 합니다.

Link => [https://biomejs.dev/](https://biomejs.dev/)

실제로 사용해 본 블로그를 보니 Prettier 에 비해 설정도 쉽고 속도는 체감으로 느끼진 못했지만 실제로 확인해보니 빠르다고 하네요

Link => [Biome: 차세대 JS Linter와 Formatter](https://dabletech.oopy.io/5a038d59-1cee-47f2-aaff-68e00115a399)

### Bun & Vite

bun도 들어본지 얼마 안된 것 같은데 많이 사용중 인것 같습니다.

vite는 bun 보다 1년 먼저 나오긴 했는데 근소한 차이로 3위를 했습니다.

vite를 웹 개발에 bun은 런타임, 빌드, 패키지 매니저로 사용 가능하니

두 개를 사용하여 실험적인 프로젝트를 만들면 재밌을 것 같습니다.

Link => [Build a frontend using Vite and Bun](https://bun.sh/guides/ecosystem/vite)

## Desktop

### Tauri

tauri 는 몇 번 들어보긴 했는데 electron에 비해 압도적으로 사용량이 많을 줄은 몰랐습니다.

이번 기회에 tauri를 접하면서 Rust도 공부를 해봐야겠다는 생각이 들었습니다.

# 결론

웹 개발의 생태계는 정말 빠르게 변하는 것 같습니다.

눈 깜박 하면 새로운게 나오고 적응 됐다 싶으면 점점 역사의 뒤편으로 사리지는 것 같습니다.

나루토의 그림자 분신술 처럼 몸을 여러개로 만들 수 있지 않는 한 모두 다 접해보기는 어려울 것 같고

선택과 집중을 잘 해야 할 것 같습니다.

---
# 참조

* [https://risingstars.js.org/2024/en](https://risingstars.js.org/2024/en)
