---
layout: post
title:  "External dns와 K8s의 버전 호환성"
date:   2022-06-02 20:00:00 +0900
author: hsyoo
categories: aws eks external-dns k8s
---
<hr/>

최근 AWS EKS 버전 업그레이드를 했음.

![img.png](/assets/img.png)

panel-hub 클러스터에서 개발용 도메인 추가할 게 있어서 ingress에 도메인 추가했음

이미 클러스터에는 external-dns가 설치되어 있는 상황

아무리 기다려도 레코드 추가가 안됨

<img src="/assets/img_1.png" style="width:20rem;" />

삽질 끝에 원인을 알아냄

클러스터에 설치한 external-dns의 버전은 아래와 같았음.

`image: k8s.gcr.io/external-dns/external-dns:v0.7.6`

external-dns 공식 문서에 나온 호환성..

엌

external-dns의 버전은 0.10.2+로 바꾼 뒤 바로 레코드가 추가되었다..

오늘의 교훈

# **버전업 함부로 하지말자**


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
