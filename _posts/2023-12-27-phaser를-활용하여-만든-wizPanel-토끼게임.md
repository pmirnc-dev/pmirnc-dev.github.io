---
layout: post
title:  "phaser를 활용하여 만든 wizPanel 토끼게임"
date:   2023-12-26 09:00:00 +0900
author: ybcho
categories: ['phaser3']
---
<hr/>
<!--나중에 통합-->
<!--인호 과장님-->

# 황인호

**역할**

* 메인 게임 개발

## 이슈 & 해결
Phaser 를 잘 알지 못할때는 진짜 간단한 가위바위보나 카드 뒤집기 정도 수준만 개발을 해봤는데 실제 상용화 되는 게임을 만들어야 하니 걱정이 됐습니다.

개발 시작단계에는 용빈 주임이 많이 바빠서 상대적으로 시간이 좀 있었던 제가 게임 파트를 만들고 인트로, 랭킹, 통신 같은 후반 작업을 용빈 주임에게 맡기기로 했습니다.

### 개발 순서

다음과 같이 핵심 기능부터 스택을 쌓듯이 차례대로 개발을 했습니다.

    우주선 → 토끼, 쥐 → 폭탄 → 점수 & 폭탄 이벤트 → 시계 & 시계 이벤트 → 점수 & 시간 UI 작업 → Game Over → 폭탄 판정 범위


### 우주선(구 바구니)

이 부분은 그렇게 어렵진 않았고 키보드로 움직이게 해야 하나 Drag로 해야 하나 고민했는데 

모바일에서도 사용한다고 하여 Drag 방식으로 변경 했습니다.

### Object Physics Group

