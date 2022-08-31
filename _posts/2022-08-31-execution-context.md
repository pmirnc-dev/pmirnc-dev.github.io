---
layout: post
title:  "Execution Context (Lexical Environment)"
date:   2022-08-31 09:00:00 +0900
author: sh.kim
categories: javascript execution-context lexical-environment
---
<hr/>

# Execution Context
Execution Context는 javascript가 실행되는 환경의 추상적인 개념으로 **scope, hoisting, closure, this** 등의 개념과 밀접하게 연관되어 있습니다.

---

# Execution Context 유형

### Global Execution Context
기본 context로 javascript에선 단 하나의 global context 만을 갖습니다.

어떤 함수 안에도 포함되지 않는 코드가 갖는 context 입니다.

global context에서는 전역(global 브라우저인 경우 window) 객체를 만들고, this의 값을 전역 객체로 바인딩합니다.

### Functional Execution Context
각 함수는 고유한 context를 가지며 함수가 호출될 때마다 생성됩니다.

global context와는 달리 여러개의 context가 존재할 수 있습니다. 

### Eval Function Execution Context
eval 함수 내에서 실행되는 코드의 context입니다. 

eval 함수는 사용을 권장하지 않기 때문에 넘어가겠습니다.

---

# Execution Stack (Call Stack)
call stack은 호출된 함수의 context 저장하여 실행 순서를 제어해주는 역할을 합니다.

javascript 엔진은 코드가 실행되면 global context를 생성하고 call stack에 푸시합니다.

그리고 함수가 호출될 때마다 functional context를 생성하고 call stack에 푸시합니다.

call stack의 최상단에 있는 context가 실행하고, 실행이 완료되면 해당 context를 제거합니다.

[예제]
```javascript
const a = 'a';

function fn1() {
  fn2();
}

function fn2() {
  console.log('fn2');
}

fn1();
```

#### 1. 코드가 실행되면 global context(anonymous)가 생성되고 call stack에 푸시됩니다.

![img.png](/assets/images/sh/20220831/call-stack/img.png)

#### 2. fn1 함수가 실행되고 fn1의 context가 call stack에 푸시됩니다.

![img_2.png](/assets/images/sh/20220831/call-stack/img_2.png)

#### 3. fn2 함수가 실행되고 fn2의 context가 call stack에 푸시됩니다.

![img_3.png](/assets/images/sh/20220831/call-stack/img_3.png)

#### 4. fn2 함수가 종료 되고 fn2의 context가 call stack에서 제거됩니다.

![img_4.png](/assets/images/sh/20220831/call-stack/img_4.png)

#### 5. fn1 함수가 종료 되고 fn1의 context가 call stack에서 제거됩니다.

![img_5.png](/assets/images/sh/20220831/call-stack/img_5.png)

#### 6. 모든 코드의 실행이 완료되면 global context를 call stack에서 제거하고 종료됩니다.

![img_6.png](/assets/images/sh/20220831/call-stack/img_6.png)

---

# Lexical Environment
Execution Context는 Lexical Environment와 Variable Environment 두 개의 컴포넌트를 가집니다. 

Lexical Environment는 자바스크립트 코드에서 변수나 함수 등의 식별자를 정의하는데 사용되며
함수, 블록문, catch 등의 구문을 만나면 해당 block 내부에서 새로운 Lexical Environment를 생성합니다.

Lexical Environment는 Environment Record와 outer라는 포인터를 구성요소로 갖고 있습니다.

Environment Record는 Lexical Environment 범위 내에서 선언된 식별자 정보를 저장하고

outer는 외부 Lexical Environment를 참조하는 포인터로, 스코프를 탐색을 하기 위해 사용됩니다.

[LE 예제]
```javascript
const a = 'a';
const b = 'b';

function fn1() {
  const c = 'c';
  
  function fn2() {
    const d = 'd';
  }
} 
```
위 코드의 Lexical Envrionment를 표현하면 다음과 같습니다.
```
globalLexicalEnvironment = {
  EnvironmentRecord: {
    a: 'a',
    b: 'b',
    fn1: <function>
  },
  outer: null,
}

fn1LexicalEnvironment = {
  EnvironmentRecord: {
    c: 'c',
    fn2:  <function>
  },
  outer: globalLexicalEnvironment,
}

fn2LexicalEnvironment = {
  EnvironmentRecord: {
    d: 'd',
  },
  outer: fn1LexicalEnvironment,
}
```

