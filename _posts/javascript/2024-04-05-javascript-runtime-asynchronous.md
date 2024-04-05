---
title: JavaScript 런타임의 비동기 제어
date: 2024-04-05 15:25:07 +0900
last_modified_at: 2024-04-05 15:25:07 +0900
categories: [JavaScript]
tags:
  [
    javascript,
    runtime,
    browser,
    call-stack,
    web-apis,
    task-queue,
    event-loop,
    microtask-queue
  ]
---

JavaScript 런타임에서 비동기를 다루는 호출 스택, 웹 API, 태스크 큐, 이벤트 루프, 마이크로태스크 큐에 대해 알아보겠습니다.

## JavaScript 런타임의 비동기 제어

JavaScript Runtime

- Call Stack
- Web APIs
- Event Loop
- Task Queue
- Microtask Queue

### 호출 스택(Call Stack)

- 호출 스택은 프로그램 실행을 관리
- 함수를 호출하면, 새로운 실행 컨텍스트가 생성되어 호출 스택에 푸시
- 함수가 실행을 완료하면, 실행 컨텍스트가 호출 스택에서 팝

JavaScript

- JavaScript는 단일 스레드로, 단일 Call Stack으로 작업
- 즉, 한 번에 하나의 작업만 처리 가능
- 한 작업이 오래 실행되면, 다른 작업의 실행을 차단해 프로그램이 정지된 것처럼 보임

```javascript
fetch("https://website.com/api/posts"); // 데이터를 반환할 때까지 호출 스택에 머무르면 다른 작업 실행을 막게 되나?
```

- `fetch` 등의 API를 사용해 서버에서 데이터가 언제 반환되는지 알 수 없을 때, 데이터가 반환될 때까지 호출 스택에 머무르지 않아도 됨
- 위 기능은 자바스크립트 자체의 일부가 아니고 웹 API를 통해 제공됨

### 웹 API(Web APIs)

- 브라우저가 활용하는 기능과 상호작용할 수 있는 인터페이스 세트를 제공
- JavaScript로 구축할 때 자주 사용하는 기능이 포함
- DOM, `fetch`, `setTimeout`, ...
- JavaScript 런타임과 브라우저 기능 사이의 브리지 역할
  - JavaScript 자체 기능 이상의 기능을 사용할 수 있게 함
- 비동기 작업을 시작하고 오래 실행되는 작업을 브라우저로 오프로드할 수 있게 해줌
- 이러한 API에 의해 노출된 메서드를 호출하는 것은 실제로 장기 실행 작업을 브라우저 환경으로 오프로드하고 이 작업의 최종 완료를 처리하도록 핸들러를 설정하는 것
- 결과를 기다리지 않고 비동기 작업을 시작한 후 실행 컨텍스트가 호출 스택에서 빠르게 사라질 수 있음
  - 논 블로킹(non-blocking)
- 비동기 기능을 노출하는 웹 API는 콜백 기반 또는 프라미스 기반 접근 방식을 사용
- 비동기 작업을 시작하는 함수 호출은 호출 스택에 추가되지만, 이는 브라우저에 전달하기 위한 것
  - 실제 비동기 작업은 백그라운드에서 처리되며 호출 스택에 남아있지 않음

브라우저

- 수많은 기능을 활용하는 강력한 플랫폼
- 컨텐츠를 표시하기 위한 렌더링 엔진
- 네트워크 요청을 위한 네트워킹 스택
- 센서, 파일 시스템, 카메라 등

콜백 기반 APIs

- 이 함수를 호출하면 새로 생성된 실행 컨텍스트가 호출 스택에 푸시
- 웹 API에 대한 콜백을 등록한 다음 작업을 브라우저로 오프로드하는 것임
- 그 후 함수는 호출 스택에서 팝되고 이제부터 브라우저의 책임
- 모든 작업은 백그라운드에서 진행되므로 호출 스택은 계속해서 다른 작업을 수행하고 실행 가능
- 작업이 완료된 후 콜백을 단순히 호출 스택으로 바로 푸시할 수는 없음
- 그렇게 하면 이미 실행 중인 작업이 잠재적으로 중단되어 예측할 수 없는 동작과 잠재적인 충돌이 발생할 수 있기 때문
- JavaScript 엔진은 한 번에 하나씩 작업을 처리할 수 있으므로 예측 가능하고 체계적인 실행 환경을 보장

