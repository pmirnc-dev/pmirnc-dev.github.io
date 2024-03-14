---
layout: post
title:  "vitepress 사용기"
date:   2024-03-14 9:54:07 +0900
author: tkkim
categories: vue vitepress 
---
<hr/>


# vitepress 사용기

## SSG 란?

**웹 사이트의 모든 페이지를 미리 렌더링**하고 클라이언트의 요청에 따라 페이지를 제공하는 방법 ( 정적인 HTML 웹 사이트를 생성 )

데이터 베이스 또는 서버 측 프로세스가 거의 필요하지 않은 정적 HTML 기반 사이트 생성기능을 제공 합니다.

→ 그만큼 빠르다는 장점을 가지고 있습니다.

매 번 동적으로 생성할 필요가 없는 회사 소개 페이지에 적합합니다!


## 환경설정

1. vue 프로젝트 생성

   → vue의 구성요소 사용을 위해 생성해야 됩니다. ( 필자는 vue3 기준으로 생성 )


1. vite press 설치

   `cd [vue 프로젝트 생성 directory]`

   `npm add -D vitepress`


1. vite press 플젝 생성

   `npx vitepress init`

    ```jsx
    ┌  Welcome to VitePress!
    │
    ◇  Where should VitePress initialize the config?
    │  ./docs // config 생성위치
    │
    ◇  Site title:
    │  My Awesome Project // 사이트 제목
    │
    ◇  Site description:
    │  A VitePress Site // 사이트 설명
    │
    ◆  Theme: // 적용할 테마 선택
    │  ○ Default Theme (Out of the box, good-looking docs) // 기본 
    │  ● Default Theme + Customization // 기본 + css적용하기 쉽게 guide 제공 
    │  ○ Custom Theme // 기본없이 생으로 커스텀
    └
    ```


→ 플젝 생성 완료!


## 구조 설명

```jsx
.
├─ docs                # project root
│  ├─ .vitepress
│  │  ├─ theme
│  │  │  └─ index.js   # theme entry
│  │  └─ config.js     # config file
│  ├─ api-examples.md
│  ├─ markdown-examples.md
│  └─ [index.md](http://index.md/)
└─ package.json
```

`./doc` config 생성 디렉토리

내부에서 독립형 VitePress 사이트를 구축할 수 있습니다.

프로젝트의 다른 소스코드와 섞이지 않게 중첩도록 구성하는것을 권장드립니다.

`.vitepress`

구성 파일, 개발 서버 캐시, 빌드 출력 및 선택적 테마 사용자 지정 코드에 대한 예약된 위치

`theme` 디렉토리 안에있는 css를 만지면 전역 스타일 적용이 가능합니다.

`~~.md`

.html같은 경로로 컴파일되어 주소처리가 됩니다

