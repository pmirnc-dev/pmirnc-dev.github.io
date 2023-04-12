---
layout: post
title:  "Javascript Proxy객체"
date:   2023-04-12 12:10:00 +0900
author: yrbyeon
categories: proxy
---
<hr/>



# Javascript의 Proxy객체

### 

![](/assets/images/yrbyeon/proxyObject.png)

## 1. Javascript Proxy 객체

### 프록시?

`Proxy`는 특정 객체를 감싸 프로퍼티 읽기, 쓰기와 같은 객체에 가해지는 작업을 **중간에서 가로채는 객체**로, 가로채진 작업은 `Proxy`자체에서 처리되기도 하고, 원래 객체가 처리하도록 그대로 전달되기도 한다.

`Proxy`객체는 본래의 객체 대신 사용할 수 있는 개체를 만들고 이 객체의 속성 가져오기, 설정 및 정의와 같은 기본 객체 작업을 재정의 할 수 있다.

### 프록시 객체의 사용 예시

`Proxy` 객체는 다양한 용도로 사용될 수 있다. 예를 들어, 객체의 프로퍼티에 대한 접근을 제한하거나, 객체의 프로퍼티에 접근할 때마다 로깅을 수행하거나, 객체에 대한 검증을 수행할 수 있다. 이를 통해 소프트웨어 개발에서 보안성과 유지보수성을 향상시킬 수 있다.

- `target`: 프록시할 원본 객체. 기본 동작을 가로챌(즉, 감싸게 될) 객체. 함수를 포함한 모든 객체 가능.
- `handler`: 가로채는 작업과 가로챈 작업을 재정의하는 객체. 동작을 가로채는 메서드인 ‘trap’을 가지고 있는 객체. 여기에서 프록시 설정.

`target` 과 `handler` 두개의 매개변수를 사용하여 `proxy` 를 생성한다. 

`handler` 로 가로챌 수 있는 동작은 다음과 같다.

| 핸들러 메서드 | 작동 시점 |
| --- | --- |
| get | 프로퍼티를 읽을 때 |
| set | 프로퍼티에 값을 쓸 때 |
| has | in연산자가 작동할 때 |
| deleteProperty | delete연산자가 작동할 때 |
| apply | 함수를 호출할때 |
| constructor | new연산자가 작동할 때 |
| getPrototypeOf | Object.getPrototypeOf |
| setPrototypeOf | Object.setPrototypeOf |
| isExtensible | Object.isExtensible |
| proeventExtensions | Object.preventExtensions |
| getOwnPropertyDescriptor | Object.getOwnPropertyDescriptor |
| ownKeys | Object.getOwnPropertyNames
Object.getOwnPropertySymbols |

`proxy`에 작업이 가해지고, `handler`에 작업과 상응하는 트랩이 있으면 트랩이 실행되어 `proxy` 가 이 작업을 처리할 기회를 얻게 된다. 트랩이 없으면 `target`에 작업이 직접 수행된다.

### 

## 2. 예제 코드

### (1) handler가 없는 proxy

```jsx
const target= {};
const proxy = new Proxy(target, {}) //핸들러(트랩)가 없는 proxy 객체 정의

proxy.test = 10; // 1. proxy에 새로운 프로퍼티 'test'추가
console.log(proxy.test) //2. proxy의 test 프로퍼티를 읽을 수 있다. -> 10 출력
console.log(target.test) //3. 원래 객체인 target의 test프로퍼티에도 10이 저장된다. -> 10 출력
```

`handler` 가 없는 `Proxy` 는 그냥 `target`객체를 감사는 투명한 래퍼가 된다고 생각하면 된다. 중간에 가로채는 트랩이 없으니 `target`의 기본 메서드를 사용하며 `target` 처럼 작동한다.

### (2) Proxy: get

```jsx
const target= {};
const proxy = new Proxy(target, {
	get(target, prop, receiver){
      console.log(target,prop, receiver)
    }
})

proxy.test = 5;
console.log(proxy.test) //출력: { test: 5 } test { test: 5 }
```

