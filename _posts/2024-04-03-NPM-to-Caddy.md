---
layout: post
title:  "NPM에서 Caddy로"
date:   2024-04-03 09:00:00 +0900
author: hsyoo
categories: nginx caddy
published: true
---
<hr>

## 배경

피앰아이는 약 4-5년 전부터 신규 서비스를 하는데 클라우드를 활용해왔습니다.

그 편리함과 확장성, 고가용성은 더 이상 온프레미스 서버를 쓰고 싶지 않게 만들 정도였고,

저를 클라우드 맹신론자(?)로 만들기까지 했습니다.

AWS를 시작으로, 정부 과제를 하면서 사용하게 된 네이버 클라우드, 그리고 Azure까지.

클라우드는 여전히 매력적이고 편리했지만, 역시나 비용이 발목을 잡았습니다.

설문 조사가 많아지고, 대용량의 미디어파일이 제공되는 설문 조사가 많이 진행됨에 따라 데이터 전송 비용도 만만치 않게 되었습니다.

저희가 제공하는 서비스들이 모두 많은 매출과 이익을 안겨주면 좋겠지만, 그렇지 않은 서비스들도 당연히 있기 때문에,

사용량이 예측이 되는 서비스들은 온프레미스로 돌아가기로 결정했습니다.

최근 IDC에 있던 윈도우 서버를 지우고 새롭게 리눅스 서버로 변경했습니다.

os는 우분투를 설치했고, docker 기반의 운영을 위한 포테이너를 설치했습니다.

그동안 nginx에 익숙해졌기에, nginx proxy를 쉽고 편하게 관리해주는 오픈 소스인 NPM(nginx proxy manager)을 설치했습니다.


<hr/>
## Nginx proxy manager

