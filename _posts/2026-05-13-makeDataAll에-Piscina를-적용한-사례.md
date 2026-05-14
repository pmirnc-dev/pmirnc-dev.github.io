---
layout: post
title:  "메타서베이 Piscina(Worker Threads) 적용 사례"
date:   2026-05-14 09:00:00 +0900
author: shpark
categories: ["nodejs", "worker_threads", "OOM", "piscina"]
published: true
---
<hr/>

메타서베이 전체 데이터 다운로드 기능의 병렬 처리 도입과 OOM 해결 과정을 다룹니다.

## 1. 개요

MetaSurvey 웹에서 **데이터 다운로드**는 두 가지로 나뉘는데, **완료자 기준**과 **전체 기준**이 있습니다.

| 구분 | 담당 파일 | 동작 |
|---|---|---|
| 완료자 데이터 | makeData.js | 응답 완료 즉시 1건씩 DATA_{SNUM} 컬렉션에 저장. 별도 최적화 불필요. |
| 전체 데이터 | makeDataAll.js | 버튼 클릭 시 ANSWER 전체를 find하여 문항 유형별로 가공 후 DATA_ALL_{SNUM} 컬렉션 생성 & Excel 다운로드 |

이번 포스팅에서는 `makeDataAll.js`에 Piscina를 적용해 병렬 처리를 도입하고, 그 과정에서 발생한 OOM 문제를 해결한 사례를 다룹니다.

## 2. 데이터 다운로드 처리 흐름

전체 데이터 다운로드는 Phase 1(데이터 가공)과 Phase 2(엑셀 생성) 두 단계로 처리됩니다. #355에서는 두 단계 모두 아래와 같이 개선하였습니다.

### Before (#355 반영 전) — Piscina 없음

```
Phase 1: 데이터 가공 (메인 스레드 직렬 처리)

1. ANSWER 컬렉션을 toArray()로 전체 N건 메모리에 로드
2. ANSWERS 배열에서 1건 꺼냄 (splice)
3. 해당 1건의 UID로 LIST_{SNUM} 1건 조회
4. MakeData()로 가공 → DATA_ALL_{SNUM}에 insert
5. 2~4 반복 (N번)

Phase 2: 엑셀 생성

DATA_ALL_{SNUM}에서 skip(n).limit(1)로 1건씩 조회 → Excel에 addRow → commit
→ 다시 skip(n+1).limit(1)로 다음 1건 조회 → 반복
→ xlsx 파일 완성 → S3 업로드 / 이메일 발송
```

**문제점:**

- `toArray()`로 전체 데이터를 메모리에 올림
- LIST 조회가 1건마다 1번씩 = N번 DB 호출
- MakeData를 메인 스레드에서 1건씩 직렬 처리
- Phase 2에서 `skip(n).limit(1)` = O(n²) 성능 (매번 처음부터 n건 건너뜀)

### After (#355 반영 후) — Piscina 병렬 + cursor

```
Phase 1: 데이터 가공 (Piscina 병렬 처리)

1. ANSWER 컬렉션을 cursor로 500건씩 읽기 + 해당 500건의 LIST_{SNUM} $in 매칭
2. 다음 500건은 프리페치로 미리 읽어 대기 (DB 대기 시간 제거)
3. 1건씩 Piscina 워커에 전달 → MakeData 병렬 가공
4. 500건 처리 완료 → DATA_ALL_{SNUM}에 insertMany
5. 대기 중이던 500건이 현재가 됨 → 1~4 반복

Phase 2: 엑셀 생성

DATA_ALL_{SNUM}을 cursor로 1건씩 순차 읽기 → Excel에 addRow → commit
→ cursor.next()로 다음 1건 → 반복
→ xlsx 파일 완성 → S3 업로드 / 이메일 발송
```

**개선점:**

- Phase 1: 메모리에 최대 1,000건(현재 500 + 대기 500)만 존재
- Phase 1: LIST 조회 N번 → N/500번 (`$in`)
- Phase 1: MakeData 메인 스레드 직렬 → Piscina 워커 병렬
- Phase 1: 배치 단위(500건) insertMany 후 배열 참조 해제 (`insertDocs = null`) → 배치 간 결과 누적 없음
- Phase 2: `skip(n).limit(1)` O(n²) → cursor O(n)

## 3. Node.js 싱글 스레드의 한계와 Piscina

