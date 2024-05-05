---
title: 애니메이션 렌더링과 requestAnimationFrame
date: 2024-05-04 12:27:16 +0900
last_modified_at: 2024-05-04 12:27:16 +0900
categories: [JavaScript]
tags: [javascript, requestanimationframe, cancelanimationframe]
---

requestAnimationFrame

## window.requestAnimationFrame

브라우저에게 수행하기를 원하는 애니메이션을 알리고, 다음 리페인트 바로 전에 브라우저가 애니메이션을 업데이트할 지정된 함수를 호출하도록 요청

다음 리페인트에서 다른 프레임을 애니메이션 적용하려는 경우

- 콜백 루틴이 반드시 스스로 `requestAnimationFrame()`을 호출해야 함
- `requestAnimationFrame()`은 one-shot

화면에 애니메이션을 업데이트할 준비가 될 때마다 이 메서드를 호출해야 함

- 이는 브라우저가 다음 리페인트를 수행하기 전에 애니메이션 함수를 호출하도록 요청함
- 콜백의 수는 보통 1초에 60회
- 일반적으로 대부분의 웹 브라우저에서는 W3C 권장사항에 따라 디스플레이 주사율과 일치됨
- 120hz 디스플레이도 브라우저가 알아서 지원

배터리 수명과 성능 향상을 위해 대부분의 브라우저에서 백그라운드 탭들이나 숨겨진 `<iframe>`들이 실행될 때 중단됨

콜백 메서드에는 단일 인수 `DOMHighResTimeStamp`가 전달되어 현재 시간을 나타냄

- `requestAnimationFrame()`에 의해 대기 중인 다수의 콜백이 한 프레임에서 실행되기 시작할 때, 각각의 콜백은 모든 이전 콜백의 작업량을 계산하는 동안 시간이 지나갔음에도 불구하고 동일한 타임스탬프를 받음
- 이 타임스탬프는 밀리초 단위의 십진수이지만 최소 정밀도는 1ms
- 항상 프레임에서 얼마나 많은 애니메이션이 진행될 것인지 계산하기 위해 첫 번째 인수(혹은 현재 시간을 가질 수 있는 몇몇 다른 메서드)를 사용해야 함
- 그렇지 않으면 높은 주사율에서 애니메이션이 빠르게 실행될 것

```javascript
requestAnimationFrame(callback);
```

- `callback`: 다음 리페인트를 위한 애니메이션을 업데이트할 때 호출할 함수
- 콜백 함수에는 `requestAnimationFrame()`이 콜백 함수들의 실행을 시작할 시점을 나타내는 `DOMHighResTimeStamp` 단일 인수가 전달됨
- 반환값으로 콜백 목록의 항목을 고유하게 식별하는 요청 id 정수값을 반환하고, 이 값을 `window.cancelAnimationFrame()`에 전달해 콜백 요청 취소 가능
- 시스템이 프레임을 그릴 준비가 되면 애니메이션 프레임을 호출해 애니메이션 웹페이지를 원활하고 효율적으로 생성
- 실제 화면이 갱신되어 표시되는 주기에 따라 함수를 호출하기 때문에 자바스크립트가 프레임 시작 시 실행되도록 보장해줘 밀림 현상을 방지
- `requestAnimationFrame()`이 실행되면 브라우저는 다음 프레임이 그려지기 전에 함수를 실행하도록 예약하기 때문에 각 프레임이 정확히 16.6ms 간격으로 렌더링됨

```javascript
cancelAnimationFrame(requestID);
```

Animation Frames

- `requestAnimationFrame()` 콜백은 Animation frames 큐에 들어가 처리됨
- 한 tick에 큐에 있던 모든 작업을 실행하고, 같은 tick에 추가된 작업은 실행되지 않고 다음 tick을 기다리며 큐에 남음
- 브라우저의 CPU, GPU 사용량에 따라 콜백 함수 실행이 지연될 수 있음

Task Queue (1개) -> Microtask Queue (전부) -> Animation Frames (있던 콜백만) -> 렌더링 -> Task Queue -> ...

## 브라우저 프레임

60hz, 60fps

- hz: 1초 동안 모니터 화면의 출력 빈도
- fps: frame per second
- 사람의 눈은 1초에 60번 장면이 넘어가야 부드럽다고 느낌
- 초당 60개의 프레임을 렌더링해야 함
- 즉, `16.667ms` (1000ms / 60) 간격으로 프레임 생성 필요

이것을 16ms마다 JS 코드를 호출하는 식으로 구현할 수 있음

## 타이머 함수를 이용한 애니메이션 스크립팅

```javascript
// setInterval 사용
const performAnimationWithInterval = () => {
  // ..스타일 조정 스크립트..
};
setInterval(performAnimationWithInterval, 1000 / 60);

// setTimeout 사용
const performAnimationWithTimeout = () => {
  // ..스타일 조정 스크립트..
  setTimeout(performAnimationWithTimeout, 1000 / 60);
};
setTimeout(performAnimationWithTimeout, 1000 / 60);
```

타이머 API의 문제점

- 프레임을 신경 쓰지 않고 주어진 시간 내 동작
- 타이머 함수는 프레임 단위의 프레임 시작 시간에 맞춰 실행됨을 보장하지 않음

16ms 간격으로 프레임을 생성하지 않고 브라우저가 다른 작업을 수행

- 콜백이 프레임 16ms 단위 중간에서 호출
- 리플로우 발생
- 브라우저 렌더링 단계가 다시 발생함 (리플로우, 리페인트)
- 프레임이 생성되지 않고 누락되어 결국 프레임 드랍이 발생해 화면이 버벅거려 사용자 경험 저하

`setInterval`과 달리 백그라운드에서 실행이 중단됨

## 참고

[Window: requestAnimationFrame() method](https://developer.mozilla.org/ko/docs/Web/API/window/requestAnimationFrame)

[🌐 웹 애니메이션 최적화 requestAnimationFrame 가이드](https://inpa.tistory.com/entry/🌐-requestAnimationFrame-가이드)

[자바스크립트 애니메이션 - requestAnimationFrame 활용하기](https://simsimjae.tistory.com/402)