[index.md](http://index.md) → index.html 로 주소가 컴파일 된다 라고 생각하시면 됩니다.

컴파일된 결과 사이트는`/` 루트 기준 방문이 가능합니다.

- e.g.

    ```jsx
    index.md                  -->  /index.html (accessible as /)
    prologue.md               -->  /prologue.html
    guide/index.md            -->  /guide/index.html (accessible as /guide/)
    guide/getting-started.md  -->  /guide/getting-started.html
    ```


## vite press 서버 랜더링

문서를 우선 build

→ 커맨드 라인에서 실행시킬 경우 Vite press가 최근에 작업했던 디렉토리를 프로젝트 루트로 사용합니다.

`npm run docs:build`

build 이후 로컬로 랜더링된 화면 미리 확인 할수있습니다.

`npm run docs:preview`


## vite press 홈페이지

[https://vitepress.dev/reference/default-theme-home-page](https://vitepress.dev/reference/default-theme-home-page) - 해당 사이트 기준으로 제작했습니다.

- init 사이트 예시
  ![Untitled](/assets/images/tkkim/20240314/image.png)


### Hero 섹션

홈페이지 상단에 위치해 있습니다.

```jsx
---
layout: home

hero:
  name: VitePress 
  text: Vite & Vue powered static site generator. 
  tagline: Lorem ipsum... 
  image: 
    src: /logo.png
    alt: VitePress
  actions: 
    - theme: brand 
      text: Get Started 
      link: /guide/what-is-vitepress 

---
```

- **Hero 구성**
  - **name**

    보통은 짧은 브랜드명이 들어갑니다.

  - **text**

    메인 텍스트 부분으로 h1 태그로 감싸져있다고 생각하시면 됩니다.

  - **tagline**

    'text' 하단에 배치되어있는 텍스트입니다.

  - **image**

    'text','tagline' 옆에 배치되어있습니다.

  - **actions**

    액션버튼으로 보통은 사이트 이동 시 사용됩니다.

    - **theme**

      적용할 테마로 'brand','alt'가 있는데 brand를 default로 두고있습니다.

    - **text**

      버튼의 라벨 부분입니다.

    - **link**

      버튼을 클릭했을 때 이동하고자하는 주소를 입력하면됩니다.


hero 브랜드명의 색상을 다음과 같은 변수를 이용해 적용 시킬 수 있습니다.

```jsx
:root {
--vp-home-hero-name-color: transparent; // 이름 색상 

// 그라데이션을 주고싶을때 다음과 같이 커스텀 할수도 있습니다.
--vp-home-hero-name-background: -webkit-linear-gradient(120deg, #bd34fe, #41d1ff); 
}
```

### Features 섹션

홈페이지 상단에 위치해 있습니다.

```jsx
---
layout: home

features:
  - icon: 🛠️
    title: Simple and minimal, always
    details: Lorem ipsum...
  - icon:
      src: /cool-feature-icon.svg
    title: Another cool feature
    details: Lorem ipsum...
  - icon:
      dark: /dark-feature-icon.svg
      light: /light-feature-icon.svg
    title: Another cool feature
    details: Lorem ipsum...
---
```

- **Features 구성**
  - **icon**

    src 필드를 추가하면 이미지도 삽입가능합니다.

    dark, light 필드를 추가하면 모드에 따라 이미지 변경도 가능합니다

  - **title**

    메인 텍스트 부분

  - details

    세부내역 기입부분

  - **link**

    링크 등록 시 feature 클릭하면 해당 주소로 이동가능합니다.

  - **actions**

    액션버튼으로 보통은 사이트 이동 시 사용됩니다.

    - **theme**

      적용할 테마로 'brand','alt'가 있는데 brand를 default로 두고있습니다.

    - **text**

      버튼의 라벨 부분입니다.

    - **link**

      버튼을 클릭했을 때 이동하고자하는 주소를 입력하면됩니다.


## vite press 내 vue 사용 

공식 사이트 설명을 인용하자면

VitePress에서는 각 Markdown 파일을 HTML로 컴파일 후 Vue Single-File Component로 처리한다고 합니다.

→ <script> 태그를 추가해서 동적 템플릿, Vue 컴포넌트 사용 그리고 임의 페이지 내 Vue 컴포넌트 로직을 포함한 Markdown 내부의 모든 Vue 기능을 사용할 수 있습니다.


## vitepress 사용하면서 알게된 점

- `**<template>` tag 로 감쌀 수 가 없습니다**

  root-level content가 마크다운이라 사용이 불가능합니다.


- 마크다운내에선 `<style scoped>` 사용은 피하는것이 좋습니다.

  `<style scoped>` tag 사용 시 요소마다 특별 속성을 추가해야하므로 페이지 size가 커지기 때문에

  페이지 내에 로컬범위 스타일링이 필요한 경우 `<style module>`을 지향합니다.

- img 파일은 public 디렉토리 내부에!

  소스 디렉토리 아래에 `public` 생성해 파일을 배치시킵니다.

  ex ) `txt`, favicons, and PWA icons , 이미지 파일 등…

  ⇒ 이미지 깨짐현상을 방지할 수 있습니다!


- vue 형식으로 작성 시 한줄 띄어쓰기 할 경우 오류가 발생하므로 코드 붙여서 작성해야합니다.




---

[참조]

[https://github.com/vuejs/vitepress/tree/main/src/client/theme-default/styles](https://github.com/vuejs/vitepress/tree/main/src/client/theme-default/styles) 

[https://vitepress.dev/reference/default-theme-config](https://vitepress.dev/reference/default-theme-config)

[https://vitepress.dev/reference/default-theme-nav](https://vitepress.dev/reference/default-theme-nav)

[https://learnvue.co/articles/write-docs-with-vitepress](https://learnvue.co/articles/write-docs-with-vitepress)

[https://vitepress.dev/guide/using-vue](https://vitepress.dev/guide/using-vue)

[https://vitepress.dev/guide/asset-handling](https://vitepress.dev/guide/asset-handling)

[https://www.elib.dev/vitepress-intro/](https://www.elib.dev/vitepress-intro/)