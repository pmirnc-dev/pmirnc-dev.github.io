---
layout: post
title: "MetaSurvey script 작업할때 주의할 점"
date: 2022-09-21 17:10:00 +0900
author: mclee
categories: javascript jquery
---
<hr/>

### 개요
$(document).on(’[이벤트]’, ‘[지정요소]’, function(){}) Meta에서 사용시 주의 할점
이전 문항의 스크립트와 현 문항 스크립트의 지정요소가 같을 경우 작동하는 이슈

### 분석
1. $(document). ready(function() {  }).on(’[이벤트]’, ‘[공통 지정 요소]’, function(){ });
   지정요소가 같을 경우
```javascript
//Q1 문항 스크립트
var node1 = "<span>Q1 Test</span>";
var node2 = "<br><span>Q1 Event</span>";
$(document).ready(function() {
    //console.log('ready', node1);
    $('div[question-name]').append(node1)
}).on('change', '.pure-radio', function() {
    $('div[question-name]').append(node2)
    //console.log('on', node1, 'Q1');
});
//Q2 문항 스크립트
var node1 = "<span>Q2 Test</span>";
var node2 = "<br><span>Q2 Event</span>";
$(document).ready(function() {
    //console.log('ready', node1);
    $('div[question-name]').append(node1)
}).on('change', '.pure-radio', function() {
    $('div[question-name]').append(node2)
    //console.log('on', node1, 'Q2');
});
```
- 해당 스크립트는 HTML 내에 공통되는 클래스명이 .pure-radio로 검색하여 이벤트가 작동되는 방식이며,
  이전 문항과 현 문항이 동일한 클래스명을 이벤트 지정 요소로 작성될 때 동일한 이벤트가
  중복 작동을 하게 됩니다.  
- Test URL : http://metasurvey.io/?ACCESS_KEY=d0ba8a19746649babd86  

2. $(document). ready(function() {  }).on(’[이벤트]’, ‘[고유 지정 요소]’, function(){ });
   각 문항별로 지정요소가 다를 경우  

```javascript
//Q1 문항 스크립트
var node1 = "<span>Q1 Test</span>";
var node2 = "<br><span>Q1 Event</span>";
$(document).ready(function() {
   //console.log('ready', node1);
   $('div[question-name]').append(node1)
}).on('change', 'div[question-name="Q1"] .pure-radio', function() {
   $('div[question-name]').append(node2)
   //console.log('on', node1, 'Q1');
});

//Q2 문항 스크립트
var node1 = "<span>Q2 Test</span>";
var node2 = "<br><span>Q2 Event</span>";
$(document).ready(function() {
   //console.log('ready', node1);
   $('div[question-name]').append(node1)
}).on('change', 'div[question-name="Q2"] .pure-radio', function() {
   $('div[question-name]').append(node2)
   //console.log('on', node1, 'Q2');
});
```
- 이벤트 고유 지정 요소를 div[question-name=’[문항번호]’]로 지정할 경우 위 스크립트 처럼 현 문항에서만 작동을 합니다.  
- Test URL : http://metasurvey.io/?ACCESS_KEY=21f366eedb6f4e7cb6d0

### 원인  
개발자 도구에서 $(document) 실행시 객체 내용중에 
jquery[난수] 캐싱 - events 확인할 수 있습니다.  
해당 events 객체 내에서 이벤트 데이터들이 점점 쌓여지는게 확인됩니다.  
![jquery_$(document)](/assets/images/mclee/20220921/mclee_document.jpg)  
MetaSurvey 설문 문항을 진행시 페이지 이동을 하지 않고 
현 페이지에서 문항을 완료시 다음 문항으로 불러오는 방식으로 진행이 됩니다.

### 결론
$(document)객체에 jquery[난수] 캐싱이 초기화 되지 않고 문항별로 들어올때 문항 스크립트쪽에 jquery 이벤트가 있을 경우  
해당 events 객체에 이벤트 데이터가 쌓이게 됩니다.
이벤트를 작동할 지정 요소를 특정한 요소로 지정하여 다른 이벤트가 작동하지 않도록 하는게 중요합니다.  

```javascript
//Q1 문항 스크립트
var node1 = "<span>Q1 Test</span>";
var node2 = "<br><span>Q1 Event</span>";
$('div[question-name=Q1] [지정요소]').on('click', function() {
    $('div[question-name=Q1]').append(node2)
})
;
//Q2 문항 스크립트
var node1 = "<span>Q2 Test</span>";
var node2 = "<br><span>Q2 Event</span>";
$('div[question-name=Q2] [지정요소]').on('click', function() {
    $('div[question-name=Q2]').append(node2)
})
;
```