get 은 속성을 조회할 때 호출되는 함수이다. **닷노테이션(.)** 으로 속성을 조회할때 이를 가로채서 추가작업을 할 수 있다. get메서드의 파라미터로 받은 인자들로 새로운 동작을 정의하면 된다. get메서드는 기본적으로 다음 세가지 인자를 받는다.

- `target` : new Proxy 의 첫번째 인자로 주었던 해당 `target` 객체.
- `prop` : 조회한 property(속성).
- `receiver` : getter가 호출 될 때 this. 기본적으로는 객체 자신을 호출하지만 상속받은 객체일 경우에는 상속한 객체가 this가 된다.

```jsx
const target= {};
const proxy = new Proxy(target, {
	get(target, prop){
      return `${prop}을 조회 :::>> ${target[prop]} ~!`;
    }
})

proxy.test = 5;
console.log(proxy.test) // test을 조회 :::>> 5 ~!
```

`handler` 가 추가됐기 때문에 get 동작을 했을 때 원래 값인 5이 출력되는 것이 아니라 get handler가 가로채서 'test을 조회 :::>> 5 ~!'이 출력된 것이다.

<aside>
💡 타깃 객체의 위치와 상관없이 프락시 객체는 타깃 객체를 덮어써야만 한다. 객체를 프락시로 감싼 이후엔 절대로 타깃 객체를 참조하는 코드가 없어야 한다.

</aside>

### (3) Proxy: set

```jsx
const target= {};
const proxy = new Proxy(target, {
	set(target, prop, value, receiver){
      console.log(target,prop, value, receiver)
    }
})

proxy.test = 5; // 출력: {} test 5 {}
```

set은 속성에 값을 할당할때 호출되는 함수이다. 위의 예제를 보면 test 속성에 값을 할당할떄 이를 가로채서 실행되고 있다. 또한 위의 두가지 예시인 **get**과 **set**의 예제를 보면 가장 놀라운 부분은 `Proxy`에는 아무 속성이 없는 빈객체로 설정하였으나 이상태에서 새로운 속성을 할당해도 이를 캐치하고 수행한다는 점이다.

set메서드는 기본적으로 다음 네가지 인자를 받는다

- `target` : new Proxy의 첫번째 인자로 주었던 target 객체
- `prop` : 작성한 property 이름
- `val` : 작성한 property 값
- `receiver` : get 트랩과 유사하게 동작하는 객체. setter 프로퍼티에만 관여.

또한 set에서는 유효성 검사를 진행하는 식의 활용도 생각할 수 있다.

```jsx
const target= {};
const proxy = new Proxy(target, {
	set(target, prop, value, receiver){
      if (typeof value !== 'string') {
          console.log('ERROR:문자열을 입력해주세요');
          return false;
        }
      if (value.length<4) {
        console.log('ERROR:4자 이상의 문자열을 입력해주세요');  
        return false;
      }
      console.log(`${prop}에 '${value}' 저장`)
      return true;
    }
})

proxy.test = 5;  // ERROR:문자열을 입력해주세요
proxy.test = 'abc'; //ERROR:4자 이상의 문자열을 입력해주세요
proxy.test = '안녕하세요'; //test에 '안녕하세요' 저장
```

target의 test는 4자 이상의 문자열만 받아야한다고 가정한다. test에 값이 할당 될 때마다 `handler`가 실행되며 유효성 검사를 실시하게 된다.

```jsx
let numbers = [];

numbers = new Proxy(numbers, { // (*)
  set(target, prop, val) { // 프로퍼티에 값을 쓰는 동작을 가로챈다.
    if (typeof val == 'number') {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  }
});

numbers.push(1); // 추가성공.
numbers.push(2); // 추가성공.
alert("Length is: " + numbers.length); // 2

numbers.push("test"); // Error: 'set' on proxy -> 문자열은 입력 불

alert("윗줄에서 에러가 발생했기 때문에 이 줄은 절대 실행되지 않습니다.");
```

