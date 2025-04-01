---
layout: post
title:  "Swiper의 autoPlay 어떻게?"
date:   2025-03-26 09:00:00 +0900
author: ysson
categories: ["js", "react"]
published: true
---
<hr/>

#  Swiper AutoPlay 기능 및 Progress Bar 구현

## 1. 개요
pmi 게이트웨이의 공지사항을 보여주는 바를 보시면 현재 autoPlay기능으로 자동으로 다음 슬라이드를 보여주고 있습니다.
하지만 언제 슬라이드가 넘어갈지 몰라서 답답함을 경험해보신 적 한번씩 있으신가요?

[Nexon DF](https://df.nexon.com/) 넥슨의 공지사항 바를 살펴보면 이렇게 총 공지사항의 수와 다음 슬라이드로 언제쯤 이동되는지 progress bar를 통해 보여주고 있습니다. 이러한 작은 기능들이 사용자의 편리성을 더 높혀줄 것 같습니다.

그래서 이런 예시를 바탕으로 pmi 직원들의 편의성을 향상 시키기 위해 gateway의 기본적인 autoPlay 슬라이더에 progress bar를 만들고, 마우스 hover 시에 슬라이드를 정지/재개하는 기능을 구현해 보았습니다.

## 2. Swiper AutoPlay 살펴보기,  Event / Parameters
progress를 어떻게 구현하면 좋을까요?
슬라이드 전환 시간에 맞춰 진행 상태를 업데이트하려면 setInterval() 함수로 시간을 수동으로 관리해야 할 것 입니다.
여기에 마우스 이벤트에 따라 정지/재생하는 기능까지 추가한 예시코드를 보여드리겠습니다.
```text
  useEffect(() => {
    if (!isPaused) {
      intervalRef.current = setInterval(() => {
        setProgress(prevProgress => {
          if (prevProgress >= 100) {
            return 0;  // progress가 100%가 되면 초기화
          }
          return prevProgress + (100 / (slideDuration / 100));  // 매 10ms마다 progress 증가
        });
      }, 10);  // 10ms마다 업데이트
    } else {
      clearInterval(intervalRef.current);  // pause 상태에서는 진행 중인 interval 정지
    }
    
    return () => clearInterval(intervalRef.current);  // 컴포넌트 unmount 시 interval 정리
  }, [isPaused]);

  const handleMouseEnter = () => {
    setIsPaused(true);  // 마우스 올리면 멈춤
  };

  const handleMouseLeave = () => {
    setIsPaused(false);  // 마우스 벗어나면 재개
  };
```
이렇게만 봐도 복잡해 보이지 않습니까? 또한 결과물도 수동으로 progress를 업데이트 시키면 매끄럽게 진행되지 않고 끊김이 중간중간 있을 것 같고 슬라이더와 progress bar가 잘 동기화될 수 있을 지도 확실하지 않습니다.

Swiper에서는 초창기부터 진행 상태를 보여주는 progress 기능을 제공해 오고 있습니다. Swiper의 progress는  기본적으로 슬라이드가 진행되는 동안, 슬라이드의 이동 상태를 백분율로 추적하여 화면에 표시할 수 있도록 해주는 기능입니다. 이 기능을 사용하면 복잡한 로직없이 손쉽게 progress Bar를 만들 수 있습니다.

이제부터 Swiper의 progress기능을 이용해서 슬라이더의 progress Bar를 만들어보도록 하겠습니다.
밑에는 기본적인 AutoPlay의 API 명세서([swiper-api#autoplay](https://swiperjs.com/swiper-api#autoplay))를 통해 가져온 내용입니다.
#### - Autoplay Parameters

| Name                | Type    | Default | Description |
|---------------------|--------|---------|-------------|
| `delay`            | number | 3000    | 전환 사이의 지연 시간 (ms 단위). 이 매개변수를 지정하지 않으면 자동 재생이 비활성화됩니다. 각 슬라이드에 대해 `data-swiper-autoplay` 속성을 사용하여 다른 지연 시간을 지정할 수 있습니다. |
| `disableOnInteraction` | boolean | true  | `false`로 설정하면 사용자가 상호작용(스와이프)한 후에도 자동 재생이 비활성화되지 않습니다. 상호작용 후마다 자동 재생이 다시 시작됩니다. |
| `pauseOnMouseEnter` | boolean | false  | 1.  -   활성화되면 마우스 포인터가 Swiper 컨테이너에 들어갈 때 자동 재생이 일시정지됩니다. |
이 부분은 autoPlay를 설정하는 데에 쓰입니다.

#### - Autoplay Events

| Name               | Arguments                        | Description |
|--------------------|--------------------------------|-------------|
| `autoplay`        | (swiper)                       | Event will be fired when slide changed with autoplay. |
| `autoplayPause`   | (swiper)                       | Event will be fired on autoplay pause. |
| `autoplayResume`  | (swiper)                       | Event will be fired on autoplay resume. |
| `autoplayStart`   | (swiper)                       | Event will be fired when autoplay starts. |
| `autoplayStop`    | (swiper)                       | Event will be fired when autoplay stops. |
| `autoplayTimeLeft`| (swiper, timeLeft, progress) | Event triggers continuously while autoplay is enabled. It contains time left (in ms) before transition to next slide and percentage of that time related to autoplay delay. |
슬라이더의 상태를 추적해주는 이벤트를 통해서 여러가지 핸들러와 추가적인 설정을 해줄 수 있습니다.
그 중 **autoPlayTimeLeft** 와 같은 이벤트는 슬라이드 전환에 남은 시간을 추적하게 해주며 인자로 progress를 넘겨주고 있습니다. 이를 통해 진행률을 알 수 있습니다.
&nbsp;


### (1) 기본 AutoPlay 기능 구현
 ```text
<Swiper
      ref={swiperRef}  // Swiper 인스턴스를 swiperRef에 저장, 이 객체를 통해 각각의 Swiper 객체에 접근
      loop={true}  // 슬라이드를 반복하도록 설정
      autoplay={{  //autoplay 설정
        delay: 3000, // 간격
        disableOnInteraction: false,  // 사용자가 슬라이드 조작해도 AutoPlay 멈추지 않음
      }}
      modules={[Autoplay]}  // Autoplay 모듈 활성화
      //이벤트 설정
      onSlideChange={(swiper: SwiperCore) => {  // 슬라이드가 변경될 때
        setCurrentSlide(swiper.realIndex);  
      }}
      onBeforeInit={(swiper: SwiperCore) => (swiperRef.current = swiper)}  // 인스턴스가 초기화 되기 전에
      onAutoplayTimeLeft={handleAutoplayTimeLeft}  // Autoplay 남은 시간이 있을 때
    >
      ...
    </Swiper>
```
&nbsp;

### (2) Progress Bar 구현

- autoPlay의 이벤트인 onAutoplayTimeLeft의 콜백함수로 handleAutoplayTimeLeft를 만들어줍니다.
-  이 함수에서는 3개의 인자(arguments)로 swiper, timeLeft, progress를 넘겨받아 사용할 수 있습니다.
- handleAutoplayTimeLeft 함수의 역할은 현재 슬라이더의 현재 진행상태인 progress를 넘겨받아서 상태로 저장해 줍니다.
- 이때 인자로 받은 progress는 0부터 1사이의 값을 가지므로 진행률(%)로 변환하기 위해 * 100을 했습니다.
```
  const handleAutoplayTimeLeft = (swiper: SwiperCore, timeLeft: number, progress: number) => {
   //setProgress((progress) * 100); 
    setProgress((1 - progress) * 100); 
  };
```
- progress 의 상태를 보여주는 ui요소인 progress bar를 만들어 줍니다.
- progress 만큼 너비가 채워지면서(0부터 100까지) 사용자는 진행상태를 볼 수 있습니다.
```text
	<div>
      <div
        style={{ width: `${progress}%` }} 
      />
	</div>
```
&nbsp;

## 3. Swiper AutoPlay 살펴보기, Method

#### - Autoplay Method

| Name            | Description         |
|----------------|---------------------|
| `swiper.pause()`  | Pause autoplay (일시정지 ⏸️) |
| `swiper.resume()` | Resume autoplay (재개 ▶️) |
| `swiper.start()`  | Start autoplay (자동재생 시작🔼) |
| `swiper.stop()`   | Stop autoplay (자동재생 중지 ⏹️) |
&nbsp;

### (3) 마우스 Hover에 따라 슬라이드 정지/재생

- 먼저 Swiper를 전체를 감싸는 html dom 요소에 마우스 이벤트 핸들러를 추가해줍니다.
```text
        <div className={styles.sliderWrapper} 
             onMouseEnter={handleMouseEnter} 
             onMouseLeave={handleMouseLeave} > 
           <Swiper>
           ...
	       </Swiper>
	   </div>
```
- AutoPlay의 네 개의 Method 중에서 일시정지와 재개의 역할을 하는 pause(), resume()을 사용해서 마우스가 현재 swiper 요소에 들어왔을때는 일시정지, 요소를 벗어나면 재개되도록 하는 함수를 만들어줍니다.
```text
  const handleMouseEnter = () => {
    if (swiperRef.current) {
        swiperRef.current.autoplay.pause();
    }
  };

  const handleMouseLeave = () => {
      if (swiperRef.current) {
          swiperRef.current.autoplay.resume();
      }
  };
 ``` 
&nbsp;

## 4. start, stop vs pause, resume
여기서 왜 **start()**, **stop()** 이 아니라 **pause()** , **resume()** 을 썼는지에 궁금할 것 같습니다. 둘의 차이점을 보면 바로 이해될 것 같습니다. 저는 처음부터 nexon 홈페이지의 progress Bar처럼 슬라이더가 멈췄다가 진행되는 상태을 자연스럽게 보여주고 싶었습니다.
이때  **start()** 와 **stop()** 를 쓰면 이 두 함수는 단순 정지와 재생의 역할을 하기 때문에 슬라이더를 멈추면 초가 초기화되고 재생시키면 0부터 다시 시작하게 됩니다. 하지만 일시정지와 재개의 기능을 하는 **pause()** 와 **resume()** 를 사용하면 초가 초기화 되지 않고 자연스러운 진행상태를 보여줄 수 있습니다.
네 개의 함수의 역할을 잘 구분해서 적재적소에 쓰면 좋을 것 같습니다!
&nbsp;


## 5. 마치며
처음엔 api 명세를 읽지 않고 start(), stop()을 쓰며 pause(), resume() 같은 기능을 구현하려고 하다보니 setInterval이며 animate며.. 점점 코드가 산으로 갔습니다. 처음부터 명세를 읽었더라면 삽질하는 시간을 줄였을 것 같습니다. 다음부턴 api 명세를 보고 기능을 쓰도록 해야겠다는 생각을 했습니다!