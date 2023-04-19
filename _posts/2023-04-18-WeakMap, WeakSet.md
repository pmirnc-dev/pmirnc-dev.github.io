---
layout: post
title:  "WeakMap, WeakSet"
date:   2023-04-18 09:00:00 +0900
author: skan
categories: javascript Data Structures
---
<hr/>

# WeakMap, WeakSet
#

WeakMap을 알려면 Map에 대해 어느정도 알고 있어야 합니다. 간단하게 Map을 설명하겠습니다.

## Map

Map 객체는 Object 와 동일하게 키-값 쌍을 저장하며 삽입 순서도 기억하는 콜렉션 입니다. 어떤 값이라도 key, value 로 설정할 수 있습니다.
Object와 달리 넣은 순서가 기억되고, 키가 문자열이 아니어도 됩니다.

Map에는 기본적으로 제공하는 속성과 메소드가 있습니다.
```typescript
var map = new Map([['zero', 'ZeroCho']]);
map.set('hero', 'Hero');
map.get('zero'); // 'ZeroCho'
map.size; // 2
map.has('hero'); // true
map.has('nero'); // false
map.entries(); // {['zero', 'ZeroCho'], ['hero', 'Hero']}
map.keys(); // {'zero', 'hero'}
map.values(); // {'ZeroCho', 'Hero'}
map.delete('hero');
map.clear();
```

## WeakMap
그럼 WeakMap은 무엇이냐? 키가 객체인 Map이라고 보면 됩니다.

map과 차이점<br> 
위크맵은 맵과 다르게 반드시 객체 타입의 키만을 허용한다. 
그리고 key를 순회할 수 없다는 것 입니다.
```typescript
let weakMap = new WeakMap();
let obj = {};

weakMap.set(obj, "ok");
console.log(weakMap);         // WeakMap { <items unknown> }

weakMap.set("test", "error"); // TypeError: Invalid value used as weak map key
```

자바스크립트에선 가비지 컬렉션을 통해 더 이상 사용되지 않는 객체들을 메모리에서 지워나간다.
이 때, 자료구조에 속한 객체들은 가비지 컬렉션의 대상에서 제외되는데, WeakMap과 WeakSet을 이용하면 약간 연결 관계를 지속할 수 있다.
```typescript
let john = { name: "John", age: 30 };
let weakMap = new WeakMap();
weakMap.set(john, "something");

john = null;
```


위와 같이 위크맵에 객체를 키로 키-값 요소를 넣은 후 해당 키를 null 처리하면, 해당 객체는 가비지 컬렉션의 대상이 된다. john 이 가리키는 객체는 오로지 위크맵의 키로만 사용되므로, 참조를 덮어쓰게 되면 이 객체는 위크맵과 메모리에서 자동으로 삭제된다.

위크맵은 아래 네 개의 메서드만 지원한다.
 - weakMap.get(key)
 - weakMap.set(key, value)
 - weakMap.delete(key)
 - weakMap.has(key)

적은 메서드를 제공하는 이유는 가비지 컬렉션의 동작 방식 때문이다. 객체는 모든 참조를 잃게 되면 자동으로 가비지 컬렉션의 대상이 되는데, 동작 시점을 정확히 알 수는 없다.

가비지 컬렉션이 일어나는 시점은 자바스크립트 엔진이 결정한다. 객체는 모든 참조를 잃었을 때, 그 즉시 메모리에서 삭제될 수도 있고, 다른 삭제 작업이 있을 때까지 대기하다가 한 번에 같이 삭제될 수도 있다. 위크맵은 키가 객체로 설정되기에, 그 안에 요소가 몇 개 있는지 정확히 파악하는 것 자체가 불가능하다.

## 언제 사용하나?

- 데이터를 추가할 때

```typescript
let visitsCountMap = new Map();

function countUser(user) {
    let count = visitsCountMap.get(user) || 0;
    visitsCountMap.set(user, count + 1);
}

let john = { name: "John" };

countUser(john);

john = null;
```

예를 들어, 유저의 정보가 담긴 객체를 키로 Map에 방문 기록 횟수를 저장한다고 하자. 위와 같이 john 이라는 유저가 삭제될 상황에서 단순히 null 처리를 해 메모리에서 객체가 제거되었다고 생각할 수 있지만, 해당 객체를 키로 갖는 맵의 요소가 있기에 제거되지 않는다.

유저 수가 많아지면 제거된 유저 정보들이 불필요하게 차지하고 있는 메모리를 정리하지 못하게 된다. 이런 문제를 위크맵을 사용해 사전에 예방할 수 있다.

- 캐싱 작업에서

```typescript
let cache = new Map();

function process(obj) {
    if (!cache.has(obj)) {
        let result = obj.val * obj.val;

        cache.set(obj, result);
    }

    return cache.get(obj);
}

let obj = { val: 0 };

let result1 = process(obj);
let result2 = process(obj);

obj = null;

console.log(cache.size); // 1
```
위 예제는 맵을 캐시로 사용하는 간단한 코드다. 간단히 하나의 키-값 쌍만 사용한다고 가정했을 때, 함수 process()는 캐시 내부에 연산한 기록이 있으면 가져다 쓰고, 없다면 제곱한 결과를 저장한다.

위 코드도 마찬가지로, 객체 참조를 끊는다 해도 가비지 컬렉션의 대상이 되지 않고 캐시가 계속 쌓이게 될 거다. 이런 경우에도 위크맵을 사용하면 메모리를 효율적으로 사용할 수 있을 것이다.


## WeakSet


객체만을 키로 갖는 위크맵과 유사하게, 위크셋은 값으로 객체만을 저장할 수 있다. <br>위크셋 안의 객체는 도달 가능할 때만 메모리에서 유지된다. 즉, 위크셋 안의 객체 참조를 끊으면 가비지 컬렉션의 대상이 되어 추후에 제거될 것이다.
또, 위크맵과 마찬가지로 반복 작업이 불가능하다. 이런 불편함을 감수하고도, 객체를 기준으로 데이터를 추가하는 작업에 잔재되는 객체 데이터들을 지우는데 용이해 사용한다고 한다.

Set 객체와의 차이점은 아래와 같습니다.

 - Set 과 다르게 WeakSet 은 객체의 집합이며, 객체만 저장할 수 있습니다. 특정 type 의 값을 저장할 수 없습니다.
 - 약한 참조를 가집니다. WeakSet 내에 저장된 객체에대한 참조가 없어지면, 가비지 컬렉션 (GC) 의 대상이 되어 수거됩니다.
 - 내부를 순회할 수 없습니다.

### WeakSet 메소드 
 - weakSet.add(value)
 - weakSet.delete(value)
 - weakSet.has(value)

```typescript
    const ws = new WeakSet()
    const obj = {}
    const str = 'cat'

    ws.add(obj)
    ws.add(str) // Error :: Invalid value used in weak set

    ws.has(obj)    // true
    ws.delete(obj)
    ws.has(obj)    // false
```



## 끝

---

## Reference

[https://ddb8036631.github.io/modernjavascripttutorial/27.-WeakMap,-WeakSet/](https://ddb8036631.github.io/modernjavascripttutorial/27.-WeakMap,-WeakSet/)<br/>
[https://velog.io/@2weeks0/JavaScript-WeakMap%EA%B3%BC-WeakSet](https://velog.io/@2weeks0/JavaScript-WeakMap%EA%B3%BC-WeakSet)<br/>