### Variable Environment
Variable Environment는 기본적으로 Lexical Environment와 구성요소가 같고,
초기에는 같은 Lexical Environment를 참조합니다.

이해하기 쉽게.. Variable Environment는 var 키워드로 생성된 변수들의 식별자 정보를 갖고 있고,
Lexical Environment는 그 외의 식별자 정보를 갖고 있다고 생각해 주시면 됩니다..

---
# 생성단계
Execution Context는 생성단계(Creation Phase)와 실행단계(Execution Phase) 두 단계를 거쳐 실행됩니다.

생성단계에선 Lexical Environment와 Variable Environment가 생성되고, 각각의 Environment는 Environment Record에 변수 및 함수 선언문의 식별자를 저장합니다.

[ER 예제]
```javascript
var varV = 'varV';  
let letV = 'letV';  
const constV = 'constV';  
const fn1 = function () {};  
function fn2 () {};
```
위와 같은 코드가 있을 때 Environment Record에 다음과 같이 식별자가 저장됩니다. 
```
LexicalEnvironment {
  EnvironmentRecord: {
    letV: <uninitialized>,
    constV: <uninitialized>,
    fn1: <uninitialized>,
    fn2: <function>,
  },
  outer: null,
},
VariableEnvironment: {
  EnvironmentRecord: {
    varV: undefined,
  },
  outer: null,
}
```
## hoisting
모든 선언문의 식별자는 생성 단계에서 Environment Record에 저장되며 값은 초기화 되지 않습니다. 예외적으로 var로 선언된 변수들만 Lexical Environment (Variable Environment)가 인스턴스화 될 때
undefined로 초기화 되는데 이것이 저희가 흔히 알고 있는 **hoisting**입니다.

> JavaScript에서 호이스팅(hoisting)이란, 인터프리터가 변수와 함수의 메모리 공간을 미리 할당하는 것을 의미합니다.

Environment Record에 선언문이 저장될 때 메모리를 할당하기 때문에 hoisting은 사실 모든 선언문을 대상으로 발생한다고 볼 수 있습니다. 

함수 선언식의 경우 함수 자체가 선언문이기 때문에 hoisting 되어 코드상의 선언문보다 먼저 호출할 수 있게 되는 것입니다. 

[hoisting 예제]
```javascript
// 함수 표현식은 변수 할당과 똑같습니다
// var로 선언된 함수(변수)는 hoisting 되며 undefined로 초기화 됩니다.
var fn1 = function () {
  console.log('in fn1');  
};
// const로 선언된 함수(변수)는 hoisting 되지만 참조할 수는 없습니다.
const fn2 = function () {
  console.log('in fn2');
};
// 함수 선언식으로 hoisting 되어 선언문 이전에 함수를 호출할 수 있습니다.
function fn3() {
  console.log('in fn3');
};
```

---

# Reference to the outer environment
Lexical Environment는 outer라는 외부 환경을 참조하는 포인터를 가지고 있습니다.

outer를 통해 외부에 선언된 변수에 접근이 가능한 것입니다.

[outer 예제1]
```javascript
const a = 'a';

function fn1() {
  const b = 'b'
  console.log('fn1 a: ', a);
  console.log('fn1 b: ', b);
  fn2();
}

function fn2() {
  console.log('fn2 a: ', a);
  console.log('fn2 b: ', b); // ReferenceError: b is not defined
}

fn1();
```
LexicalEnvironment
```
globalLexicalEnvironment: {
  EnvironmentRecord: {
    a: 'a',
    fn1: <function>,
    fn2: <function>,
 },
  outer: null, 
},
fn1LexicalEnvironment: {
  EnvironmentRecord: {
    b: 'b',
  },
  outer: globalLexicalEnvironment,
},
fn2LexicalEnvironment: {
  EnvironmentRecord: {
  },
  outer: globalLexicalEnvironment,
}
```
[outer 예제1]의 fn2함수 두 번째 log에서 변수 b에 대한 ReferenceError가 발생합니다.

fn2 함수에서 변수 b를 찾는 과정은 다음과 같습니다.

1. fn2 함수 자신의 Environment Record에서 변수 b에 대한 정보를 찾습니다.
2. Record에 변수 정보가 없으면 outer가 참조하고 있는 global(외부 Environment)로 이동합니다.
3. 다시 global의 Environment Record에서 변수 b에 대한 정보를 찾습니다.
4. 위의 내용을 반복하며 outer의 참조가 null(global)이면 에러가 발생합니다.