이전 포스트에서 멀티 스레드를 다루었었는데요, 관련해서 궁금하신 분들은 해당 포스트([worker-threads를 활용하여 Node.js에서 multi-thread 구현](https://pmirnc-dev.github.io/nodejs/worker-threads/multi-thread/2023/06/26/worker-threads%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-nodejs%EC%97%90%EC%84%9C-multi-thread-%EA%B5%AC%ED%98%84.html))도 함께 참고하면 좋을 것 같습니다.

### I/O 작업 vs CPU 작업

Node.js는 메인 스레드 하나에서 이벤트 루프를 통해 작업을 처리합니다. 작업 유형에 따라 동작이 다른데, **I/O 작업은 Node.js 내부(libuv)가 백그라운드에서 처리**하므로 메인 스레드는 응답 대기 동안 다른 일을 할 수 있습니다. 반면 **CPU 작업은 위임할 곳이 없어 메인 스레드가 직접 실행**하므로, 그 시간 동안 다른 일은 멈춥니다.

| 유형 | 예시 | 싱글 스레드에서 |
|---|---|---|
| **I/O** | DB 조회, 파일 읽기/쓰기, 네트워크 요청 | libuv가 백그라운드 처리 → 메인 스레드는 다른 일 가능 → 효율적 |
| **CPU** | 반복 계산, 객체 가공, 문자열 처리 | 메인 스레드가 직접 실행 → 점유 상태 → 메인 스레드 블로킹 |

### Node.js worker_threads를 통한 병렬 처리

Node.js에서 CPU 집약 작업을 메인 스레드와 분리해서 처리하려면 `worker_threads` 모듈을 사용합니다. 같은 프로세스 안에서 별도의 스레드를 띄워 작업을 위임할 수 있으며, 각 워커는 독립된 V8 인스턴스를 가지므로 메인과 메모리를 직접 공유하지 않고 메시지(IPC)로 데이터를 주고받습니다. makeDataAll.js의 `MakeData()`는 문항 유형별 분기, 객체 키 생성, 숫자 변환 등 순수한 JavaScript 연산이고 I/O가 없는 CPU 집약 작업이므로, `worker_threads`를 활용한 병렬 처리를 도입했습니다.

### worker_threads를 직접 사용할 경우

아래는 이전 포스트의 worker_threads 코드 예시입니다.

**메인 스레드** — 워커를 생성·관리하는 코드입니다.

```typescript
// worker.multi.thread.ts
import { Worker } from 'worker_threads';
import path from 'path';
const workerJs = path.resolve(__dirname, './worker.thread.js');

const count = 10;

function multiThread() {
    let computed = 0;                          // 완료 카운트 관리
    console.time('multi threads');
    for (let i = 1; i <= count; i++) {
        const worker = new Worker(workerJs);   // 워커 생성
        worker.on('message', (data) => {       // 메시지 콜백 등록
            computed += 1;                     // 완료 카운터 증가
            if (computed === count) {          // 전체 완료 조건
                console.timeEnd('multi threads');
            }
        });
        worker.postMessage(i);                 // 작업 전달
    }
}
multiThread();
```

**워커 파일** — 실제 CPU 작업을 수행하는 코드입니다.

```javascript
// worker.thread.js
const { parentPort } = require('worker_threads');

function computing(index) {
    let sum = 0;
    // 매우 무거운 CPU 작업 시뮬레이션
    for (let i = 0; i < 1e9; i++) {
        sum += i;
    }
    return index;
}

parentPort?.on('message', (index) => {          // 메시지 수신
    const result = computing(index);
    parentPort?.postMessage(result);            // 결과 전송
    parentPort?.close();                        // 워커 종료
});
```

### 각 워커는 독립된 V8 인스턴스

위처럼 생성된 각 워커는 메인과 분리된 **독립된 V8 인스턴스**를 갖습니다. **V8 인스턴스**는 JavaScript를 실행하는 엔진의 독립된 실행 환경입니다. 한 인스턴스 안에 자체 힙 메모리, 실행 스택, 로드된 모듈, 글로벌 변수가 모두 들어있습니다. 워커 하나당 V8 인스턴스 하나가 새로 생성되고, 메인과 워커는 메모리를 직접 공유할 수 없습니다.

> **생성·초기화 비용:** 워커 1개를 만들 때마다 (1) V8 인스턴스 메모리 할당, (2) 워커 파일 코드 로드/파싱, (3) require된 모듈 로드, (4) workerData 직렬화/역직렬화가 발생합니다. 짧은 작업에서는 이 비용이 작업 자체보다 클 수 있고, 워커 수가 많아지면 메모리도 비례해서 늘어납니다.

일반적인 멀티 스레드 방식에서는 **작업 N개에 워커 N개를 생성**합니다. 작업이 5,000건이면 워커 5,000개를 모두 동시에 만들고, 작업이 끝나면 폐기합니다. 워커를 재사용하려면 워커 N개를 미리 생성해두고, 각 워커의 busy 상태 관리, 작업 분배, 큐잉, 워커 크래시 시 재생성 로직을 직접 작성해야 합니다. **Piscina는 이 모든 풀 관리를 간편하게 처리해주며, `maxThreads` 옵션 하나로 동시 실행 워커 수를 지정하면 워커 재사용·큐잉·재생성이 자동으로 동작합니다.**

### Piscina를 사용할 경우

위와 동일한 작업(10건 computing)을 Piscina로 처리하면 워커 생성, 메시지 송수신, 완료 감지 로직이 자동화되어 코드가 줄어듭니다. 추가로 워커 재사용과 큐잉도 기본으로 제공됩니다.

**메인 스레드** — 풀을 생성하고 `pool.run()`으로 작업을 위임합니다.

```typescript
// 메인 스레드 — Piscina
import Piscina from 'piscina';
import path from 'path';

const pool = new Piscina({
    filename: path.resolve(__dirname, './worker.thread.js'),
    maxThreads: 6                              // 워커 6개 생성 후 재사용
});

const count = 10;

async function multiThread() {
    console.time('multi threads');
    const results = await Promise.all(
        Array.from({length: count}, (_, i) => pool.run(i + 1))
    );
    console.log(results);
    console.timeEnd('multi threads');
}

multiThread();
```

**워커 파일** — 작업 함수를 그냥 `module.exports`로 내보내면 됩니다.

```javascript
// 워커 스레드 — Piscina (worker.thread.js)
function computing(index) {
    let sum = 0;
    for (let i = 0; i < 1e9; i++) sum += i;
    return index;
}

module.exports = computing;                    // parentPort, postMessage, close 불필요
```

### 측정 방법

worker_threads 직접 사용과 Piscina의 차이를 정량적으로 보기 위해 다음과 같이 측정했습니다.

- **작업 부하**: 워커 1개당 `for (let i = 0; i < 1e9; i++) sum += i;` — 10억 번 덧셈을 수행하는 CPU 집약 작업 (1건당 약 1초). 이 작업을 100건 처리.
- **실행 시간**: `console.time()` / `console.timeEnd()`로 처음~끝 경과 시간 측정
- **메모리**: `setInterval`로 50ms마다 `process.memoryUsage().rss`를 폴링하여 작업 중 가장 높았던 값(**peak rss**)을 기록. rss는 프로세스가 차지한 실제 물리 RAM 전체(워커 스레드의 V8 힙 포함)
- **측정 횟수**: 같은 명령을 3회 실행하여 평균

### 실행 결과 (작업 100건)

| 방식 | 실행 시간 (평균) | peak rss (평균) | 특징 |
|---|---|---|---|
| 싱글 스레드 (single) | 103.7 s | 35 MB | 메인 스레드에서 순차 처리 |
| worker_threads 직접 (워커 100개) | 11.54 s | 1,204 MB | 작업당 워커 1개 생성·폐기 |
| Piscina (maxThreads=12) | 12.15 s | 178 MB | 워커 12개를 풀에서 재사용 |

### 의미

worker_threads 직접 방식은 작업 100개에 워커 100개를 모두 동시에 생성하여 약 1,200 MB의 메모리를 사용했습니다. 반면 Piscina는 maxThreads(12)만큼 워커를 미리 만들어두고 풀에서 재사용하여, 같은 작업을 처리하면서도 메모리가 약 178 MB로 일정하게 유지되었습니다.

Piscina는 워커의 생성·재사용·큐잉을 알아서 관리해주기 때문에, 작업 수가 늘어나도 워커는 maxThreads만큼만 유지됩니다. 풀을 효율적으로 활용함으로써 메모리 사용량이 작업 수와 관계없이 안정적으로 유지된다는 것이 이번 측정의 핵심입니다.

### Piscina의 장점

| 항목 | worker_threads 직접 사용 | Piscina |
|---|---|---|
| 워커 생성 | 작업마다 `new Worker()` 생성하거나 풀 직접 구현 | 풀에서 자동 재사용 |
| 동시 실행 제한 | 직접 구현 (busy 상태 관리, 큐잉) | `maxThreads`로 자동 제한, 초과분은 내부 큐 대기 |
| 결과 수신 | `worker.on('message')` 콜백 | `pool.run()` → Promise 반환 |
| 완료 감지 | 카운터 직접 관리 | `Promise.all()` |
| 에러 처리 | `worker.on('error')` 직접 핸들링 | Promise reject로 자동 전파 |
| 워커 크래시 | 직접 감지 후 재생성 | 자동 재생성 |
| 공유 데이터 | 매번 `postMessage`로 전달 | `workerData`로 초기화 시 1회 전달 |

## 4. 메타서베이에 적용한 메모리 효율화

Piscina를 도입해 MakeData 가공을 병렬 처리하면서 다운로드 속도는 개선되었으나, 행과 열이 모두 많은 **대규모 설문(컬럼 수천 개, 응답자 수만 명)** 을 다운로드하던 중 OOM이 발생했습니다.

OOM 없이 대규모 데이터도 안정적으로 처리할 수 있도록 메모리 사용량을 다음과 같이 관리하였습니다.

### 4-1. 배치 처리 + 프리페치 파이프라인

기존에는 `toArray()`로 ANSWER 전체 N건을 한번에 메모리에 올렸습니다. N이 커지면 메모리 사용량도 그대로 커집니다. 이를 **cursor로 500건씩 읽어 처리**하도록 변경했습니다.

다만 단순히 500건씩 순차적으로 읽으면 매 배치 사이마다 DB 응답을 기다리는 시간이 생기고, 그동안 워커는 유휴 상태가 됩니다. 이 시간을 활용하기 위해 **현재 500건을 Piscina로 처리하는 동안 다음 500건을 미리 DB에서 읽어두는** 프리페치 파이프라인을 함께 도입했습니다.

그 결과 처리 흐름이 끊기지 않으면서, 메모리에 존재하는 ANSWER 원본은 **현재 처리 중 500건 + 프리페치 대기 500건 = 최대 1,000건**으로 일정하게 유지됩니다. 전체 ANSWER가 몇 만 건이든 이 상한선은 변하지 않습니다.

```javascript
// (1) ANSWER 컬렉션을 cursor로 열어둠 (toArray 사용 안 함)
queueData.answerCursor = queueData.db.collection("ANSWER").find(find, {
    projection: project,           // 필요한 필드만 가져오기
    noCursorTimeout: true          // 대량 처리 중 cursor 만료 방지
}).sort({SEQ: 1});
```

```javascript
// (2) fetchBatchWithList — cursor에서 500건 읽고, 해당 UID의 LIST 데이터를 일괄 조회
var ANSWER_BATCH_SIZE = 500;

async function fetchBatchWithList(cursor, db, SNUM) {
    var batch = [];
    // cursor에서 500건씩 읽어 batch 배열에 모음
    while (batch.length < ANSWER_BATCH_SIZE && await cursor.hasNext()) {
        batch.push(await cursor.next());
    }
    if (batch.length === 0) return null;   // 더 이상 읽을 데이터 없음

    // batch에서 UID만 추출 (LIST 조회용)
    var uniqueUids = batch.map(a => a.UID);

    // 500건의 UID로 LIST 한 번에 조회 (1건씩 조회하면 N번 DB 호출)
    var listDocs = await db.collection('LIST_' + SNUM)
        .find({_id: {$in: uniqueUids}}).toArray();

    // UID로 LIST 객체를 빠르게 찾기 위한 맵 생성
    var listMap = {};
    listDocs.forEach(doc => { listMap[doc._id] = doc; });

    return { batch: batch, listMap: listMap };
}
```

```javascript
// (3) 프리페치 파이프라인 — 처리와 다음 배치 읽기가 동시에 진행됨
var current = await fetchBatchWithList(cursor, db, SNUM);   // 첫 배치 동기로 읽기

while (current) {
    // 다음 배치 읽기를 시작하되 await 하지 않음 → 동시 진행
    var nextPromise = fetchBatchWithList(cursor, db, SNUM);

    // current 500건을 Piscina로 처리 (이 동안 nextPromise는 백그라운드로 DB 읽기 중)
    // ... pool.run() ... insertMany() ...

    // 처리 완료 시점에 nextPromise는 이미 끝나 있어 대기 없이 다음 배치로 전환
    current = await nextPromise;
}
```

<p style="font-size:14px;font-weight:700;color:#555;margin:20px 0 10px 0;">프리페치 적용 — 단계별로 처리와 DB 읽기가 동시에 진행</p>

<div style="overflow-x:auto;-webkit-overflow-scrolling:touch;background:#fff;border-radius:8px;padding:12px;" markdown="0">
<svg viewBox="0 0 720 200" xmlns="http://www.w3.org/2000/svg" width="720" height="200" style="display:block;min-width:720px;font-family:'Pretendard',-apple-system,sans-serif;background:#fff;">
  <g>
    <rect x="110" y="32" width="140" height="40" rx="6" fill="#eeeeee" stroke="#bbb" stroke-dasharray="4 3"/>
    <rect x="260" y="32" width="140" height="40" rx="6" fill="#66bb6a"/>
    <rect x="410" y="32" width="140" height="40" rx="6" fill="#66bb6a"/>
    <rect x="560" y="32" width="140" height="40" rx="6" fill="#66bb6a"/>
    <rect x="110" y="82" width="140" height="40" rx="6" fill="#42a5f5"/>
    <rect x="260" y="82" width="140" height="40" rx="6" fill="#42a5f5"/>
    <rect x="410" y="82" width="140" height="40" rx="6" fill="#42a5f5"/>
    <rect x="560" y="82" width="140" height="40" rx="6" fill="#eeeeee" stroke="#bbb" stroke-dasharray="4 3"/>
  </g>
  <g font-family="'Pretendard',sans-serif">
    <text x="180" y="18" font-size="11" fill="#888" text-anchor="middle">단계 1</text>
    <text x="330" y="18" font-size="11" fill="#888" text-anchor="middle">단계 2</text>
    <text x="480" y="18" font-size="11" fill="#888" text-anchor="middle">단계 3</text>
    <text x="630" y="18" font-size="11" fill="#888" text-anchor="middle">단계 4</text>
    <text x="20" y="56" font-size="13" font-weight="600" fill="#333">처리 (Piscina)</text>
    <text x="180" y="56" font-size="13" fill="#999" text-anchor="middle">대기</text>
    <text x="330" y="57" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">배치 1 처리</text>
    <text x="480" y="57" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">배치 2 처리</text>
    <text x="630" y="57" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">배치 3 처리</text>
    <text x="20" y="106" font-size="13" font-weight="600" fill="#333">DB 읽기</text>
    <text x="180" y="107" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">DB 1 읽기</text>
    <text x="330" y="107" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">DB 2 프리페치</text>
    <text x="480" y="107" font-size="13" font-weight="600" fill="#fff" text-anchor="middle">DB 3 프리페치</text>
    <text x="630" y="106" font-size="13" fill="#999" text-anchor="middle">완료</text>
    <text x="180" y="150" font-size="11" fill="#666" text-anchor="middle">첫 배치 동기 로드</text>
    <text x="330" y="150" font-size="11" fill="#666" text-anchor="middle">처리 중 다음 배치 미리 읽기</text>
    <text x="480" y="150" font-size="11" fill="#666" text-anchor="middle">처리 중 다음 배치 미리 읽기</text>
    <text x="630" y="150" font-size="11" fill="#666" text-anchor="middle">cursor 소진, 마지막 처리만</text>
  </g>
</svg>
</div>

> **핵심:** 매 단계마다 처리(초록)와 DB 읽기(파랑)가 같은 시간에 동시 진행되므로, 배치 전환 시 추가 대기가 없습니다. 메모리에는 ANSWER 원본이 항상 **최대 1,000건**(현재 500 + 프리페치 대기 500)만 유지됩니다.

### 4-2. Piscina에 데이터를 전달하는 방식

cursor에서 읽어온 500건의 배치를 워커로 분배하는 단계에서도, 아래 3개 변수로 한 번에 메모리에 올라오는 양을 제한해두었습니다.

| 제어 변수 | 값 | 무엇을 제한 |
|---|---|---|
| **batch 크기** | 500건 | cursor에서 한 번에 메모리에 들고 있는 ANSWER 수 (4-1) |
| **Piscina maxThreads** | 6 (CPU 코어 절반) | 실제 동시에 실행되는 워커 수 |
| **p-limit** | 12 (maxThreads × 2) | Piscina 워커 호출(pool.run())이 동시에 진행될 수 있는 수 |

#### 구현 방식 — p-limit + maxThreads

p-limit으로 `pool.run()` 동시 호출 수를, maxThreads로 워커 동시 실행 수를 제한하는 구조로 구현했습니다. 실제 코드를 단계별로 보면 다음과 같습니다.

**1) Piscina 워커 풀 생성** — 풀은 모듈 레벨 변수에 보관해 분할 다운로드 시 재사용

```javascript
var maxThreads = Math.max(1, Math.floor(os.cpus().length / 2));   // 6 (CPU 코어의 절반)

if (!piscinaPool) {
    piscinaPool = new Piscina({
        filename: path.join(__dirname, 'makeDataAllBatchWorker.js'),
        workerData: {                            // 풀 생성 시 1회만 전달, 모든 워커가 공유
            SNUM: queueData.SNUM,
            question: JSON.parse(JSON.stringify(question)),
            config: datas.CONFIG,
            qtypeNums: datas.qtypeNums,
            ktypeNames: datas.ktypeNames,
            DOWNLOAD_BY_GROUP_ID: queueData.DOWNLOAD_BY_GROUP_ID || [],
            PARTIAL_DEFAULT_HEAD: queueData.PARTIAL_DEFAULT_HEAD || null
        },
        maxThreads: maxThreads                   // 실제 워커 수 제한
    });
}
var pool = piscinaPool;
```

**2) p-limit으로 pool.run() 동시 호출 제한**

