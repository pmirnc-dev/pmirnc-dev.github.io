---
layout: post
title:  "Turborepo를 활용하여 모노레포 환경 구축하기 (React,NestJS)"
date:   2024-05-09 09:00:00 +0900
author: dhjeon
categories: react nestjs
published: true
---
<hr>

이번 비즈챗 프로젝트를 진행하면서 적용시켜본 모노레포 구조에 대해서 소개해드릴까 합니다.        

모노레포 소개에 앞서서 왜 이런 프로젝트 환경을 구축했는지가 먼저 나와야할 것 같아요.

저희 프로젝트는 자바스크립트 기반으로 되어있고 하나의 레포지토리에서 프론트와 백엔드가 함께 관리되고 있습니다.       

그러다 보니 프론트와 백엔드간의 겹치는 인터페이스가 존재하여 만약 수정이 발생된다면 프론트와 백엔드 각각 수정을 해줘야 했으며 만약 이 부분을 놓쳤을 경우 프론트와 백엔드간의 타입이 서로 다른 경우가 발생하게 됩니다.       

그래서 하나의 인터페이스로 서로 공유한다면 이런 문제가 해소되지 않을까에서 시작되어 찾아보게 되었습니다.

## 모노레포란?

그렇다면 모노레포는 무엇일까?        

모노레포란 2 개 이상의 프로젝트 코드를 하나의 레포지토리에서 관리하는 방법을 의미합니다.     

저희 프로젝트의 대부분 레포지토리 마다 각각 다른 프로젝트를 담당하고 있는 모놀리식으로 구성되어있는데요.      

레포지토리 마다 각각의 프로젝트라고 한다면 다른 레포지토리와 비슷하게 프론트와 백엔드를 함께 관리하는 비즈챗 프로젝트도 모노리식과 동일한게 아닌가 라는 생각이 들 수 있습니다. 

프론트와 백엔드의 환경이 분리되어있기 때문에 각각의 프로젝트로 생각하면 될 것 같아요. 저희 대부분의 프로젝트도 하나의 레포지토리 안에 각각 프론트와 백엔드가 함께 있지만 서로간의 환경이 분리되어있어 따로 관리되었다면, 모노레포 구조 안에서는 프론트와 백엔드를 함께 관리할 수 있습니다.

![repo-sample](/assets/images/dhjeon/240509/sample.png)

## Tools

모노레포 구축을 도와주는 여러 툴이 존재합니다. 

![주요 모노레포 툴 특징](/assets/images/dhjeon/240509/tool.png)

![2021년 기준 모노레포 툴 선호도 이미지](/assets/images/dhjeon/240509/monorepo-tool.png)

