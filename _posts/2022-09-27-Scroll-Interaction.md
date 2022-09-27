---
layout: post
title: "Scroll interaction"
date: 2022-09-27 09:00:00 +0900
author: ymshim
categories: scroll interaction
published: true
---
<hr/>

# Scroll Interaction

<img src="/assets/images/ymshim/Apple.png" style="max-width: 200px; height: auto;" />

[Apple](https://apple.co.kr)의 사이트에는 획기적인 기술들이 적용되어 있습니다.

그 중 Scroll Interaction에 대해 실습 해보며 알게 된 것들을 공유하고자 합니다.

---

### Apple 사이트

[1.mp4](/assets/images/ymshim/1.mp4){:target="_blank"}

**위 영상처럼 스크롤에 따라 애니메이션이 실행됩니다.**

### 기본 개념

[https://www.youtube.com/watch?v=7BW4-IVO5Jc](https://www.youtube.com/watch?v=7BW4-IVO5Jc){:target="_blank"}

**다들 한번은 교과서에 심심할 때 그려봤던 경험이 있을 거라 생각이 됩니다!**

이 원리를 통해 사용자의 스크롤 양을 체크하여 연속되는 사진을 화면에 출력하여 애니메이션을 만들어 봅시다.

### 1. 이미지 그리기

애플이 아주 멋지게 만들어 놓은 이미지 소스를 사용합니다

[https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_00001.jpg](https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_00001.jpg){:target="_blank"}

url의 마지막 00001을 00002, 00003으로 바꿔보시면 애니메이션의 프레임 순으로 되어있는 것을 알 수 있습니다.

저 숫자를 동적으로 바꿔주기 위해 함수를 하나 만들어줍니다.

```jsx
const currentFrame = index => (
    `https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_${index.toString().padStart(5, '0')}.jpg`
)
```

index는 정수이므로 문자열로 변환 후 padStart(5, '0')파일 이름과 일치하는 5 자리 숫자에 도달 할 때까지 색인 앞에 0을 추가 하는 데 사용해야 합니다.

예를 들어 index에 1을 전달하면 00001 이 반환 됩니다.

**index.html**

[index.html](/assets/images/ymshim/index.html){:target="_blank"}

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        html {
            height: 100vh;
        }

        body {
            background: #000;
            height: 500vh;
        }

        #ipad-canvas {
            position: fixed;
            left: 50%;
            top: 50%;
            max-height: 100vh;
            max-width: 100vw;
            transform: translate(-50%, -50%);
        }
    </style>
</head>
<body>
<canvas id="ipad-canvas" width="1400" height="1400"></canvas>
<script src="apple.js"></script>
</body>
</html>
```

html의 높이는 100vh

body의 높이는 스크롤을 위해 500vh 로 만듭니다.

canvas 는 페이지의 정 중앙에 고정합니다.

**apple.js**

```jsx
const canvas = document.getElementById('ipad-canvas');
const context = canvas.getContext('2d');

const currentFrame = index => (
    `https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_${index.toString().padStart(5, '0')}.jpg`
)

const img = new Image();
img.src = currentFrame(0);

img.onload = function () {
    context.drawImage(img, 0, 0);
};
```

canvas를 2d로 설정해준 뒤 img 객체를 만들고 img 소스에 위에서 설명했던 함수에 매개변수로 0을 전달하여 00000번째 사진을 가져옵니다.

img가 생성된 뒤 img 객체를 캔버스의 왼쪽 0, 위 0의 위치에 그려줍니다.

### 2. 스크롤 값을 이용해 해당하는 프레임 구하기

이제부터 이 기술의 핵심이 나옵니다.

처음 접하면 살짝 헷갈릴 수 있지만 천천히 읽어보시면 당연한 얘기 입니다.

```jsx
const scrollTop = html.scrollTop;
```

html.scrollTop은 현재 스크롤의 위치를 반환합니다.

```jsx
const maxScrollTop = html.scrollHeight - window.innerHeight;
```

전체 스크롤 길이 - 내부 스크린 높이 = 최대 스크롤 값

```jsx
const scrollFraction = scrollTop / maxScrollTop;
```

현재 스크롤 위치 / 최대 스크롤 값 = 현재 스크롤 위치의 백분율

ex) 50 / 100 = 0.5 즉 50은 100의 백분율 50%

ex) 50 / 200 = 0.25 즉 50은 200의 백분율 25%

```jsx
const frame = Math.ceil(scrollFraction * frameCount)
```

백분율 * 프레임 수 = 전체 프레임 중 백분율에 해당하는 프레임

나온 값이 소수이기에 소수점 올림 하여 정수로 변환합니다.

ex) 총 100컷 * 0.5 ⇒ 50번째 컷

ex) 총 200컷 * 0.25 ⇒ 50번째 컷

```jsx
const frameIndex = Math.min(frameCount, frame);
```

const c = Math.min(a, b) a , b 중 작은 값을 c에 할당 하는 함수
전체 프레임 컷보다 커지면 안되기 때문입니다.

그렇게 추가하여

**apple.js**

```jsx
const html = document.documentElement;
const canvas = document.getElementById('ipad-canvas');
const context = canvas.getContext('2d');
const frameCount = 113;

const currentFrame = index => (
    `https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_${index.toString().padStart(5, '0')}.jpg`
)

const img = new Image();
img.src = currentFrame(0);

img.onload = function () {
    context.drawImage(img, 0, 0);
};

window.addEventListener('scroll', () => {
    const scrollTop = html.scrollTop; 
    const maxScrollTop = html.scrollHeight - window.innerHeight;
    const scrollFraction = scrollTop / maxScrollTop; 
    const frame = Math.ceil(frameCount * scrollFraction)
    const frameIndex = Math.min(frameCount, frame);
		//console.log(frameIndex)
})
```

### 3. 이미지 업데이트

이제 얻어 낸 프레임 인덱스를 이미지 소스에 넣어 업로드 해봅시다.

```jsx
    const updateImage = index => {
        img.src = currentFrame(index);
        context.drawImage(img, 0, 0);
    }

    requestAnimationFrame(() => updateImage(frameIndex))
```

이미지 소스를 동적으로 변경해주는 currentFrame() 함수에 구한 frameIndex를 넣고

canvas 0, 0 좌표에 그려줍니다. 

**완성된 자바스크립트 코드**

**apple.js**

```jsx
const html = document.documentElement;
const canvas = document.getElementById('ipad-canvas');
const context = canvas.getContext('2d');
const frameCount = 113;

const currentFrame = index => (
    `https://www.apple.com/105/media/us/ipad-air/2022/5abf2ff6-ee5b-4a99-849c-a127722124cc/anim/m1/image-sequence/large/large_${index.toString().padStart(5, '0')}.jpg`
)

const img = new Image();
img.src = currentFrame(0);

img.onload = function () {
    context.drawImage(img, 0, 0);
};

window.addEventListener('scroll', () => {
    const scrollTop = html.scrollTop; 
    const maxScrollTop = html.scrollHeight - window.innerHeight;
    const scrollFraction = scrollTop / maxScrollTop; 
    const frame = Math.ceil(frameCount * scrollFraction)
    const frameIndex = Math.min(frameCount, frame);

    const updateImage = index => {
        img.src = currentFrame(index);
        context.drawImage(img, 0, 0);
    }

    requestAnimationFrame(() => updateImage(frameIndex))
})
```

응용해서 만들어 본 페이지 (반응형 디자인 최적화 미완성)

[https://hprl.github.io/scroll-interaction-2/index.html](https://hprl.github.io/scroll-interaction-2/index.html){:target="_blank"}

참고 및 출처 페이지

[https://oddcode.tistory.com/155](https://oddcode.tistory.com/155){:target="_blank"}
