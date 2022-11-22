---
layout: post
title:  "Rolling Index Builds on Replica Sets"
date:   2022-11-22 09:00:00 +0900
author: hsyoo
categories: mongodb index
---
<hr/>

PMI의 웹설문 조사 시스템은 Mongodb를 cloud 플랫폼인 atlas를 사용하고 있습니다.  
사용자의 설문 raw data가 쌓이는 컬렉션에 인덱스를 생성해야 할 일이 생겼습니다.  
초반에 만들었어야 되는데 놓쳤다가 인덱스 관련 오류가 뜨면서 발견이 되었습니다.  
사실 atlas를 사용하기 때문에 performance advisor에서 추천을 해줬었지만 크게 신경을 안썼던 불찰이 크죠..

어쨌거나 400M이 넘는 row size에 document size도 제각각에 꽤 큰 컬렉션이고  
실시간으로 많이 쌓이는 상태라 인덱스 생성을 할 경우 서비스에 장애가 생길 것으로 예상이 됐습니다.

mongodb 4.4이상부터는 롤링 인덱스 생성을 지원 합니다  

[https://www.mongodb.com/docs/manual/tutorial/build-indexes-on-replica-sets/](https://www.mongodb.com/docs/manual/tutorial/build-indexes-on-replica-sets/){:target="_blank"}

순서는 다음과 같습니다  

A. Stop One Secondary and Restart as a Standalone

B. Build the Index

C. Restart the Program mongod as a Replica Set Member

D. Repeat the Procedure for the Remaining Secondaries

E. Build the Index on the Primary

레플리카 셋을 순차적으로 down시키면서 인덱스를 생성하는 형태군요. 작업이 꽤 복잡해보입니다.
친절한 atlas에서는 해당 작업을 체크 박스 하나로 수행할 수 있습니다

![img.png](/assets/images/hsyoo/build_index_via_rolling_process.png)

위 그림처럼 체크박스를 선택한 상태에서 진행을 하게 되면  
해당 클러스터에서 레플리카 셋이 순차적으로 down이 되면서 인덱스가 생성됩니다  
(이건 미처 캡쳐를 못했습니다)

![img.png](/assets/images/hsyoo/index_build_done.png)

인덱스 생성을 운영 서버에서 무사히 수행했습니다.