[이미지 출처 - https://2021.stateofjs.com/ko-KR/libraries/monorepo-tools/](https://2021.stateofjs.com/ko-KR/libraries/monorepo-tools/){:target="_blank"}     

여러 레퍼런스를 찾아보다가 turbo의 경우 기본적으로 타입스크립트를 사용한다하여 이번 프로젝트에서 적용시켜보았습니다.

[https://turbo.build/repo/docs - Trubo 공식 문서](https://turbo.build/repo/docs){:target="_blank"}      

[Turbo 소개 링크](https://erwinousy.medium.com/turborepo%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%84%EB%9E%B5%ED%95%9C-%EC%86%8C%EA%B0%9C-adf78ddb4787){:target="_blank"}

## 프로젝트 구조

turbo 환경 만들기

```bash
# cli 를 사용하기 위해 설치하는 것이지 꼭 설치하지 않아도 됩니다.!
npm install turbo --global

npx create-turbo@latest
```

위 명령어를 통하여 turbo 환경을 구축할 수 있습니다. 하지만 저는 한땀한땀.. 설치하면서 진행했습니다. 구축된 파일 구조는 아래 이미지와 같습니다.

![프로젝트 구조](/assets/images/dhjeon/240509/project.png)

### apps

주요 프로젝트가 들어가는 위치 입니다. 프론트 프로젝트와 백엔드 프로젝트가 주요 프로젝트 이기에 apps 안에 위치합니다.

### packages

apps 안에 있는 프로젝트에 함께 사용될 패키지 모음입니다.

turbo의 경우 패키지 기반으로 관리되어집니다. 해당 디렉토리 안에서 생성된 패키지를 apps 내의 프로젝트에서 공유하여 사용할 수 있습니다.

현재 비즈챗 프로젝트에서는 enum type, interface, utility function 등 프론트와 백엔드 간 서로 공유할 수 있는 파일을 패키지화하여 사용하고 있습니다.

공유할 패키지를 생성하고 각각의 프로젝트에서 dependency만 추가하면 사용할 수 있습니다.

##### backend package.json

![backend package.json](/assets/images/dhjeon/240509/backend.png)

##### frontend package.json

![frontend package.json](/assets/images/dhjeon/240509/frontend.png)

[참고 비즈챗 깃허브 이슈 - https://github.com/orgs/pmirnc-dev/projects/32?pane=issue&itemId=59997558](https://github.com/orgs/pmirnc-dev/projects/32?pane=issue&itemId=59997558){:target="_blank"}

## 주의할 점

### Compile Option

모노레포 레퍼런스를 확인해보면 대부분의 레퍼런스가 프론트 관련한 프로젝트로 되어있어 저희 프로젝트와 환경이 달랐습니다.

비즈챗의 경우 프론트는 React, 백엔드는 NestJS로 구성되어있는데 각각의 컴파일 되는 모듈이 다릅니다.

NestJS의 경우 컴파일 시 CommonJS 기반으로 되고, React의 경우 ESNext 기반으로 컴파일 되어집니다.

서로 간 공유를 하기 위해서는 이 환경을 일치시켜줘야합니다.

그래서 프론트에서 컴파일 시 CommonJS 기반으로 컴파일 되어지도록 플러그인의 도움을 받았습니다.

```
npm install @rollup/plugin-commonjs --save-dev
```

[https://www.npmjs.com/package/@rollup/plugin-commonjs](https://www.npmjs.com/package/@rollup/plugin-commonjs){:target="_blank"}

### Vite Option

프론트의 경우 Vite 로 구성되어있는데 패키지 인식이 잘 되지 않는 문제가 있었습니다.       

원인은 Vite에 잡혀있는 캐싱 문제였는데 Vite 프로젝트가 직접적으로 변경이 됐을 경우 잡혀있는 캐싱을 지우고 새로운 환경으로 캐싱을 한다고하는데 위의 경우 외부 패키지에 변경이 이루어진 것이라 캐싱 초기화가 되지 않을 것으로 보여집니다.

dev 환경에서 프로젝트 실행 시 무조건 캐시 초기화 되도록 --force 옵션 추가

```json
{
  "scripts": {
    "dev": "vite --force"
  }
}
```

[관련 내용 Vite 공식 문서 - https://ko.vitejs.dev/guide/dep-pre-bundling](https://ko.vitejs.dev/guide/dep-pre-bundling){:target="_blank"}

### arm 기반 mac 으로 실행 시 추가 플러그인 필요 

m1 칩을 사용한 맥북에서 프로젝트가 실행이 되지 않아 관련 내용을 찾아보다가 해당 플러그인을 추가로 다운로드해야 실행이 가능했습니다.

```bash
npm i @rollup/rollup-darwin-arm64
```

[https://www.npmjs.com/package/@rollup/rollup-darwin-arm64](https://www.npmjs.com/package/@rollup/rollup-darwin-arm64){:target="_blank"}

## 레퍼런스

[https://beomy.github.io/tech/etc/monorepo-concept/](https://beomy.github.io/tech/etc/monorepo-concept/){:target="_blank"}

[https://blog.mathpresso.com/%ED%8C%80%EC%9B%8C%ED%81%AC-%ED%96%A5%EC%83%81%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%AA%A8%EB%85%B8%EB%A0%88%ED%8F%AC-monorepo-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%B6%95-3ae1b0112f1b](https://blog.mathpresso.com/%ED%8C%80%EC%9B%8C%ED%81%AC-%ED%96%A5%EC%83%81%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%AA%A8%EB%85%B8%EB%A0%88%ED%8F%AC-monorepo-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%B6%95-3ae1b0112f1b){:target="_blank"}


