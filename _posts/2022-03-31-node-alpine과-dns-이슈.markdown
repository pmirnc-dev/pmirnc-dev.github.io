---
layout: post
title:  "node alpine과 dns 이슈"
date:   2022-03-31 17:55:00 +0900
author: hsyoo
categories: docker dns node.js
---

현재 PMI의 설문 조사 IN / OUT router app은 IDC에 있는 linux 서버에

docker-compose형태로 node js App(nest.js)으로 구현되어 있음

각 2개의 앱 모두 [axios](https://github.com/axios/axios)를 이용해서 따로 개발된 endpoint(aws EKS로 개발된 또다른 node app)을

호출하게 되어있는데 하루에 많진 않지만 간헐적으로

**getaddrinfo EAI_AGAIN** 이라는 오류를 뱉는 것이였다.

처음엔 EKS에 올라가 있는 app을 무조건 의심하면서 해결점을 찾고 있었다

그치만 찾으면 찾을수록 AWS EKS에서 발생되는 이슈가 생각보다 없는 게 아닌가..

분명 이런 이슈라면 다른 많은 개발자들도 경험했을 거라 생각했는데 없었다.

혹시 요청하는 서버의 문제는 아닐까라고 반대로 생각하고 찾아본 결과 내 예상이 맞았다

```docker
FROM node:14-alpine
RUN apk --no-cache add tzdata && \
  cp /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
  echo "Asia/Seoul" > /etc/timezone
```

alpine이미지에서 사용되는 [musl libc](https://www.musl-libc.org/)에서 dns lookup 관련하여 문제를 일으키는 것..

```docker
FROM public.ecr.aws/bitnami/node:14-debian-10
ENVTZ="Asia/Seoul"
```

위와 같이 debian으로 변경 후에 새롭게 빌드 후 배포하니 간헐적으로 생기던 dns관련 이슈는 사라졌다.

전혀 알수도 없고 예측할 수 없었던 이 삽질은 거의 한달만에 해결이 됨...

참고링크

[https://gracefullight.dev/2020/11/29/nodejs-dns-cache-on-k8s/](https://gracefullight.dev/2020/11/29/nodejs-dns-cache-on-k8s/)

[https://github.com/nodejs/docker-node/issues/602](https://github.com/nodejs/docker-node/issues/602)


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
