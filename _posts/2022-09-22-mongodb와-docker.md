---
layout: post
title:  "mongodb docker container 이슈"
date:   2022-09-22 18:30:00 +0900
author: hsyoo
categories: mongodb docker
published: true
---
<hr/>

_필자가 편한 형태로 쓴 글이니 너그럽게 봐주세요_

### 밑밥
전임자가 셋팅한 윈도우 데스크탑에 centOS를 설치한 서버(?)를 개발용으로 막 쓰고 있는데  
mongodb를 docker container로 올려놓은 상태였음  
4년 정도 사용하고 있는데 개발 서버지만 내부에서 돌리는 건 데이터도 넣는 등  
쓰임새가 많아지고 용량도 점점 커지고 있었음  

최근에 해당 서버의 터미널 접속이 느려지는 등 문제점이 포착되서 확인해보니  
mongodb가 너무 많은 리소스를 잡아 먹고 있었다  

### 문제점 발견

아무리 생각해도 이정도에 힘들어하는 게 이해가 안가서 이것저것 쉐복을 하고 있었는데  
아래 블로그에서 경험담을 발견

[https://bearfoon.tistory.com/52](https://bearfoon.tistory.com/52){:target="_blank"}

이유인 즉 컨테이너에서 구동되고 있는 MongoDB가 컨테이너에게 할당된 메모리가 아닌 호스트 머신이 가지고 있는 메모리를 인식하여 cache를 생성하기 때문이라고 함..
mongoDB가 바깥 세상의 메모리를 인식하고 있었던 것이었음...(like 트루먼쇼)


<img src="https://1.gall-img.com/hygall/files/attach/images/82/866/204/226/c71cb3662e989a86a08e5da2215aab42.jpg" style="max-width: 400px" />


<img src="https://t1.daumcdn.net/cfile/tistory/99C5C33A5C04774207" style="max-width: 400px" />

### 해결책

arguments --wiredTigerCacheSizeGB=2 추가(2GB로 제한)

### 결과

![img.png](/assets/images/hsyoo/docker-mongodb.png)

좀 더 지켜보긴 하겠지만 4기가에 육박하던 메모리 사용량은 낮아졌음  
근데 cpu가 인텔 i3 3세대던데.. 좀 너무한 거 아닌가 하는 생각이 든다(2012년에 나왔다고..)  
거의 아동복 입은 느낌..(~~그것도 유행 다 지난~~)

### 여담
centOS는 212년에 지원이 종료되었다

