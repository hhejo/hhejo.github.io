---
title: 모던 JavaScript 튜토리얼 11 - 프라미스와 async, await 1
date: 2023-11-26 07:42:19 +0900
last_modified_at: 2024-01-21 09:39:45 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    asynchronous,
    callback,
    promise,
    promise-chaining,
    unhandledrejection
  ]
---

콜백, 프라미스, 프라미스 체이닝, 프라미스와 에러 핸들링

## 콜백

비동기(asynchronous) 동작 스케줄링

- 자바스크립트 호스트 환경이 제공하는 여러 함수를 사용해 비동기 동작 스케줄링 가능
- `setTimeout`은 스케줄링에 사용되는 대표적인 함수이고, 스크립트나 모듈 로딩 또한 비동기 동작

```javascript
function loadScript(src) {
  let srcipt = document.createElement("script");
  script.src = src;
  document.head.append(srcipt);
} // <script src="...">를 동적으로 만들고 이를 문서에 추가
loadScript("/my/script.js"); // 비동기적으로 실행됨
newFunction(); // script.js에 있는 함수인데 함수가 존재하지 않는다는 에러 발생
```

콜백(callback) 함수

- 나중에 호출할 함수를 의미
- 콜백 기반(callback-based) 비동기 프로그래밍
- 무언가를 비동기적으로 수행하는 함수
  - 함수 내 동작이 모두 처리된 후 실행되어야 하는 함수가 들어갈 콜백을 인수로 반드시 제공
- `loadScript`의 두 번째 인수로 스크립트 로딩이 끝난 후 실행될 함수가 됨

```javascript
function loadScript(src, callback) {
  let script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}
loadScript("/my/script.js", function () {
  newFunction(); // 제대로 동작
}); // 콜백 함수는 스크립트 로드가 끝나면 실행됨
```

### 콜백 속 콜백

- 스크립트가 두 개 있는 경우, 두 번째 스크립트 로딩은 첫 번째 스크립트의 로딩이 끝난 이후가 되려면
- 콜백 함수 안에서 두 번째 콜백 함수를 호출하면 됨
- 콜백 안에 콜백을 넣는 것은 수행하려는 동작이 많은 경우에는 좋지 않음

```javascript
loadScript("/my/script.js", function (script) {
  alert("첫 번째 스크립트 로딩 완료");
  loadScript("/my/script2.js", function (script) {
    alert("두 번째 스크립트 로딩 완료");
    loadScript("/my/script3.js", function (script) {
      alert("세 번째 스크립트 로딩 완료");
      ...
    });
  });
});
```

### 에러 핸들링

오류 우선 콜백(error-first callback)

- 스크립트 로딩이 실패할 경우, 콜백 함수는 이런 에러를 핸들링할 수 있어야 함
- 아래처럼 에러를 처리하는 방식은 흔히 사용되는 패턴
- 단일 콜백 함수에서 에러 케이스와 성공 케이스를 처리 가능
- `callback`의 첫 번째 인수는 에러가 발생하면 이 인수를 이용해 `callback(err)` 호출
- 두 번째 인수는 에러가 발생하지 않았을 때를 위함. `callback(null, result1, result2, ...)` 호출

```javascript
function loadScript(src, callback) {
  let script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(null, script); // 성공
  script.onerror = () => callback(new Error(`${src} 로딩 도중 에러 발생`)); // 실패
  document.head.append(script);
}
loadScript("/my/script.js", function (error, script) {
  if (error) {
  } // 에러 처리
  else {
  } // 스크립트 로딩 성공
});
```

### 멸망의 피라미드

- 콜백 지옥(callback hell), 멸망의 피라미드(pyramid of doom)

```javascript
loadScript("1.js", function (error, script) {
  if (error) {
    handleError(error);
  } else {
    loadScript("2.js", function (error, script) {
      if (error) {
        handleError(error);
      } else {
        loadScript("3.js", function (error, script) {
          if (error) {
            handleError(error);
          } else {
            loadScript("4.js", function (error, script) {
              if (error) {
                handleError(error);
              } else {
                loadScript("5.js", function (error, script) {
                  if (error) {
                    handleError(error);
                  } else {
                    loadScript("6.js", function (error, script) {
                      if (error) {
                        handleError(error);
                      } else {
                        loadScript("7.js", function (error, script) {});
                      }
                    });
                  }
                });
              }
            });
          }
        });
      }
    });
  }
});
```

각 동작을 독립적인 함수로 만들기

- 깊은 중첩이 없고 콜백 기반 스타일 코드와 동일하게 동작
- 코드가 찢어진 종잇조각 같이 보여 읽기 어려워지고 함수 재사용 불가
- 네임 스페이스 복잡해짐(namespace cluttering)
- 프라미스를 사용해 문제를 해결할 수 있음