javascript outer는 선언 시점을 기준으로 결정되는데 이것을 **lexical scope**라고 합니다. 

[outer 예제2]
```javascript
const a = 'a';

function fn1() {
  function fn2() {
    console.log('fn2 a: ', a);
    console.log('fn2 b: ', b);
  }
  const b = 'b'
  console.log('fn1 a: ', a);
  console.log('fn1 b: ', b);
  fn2();
}

fn1();
```
LexicalEnvironment
```
globalLexicalEnvironment: {
  EnvironmentRecord: {
    a: 'a',
    fn1: <function>,
  },
  outer: null, 
},
fn1LexicalEnvironment: {
  EnvironmentRecord: {
    b: 'a',
    fn2: <function>,
  },
  outer: globalLexicalEnvironment,
},
fn2LexicalEnvironment: {
  EnvironmentRecord: {
  },
  outer: fn1LexicalEnvironment,
}
```
[outer 예제2]에서는 fn2함수를 fn1함수 안에 선언되었고 코드가 잘 실행되고, scope를 확인해 보면 closure(fn1)가 있는 것을 확인할 수 있습니다.

![img.png](/assets/images/sh/20220831/closure.png)

## closure
mdn에선 아래와 같이 closure를 정의하고 있습니다.
> 클로저는 함수와 함수가 선언된 어휘적 환경의 조합이다.

즉, closure는 함수와 해당 함수의 Lexical Environment라고 할 수 있습니다.

[closure 예제]
```javascript
function addMaker(num1) {
  return function (num2) {
    return num1 + num2;
  }
}

const add5 = addMaker(5);
const add10 = addMaker(10);
add5(5);
add10(5);
```
---
# This binding
this의 바인딩은 context와 함수 호출에 의해 결정됩니다.

global context에서 this는 global(browser인 경우 window)객체로 바인딩됩니다.

functional context에서 this는 객체 참조에 의해 호출되면 해당 객체로, 그렇지 않은 경우엔 global객체 또는 undefined(엄격모드)로 바인딩됩니다.  

**쉽게 말해서.. this는 객체의 메서드에서 사용되면 해당 객체로, 그 외의 경우에는 모두 global 객체로 바인딩된다고 볼 수 있습니다.**

[this 예제]
```javascript
const person = {
  name: 'sh',
  birthYear: 1999,
  calcAge() {
    console.log(2022 - this.birthYear);
  }
}

// 'this' refers to 'person', 
// because 'calcAge' was called with 'person' object reference 
person.calcAge();

// 'this' refers to the global window object, 
// because no object reference was given
const calculateAge = person.calcAge;
calculateAge();
```

**화살표 함수는 this를 바인딩하지 않습니다.**

화살표 함수는 자신의 Environment에 this를 갖고 있지 않습니다. 대신 자신의 outer가 참조하고 있는 Environment의 this를 참조합니다.

[this 예제2]
```javascript
const person = {
  name: 'sh',
  birthYear: 1999,
  calcAge() {
    const fn1 = function() {
      // this === global object
      console.log('fn1 this: ', this); 
    }
    const fn2 = () => {
      // this === person
      console.log('fn2 this: ', this); 
    }
    fn1();
    fn2();
  }
}
person.calcAge();
```

# 끝

---
# Reference
[https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0) <br/>
[https://dkje.github.io/2020/08/30/ExecutionContext/](https://dkje.github.io/2020/08/30/ExecutionContext/) <br/>
[https://www.youtube.com/watch?v=QtOF0uMBy7k](https://www.youtube.com/watch?v=QtOF0uMBy7k) <br/>
[https://www.youtube.com/watch?v=bwwaSwf7vkE](https://www.youtube.com/watch?v=bwwaSwf7vkE) <br/>
[https://262.ecma-international.org/6.0/#sec-execution-contexts](https://262.ecma-international.org/6.0/#sec-execution-contexts) <br/>
[https://meetup.toast.com/posts/123](https://meetup.toast.com/posts/123) <br/>
[https://meetup.toast.com/posts/129](https://meetup.toast.com/posts/129) <br/>
[https://developer.mozilla.org/ko/docs/Glossary/Hoisting](https://developer.mozilla.org/ko/docs/Glossary/Hoisting) <br/>
[https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures](https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures) <br/>
