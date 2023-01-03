---
layout: post
title:  "Typescript decorator"
date:   2023-01-03 09:00:00 +0900
author: hsyoo
categories: typescript decorator
---
<hr/>

## 잡설

2023년 계묘년이 밝았습니다.   
돌이켜보니 2022년 작년 한해 동안 많은 일들이 있었습니다  
새해에는 작년에 미처 달성하지 못했던 목표를 다시금 이룰 수 있는 한해가 되었으면 하고(개인적)  
부서 구성원들의 역량을 키우기 위해 더 시간을 투자하고 의미있게 만들도록 노력하겠습니다.


이만 각설하고   
## 올해 첫 포스팅은 typescript에서의 decorator에 대해 알아보겠습니다   
공식 문서에는 데코레이터에 대해 아래와 같이 설명합니다   

>TypeScript 및 ES6에 클래스가 도입됨에 따라, 클래스 및 클래스 멤버에 어노테이션을 달거나 수정하기 위해 추가 기능이 필요한 특정 시나리오가 있습니다.데코레이터는 클래스 선언과 멤버에 어노테이션과 메타-프로그래밍 구문을 추가할 수 있는 방법을 제공합니다. 데코레이터는 JavaScript에 대한 2단계 제안이며 TypeScript의 실험적 기능으로 이용 가능합니다.
> - [Typescript Decorator](https://www.typescriptlang.org/ko/docs/handbook/decorators.html){:target="_blank"}

데코레이터를 사용하기 위해선 tsconfig.json에 다음과 같이 옵션을 활성화해야 합니다
```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```

데코레이터는 다음에서 적용될 수 있습니다
- Class (클래스)
- Class Property (클래스 속성)
- Class Method (클래스 메서드)
- Class Accessor (클래스 접근자)
- Class Method Parameter (클래스 메서드 파라미터)

그리고 5가지 유형이 있습니다
- Class Decorators (클래스 데코레이터)
- Method Decorators (메서드 데코레이터)
- Accessor Decorators (접근자 데코레이터)
- Property Decorators (속성 데코레이터)
- Parameter Decorators (매개변수 데코레이터)


### [클래스 데코레이터 (Class Decorators)](https://www.typescriptlang.org/ko/docs/handbook/decorators.html#%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-class-decorators){:target="_blank"}
> 클래스 데코레이터는 클래스 선언 직전에 선언됩니다. 클래스 데코레이터는 클래스 생성자에 적용되며 클래스 정의를 관찰, 수정 또는 교체하는 데 사용할 수 있습니다. 클래스 데코레이터는 선언 파일이나 다른 주변 컨텍스트 (예: 선언 클래스)에서 사용할 수 없습니다.

공식 문서에 있는 예제를 한번 같이 살펴보겠습니다

{% include codepen.html hash="vYaKQRW" tab="js,result" title="class 생성자 데코레이터 예제" %}

위 예제에서는 `@reportableClassDecorator` 데코레이터를 사용하여 `BugReport` 클래스의 생성자를 재정의 했습니다. 
데코레이터는 클래스의 생성자를 반환해야 하며, 반환된 생성자는 클래스의 인스턴스를 생성할 때 호출됩니다.
reportingURL은 클래스의 생성자에 추가된 속성입니다.



### [메서드 데코레이터 (Method Decorators)](https://www.typescriptlang.org/ko/docs/handbook/decorators.html#%EB%A9%94%EC%84%9C%EB%93%9C-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-method-decorators){:target="_blank"}
> 메서드 데코레이터는 메서드 선언 직전에 선언됩니다. 데코레이터는 메서드의 프로퍼티 설명자(Property Descriptor) 에 적용되며 메서드 정의를 관찰, 수정 또는 대체하는 데 사용할 수 있습니다. 메서드 데코레이터는 선언 파일, 오버로드 또는 기타 주변 컨텍스트(예: 선언 클래스)에서 사용할 수 없습니다.

예제를 한번 살펴보겠습니다

{% include codepen.html hash="MWBezNa" tab="js,result" title="메서드 데코레이터 예제" %} 


`@enumerable(false)`데코레이터는 데코레이터 팩토리입니다. `@enumerable(false)` 데코레이터가 호출되면 프로퍼티 설명자의 enumerable 프로퍼티를 수정합니다.   
greet 메서드는 `@enumerable` 데코레이터에 true를 전달하여 데코레이터를 호출합니다.   
이는 greet 메서드가 열거 가능하다는 것을 의미합니다. (for ~ in루프문에서 확인 가능)   
반대로 bye 메서드는 `@enumerable` 데코레이터에 false를 전달해서 bye 메서드가 열거 되지 않습니다.




### [접근자 데코레이터 (Accessor Decorators)](https://www.typescriptlang.org/ko/docs/handbook/decorators.html#%EC%A0%91%EA%B7%BC%EC%9E%90-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-accessor-decorators){:target="_blank"}
> 접근자 데코레이터는 접근자 선언 바로 전에 선언됩니다. 접근자 데코레이터는 접근자의 프로퍼티 설명자에 적용되며 접근자의 정의를 관찰, 수정 또는 교체하는 데 사용할 수 있습니다. 접근자 데코레이터는 선언 파일이나 다른 주변 컨텍스트(예: 선언 클래스)에서 사용할 수 없습니다.


접근자 데코레이팅에는 특이점이 있는데 단일 멤버에 대한 get / set 접근자를 데코레이팅 할 수 없습니다.   
대신 멤버의 모든 데코레이터를 문서 순서대로 지정된 첫 번째 접근자에 적용해야 합니다.   
데코레이터는 각각의 선언이 아닌 get과 set 접근자를 결합한 프로퍼티 설명자에 적용되기 때문입니다.

위 특이점을 인지하고 예제를 다뤄보겠습니다.

{% include codepen.html hash="RwBREae" tab="js,result" title="접근자 데코레이터" %}

x 프로퍼티의 get 접근자에는 `@configurable(false)` 데코레이터가 적용되어 configurable이 false이고
y는 `@configurable(true)` 데코레이터가 적용되어 있는 게 확인이 됩니다.    



### [속성 데코레이터 (Property Decorators)](https://www.typescriptlang.org/ko/docs/handbook/decorators.html#%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-property-decorators){:target="_blank"}
> 프로퍼티 데코레이터는 프로퍼티 선언 바로 전에 선언됩니다. 프로퍼티 데코레이터는 선언 파일이나 다른 주변 컨텍스트(예: 선언 클래스)에서 사용할 수 없습니다.

프로퍼티 데코레이터의 표현 식은 런타임에 다음 두 개의 인수와 함께 함수로 호출됩니다   
- 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입
- 멤버의 이름

{% include codepen.html hash="abjZPgj" tab="js,result" title="프로퍼티 데코레이터" %}

@format("Hello, %s")가 호출되면 reflect-metadata 라이브러리의 Reflect.metadata 함수를 사용하여 프로퍼티에 대한 메타데이터 항목을 추가합니다.   
getFormat이 호출되면 형식의 메타데이터 값을 읽습니다.


### [매개변수 데코레이터 (Parameter Decorators)](https://www.typescriptlang.org/ko/docs/handbook/decorators.html#%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-parameter-decorators){:target="_blank"}
> 매개변수 데코레이터는 매개 변수 선언 직전에 선언됩니다. 매개변수 데코레이터는 클래스 생성자 또는 메서드 선언의 함수에 적용됩니다. 매개변수 데코레이터는 선언 파일, 오버로드 또는 다른 주변 컨텍스트(예: 선언 클래스)에서 사용할 수 없습니다.

매개 변수 데코레이터의 표현식은 런타임시 다음 세 개의 인수와 함께 함수로 호출됩니다.
- 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입
- 멤버의 이름
- 함수의 매개 변수 목록에 있는 매개 변수의 서수 색인(ordinal index)

{% include codepen.html hash="gOjMqbp" tab="js,result" title="매개변수 데코레이터" %}

<hr/>

## 결론

nestjs에서는 데코레이터를 사용하여 컨트롤러, 서비스, 모듈 등을 정의하는 만큼 데코레이터에 대한 이해가 필수적입니다.   
데코레이터는 자바스크립트의 기능이 아니라 타입스크립트의 고유 기능입니다.   
그만큼 타입스크립트는 be, fe에서 모두 필수적으로 사용되는 언어가 되었고, 데코레이터는 유용한 기능이라고 생각합니다.
오늘 예제에서 언급되었던 enumerable, configurable, writable는 다음 포스팅 주제인 __object defineProperty__에서 자세히 다루도록 하겠습니다. (지목하겠습니다)

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty){:target="_blank"}



참고 자료
- [Typescript Decorator](https://www.typescriptlang.org/ko/docs/handbook/decorators.html){:target="_blank"}
- [https://itchallenger.tistory.com/765](https://itchallenger.tistory.com/765){:target="_blank"}
