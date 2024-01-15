---
title: 모던 JavaScript 튜토리얼 20 - 기타 2
date: 2024-01-11 09:07:43 +0900
last_modified_at: 2024-01-15 23:41:59 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

이벤트 루프와 매크로태스크, 마이크로태스크

## 이벤트 루프와 매크로태스크, 마이크로태스크

브라우저 측 자바스크립트 실행 흐름

- Node.js와 마찬가지로 이벤트 루프에 기반함

### 이벤트 루프

이벤트 루프(event loop)

- 태스크가 들어오길 기다리고, 들어오면 처리하고, 없으면 잠듦
- 끊임 없이 돌아가는 자바스크립트 내 루프

자바스크립트 엔진이 돌아가는 알고리즘

- 하나의 집합을 이루고 있는 태스크들을 차례대로 처리하고 새로운 태스크가 추가될 때까지 기다림
  1. 처리해야 할 태스크가 있다면 먼저 들어온 태스크부터 순차적으로 처리
  2. 처리해야 할 태스크가 없다면 잠들어 있다가 새로운 태스크가 추가되면 다시 1로 돌아감
- 자바스크립트 엔진은 대부분의 시간 동안 아무런 일도 하지 않고 쉬고 있음
  - 기다리는 동안은 CPU 자원 소비가 0에 가까워짐
  - 스크립트나 핸들러, 이벤트가 활성화될 때만 돌아감
- 새로운 태스크는 엔진이 바쁠 때 추가될 수 있고, 이 태스크는 큐에 추가됨
- 브라우저로 인터넷을 서핑할 때 사용되는 알고리즘

자바스크립트 엔진을 활성화하는 태스크

- 외부 스크립트 `<script src="...">`가 로드될 때, 이 스크립트를 실행하는 것
- 사용자가 마우스를 움직일 때, `mousemove` 이벤트와 이벤트 핸들러를 실행하는 것
- `setTimeout`에서 설정한 시간이 다 된 경우, 콜백 함수를 실행하는 것
- 기타 등등

매크로태스크 큐(macrotask queue)

- V8 용어로, 태스크가 추가되는 큐
- script, mousemove, setTimeout, ...

```
 ----->-----
/           \
event     macrotask
loop      queue
\           /
 -----<-----
```

구체적인 사례

- 엔진이 `script`를 처리하느라 바쁨
- 사용자가 마우스를 움직여 `mousemove` 이벤트 활성화
- 바로 이어서 `setTimeout`에서 설정한 시간이 지남
- 세 태스크는 큐에 하나씩 추가되고 큐에 있는 태스크들은 들어간 순서대로 처리됨
  - `script` -> `mousemove` -> `setTimeout`

두 가지 세부 사항

1. 엔진이 특정 태스크를 처리하는 동안에는 렌더링이 절대 일어나지 않음
   - 태스크를 처리하는 데 걸리는 시간이 길지 않으면 문제되지 않음
   - 처리가 끝나는 대로 DOM 변경을 화면에 반영하면 됨
2. 태스크 처리가 오래 걸리면, 브라우저는 태스크를 처리하는 동안에 발생한 사용자 이벤트 등의 새로운 태스크들을 처리하지 못함
   - 인터넷 서핑 시 응답 없는 페이지(Page Unresponsive) 얼럿 창을 만나게 되는 경우
   - 아주 복잡한 계산이 필요하거나, 프로그래밍 에러 때문에 무한 루프에 빠지게 될 때 해당 얼럿 창이 나타남
   - 브라우저는 얼럿 창을 통해 사용자에게 페이지 전체와 함께 해당 태스크를 취소시킬지 말지를 선택하도록 유도

### 유스 케이스 1: CPU 소모가 많은 태스크 쪼개기

- 전체 실행 시간을 많이 희생하지 않으면서도 사용자와의 상호작용에 막힘이 없어짐

태스크를 쪼개지 않은 경우

```javascript
let [i, start] = [0, Date.now()];
function count() {
  for (let j = 0; j < 1e9; j++) i++; // CPU 소모가 많은 무거운 작업
  alert(`처리에 걸린 시간: ${Date.now() - start}ms`); // 1289ms
}
count(); // 엔진이 몇 초간 멈추고 브라우저라면 상호작용이 지연됨
```

중첩 `setTimeout`으로 태스크를 쪼개 문제 예방

```javascript
let [i, start] = [0, Date.now()];
function count() {
  do i++;
  while (i % 1e6 != 0); // 무거운 작업을 쪼개 count 태스크 일부 처리
  if (i == 1e9) alert(`처리에 걸린 시간: ${Date.now() - start}ms`); // 6505ms
  else setTimeout(count); // 카운팅이 끝나지 않았다면 새로운 호출을 스케줄링
}
count(); // 숫자를 세는 도중에도 브라우저가 제 기능을 다할 수 있음
```

