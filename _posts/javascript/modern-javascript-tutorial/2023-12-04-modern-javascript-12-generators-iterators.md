---
title: 모던 JavaScript 튜토리얼 12 - 제너레이터와 비동기 이터레이션
date: 2023-12-04 08:07:03 +0900
last_modified_at: 2023-12-04 08:07:03 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

제너레이터, async 이터레이터와 제너레이터

## 제너레이터

일반 함수는 하나의 값(혹은 0개의 값)만을 반환

제너레이터(generator)를 사용하면 여러 개의 값을 필요에 따라 하나씩 반환(yield)할 수 있음

- 제너레이터와 이터러블 객체를 함께 사용하면 손쉽게 데이터 스트림을 만들 수 있음

### 제너레이터 함수

제너레이터를 만들려면 제너레이터 함수라 불리는 특별한 문법 구조 `function*`이 필요

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
```

제너레이터 함수는 일반 함수와 동작 방식이 다름

- 제너레이터 함수를 호출하면 코드가 실행되지 않음
- 대신 실행을 처리하는 특별 객체, 제너레이터 객체가 반환됨

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
let generator = generateSequence();
alert(generator); // [object Generator]
```

- 함수 본문 코드는 아직 실행되지 않음

`next()`

- 제너레이터의 주요 메서드
- 호출하면 가장 가까운 `yield <value>`문을 만날 때까지 실행이 지속됨
- `value`를 생략할 수도 있는데 이 경우에는 `undefined`가 됨
- 이후 `yield <value>`문을 만나면 실행이 멈추고 산출하고하자는 값인 `value`가 바깥 코드에 반환됨

`next()`는 항상 아래 두 프로퍼티를 가진 객체를 반환

- `value`: 산출값
- `done`: 함수 코드 실행이 끝났으면 `true`, 아니면 `false`

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
let generator = generateSequence();
let one = generator.next();
alert(JSON.stringify(one)); // {"value":1,"done":false}
```

- 함수 실행은 두 번째 줄에서 멈춤

```javascript
let two = generator.next();
alert(JSON.stringify(one)); // {"value":2,"done":false}
```

- 함수 실행은 세 번째 줄에서 멈춤

```javascript
let three = generator.next();
alert(JSON.stringify(one)); // {"value":3,"done":false}
```

- 실행은 `return`문에 다다르고 함수가 종료됨
- 제너레이터가 종료됨

제너레이터가 종료되면 `generator.next()`를 여러번 호출해도 객체 `{ done: true }`가 반환될 뿐임

`function* f(...)` vs. `function *f(...)`

- 둘 다 맞음
- `*`는 제너레이터 `함수`를 나타내므로 첫 번째 문법이 선호됨
- `*`는 종류를 나타내는 것이지 이름을 나타내는 것이 아니기 때문

### 제너레이터와 이터러블

제너레이터는 이터러블

- `next()` 메서드에서 짐작할 수 있음
- `for..of` 반복문 사용 가능
- 전개 구문(`...`) 같은 관련 기능 사용 가능

```javascript
function* generateSequence() {
  yield 1; // { done: false, value: 1}
  yield 2; // { done: false, value: 2}
  return 3; // { done: true, value: 3}
}
let generator = generateSequence();
for (let value of generator) alert(value); // 1, 2
```

- `next().value`도 가능
- 위 예시는 `3`은 출력되지 않음
  - `for..of` 이터레이션이 `done: true`일 때 마지막 `value`를 무시하기 때문
- `for..of`를 사용했을 때 모든 값이 출력되게 하려면 `yield`로 값을 반환해야 함

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}
let sequence = [0, ...generateSequence()]; // [0, 1, 2, 3]
```

### 이터러블 대신 제너레이터 사용하기

```javascript
let range = {
  from: 1,
  to: 5,
  // for..of 최초 호출 시, Symbol.iterator가 호출됨
  [Symbol.iterator]() {
    // Symbol.iterator는 이터레이터 객체를 반환
    // for..of는 반환된 이터레이터 객체만을 대상으로 동작
    return {
      current: this.from,
      last: this.to,
      // for..of 반복문에 의해 각 이터레이션마다 next()가 호출됨
      next() {
        if (this.current <= this.last)
          return { done: false, value: this.current++ };
        return { done: true, value: this.current };
      }
    };
  }
};
alert([...range]); // 1,2,3,4,5
```