```javascript
loadScript("1.js", step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    loadScript("2.js", step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    loadScript("3.js", step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    loadScript("4.js", step3);
  }
}
```

## 프라미스

프라미스 객체 생성 문법

```javascript
let promise = new Promise(function (resolve, reject) {
  // executor
});
```

- `executor`: `new Promise`에 전달되는 함수. 실행자, 실행 함수
  - `new Promise`가 만들어질 때 자동으로 실행됨. 결과를 최종적으로 만들어내는 코드를 포함
- `resolve(value)`: 일이 성공적으로 끝난 경우 그 결과를 나타내는 `value`와 함께 호출
- `reject(error)`: 에러 발생 시 에러 객체를 나타내는 `error`와 함께 호출
- `resolve()`와 `reject()`는 자바스크립트에서 자체 제공하는 콜백
- executor는 자동으로 실행되어 원하는 일이 처리됨
- 처리가 끝나면 executor는 처리 성공 여부에 따라 `resolve`나 `reject`를 호출

`promise` 객체가 갖는 내부 프로퍼티

- `state`: 처음에는 `pending`. `resolve`가 호출되면 `fulfilled`, `reject`가 호출되면 `rejected`로 변화
- `result`: 처음에는 `undefined`. `resolve(value)`가 호출되면 `value`. `reject(error)`가 호출되면 `error`로 변화
- fulfilled promise, rejected promise: 이행된 프라미스, 거부된 프라미스
- settled promise, pending promise: 처리된 프라미스, 대기 상태의 프라미스

```
                                        [ state: "fulfilled" ]
new Promise(executor)                   [ result: value      ]
[ state: "pending"  ] -resolve(value)->
[ result: undefined ] -reject(error)->
                                        [ state: "rejected" ]
                                        [ result: error     ]
```

프라미스의 결과

- 프라미스는 성공 또는 실패만 함. executor에 의해 처리가 끝난 일은 결과 혹은 에러만 가질 수 있음
- executor는 반드시 `resolve`나 `reject` 중 하나를 호출하고 이때 변경된 상태는 더 이상 변하지 않음
- 처리가 끝난 프라미스에 `resolve`나 `reject`를 호출하면 무시됨
- `resolve`나 `reject`는 인수를 하나만 받거나 아무것도 받지 않고 그 이외의 인수는 무시
- `resolve`나 `reject` 함수를 즉시 호출할 수도 있는데, 이렇게 하면 프라미스는 즉시 이행 상태가 됨

프라미스 객체의 `state`와 `result` 프로퍼티

- 내부 프로퍼티로 개발자가 직접 접근 불가
- `.then/catch/finally` 메서드로 접근

`Error` 객체와 함께 거부하기

- `reject`의 인수는 `resolve`와 마찬가지로 어떤 타입도 가능
- `Error` 객체 또는 `Error`를 상속받은 객체를 사용할 것을 권장

### 소비자: then, catch, finally

then

```javascript
promise.then(
  function (result) {},
  function (error) {}
);
```

- `.then(f1, f2)`
- `f1`: 프라미스가 이행되었을 때 실행되는 함수. 실행 결과를 받음
- `f2`: 프라미스가 거부되었을 때 실행되는 함수. 에러를 받음
- `.then`에 인수를 하나만 전달하면 작업이 성공적으로 처리된 경우만 다룸

```javascript
let promise = new Promise(function (resolve, reject) {
  setTimeout(() => resolve("완료!"), 1000); // setTimeout(() => reject(new Error("에러!")), 1000);
});
promise.then(
  (result) => alert(result),
  (error) => alert(error)
);
```

catch

- `.catch(f)`
- `.catch(f)`는 `.then(null, f)`과 동일하게 동작

```javascript
let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("에러!")), 1000);
});
promise.catch(alert); // Error: 에러!
```

finally

- `.finally(f)`
- 프라미스가 처리되면(이행이나 거부) 항상 실행됨
- 결과가 어떻든 마무리가 필요할 때 유용. 프라미스 결과를 처리하기 위해 만들어진 것이 아님
- `.then(f, f)`와 완전히 같지는 않음
- `finally` 핸들러에는 인수가 없고 프라미스가 이행되었는지 거부되었는지 알 수 없음
- `finally` 핸들러는 자동으로 다음 핸들러에 프라미스 결과와 에러를 전달
- `.finally(f)`는 `.then(f, f)` 보다 함수 `f`를 중복해서 쓸 필요가 없어 문법 측면에서 더 편리

```javascript
new Promise((resolve, reject) => {
  setTimeout(() => resolve("결과"), 2000);
})
  .finally(() => alert("프라미스 준비됨")) // result가 finally를 거쳐 then에 전달됨
  .then(result = > alert(result)); // .then에서 result를 다룰 수 있음

new Promise((resolve, reject) => {
  throw new Error("에러 발생!");
})
  .finally(() => alert("프라미스 준비됨")) // error가 finally를 거쳐 catch에 전달됨
  .catch((err) => alert(err)); // .catch에서 에러 객체를 다룰 수 있음
```