```javascript
var limit = pLimit(maxThreads * 2);    // 12 (워커 수의 2배)
var insertDocs = [];
var batchListMap = current.listMap;
```

`pool.run()`은 호출 즉시 작업 데이터를 직렬화해 Piscina에 적재합니다. 워커가 6개라 실제 처리는 6건씩만 가능하지만 `pool.run()` 호출 자체는 막히지 않기 때문에, p-limit 없이 500건을 한 번에 호출하면 직렬화된 데이터가 500개 모두 메모리에 쌓입니다. p-limit은 `pool.run()` 호출 시점 자체를 동시 12건으로 분산시켜, 워커가 처리할 수 있는 만큼만 메모리에 올라오도록 합니다.

**3) 500건의 ANSWER를 각각 1건씩 워커에 전달**

```javascript
var promises = current.batch.map(function(answer) {
    return limit(function() {                    // ← limit이 한 번에 12개만 안의 함수를 실행
        return pool.run({                        // ← Piscina에 1건만 전달
            answers: [answer],
            listDataArr: [batchListMap[answer.UID] || {}]
        })
        .then(function(results) {
            if (results && results.length > 0 && results[0] != null) {
                var r = results[0];
                // (DATA_HEAD/GUIDE는 첫 결과에서 한 번만 처리)
                insertDocs.push(r.insert);       // ← insert만 수집
            }
            return null;                         // ← 워커 결과 참조 해제 → GC 대상
        });
    });
});
```

