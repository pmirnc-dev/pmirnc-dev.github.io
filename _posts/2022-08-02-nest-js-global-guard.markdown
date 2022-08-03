---
layout: post
title:  "nestjs 전역 guard와 guard 예외처리(metadata and reflection)"
date:   2022-08-02 10:00:00 +0900
author: sh.kim
categories: nestjs
---

## 인트로

---
nestjs에서 route에 대한 접근 권한을 제어할 때 guard를 사용하는 경우가 있다

> [NestJs Guard](https://docs.nestjs.com/guards)

지금까지 nestjs를 사용한 프로젝트의 경우에도 항상 guard를 사용하여 권한을 제어했었다

![img.png](/assets/images/sh/20220802/guard-1.png)

이런식으로...

하나의 route마다 @useGuards를 일일이 걸어주었는데

저렇게 한 이유는 일부 route에서 guard를 사용하지 않기 위해서였는데

여기에선 저부분을 다른 방식으로 해결해 보려고 한다

## 컨트롤러 & 전역 Guard

---

여러 개의 route에 guard를 한 번에 걸어주는 방법은 아주 간단하다

* 컨트롤러

![img.png](/assets/images/sh/20220802/guard-2.png)

* 전역

![img.png](/assets/images/sh/20220802/guard-3.png)

컨트롤러에 guard를 적용한 경우 해당 컨트롤러에 포함되는 모든 route에 guard가 적용되고

전역으로 guard를 적용한 경우에는 모든 route에 guard가 적용된다

## 문제점...

---

위에서처럼 한 번에 guard를 걸어줄 수도 있지만

문제는 guard가 필요하지 않은 route들도 있다는 것이다

이걸 가장 간단하게 해결한 방법이 지금까지 했던 것처럼 route 하나하나에 guard를 걸어주는 것이고 

또 다른 방법은 metadata와 reflection을 이용하여 특정 route만 제외될 수 있도록 하는 것이다

## Reflection & metadata

---

> [NestJs metadata & reflection](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata])

metadata와 reflection에 대한 설명은 nestjs 공식 문서에 잘 나와있다

대략적으로 설명하면 특정 class(controller)나 route에 추가적인 정보(metadata)를 설정하고<br/>
이를 사용할 수 있게 해주는 것(reflection)이라고 보면 될 것 같다

### metadata
metadata는 nestjs에서 제공하는 SetMetadata라는 decorator를 사용하여 설정한다.

SetMetadata의 첫 번째 인자는 '키' 두 번째 인자는 '값'으로 metadata를 설정할 수 있다.

* directly

![img.png](/assets/images/sh/20220802/meta-1.png)

* custom decorator (set)

![img.png](/assets/images/sh/20220802/meta-2.png)

* custom decorator (use)

![img.png](/assets/images/sh/20220802/meta-3.png)

위의 이미지는 공식 문서에서 setMetadata를 사용하는 방법을 보여주는 예시이다

첫 번째 이미지는 직접 SetMetadata를 사용하는 것이고

나머지 이미지는 decorator를 만들어서(두 번째) 사용하는 것(세 번째)을 보여주는 것이다

위처럼 사용했을 경우 키가 'roles'이고 값이 '[admin]'인 metadata가 생성된다

### reflection

reflection은 nestjs에서 제공하는 reflector 객체를 사용한다

![img.png](/assets/images/sh/20220802/img_3.png)

reflector 객체는 4개의 메서드를 가지고 있는데 자세한 설명은 공식 문서에서 확인하고

여기서는 우선 get 메서드에 대해서만 설명하겠다

![img.png](/assets/images/sh/20220802/img_4.png)

get 메서드는 2개의 인자를 받는다

첫 번째 인자는 metadata를 설정할 때 사용했던 '키'

두 번째 인자는 '타겟' 인데 설명을 보면 context(decorated object) 이렇게 나와있다

즉 metadata를 설정한 class 혹은 route의 context를 넣어주면 된다 

**class면 context.getClass()**

**route면 context.getHandler()**

아래는 예시

* setMetadata

![img.png](/assets/images/sh/20220802/img.png)

* get

![img.png](/assets/images/sh/20220802/img_1.png)

* result

![img.png](/assets/images/sh/20220802/img_2.png)

대충 이해가 될 거라고 생각... 한다

## Guard 제어

---

위에서 살펴본 metadata와 reflection을 이용하여 guard를 제어해 보자

![img.png](/assets/images/sh/20220802/img_5.png)

우선 전역은 아니지만 ProjectControlle에 ProjectGuard를 걸어주었고

이제 ProjectController의 모든 route는 ProjectGuard를 거쳐가게 된다

그중에서 위에 있는 route에만 @SetMetadata를 이용하여 noGuard = true 값을 주었다

![img.png](/assets/images/sh/20220802/img_6.png)

ProjectGuard 안에서 reflector를 사용하여 noGuard 값을 가져오고

해당 값이 true 면 무조건 return true를

아닌 경우에는 기존의 검증 절차를 수행하여 true나 false 값을 넘겨주면 된다

## Custom Decorator

--- 

위에서 진행한 방법대로 그냥 SetMetadata decorator를 사용해도 되지만

custom decorator를 만들어서 SetMetadata를 좀 더 간결하게 사용할 수도 있다

* decorator(set)

![img.png](/assets/images/sh/20220802/img_7.png)

* decorator(use)

![img.png](/assets/images/sh/20220802/img_8.png)

## Reference

[https://docs.nestjs.com/]()


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
