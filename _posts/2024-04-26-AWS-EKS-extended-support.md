---
layout: post
title:  "AWS EKS extended support"
date:   2024-04-26 09:00:00 +0900
author: hsyoo
categories: AWS EKS k8s
published: true
---
<hr>

피앰아이는 많은 수의 웹앱 및 워커앱을 AWS EKS로 운영하고 있습니다.

예전 포스팅에도 언급한 적이 있지만 k8s 버전으로 이슈를 겪은 적이 있기에 업그레이드에 매우 신중하게 진행하고 있습니다.

요금 모니터링을 하는 도중 이번 달(24년 4월)부터 요금이 더 높아진 것을 확인했습니다.

![img.png](/assets/images/hsyoo/20240426-01.png)

24년 4월 1일부터 k8s 버전에 따라 "표준" 혹은 "추가"으로 구분하여 요금을 더 청구한다는 내용이 있었습니다.

![img.png](/assets/images/hsyoo/20240426-02.png)


1.26과 1.23 버전의 클러스터를 운영하고 있는데, 1.23 버전에 대해 추가 요금이 매일 $12씩 청구되고 있었습니다...

(이미지에서 보듯이 바로 업그레이드 중)

AWS EKS를 운영하시는 분들께서는 이번에 추가 요금이 청구되는 부분을 꼭 확인하시고, 업그레이드를 진행하시는 것을 추천드립니다.

참고문서 

[https://aws.amazon.com/ko/blogs/containers/amazon-eks-extended-support-for-kubernetes-versions-pricing/](https://aws.amazon.com/ko/blogs/containers/amazon-eks-extended-support-for-kubernetes-versions-pricing/){:target="_blank"}
[https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/kubernetes-versions.html#kubernetes-release-calendar](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/kubernetes-versions.html#kubernetes-release-calendar){:target="_blank"}