워커가 반환한 결과 객체(`results`)에는 `insert` 외에도 `DATA_HEAD`, `DATA_GUIDE`, `LOOP_DATA_*` 등 많은 필드가 포함되어 있습니다. 필요한 `insert`만 `insertDocs`로 옮긴 뒤 `.then()`에서 `return null`로 참조를 끊어주면, Promise가 더 이상 `results`를 붙들지 않아 다음 GC 시점에 회수됩니다. `return null`로 결과 참조를 끊어주지 않으면, 각 Promise가 자신의 resolved value(=`results`)를 붙들고 있어 `Promise.all`이 끝날 때까지 500건의 결과가 메모리에 누적됩니다.

**4) 500건 완료까지 대기 후 한꺼번에 DB 저장**

```javascript
await Promise.all(promises);

if (insertDocs.length > 0) {
    await queueData.onPremiseDb
        .collection("DATA_ALL_" + queueData.SNUM)
        .insertMany(insertDocs);
}
```

### 4-3. 데이터 처리 흐름

500건의 `pool.run()` 호출이 한꺼번에 일어나지 않고, 워커가 비는 속도에 맞춰 하나씩 진행됩니다.

- p-limit은 12개씩만 `pool.run()`을 호출
- 그 12건 중 6건은 워커가 실행, 나머지 6건은 Piscina 큐에서 대기
- 워커가 1건을 마치면 → Piscina 큐에서 대기 중이던 1건이 워커로 진입 → p-limit 대기열에 있던 1건이 Piscina 큐로 이동
- 마친 결과 객체는 `.then()`에서 `return null`로 즉시 참조가 끊겨 GC 대상이 됨

