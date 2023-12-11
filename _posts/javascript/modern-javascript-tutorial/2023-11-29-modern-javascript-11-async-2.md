---
title: 모던 JavaScript 튜토리얼 11 - 프라미스와 async, await 2
date: 2023-11-29 13:01:02 +0900
last_modified_at: 2023-12-10 11:40:19 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

프라미스 API, 프라미스화, 마이크로태스크, async와 await

## 프라미스 API

`Promise` 클래스의 5가지 정적 메서드

- `Promise.all`
- `Promise.allSettled`
- `Promise.race`
- `Promise.resolve`
- `Promise.reject`

### Promise.all

`Promise.all`

```javascript
let promise = Promise.all([...promises...]);
```

- 여러 프라미스를 동시에 실행시키고, 모두 완료될 때까지 대기
  - 복수의 URL에 동시에 요청을 보내고, 다운로드가 모두 완료된 후에 콘텐츠를 처리하는 경우
- 요소 전체가 프라미스인 배열을 받음
  - 엄밀히 따지만 이터러블 객체이지만 대개는 배열
- 배열 안 프라미스가 모두 처리되면 새로운 프라미스 반환
- 새로운 프라미스의 `result`는 배열 안 프라미스의 결괏값을 담은 배열
- 배열 `result`의 요소 순서는 `Promise.all`에 전달되는 프라미스 순서
  - `Promise.all`의 첫 번째 프라미스가 가장 늦게 이행되더라도, 처리 결과는 배열의 첫 번째 요소가 됨

```javascript
Promise.all([
  new Promise((resolve) => setTimeout(() => resolve(1), 3000)),
  new Promise((resolve) => setTimeout(() => resolve(2), 2000)),
  new Promise((resolve) => setTimeout(() => resolve(3), 1000))
]).then(alert); // 3초 후 프라미스 전체가 처리되고, 반환된 프라미스의 result는 [1, 2, 3] (전달된 프라미스 순서)
```

작업할 데이터가 담긴 배열을 프라미스 배열로 매핑하고 `Promise.all`로 감싸기

```javascript
let url = "https://api.github.com/users";
let names = ["iliakan", "Violet-Bora-Lee", "jeresig"];
let requests = names.map((name) => fetch(`${url}/${name}`));
Promise.all(requests)
  .then((responses) => {
    for (let { url, status } of responses) alert(`${url}: ${status}`);
    return responses;
  })
  .then((responses) => Promise.all(responses.map((r) => r.json())))
  .then((users) => users.forEach((user) => alert(user.name)));
```

에러가 발생하는 경우

- `Promise.all`에 전달되는 프라미스 중 하나라도 거부되는 경우
- `Promise.all`이 반환하는 프라미스는 에러와 함께 바로 거부됨
- 배열에 저장된 다른 프라미스의 결과는 이행되었어도 완전히 무시됨
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

프라미스가 아닌 객체를 인수로 전달하는 경우

- 요소 그대로 배열에 전달됨
- 이 특징을 이용해 이미 결과를 알고 있는 값은 `Promise.all`에 그냥 전달하면 됨
- 이터러블 객체가 아닌 일반 값도 넘길 수 있음
- 대개 프라미스가 요소인 객체(대부분 배열)를 받음

```javascript
Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000);
  }),
  2,
  3
]).then(alert); // 1,2,3. [1, 2, 3]을 받음
```

### Promise.allSettled

`Promise.allSettled`

- 여러 요청 중 하나가 실패해도 다른 요청 결과는 여전히 필요한 경우 사용
- 모든 프라미스가 처리될 때까지 대기
- 반환되는 배열이 갖는 요소
  - 응답 성공: `{ status: "fulfilled", value: result }`
  - 에러 발생: `{ status: "rejected", reason: error }`

