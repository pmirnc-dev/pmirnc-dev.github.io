---
layout: post
title:  "Webstorm typescript plugin 소개"
date:   2023-08-31 17:10:00 +0900
author: hsyoo
categories: webstorm typescript
---
<hr/>

webstorm에서 .ts 파일을 실행할 때 간단한 방법을 소개합니다.

바쁜 현대인답게 거두절미하고 본론만 적겠습니다.

### 타입스크립트 셋팅

```shell
npm i -D typescript
# 타입 스크립트 설치

npm i -D ts-node-dev
# ts-node-dev 설치

npx tsc --init
# tsconfig.json 생성
```


###  plugin 설치

["Run configuration for TypeScript"](https://plugins.jetbrains.com/plugin/10841-run-configuration-for-typescript){:target="_blank"}

<img src="/assets/images/hsyoo/20230831-01.png" />  

이후 재시작

### 설정

<img src="/assets/images/hsyoo/20230831-02.png" />

<img src="/assets/images/hsyoo/20230831-03.png" />

해당 플러그인은 기본적으로 ts-node를 사용하지만 ts-node-dev를 대신해서 사용할 수 있습니다.

extra ts-node options에 [ts-node-dev](https://github.com/wclr/ts-node-dev){:target="_blank"}의 옵션을 넣어주면 됩니다

--respawn 옵션을 넣어주면 파일이 변경될 때마다 자동으로 재실행을 해줍니다.

--transpile-only 옵션을 넣어주면 타입체크를 하지 않습니다.

--inspect 옵션을 넣어주면 디버깅을 할 수 있습니다.

개발할 때 유용하게 사용할 수 있겠습니다.
