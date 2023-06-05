---
layout: post
title:  "k8s cert manager를 이용한 TLS 인증서 발급"
date:   2023-06-05 09:00:00 +0900
author: hsyoo
categories: k8s ssl letsencrypt
---

<hr/>

피앰아이에서는 AWS EKS와 ncloud k8s service를 활용해 웹 / 워커 서버를 서비스하고 있습니다.  

이번에는 피앰아이가 서비스 중인 [유니서베이](https://unisurvey.co.kr){:target="_blank"}에   

[cert manager](https://github.com/cert-manager/cert-manager){:target="_blank"}와 [let's encrypt](https://letsencrypt.org/){:target="_blank"}를 이용해 무료 TLS 인증서를 발급하고 적용하는 방법을 알아보겠습니다.


<div style="display: flex; justify-content: space-around; align-items: center;">
<div>
    <img src="https://rastalion.me/wp-content/uploads/2019/09/kubernetes-logo-300x296.png" width="200" />
</div>

<div>
    <img src="https://avatars.githubusercontent.com/u/39950598?s=280&v=4" width="`200" />
</div>
<div>
    <img src="https://labs.consol.de/assets/images/letsencrypt.png" width="200" />
</div>
</div>


### kubernetes
kubernetes에 대한 자세한 설명은 생략하도록 하겠습니다.  

### let's encrypt
Let's Encrypt는 보안 웹사이트를 위한 인증서의 수동 생성, 유효성 확인, 디지털 서명, 설치, 갱신 등 종전의 복잡한 과정을 없애주는 자동화된 프로세스를 통해 전송 계층 보안(TLS) 암호화를 위해 무료 X.509 인증서를 제공하는 인증 기관입니다.




### cert-manager

Cert-manager는 Kubernetes 내부에서 HTTPS 통신을 위한 인증서를 생성하고, 또 인증서의 만료 기간이 되면 자동으로 인증서를 갱신해주는 역할을 하는 Certificate manager controller입니다.

물론 오픈소스입니다.

10.4k의 star가 입증하듯 많은 엔지니어들이 활용하고 있습니다.

AWS EKS에는 cert-manager가 기본적으로 설치가 되어었으므로, ncloud를 기준으로 설명드리겠습니다.

먼저 아래 템플릿을 설치합니다.

```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
```

엄청난 양의 리소스들이 설치되면 잘 설치가 되었는지 pods를 확인해 볼 수 있습니다.

```shell
kubectl get pod -n cert-manager
```

다음으로 cluster-issuer를 생성합니다.  
ingress nginx controller를 사용하고 있기 때문에 class에 nginx를 적었습니다.  
이메일 주소는 담당자 이메일을 적으시면 됩니다.  

```yaml

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-issuer
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: teamcp@pmirnc.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-issuer
    # Enable the HTTP-01 challenge provider
    solvers:
      - http01:
          ingress:
            class:  nginx


```

이미 배포되어 있는 ingress에 몇 가지를 추가 하겠습니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: uni-survey-ingress
  namespace: uni-survey
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/ssl-protocols: "TLSv1.3"
    nginx.org/websocket-services: "socket"
    kubernetes.io/tls-acme: "true" # <- 추가
    cert-manager.io/cluster-issuer: "letsencrypt-issuer" # <- 앞서 생성한 cluster-issuer의 이름
spec:
  tls:
    - hosts:
        - unisurvey.co.kr
      secretName: unisurvey-secret # <- 인증서가 저장될 secret 이름입니다. 
  rules:
    - host: unisurvey.co.kr
      http:
        paths:

```

이제 끝났습니다.

정말 좋은 점은 인증서가 유효한지 검증하여 만료되기 전에 인증서 갱신을 시도해준다는 점입니다.

매번 인증서를 구매, 갱신할 필요 없이 쉽게 적용할 수 있었습니다.

<hr/>

참고 URL
- [https://im-nc2u.tistory.com/entry/kubernetes-%EC%97%90%EC%84%9C-cert-manager%EB%A1%9C-%EB%AC%B4%EB%A3%8C-letsencrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89-%EC%84%A4%EC%A0%95](https://im-nc2u.tistory.com/entry/kubernetes-%EC%97%90%EC%84%9C-cert-manager%EB%A1%9C-%EB%AC%B4%EB%A3%8C-letsencrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89-%EC%84%A4%EC%A0%95){:target="_blank"}
- [https://cert-manager.io/docs/](https://cert-manager.io/docs/){:target="_blank"}