<p style="font-size:14px;font-weight:700;color:#555;margin:20px 0 10px 0;">시간이 흐르며 500건이 이동하는 모습 (maxThreads=6, p-limit=12)</p>

<div style="overflow-x:auto;-webkit-overflow-scrolling:touch;background:#fff;border-radius:8px;padding:12px;" markdown="0">
<svg viewBox="0 0 880 480" xmlns="http://www.w3.org/2000/svg" width="880" height="480" style="display:block;min-width:880px;font-family:'Pretendard',-apple-system,sans-serif;background:#fff;">
  <!-- p-limit band (spans Stage 2 + Stage 3) -->
  <rect x="350" y="5" width="530" height="22" rx="6" fill="#e3f2fd" stroke="#1976d2" stroke-dasharray="4,3"/>
  <text x="615" y="20" font-size="11" font-weight="700" fill="#1976d2" text-anchor="middle">◄ p-limit이 허용한 동시 진행 12건 (maxThreads × 2) ►</text>

  <!-- Column headers -->
  <text x="80" y="50" font-size="12" font-weight="700" fill="#555">① p-limit 대기 (488건)</text>
  <text x="350" y="50" font-size="12" font-weight="700" fill="#555">② Piscina 큐 (6건)</text>
  <text x="620" y="50" font-size="12" font-weight="700" fill="#555">③ 워커 실행 (6건)</text>

  <!-- ===== Snapshot 1: t=0 ===== -->
  <!-- Step label -->
  <rect x="0" y="60" width="70" height="70" rx="6" fill="#1976d2"/>
  <text x="35" y="92" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">시작</text>
  <text x="35" y="110" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">t=0</text>
  <!-- Waiting cell -->
  <rect x="80" y="60" width="260" height="70" rx="6" fill="#e8f5e9"/>
  <rect x="80" y="60" width="3" height="70" fill="#2e7d32"/>
  <text x="95" y="80" font-size="11" font-weight="600" fill="#666">아직 pool.run() 호출 전</text>
  <text x="95" y="108" font-family="monospace" font-size="13" font-weight="700" fill="#222">13, 14, 15, ... , 500</text>
  <!-- Queue cell -->
  <rect x="350" y="60" width="260" height="70" rx="6" fill="#fff3e0"/>
  <rect x="350" y="60" width="3" height="70" fill="#f57c00"/>
  <text x="365" y="80" font-size="11" font-weight="600" fill="#666">직렬화 후 워커 대기</text>
  <text x="365" y="108" font-family="monospace" font-size="13" font-weight="700" fill="#222">7, 8, 9, 10, 11, 12</text>
  <!-- Running cell -->
  <rect x="620" y="60" width="260" height="70" rx="6" fill="#ffebee"/>
  <rect x="620" y="60" width="3" height="70" fill="#c62828"/>
  <text x="635" y="80" font-size="11" font-weight="600" fill="#666">워커에서 처리 중</text>
  <text x="635" y="108" font-family="monospace" font-size="13" font-weight="700" fill="#222">1, 2, 3, 4, 5, 6</text>

  <!-- Event 1 -->
  <rect x="0" y="142" width="880" height="24" rx="6" fill="#f5f7fa"/>
  <text x="10" y="158" font-size="12" fill="#555">▼ <tspan font-weight="700">워커가 1번 작업을 완료</tspan> → 빈 워커가 Piscina 큐에서 7번을 가져감 → p-limit이 13번을 풀어 Piscina 큐로 진입</text>

  <!-- ===== Snapshot 2: 1건 완료 후 ===== -->
  <rect x="0" y="178" width="70" height="70" rx="6" fill="#1976d2"/>
  <text x="35" y="210" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">1건</text>
  <text x="35" y="228" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">완료 후</text>
  <!-- Waiting -->
  <rect x="80" y="178" width="260" height="70" rx="6" fill="#e8f5e9"/>
  <rect x="80" y="178" width="3" height="70" fill="#2e7d32"/>
  <text x="95" y="198" font-size="11" font-weight="600" fill="#666">아직 pool.run() 호출 전</text>
  <text x="95" y="226" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#d32f2f">14</tspan><tspan fill="#222">, 15, 16, ... , 500</tspan></text>
  <!-- Queue -->
  <rect x="350" y="178" width="260" height="70" rx="6" fill="#fff3e0"/>
  <rect x="350" y="178" width="3" height="70" fill="#f57c00"/>
  <text x="365" y="198" font-size="11" font-weight="600" fill="#666">직렬화 후 워커 대기</text>
  <text x="365" y="226" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#222">8, 9, 10, 11, 12, </tspan><tspan fill="#d32f2f">13</tspan></text>
  <!-- Running -->
  <rect x="620" y="178" width="260" height="70" rx="6" fill="#ffebee"/>
  <rect x="620" y="178" width="3" height="70" fill="#c62828"/>
  <text x="635" y="198" font-size="11" font-weight="600" fill="#666">워커에서 처리 중</text>
  <text x="635" y="226" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#222">2, 3, 4, 5, 6, </tspan><tspan fill="#d32f2f">7</tspan></text>

  <!-- Event 2 -->
  <rect x="0" y="260" width="880" height="24" rx="6" fill="#f5f7fa"/>
  <text x="10" y="276" font-size="12" fill="#555">▼ <tspan font-weight="700">워커가 2번 작업을 완료</tspan> → 큐에서 8번 워커로, p-limit에서 14번 큐로</text>

  <!-- ===== Snapshot 3: 2건 완료 후 ===== -->
  <rect x="0" y="296" width="70" height="70" rx="6" fill="#1976d2"/>
  <text x="35" y="328" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">2건</text>
  <text x="35" y="346" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">완료 후</text>
  <rect x="80" y="296" width="260" height="70" rx="6" fill="#e8f5e9"/>
  <rect x="80" y="296" width="3" height="70" fill="#2e7d32"/>
  <text x="95" y="316" font-size="11" font-weight="600" fill="#666">아직 pool.run() 호출 전</text>
  <text x="95" y="344" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#d32f2f">15</tspan><tspan fill="#222">, 16, 17, ... , 500</tspan></text>
  <rect x="350" y="296" width="260" height="70" rx="6" fill="#fff3e0"/>
  <rect x="350" y="296" width="3" height="70" fill="#f57c00"/>
  <text x="365" y="316" font-size="11" font-weight="600" fill="#666">직렬화 후 워커 대기</text>
  <text x="365" y="344" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#222">9, 10, 11, 12, 13, </tspan><tspan fill="#d32f2f">14</tspan></text>
  <rect x="620" y="296" width="260" height="70" rx="6" fill="#ffebee"/>
  <rect x="620" y="296" width="3" height="70" fill="#c62828"/>
  <text x="635" y="316" font-size="11" font-weight="600" fill="#666">워커에서 처리 중</text>
  <text x="635" y="344" font-family="monospace" font-size="13" font-weight="700"><tspan fill="#222">3, 4, 5, 6, 7, </tspan><tspan fill="#d32f2f">8</tspan></text>

  <!-- Event 3 -->
  <rect x="0" y="378" width="880" height="24" rx="6" fill="#f5f7fa"/>
  <text x="10" y="394" font-size="12" fill="#555">▼ ... (488번 작업이 워커로 들어가기까지 계속 반복)</text>

  <!-- ===== Snapshot 4: 488건 완료 후 ===== -->
  <rect x="0" y="414" width="70" height="60" rx="6" fill="#1976d2"/>
  <text x="35" y="441" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">488건</text>
  <text x="35" y="459" font-size="13" font-weight="700" fill="#fff" text-anchor="middle">완료 후</text>
  <rect x="80" y="414" width="260" height="60" rx="6" fill="#e8f5e9"/>
  <rect x="80" y="414" width="3" height="60" fill="#2e7d32"/>
  <text x="95" y="434" font-size="11" font-weight="600" fill="#666">대기열 모두 소진</text>
  <text x="95" y="460" font-family="monospace" font-size="13" font-weight="700" fill="#aaa">(비어있음)</text>
  <rect x="350" y="414" width="260" height="60" rx="6" fill="#fff3e0"/>
  <rect x="350" y="414" width="3" height="60" fill="#f57c00"/>
  <text x="365" y="434" font-size="11" font-weight="600" fill="#666">직렬화 후 워커 대기</text>
  <text x="365" y="460" font-family="monospace" font-size="13" font-weight="700" fill="#222">495, 496, 497, 498, 499, 500</text>
  <rect x="620" y="414" width="260" height="60" rx="6" fill="#ffebee"/>
  <rect x="620" y="414" width="3" height="60" fill="#c62828"/>
  <text x="635" y="434" font-size="11" font-weight="600" fill="#666">워커에서 처리 중</text>
  <text x="635" y="460" font-family="monospace" font-size="13" font-weight="700" fill="#222">489, 490, 491, 492, 493, 494</text>