주목 할 점은 배열 관련 기능들은 여전히 사용할 수 있다는 것이다. `push`를 사용해 배열에 새로운 요소를 추가하고 `length` 프로퍼티는 이를 잘 반영하고 있다는 것을 통해 이를 확인할 수 있다. `Proxy` 를 사용해도 기존에 있던 기능은 절대로 손상되지 않는다.

`push`나 `unshift` 같이 배열에 값을 추가해주는 메서드들은 내부에서 `[[Set]]`을 사용하고 있기 때문에 메서드를 오버라이드 하지 않아도 프락시가 동작을 가로채고 값을 검증해준다.

<aside>
💡 `set` 트랩을 사용할 땐 값을 쓰는 게 성공했을 때 반드시 `true`를 반환해줘야 한다.

`true`를 반환하지 않았거나 false한 값을 반환하게 되면 `TypeError`가 발생한다.

</aside>

### (4) Proxy: deleteProperty

deleteProperty는 객체 속성을 삭제할때 호출되는 함수이다. 이를 통해 삭제하면 안되는 속성을 체크해서 삭제를 못하게 하는등의 활용이 가능하다. 

```jsx
const proxy= new Proxy({}, {
  deleteProperty(target, prop) {
    const isName = prop === 'name';
   
    if(isName)
      delete target[prop];
    
    return true;
  }
});

proxy.name = 'Yoora';
proxy.test = 5;

delete proxy.name;
delete proxy.test ; // name 속성이 아니기때문에 삭제되지 않는다.
console.log(proxy); // {test: 5}
```

### (5) Proxy: defineProperty

defineProperty는 **객체에 직접 새로운 속성을 정의하거나 이미 존재하는 속성을 수정한 후 그 객체를 반환**할 때 실행된다. defineProperty() 메소드를 사용하면 enumerable, configurable 등의 속성을 상세하게 조절할 수 있고, 값을 getter나 setter를 사용하여 다른 객체나 변수에서 읽어올 수도 있다.

```jsx
const person = new Proxy({}, {
     defineProperty(target, prop, descriptor) {
      console.log('called: ' + prop);
      console.log(target);
      return true;
    }
  });

Object.defineProperty(person, 'weight', {weight: 70, gender: 'man'});
```

### (6) Proxy: ownKeys , getOwnPropertyDescriptor

`Object.keys`, `for..in` 반복문을 비롯한 프로퍼티 순환 관련 메서드 대다수는 내부 메서드`[[OwnPropertyKeys]]`(트랩 메서드는 `ownKeys`임)를 사용해 프로퍼티 목록을 얻는다.

- `Object.getOwnPropertyNames(obj)` – 심볼형이 아닌 키만 반환.
- `Object.getOwnPropertySymbols(obj)` – 심볼형 키만 반환.
- `Object.keys/values()` – `enumerable` 플래그가 `true`이면서 심볼형이 아닌 키나 심볼형이 아닌 키에 해당하는 값 전체를 반환.
- `for..in` 반복문 – `enumerable` 플래그가 `true`인 심볼형이 아닌 키, 프로토타입 키를 순회.

```jsx
let user = {
  name: "Jenny",
  age: 20,
  _password: "***"
};

user = new Proxy(user, {
  ownKeys(target) {
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
});

// "ownKeys" 트랩은 _password를 건너뛴다.
for(let key in user) console.log(key); // name, age

// 아래 두 메서드에도 동일한 로직이 적용된다.
console.log( Object.keys(user) ); // name,age
console.log( Object.values(user) ); // John,30
```

`ownKeys` 트랩을 사용해 `_` 로 시작하는 프로퍼티는 `for..in`  반복문의 순환 대상에서 제외하도록 해보았다. `ownKeys` 를 사용했기 때문에 `Object.keys` 와 `Object.values` 에도 동일한 로직이 적용되는 것을 확인할 수 있다.

```jsx
let user = { };

user = new Proxy(user, {
  ownKeys(target) {
    return ['a', 'b', 'c'];
  }
});

console.log( Object.keys(user) ); // <빈 문자열>
```