- `onclick` 이벤트 같은 새로운 태스크가 생기면 큐에 들어감
  - 부분 카운팅이 끝난 후 다음 카운팅이 시작되기 전에 처리
- 부분 카운팅 실행 중간 중간에 '환기'를 해줘 이벤트 루프가 돌아갈 수 있게 됨
  - 사용자 이벤트에 반응하면서 무거운 태스크 처리 가능

코드를 다듬어 시간차 줄이기

```javascript
let [i, start] = [0, Date.now()];
function count() {
  if (i < 1e9 - 1e6) setTimeout(count); // 앞에서 새로운 호출을 스케줄링
  do i++;
  while (i % 1e6 != 0);
  if (i == 1e9) alert(`처리에 걸린 시간: ${Date.now() - start}ms`); // 4707ms
}
count();
```

- `count()`가 호출되고 아직 원하는 숫자를 다 세지 못한 경우, 부분 카운팅 시작 전에 부분 카운팅 재스케줄링이 이뤄짐
- 중첩 `setTimeout` 호출이 많은 경우, 브라우저 최소 대기 시간이 `4ms`가 됨
  - 코드 상으로 대기 시간이 `0`이더라도 실제 대기시간은 `4ms` 이상이 됨
- 숫자를 세기 전에 스케줄링 하면 숫자를 세면서 대기 시간을 소모할 수 있어 더 빨리 실행

### 유스 케이스 2: 프로그레스 바

- 태스크를 여러 개로 쪼개면, 진행 상태를 나타내주는 프로그레스 바(progress bar)를 만들 때도 좋음

브라우저 동작 방식

- 시간이 오래 걸리든 아니든 상관 없이 현재 작업 중인 태스크가 끝나야 DOM 변경분을 화면에 렌더링
- 완성되지 않은 '중간' 상태의 화면이 사용자에게 노출되는 것을 막아줌
- 요소를 여러 개 만들고, 이 요소들을 하나씩 화면에 추가한 다음, 원하는 요소의 스타일을 변경시키는 함수가 있는 경우
  - 이 함수를 실행하는 동안에 변경사항 모두가 사용자에게 노출된다면 사용자는 혼란을 느낄 수 있음

```html
<div id="progress"></div>
<script>
  function count() {
    for (let i = 0; i < 1e6; i++) {
      i++;
      progress.innerHTML = i;
    }
  }
  count();
</script>
```

- 함수가 끝날 때까지 사용자는 `i`가 변하는 것을 볼 수 없음
- 화면에는 마지막 상태만 출력

```html
<div id="progress"></div>
<script>
  let i = 0;
  function count() {
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);
    if (i < 1e7) setTimeout(count); // 태스크 쪼개기
  }
  count();
</script>
```

- 작업 진척 상태를 보여주는 프로그레스 바 등의 인디케이터를 만들어야 하는 경우
- `setTimeout`으로 태스크를 여러 개로 쪼개면 서브 태스크 중간마다 상태 변화를 볼 수 있음
- 프로그레스 바처럼 `<div>`에 `i`가 변하는 과정 출력

### 유스 케이스 3: 이벤트 처리가 끝난 이후에 작업하기

이벤트 버블링이 끝나 모든 DOM 트리 레벨에서 이벤트가 핸들링 될 때까지 특정 액션을 연기시켜야 하는 경우

- 연기시킬 액션 관련 코드를 지연 시간이 0인 `setTimeout`으로 감싸 원하는 동작 구현

```javascript
menu.onclick = function () {
  // ...
  let customEvent = new CustomEvent("menu-open", { bubbles: true }); // 클릭한 메뉴 내 항목 정보가 담긴 커스텀 이벤트 생성
  setTimeout(() => menu.dispatchEvent(customEvent)); // 비동기로 커스텀 이벤트를 디스패칭
};
```

### 매크로태스크와 마이크로태스크

태스크(task)

- 매크로태스크(macrotask)
- 마이크로태스크(microtask)

마이크로태스크(microtask)

- 코드를 통해서만 생성되는데, 주로 프라미스를 사용해 생성
- `.then/catch/finally` 핸들러가 마이크로태스크가 됨
- `await`를 사용해도 생성 가능
- 표준 API인 `queueMicrotask(func)`로 함수 `func`를 마이크로태스크 큐에 넣어 처리 가능
- 자바스크립트 엔진은 매크로태스크 하나를 처리할 때마다 또 다른 매크로태스크나 렌더링 작업을 하기 전에,
- 마이크로태스크 큐에 쌓인 마이크로태스크 전부를 처리

```javascript
setTimeout(() => alert("timeout")); // 3
Promise.resolve().then(() => alert("promise")); // 2
alert("code"); // 1
```