labs: [https://labs.phaser.io/edit.html?src=src\\physics\\arcade\\group vs group.js](https://labs.phaser.io/edit.html?src=src%5C%5Cphysics%5C%5Carcade%5C%5Cgroup%20vs%20group.js)

랜덤한 Object Group 을 상단에서 하단으로 떨어트려야 하는데 어떻게 해야 할지 몰라서 Phaser 사이트를 다 뒤져보기 시작했습니다. 

Object에 물리 그룹을 만들어 좌표, 개수, 가속도 등등 설정을 할 수 있다는걸 알게 되었고 위에서 아래로 떨어트리는건 성공 했습니다.

### overlap

labs: [https://labs.phaser.io/edit.html?src=src\\physics\\arcade\\sprite overlap group.js](https://labs.phaser.io/edit.html?src=src%5C%5Cphysics%5C%5Carcade%5C%5Csprite%20overlap%20group.js)

게임에서 설정한 Object들이 서로 부딪치면 특정 이벤트를 줄 수 있는 기능 입니다.

우주선과 오브젝트 그룹이 닿게 되면 사라지는 것 까진 설정 했는데 화면을 넘어가면 object가 그대로 남아있어서 메모리가 상승하기 시작했습니다.

**문제 해결 시도**

1. 화면 밖을 벗어나면 object를 파괴 → 방법을 찾아보려 했으나 뭔가 잘 안됐습니다.
2. 일정 시간이 지나면 스스로 파괴 → 그렇기엔 메모리 상승을 조금이라도 줄이고 싶었습니다.

**꼼수**

전에 Unity를 잠깐 했을 때 화면 바깥에 보이지 않는 벽을 세워 object가 밖으로 나가지 못하게 했던 것이 생각이 났습니다. 

그래서 여기도 화면 제일 하단에 보이지 않는 투명한 얇은 바닥을 두고 그곳에 닿으면 파괴되도록 해서 개발 시간을 약간 아낄 수 있었습니다.

![rabbit-02](/assets/images/ihhwang/rabbit-02.png)

### 시계 획득시 제한시간 증가

이 부분이 진짜 어려웠던 것 같습니다.

단순히 생각해서 제한시간 30초 가 있고 시계를 획득하면 5초가 증가한다는 개념은 머리로 이해가 가는데

실제로 코드가 적용 되지 않아 정말 머리를 쥐어뜯었던 것 같습니다.

제한시간과 Time Event는 따로 흘러간다는 것을 뒤늦게 알았습니다.

**문제**

시계를 획득하면 제한시간 + 5초를 더하니까 제한시간은 늘어나는데 게임 시간은 30초 고정으로 흘러갔습니다.

**해결**

현재 남은 시간 + 5초를 더하여 Time Event를 다시 리셋을 하니 성공 했습니다.
**main.scene.ts**

```jsx
// 우주선과 시계가 overlap 될 때
this.physics.add.overlap(
  this.basket,
  this.clockGroup,
  (basket, clock) => {
    this.registry.set("clock");
    this.clockGetSound.play();
    const currentTime = this.countdown.getSecond().toString();
    this.countdown.addSecond(
      currentTime,
      5000,
      this.handleCountDownFinished.bind(this)
    );
    clock.destroy();
  }
);
```

**countdown.controller.ts**

```jsx
function start(callback) { // 본 게임은 30초
  this.stop();
	// 최초 시간 등록
  this.timerEvent =  this.scene.time.addEvent({
    delay: this.duration,
    callback: () => {
      this.label.text = ''

      this.stop()

      if (callback) {
        callback()
      }
    }
  })
}

function addSecond(currentTime: string, time: number, callback: any) {
  const remainTime = (Number(currentTime) * 1000) + time;
  this.duration = remainTime
  // 시간 재설정
  this.timerEvent.reset({
    delay: remainTime,
    callback: () => {
      this.label.text = ''

      this.stop()

      if (callback) {
        callback()
      }
    }
  })
}
```

## UI 부분

먼저 Phaser 는 Scene 이라는 개념이 있습니다.

한글로 해석하면 장면이고 사용자에게 보여 줄 화면을 나타냅니다.

`**scene.start(’key’)**` 를 통해 실행시키고자 하는 Scene Class 를 실행할 수도 있고

**pause**, **destroy** 같이 Scene 상태를 다양하게 제어할 수 있습니다.

항상 화면에 떠 있어야 하는 **Top-Bar** 와 게임 시작 시에만 나타나는 **UI scene**을 구현하는게 헷갈렸었습니다.

Vue.js 에서는 다음과 같이 Navigation 영역, Main 영역을 쉽게 나눌 수 있지만

Phaser는 영역 구분이 잘 안되고 반드시 Scene 위에 다른 Scene 을 쌓아야 합니다.

```html
<template>
	<v-nav>
		<NavigationView />
	<v-nav>
	<v-main>
		<router-view />
	</v-main>
</template>
```

(아니면 제가 못 찾은 것 일수도…)

다음 화면과 같이 Main Scene 위에 TopBar + UI 를 겹친 화면 입니다.

Vue.js 로 표현하면 자식 컴포넌트 같은 개념입니다.

![rabbit-01](/assets/images/ihhwang/rabbit-01.png)

**UI Scnee**

처음엔 Main 과 UI 를 같이 합칠까 하다가 코드가 너무 길어져서 가독성이 떨어질 것 같아 분리 했습니다.

**Top-Bar Scene**

이 코드는 어떻게 해줄 지 몰라서 Scene 마다 같이 실행되게 코드를 모두 넣었습니다.

뭔가 Floating Button 같은 전체적으로 통제하는 방법이 있을 법 한데 찾지 못해서 이렇게 했습니다😥

`**this.scene.launch("top-bar");` // Scene 동시 실행**

## Time Event

javascript에서 사용하는 setTimeout 이나 setInterval 같은 시간 관련 함수 입니다.

물론 setTimeout 이나 setInterval 를 사용해도 되지만 가끔씩 잘 적용되지 않는 경우가 있어

Phaser 에서 제공하는 Time Evnet를 사용하는 것이 이벤트를 제어하기도 쉽고 더 좋은 것 같습니다.

```tsx
// setTimeout과 동일한 기능을 하는 time event
// 3초 카운트다운 후 핵심 게임 로직들 실행
this.time.delayedCall(3000, () => {
  // 핵심 게임 로직들
});
```

## Event & Registry

런칭 막바지 쯤, 용빈 주임님이 재도전을 하면 할수록 시간이 너무 빨리 줄어드는 것 같다고 하셔서 확인을 해보니

첫 판은 괜찮은데 재도전을 할 수록 시간이 점점 빨리 줄어드는게 보였습니다.

무슨 문제인진 모르겠지만 시간 관련 이벤트가 중복으로 호출된다는 느낌을 받았습니다.

로그를 확인해보니 제 생각이 맞았었습니다.

**registry** 라는 vue로 치면 store 같이 전역으로 사용할 수 있는 데이터 매니저가 있습니다.

main 과 uiscene 에서 시간, 점수들을 공유해야 했기 때문에 registry 를 사용했습니다.

Scene이 종료되면 자동으로 사라지는 줄 알았는데 재도전을 하면 한번 더 등록하기 때문에

이벤트가 중첩으로 호출 되고 있었습니다.

registry에 event를 등록한 후, 해제하지 않으면 계속 남아있는 줄도 모르고 썼다가 큰일 날 뻔 했습니다.

화면 전환 혹은 게임 종료 시 수동으로 종료를 해줘야 한다는 것도 알게 되었습니다.

**ui.scene.ts**

```jsx
this.registry.events.on('changedata', this.updateScore, this); // changedata 이벤트 등록
```

**main.scene.ts**

```tsx
// 게임 종료시 모든 object, physics, time, event, sound 종료
function handleCountDownFinished() {
  // pause physics event
  this.physics.pause();
  // remove Time event
  this.time.removeAllEvents();
  // 바구니 drag off
  this.input.off("drag");

  // event off
  this.events.off('addScore')
  this.events.off('subScore')
  this.events.off('resetScore')
  this.registry.events.off('changedata');

  this.clockSound.stop();
  this.clockGetSound.stop();
  this.countdown.stop();

  this.scene.stop();
  this.scene.run("game_over", { score: this.score });
}
```

## 느낀점

정말 스스로 많이 부족함을 느꼈습니다.

그러나 개발을 하면 할수록 막연하게 생각했던 객체 지향에 대해 드디어 깨닫기 시작한 것 같습니다.

현실세계의 것을 코드로~~ 상속~~  등등 객체지향의 막연한 개념과 얕은 지식으로만 알고 있었는데

반복적이고 장황한 코드를 줄이기 위해 Object Class를 생성하여 코드를 줄이는데 성공 했습니다.

- before - 불편

  ![img](/assets/images/ihhwang/before.gif)

  일단 만들어 보고 나중에 리팩터링 해야지 하는 마음으로 일단 급하게 코드를 짰는데

  지금 보면 진짜 답답해 보인다.


<details>
<summary>BEFORE-CODE</summary>

**main.ts**


```tsx
// rabbit
const rabbits = this.physics.add.group({
  collideWorldBounds: true,
  velocityY: this.gameConfig.velocityY,
  maxSize: this.gameConfig.rabbit.count,
  gravityY: this.gameConfig.gravity,
});

// mouse
const mouses = this.physics.add.group({
  collideWorldBounds: true,
  velocityY: this.gameConfig.velocityY + 50,
  maxSize: this.gameConfig.mouse.count,
  gravityY: this.gameConfig.gravity
});

// bombs
const bombs = this.physics.add.group({
  collideWorldBounds: true,
  velocityY: this.gameConfig.velocityY + 100,
  maxSize: this.gameConfig.bomb.count,
  gravityY: this.gameConfig.gravity
});

// clock
const clocks = this.physics.add.group({
  collideWorldBounds: true,
  velocityY: this.gameConfig.velocityY + 100,
  maxSize: this.gameConfig.clock.count,
  gravityY: this.gameConfig.gravity
});

rabbits.clear();
mouses.clear();
bombs.clear();
clocks.clear();
const createRabbit = () => {
  const x = this.getXPos(0, width);
  const y = this.getXPos(50, 90);
  rabbits.create(x , y, 'rabbit');
}

const createMouse = () => {
  const x = this.getXPos(100, width - 100);
  const y = this.getXPos(50, 90);
  mouses.create(x, y, 'mouse');
}

const createBomb = () => {
  const x = this.getXPos(100, width - 100);
  const y = this.getXPos(0, 90);
  bombs.create(x, y, 'bomb');
}

const createClock = () => {
  const x = this.getXPos(100, width - 100);
  const y = this.getXPos(0, 90);
  clocks.create(x, y, 'clock');
}

this.time.addEvent({
  delay: this.gameConfig.rabbit.delay,
  callback: createRabbit,
  callbackScope: this,
  loop: true,
  startAt: 0
});

this.time.addEvent({
  delay: this.gameConfig.mouse.delay,
  callback: createMouse,
  callbackScope: this,
  loop: true,
  startAt: 0
});

this.time.addEvent({
  delay: this.gameConfig.bomb.delay,
  callback: createBomb,
  callbackScope: this,
  loop: true,
  startAt: 0
});

this.time.addEvent({
  delay: this.gameConfig.clock.delay,
  callback: createClock,
  callbackScope: this,
  loop: true,
  startAt: 0
});
```


</details>

    

- after-편안

  ![img](/assets/images/ihhwang/after.gif)

<details>
<summary>AFTER-CODE</summary>

**main.ts**


```tsx
const rabbitPosition: PositionInfo = {
  startX: 0,
  startY: 50,
  widthX: width,
  widthY: 90
}

const bombPosition: PositionInfo = {
  startX: 100,
  startY: 0,
  widthX: width - 100,
  widthY: 90
}

const rabbits = new ObjectController(this, 'rabbit', this.gameConfig.rabbit, rabbitPosition).init();
const mouses = new ObjectController(this, 'mouse', this.gameConfig.mouse, rabbitPosition).init();
const bombs = new ObjectController(this, 'bomb', this.gameConfig.bomb, bombPosition).init();
const clocks = new ObjectController(this, 'clock', this.gameConfig.clock, bombPosition).init();
```


**object.controller.ts**


```tsx
import Phaser from "phaser";
import {GameConfig, GameValance, ObjectInfo, PositionInfo} from "../util/game.config";

export class ObjectController  {
  private scene: Phaser.Scene;
  private key: string;
  private objectGroup: Phaser.Physics.Arcade.Group;
  private objectConfig: ObjectInfo;
  private positionConfig: PositionInfo;
  private gameConfig: GameConfig;

  constructor(scene: Phaser.Scene, key: string, objectConfig: ObjectInfo, positionConfig: PositionInfo, gameConfig: GameConfig) {
    this.scene = scene;
    this.key = key;
    this.positionConfig = positionConfig;
    this.objectConfig = objectConfig;
    this.gameConfig = gameConfig;
  }

  init() {
    this.objectGroup = this.scene.physics.add.group({
      collideWorldBounds: true,
      velocityY: this.gameConfig.velocityY,
      maxSize: this.objectConfig.count,
      gravityY: this.gameConfig.gravity,
    })

    return this.objectGroup;
  }
}
```


</details>


요즘은 유니티같은 클릭 몇번과 짧은 타이핑으로 게임을 구현할 수 있는 프레임워크도 있습니다.

그에 비해 Phaser 프레임워크는 아무것도 없는 빈 화면부터 시작해야 하기 때문에 진입 장벽이 높을 것 같습니다.

물론 예제 코드들이 많긴 하지만 익숙해지는 시간이 많이 소요가 됩니다.

하지만 이런 점이 오히려 개발자 스러운 것 같다는 생각이 들었습니다.

불편했지만 하나하나 찾아가면서 코드를 작성한게 오히려 개인적으로 큰 깨달음을 얻을 수 있었습니다.

게임 개발에만 집중할 수 있었다면 더 빠르고 효율적으로 만들 수 있을거란 생각이 들었지만

이 경험을 바탕으로 다음번엔 좀 더 빠르고 효율적으로 개발을 할 수 있을거라는 자신감이 생겼습니다.

내년에도 기회가 있다면 좀 더 고 퀄리티로 만들어보고 싶습니다.

And Special Tanks to… **Chat GPT**

![img](https://media3.giphy.com/media/dOJt6XZlQw8qQ/giphy.gif?cid=7941fdc6k0csevxoys907h9t9f00qvp9g51ymed5o1lcyyb7&ep=v1_gifs_search&rid=giphy.gif&ct=g)

---

# 조용빈
<!--조용빈-->
## - 게임플레이 외적 부분
게임 플레이 외에 부수적인 부분을 담당했습니다.

### 인트로
기획 디자인을 처음 보고 이걸 어떻게 만들지 생각하다가 그냥 이미지를 다 따서 만들자
생각 했습니다.

![intro](/assets/images/ybcho/game/intro.png)

### 게임결과
![game_result](/assets/images/ybcho/game/game_result.png)

**객체 배치**

배치는 setOrigin 0.5를 줘서 객체 기준점을 중앙으로 설정하고 메인 카메라의 넓이를
이용하여 화면 중앙값을 구해 중앙 기준으로 배치시켰습니다.

(처음엔 일일이 숫자를 주며 배치를....)


```tsx
const center = this.cameras.main.width / 2;

const container = this.add.image(center, 420, "container");
container.setScale(0.7).setOrigin(0.5).setDepth(0);
this.add
        .image(center, 150, "game_result")
        .setScale(0.7)
        .setOrigin(0.5)
        .setDepth(0);
this.rankBtn = this.add.image(center, 750, "rank_btn");
this.rankBtn
        .setScale(0.7)
        .setOrigin(0.5)
        .setDepth(0)
        .setInteractive({ cursor: "pointer" })
        .on("pointerup", () => {
          this.playSound();
          this.scene.stop();
          this.scene.switch("rankingList");
        });
```

**다이얼로그**

![retry](/assets/images/ybcho/game/retry.png)

[https://rexrainbow.github.io/phaser3-rex-notes/docs/site/ui-dialog](https://rexrainbow.github.io/phaser3-rex-notes/docs/site/ui-dialog)

다이얼로그를 이해하는데 많은 시간이 걸렸고 만드는데 이미지가 제멋대로 배치되서 너무 힘들었습니다ㅠ

rexUI 다이얼로그가 자동으로 뒤 객체들을 클릭하지 못하게 만들어주지 않기 때문에
버튼에 이벤트를 걸어 뒤 버튼들을 disable 시켜주었습니다.
layout()으로 객체들을 다이얼로그에 맞게 배치되도록 하였습니다.
```tsx
this.retryDialog = this.rexUI.add
      .dialog({
        x: center,
        y: 450,
        width: 430,
        height: 310,
        background: this.rexUI.add
          .roundRectangle(0, 0, 450, 377, 20, 0xffffff)
          .setDepth(2),
        content: this.add
          .image(0, 0, "dialog_text")
          .setScale(0.7)
          .setOrigin(0.5)
          .setDepth(3),
        actions: [
          this.add
            .image(0, 0, "cancel_btn")
            .setScale(0.7)
            .setOrigin(0.5)
            .setDepth(3)
            .setInteractive({ cursor: "pointer" })
            .on("pointerup", () => {
              this.activateButton();
              this.playSound();
              this.retryDialog.popUp(300).setVisible(false);
            }),
          this.modalOkBtn,
        ],
        space: {
          top: 50,
          bottom: 0,
          title: 0,
          titleLeft: 0,
          content: 50,
          action: 15,
        },
        expand: {
          title: false,
          description: false,
          content: false,
          actions: false,
          choices: true,
        },
        align: {
          title: "center",
          content: "center",
          choices: "center",
          actions: "center", // 'center'|'left'|'right'
        },
      })
      .layout()
      .setVisible(false);

    function disableButton() {
      this.rankBtn.disableInteractive();
      this.confirmBtn.disableInteractive();
      this.retryBtn.disableInteractive();
    }

    function activateButton(){
      this.rankBtn.setInteractive({ cursor: "pointer" });
      this.confirmBtn.setInteractive({ cursor: "pointer" });
      this.retryBtn.setInteractive({ cursor: "pointer" });
    }
```


### 랭킹

**스크롤 패널**

![ranking](/assets/images/ybcho/game/ranking.png)

[https://rexrainbow.github.io/phaser3-rex-notes/docs/site/ui-scrollablepanel/](https://rexrainbow.github.io/phaser3-rex-notes/docs/site/ui-scrollablepanel/)

rexUI의 scrollablePanel을 이용하여 스크롤이 되는 리스트를 만들었습니다.

```tsx
function setScrollPanel() {
    const center = this.cameras.main.width / 2;
    if (this.scrollPanel) {
      this.scrollPanel.destroy();
    }
    this.scrollPanel = this.rexUI.add
      .scrollablePanel({
        x: center,
        y: 570,
        width: 500,
        height: 600,
        scrollMode: "y",
        mouseWheelScroller: {
          focus: true,
          speed: 0.2,
        },
        background: this.rexUI.add
          .roundRectangle(0, 450, 450, 377, 20, 0xffffff)
          .setDepth(2),
        panel: {
          child: this.CreatePanel(),
          mask: { padding: 10 },
        },
        slider: {
          thumb: this.rexUI.add
            .roundRectangle(0, 0, 5, 100, 13, 0x000000, 0.3)
            .setDepth(3),
        },
        space: {
          left: 5,
          right: 5,
          top: 95,
          bottom: 30,
          panel: -20,
        },
      })
      .setOrigin(0.5)
      .layout();
  }
```
**랭킹 새로고침**

새로고침 아이콘을 클릭 시 time.addEvent를 이용하여 특정 시간만큼 이미지가 돌아가도록 설정하였습니다.

```tsx
this.reload = this.add
        .image(center + 230, 240, "reload")
        .setScale(0.6)
        .setOrigin(0.5)
        .setDepth(0)
        .setInteractive({ cursor: "pointer" })
        .on("pointerdown", async () => {
          this.playSound();
          const update = this.time.addEvent({
            delay: 10,
            callback: () => {
              this.reload.rotation += 0.1;
            },
            loop: true,
          });
          const { data } = await getRankingList(site, mb_sn);
          this.myRanking = data.myRanking;
          this.userList = data.data;
          setTimeout(() => {
            this.setScrollPanel(); //setScrollPanel에선 기존 panel destroy
            update.remove();
            this.reload.setRotation(0);
          }, 1500);
        });
```

**탭구현**

![winner](/assets/images/ybcho/game/winner.png)

탭 구현은 원하는 예제를 못찾아 꼼수로 이미지를 겹쳐 탭처럼 보이도록 하였습니다.

```tsx
const tab1 = this.add
      .image(this.center, 180, "tab1")
      .setScale(0.77)
      .setOrigin(0.5)
      .setDepth(3);
    const tab_text1 = this.add
      .image(this.center + 10, 220, "tab_text1")
      .setScale(0.77)
      .setOrigin(0.5)
      .setDepth(3);
    const tab2 = this.add
      .image(this.center, 180, "tab2")
      .setScale(0.77)
      .setOrigin(0.5)
      .setDepth(3)
      .setVisible(false);
    const tab_text2 = this.add
      .image(this.center, 220, "tab_text2")
      .setScale(0.77)
      .setOrigin(0.5)
      .setDepth(3)
      .setVisible(false);
    const tab_btn1 = this.add
      .image(this.center - 109, 176, "tab_btn1")
      .setScale(0.77)
      .setOrigin(0.5, 1)
      .setDepth(4)
      .setInteractive({ cursor: "pointer" })
      .on("pointerup", async () => {
        this.playSound();
        tab1.setVisible(true);
        tab2.setVisible(false);
        tab_text1.setVisible(true);
        tab_text2.setVisible(false);
        notice.setVisible(false);
        const { data } = await getWinnerList(site);
        this.userList = data;
        this.setScrollPanel("지난주 당첨자"); //탭 클릭시 기존 panel destroy
      });
    const tab_btn2 = this.add
      .image(this.center + 113, 176, "tab_btn2")
      .setScale(0.77)
      .setOrigin(0.5, 1)
      .setDepth(4)
      .setInteractive({ cursor: "pointer" })
      .on("pointerup", async () => {
        this.playSound();
        tab2.setVisible(true);
        tab1.setVisible(false);
        tab_text2.setVisible(true);
        tab_text1.setVisible(false);
        notice.setVisible(true);
        const { data } = await getRankingList(site, mb_sn);
        this.myRanking = data.myRanking;
        this.userList = data.data;
        this.setScrollPanel("이번주 랭킹"); //탭 클릭시 기존 panel destroy
      });
```


## 이슈 & 해결
저에겐 하나하나가 다 이슈였습니다....

### - 요소 배치
1. 팝업창에 이미지를 추가 했는데 보이질 않았습니다.
잘못 추가했는지 검색하고 didalog()를 잘못 이해했나 원인을 찾아 보았지만 해결이 안되서
노가다로 테스트를 하며 결국 원인을 찾았는데 depth 문제였습니다. 
이미지가 팝업 배경이미지 뒤로 배치가 되서 안보였던것 뿐이였습니다. 알고보니 정말 어이없는 제 실수였습니다.

2. 아무리 이미지, 도형을 배치해도 엉뚱한 위치에 배치가 되서 한참을 헤메었습니다.
문서를 보니 layout()이라는 부모?객체에 알맞게 정렬해 주는 메서드가 있어 사용하여 해결했습니다.

### - 탭
디자인이 중간에 변경되어 탭 기능이 추가되었습니다.
rexUI 플러그인에 탭 기능이 있긴했지만 사용하지 않고 이미지를 겹쳐 탭처럼 보이게끔 하여 해결했습니다.

### - CORS 에러
백엔드에 처음 연동했는데 cors에러가 발생해 조정훈 부장님께서 아무리 cors 설정을 하셨지만 계속 에러가 발생했습니다.
유부장님께서 backend에서 위즈패널backend로 api를 호출할 수 있도록 만들어 주셔서 해결되었습니다.
(브라우저에서 요청시에 발생하는 문제로, backend에서 backend로 요청시에는 발생하지 않는다고 합니다) 



## 느낀점
처음 토끼게임 기획서를 봤을때 어려워보여 어떻게 만들어야 할지 막막했지만
차근차근 하나씩 작업 하니 어느새 완성을 했네요.

물론 시행착오가 무수히 많았습니다.

phaser3도 너무 오랜만이기도 하고, 인터넷에 예제도 별로 없고, 
인터넷에 있는 예제를 똑같이 따라했는데 안될 때, 이 기능을 구현하는데엔 어떤 메소드를 써야 될지, 백엔드에 어떤 데이터 요청이 필요할지
감이 잘 안오고, 때때로 이벤트가 안먹히거나 object가 안보일 때 에러 표시도 안되어서 한참을 헤메었을 때도 있었고, 기능을 구현하는데 너무 복잡해서
하나하나씩 값을 변경하여 테스트 하면서 어떤 속성인지 확인할 때, 인터넷에 있는 예제를 똑같이 따라했는데 안될 때, 디자인 및 요구사항이 변경되었을 때 등등
중간에 포기하고 싶고 미치고 싶을 때가 많았습니다(물론 제가 부족한 탓이였어요ㅎㅎ)

그래도 상용 후 주변에서 참여도 많이 해주시고 비록 게임플레이 부분을 만든건 아니지만 재밌다고 해주셔서 힘들었던 기억이 잊혀질 만큼 뿌듯했습니다.