그런데 객체 내에 존재하지 않는 키를 반환하려고 하면 `Object.keys` 는 이 키를 제대로 보여주지 않는다. `Object.keys` 는 `enumerable`  플래그가 있는 프로퍼티만 반환하기 때문이다. 이를 확인하기 위해 `Object.keys` 는 내부 메서드인 `[[GetOwnProperty]]` 를 호출해 모든 프로퍼티의 설명자를 확인한다.

```jsx
let user = { };

user = new Proxy(user, {
  ownKeys(target) { // 프로퍼티 리스트를 얻을 때 딱 한 번 호출.
    return ['a', 'b', 'c'];
  },

  getOwnPropertyDescriptor(target, prop) { // 모든 프로퍼티를 대상으로 호출.
    return {
      enumerable: true,
      configurable: true
      /* 이 외의 플래그도 반환할 수 있다. "value:..."도 가능. */
    };
  }

});

console.log( Object.keys(user) ); // a, b, c
```

`Object.keys`  호출 시 프로퍼티를 반환하게 하려면 `enumerable`  플래그를 붙여줘 프로퍼티가 객체에 존재하도록 하거나 `[[GetOwnProperty]]` 가 호출될 때 이를 중간에서 가로채서 설명자  `enumerable: true` 를 반환하게 해주면 되는데 `getOwnPropertyDescriptor`  트랩이 바로 이때 사용된다.

### (7) 여러 트랩을 사용하여 Property 보호하기

`_`(밑줄)이 앞에 붙은 프로퍼티나 메서드를 내부용으로만 쓰도록 하는 컨벤션은 널리 사용되고 있는 컨벤션 중 하나이며 `_`이 앞에 붙으면 객체 바깥에선 이 프로퍼티에 접근해선 안 되지만 기술적으로는 가능하다.

```jsx
let user = {
  name: "Honey",
  _password: "비밀"
};

console.log(user._password); // 비밀
```

`Proxy` 의 get, set, deleteProperty, ownKeys 트랩을 사용하여 `_` 로 시작하는 프로퍼티에 접근하지 못하도록 막는 기능을 구현할 수 있다.

```jsx
let user = {
  name: "Honey",
  _password: "***"
};

user = new Proxy(user, {
  get(target, prop) {
    if (prop.startsWith('_')) {
      throw new Error("접근이 제한되어있습니다.");
    }
    let value = target[prop];
    return (typeof value === 'function') ? value.bind(target) : value;
  },
  set(target, prop, val) { // 프로퍼티 쓰기.
    if (prop.startsWith('_')) {
      throw new Error("접근이 제한되어있습니다.");
    } else {
      target[prop] = val;
      return true;
    }
  },
  deleteProperty(target, prop) { // 프로퍼티 삭제.
    if (prop.startsWith('_')) {
      throw new Error("접근이 제한되어있습니다.");
    } else {
      delete target[prop];
      return true;
    }
  },
  ownKeys(target) { // 프로퍼티 순회.
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
});

// "get" 트랩.
try {
  alert(user._password); // Error: 접근이 제한되어있습니다.
} catch(e) { console.log(e.message); }

// "set" 트랩.
try {
  user._password = "test"; // Error: 접근이 제한되어있습니다.
} catch(e) { console.log(e.message); }

// "deleteProperty" 트랩.
try {
  delete user._password; // Error: 접근이 제한되어있습니다.
} catch(e) { console.log(e.message); }

// "ownKeys" 트랩이 순회 대상에서 _password를 제외.
for(let key in user) console.log(key); // name
```

get 트랩에서 return시에는 함수인지 여부를 확인하여 `value.bind(target)` 를 호출 한다. `user.checkPassword()` 같은 객체 메서드가 `_password` 에 접근할 수 있도록 해주는 것이다. 

```jsx
checkPassword(value) {
  // checkPassword(비밀번호 확인)는 _password를 읽을 수 있어야 한다.
  return value === this._password;
}
```