1. `code`: 일반적인 동기 호출로 가장 먼저 실행
2. `promise`: 현재 코드(`alert("code")`)가 실행되고 난 후에 실행
   - `.then`은 마이크로태스크 큐에 들어가 처리되기 때문
3. `timeout`: 가장 마지막에 출력
   - `setTimeout`에서 설정한 시간이 끝난 후 콜백 함수를 실행하는 것은 매크로태스크

매크로태스크(macrotask)

- 다른 이벤트 핸들러나 렌더링 작업, 혹은 다른 매크로태스크가 실행되기 전에 처리

```
        --->---
       /       \
      |         ...
      |         [ script ]
      |           microtask
event |           render
loop  |         [ mousemove ]
      |           microtask
      |           render
      |         [ setTimeout ]
      |         ...
       \       /
        ---<---
```

- 매크로태스크 하나 처리
  - script, mousemove(evnet), setTimeout 등
- 마이크로태스크 전부 처리
- 렌더링 진행
- 반복

이런 처리 순서가 중요한 이유

- 애플리케이션 환경에 변화를 주는 작업에 영향을 받지 않고, 모든 마이크로태스크를 동일한 환경에서 처리 가능
  - 마우스 좌표 변경, 네트워크 통신에 의한 데이터 변경 같은 작업
- 그런데 직접 만든 함수를 현재 코드 실행이 끝난 후, 새로운 이벤트 핸들러가 처리되기 전이면서 렌더링이 실행되기 전에 비동기적으로 실행해야 하는 경우가 생김
- 이럴 때 `queueMicrotask`를 사용해 커스텀 함수를 스케줄링

```html
<div id="progress"></div>
<script>
  let i = 0;
  function count() {
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);
    if (i < 1e6) queueMicrotask(count); // setTimeout 대신 queueMicrotask로 함수 count 재스케줄링
  }
  count();
</script>
```

- 동기 코드처럼 카운팅이 다 끝났을 때 숫자가 렌더링 됨

### 요약

이벤트 루프 알고리즘

1. 매크로태스크 큐에서 가장 오래된 태스크를 꺼내 실행
2. 마이크로태스크 큐가 빌 때까지 모든 마이크로태스크를 오래된 순서대로 실행
3. 렌더링할 것이 있으면 처리
4. 매크로태스크 큐가 비어있으면 새로운 매크로태스크가 나타날 때까지 대기
5. 1번으로 돌아감

새로운 마이크로태스크를 스케줄링하는 방법

- `queueMicrotask(f)` 사용
- 이외에도 프라미스 핸들러는 마이크로태스크 큐에 들어가 처리됨
- 마이크로태스크 전체가 처리되는 동안에는 UI 변화나 네트워크 이벤트 핸들링이 일어나지 않음
- 렌더링이나 네트워크 요청 등의 작업들은 마이크로태스크 전부가 처리되고 난 직후 처리됨
- 이런 처리 순서 덕분에 `queueMicrotask`를 사용해 함수를 비동기적으로 처리할 때 애플리케이션 상태의 일관성이 보장됨

새로운 매크로태스크를 스케줄링하는 방법

- 지연시간이 0인 `setTimeout(f)` 사용
- 계산이 복잡한 태스크 하나를 여러 개로 쪼갤 수 있음
- 태스크 중간중간 사용자 이벤트에 반응
- 작업 진척 상태 화면에 표시 가능
- 이벤트가 완전히 처리되고 난 후(버블링이 끝난 후)에 특정 작업을 수행하도록 스케줄링할 때도 사용

웹 워커(Web Worker)

- 이벤트 루프를 막을 우려가 있는 무거운 연산 처리에 사용
- 별도의 백그라운드 스레드에서 코드를 병렬적으로 실행 가능
- 메인 스레드와 메시지를 교환할 수 있기는 하지만 웹 워커에는 메인 스레드와 연관 없는 고유 변수들과 자체 이벤트 루프가 있음
- 웹 워커는 DOM에 접근할 수 없기 때문에 여러 CPU 코어를 동시에 사용해야 하는 연산에 주로 사용

마이크로태스크 큐, 잡 큐

매크로태스크 큐, 이벤트 큐, 콜백 큐

## 참고

> [기타](https://ko.javascript.info/ui-misc)

> [[번역] 자바스크립트 이벤트 루프](https://chanmi-lee.github.io/articles/2020-06/JavaScript-Visualized-Event-Loop)

> [자바스크립트와 이벤트 루프](https://meetup.nhncloud.com/posts/89)

> [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

> [Philip Roberts Help I'm stuck in an event loop](https://www.youtube.com/watch?v=6MXRNXXgP_0)

> [어쨌든 이벤트 루프는 무엇입니까? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

> [Difference between microtask and macrotask within an event loop context](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)

> [Nodejs Event Loop](https://stackoverflow.com/questions/10680601/nodejs-event-loop)
