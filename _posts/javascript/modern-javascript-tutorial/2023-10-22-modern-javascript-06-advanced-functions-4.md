---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 4
date: 2023-10-22 12:06:46 +0900
last_modified_at: 2023-11-08 15:19:42 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

setTimeout과 setInterval을 이용한 호출 스케줄링

## setTimeout과 setInterval을 이용한 호출 스케줄링

호출 스케줄링(scheduling a call)

- 일정 시간이 지난 후에 원하는 함수를 예약 실행(호출)할 수 있게 하는 것

호출 스케줄링을 구현하는 방법

- `setTimeout`을 이용해 일정 시간이 지난 후에 함수를 실행하는 방법
- `setInterval`을 이용해 일정 시간 간격을 두고 함수를 실행하는 방법

자바스크립트 명세서에는 `setTimeout`과 `setInterval`이 명시되어 있지 않음

- 시중에 나와 있는 모든 브라우저, Node.js를 포함한 자바스크립트 호스트 환경 대부분이 이와 유사한 메서드와 내부 스케줄러 지원

### setTimeout

```javascript
let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)
```

- `func|code`: 실행하고자 하는 코드로 함수 또는 문자열 형태
  - 하위 호환성을 위해 문자열도 받을 수 있게 했으나 함수를 사용하는 것을 추천
- `delay`: 실행 전 대기 시간. 단위는 밀리초. 기본값은 0
- `arg1`, `arg2`, ...: 함수에 전달할 인수들

```javascript
function sayHi() {
  alert("안녕하세요.");
}
setTimeout(sayHi, 1000); // 1초 후 sayHi() 호출
```

```javascript
function sayHi(who, phrase) {
  alert(`${who} 님, ${phrase}`);
}
setTimeout(sayHi, 1000, "홍길동", "안녕하세요."); // 홍길동 님, 안녕하세요.
```

```javascript
setTimeout("alert('안녕하세요.')", 1000); // 첫 번째 인수가 문자열이면 함수로 만듦
```

- 위 보다 아래 방법을 권장

```javascript
setTimeout(() => alert("안녕하세요."), 1000);
```

함수를 넘기지 않고 함수 실행 결과를 넘기는 것을 주의

### clearTimeout으로 스케줄링 취소하기

`setTimeout`을 사용하면 타이머 식별자(timer identifier)가 반환됨

- 스케줄링을 취소하고 싶을 때 식별자 사용

```javascript
let timerId = setTimeout(...);
clearTimeout(timerId);
```

```javascript
let timerId = setTimeout(() => alert("아무 일도 일어나지 않습니다."), 1000);
alert(timerId); // 1338
clearTimeout(timerId);
alert(timerId); // 1338. 타이머 식별자는 변함 없음(null이 되지 않음)
```

Node.js에서 `setTimeout`을 실행하면 타이머 객체가 반환됨

스케줄링에 관한 명세는 따로 존재하지 않기 때문에 호스트 환경마다 약간의 차이 존재

- 브라우저는 HTML5의 timers section을 준수

### setInterval

```javascript
let timerId = setInterval(func|code, [delay], [arg1], [arg2], ...)
```

`setInterval`은 함수를 주기적으로 실행

- `setTimeout`은 함수를 단 한번만 실행

함수 호출을 중단하려면 `clearInterval(timerId)`을 사용

```javascript
let timerId = setInterval(() => alert("째깍"), 2000);
setTimeout(() => {
  clearInterval(timerId);
  alert("정지");
}, 5000);
```

- 대부분의 브라우저는 `alert`, `confirm`, `prompt` 창이 떠 있는 동안에도 내부 타이머를 멈추지 않음

### 중첩 setTimeout

무언가를 일정 간격을 두고 실행하는 방법

- `setInterval`을 이용하는 방법
- 중첩 `setTimeout`을 이용하는 방법

```javascript
// let timerId = setInterval(() => alert("째깍"), 2000);
let timerId = setTimeout(function tick() {
  alert("째깍");
  timerId = setTimeout(tick, 2000);
}, 2000);
```

중첩 `setTimeout`을 이용하는 방법은 `setInterval`을 이용하는 방법보다 유연

CPU 소모가 많은 작업을 주기적으로 실행하는 경우에도 `setTimeout`을 재귀 실행하는 방법이 유용