처리된 프라미스의 핸들러는 즉각 실행됨

- 프라미스가 대기 상태일 때, `.then/catch/finally` 핸들러는 프라미스가 처리되길 기다림
- 프라미스가 이미 처리상태라면 핸들러가 즉각 실행됨

```javascript
let promise = new Promise((resolve) => resolve("완료!")); // 생성과 동시에 이행되는 프라미스
promise.then(alert); // 완료!
```

### 예시: loadScript

```javascript
function originalLoadScript(src, callback) {
  let script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`${src} 로드 에러`));
  document.head.append(script);
}

function loadScript(src) {
  return new Promise(function (resolve, reject) {
    let script = document.createElement("script");
    script.src = src;
    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`${src} 로드 에러`));
    document.head.append(script);
  });
}

let promise = loadScript("...");
promise.then(
  (script) => alert(`${script.src} 로드 성공`),
  (error) => alert(`Error: ${error.message}`)
);
promise.then((script) => alert("또다른 핸들러"));
```

프라미스

- 흐름이 자연스럽고 원하는 만큼 `.then` 호출 가능

콜백

- 함수를 호출할 때, 함께 호출할 콜백 함수가 준비되어야 하고 호출 결과로 무얼을 할지 미리 알고 있어야 함
- 콜백은 하나만 가능

### 예제

두 번 resolve하기?

```javascript
let promise = new Promise(function (resolve, reject) {
  resolve(1);
  setTimeout(() => resolve(2), 1000); // 무시됨
});
promise.then(console.log); // 1
```

프라미스로 지연 만들기

```javascript
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
delay(3000).then(() => console.log("3초후 실행"));
```

## 프라미스 체이닝

프라미스 체이닝(promise chaining)

- 스크립트를 불러오는 것 등의 순차적으로 처리해야 하는 비동기 작업이 여러 개 있다면 프라미스 체이닝으로 비동기 처리 가능
- `promise.then`을 호출하면 프라미스가 반환되기 때문에 프라미스 체이닝 가능
- 핸들러가 값을 반환할 때는 이 값이 프라미스의 `result`가 되고, 다음 `.then`은 이 값을 이용해 호출됨
- 프라미스 하나에 `.then`을 여러 개 추가하는 것은 체이닝이 아님. 독립적으로 처리됨

```javascript
new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
})
  .then(function (result) {
    alert(result); // 1
    return result * 2;
  })
  .then(function (result) {
    alert(result); // 2
    return result * 2;
  })
  .then(function (result) {
    alert(result); // 4
    return result * 2;
  });
```

```javascript
// 프라미스 체이닝이 아님
let promise = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});
promise.then(function (result) {
  alert(result); // 1
  return result * 2;
});
promise.then(function (result) {
  alert(result); // 1
  return result * 2;
});
promise.then(function (result) {
  alert(result); // 1
  return result * 2;
});
```

### 프라미스 반환하기

- `.then(handler)`에 사용된 핸들러가 프라미스를 생성하거나 반환하는 경우
- 이어지는 핸들러는 프라미스가 처리될 때까지 기다림. 처리가 완료되면 그 결과를 받음

```javascript
new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
})
  .then(function (result) {
    alert(result); // 1
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(result * 2), 1000);
    });
  })
  .then(function (result) {
    alert(result); // 2
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(result * 2), 1000);
    });
  })
  .then(function (result) {
    alert(result); // 4
  });
```

thenable 객체

- `.then`이라는 메서드를 가진 객체. 프라미스와 같은 방식으로 처리됨
- 핸들러는 프라미스가 아닌 `thenable` 객체를 반환하기도 함
- 서드파티 라이브러리가 프라미스와 호환 가능한 자체 객체를 구현할 수 있다는 점에서 나옴
- `.then`이 있기 때문에 네이티브 프라미스와도 호환 가능

```javascript
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    setTimeout(() => resolve(this.num * 2), 1000);
  } // Promise를 상속받지 않고도 커스텀 객체로 프라미스 체이닝 생성 가능
}

new Promise((resolve) => resolve(1))
  .then((result) => new Thenable(result))
  .then(alert); // 2
```

### fetch와 체이닝 함께 응용하기

프론트 단에서는 네트워크 요청 시 프라미스를 자주 사용

```javascript
let promise = fetch(url);
```

### 예제

프라미스: then vs. catch

```javascript
promise.then(f1).catch(f2); // (1)
promise.then(f1, f2); // (2)
```

