---
title: 비활성화된 브라우저 탭에서의 타이머 API 동작
date: 2024-05-07 15:27:44 +0900
last_modified_at: 2024-05-08 08:12:07 +0900
categories: [JavaScript]
tags: [javascript, visibilitychange]
---

브라우저 탭이 비활성화 되었을 때 setTimeout, setInterval은 어떻게 동작할까요?

## 비활성화된 브라우저 탭에서 실행 중인 setTimeout, setInterval 타이머의 동작

비활성 브라우저 탭에서 실행되는 `setTimeout()`/`setInterval()` 메서드의 시간 지연은 코드에 정의된 값에 관계 없이 1초로 설정됨

백그라운드 탭에서 실행되는 타이머 메서드 `setTimeout()`/`setInterval()`은 리소스를 많이 소모할 수 있음

백그라운드 탭에서 매우 짧은 간격으로 콜백을 실행하는 애플리케이션은 현재 활성 상태인 탭의 작업에 영향을 미칠 정도로 많은 메모리를 소모할 수 있음

최악의 경우 브라우저가 충돌하거나 디바이스의 배터리가 빠르게 소모될 수 있음

이러한 경우 백그라운드 탭에서 실행되는 타이머에 몇 가지 제한을 두는 것이 좋음

브라우저는 정확히 이 작업을 수행

비활성 탭의 경우 코드에 지정된 원래 지연에 관계없이 1초마다 실행되도록 타이머를 자동으로 조절

예

코드에서 50ms마다 일부 코드를 실행하기 위해 `setInterval()`을 사용한 경우 애플리케이션이 백그라운드 탭으로 이동하면 자동으로 간격이 1000ms가 됨

그러나 탭을 다시 활성화라면 원래 간격 또는 지연이 원래 값으로 돌아감

21년 1월 업데이트: Chrome 88+부터는 특정 조건에서 타이머를 1분으로 조절할 수 있음

### 타이머 스로틀링으로 인해 버그가 발생하지 않도록 하기

사용자가 다른 탭으로 이동하거나 브라우저를 최소화할 때 애플리케이션의 타이머가 스로틀될 수 있는 시나리오를 고려하는 것이 중요

예

애플리케이션에서 `setInterval()`을 사용하여 애니메이션을 실행하는 경우 사용자가 원래 탭으로 돌아갈 때 왜곡된 형태의 애니메이션이 표시될 수 있음

이러한 왜곡은 몇 ms마다 실행되어야 하는 코드가 탭이 비활성 상태인 동안 1초마다 실행되었기 때문에 발생할 수 있음

Page Visibility API를 사용하면 사용자가 탭에서 다른 곳으로 이동했는지 또는 다시 탭으로 돌아왔는지 확인 가능

이를 파악하면 탭이 비활성 상태일 때 타이머를 일시적으로 일시중지할 수 있음

`visibilitychange` 이벤트

```javascript
document.addEventListener("visibilitychange", function () {
  if (document.hidden) {
    // tab is now inactive
    // temporarily clear timer using clearInterval() / clearTimeout()
  } else {
    // tab is active again
    // restart timers
  }
});
```

일부 사이트의 경우 아래의 간단한 최적화로 CPU 사용량을 최대 75%까지 줄일 수 있음

```javascript
var doVisualUpdates = true;

document.addEventListener("visibilitychange", function () {
  doVisualUpdates = !document.hidden;
});

function update() {
  if (!doVisualUpdates) {
    return;
  }
  doStuff();
}
```

`requestAnimationFrame()`

- 페이지가 백그라운드에 있을 때 Chrome은 `requestAnimationFrame()`을 호출하지 않음

## 참고

[What Happens to setTimeout() / setInterval() Timers Running on Inactive Browser Tabs ?](https://usefulangle.com/post/280/settimeout-setinterval-on-inactive-tab)

[Chrome 57의 백그라운드 탭](https://developer.chrome.com/blog/background_tabs?hl=ko)

[Page Visibility API](https://developer.mozilla.org/ko/docs/Web/API/Page_Visibility_API)