- 작업에 걸리는 시간에 따라 다음 작업을 유동적으로 계획할 수 있음

중첩 `setTimeout`을 이용하는 방법은 지연 간격을 보장하지만, `setInterval`은 이를 보장하지 않음

```javascript
let i = 1;
setInterval(function () {
  func(i++);
}, 100);
```

- 내부 스케줄러가 `func(i++)`를 100ms마다 실행
- `setInterval`을 사용하면 `func` 호출 사이의 지연 간격이 실제 명시한 간격(100ms)보다 짧아짐
- `func`을 실행하는 데 소모되는 시간도 지연 간격에 포함시키기 때문
- `func`을 실행하는 데 걸리는 시간이 명시한 지연 간격보다 길면 엔진이 `func`의 실행이 종료될 때까지 기다려주고 `func`의 실행이 종료되면 엔진은 스케줄러를 확인하고 지연 시간이 지났으면 다음 호출을 바로 시작
- 함수 호출에 걸리는 시간이 매번 `delay`ms보다 길면 모든 함수가 쉼 없이 계속 연속 호출됨

```javascript
let i = 1;
setTimeout(function run() {
  func(i++);
  setTimeout(run, 100);
}, 100);
```

- 명시한 지연(100ms)이 보장됨
- 이전 함수의 실행이 종료된 이후에 다음 함수 호출에 대한 계획이 세워지기 때문

가비지 컬렉션과 setInterval-setTimeout

- `setInterval`이나 `setTimeout`에 함수를 넘기면 함수에 대한 내부 참조가 새로 만들어지고 이 참조 정보는 스케줄러에 저장됨
- 따라서 해당 함수를 참조하는 것이 없어도 `setInterval`과 `setTimeout`에 넘긴 함수는 가비지 컬렉션의 대상이 되지 않음

```javascript
setTimeout(function () {...}, 100); // 스케줄러가 함수를 호출할 때까지 함수는 메모리에 유지됨
```

- `setInterval`의 경우는 `clearInterval`이 호출되기 전까지 함수에 대한 참조가 메모리에 유지됨
- 외부 렉시컬 환경을 참조하는 함수가 있을 때, 이 함수가 메모리에 남아있는 동안엔 외부 변수 역시 메모리에 남아있음
- 이렇게 되면 실제 함수가 차지했어야 하는 공간보다 더 많은 메모리 공간이 사용됨
- 따라서 스케줄링할 필요가 없어진 함수는 취소하는 것을 권장

### 대기 시간이 0인 setTimeout

`setTimeout(func, 0)`, `setTimeout(func)`를 사용하면 대기 시간을 0으로 설정할 수 있음

- `func`을 가능한 한 빨리 실행 가능
- 이때 스케줄러는 현재 실행 중인 스크립트의 처리가 종료된 이후에 스케줄링한 함수를 실행

```javascript
setTimeout(() => alert("World")); // 2
alert("Hello"); // 1
```

브라우저 환경에서 실제 대기 시간은 0이 아님

- 브라우저는 HTML5 표준에서 정한 중첩 타이머 실행 간격 관련 제약을 준수
  - 다섯 번째 중첩 타이머 이후엔 대기 시간을 최소 4ms 이상으로 강제해야 한다는 제약이 명시됨

```javascript
let start = Date.now();
let times = [];
setTimeout(function run() {
  times.push(Date.now() - start); // 이전 호출이 끝난 시점과 현재 호출이 시작된 시점의 시차
  if (start + 100 < Date.now()) alert(times);
  else setTimeout(run);
});
// 0,0,0,0,6,10,15,20,25,30,35,41,46,51,56,61,66,73,77,82,87,93,98,103
```

- 다섯 번째 중첩 타이머 이후엔 지연 간격이 4ms 이상 됨
- `setInterval`에도 적용됨
  - 처음 몇 번은 지연 없이 실행하지만 나중에는 지연 간격을 4ms 이상으로 늘림
- 오래 전부터 있던 제약
- 서버 측에는 이런 제약이 없음(위 제약은 브라우저에 한정)
  - Node.js의 `process.nextTick`과 `setImmediate`를 이용하면 비동기 작업을 지연 없이 실행 가능

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
