---
layout: post
title:  "verdaccio 도입기"
date:   2022-09-19 18:30:00 +0900
author: hsyoo
categories: npm verdaccio registry
published: true
---
<hr/>

안녕하세요 피앰아이 개발부 유호석입니다.  
팀원들이 늘고, 수행하는 프로젝트들이 많아 지면서 코드의 재가용성에 대해 고민을 하게 되었습니다.

예를 들어 간단한 로그인 vue 컴포넌트만 해도  
진행할 때마다 개인별로 차이도 있고, 구현 방법에도 차이가 있기 때문이죠..

팀원들만 공유할 수 있는 npm package를 공유할 수 있는 방법이 없을까 고민을 시작했고,  
구글링을 하자마자 답을 찾았습니다.  
바로 오늘 소개해 드릴 **Verdaccio**입니다

# Verdaccio
![https://verdaccio.org/fr-FR/assets/images/verdaccio-tiny@3x-d3bb8784f8646c7ed0e42525eff92783.png](https://verdaccio.org/fr-FR/assets/images/verdaccio-tiny@3x-d3bb8784f8646c7ed0e42525eff92783.png)

> A lightweight private npm proxy registry

> Verdaccio는 설정(configuration) 없이 로컬 npm 레지스트리를 만들 수있는 npm 패키지입니다.

마치 이태리에 있는 느낌을 주는(~~아마 개발자가 이탈리아계가 아닐까~~)  
이 간결하고 유용한 저장소는 제가 생각한 것에 딱 맞는 거였습니다..

내부 서버(docker compose가 올라간)에 설치할 계획이 었으므로 docker image로 빌드하고  
path를 바인딩 시켜서 올리면 되겠네요
배포할 서버를 192.168.1.1이라고 해두겠습니다

우선 설치부터 해보겠습니다

```bash
npm install -g verdaccio
```

설치가 완료되면 verdaccio를 실행시켜봅니다

```bash
verdaccio
```

아래와 같이 기본적으로 4873으로 웹이 뜨게 되네요
```bash

 warn --- config file  - C:\Users\PC-082\AppData\Roaming\verdaccio\config.yaml
 warn --- "crypt" algorithm is deprecated consider switch to "bcrypt". Read more: https://github.com/verdaccio/monorepo/pull/580
 info --- plugin successfully loaded: verdaccio-htpasswd
 info --- plugin successfully loaded: verdaccio-audit
 warn --- http address - http://localhost:4873/ - verdaccio/5.15.3
 http --- 127.0.0.1 requested 'GET /'
 http --- 200, user: null(127.0.0.1), req: 'GET /', bytes: 0/506
 http --- 127.0.0.1 requested 'GET /-/verdaccio/data/packages'
 http --- 200, user: null(127.0.0.1), req: 'GET /-/verdaccio/data/packages', bytes: 0/2

```

설치화면

![img.png](/assets/images/hsyoo/img.png)

몇 가지 설정을 위해 config.yaml을 작성하겠습니다
```yaml

storage: /data #서버와 연결되는 경로
auth:
  htpasswd:
    file: ./htpasswd  #유저 계정이 관리될 파일
    # max_users: -1 # 유저 추가 막기
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  "@pmirnc/*":  # prefix로 @pmirnc로 시작한 경우 private으로 구분
    access: $all # 어차피 내부에서만 사용될 거라 id없이 접근하도록 설정
    publish: $all
    unpublish: $authenticated
    proxy: npmjs
  "**":
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
log: { type: stdout, format: pretty, level: http }
listen:
  - 0.0.0.0:3000  # listen할 PORT
max_body_size: 100mb # publish 할때 허용할 사이즈
```
시작 스크립트
```bash
verdaccio --config ./config.yaml
```

잘 적용이 되었습니다.
이제 docker 빌드를 해보죠
```dockerfile
FROM public.ecr.aws/bitnami/node:14-debian-10
ENV TZ="Asia/Seoul"
RUN mkdir -p /app
WORKDIR /app
RUN mkdir ./data
COPY . .
RUN npm install
ENTRYPOINT ["npm", "run", "start"]
EXPOSE 3000
```
docker compose 설정은 생략하겠습니다.

이제 사용자는 npm 설정을 해줘야 @pmirnc라는 prefix를 쓸 때
내부 private registry를 참조할 것입니다
```bash
npm config set @pmirnc:registry http://192.168.1.1:3000
```

패키지 퍼블리싱은 아래와 같이 합니다
```bash
npm publish --registry http://192.168.1.1.:3000
```

이제 배포된 패키지는 아래와 같이 설치할 수 있습니다
```bash
npm install @pmirnc/babylon
```

verdaccio를 설치함으로써 javascript 패키지 제작에 있어서  
좀 더 부담없이 시도할 수 있게 됐습니다.
이제 모두 패키지 제작을 시작합시다.