</svg>
</div>

> **핵심:** 워커가 1건을 끝낼 때마다 ③ → 빠짐, ② → ③ 이동, ① → ② 이동의 한 칸 전진이 반복됩니다. Piscina 큐에 적재되는 각 작업은 워커 전달을 위해 메인 스레드 메모리에 복사본으로 저장됩니다. p-limit 없이 그냥 호출했다면 500건의 작업이 Piscina 큐에 적재되어 그만큼의 복사본이 메모리에 누적되었겠지만, p-limit 덕분에 **무거운 작업은 항상 12건만 진행되고** 나머지 488건은 가벼운 상태로 차례를 기다립니다.

## 5. 전체 처리 흐름 (최종)

```
Phase 1: 데이터 가공 (병렬)

ANSWER 컬렉션 → cursor 500건 읽기 + LIST_{SNUM} 데이터 매칭
                ↓
        ┌─ 현재 500건: 1건씩 Piscina 워커에 전달 → MakeData 가공 → insert 수집
        │
        └─ 다음 500건: 프리페치로 미리 cursor 읽기 + LIST 매칭 (대기)
                ↓
        현재 500건 처리 완료 → DATA_ALL_{SNUM}에 insertMany
                ↓
        대기 중이던 500건이 현재가 됨 → 다음 500건 프리페치 시작 → 반복

        ※ 메모리에는 현재 500건 + 대기 500건 = 최대 1,000건만 존재
        ※ 프리페치로 DB 대기 시간 제거 → 끊김 없이 연속 처리

Phase 2: 엑셀 생성 (직렬)

DATA_ALL_{SNUM} → cursor 1건씩 읽기
                ↓
        숫자 변환 + 셀 스타일 적용
                ↓
        sheet.addRow() → commit() → 디스크에 flush, 메모리 해제
                ↓
        다음 1건 → 반복
                ↓
        workbook.commit() → xlsx 파일 완성 → S3 업로드 / 이메일 발송
```