```javascript
let urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/Violet-Bora-Lee",
  "https://no-such-url"
];
Promise.allSettled(urls.map((url) => fetch(url))).then((results) => {
  for (let [index, result] of results.entries()) {
    if (result.status == "fulfilled")
      console.log(`${urls[index]}: ${result.value.status}`);
    if (result.status == "rejected")
      console.log(`${urls[index]}: ${result.reason}`);
  }
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

`Promise.race`

```javascript
let promise = Promise.race(iterable);
```

- `Promise.all`과 비슷
- 가장 먼저 처리되는 프라미스의 결과(혹은 에러)를 반환
- 다른 프라미스의 결과 또는 에러는 무시됨

```javascript
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("에러 발생!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1. 가장 먼저 처리된 프라미스의 결과
```

### Promise.resolve와 Promise.reject

`Promise.resolve(value)`

```javascript
let promise = new Promise((resolve) => resolve(value));
```

- 결괏값이 `value`인 이행 상태 프라미스 생성
- 호환성을 위해 함수가 프라미스를 반환하도록 해야 할 때 사용 가능
- `async/await` 문법이 생긴 후로 쓸모 없어졌기 때문에 거의 사용하지 않음

```javascript
let cache = new Map();
function loadCached(url) {
  if (cache.has(url)) return Promise.resolve(cache.get(url)); // 프라미스 반환을 보장
  return fetch(url)
    .then((response) => response.text())
    .then((text) => {
      cache.set(url, text);
      return text;
    });
} // loadCached는 프라미스를 반환
loadCached(url).then();
```

`Promise.reject(error)`

```javascript
let promise = new Promise((resolve, reject) => reject(error));
```

- 결괏값이 `error`인 거부 상태 프라미스를 생성
- 실무에서 쓸 일이 거의 없음
- `async/await` 문법이 생긴 후로 쓸모 없어졌기 때문에 거의 사용하지 않음

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

프라미스 핸들러 `.then/catch/finally`

- 항상 비동기적으로 실행
- 프라미스가 즉시 이행되었어도 마찬가지
- `.then/catch/finally` 아래에 있는 코드는 이 핸들러들이 실행되기 전에 실행됨

```javascript
let promise = Promise.resolve(); // 즉시 이행 상태
promise.then(() => alert("프라미스 성공!"));
alert("코드 종료"); // 가장 먼저 출력됨
```

### 마이크로태스크 큐

마이크로태스크 큐(microtask queue)

- 비동기 작업 처리를 위해서는 적절한 관리가 필요
- 이를 위해 ECMA에서는 `PromiseJobs`라는 내부 큐를 명시
  - 내부 프라미스 잡 큐(internal promise job queue)
- V8 엔진에서는 마이크로태스크 큐라고 부름
- 모든 프라미스 동작은 내부 프라미스 잡 큐에 들어가서 처리됨
- 따라서 프라미스 핸들링은 항상 비동기로 처리됨
  - `.then/catch/finally` 핸들러는 항상 현재 코드가 종료되고 난 후에 호출됨
- 실행할 것이 아무것도 남아있지 않을 때만 마이크로태스크 큐에 있는 작업 실행
- 마이크로태스크 큐는 먼저 들어온 작업을 먼저 실행(FIFO)

어떤 프라미스가 준비된 경우

- 이 프라미스의 `.then/catch/finally` 핸들러가 큐에 들어감
- 이때 핸들러들은 여전히 실행되지 않음
- 현재 코드에서 자유로운 상태가 되었을 때에서야 자바스크립트 엔진은 큐에서 작업을 꺼내 실행
- 프라미스 핸들러는 항상 내부 큐를 통과

```
promise.then(handler); // 핸들러가 큐에 저장됨(enqueue)
...
alert("코드 종료");
// 스크립트 실행이 끝나야 큐에 저장된 핸들러가 실행됨
```

여러 개의 `.then/catch/finally`를 사용해 만든 체인의 경우

- 각 핸들러는 비동기적으로 실행됨
- 큐에 들어간 핸들러 각각은 현재 코드가 완료되고, 큐에 적체된 이전 핸들러의 실행이 완료되었을 때 실행됨
- 실행 순서가 중요한 경우에는 `.then`으로 체인에 추가하고 큐에 넣으면 됨

### 처리되지 못한 거부

처리되지 못한 거부(unhandled rejection)

- 마이크로태스크 큐 끝에서 프라미스 에러가 처리되지 못할 때 발생
- 정상적인 경우, 개발자는 에러를 대비해 프라미스 체인에 `.catch`를 추가해 에러를 처리
- `.catch`가 없으면 엔진은 마이크로태스크 큐가 빈 이후에 `unhandledrejection` 이벤트를 트리거

`unhandledrejection` 이벤트

- 마이크로태스크 큐에 있는 작업 모두가 완료되었을 때 생성됨
- 엔진은 프라미스들을 검사
- 이 중 하나라도 거부(rejected) 상태이면 `unhandledrejection` 핸들러를 트리거
- `setTimeout`으로 추가된 `.catch`는 `unhandledrejection` 발생 후에 트리거

```javascript
let promise = Promise.reject(new Error("프라미스 실패!"));
promise.catch((err) => alert("잡았다!")); // 잡았다!
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // 에러가 잘 처리되었으므로 실행되지 않음

// 처리되지 못한 거부
let promise = Promise.reject(new Error("프라미스 실패!"));
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // Error: 프라미스 실패!

// setTimeout으로 에러를 나중에 처리하는 경우
let promise = Promise.reject(new Error("프라미스 실패!"));
setTimeout(() => promise.catch((err) => alert("잡았다!")), 1000); // 잡았다!. 나중에 출력
window.addEventListener("unhandledrejection", (event) => alert(event.reason)); // Error: 프라미스 실패!. 먼저 출력
```

이벤트 루프(event loop), 매크로태스크

- 마이크로태스크가 이벤트 루프와 매크로태스크와 깊은 연관 관계를 맺음
  - 브라우저와 Node.js를 포함하는 대부분의 자바스크립트 엔진의 경우
- 이벤트 루프와 매크로태스크는 프라미스와는 직접적인 연관성이 없음

마이크로태스크 큐(microtask queue)

- `process.nextTick()`
- `Promise callback`
  - `.then/catch/finally`
- `async functions`
- `queueMicrotask`

매크로태스크 큐(macrotask queue)

- 마이크로태스크 큐 작업이 모두 진행된 이후에 매크로태스크 큐 작업이 진행됨
- `setTimeout()`
- `setInterval()`
- `setImmediate()`
- I/O Operations
- UI rendering
- ...

## async와 await

특별한 문법 `async`와 `await`

- 프라미스를 좀 더 편하게 사용할 수 있음

### async 함수

`async`

```javascript
async function f() {}
```

- `async` 키워드는 function 앞에 위치
- function 앞에 `async`를 붙이면 해당 항수는 항상 프라미스를 반환
- 프라미스가 아닌 값을 반환하는 경우
- 이행 상태의 프라미스(resolved promise)로 값을 감싸 반환

```javascript
async function f1() {
  return 1;
}
f1().then(alert); // 1. result가 1인 이행 프라미스 반환

async function f2() {
  return Promise.resolve(1);
}
f2().then(alert); // 1. 명시적으로 프라미스 반환
```

### await

`await`

```javascript
let value = await promise;
```

- `await` 키워드는 `async` 함수 안에서만 동작
- 자바스크립트는 `await` 키워드를 만나면 프라미스가 처리될 때까지 대기
- 프라미스가 처리되면 그 결과를 반환하며 실행 재개
- `promise.then`보다 쓰기 쉽고 가독성 좋음
- 일반 함수에는 `await` 사용 불가
- 프라미스가 처리되길 기다리는 동안, 엔진이 다른 일을 할 수 있어 CPU 리소스가 낭비되지 않음
  - 다른 스크립트를 실행, 이벤트 처리 등의 다른 일

```javascript
async function f() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("완료!"), 1000);
  });
  let result = await promise; // (*). 프라미스가 이행될 때까지 대기
  console.log(result); // 완료!. 프라미스 객체의 result 값이 들어있음
}
f();
```

```javascript
function f() {
  let promise = Promise.resolve(1);
  let result = await promise; // SyntaxError: await is only valid in async functions and the top level bodies of modules
}
```

top-level await

- `await`는 최상위 레벨 코드(top-level code)에서 작동하지 않음
- 익명 `async` 함수로 코드를 감싸면 최상위 레벨 코드에도 `await` 사용 가능
- ES2022부터는 top-level await 가능

```javascript
// SyntaxError: await is only valid in async functions and the top level bodies of modules
let response = await fetch("/article/promise-chaining/user.json");
let user = await response.json();

