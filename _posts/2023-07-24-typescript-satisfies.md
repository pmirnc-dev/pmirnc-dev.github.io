---
layout: post
title:  "typescript satisfies"
date:   2023-07-24 09:00:00 +0900
author: hsyoo
categories: typescript satisfies
---
<hr/>

typescript 4.9 버전에서 satisfies라는 새로운 키워드가 추가되었습니다.  

공식 문서를 바탕으로 설명하겠습니다.  

```typescript

// 각 프로퍼티는 문자열 또는 RGB 튜플일 수 있습니다.
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255]
//  ^^^^ sacrebleu - 오타를 냈습니다!
};
// 우리는 배열 메서드를 'red'에 사용하고 싶습니다...
const redComponent = palette.red.at(0);
// 혹은 'green'에 문자열 메서드를 사용하고 싶을 수 있습니다...
const greenNormalized = palette.green.toUpperCase();

```

위에서 bleu라는 오타를 냈습니다. palette에 타입을 표기해서 bleu 오타를 잡을 수도 있지만, 그렇게 되면 각 프로퍼티에 대한 정보를 잃게 됩니다.  

```typescript

type Colors = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];
const palette: Record<Colors, string | RGB> = {
    red: [255, 0, 0],
    green: "#00ff00",
    bleu: [0, 0, 255]
//  ~~~~ 이제 오타를 올바르게 감지했습니다.
};
// 하지만 여기서 원치 않는 문제가 발생했습니다. 'palette.red'가 문자열이 "될 수 있다"는것 입니다.
const redComponent = palette.red.at(0);

```

satisfies 연산자를 사용하면 표현식의 결과 타입을 변경하지 않고 표현식의 타입이 특정 타입과 일치하는지 검증할 수 있습니다. 

statisfies 를 사용할 경우 유니온 타입과 호환되는 타입의 메소드 호출 시도 시 정적 에러를 발생시키지 않습니다.   

blue의 오타를 발견함과 동시에 value의 메소드를 사용할 수 있습니다.


```typescript
type Colors = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];
const palette = {
    red: [255, 0, 0],
    green: "#00ff00",
    bleu: [0, 0, 255]
//  ~~~~ 오타가 잡혔습니다!
} satisfies Record<Colors, string | RGB>;
// 두 메서드 모두 여전히 접근할 수 있습니다!
const redComponent = palette.red.at(0);
const greenNormalized = palette.green.toUpperCase();
```

satisfies는 타입을 변경하지 않고 타입을 검증하는 것이기 때문에, palette의 타입은 여전히 Record\<Colors, string \| RGB\>입니다.

타입 검증에 큰 도움이 될 것 같네요.

<hr/>
참고문서

[https://www.typescriptlang.org/ko/docs/handbook/release-notes/typescript-4-9.html](https://www.typescriptlang.org/ko/docs/handbook/release-notes/typescript-4-9.html){:target="_blank"}  
[https://velog.io/@jay/TS-4.9-satisfies](https://velog.io/@jay/TS-4.9-satisfies){:target="_blank"}  
[https://www.zerocho.com/category/TypeScript/post/638c327844d418915ee64b0b](https://www.zerocho.com/category/TypeScript/post/638c327844d418915ee64b0b){:target="_blank"}


