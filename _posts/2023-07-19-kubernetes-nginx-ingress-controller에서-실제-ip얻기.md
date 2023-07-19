---
layout: post
title:  "kubernetes nginx ingress controller에서 실제 ip얻기"
date:   2023-07-18 09:00:00 +0900
author: hsyoo
categories: kubernetes nginx ingress controller
---
<hr/>

산업통상자원부의 정부과제를 진행하면서 ncloud의 kubernetes를 사용하고 있습니다.  

nlb 로드밸런서(ncloud에선 nplb- network proxy load balancer)와 nginx ingress controller를 설치해서 서비스를 노출시키고 있습니다.

api log를 남기면서 요청자의 ip 주소를 확인해보니 로드밸런서의 private ip가 찍히고 있었습니다.

이를 해결하기 위해 nginx ingress controller의 설정을 변경해야 했습니다.

[https://guide.ncloud-docs.com/docs/nks-nks-1-4](https://guide.ncloud-docs.com/docs/nks-nks-1-4){:target="_blank"}

문서에는 아래와 같이 설명합니다

```markdown
ingress-nginx를 통해 Client의 Real IP를 온전하게 취득하기 위해서는 LoadBalancer의 proxy protocol 설정과 ingress-nginx의 use-proxy-protocol: true ConfigMap이 설정되어 있어야 합니다.

Client의 IP를 확인하고자 하는 경우에는 service.beta.kubernetes.io/ncloud-load-balancer-proxy-protocol의 값을 true로 선언합니다.

```

생각보다 간단하군요. 

- 로드밸런서에 service.beta.kubernetes.io/ncloud-load-balancer-proxy-protocol 어노테이션 추가
- ingress nginx의 cm에 use-proxy-protocol 설정

먼저 로드밸런서의 proxy protocol 설정은 아래와 같이 변경하였습니다. (필요없는 정보는 삭제했습니다.)

```yaml
# loadbalancer

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx  
  annotations:
    service.beta.kubernetes.io/ncloud-load-balancer-proxy-protocol: "true"
```

어노테이션 추가하는 게 큰 이슈가 될 거라곤 생각 못하고 있었는데, 여기서 첫번째 좌절을 느낍니다.  
적용 후 배포된 웹에 접속해보니 ERR_SSL_PROTOCOL_ERROR이 떴습니다.

부랴부랴 롤백 후 ncloud에 문의를 해봤지만 문서에 명시되어있는 답변이 돌아왔습니다.  
그리고 ingress nginx같은 써드파티는 지원이 어렵다고 하네요.   
지원은 어렵지만 문서에 명시되어있다..   
조금 이해하기 어려웠습니다 

왠지 하라는 2개 중 1개만 적용한 게 의심됩니다.  
이번엔 로드밸런서와 cm 수정을 함께 적용해보기로 했습니다.


```yaml
#ingress nginx cm
apiVersion: v1
data:
  use-forwarded-headers: "true" #X-forwarded-* 헤더를 포워딩
  use-proxy-protocol: "true"
kind: ConfigMap

```


정상작동하네요. 시키는 대로만 되는데 그게 어렵습니다.  
ingress nginx에 로그도 잘 찍힙니다.

하지만 여전히 nestjs로 배포된 pods에서는 로드밸런서의 private ip가 찍히고 있었습니다.  
여기서 두번째 멘붕이 왔습니다.
 
여기서 BE app에서 잘못 처리하고 있는 게 아닐까 하는 킹리적 갓심을 하게 됩니다.  
불현듯 nest를 쓰기 전 express로 코딩하던 때가 생각납니다.  
proxy 관련 설정을 해줘야 했던 것 같은데..  

nest.js는 기본적으로 express.js를 사용하는데 express.js가 req.ip를 처리하는 방법을 알아볼 필요가 있겠습니다.

```javascript
defineGetter(req, 'ip', function ip() {
    var trust = this.app.get('trust proxy fn');
    return proxyaddr(this, trust);
});
```

[https://expressjs.com/en/4x/api.html#trust.proxy.options.table](https://expressjs.com/en/4x/api.html#trust.proxy.options.table){:target="_blank"}  
proxyaddr()은 X-Forwarded-For값을 가져와서 마지막 ip를 반환합니다.
우리에게 필요한건 XFF의 첫번째 ip이기 때문에 아래와 같이 추가해줬습니다.

```typescript
// main.ts
app.set('trust proxy', true);
```


BE를 다시 배포한 뒤 로그를 확인해보니 드디어 정상적으로 client ip가 찍히고 있었습니다.
덕분에 제 소중한 저녁 시간을 소비하고 좋은 경험을 했습니다.  

참고 문서  
[https://blog.naver.com/alice_k106/221505569455](https://blog.naver.com/alice_k106/221505569455){:target="_blank"}  
[https://findstar.pe.kr/2021/08/22/nginx-ingress-controller-use-forwarded-for-option/](https://findstar.pe.kr/2021/08/22/nginx-ingress-controller-use-forwarded-for-option/){:target="_blank"}  
[https://satisfactoryplace.tistory.com/368](https://satisfactoryplace.tistory.com/368){:target="_blank"}
