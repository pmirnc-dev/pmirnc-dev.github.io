---
layout: post
title:  "Design Pattern - Singleton & Strategy"
date:   2023-04-06 09:00:00 +0900
author: shkim
categories: typescript design-pattern
---
<hr/>

## Design Pattern

> 소프트웨어 디자인 패턴(software design pattern)은 소프트웨어 공학의 소프트웨어 디자인에서 특정 문맥에서 공통적으로 발생하는 문제에 대해 재사용 가능한 해결책이다. 소스나 기계 코드로 바로 전환될수 있는 완성된 디자인은 아니며, 다른 상황에 맞게 사용될 수 있는 문제들을 해결하는데에 쓰이는 서술이나 템플릿이다. 디자인 패턴은 프로그래머가 어플리케이션이나 시스템을 디자인할 때 공통된 문제들을 해결하는데에 쓰이는 형식화 된 가장 좋은 관행이다. - 위키피디아

디자인 패턴에 대한 일반적인 정의는 위와 같습니다. 좀 더 쉽게 표현하면 **코드를 효율적으로 작성하는 방법론** 정도로 말할 수 있을 것 같습니다.

---

## Singleton

Singleton 패턴은 프로그램 내에서 클래스의 인스턴스가 하나만 생성되도록 제한하는 패턴입니다.<br/>
Singleton을 구현하는 방법은 간단합니다. <br/>
1. 클래스의 생성자를 private으로 선언하여 new 키워드를 이용해 인스턴스를 생성할 수 없도록 합니다.
2. 인스턴스를 생성하여 반환해 줄 static 멤버를 추가해 줍니다.

```typescript
class Singleton {
    static instance: Singleton;

    private constructor() {}

    public static getInstance(): Singleton {
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
        }
        return Singleton.instance;
    }
}

```
생성자를 private으로 선언했기 때문에 외부에서 new 키워드를 통해 인스턴스를 생성할 수 없습니다.

```typescript
const instance = new Singleton() // 에러
```

대신 static 메서드로 선언한 getInstance 메서드를 사용하여 인스턴스를 얻을 수 있습니다.<br/>
getInstance 메서드는 멤버 변수 instance이 값이 존재하지 않는 경우 새 인스턴스를 생성하여 변수에 할당합니다.<br/>  
이런 방법으로 새 인스턴스의 생성을 제한하고 항상 같은 인스턴스를 반환해 주도록 합니다.

```typescript
const instance: Singleton = Singleton.getInstance();
```

Singleton 패턴은 다음과 같은 상황에서 사용할 수 있습니다.

1. 리소스 사용을 제한하려는 경우
2. 로깅
3. 캐싱

Singleton 패턴은 분명 유용한 점이 있지만 구조적인 문제로 인해 사용하는 것을 지양해야 한다는 의견도 있습니다.
해당 내용은 여기에서 다루진 않고 궁금하신 분들은 검색을 해보시는 것을 추천드립니다.

---

## Strategy

Strategy 패턴은 알고리즘 계열을 정의하고 각 알고리즘을 캡슐화하며 상호 교환할 수 있게 만들어 주는 패턴입니다.<br/>
Strategy 패턴을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있습니다.


특정 사이트에서 데이터를 스크랩하기 위한 Crawler 클래스가 있다고 가정해 보겠습니다.<br/>

```typescript
class Crawler {
    dataScrape(site: string) {
        if (site === 'naver') {
            console.log('naver')
        } else if (site === 'google') {
            console.log('google')
        } else if (site === 'daum') {
            console.log('daum')
        }
    }
}

// execute code
(() => {
    const crawler = new Crawler();
    crawler.dataScrape('naver');
})();
```

위 코드에서 각 사이트 별로 데이터를 스크랩하기 위해 Crawler 클래스의 dataScrape 메서드를 사용합니다.<br/>
dataScrape 메서드는 사이트 정보를 site 파라미터로 전달받아 if/else 문을 돌면서 해당하는 로직을 찾아 실행합니다.<br/>
각 사이트 별 스크랩 로직이 복잡하고 길어지는 경우 가독성이 나빠지며 코드를 수정하기도 좋지 않습니다.<br/>
각 사이트별로 다른 팀원이 스크랩 로직을 작성한 뒤 하나로 합쳐야 하는 경우도 있을 수 있으며<br/>
새로운 사이트가 추가될 때마다 dataScrape 메서드를 수정해야 합니다.<br/>
이는 유지 보수 측면에서도 바람직하지 못하며 이를 해결하기 위해서 Strategy 패턴을 사용할 수 있습니다.

```typescript
interface Scraper {
    scraping(): void;
}

class NaverScraper implements Scraper {
    scraping(): void {
        console.log('naver');
    }
}

class GoogleScraper implements Scraper {
    scraping(): void {
        console.log('google');
    }
}

class Crawler {
    dataScrape(scraper: Scraper) {
        return scraper.scraping();
    }
}

// execute code
(() => {
    const crawler = new Crawler();
    crawler.dataScrape(new NaverScraper());
})();
```
Strategy 패턴을 활용하여 재구성한 코드입니다.<br/>
우선 스크랩 기능을 수행할 메서드를 interface를 사용하여 추상화했습니다.<br/>
각 사이트별 스크랩 로직은 Scraper 인터페이스를 구현하는 클래스를 생성하여 scraping 메서드를 오버라이딩하여 구현합니다.<br/>
다음으로 Crawler 클래스의 dataScrape 메서드에서는 파라미터로 Scraper 인스턴스를 받도록 수정하고 인스턴스의 scraping 메서드를 호출합니다.<br/>


Strategy 패턴은 다음과 같은 상황에서 사용할 수 있습니다.
1. 위와 같이 if/else 또는 switch 분기가 많은 경우
2. 유사한 기능을 하지만 구체적인 구현 방식이 다른 코드

Strategy 패턴을 사용하면 코드의 가독성이 증가하고 테스트가 용이해진다는 장점이 있습니다.

## 끝

---

## Reference

[https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)<br/>
[https://blog.bitsrc.io/the-singleton-pattern-in-typescript-b906303fda93](https://blog.bitsrc.io/the-singleton-pattern-in-typescript-b906303fda93)<br/>
[https://codeburst.io/implementing-strategy-pattern-on-typescript-b74c447da37b](https://codeburst.io/implementing-strategy-pattern-on-typescript-b74c447da37b)<br/>
[https://medium.com/angular-in-depth/level-up-your-components-api-using-the-strategy-pattern-61762583a7d8](https://medium.com/angular-in-depth/level-up-your-components-api-using-the-strategy-pattern-61762583a7d8)<br/>