[https://github.com/NginxProxyManager/nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager){:target="_blank"}  

webUI를 기본적으로 제공하고 있기 때문에, 설정이 쉽고 tls 인증서 발급 역시 매우 쉽고 직관적으로 할 수 있었습니다.

이정도라면 팀원들 누구나 proxy를 설정하고 수정 / 배포할 수 있을 거라 생각했습니다. 

![20240403_1.png](/assets/images/hsyoo/20240403_2.png)
<sub>[그림1]nginx proxy manger console</sub>

서비스를 모두 옮기고, 조금은 느리다는 느낌을 받았지만 크게 문제될 정도는 아니였습니다.

하지만 시간이 지나면서 서비스가 느려지는 현상이 발생했고, 인증서 재발급 과정에서도 문제가 계속 발생되었습니다.

SPA로 개발된 웹사이트를 서비스할때는 원인을 알 수 없는 속도 저하가 발생하기도 했고, 최근엔 꽤 빈번하게 순단 현상이 발생하기도 했습니다.

![20240403_1.png](/assets/images/hsyoo/20240403_1.png)  

<sub>[그림2] NPM github</sub>

github star만 보고 선택한 것이 문제였습니다. nginx proxy manager의 issue는 **1.3k**가 넘었고

버전에 따라 많은 문제들이 있는 걸 뒤늦게 알았습니다.

결국 nginx proxy manager를 대체할 만한 것을 찾기로 했습니다.

<hr/>
## Caddy

![20240403_4.png](/assets/images/hsyoo/20240403_4.png)

![20240403_5.png](/assets/images/hsyoo/20240403_5.png)

<sub>[그림3, 4] npm issue에서 Caddy를 추천하는 사람들</sub>


NPM의 쌓여가는 이슈에 해결되는 게 없자 많은 엔지니어들이 Caddy를 추천하는 걸 볼 수 있었습니다.

[https://github.com/caddyserver/caddy](https://github.com/caddyserver/caddy){:target="_blank"}  

![20240403_3.png](/assets/images/hsyoo/20240403_3.png)

<sub>[그림5] Caddy github</sub>


NPM과 비교가 실례일 정도로 관리가 잘되고 있었고, star도 몇 배나 많습니다. 익숙한 nginx를 써야된다는 생각에 사로잡힌 나머지 많은 걸 놓치고 있었습니다..

다음은 Chat GPT에 Caddy의 장점에 대해 물어봤습니다.

>HTTPS 기본 지원: Caddy는 자동으로 SSL/TLS 인증서를 발급하고 갱신합니다. Let's Encrypt를 사용하여 무료 인증서를 제공하며, 필요한 경우 인증서 관리에 대해 걱정할 필요가 없습니다.

> 간편한 구성: Caddy는 사용자 친화적인 Caddyfile로 설정을 관리할 수 있으며, 구성이 상대적으로 간단합니다. 이는 개발자와 시스템 관리자가 빠르게 웹 서버를 설정하고 관리할 수 있게 해줍니다.

> 자동 HTTPS 리디렉션: HTTP 요청이 들어오면 자동으로 HTTPS로 리디렉션합니다. 이는 보안을 강화하고 최신 웹 표준을 준수하는 데 도움이 됩니다.

> 확장 가능성: 다양한 플러그인과 모듈을 통해 기능을 확장할 수 있습니다. 이를 통해 로드 밸런싱, 캐싱, 헬스 체크, 백엔드 서비스에 대한 리버스 프록시 등 다양한 기능을 추가할 수 있습니다.

> 고성능: Go 언어로 개발되었기 때문에 Caddy는 고성능을 제공합니다. 빠른 속도와 효율적인 메모리 사용으로 인해 높은 트래픽을 처리할 수 있습니다.

> 크로스 플랫폼 지원: Caddy는 다양한 플랫폼에서 실행될 수 있으며, Linux, macOS, Windows 등에서 사용할 수 있습니다.

> API를 통한 동적 구성: Caddy는 API를 통해 실행 중에도 서버 설정을 변경할 수 있어, 유연하게 서비스를 관리할 수 있습니다.

이제 온프레미스 서버에 Caddy를 적용했던 경험을 공유 하겠습니다.

<hr/>
## Portainer stacks 활용 & Route53 module 추가

포테이너 stacks는 포테이너에서 docker-compose를 관리하는 기능입니다.

Caddy를 아래와 같이 추가했습니다.

```yaml

version: "3.8"
services:
  caddy:
    image: 1234567890.dkr.ecr.ap-northeast-2.amazonaws.com/mgm:caddy-route53
    container_name: caddy
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
      - "2019:2019"
    volumes:
      - caddy_data:/data
      - caddy_config:/config
      - ./Caddyfile:/etc/caddy/Caddyfile
```

- image가 ECR 주소인 이유는 route53을 통해 인증서를 받기 위해 모듈을 추가해서 따로 빌드했기 때문입니다. (아래에 docker 파일 첨부)

- Caddyfile은 nginx의 conf파일 같은 역할을 합니다.

[https://caddyserver.com/docs/modules/dns.providers.route53](https://caddyserver.com/docs/modules/dns.providers.route53){:target="_blank"} 
[https://github.com/caddy-dns/route53](https://github.com/caddy-dns/route53){:target="_blank"}

```dockerfile
FROM caddy:2.7.6-builder as builder

RUN xcaddy build v2.7.6 \
    --with github.com/caddy-dns/route53@v1.3.3

FROM caddy:2.7.6

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
RUN /usr/bin/caddy version
RUN /usr/bin/caddy list-modules --skip-standard --versions

```
<hr/>

## Caddyfile

아래는 Caddyfile의 예시입니다. FE와 BE를 분리하고 역방향 프록시로 연결했습니다.  
인증서 요청을 위해 route53 모듈을 추가했습니다.

```caddyfile
*.teamcp.pmirnc.io {

   # 개발 블로그
   @blog host blog.teamcp.pmirnc.io
   handle @blog {
      handle /api-docs/* {
         reverse_proxy blog-be:3000
      }
      reverse_proxy blog-fe:8080
   }

   tls {
      dns route53 {
         max_retries 3
         access_key_id "AWS ACCESS KEY"
         secret_access_key "AWS SECRET KEY"
         region "ap-northeast-2"
      }
   }
}
```

사실 이게 전부입니다.

이렇게 되면 인증서는 automate 갱신됩니다.

<hr/>
## 결론

Caddy를 사용하고 나서 속도 저하나 순단 현상이 전혀 발생하지 않았습니다.

앞서 문제가 되었던 SPA 웹사이트의 속도 역시 바로 해결되었습니다.

Caddy는 NPM보다 더 쉽고 빠르게 설정할 수 있었고, 인증서 관리 및 automate renewal 역시 매우 편리했습니다.

한가지 webUI가 없다는 점인데, Caddyfile의 심플함을 보니 그런 부분은 크게 문제가 되지 않았습니다.

이번에 **크게 얻은 교훈이 있다면, star 수만 보고 선택하지 말고, issue와 PR도 고려해서 도입을 검토해야 한다는 점입니다.**

<hr/>
## NPM의 미래(?)

nginx는 대표적인 웹서버입니다. 이미 그 성능과 범용성을 바탕으로 전세계적인 사랑을 받아왔습니다.

NPM은 nginx의 문제가 아닌 점을 명확히 하고 싶습니다. 

직관적이고 쉬운 webUI를 바탕으로 비개발자들에게도 많이 활용되었던 NPM은 더이상

메인테이너가 많은 이슈를 처리하지 못하고 있는 것처럼 보여집니다.(이미 손을 놓고 있을지도 모릅니다)

NPM의 미래는 어떨지 모르겠지만, Caddy와 같은 경쟁자들이 더 많은 사용자들을 끌어들일 것으로 보입니다.