`Symbol.iterator` 대신 제너레이터 함수를 사용하면, 제너레이터 함수로 반복 가능

```javascript
let range = {
  from: 1,
  to: 5,
  // [Symbol.iterator]: function*() {}을 짧게 줄임
  *[Symbol.iterator]() {
    for (let value = this.from; value <= this.to; value++) yield value;
  }
};
alert([...range]); // 1,2,3,4,5
```

제너레이터는 이터레이터를 어떻게 하면 쉽게 구현할지를 염두에 두며 자바스크립트에 추가됨

제너레이터는 무한한 값을 만들 수도 있음

- 끊임없는 의사 난수 생성 가능

### 제너레이터 컴포지션

제너레이터 컴포지션(generator composition)

- 제너레이터 안에 제너레이터를 임베딩(embedding, composing)할 수 있게 해주는 제너레이터의 특별한 기능
- 한 제너레이터의 흐름을 자연스럽게 다른 제너레이터에 삽입 가능
- 중간 결과 저장 용도의 추가 메모리가 필요하지 않음

```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
function* generatePasswordCodes() {
  yield* generateSequence(48, 57);
  yield* generateSequence(65, 90);
  yield* generateSequence(97, 122);
}
let str = "";
for (let code of generatePasswordCodes()) str += String.fromCharCode(code);
alert(str); // 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
```

`yield*` 지시자

- 실행을 다른 제너레이터에게 위임(delegate)
- 위임은 `yield* gen`이 제너레이터 `gen`을 대상으로 반복을 수행하고, 산출값들을 바깥으로 전달한다는 것을 의미
  - 외부 제너레이터에 의해 값이 산출된 것처럼

```javascript
function* generateAlphaNum() {
  for (let i = 48; i <= 57; i++) yield i;
  for (let i = 65; i <= 90; i++) yield i;
  for (let i = 97; i <= 122; i++) yield i;
}
```

- 중첩 제너레이터(`generateSequence`)의 코드를 직접 써줘도 결과는 같음

### 'yield'를 사용해 제너레이터 안·밖으로 정보 교환하기

`yield`가 양방향 길과 같은 역할을 함

- 결과를 바깥으로 전달할 뿐만 아니라 값을 제너레이터 안으로 전달하기까지 함
- 값을 안, 밖으로 전달하려면 `generator.next(arg)`를 호출해야 함
  - 인수 `arg`는 `yield`의 결과가 됨

```javascript
function* gen() {
  let result = yield "2 + 2 = ?";
  alert(result);
}
let generator = gen();
let question = generator.next().value;
generator.next(4);
```

1. `generator.next()`를 처음 호출할 때는 항상 인수가 없어야 함
   - 인수가 넘어오더라도 무시되어야 함
   - `generator.next()`를 호출하면 실행이 시작되고 첫 번째 `yield "2 + 2 = ?"`의 결과가 반환됨
2. 그 후 `yield`의 결과가 제너레이터를 호출하는 외부 코드에 있는 변수 `question`에 할당됨
3. `generator.next(4)`에서 제너레이터가 다시 시작되고 `4`는 `result`에 할당됨(`let result = 4`)

- 외부 코드에서는 `next(4)`를 즉시 호출하고 있지 않음
- 제너레이터가 기다려주기 때문에 호출을 나중에 해도 문제가 되지 않음

```javascript
setTimeout(() => generator.next(4), 1000);
```

일반 함수와 다르게 제너레이터와 외부 호출 코드는 `next/yield`를 이용해 결과를 전달 및 교환

```javascript
function* gen() {
  let ask1 = yield "2 + 2 = ?";
  alert(ask1); // 4
  let ask2 = yield "3 * 3 = ?";
  alert(ask2); // 9
}
let generator = gen();
alert(generator.next().value); // 2 + 2 = ?
alert(generator.next(4).value); // 3 * 3 = ?
alert(generator.next(9).done); // true
```

```
2 + 2 = ?
4
3 * 3 = ?
9
true
```

### generator.throw

외부 코드는 `yield`의 결과가 될 값을 제너레이터 안에 전달하기도 함

그런데 외부 코드가 에러를 만들거나 던질 수도 있음

