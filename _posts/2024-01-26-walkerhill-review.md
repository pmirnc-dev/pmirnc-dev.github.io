---
layout: post
title: "walkerhill 리뉴얼 리뷰"
date: 2024-01-26 17:00:00 +0900
author: mclee
categories: Vue3 typescript
---
<hr/>


## web-report-template을 이용하여 walkerhill 사이트 작성하면서 느꼈던점

1. 로그인

    ~~walkerhill 사이트는 로그인 페이지가 없어서 사용하지 못하였습니다.~~
   급작스럽게 로그인 페이지를 추가를 해야 하는 상황이 생겼습니다. 

   우선 두가지의 로그인화면 형태가 있습니다.

      1). 이미지가 없을때 표기 (왼쪽)
 
      2). 이미지가 있을때 표기 (오른쪽)

    ![로그인 페이지](/assets/images/mclee/240126/walkerhill_login_form.png)

    - 느낀점
      
      기본적으로 DB에 해당 프로젝트 정보들을 등록해야만 로그인 화면이 표기되므로 처음 사용했을때에는 어려움이 있었습니다.

      DB 경로 : common-project
      ```JSON
         {
             "_id" : "2",
             "title" : "Walkerhill 웹리포트",
             "login" : {
                "title" : "Walkerhill 웹리포트",
                "btnText" : "접속",
                "id" : {
                   "text" : "아이디",
                   "type" : "text"
                },
                "password" : {
                   "text" : "패스워드",
                   "type" : "password"
                }
             },
             "sidebar" : [
          
             ],
             "favicon" : "",
             "icon" : "",
             "theme" : "SIMPLE"
          }
      ```
        불편한 점을 제외하면 괜찮았습니다...

2. 게시판

    ~~로그인과 마찬가지로~~ 해당 컴포넌트를 사용하지 않았습니다.

3. 차트
    - 차트 컴포넌트

        구현하는 방법은 web-report-template wiki에 작성을 하였습니다. 참조하시면 되겠습니다.
        차트에 들어갈 데이터 형태를 구현하는데 어려움은 없었습니다.
        각 페이지별로 구현할수록 차트 컴포넌트에 필요한 기능들 추가하였습니다.

     [차트 컴포넌트](https://github.com/pmirnc-dev/web-report-template/wiki/1.-%EC%B0%A8%ED%8A%B8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EA%B5%AC%ED%98%84#%EC%B0%A8%ED%8A%B8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EA%B5%AC%EC%A1%B0)

    - 검색 필터 영역

        컴포넌트내에 있는 구성요소들은 radio, select, 날짜 선택(Datepicker), 검색버튼으로 구성되어 있습니다.
        각 페이지별로 어떤 항목들을 표기 할지 로직 설정을 하시면 됩니다.

    [검색 필터](https://github.com/pmirnc-dev/web-report-template/wiki/2.-%EA%B2%80%EC%83%89-%ED%95%84%ED%84%B0-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)

    - 테이블 데이터

        각 페이지별로 보이는 테이블을 생성하기 위해서 테이블 컴포넌트에 들어갈 데이터 형태

    - 개선사항

        추후에 테이블 속성인 colspan, rowspan을 사용할수 있도록 기능 추가하도록 하겠습니다.
 
- 객실 만족도 결과 - 응답자 현황 페이지 - 테이블 데이터
```json
{
    "body": [
        ["243(6.0)", "1,374(33.8)", "1,321(32.5)", "799(19.7)", "326(8.0)" ],
        [ "519(9.1)", "2,107(37.0)", "1,808(31.7)", "940(16.5)", "325(5.7)" ]
    ],
    "heads": [ "20대", "30대", "40대", "50대", "60대 이상", "총합" ],
    "rowHead": [ "남성 N(%)", "여성 N(%)" ],
    "topHead": "구분",
    "sum": [ 4063, 5699 ]
}
```
   ![응답자 테이블](/assets/images/mclee/240126/응답자_테이블.JPG)

- 객실 만족도 결과 - 아이 동반 현황 페이지 - 테이블 데이터
```json
{
  "body": [
    [ "3,183(38.6)","5,053(61.4)" ]
  ],
  "heads": [ "동반", "미동반", "총합" ],
  "rowHead": [ "N(%)" ],
  "topHead": "",
  "sum": [ 8236 ]
}
```
   ![아이 동반 테이블](/assets/images/mclee/240126/아이_동반_테이블.JPG)

### 마무리

리뉴얼 작업이 진행되면서 웹리포트템플릿에서 부족했던 부분들이 보였습니다. 작업이 진행하는 도중 필요한 부분들을 추가 및 수정하고 다른 웹리포트에서도 사용할수 있도록 소스를 재작성하였습니다.
    
부족한 리뷰 글을 끝까지 읽어 주셔서 감사합니다.
