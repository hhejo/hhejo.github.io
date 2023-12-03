---
title: 모던 JavaScript 튜토리얼 11 - 프라미스와 async, await 2
date: 2023-11-29 13:01:02 +0900
last_modified_at: 2023-12-03 15:06:32 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

프라미스 API, 프라미스화, 마이크로태스크, async와 await

## 프라미스 API

`Promise` 클래스에는 5가지 정적 메서드가 있음

### Promise.all

여러 개의 프라미스를 동시에 실행시키고 모든 프라미스가 준비될 때까지 기다리는 상황

- 복수의 URL에 동시에 요청을 보내고, 다운로드가 모두 완료된 후에 콘텐츠를 처리하는 경우

`Promise.all`

```javascript
let promise = Promise.all([...promises...]);
```

- 요소 전체가 프라미스인 배열을 받고 새로운 프라미스를 반환
  - 엄밀히 따지만 이터러블 객체이지만 대개는 배열임
- 배열 안 프라미스가 모두 처리되면 새로운 프라미스가 이행됨
- 배열 안 프라미스의 결괏값을 담은 배열이 새로운 프라미스의 `result`가 됨

```javascript
Promise.all([
  new Promise((resolve) => setTimeout(() => resolve(1), 3000)),
  new Promise((resolve) => setTimeout(() => resolve(2), 2000)),
  new Promise((resolve) => setTimeout(() => resolve(3), 1000))
]).then(alert); // 프라미스 전체가 처리되면 1, 2, 3이 반환됨
```

- `Promise.all`은 3초 후에 처리되고, 반환되는 프라미스의 `result`는 `[1, 2, 3]`
- 배열 `result`의 요소 순서는 `Promise.all`에 전달되는 프라미스 순서
- `Promise.all`의 첫 번째 프라미스는 가장 늦게 이행되더라도 처리 결과는 배열의 첫 번째 요소에 저장됨

작업해야 할 데이터가 담긴 배열을 프라미스 배열로 매핑하고, 이 배열을 `Promise.all`로 감싸는 트릭은 자주 사용됨

```javascript
let urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/Violet-Bora-Lee",
  "https://api.github.com/users/jeresig"
];

let requests = urls.map((url) => fetch(url)); // fetch를 사용해 url을 프라미스로 매핑

Promise.all(requests).then((responses) =>
  responses.forEach((response) => alert(`${response.url}: ${response.status}`))
); // Promise.all은 모든 작업이 이행될 때까지 기다림
```

```javascript
let names = ["iliakan", "Violet-Bora-Lee", "jeresig"];
let requests = names.map((name) =>
  fetch(`https://api.github.com/users/${name}`)
);

Promise.all(requests)
  .then((responses) => {
    for (let response of responses)
      alert(`${response.url}: ${response.status}`);
    return responses;
  })
  .then((responses) => Promise.all(responses.map((r) => r.json())))
  .then((users) => users.forEach((user) => alert(user.name)));
