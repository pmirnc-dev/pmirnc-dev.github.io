---
layout: post
title: "Tree Shaking"
date: 2024-05-07 17:00:00 +0900
author: mclee
categories: typescript javascript
published: true
---
<hr/>

## Tree Shaking 이란 뭘까요?


Tree Shaking은 다양한 방식으로 해석할 수 있습니다.

1. 생태학적 측면
   - 나무 흔들림이 생태계에 미치는 영향, 흔들림의 원인 및 종류, 관련 연구 결과 등

2. 임업적 측면
   - 나무 흔들림으로 인한 피해, 흔들림 방지 기술, 산림 관리 전략 등

3. 문화적 측면
   - 나무 흔들림과 관련된 신화, 전설, 속담, 예술 작품 등

4. 비유적 의미
   - 나무 흔들림을 이용한 비유 표현, 흔들림이 상징하는 의미, 문학 작품에서의 활용 등

5. 프로그래밍 언어 측면
   - 주로 JavaScript에서 사용되며, 번들링 과정에서 사용되지 않는 코드를 자동으로 제거하는 최적화 기술로,
     실제로 사용되지 않는 코드를 제거하여 번들 크기를 줄이고 로딩 속도를 향상시키는데 목적을 두고 있습니다.

## Tree Shaking 작동 방식

3단계를 거쳐 작동합니다.

1. 모듈 분석 
   - 번들러는 모든 모듈을 분석하고 각 모듈에서 사용되는 변수와 함수를 파악합니다.

2. 사용되지 않는 코드 식별
   - 분석 결과를 기반으로 실제로 사용되지 않는 코드를 식별합니다.

3. 번들 최적화
   - 식별된 사용되지 않는 코드를 번들에서 제거합니다.


### 장점

1. 번들크기 감소
     - 사용되지 않는 코드를 제거하여 번들 크기를 줄여 로딩 속도를 향상시킵니다.
2. 성능 향상
     - 작은 번들은 네트워크 대역폭 사용량을 줄이고 페이지 로딩 시간을 단축하여 사용자 경험을 개선합니다.
3. 유지 관리 용이성
     - 코드베이스를 더 작고 가독성이 높게 만들어 유지 관리를 용이하게 합니다.

### 단점

1. 설정 및 유지 관리 복잡성 증가
2. 코드 가독성 저하 가능성
3. 예상치 못한 동작 가능성
4. 모든 번들러에서 동일하게 지원되지 않음
5. 테스트 및 코드 리뷰 어려움 증가
6. 성능 저하 가능성

### 대표적인 번들러

Webpack : 가장 인기 있는 번들러 중 하나, 다양한 기능과 확장성을 제공 

Rollup : ES6모듈을 지원, 경량하고 사용하기 쉬운 버들러

Parcel : 빠른 개발 속도를 제공, 설정 없이 바로 사용할 수 있는 번들러


현재 예시로 Rollup 사이트 체험을 하도록 하겠습니다.
[rollup 사이트](https://rollupjs.org/)

---

다음시간에는 enum 열거형에 관련된 애기를 할려고 합니다.

주제는 Tree Shaking 관점에서 TypeScript enum을 사용하지 않는게 좋은 이유 입니다.

순위 : Union Types > const enum > enum