### 태스크 큐(Task Queue)

- 대신에 콜백이 태스크 큐(콜백 큐(Callback Queue))에 추가됨
- 태스크 큐는 미래의 특정 시점에 실행되기를 기다리는 웹 API 콜백 및 이벤트 핸들러를 보유
- 콜백 기반 웹 API에서 비동기 작업이 완료된 후 콜백을 대기열에 추가하는 데 사용됨

### 이벤트 루프(Event Loop)

- 호출 스택이 비어있는지 지속적으로 확인
- 호출 스택이 비어 있을 때마다(현재 실행 중인 작업이 없음을 의미) 태스크 큐에서 사용 가능한 첫 번째 작업을 가져와 이를 콜백이 실행되는 호출 스택으로 옮김
- 이벤트 루프는 호출 스택이 비어있는지 지속적으로 확인하고, 비어 있으면 태스크 큐에서 사용 가능한 첫 번째 작업을 확인한 후 이를 호출 스택으로 이동하여 실행함

### 마이크로태스크 큐(Microtask Queue)

- 태스크 큐보다 우선순위가 높은 런타임의 또 다른 큐
- 프라미스 핸들러 콜백 `then(callback)`, `catch(callback)`, `finally(callback)`
- `await` 후 `async` 함수 본문 실행
- `MutationObserver` 콜백
- `queueMicrotask` 콜백
- 호출 스택이 비어 있으면, 이벤트 루프는 태스크 큐로 이동하기 전에 먼저 마이크로태스크 큐의 모든 작업을 처리
- 태스크 큐에서 단일 작업을 완료하고 호출 스택이 비어 있으면, 이벤트 루프는 다음 작업으로 다시 이동하기 전에 마이크로태스크 큐의 모든 마이크로태스크를 처리하여 효과적으로 "다시 시작"(starts over)됨
- 이를 통해 방금 완료된 작업관 관련된 마이크로태스크가 즉시 처리되어 프로그램의 응답성과 일관성이 유지됨
- 마이크로태스크는 다른 마이크로태스크를 예약할 수도 있음
  - 이는 무한 마이크로태스크 루프를 생성하여 태스크 큐를 무한정 지연하고, 프로그램의 나머지 부분을 정지시킬 수 있어 주의해야 함
  - 이러한 시나리오는 태스크큐에서는 발생할 수 없음
  - 이벤트 루프는 태스크 큐의 작업을 하나씩 처리한 다음, 마이크로태스크 큐를 확인하여 "다시 시작"함

모든 웹 API는 비동기적으로 처리되나?

- 아님. 비동기 작업을 시작하는 작업만 가능
- `document.getElementById()` 또는 `localStorage.setItem()` 등은 동기식으로 처리됨

### 요약

호출 스택이 비어있음

이벤트 루프는 마이크로태스크 큐에서 작업을 이동 (완전히 빌 때까지)

그런 다음 태스크 큐로 이동해 사용 가능한 첫 번째 작업을 호출 스택으로 이동

사용 가능한 첫 번째 작업을 처리한 후, 마이크로태스크 큐를 확인하여 "다시 시작"함

매우 작은 지연으로 `setTimeout` 대신 `queueMicrotask`를 사용

- 지연 없이 마이크로태스크를 예약하므로 `setTimeout` 보다 더 안정적이고 효율적

## 참고

> [JavaScript Visualized: Event Loop, Web APIs, (Micro)task Queue](https://www.lydiahallie.com/blog/event-loop)

> [⭐️🎀 JavaScript Visualized: Promises & Async/Await](https://dev.to/lydiahallie/javascript-visualized-promises-async-await-5gke)

> [Difference between microtask and macrotask queue in the event loop](https://dev.to/jeetvora331/difference-between-microtask-and-macrotask-queue-in-the-event-loop-4i4i)