- 에러는 결과의 한 종류이기 때문에 자연스러운 현상

에러를 `yield` 안으로 전달하려면 `generator.throw(err)`를 호출해야 함

- `generator.throw(err)`를 호출하게 되면 `err`는 `yield`가 있는 줄로 던져짐

```javascript
function* gen() {
  try {
    let result = yield "2 + 2 = ?"; // (1)
    alert("위에서 에러가 던져졌기 때문에 실행 흐름은 여기까지 다다르지 못함");
  } catch (e) {
    alert(e); // Error: 데이터베이스에서 답을 찾지 못했습니다.
  }
}
let generator = gen();
let question = generator.next().value;
generator.throw(new Error("데이터베이스에서 답을 찾지 못했습니다.")); // (2)
```

- `(2)`에서 제너레이터 안으로 던진 에러는 `yield`와 함께 `(1)`에서 예외를 만듦
- 예외는 `try..catch`에서 잡힘

제너레이터 안에서 예외를 처리하지 않았다면 예외는 여타 예외와 마찬가지로 제너레이터 호출 코드(외부 코드)로 떨어져 나옴

```javascript
function* gen() {
  let result = yield "2 + 2 = ?"; // Error in this line
}
let generator = gen();
let question = generator.next().value;
try {
  generator.throw(new Error("데이터베이스에서 답을 찾지 못했습니다."));
} catch (e) {
  alert(e); // Error: 데이터베이스에서 답을 찾지 못했습니다.
}
```

- `generator.throw`에서 제너레이터를 호출하고 있으므로 에러를 거기서 잡아도 됨

모던 자바스크립트에서는 제너레이터를 잘 사용하지 않음

- 그러나 제너레이터를 사용하면 실행 중에도 제너레이터 호출 코드와 데이터를 교환할 수 있기 때문에 유용한 경우가 있음
- 제너레이터를 사용하면 이터러블 객체를 쉽게 만들 수도 있음
- 비동기 제너레이터는 웹 프로그래밍에서 페이지네이션을 사용해 전송되는 비동기 데이터 스트림을 다룰 때 유용

## async 이터레이터와 제너레이터

비동기 이터레이터(asynchronous iterator)

- 비동기적으로 들어오는 데이터를 필요에 따라 처리할 수 있음
- 네트워크를 통해 데이터가 여러 번에 걸쳐 들어오는 상황 처리
- 비동기 제너레이터(asynchronous generator)도 추가로 사용하면 데이터를 좀 더 편하게 처리 가능

### async 이터레이터

비동기 이터레이터

- 일반 이터레이터와 유사
- 약간의 문법적 차이 존재

이터러블 객체를 비동기적으로 만들기

1. `Symbol.iterator`대신 `Symbol.asyncIterator` 사용
2. `next()`는 프라미스를 반환
3. `for await (let item of iterable)` 반복문을 사용해 처리
   - 비동기 이터러블 객체를 대상으로 하는 반복 작업 처리

```javascript
let range = {
  from: 1,
  to: 5,
  // (1)
  [Symbol.asyncIterator]() {
    return {
      current: this.from,
      last: this.to,
      // 반환 객체는 async에 의해 자동으로 프라미스로 감싸짐 (2)
      async next() {
        await new Promise((resolve) => setTimeout(resolve, 1000)); // (3)
        if (this.current <= this.last)
          return { done: false, value: this.current++ };
        return { done: true };
      }
    };
  }
};
(async () => {
  for await (let value of range) alert(value); // (4)
})();
```

1. `Symbol.asyncIterator` 메서드 반드시 구현 - `(1)`
2. `Symbol.asyncIterator`는 프라미스를 반환하는 메서드인 `next()`가 구현된 객체를 반환 - `(2)`
3. `next()`는 async 메서드일 필요는 없음 - `(3)`
   - 다만 `async`를 사용하면 `await`도 사용할 수 있음
4. `for await (let value of range)`로 반복 작업 - `(4)`
   - `for await (let value of range)`가 실행될 때 `range[Symbol.asyncIterator]()`가 일회 호출되는데 그 이후에는 각 값을 대상으로 `next()`가 호출됨

