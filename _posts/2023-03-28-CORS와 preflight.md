---
layout: post
title:  "CORS와 preflight"
date:   2023-03-28 13:59:07 +0900
author: tkkim
---
## CORS

![](/assets/images/tkkim/20230328/Untitled.png)

프론트 웹 개발 중 백엔드 서버에 보낸요청이 CORS 에러로 돌아왔던 경험을 겪었던적이 있을것이다.

CORS에러는 왜 발생하는 것인지 CORS가 정확히 무엇인지에 대해 오늘 알아보고자 한다. 


### CORS 란
**Cross-Origin Resource Sharing**

교차 출처 리소스 공유로, 다른 출처에 리소스 요청시 해당 리소스에 접근할수있도록 HTTP header에 추가 설정해 접근권한을 부여하고 브라우저에 알려주는 정책

![](/assets/images/tkkim/20230328/Untitled 1.png)

- **Cross-Origin** 정책을 지원하는 리소스
    - `<link>` 의 href 에서 타 사이트의 `.css 리소스`에 접근하는 것이 가능
    - `<img>` 의 src 에서 타 사이트의 `.png, .jpg 등의 리소스`에 접근하는 것이 가능
    - `<script>` 의 src 에서 타 사이트의 `.js 리소스`에 접근하는 것이 가능
        
        (type="module" 속성은 제외)
        

- **Same-Origin** 정책을 지원하는 리소스
    - 타 도메인 소스에 대해 자바스크립트 `ajax 요청 API 호출`시
    - 웹 폰트 CSS 파일 내 @font-face에서 `타 도메인의 폰트 사용` 시


→ 즉, Cross-Origin 정책 지원 리소스는 타 사이트의 리소스에 접근하는것이 가능한 반면에 Same-Origin 정책 지원 리소스는 타 사이트에 대한 접근 제한이 있기 때문에 CORS 를 사용해 접근가능하다.


- **origin**
    
    사이트 접속 시 인터넷 주소창에서 볼수있는 URL 
    
    port 부분 까지가 origin에 해당한다.

![](/assets/images/tkkim/20230328/Untitled 2.png)
    

## CORS 의 필요성

허용되지 않은 출처에서 보내는 요청들을 무작위로 받게될 경우 로그인 정보를 뺴내거나 무한 요청으로 인한 웹서버의 과도한 트래픽소모, 최종적으로 서버 먹통을 유발 시킬수있다.

→  CORS 정책을 통해 허용되지 않은 출처에서 보내는 요청을 브라우저 단에서 막아준다

## CORS 동작원리

1. 클라이언트에서 HTTP요청의 헤더에 Origin을 담아 전달
2. 서버는 응답헤더에 Access-Control-Allow-Origin(리소스 접근 허용 출처)을 담아 클라이언트에 전달
3. 클라이언트에서 Origin과 서버가 보내준Access-Control-Allow-Origin을 비교해 차단여부를 결정한다.

- **Simple Request -** 서버에 바로 요청
- **Preflight** - 요청 이전에 유효성 확인을 위한 예비요청

## **Preflight**

서버에서 바로 요청을 보내는 것이 아닌 유효성 확인을 위해 `OPTIONS`메소드를 사용해 예비 요청을 보내는 방식

→ 유효하지 않은 요청을 사전 차단 시켜 불필요한 리소스 낭비를 막아줄수있다.

- preflight request가 보내지는 조건
    - `GET`,`HEAD`,`POST` 요청
    - 경우에 따른 Contend-Type 값
        - `pplication/x-www-form-urlencoded`
        - `multipart/form-data`
        - `text/plain`
    - 요청에 사용된 `XMLHttpRequestUpload` 객체에 이벤트 리스너가 등록되어 있지 않을 경우
    - 요청에 `ReadableStream` 객체가 사용되지 않을경우


---

참조

[https://velog.io/@wjdwl002/CORS의-기본-개념과-동작-방식부제-Preflight-요청이란](https://velog.io/@wjdwl002/CORS%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EB%B0%A9%EC%8B%9D%EB%B6%80%EC%A0%9C-Preflight-%EC%9A%94%EC%B2%AD%EC%9D%B4%EB%9E%80)

[https://jeleedev.tistory.com/178](https://jeleedev.tistory.com/178)