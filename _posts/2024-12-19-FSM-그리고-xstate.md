---
layout: post
title: "FSM, 그리고 XState"
date: 2024-12-13 15:00:00 +0900
author: hsyoo
categories: javascript xstate FSM
published: true
---
<hr/>

_아래 블로그는 chatGPT를 통해 작성되었습니다._

# 🎯 웹 개발자를 위한 상태 머신(FSM) 개념 정리

웹 개발을 하다 보면 **"이 컴포넌트가 현재 어떤 상태인지?"**, **"다음 상태로 어떻게 전환해야 하는지?"** 와 같은 상태 관리가 중요해지는 순간이 있습니다.  
이때 강력한 해결책 중 하나가 바로 **유한 상태 머신(Finite State Machine, FSM)** 입니다.

FSM은 프론트엔드와 백엔드 모두에서 유용하게 활용될 수 있으며,  
특히 **UI 상태 관리, 폼 처리, 인증 흐름, 게임 개발, 백엔드 비즈니스 로직** 등에 활용됩니다.

---

## 🔍 유한 상태 머신(Finite State Machine)이란?

**유한 상태 머신(FSM)** 은 **일정한 개수(유한한 개수)의 상태를 가지며, 특정한 규칙에 따라 한 상태에서 다른 상태로 전환되는 모델**입니다.

즉, 어떤 시스템이 **현재 어떤 상태인지**, 그리고 **어떤 조건에서 상태가 변경될 수 있는지**를 정의하는 구조입니다.

FSM의 특징은 다음과 같습니다:

1. **유한한 개수의 상태(Finite States)를 가진다.**
2. **현재 상태(Current State)가 하나만 존재한다.**
3. **이벤트(Event)에 의해 상태가 변경(Transition)된다.**
4. **모든 상태 전이는 명확한 규칙을 따른다.**

---

## 📌 상태 머신의 기본 구조

FSM은 다음과 같은 요소들로 구성됩니다:

- **State (상태):** 시스템이 가질 수 있는 각 상태 (예: `idle`, `loading`, `success`, `error`)
- **Event (이벤트):** 상태를 변화시키는 외부 요인 (예: 버튼 클릭, API 응답)
- **Transition (전이):** 특정 이벤트가 발생했을 때 상태가 변화하는 규칙
- **Initial State (초기 상태):** 시스템이 처음 시작할 때의 상태

---

## 🚀 웹 개발에서 FSM이 왜 필요할까?

웹 애플리케이션에서는 다양한 상태 변화가 발생합니다.  
**단순한 if-else 분기문으로 상태를 관리하면 코드가 복잡해지고, 버그 발생 확률이 높아질 수 있습니다.**

FSM을 활용하면 **코드를 더 직관적이고 유지보수하기 쉽게 만들 수 있습니다.**  
특히 다음과 같은 경우에 유용합니다:

1. **UI 상태 관리** (예: 버튼의 활성화/비활성화, 모달 열기/닫기)
2. **비동기 데이터 처리** (예: API 요청 중 로딩 → 성공 또는 실패)
3. **폼 입력 검증 및 상태 관리** (예: 유효한 입력값을 받을 때만 다음 단계로 이동)
4. **페이지 내 탐색 흐름 제어** (예: 다단계 회원가입 프로세스)
5. **게임 및 애니메이션 상태 제어** (예: 캐릭터 상태 변화, UI 애니메이션 트리거)

---

## 🎯 FSM의 예제: API 요청 상태 관리

웹 개발에서 가장 흔한 상태 관리는 **비동기 API 요청**입니다.  
아래는 **"데이터 로딩 상태를 상태 머신으로 관리하는 예시"** 입니다.

### 1️⃣ 기존 방식 (if-else 기반)
```javascript
let state = "idle";

function fetchData() {
  if (state === "loading") return;
  state = "loading";
  console.log("데이터를 불러오는 중...");

  fetch("https://api.example.com/data")
    .then((response) => {
      state = "success";
      console.log("데이터 불러오기 성공!");
    })
    .catch((error) => {
      state = "error";
      console.log("데이터 불러오기 실패!", error);
    });
}
```

- **상태가 여러 개 생길 경우 if-else 문이 많아지면서 유지보수가 어려워질 수 있습니다.**
- **코드가 직관적이지 않고, 상태 간 전이가 명확하지 않습니다.**

---

## 🎯 FSM을 적용한 방식

