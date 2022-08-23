---
layout: post
title:  "메타서베이"
date:   2022-08-18 09:30:00 +0900
author: sh.kim
categories: metasurvey
---
<hr/>


목차
======
------
* [개요](#개요)
* [설정](#설정)
* [질문지](#질문지)
  * [문항생성](#문항생성)
  * [문항복사](#문항복사)
  * [테스트](#테스트)
  * [문항유형](#문항유형)
  * [LOOP](#loop)
  * [제시조건](#제시조건)
  * [확인문구](#확인문구)
  * [AO](#ao)
  * [Z](#z)
  * [Replacer](#replacer)
  * [TEMP](#temp)
  * [주의사항](#주의사항)
* [할당](#할당)
* [DATA](#data)

# 개요

-----
메타서베이는 온라인 설문 제작 솔루션입니다.

설문 제작은 주로 타 부서에서 작업하며 개발부에선 설문을 직접 제작하는 일은 거의 없습니다.

하지만 구현하기 어려운 기능이나 복잡한 로직이 있는 경우 개발부에 업무협조 요청을 하는 경우가 있어 메타서베이에 대해 어느 정도 알고 있어야 합니다.

때문에.. 이번 포스팅에서 메타서베이의 기본 사용법에 대해 준비하였습니다.

<br/>
# 설정

------

프로젝트를 생성하고 해당 프로젝트로 들어가면 설정 화면이 나옵니다.

![img.png](/assets/images/sh/20220822/info/img.png)

테스트를 위해 필요한 부분만 설정합니다.

진행상태: 대기 -> 진행

네비게이션: 테스트 중에만 보여줌

<br/>
# 질문지

------
## 문항생성
질문지 탭으로 이동하면 처음에는 빈 화면으로 시작합니다.

![img.png](/assets/images/sh/20220822/question/create/img.png)

상단 메뉴 중 '새 문항' 버튼을 클릭하면 문항이 생성됩니다. (아무거나)

![img_3.png](/assets/images/sh/20220822/question/create/img_3.png)

문항을 생성하면 문항을 수정할 수 있는 화면이 나옵니다. 우선 샘플 버튼을 클릭합니다.

![img_2.png](/assets/images/sh/20220822/question/create/img_2.png)

샘플 항목에서 문항을 선택하고

![img_1.png](/assets/images/sh/20220822/question/create/img_1.png)

저장

![img_4.png](/assets/images/sh/20220822/question/create/img_4.png)

------
## 문항복사
이전에 진행했던 프로젝트에서 문항을 복사해야 하는 경우가 있습니다.

문항을 생성하고 복사해오기를 클릭하면 

![img_1.png](/assets/images/sh/20220822/question/copy/img_1.png)

'설문 선택'에서 이전에 생성한 프로젝트 목록을 확인할 수 있고

![img.png](/assets/images/sh/20220822/question/copy/img.png)

이전 프로젝트를 선택하고 '문항 선택'에서 문항을 선택하고 복사 버튼을 누르면 복사가 됩니다.  
(ctrl 또는 shift 키를 이용하여 여러 개의 문항을 한번에 선택 가능)

![img_2.png](/assets/images/sh/20220822/question/copy/img_2.png)








---
## 테스트
문항을 생성하고 상단 메뉴에서 테스트를 클릭하면 설문을 테스트할 수 있습니다.

![img_5.png](/assets/images/sh/20220822/question/test/img_5.png)

테스트와 관련된 메뉴는 총 4개 입니다.

![img.png](/assets/images/sh/20220822/question/test/img.png)

|**테스트 ID**|UID를 입력하기 위한 폼<br/>**UID는 설문 참여자의 고유번호**이며 설문에 참여하기 위해 반드시 필요함|
|**테스트**|설문을 처음부터 테스트|
|**실제참여**|실제 참여링크로 접속|
|**문항테스트**|선택된 문항부터 테스트|

테스트할 땐 굳이 UID를 입력하지 않아도 테스트가 가능합니다.

이를 이해하기 위해 잠시 URL 탭으로..

### URL

![img_3.png](/assets/images/sh/20220822/question/test/img_3.png)

URL 탭으로 이동하면 설문참여 링크 2개가 기본으로 생성되어 있는 것을 확인할 수 있습니다.

하나는 TEST

![img_1.png](/assets/images/sh/20220822/question/test/img_1.png)

하나는 REAL(실제참여)

![img_2.png](/assets/images/sh/20220822/question/test/img_2.png)

**차이점**

|OPTION|TEST|REAL|
|---|---|---|
|TEST|Y|N|
|뒤로가기|Y|N|
|UID 임의생성|Y|N|

TEST는 UID 임의생성이 Y로 되어 있기 때문에 UID를 입력하지 않을 경우 자동으로 생성해 줍니다

![img_4.png](/assets/images/sh/20220822/question/test/img_4.png)

이런 값으로.. 그래서 TEST는 굳이 UID를 입력하지 않아도 됩니다.

참고로 **REAL 링크는 진행 상태가 진행으로 되어 있을 때만 접속할 수 있습니다.**

![img.png](/assets/images/sh/20220822/question/test/img_6.png)

---
## 문항유형
문항유형은 대표적인 유형과 해당유형의 값이 어떻게 저장되는지에 대해서만 간단히 설명하겠습니다.

### 1. RADIO
여러 개의 보기 중 한 개의 보기를 선택하는 문항입니다.  
일반적으로 문항의 보기는 키와 값을 갖고 있습니다.  
값은 설문지에 표시되는 항목, 키는 실제로 저장되는 값입니다.  

![img_2.png](/assets/images/sh/20220822/question/type/img_2.png)

설문에서 남성을 선택하면

![img.png](/assets/images/sh/20220822/question/type/img.png)

남성에 해당하는 1의 값이 저장됩니다.

![img_1.png](/assets/images/sh/20220822/question/type/img_1.png)

RADIO 문항의 데이터는 KEY가 '1'로 저장됩니다.

만약 위에서 여성을 선택했을 경우 데이터는 '1': '2'로 저장됩니다.  

### 2. CHECK
여러 개의 보기를 선택할 수 있는 문항입니다.

![img_3.png](/assets/images/sh/20220822/question/type/img_3.png)

![img_4.png](/assets/images/sh/20220822/question/type/img_4.png)

![img_5.png](/assets/images/sh/20220822/question/type/img_5.png)

### 3. GRADE
순위형 문항입니다.

![img_6.png](/assets/images/sh/20220822/question/type/img_6.png)

![img_7.png](/assets/images/sh/20220822/question/type/img_7.png)

![img_8.png](/assets/images/sh/20220822/question/type/img_8.png)

### 4. TEXT (OPEN)
텍스트나 숫자를 입력받기 위한 문항입니다.

[TEXT_n] = 텍스트  
[NUMBER_n] = 숫자

![img_9.png](/assets/images/sh/20220822/question/type/img_9.png)

![img_10.png](/assets/images/sh/20220822/question/type/img_10.png)

![img_11.png](/assets/images/sh/20220822/question/type/img_11.png)

이외의 유형이나 자세한 내용은 가이드를 참조해 주시면 됩니다.

---
## LOOP
LOOP는 CHECK 문항의 보기별로 반복되는 설문을 구현하기 위한 기능입니다.

먼저 LOOP의 기준이 되는 CHECK 문항을 만듭니다.

![img.png](/assets/images/sh/20220822/question/loop/img.png)

반복할 문항을 생성하고 기타 옵션의 반복 문항 항목에 CHECK 문항번호를 입력합니다.  
([LOOP_VALUE:CHECK 문항번호] = LOOP 보기 값)

![img_1.png](/assets/images/sh/20220822/question/loop/img_1.png)

CHECK 문항에서 값을 선택합니다.

![img_2.png](/assets/images/sh/20220822/question/loop/img_2.png)

첫 번째 반복문항

![img_3.png](/assets/images/sh/20220822/question/loop/img_3.png)

두 번째 반복문항

![img_4.png](/assets/images/sh/20220822/question/loop/img_4.png)

---
## 제시조건
제시조건은 특정 조건에 따라 문항이나 보기를 제시하거나 생략해 주는 기능입니다.

### 문항제시 조건

문항제시 조건을 클릭하면 문항생략과 문항제시를 설정하는 영역이 나옵니다.

문항생략에 조건을 입력하면 기본적으로 문항을 제시하고 조건에 맞는 경우에만 문항을 생략하고

문항제시에 조건을 입력하면 기본적으로 문항을 생략하고 조건에 맞는 경우에만 문항을 제시합니다.

![img.png](/assets/images/sh/20220822/question/condition/img.png)

예시로 문항생략 조건에 [SQ1=1]라는 값을 넣어 주었습니다.

[SQ1=1]는 SQ1 문항에서 1번 보기를 선택했는지를 확인합니다.

SQ1 문항에서 1번을 선택했다면 조건이 true가 되어 해당 문항은 생략됩니다.

조건에서 사용할 수 있는 문법들에 대해서는 가이드를 참조해 주세요..

### 보기제시 조건

보기제시 조건은 '보기들' 영역이 하나 더 추가되었습니다.

조건을 입력하고 '보기들'에 각 보기의 키값들을 넣어주면

조건이 true가 되는 경우 해당 보기들을 제시하거나 생략합니다 

![img_1.png](/assets/images/sh/20220822/question/condition/img_1.png)

만약 이전문항과 보기 항목이 같은 경우 보기들에 이전문항의 번호를 넣어주면

이전문항에서 선택한 보기들만 제시하거나 생략할 수 있습니다.

![img_2.png](/assets/images/sh/20220822/question/condition/img_2.png)

---
## 확인문구
확인문구는 응답을 재확인할 수 있도록 알림창을 띄우거나

![img_1.png](/assets/images/sh/20220822/question/confirm/img_1.png)

![img_2.png](/assets/images/sh/20220822/question/confirm/img_2.png)

특정 조건에서 다음 문항으로 넘어가지 못하도록 제어할 수 있습니다.

![img_3.png](/assets/images/sh/20220822/question/confirm/img_3.png)

![img_4.png](/assets/images/sh/20220822/question/confirm/img_4.png)

---
## AO 
AO는 응답데이터가 저장되는 객체입니다.

작업을 하다보면 이전 문항의 응답 값을 가져와야 하는 경우가 있습니다.

이때 AO를 사용하여 이전 문항의 응답 값을 가져옵니다.

![img_1.png](/assets/images/sh/20220822/question/ao/img_1.png)

![img_2.png](/assets/images/sh/20220822/question/ao/img_2.png)

SQ1문항과 SQ2문항을 위와 같이 응답한 경우

![img.png](/assets/images/sh/20220822/question/ao/img.png)

AO에 저장된 값입니다.

![img_3.png](/assets/images/sh/20220822/question/ao/img_3.png)

이렇게 값을 가져올 수 있습니다.

---
## Z
Z는 이전문항의 값을 가져오거나 응답 여부를 확인하는 기능을 제공합니다.

모든 문항에 적용되진 않아 AO와 같이 사용하는 것이 좋습니다.

### SINGLE
단일문항(RADIO)류에서 응답한 보기의 정보를 가져올 수 있습니다.

![img.png](/assets/images/sh/20220822/question/z/img.png)

주로 KEY와 VALUE를 많이 사용합니다.

![img_1.png](/assets/images/sh/20220822/question/z/img_1.png)

### ARY
다중문항(CHECK)류에서 응답한 보기들의 정보를 배열의 형태로 가져올 수 있습니다.

![img_2.png](/assets/images/sh/20220822/question/z/img_2.png)

![img_3.png](/assets/images/sh/20220822/question/z/img_3.png)

SINGLE의 경우 AO를 사용하는 것에 비해 효용성이 크진 않지만

ARY는 다중문항의 경우 키값이 배열로 필요한 경우가 꽤 있어서 AO보다 좀 더 유용하게 사용할 수 있습니다.

### TF.HAS
TF.HAS는 응답 여부를 확인하는 기능을 제공합니다.

![img_4.png](/assets/images/sh/20220822/question/z/img_4.png)

question: 문항번호  
key: 보기 KEY값

![img_5.png](/assets/images/sh/20220822/question/z/img_5.png)

---
## Replacer
Replacer는 값 치환과 관련된 기능을 제공합니다.

AO나 Z와 마찬가지로 이전 응답값을 가져오지만 스크립트 영역이 아닌 곳에서 주로 사용됩니다.

메타서베이 도움말에 치환과 관련된 항목을 정리해 놓은 글이 있으니 참고하시면 됩니다.

![img.png](/assets/images/sh/20220822/question/replacer/img.png)

![img_1.png](/assets/images/sh/20220822/question/replacer/img_1.png)

위와 같은 치환값은 내부적으로 Replacer를 사용하여 값을 치환합니다.

![img_2.png](/assets/images/sh/20220822/question/replacer/img_2.png)

---
## TEMP

TEMP는 임시 변수나 함수를 만들어주는 기능을 제공하며 치환값으로 사용할 수도 있습니다.

개인적으로 가장 유용하게 사용하는 기능입니다.

![img.png](/assets/images/sh/20220822/question/temp.png)

마찬가지로 메타서베이 도움말 참고..

---
## 주의사항

### 1. 설문에서 작성한 스크립트는 모든 문항에 영향을 미칩니다.

![img.png](/assets/images/sh/20220822/question/caution/img.png)

SQ1 문항에서 다음 버튼에 click 이벤트를 bind 해주었습니다.

해당 스크립트는 SQ1 문항에서만 작동될 것으로 생각할 수 있습니다.

![img_1.png](/assets/images/sh/20220822/question/caution/img_1.png)

하지만...

![img_2.png](/assets/images/sh/20220822/question/caution/img_2.png)

SQ2 문항에서도 alert 창이 뜨는 것을 확인할 수 있습니다.

메타서베이에서 한번 실행된 스크립트는 새로고침하지 않는 이상 지속됩니다.

공통으로 사용되는 항목들(다음버튼 등..)에 대한 스크립트 적용은 되도록 지양하는 것이 좋습니다. 
   
### 2. IE 사용자를 고려하여 작업 해야합니다.

공식적으로 IE는 서비스 종료가 되었지만 그럼에도 불구하고.. 아직 IE를 사용하는 응답자들이 있습니다.

때문에 메타서베이 작업 시 IE에서 지원하는 문법으로만 스크립트를 작성해야 합니다.

가장 많이 실수하는 문법이 const, let, 화살표 함수이며

let/const -> var, 화살표 함수 -> 일반함수로 작성해 주셔야 합니다.

이 외에도 IE에서 지원하지 않는 문법이 꽤 있기 때문에 **테스트 시 IE에서도 꼭 확인** 해주세요..

---
# 할당
할당은 쿼터(QUOTA)라고도 하며 각 문항의 보기별로 응답 수를 지정합니다

![img.png](/assets/images/sh/20220822/quota/img.png)

![img_1.png](/assets/images/sh/20220822/quota/img_1.png)

위의 두 문항에 대해 설정된 할당입니다.

![img_2.png](/assets/images/sh/20220822/quota/img_2.png)

설문에서 할당이 모두 채워진 경우

![img_3.png](/assets/images/sh/20220822/quota/img_3.png)

조사가 바로 종료됩니다

![img_4.png](/assets/images/sh/20220822/quota/img_4.png)

---
# DATA
DATA 탭에선 데이터를 다운로드하거나 삭제할 수 있습니다.

작업을 할 때 **데이터가 제대로 저장되는지 확인하는 것이 중요**합니다. 

'DATA' 혹은 '전체' 버튼을 클릭하면 데이터 다운로드할 수 있습니다.  
(DATA는 완료데이터만 다운로드합니다)

![img.png](/assets/images/sh/20220822/data/data_down.png)

데이터를 다운로드하면 생성날짜가 표시되고 'DATA 다시 생성' 버튼이 생깁니다.

![img_1.png](/assets/images/sh/20220822/data/img_1.png)

데이터를 초기화 하거나 수정한 경우 '완료건 재생성' 버튼을 누릅니다.

잠시 뒤에 'DATA 다시 생성' 버튼을 누르면 수정된 데이터를 다시 받을 수 있습니다.

**데이터 초기화(삭제)는 되도록 하지 않는 것을 추천**합니다.


<br/>
# 끝

