// 익명 async 함수로 코드를 감싸 top-level await 사용
(async () => {
  let response = await fetch("/article/promise-chaining/user.json");
  let user = await response.json();
})();
```

`await`와 thenable 객체

- `await`는 thenable 객체를 받음
- thenable 객체는 `then` 메서드가 있는 호출 가능한 객체
- `promise.then`처럼, `await`에도 thenable 객체 사용 가능
- 서드파티 객체가 프라미스가 아니지만 프라미스와 호환 가능한 객체를 제공할 수 있다는 점에서 생긴 기능
- 서드파티에서 받은 객체가 `.then`을 지원하면 이 객체를 `await`와 함께 사용 가능

```javascript
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve);
    setTimeout(() => resolve(this.num * 2), 1000); // (*)
  }
}
async function f() {
  let result = await new Thenable(1); // then 메서드를 호출하고 resolve나 reject를 기다림
  alert(result);
}
f();
```

`async` 클래스 메서드

- 메서드 이름 앞에 `async`를 추가해 `async` 클래스 메서드를 선언
- 프라미스를 반환하고 `await` 사용 가능

```javascript
class Waiter {
  async wait() {
    return await Promise.resolve(1);
  }
}
new Waiter().wait().then(alert); // 1
```

### 에러 핸들링

async-await 에러 핸들링

- 프라미스가 정상적으로 이행되면 `await promise`는 프라미스 객체의 `result` 값 반환
- 프라미스가 거부되면 `throw`문을 작성한 것처럼 에러가 던져짐
- `await`가 던진 에러는 `try..catch`를 사용해 잡음
- 함수 내부에서 `await`의 에러를 잡지 못하면, 호출 시 `.catch`를 이용해 잡을 수 있음
- 실제 상황에서는 프라미스가 거부되기 전에 약간의 시간이 지체되는 경우가 있음
- 이런 경우에는 `await`가 에러를 던지기 전에 지연 발생

```javascript
async function f1() {
  await Promise.reject(new Error("에러 발생!")); // 아래 코드와 동일
}