|                                  |    이터레이터     |    async 이터레이터    |
| :------------------------------: | :---------------: | :--------------------: |
|  이터레이터를 제공해주는 메서드  | `Symbol.iterator` | `Symbol.asyncIterator` |
|      `next()`가 반환하는 값      |      모든 값      |       `Promise`        |
| 반복 작업을 위해 사용하는 반복문 |     `for..of`     |    `for await..of`     |

전개 구문 `...`은 비동기적으로 동작하지 않음

- 일반적인 동기 이터레이터가 필요한 기능은 비동기 이터레이터와 함께 사용할 수 없음
- 전개 구문은 일반 이터레이터가 필요로 하므로 아래 코드는 동작하지 않음
- 전개 구문은 `Symbol.iterator`를 찾기 때문

```javascript
alert([...range]); // TypeError: range is not iterable
```

### async 제너레이터

제너레이터는 이터러블 객체

```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
for (let value of generateSequence(1, 5)) alert(value); // 1, 2, 3, 4, 5
```

일반 제너레이터에서는 `await`를 사용할 수 없고 모든 값은 동기적으로 생산됨

- 일반 제너레이터는 동기적 문법

제너레이터 본문에서 `await`를 사용해야만 하는 상황이 발생하는 경우

- 네트워크 요청을 해야 하는 상황

제너레이터 함수 앞에 `async`를 붙여주면 됨

```javascript
async function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    yield i;
  }
}
(async () => {
  let generator = generateSequence(1, 5);
  for await (let value of generator) alert(value); // 1, 2, 3, 4, 5
})();
```

async 제너레이터의 `generator.next()` 메서드는 비동기적이 되고 프라미스를 반환

```javascript
result = await generator.next();
```

- async 제너레이터에서는 `await`를 붙여 값을 얻어야 함

### async 이터러블

반복 가능한 객체를 만들려면 객체에 `Symbol.iterator`를 추가해야 함

그런데 `Symbol.iterator`는 `next`가 구현된 일반 객체를 반환하는 것보다 제너레이터를 반환하도록 구현하는 경우가 더 많음

```javascript
let range = {
  from: 1,
  to: 5,
  *[Symbol.iterator]() {
    for (let value = this.from; value <= this.to; value++) yield value;
  }
};
for (let value of range) alert(value); // 1, 2, 3, 4, 5
```

- 커스텀 객체 `range`는 반복 가능하고, 제너레이터`*[Symbol.iterator]`에는 값을 나열해주는 로직이 구현됨
- 지금 상태에서 제너레이터에 비동기 동작을 추가하려면 `Symbol.iterator`를 async `Symbol.asyncIterator`로 바꿔야 함

```javascript
let range = {
  from: 1,
  to: 5,
  async *[Symbol.asyncIterator]() {
    for (let value = this.from; value <= this.to; value++) {
      await new Promise((resolve) => setTimeout(resolve, 1000));
      yield value;
    }
  }
};
(async () => {
  for await (let value of range) alert(value); // 1, 2, 3, 4, 5
})();
```

### 실제 사례

상당히 많은 온라인 서비스가 페이지네이션(pagination)을 구현해 데이터를 전송

- 사용자 목록이 필요해 서버에 요청을 보내면 서버는 일정 숫자 단위로 사용자를 끊어 한 페이지로 구성한 후, 다음 페이지를 볼 수 있는 URL과 함께 응답

띄엄띄엄 들어오는 데이터 스트림을 다뤄야 하는 경우

- 용량이 큰 파일을 다운로드하거나 업로드 할 때

이런 데이터를 처리할 때 async 제너레이터를 사용할 수 있음

브라우저 등의 몇몇 호스트 환경은 데이터 스트림을 처리할 수 있게 해주는 API인 Streams를 제공

```javascript
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`;
  while (url) {
    const response = await fetch(url, {
      headers: { "User-Agent": "Our script" }
    });
    const body = await response.json();
    let nextPage = response.headers.get("Link").match(/<(.*?)>; rel="next"/);
    nextPage = nextPage?.[1];
    url = nextPage;
    for (let commit of body) yield commit;
  }
}

(async () => {
  let count = 0;
  for await (const commit of fetchCommits(
    "javascript-tutorial/en.javascript.info"
  )) {
    console.log(commit.author.login);
    if (++count == 100) break;
  }
})();
```

## 참고

> [제너레이터와 비동기 이터레이션](https://ko.javascript.info/generators-iterators)
