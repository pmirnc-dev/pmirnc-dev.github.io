---
layout: post
title:  "Do you know Closure?"
date:   2024-02-15 09:00:00 +0900
author: ihhwang
categories: ['javascript', 'closure']
---
<hr/>

안녕하세요. 

You Don’t Know JS Yet 책 리뷰를 위해 읽고 있다가 클로저라는 단어는 많이 듣긴 했는데

실제로 어떤 개념인지 잘 와닿지 않아서 스터디 주제로 선정 했습니다.

# 클로저란?

> 폐쇄, 마감 by 파파고

사전적 의미로는 폐쇄, 마감을 뜻 한다.

javascript 에서는 중첩 함수가 있을 때 내부 함수가 외부 함수의 컨텍스트에 접근할 수 있고

함수가 렉시컬 환경 밖에서 실행될 때에도 이 스코프에 접근할 수 있게 해주는 기능 입니다.

javascript 에서 클로저는 함수가 생성이 될 때 마다 같이 생성이 됩니다.

## 스코프란?

혹시라도 모르는 사람이 있을 수 있기 때문에 설명하자면

자신이 선언된 위치에 의해 다른 코드가 식별자 자신을 참조할 수 있는 유효범위가 결정되는데

이 유효 범위가 스코프 입니다.

더 풀어서 말하면 변수나 함수가 선언 된 위치에 따라 전역 혹은 지역 변수로 되는 것 입니다.


    var IM_GLOBAL = 'global value'
    
    function regionFunction() {
        var IM_REGION = 'region value'
    }

위 예시를 보면 IM_GLOBAL 은 전역 변수이고 regionFunction() 내부에 있는 IM_REGION 은 지역변수 입니다.

ES6 이전에는 함수 스코프와 전역 스코프 두 가지만 존재했습니다.

    if (Math.random() > 0.5) {
        var x = 1;
    } else {
        var x = 2;
    }
    console.log(x); // 블록 외부에서 중괄호 내부에 있는 변수 접근 가능

ES6 이후 let과 const 가 등장 하면서 블록 외부에서 접근할 수 없습니다.

    if (Math.random() > 0.5) {
        const x = 1;
    } else {
        const x = 2;
    }
    console.log(x); // 참조 에러: x가 정의되어 있지 않습니다.


## 렉시컬 스코프란?

중첩된 함수 그룹 내부에서 내부 함수가 상위 함수의 변수에 접근할 수 있음을 의미 합니다.

함수의 선언 위치에 따라 상위 스코프가 결정됩니다.

ex1 - 글로벌 변수와 함수 호출

    var x = 1 // global
    
    function fnA() {
        var x = 10
        fnB()
    }
    
    function fnB() {
        console.log(x)
    }
    
    fnA() // 1
    fnB() // 1
    console.dir(fnA)
    console.dir(fnB)

javascript를 처음 보는 사람이라면 fnA() 와 fnB() 는 10을 출력할 것으로 예상하지만 결과는 1, 1 이 출력됩니다.

![closure01.png](/assets/images/ihhwang/closure/closure01.png)

fnA() 와 fnB() 의 scope는 global scope 만 갖고 있습니다.

예제2 - 중첩함수

    var x = 1 // global
    
    function fnA() {
        var x = 10
        function fnB() {
            console.log(x)
        }
        fnB()
        console.dir(fnB)
    }
    
    fnA()
    // fnB()
    console.dir(fnA)

![closure02.png](/assets/images/ihhwang/closure/closure02.png)

fnA() 는 global scope 만 갖고 있고, fnB() 는 closure 와 global 을 갖고 있습니다.

예제3

    function makeFunc() {
        const name = "Mozilla";
        function displayName() {
            console.log(name);
        }
        return displayName;
    }
    
    const myFunc = makeFunc();
    myFunc();

다음 예제는 myFunc 변수에 makeFunc() 을 할당했습니다.

이 코드를 보고 “Mozilla” 라는 결과가 나올 것을 예측하지만 

그냥 봤을때는 makeFunc() 을 할당 한 시점에서 displayName() 은 이미 return 됐기 때문에 실행이 되지 않아야 합니다. 

하지만 렉시컬 스코프를 통해 makeFunc 를 기억하고 있으므로 메서드를 호출 할 수 있습니다.

# 클로저를 사용해야 하는 이유

개발을 하면서 클로저를 알게 모르게 새용해왔었다.

그렇다면 궁극적으로 클로저를 사용해야 하는 이유는 다음과 같다.

* 함수 인스턴스가 매번 계산할 필요 없이 이전에 정보를 기억해내어 함수의 효율성을 높입니다.
* 함수 인스턴스 안에 변수를 캡슐화 하여 코드 가독성을 개선하고 스코프 노출을 제한할 수 있습니다.

예제 - 정보은닉

    function factory_movie(title){
        return {
            get_title : function (){
                return title;
            },
            set_title : function(_title){
                title = _title
            }
        }
    }

    ghost = factory_movie('Ghost in the shell');
    matrix = factory_movie('Matrix');
    
    console.log(ghost.get_title()); // Ghost in the shell
    console.log(matrix.get_title()); // Matrix
    
    ghost.set_title('공각기동대');
    
    console.log(ghost.get_title()); // 공각기동대
    console.log(matrix.get_title()); // Matrix

![closure03.png](/assets/images/ihhwang/closure/closure03.png)

다음과 같이 title에 직접 값을 할당하여 값을 변경해 주는 것이 아닌 set_title 을 통해 title의 값을 변경하고 

get_title 로 title 의 값을 불러올 수 있습니다.

    function makeCounter() {
        let count = 0;
        
        return function() {
            return count++;
        };
    }

    let counter = makeCounter();
    let counter2 = makeCounter();
    
    counter();
    counter2();

이런 방법으로도 활용 할 수 있습니다.
https://jsfiddle.net/hotae160/

우리가 javascript 를 사용하여 개발했을 때 알게 모르게 사용했던 부분들이 있었는데 

알고 있는것과 모르고 있는 것은 큰 차이가 있다고 생각합니다.

저도 완벽하게 파악한 것은 아니지만 어느정도 이해는 할 수 있었습니다.

끝!!

![그럼이만](/assets/images/ihhwang/img.gif)

---
# 참조

* You Don’t Know JS Yet
* https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures
* https://adjh54.tistory.com/64
* https://ko.javascript.info/closure#ref-1091
* https://poiemaweb.com/js-closure