FSM을 사용하면 상태 전이를 구조화하고, 불필요한 조건문을 줄일 수 있습니다.

```javascript
const stateMachine = {
  idle: { FETCH: "loading" },
  loading: { SUCCESS: "success", ERROR: "error" },
  success: { RESET: "idle" },
  error: { RETRY: "loading", RESET: "idle" },
};

let currentState = "idle";

function transition(event) {
  const nextState = stateMachine[currentState][event];
  if (nextState) {
    currentState = nextState;
    console.log(`현재 상태: ${currentState}`);
  } else {
    console.log(`이벤트 ${event}는 상태 ${currentState}에서 허용되지 않습니다.`);
  }
}

// 사용 예시
transition("FETCH");  // 현재 상태: loading
transition("SUCCESS"); // 현재 상태: success
transition("RESET");   // 현재 상태: idle
```


✅ **FSM을 활용하면 다음과 같은 장점이 있습니다.**
- **상태 전이가 명확하게 정의됨**  
  → 어떤 이벤트가 발생했을 때, 어떤 상태로 변경되는지 한눈에 파악할 수 있습니다.
- **상태 전이가 허용되지 않는 경우 예외 처리가 쉬움**  
  → 잘못된 상태 전이가 발생하지 않도록 방지할 수 있습니다.
- **if-else가 줄어들어 코드가 깔끔해짐**  
  → 유지보수가 쉬워지고, 가독성이 높아집니다.

---

## 🔥 프론트엔드에서 FSM 활용: XState 라이브러리

JavaScript에서는 **XState** 같은 FSM 라이브러리를 사용하면 더 강력한 상태 관리가 가능합니다.

### 📌 XState를 사용한 예제
```javascript
import { createMachine, interpret } from "xstate";

const fetchMachine = createMachine({
  id: "fetch",
  initial: "idle",
  states: {
    idle: { on: { FETCH: "loading" } },
    loading: { on: { SUCCESS: "success", ERROR: "error" } },
    success: { on: { RESET: "idle" } },
    error: { on: { RETRY: "loading", RESET: "idle" } },
  },
});

const service = interpret(fetchMachine)
  .onTransition((state) => console.log(`현재 상태: ${state.value}`))
  .start();

service.send("FETCH");  // 현재 상태: loading
service.send("SUCCESS"); // 현재 상태: success
service.send("RESET");   // 현재 상태: idle
```

✅ **XState의 장점**
- **상태와 전이를 선언적으로 관리할 수 있음**  
  → 상태 간 전이를 코드가 아닌 데이터로 관리할 수 있습니다.
- **리액트, Vue 등 다양한 프레임워크와 쉽게 통합 가능**  
  → 컴포넌트 상태 관리를 더 구조적으로 할 수 있습니다.
- **시각적으로 FSM을 그려볼 수 있는 도구 제공**  
  → 개발자 도구를 활용하여 상태 흐름을 시각적으로 확인할 수 있습니다.

---

## 🔗 FSM을 웹 개발에서 어떻게 활용할 수 있을까?

1. **UI 상태 관리** → 모달, 드롭다운, 탭 UI 등 상태 기반 컴포넌트 관리
2. **비동기 데이터 처리** → API 호출 후 상태 변화 (`loading → success → error`)
3. **폼 입력 검증 및 상태 흐름** → 다단계 폼에서 각 단계별 유효성 검사
4. **게임 및 애니메이션** → 플레이어 상태(이동, 공격, 대기) 또는 UI 애니메이션 트리거
5. **페이지 탐색 및 내비게이션 흐름 제어** → 인증 플로우, 온보딩 과정

---

## 🎯 결론: 웹 개발에서 FSM을 사용해야 할까?

✅ FSM은 상태가 많아지고 복잡해질수록 **코드의 유지보수성과 가독성을 높이는 데 도움을 줍니다.**  
✅ 특히 **UI 상태 관리, API 상태 관리, 복잡한 워크플로우 관리**에 유용합니다.  
✅ 기본 개념을 이해하고 **필요한 경우 XState 같은 라이브러리를 활용하면 더 강력한 상태 관리가 가능합니다.**

---

## 🚀 더 읽어보기
- [XState 공식 문서](https://xstate.js.org/docs/){:target="_blank"}
- [MDN: Finite State Machine](https://developer.mozilla.org/en-US/docs/Web/Finite_state_machine){:target="_blank"}