## 6. 정리하며

코딩을 하면서 메모리를 신경 써본 적이 없었고, OOM 또한 이번에 처음 경험하였습니다. 지금까지 '기능을 구현하는 것'에 집중되어 있었다면, 이번 작업은 평소에 의식하지 못했던 **'시스템 자원'의 중요성**을 피부로 느꼈습니다.

처음 Piscina를 도입해 확연하게 높아진 다운로드 속도에 헬렐레 하고 좋다고 하고 있었는데, OOM이 터진 것을 보고 "아 어디부터 시작해야 하지? 어디부터 어떻게 뭘 처리해야 하지?" 하는 의문 투성이였습니다. 그럴 때 이사님, 차장님께서 힌트를 주시고 갈피를 잡아가면서 이것저것 삽질도 많이 해보고 이렇게 코드가 완성이 되었습니다.

결과가 똑같아 보여도 처리 방식에 따라 시스템의 안정성이 갈린다는 점을 정말 뼈저리게 느꼈고, 무엇보다 이 과정이 누군가의 설명을 듣고 머리로만 이해한 지식이 아니라, 에러를 마주하고 병목을 좁혀가며 쌓은 경험이라는 점에서 의미가 큽니다.

앞으로도 이번 경험을 발판 삼아, 더 안정적이고 효율적인 코드를 만드는 개발자가 되도록 하겠습니다. 감사합니다.