```

에러가 발생하면 다른 프라미스는 무시됨

- `Promise.all`에 전달되는 프라미스 중 하나라도 거부되면, `Promise.all`이 반환하는 프라미스는 에러와 함께 바로 거부됨
- 배열에 저장된 다른 프라미스의 결과는 완전히 무시됨
- 이행된 프라미스의 결과도 무시됨
- `fetch`를 사용해 호출 여러 개를 만들면 그중 하나가 실패해도 호출은 계속 일어남
  - 그렇더라도 `Promise.all`은 다른 호출을 더는 신경쓰지 않음
  - 프라미스가 처리되긴 하겠지만 그 결과는 무시됨
- 프라미스에는 취소라는 개념이 없어서 `Promise.all`도 프라미스를 취소하지 않음

```javascript
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("에러 발생!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // Error: 에러 발생!
```

이터러블 객체가 아닌 일반 값도 `Promise.all(iterable)`에 넘길 수 있음

- `Promise.all`은 대개 프라미스가 요소인 객체(대부분 배열)를 받음
- 요소가 프라미스가 아닌 객체일 경우에는 요소 그대로 배열에 전달됨
- 이미 결과를 알고 있는 값은 이 특징을 이용해 `Promise.all`에 그냥 전달하면 됨

```javascript
Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000);
  }),
  2,
  3
]).then(alert); // 1,2,3
// [1, 2, 3]이 리턴됨
```

### Promise.allSettled

`Promise.allSettled`

- 모든 프라미스가 처리될 때까지 기다림
- 반환되는 배열은 다음과 같은 요소를 가짐
  - 응답 성공: `{ status: "fulfilled", value: result }`
  - 에러 발생: `{ status: "rejected", reason: error }`
- 여러 요청 중 하나가 실패해도 다른 요청 결과는 여전히 필요한 경우 사용

```javascript
let urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/Violet-Bora-Lee",
  "https://no-such-url"
];
Promise.allSettled(urls.map((url) => fetch(url))).then((results) => {
  results.forEach((result, num) => {
    if (result.status == "fulfilled") {
      alert(`${urls[num]}: ${result.value.status}`);
    }
    if (result.status == "rejected") {
      alert(`${urls[num]}: ${result.reason}`);
    }
  });
});
```

```
// results
[
  {status: 'fulfilled', value: ...응답...},
  {status: 'fulfilled', value: ...응답...},
  {status: 'rejected', reason: ...에러 객체...}
]
```

### Promise.race

`Promise.race`는 `Promise.all`과 비슷

- 가장 먼저 처리되는 프라미스의 결과(혹은 에러)를 반환

```javascript
let promise = Promise.race(iterable);
```

```javascript
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("에러 발생!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

- 첫 번째 프라미스가 가장 빨리 처리상태가 되기 때문에 첫 번째 프라미스의 결과가 result 값이 됨
- 다른 프라미스의 결과 또는 에러는 무시됨

### Promise.resolve와 Promise.reject

`Promise.resolve`와 `Promise.reject`는 `async/await` 문법이 생긴 후로 쓸모 없어졌기 때문에 거의 사용하지 않음

`Promise.resolve(value)`

- 결괏값이 `value`인 이행 상태 프라미스를 생성
- 호환성을 위해 함수가 프라미스를 반환하도록 해야 할 때 사용할 수 있음
- 아래 코드와 동일한 일을 수행

```javascript
let promise = new Promise((resolve) => resolve(value));
```

```javascript
let cache = new Map();

function loadCached(url) {
  if (cache.has(url)) return Promise.resolve(cache.get(url)); // (*)

  return fetch(url)
    .then((response) => response.text())
    .then((text) => {
      cache.set(url, text);
      return text;
    });
}
```

- `loadCached`를 호출하면 프라미스가 반환된다는 것이 보장되기 때문에 `loadCached(url).then()`을 사용할 수 있음
- `(*)`로 표시한 줄에서 `Promise.resolve`를 사용한 이유임

`Promise.reject(error)`

- 결괏값이 `error`인 거부 상태 프라미스를 생성
- 실무에서 쓸 일이 거의 없음
- 아래 코드와 동일한 일을 수행

```javascript
let promise = new Promise((resolve, reject) => reject(error));
```

## 프라미스화

프라미스화(promisification)

- 콜백을 받는 함수를 프라미스를 반환하는 함수로 바꾸는 것

```javascript
function loadScript(src, callback) {
  let script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`${src} 로드 에러`));
  document.head.append(script);
}
```

```javascript
let loadScriptPromise = function (src) {
  return new Promise((resolve, reject) => {
    loadScriptPromise(src, (err, script) => {
      if (err) reject(err);
      else resolve(script);
    });
  });
};
```

```javascript
function promisify(f) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function callback(err, result) {
        if (err) reject(err);
        else resolve(result);
      }
      args.push(callback);
      f.call(this, ...args);
    });
  };
}

let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
```

```javascript
function promisify(f, manyArgs = false) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function callback(err, ...results) {
        if (err) reject(err);
        else resolve(manyArgs ? results : results[0]);
      }
      args.push(callback);
      f.call(this, ...args);
    });
  };
}

let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
```

## 마이크로태스크

프라미스 핸들러 `.then/catch/finally`는 항상 비동기적으로 실행됨

- 프라미스가 즉시 이행되더라도 `.then/catch/finally` 아래에 있는 코드는 이 핸들러들이 실행되기 전에 실행됨

```javascript
let promise = Promise.resolve();
promise.then(() => alert("프라미스 성공!"));
alert("코드 종료"); // 가장 먼저 출력됨
```

- 프라미스는 즉시 이행상태가 되었는데도 `.then`이 나중에 트리거된 이유?

### 마이크로태스크 큐

비동기 작업을 처리하려면 적절한 관리가 필요

- 이를 위해 ECMA에서는 `PromiseJobs`라는 내부 큐(internal queue)를 명시함
- V8 엔진에서는 이를 마이크로태스크(microtask queue)라고 부름
- 모든 프라미스 동작은 마이크로태스크 큐라 불리는 내부 프라미스 잡(promise job) 큐에 들어가서 처리되기 때문에 프라미스 핸들링은 항상 비동기로 처리됨
  - 따라서 `.then/catch/finally` 핸들러는 항상 현재 코드가 종료되고 난 후에 호출됨

명세서의 설명

- 마이크로태스크 큐는 먼저 들어온 작업을 먼저 실행(FIFO)
- 실행할 것이 아무것도 남아있지 않을 때만 마이크로태스크 큐에 있는 작업이 실행되기 시작

어떤 프라미스가 준비되었을 때 이 프라미스의 `.then/catch/finally` 핸들러가 큐에 들어감

- 이때 핸들러들은 여전히 실행되지 않음
- 현재 코드에서 자유로운 상태가 되었을 때에서야 자바스크립트 엔진은 큐에서 작업을 꺼내 실행

```
promise.then(handler); // 핸들러가 큐에 저장됨(enqueue)
...
alert("코드 종료");
// 스크립트 실행이 끝나야
// 큐에 저장된 핸들러가 실행됨
```

프라미스 핸들러는 항상 내부 큐를 통과하게 됨

여러 개의 `.then/catch/finally`를 사용해 만든 체인의 경우, 각 핸들러는 비동기적으로 실행됨

- 큐에 들어간 핸들러 각각은 현재 코드가 완료되고, 큐에 적체된 이전 핸들러의 실행이 완료되었을 때 실행됨

실행 순서가 중요한 경우에는 `.then`을 사용해 체인에 추가하고 큐에 넣으면 됨

### 처리되지 못한 거부

이제 자바스크립트 엔진이 어떻게 처리되지 못한 거부(unhandled rejection)를 찾는지 정확히 알 수 있음

- '처리되지 못한 거부'는 마이크로태스크 큐 끝에서 프라미스 에러가 처리되지 못할 때 발생
- 정상적인 경우라면 개발자는 에러가 생길 것을 대비해 프라미스 체인에 `.catch`를 추가해 에러를 처리

```javascript
let promise = Promise.reject(new Error("프라미스 실패!"));
promise.catch((err) => alert("잡았다!")); // 잡았다!
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // 에러가 잘 처리되었으므로 실행되지 않음
```

```javascript
let promise = Promise.reject(new Error("프라미스 실패!"));
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // Error: 프라미스 실패!
```

- `.catch`를 추가해주는 것을 잊은 경우, 엔진은 마이크로태스크 큐가 빈 이후에 `unhandledrejection` 이벤트를 트리거 함

```javascript
let promise = Promise.reject(new Error("프라미스 실패!"));
setTimeout(() => promise.catch((err) => alert("잡았다!")), 1000); // 잡았다!
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // Error: 프라미스 실패!
```

- `setTimeout`을 이용해 에러를 나중에 처리하면 '프라미스 실패!'가 먼저, '잡았다!'가 나중에 출력
- `unhandledrejection`은 마이크로태스크 큐에 있는 작업 모두가 완료되었을 때 생성됨
- 엔진은 프라미스들을 검사하고 이 중 하나라도 거부(rejected) 상태이면 `unhandledrejection` 핸들러를 트리거 함
- 위 예시를 실행하면 `setTimeout`을 사용해 추가한 `.catch` 역시 트리거 됨
- 다만 `.catch`는 `unhandledrejection`이 발생한 이후에 트리거 되므로 '프라미스 실패!'가 출력됨

브라우저와 Node.js를 포함한 대부분의 자바스크립트 엔진에서는 마이크로태스크가 이벤트 루프(event loop)와 매크로태스크(macrotask)와 깊은 연관 관계를 맺음

- 이 둘(이벤트 루프, 매크로태스크)은 프라미스와는 직접적인 연관성이 없음

## async와 await

## 참고

> [프라미스와 async, await](https://ko.javascript.info/async)
