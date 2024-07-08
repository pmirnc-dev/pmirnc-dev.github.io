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

<br />

## 모노레포란?

그렇다면 모노레포는 무엇일까?        

모노레포란 2 개 이상의 프로젝트 코드를 하나의 레포지토리에서 관리하는 방법을 의미합니다.     

저희 프로젝트의 대부분 레포지토리 마다 각각 다른 프로젝트를 담당하고 있는 멀티레포로 구성되어있는데 이를 하나의 레포지토리에서 관리할 수 있습니다.

![repo-sample](/assets/images/dhjeon/240509/sample.png)

<br />

## Tools

모노레포 구축을 도와주는 여러 툴이 존재합니다.

![2021년 기준 모노레포 툴 선호도 이미지](/assets/images/dhjeon/240509/monorepo-tool.png)

[이미지 출처 - https://2021.stateofjs.com/ko-KR/libraries/monorepo-tools/](https://2021.stateofjs.com/ko-KR/libraries/monorepo-tools/){:target="_blank"}     

여러 레퍼런스를 찾아보다가 turbo의 경우 기본적으로 타입스크립트를 사용한다하여 이번 프로젝트에서 적용시켜보았습니다.

![주요 모노레포 툴 특징](/assets/images/dhjeon/240509/tool.png)

[https://turbo.build/repo/docs - Turbo 공식 문서](https://turbo.build/repo/docs){:target="_blank"}      

[https://erwinousy.medium.com/turborepo%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%84%EB%9E%B5%ED%95%9C-%EC%86%8C%EA%B0%9C-adf78ddb4787](https://erwinousy.medium.com/turborepo%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%84%EB%9E%B5%ED%95%9C-%EC%86%8C%EA%B0%9C-adf78ddb4787){:target="_blank"}

<br />

## 프로젝트

### turbo 환경 만들기

```bash
# cli 를 사용하기 위해 설치하는 것이지 꼭 설치하지 않아도 됩니다.!
npm install turbo --global

npx create-turbo@latest
```

위 명령어를 통하여 turbo 환경을 구축할 수 있습니다. 하지만 저는 이미 구성되어있는 프로젝트 내에서 진행해야하므로 필요한 부분만 설치하면서 구조를 변경해 나갔습니다.

![프로젝트 구조](/assets/images/dhjeon/240509/project.png)

### apps

주요 프로젝트가 들어가는 위치 입니다. 프론트 프로젝트와 백엔드 프로젝트가 주요 프로젝트 이기에 apps 안에 위치합니다.

### packages

apps 안에 있는 프로젝트에 함께 사용될 패키지 모음입니다.

turbo의 경우 패키지 기반으로 관리 되어집니다. 해당 디렉토리 안에서 생성된 패키지를 apps 내의 프로젝트에서 공유하여 사용할 수 있습니다.

현재 비즈챗 프로젝트에서는 enum type, interface, utility function 등 프론트와 백엔드 간 서로 공유할 수 있는 파일을 패키지화하여 사용하고 있습니다.

공유할 패키지를 생성하고 각각의 프로젝트에서 dependency만 추가하면 사용할 수 있습니다.

##### backend package.json

![backend package.json](/assets/images/dhjeon/240509/backend.png)

##### frontend package.json

![frontend package.json](/assets/images/dhjeon/240509/frontend.png)

[참고 비즈챗 깃허브 이슈 - https://github.com/orgs/pmirnc-dev/projects/32?pane=issue&itemId=59997558](https://github.com/orgs/pmirnc-dev/projects/32?pane=issue&itemId=59997558){:target="_blank"}

<br />

### 프로젝트 적용

위에서 언급한대로 인터페이스를 프론트와 백엔드 간 공유하여 타입을 일치 시킵니다.

첫 번째로 생각한 부분은 Api 호출이 이루어지는 곳입니다.

Request와 Response가 이루어지는 부분은 프론트와 백엔드간 타입이 같아야함으로 인터페이스를 공유할 수 있습니다.

packages 디렉토리 안에 api-interface라는 패키지를 생성하고 이 안에 필요한 인터페이스를 작성해 줍니다.

```typescript
// packages > api-interface 에 생성한 인터페이스 입니다. 
import { CampaignBase } from "../campaign";
import { Project } from "./base";
import { Reward } from "./rewards";

/** 캠페인 옵션 중 프로젝트 등록 관련한 값만 추출*/
export interface CreateProjectOptions
  extends Pick<CampaignBase, "customerName" | "sndGoalCnt"> {}

/**
 * @description 프로젝트 등록
 */
export interface CreateProjectState
  extends Omit<Project, "status">,
    CreateProjectOptions {
  // 리워드 관련
  rewards: Reward[];
}
```

생성한 인터페이스는 저희가 만든 패키지 명으로 import 받아 사용할 수 있습니다.

```typescript
import { CreateProjectState, Reward, RewardType } from '@bizchat/api-interface';
```

패키지 명은 각 패키지의 package.json에 명시되어있는 name 입니다.

```json
// api-interface package.json
{
  "name": "@bizchat/api-interface",
  "version": "0.0.34",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": "./dist/index.js"
  },
  "dependencies": {
    "typescript": "latest"
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "prod": "tsc"
  }
}
```

먼저 프론트에서 해당 인터페이스를 불러와 사용합니다.

```typescript
import { CreateProjectState, Reward, RewardType } from "@bizchat/api-interface";

export const SampleComponent = (): ReactElement => {
  const [projectState, setProjectState] = useState<CreateProjectState>(
    RegistrationFormState,
  );
}
```

프론트의 경우 interface로 타입을 체크하기 때문에 interface를 불러와 바로 적용시켜주었지만 NestJS에서는 interface보다는 dto class를 만들어서 타입을 체크하는 것을 권장하고 있습니다.        

그렇기 때문에 dto class를 만들어 주어야하는데 interface를 implements 받아 재정의하여 dto class를 생성해줍니다.

```typescript
import { CreateProjectState, Reward, RewardType } from '@bizchat/api-interface';

export class RewardDto implements Reward {
  @IsEnum(RewardType)
  type: RewardType;
  @IsNumber()
  completedReward: number;
  @IsNumber()
  eliminateReward: number;
}

export class CreateProjectDto implements CreateProjectState {
  @IsString()
  @ApiProperty({
    title: '프로젝트 명',
    type: String,
  })
  projectTitle: string;
  
  // 이하 생략
}
```

만약 공용 인터페이스가 수정됐을 경우 해당 인터페이스를 사용한 곳에서 타입이 다른 경우가 있다면 콘솔창에 에러가 발생하여 타입이 맞지 않은 곳을 쉽게 확인할 수 있습니다.


## 주의할 점

### 패키지 수정

수정이 필요한 상황이라면 프론트와 백엔드 둘 다 영향을 주기 때문에 유의 해야합니다.

단순히 한 곳을 위해 수정을 해야한다면 공유 패키지를 사용하는 것이 적합하지 않을 수 있습니다.

### Compiler Option - module

모노레포 레퍼런스를 확인해보면 대부분의 레퍼런스가 프론트 관련한 프로젝트로 되어있어 저희 프로젝트와 환경이 달랐습니다.

비즈챗의 경우 프론트는 React, 백엔드는 NestJS로 구성되어있는데 각각의 컴파일 되는 모듈이 다릅니다.

NestJS의 경우 CommonJS 기반으로 모듈이 구성되고, React의 경우 ESNext 기반으로 모듈이 내보내집니다.

[CommonJS vs Esm](https://velog.io/@tenacious_mzzz/Node.JS%EC%97%90%EC%84%9C-CommonJS-vs-ES-modules){:target="_blank"}

서로 간 공유를 하기 위해서는 이 환경을 일치시켜줘야합니다.

그래서 프론트에서 컴파일 시 CommonJS 기반으로 컴파일 되어지도록 플러그인의 도움을 받았습니다.

```
npm install @rollup/plugin-commonjs --save-dev
```

[https://www.npmjs.com/package/@rollup/plugin-commonjs](https://www.npmjs.com/package/@rollup/plugin-commonjs){:target="_blank"}

설치한 플러그인을 vite.config.ts에 등록해 줍니다.

```typescript
import { defineConfig, loadEnv } from "vite";
import react from "@vitejs/plugin-react-swc";
import tsconfigPaths from "vite-tsconfig-paths";
import commonjs from "@rollup/plugin-commonjs"; // 플러그인 사용

// https://vitejs.dev/config/
export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), "");
  return {
    define: {
      "process.env": env,
    },
    plugins: [react(), tsconfigPaths()],
    build: {
      rollupOptions: {
        plugins: [commonjs()],
      },
    },
    optimizeDeps: {
      include: ["@bizchat/api-interface/**/*", "@bizchat/utility/**/*"],
    },
  };
});
```

플러그인 등록과 함께 사용할 패키지도 optimizeDeps에 포함시키면 외부에 위치한(packages)에 접근하여 사용할 수 있습니다.

### Vite Option

프론트의 경우 Vite 로 구성되어있는데 패키지 인식이 잘 되지 않는 문제가 있었습니다.       

원인은 Vite에 잡혀있는 캐시 문제였는데 Vite 프로젝트가 직접적으로 변경이 됐을 경우 잡혀있는 캐시를 지우고 새로운 환경으로 캐싱을 한다고하는데 위의 경우 외부 패키지에 변경이 이루어진 것이라 캐시 초기화가 되지 않을 것으로 보여집니다.

dev 환경에서 프로젝트 실행 시 무조건 캐시 초기화 되도록 --force 옵션 추가하여 개발 모드 상태에서 프로젝트가 재 실행 될 때 새로운 환경으로 실행되도록 할 수 있습니다.

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

---

## 레퍼런스

[NestJS Monorepo 관련 공식 문서](https://docs.nestjs.com/cli/monorepo){:target="_blank"}

[https://github.com/belgattitude/nextjs-monorepo-example](https://github.com/belgattitude/nextjs-monorepo-example){:target="_blank"}

[https://beomy.github.io/tech/etc/monorepo-concept/](https://beomy.github.io/tech/etc/monorepo-concept/){:target="_blank"}

[https://blog.mathpresso.com/%ED%8C%80%EC%9B%8C%ED%81%AC-%ED%96%A5%EC%83%81%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%AA%A8%EB%85%B8%EB%A0%88%ED%8F%AC-monorepo-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%B6%95-3ae1b0112f1b](https://blog.mathpresso.com/%ED%8C%80%EC%9B%8C%ED%81%AC-%ED%96%A5%EC%83%81%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%AA%A8%EB%85%B8%EB%A0%88%ED%8F%AC-monorepo-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%B6%95-3ae1b0112f1b){:target="_blank"}