async function f2() {
  throw new Error("에러 발생!");
}
```

```javascript
async function f() {
  try {
    let response = await fetch("http://유효하지-않은-주소");
    let user = await response.json();
  } catch (err) {
    alert(err); // fetch와 response.json()에서 발행한 에러 모두를 여기서 잡음
  }
}
f();

// try..catch가 없는 경우
async function f2() {
  let response = await fetch("http://유효하지-않은-주소");
}
f2().catch(alert); // (*). f2()는 거부 상태의 프라미스가 되고, .catch로 처리 가능

// .catch도 추가하지 않은 경우
async function f3() {
  let response = await fetch("http://유효하지-않은-주소");
}
f3(); // 처리되지 않은 프라미스 에러. unhandledrejection으로 잡을 수 있음
```

`async/await` vs. `promise.then/catch`

- `await`가 대기를 처리해주기 때문에 `.then`이 거의 필요하지 않음
- `.catch` 대신 일반 `try..catch` 사용 가능
- `async` 함수 바깥의 최상위 레벨 코드에서 `await` 사용 불가
  - 그렇기 때문에 관행처럼 `.then/catch`를 추가해 최종 결과나 처리되지 못한 에러를 다룸
- `Promise.all`과도 함께 사용 가능

```javascript
let results = await Promise.all([fetch(url1), fetch(url2), ...]);
```

### 예제

async가 아닌 함수에서 async 함수 호출하기

- 일반 함수에서 async 함수를 호출하고 그 결과를 사용하는 방법
- async 함수를 호출하면 프라미스가 반환되므로 `.then`을 붙이면 됨

```javascript
async function wait() {
  await new Promise((resolve) => setTimeout(resolve, 1000));
  return 10;
}
function f() {
  wait().then((result) => alert(result));
}
f(); // 10
```

## 참고

> [프라미스와 async, await](https://ko.javascript.info/async)