1. `f1`에서 에러가 발생하면 `.catch`에서 에러가 처리됨
2. `f1`에서 발생한 에러를 처리하지 못함

## 프라미스와 에러 핸들링

거부된 프라미스

- 프라미스가 거부되면 제어 흐름이 제일 가까운 rejection 핸들러로 넘어감
- 프라미스 체인을 사용해 에러를 쉽게 처리
- 프라미스 중 하나라도 거부되면 `.catch`에서 에러를 잡음

```javascript
fetch("...").then().then().then().then().then().catch(error = > alert(error.message));
```

### 암시적 try...catch

- 프라미스 executor와 프라미스 핸들러 코드 주위에 보이지 않게 있음
- 예외가 발생하면 암시적 `try..catch`에서 예외를 잡고 이를 `reject`처럼 다룸
- executor 주위의 암시적 `try..catch`는 스스로 에러를 잡고 에러를 거부 상태의 프라미스로 바꿈
- executor 함수뿐만 아니라 핸들러에서도 발생

```javascript
// executor에서의 암시적 try..catch
new Promise((resolve, reject) => {
  throw new Error("에러 발생!"); // executor에서의 암시적 try..catch
}).catch(alert); // Error: 에러 발생!
new Promise((resolve, reject) => {
  reject(new Error("에러 발생!")); // 위 코드와 동일하게 동작
}).catch(alert); // Error: 에러 발생!

// handler에서의 암시적 try..catch
new Promise((resolve, reject) => {
  resolve("OK");
})
  .then((result) => {
    throw new Error("에러 발생!"); // handler에서의 암시적 try..catch
  })
  .catch(alert); // Error: 에러 발생!
new Promise((resolve, reject) => {
  resolve("OK");
})
  .then((result) => {
    blabla(); // handler에서의 암시적 try..catch
  })
  .catch(alert); // ReferenceError: blabla is not defined
```

### 다시 던지기

- `.catch` 안에서 `throw`를 사용하는 경우, 제어 흐름이 가장 가까운 곳에 있는 에러 핸들러로 넘어감
- 여기서 에러가 성공적으로 처리되면 가장 가까운 `.then` 핸들러로 제어 흐름이 넘어가 실행이 이어짐

```javascript
new Promise((resolve, reject) => {
  throw new Error("에러 발생!");
})
  .catch(function (error) {
    alert("에러가 잘 처리되었고 정상적으로 실행이 이어짐");
  })
  .then(() => alert("다음 핸들러가 실행됨"));

new Promise((resolve, reject) => {
  throw new Error("에러 발생!");
})
  .catch(function (error) {
    throw error; // 에러 다시 던지기
  })
  .then(function () {}) // 실행되지 않음
  .catch((error) => {
    alert(`알 수 없는 에러 발생: ${error}`);
  });
```

### 처리되지 못한 거부

- 발생한 에러를 `try..catch`에서 처리하지 못하면 스크립트가 죽고 콘솔 창에 메시지가 출력됨
- 거부된 프라미스를 처리하지 못했을 때도 유사한 일 발생
- 자바스크립트 엔진은 프라미스 거부를 추적하다가 아래와 같은 상황이 발생하면 전역 에러를 생성
- 브라우저 환경에서는 이런 에러를 `unhandledrejection` 이벤트로 처리 가능

```javascript
new Promise(function () {
  noSuchFunction(); // 에러 발생
}).then(() => {}); // .catch가 없어 전역 에러 생성
```

`unhandledrejection` 이벤트

- HTML 명세서에 정의된 표준 이벤트
- 브라우저 환경에서 에러가 발생했는데 `.catch`가 없으면 `unhandledrejection` 핸들러 동작
- 에러 정보가 담긴 `event` 객체를 받음. `event` 객체에는 특수 프로퍼티 2가지가 있음
  - `promise`: 에러를 생성하는 프라미스
  - `reason`: 처리하지 못한 에러 객체

```javascript
window.addEventListener("unhandledrejection", function (event) {
  alert(event.promise); // [object Promise]
  alert(event.reason); // Error: 에러 발생!
});

new Promise(function () {
  throw new Error("에러 발생!");
}); // 에러를 처리할 수 있는 .catch 핸들러가 없음
```

### 예제

```javascript
new Promise(function (resolve, reject) {
  setTimeout(() => {
    throw new Error("에러 발생!");
    // reject(new Error("에러 발생!")); // 이 에러는 받을 수 있음
  }, 1000);
}).catch(alert); // 트리거되지 않음
```

- 암시적 `try..catch`가 함수 코드를 감싸고 있으므로 모든 동기적 에러는 암시적 `try..catch`에서 처리됨
- 에러는 executor가 실행되는 동안이 아니라 나중에 발생하므로 프라미스는 에러를 처리할 수 없음

## 참고

> [프라미스와 async, await](https://ko.javascript.info/async)