`user.checkPassword()`를 호출하면 점 앞의 객체가 `this`가 되므로 `Proxy` 로감싼 `user`에 접근하게 되는데, `this._password`는 `get` 트랩(프로퍼티를 읽으려고 하면 동작함)을 활성화하므로 에러가 던져진다. 리턴할 때 객체 메서드의 컨텍스트를 원본 객체인 `target`에 바인딩시켜준 이유는 `checkPassword()`를 호출할 땐 언제든 트랩 없이 `target`이 `this`가 되게 하기 위해서이다.

이 방법은 대부분 잘 작동하긴 하는데 메서드가 어딘가에서 `Proxy` 로 감싸지 않은 객체를 넘기게 되면 엉망진창이 되어버리기 때문에 이상적인 방법은 아니다. 기존 객체와 `Proxy` 로 감싼 객체가 어디에 있는지 파악할 수 없기 때문이다. 

한 객체를 여러 번 `Proxy` 로 감쌀 경우 각 `Proxy` 마다 객체에 가하는 '수정’이 다를 수 있다는 점 또한 문제이다. `Proxy` 로 감싸지 않은 객체를 메서드에 넘기는 경우처럼 예상치 않은 결과가 나타날 수 있다.

따라서 이런 형태의 `Proxy` 는 어디서든 사용해선 안 된다.

### (8) Proxy: has

```jsx
let range = {
  start: 1,
  end: 10
};

range = new Proxy(range, {
  has(target, prop) {
    return prop >= target.start && prop <= target.end;
  }
});

console.log(5 in range); // true
console.log(50 in range); // false
```

범위를 담고 있는 객체를 in연산사를 사용하여 특정 숫자가 range내에 있는지 확인하려 할 때, has 트랩은 in 호출을 가로챈다.

- `target` – `new Proxy`의 첫 번째 인자로 전달되는 타깃 객체
- `property` – 프로퍼티 이름

### (9) Proxy: apply

함수 역시 `Proxy`로 감쌀 수 있다.

`apply(target, thisArg, args)` 트랩은 `Proxy`를 함수처럼 호출하려고 할 때 동작한다.

- `target` – 타깃 객체(자바스크립트에서 함수는 객체임)
- `thisArg` – `this`의 값
- `args` – 인수 목록

다음은 `Proxy` 를 사용하지 않고 함수를 기반으로 `delay(f, ms)`  데코레이터(decorator)를 구현한 것이다.

```jsx
function delay(f, ms) {
  // 지정한 시간이 흐른 다음에 f 호출을 전달해주는 래퍼 함수를 반환.
  return function() { 
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(user) {
  console.log(`Hello, ${user}!`);
}

//console.log(sayHi.length); // 1 (함수 정의부에서 명시한 인수의 개수)

// 래퍼 함수로 감싼 다음에 sayHi를 호출하면 3초 후 함수가 호출.
sayHi = delay(sayHi, 3000);

sayHi("Anna"); // Hello, Anna! (3초 후)
//console.log(sayHi.length); // 0 (래퍼 함수 정의부엔 인수가 없음)
```

그런데 래퍼 함수는 프로퍼티 읽기/쓰기 등의 연산은 전달해주지 못한다. 래퍼 함수로 감싸고 난 다음엔 기존 함수의 프로퍼티(`name` , `length`  등) 정보가 사라진다. 

`Proxy` 객체는 타깃 객체에 모든 것을 전달해주므로 훨씬 강력하다. 래퍼 함수 대신 `Proxy`를 사용하여 구현해보자.

```jsx
function delay(f, ms) {
  return new Proxy(f, {
    apply(target, thisArg, args) {
      setTimeout(() => target.apply(thisArg, args), ms);
    }
  });
}

function sayHi(user) {
  console.log(`Hello, ${user}!`);
}

sayHi = delay(sayHi, 3000);

console.log(sayHi.length); // 1 (*) 프락시는 "get length" 연산까지 타깃 객체에 전달.

sayHi("Anna"); // Hello, Anna! (3초 후)
```

결과는 같지만 이번엔 호출뿐만 아니라 프락시에 가하는 모든 연산이 원본 함수에 전달된 것을 확인할 수 있다. 원본 함수를 프락시로 감싼 이후엔 `sayHi.length` 가 제대로 된 결과를 반환하고 있는 것을 확인할 수 있다.