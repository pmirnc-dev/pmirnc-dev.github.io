---
layout: post
title: "Next.js 의 app router 와 서버 컴포넌트"
date: 2024-09-22 15:00:00 +0900
author: ejsin
categories: next next.js ssr
published: true
---

<hr>
처음 Create Next App 으로 Next.js 를 시작할 때
App Router 와 Page Router 방식 중 선택하라고 합니다.

이번에 App Router 방식과 Page Router 방식 중 뭘 선택해야 할지 알아보겠습니다.

## App Router vs Page Router

일단 먼저 결론부터 말씀드리자면...

**App Router** 방식으로 하는게 좋습니다.

이미 Vercel(next 만든 곳)에서 앱 라우터 방식을 추천하고 있으며
버전이 올라가면서 페이지 라우터에서 업그레이드 된게 앱 라우터라고 보시면 됩니다.

## 무엇이 바뀌었나?

- 서버 컴포넌트 사용
- 파일 및 폴더 구조의 변화
- 데이터 패칭의 간소화
- 퍼포먼스 개선
- SEO 및 메타데이터 처리 개선
- 터보팩 지원

등등 ...

많은 변화가 있었지만 가장 중요한 건 

**서버 컴포넌트** 를 사용할 수 있게된 겁니다.

## React Server Component

서버 컴포넌트는 리액트 18버전에서 처음 등장했습니다.

원래 리액트는 CSR(클라이언트 사이드 렌더링) 방식으로만 사용할 수 있었는데 SSR(서버 사이드 렌더링)도 사용할 수 있도록 하기 위해 서버 컴포넌트 가 등장했습니다.

리액트 자체에서 위 서버 컴포넌트를 사용해서 SSR 을 구현할 수 있지만...
view 만을 담당하는 리액트에서 꼭 필요하지 않다면 비추합니다.

그래서 Next 에서 서버 컴포넌트를 쉽게 사용할 수 있도록 앱 라우터 방식이 생겼습니다.

Next 에서 내가 원하는 라우트의 디렉토리를 만들고 page.tsx 를 생성하면
이게 서버 컴포넌트가 됩니다.

```tsx
// /home/page.tsx
import MainContent from './MainContent';

export default function HomePage() {
  return (
    <div>
      <h1>Title</h1>
      <h2>Sub Text</h2>

      {/* 클라이언트 컴포넌트 */}
      <MainContent /> 
    </div>
  );
}
```

클라이언트 컴포넌트를 사용하고 싶다면 그냥 "use client" 를 사용하면 됩니다.

```tsx
"use client"

export default function MainContent() {
    return <div>client rendering</div>
}
```

## App Router !!

***서버 컴포넌트*** 의 사용으로 인해 앱 라우터 방식이 페이지 라우터 방식보다 훨씬

<u>리액트스럽게(더욱 선언적으로) 코드를 작성할 수 있습니다.</u>

![components](/assets/images/ejsin/page_vs_app/components.png)

이거 말고도 중첩 레이아웃, 자동 에러 및 로딩 처리 등 이전 버전들에서

불편했던 또는 필요했던 기능들이 추가 되었으니 더욱 편리한 코드 작업을 할 수 있습니다.

<hr>

앱 라우터는 편리함을 넘어서 실질적으로 터보팩을 사용하는 등 성능 및 퍼포먼스 향상이 되었습니다. 

하지만 페이지 라우터를 사용해도 서버 컴포넌트 같은 기능을 사용하지 못할 뿐이지
서버 사이드 렌더링 기능이든 정적 서버 생성(SSG)이든 모두 구현 가능하기 때문에 <u>이미 페이지 라우터를 사용하고 있다면 굳이 바꿀 필요가 없다</u>고 생각합니다.

next 는 리액트 처럼 공식 문서가 정말 정리가 잘 되어 있습니다.

혹시나 next 작업을 해야할 때 읽어보시면 큰 도움이 됩니다.

[출처 및 참고]

- https://nextjs.org/docs
- https://tech.kakaopay.com/post/react-server-components/
- https://dionarodrigues.dev/blog/react-server-components-overview