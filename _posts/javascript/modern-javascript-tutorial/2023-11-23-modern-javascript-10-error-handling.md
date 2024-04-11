---
title: 모던 JavaScript 튜토리얼 10 - 에러 핸들링
date: 2023-11-23 08:11:35 +0900
last_modified_at: 2024-04-11 09:38:02 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, try-catch, error, throw, finally, custom-error]
---

'try..catch'와 에러 핸들링, 커스텀 에러와 에러 확장

## 'try..catch'와 에러 핸들링

에러 발생

- 스크립트가 죽고(즉시 중단됨) 콘솔에 에러 출력
- `try..catch`로 스크립트가 죽는 것을 막고 에러를 잡아 더 합당한 무언가를 할 수 있음

### 'try...catch' 문법

`try {...} catch (err) {...}`

```javascript
try {
  // 코드
} catch (err) {
  // 에러 핸들링
}
```

1. `try {...}` 안의 코드 실행
2. 에러가 없다면 `try` 안의 마지막 줄까지 실행. `catch` 블록은 건너뜀
3. 에러가 있다면 `try` 안 코드의 실행이 중단되고 `catch (err)` 블록으로 제어 흐름 넘김
   - `err`는 에러 객체를 포함. 아무 이름도 가능
   - 무슨 일이 일어났는지에 대한 설명이 담김

- 자바스크립트 엔진은 코드를 읽고 난 후 코드를 실행
- 코드가 문법적으로 잘못된 경우에는 동작하지 않음
- 실행 가능한(runnable) 코드, 유효한 코드에서 발생하는 에러만 처리 가능
- 런타임 에러(runtime error), 예외(exception)
- parse-time 에러: 코드를 읽는 중에 발생하는 에러
  - 엔진은 이 코드를 이해할 수 없기 때문에 parse-time 에러는 코드 안에서 복구 불가
- 동기적으로 동작하기 때문에 setTimeout처럼 스케줄된(scheduled) 코드에서 발생한 예외는 잡을 수 없음
  - setTimeout에 넘겨진 익명 함수는 엔진이 `try..catch`를 떠난 후 실행
  - 스케줄된 함수 내부의 예외를 잡으려면 `try..catch`를 함수 내부에 구현

```javascript
// 스케줄된 코드에서 발생한 예외는 잡을 수 없음
try {
  setTimeout(function () {
    noSuchVariable; // 스크립트는 여기서 죽음
  }, 1000);
} catch (e) {
  alert("작동 멈춤"); // 출력되지 않음
}

// 스케줄된 함수 내부의 예외 잡기
setTimeout(function () {
  try {
    noSuchVariable;
  } catch {
    alert("에러 잡음"); // 에러 잡음
  }
}, 1000);
```

### 에러 객체

내장 에러 전체와 에러 객체

- 에러가 발생했을 때, 자바스크립트는 에러 상세내용이 담긴 객체를 생성하고 `catch` 블록에 인수로 전달
- 두 가지 주요 프로퍼티 `name`, `message`를 가짐
- `name`: 에러 이름
- `message`: 에러 상세 내용을 담은 문자 메시지
- `stack`: 현재 호출 스택. 에러를 유발한 중첩 호출들의 순서 정보를 가진 문자열
  - 가장 널리 사용되는 비표준 프로퍼티로, 대부분의 호스트 환경에서 지원하며 디버깅 목적으로 사용

```javascript
try {
  lalala;
} catch (err) {
  alert(err.name); // ReferenceError
  alert(err.message); // lalala is not defined
  alert(err.stack); // ReferenceError: lalala is not defined at ...
  // 에러 전체를 보여줄 수도 있음
  // 이때 에러 객체는 "name: message" 형태의 문자열로 변환됨
  alert(err); // ReferenceError: lalala is not defined
}
```

### 선택적 'catch' 바인딩

- 에러에 대한 자세한 정보가 필요하지 않으면 `catch`에서 생략 가능

```javascript
try {
} catch {} // (err) 없이 쓸 수 있음
```

### 'try...catch' 사용하기

```javascript
let json = "{ bad json }";
try {
  let user = JSON.parse(json); // 에러 발생
  alert(user.name); // 동작하지 않음
} catch (e) {
  alert("데이터에 에러가 있어 재요청을 시도합니다.");
  alert(e.name);
  alert(e.message);
}
```

### 직접 에러를 만들어서 던지기

`throw` 연산자

```javascript
throw <error object>
```

- 에러를 생성. 이론적으로는 어떤 것이든 에러 객체(error object)로 사용 가능
  - 숫자, 문자열 같은 원시형 자료를 포함
- 내장 에러와의 호환을 위해 에러 객체에 `name`, `message` 프로퍼티를 넣는 것을 권장

표준 에러 객체 관련 생성자

```javascript
let error = new Error(message);
error = new SyntaxError(message);
error = new ReferenceError(message);
...
```

- 자바스크립트는 표준 에러 객체 관련 생성자를 지원
  - `Error`, `SyntaxError`, `ReferenceError`, `TypeError` 등
- 이 생성자들로 에러 객체 생성 가능
- 일반 객체가 아닌 내장 생성자를 사용해 만든 경우, 내장 에러 객체의 `name` 프로퍼티는 생성자 이름과 동일
- 프로퍼티 `message`의 값은 인수에서 가져옴

```javascript
let error = new Error("이상한 일이 발생했습니다. o_0");
alert(error.name); // Error
alert(error.message); // 이상한 일이 발생했습니다. o_0

let json = '{ "age": 30 }';
try {
  let user = JSON.parse(json);
  if (!user.name) throw new Error("불완전한 데이터: 이름 없음");
  alert(user.name);
} catch (e) {
  alert("JSON Error: " + e.message); // JSON Error: 불완전한 데이터: 이름 없음
}
```

### 에러 다시 던지기

- 다시 던지기(rethrowing)
- 또 다른 예기치 않은 에러가 `try {...}` 블록 안에서 발생할 수 있음
- catch는 알고 있는 에러만 처리하고 나머지는 다시 던져야 함

1. catch가 모든 에러를 받음
2. `catch(err) {...}` 블록 안에서 에러 객체 `err`를 분석
3. 에러 처리 방법을 알지 못하면 `throw err`를 함

- `instanceof`로 에러 타입 체크
- `err.name` 프로퍼티로 에러 클래스 이름 확인
  - 기본형 에러는 모두 `err.name` 프로퍼티를 가짐
- `err.constructor.name`도 사용 가능

```javascript
let json = '{ "age": 30 }';
try {
  user = JSON.parse(json); // 변수 선언 누락
} catch (err) {
  alert("JSON Error: " + err); // JSON Error: ReferenceError: user is not defined
  // 실제로는 JSON 에러가 아님
}

try {
  anotherUser = {};
} catch (err) {
  if (err instanceof ReferenceError) {
    alert("ReferenceError");
  }
}

function readData() {
  let json = '{ "age": 30 }';
  try {
    // ...
    blabla(); // 에러!
  } catch (e) {
    // ...
    if (!(e instanceof SyntaxError)) throw e; // 알 수 없는 에러 다시 던지기
  }
}
try {
  readData(); // 내부에서 에러 발생
} catch (e) {
  alert("External catch got: " + e); // 에러를 잡음
}
```

### try...catch...finally

- `finally`라는 코드 절을 가질 수 있음. `finally` 안의 코드는 다음과 같은 상황에서 실행
  - 에러가 없는 경우: `try` 실행이 끝난 후
  - 에러가 있는 경우: `catch` 실행이 끝난 후
- 무언가를 실행하고 실행 결과에 상관 없이 실행을 완료하고 싶을 경우 사용
- `try..catch..finally` 안의 변수는 지역 변수
- `finally` 절은 `try..catch` 절을 빠져나가는 어떤 경우에도 실행
- `return`으로 명시적으로 빠져나가려는 경우 역시 실행
- `catch` 절이 없는 `try..finally`도 유용
  - 에러를 처리하고 싶지 않지만, 시작한 프로세스가 마무리 되었는지 확실히 하고 싶은 경우

```javascript
try {
  // 코드를 실행
} catch (e) {
  // 에러 핸들링
} finally {
  // 항상 실행
}
```

```javascript
function func() {
  try {
    return 1;
  } catch (e) {
  } finally {
    alert("finally"); // return이 있어도 실행됨
  }
}
alert(func()); // finally, 1

// try..finally
function func() {
  try {
    // ...
  } finally {
    // 스크립트가 죽더라도 완료됨
  }
}
```

### 전역 catch

- `try..catch` 바깥에서 치명적인 에러가 발생해 스크립트가 죽은 경우
- 어딘가에 에러 내역을 기록하거나 사용자에게 에러가 발생했음을 알려주는 행위를 하는 것이 좋음
- 자바스크립트 명세서에는 이런 치명적인 에러에 대응하는 방법이 적혀있지 않음
- 하지만 `try..catch`에서 처리하지 못한 에러를 잡는 것은 아주 중요
- 그래서 자바스크립트 호스트 환경 대다수는 자체적으로 에러 처리 기능을 제공
  - 브라우저 환경: `window.onerror`
  - Node.js: `process.on("uncaughtException")`

`window.onerror` 프로퍼티

```javascript
window.onerror = function (message, url, line, col, error) {};
```

- 전역 핸들러. 여기에 함수를 할당하면 예상치 못한 에러가 발생했을 때 실행됨
- `message`: 에러 메시지
- `url`: 에러가 발생한 스크립트의 URL
- `line`, `col`: 에러가 발생한 곳의 줄과 열 번호
- `error`: 에러 객체
- 죽은 스크립트를 복구하려는 목적으로는 잘 사용되지 않음
  - 프로그래밍 에러가 발생한 경우 `window.onerror` 만으로 스크립트를 복구하는 것은 사실상 불가능
- 개발자에게 에러 메시지를 보내는 용도로 사용

```html
<script>
  window.onerror = function (message, url, line, col, error) {
    alert(`${message}]n At ${line}:${col} of ${url}`);
  };
  function readData() {
    badFunc(); // 에러가 발생한 장소
  }
  readData();
</script>
```

`window.onerror`를 제외한 에러 로깅 관련 사용 서비스가 여러 가지 있음

0. 이런 서비스들은 다음과 같은 프로세스로 동작
1. 서비스를 가입하면 자바스크립트 파일(혹은 스크립트 URL)을 받는데, 개발자는 이 파일을 페이지에 삽입
2. 받은 파일은 커스텀 `window.onerror` 함수를 설정함
3. 에러가 발생하면, 이 커스텀 함수가 에러에 관한 내용을 담아 서비스에 네트워크 요청을 보냄
4. 개발자는 서비스 사이트에 로그인해 기록된 에러를 확인

## 커스텀 에러와 에러 확장

커스텀 에러

- 자체 에러 클래스가 필요한 경우
- 네트워크 작업 관련 `HttpError`, 데이터베이스 작업 관련 `DbError`, 검색 작업 관련 `NotFoundError`
- 이 에러들은 `message`나 `name`, 가능하다면 `stack` 프로퍼티를 지원해야 함
- `HttpError` 클래스가 있다면 `statusCode` 프로퍼티를 만들 수도 있음

`Error`를 상속받아 커스텀 에러 생성하기

- `throw`의 인수에는 제약이 없음
- 커스텀 에러 클래스는 반드시 `Error`를 상속할 필요가 없음
- 그러나 `Error`를 상속받아 커스텀 에러 클래스를 만들면 `obj instanceof Error`로 에러 객체 식별 가능

### 에러 확장하기

`Error` 클래스의 슈도 코드

```javascript
class Error {
  constructor(message) {
    this.message = message;
    this.name = "Error";
    this.stack = <call stack>; // 표준은 아니지만 대다수 환경이 지원함
  }
}
```

`Error` 상속받기

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message); // 부모 생성자 호출
    this.name = "ValidationError"; // name 재설정
  }
}
function test() {
  throw new ValidationError("에러 발생!");
}
try {
  test();
} catch (err) {
  alert(err.message); // 에러 발생!
  alert(err.name); // ValidationError
  alert(err.stack);
}

function readUser(json) {
  let user = JSON.parse(json);
  if (!user.age) throw new ValidationError("No field: age");
  if (!user.name) throw new ValidationError("No field: name");
  return user;
}
try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No field: name
  } else if (err instanceof SyntaxError) {
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // 알려지지 않은 에러는 다시 던짐
  }
}
```

에러 유형 확인하기

- `err.name`으로 확인 가능. `else if (err.name == 'SyntaxError') {...}`
- `instanceof` 사용을 권장
- `ValidationError`를 확장하여 `PropertyRequiredError` 같은 새로운 확장 에러를 만드는 경우
- `instanceof`는 새로운 상속 클래스에서도 동작하기 때문

### 더 깊게 상속하기

커스텀 에러 클래스의 `name`

- 매번 커스텀 에러 클래스의 생성자 안에서 `this.name`을 할당해 주는 것은 번거로움
- 기본 에러 클래스를 만들고 커스텀 에러들이 이 클래스를 상속받게 하면 됨
- 기본 에러의 생성자에 `this.name = this.constructor.name`을 추가하면 됨

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError"; // name 설정
  }
}

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.name = "PropertyRequiredError"; // name 설정
    this.property = property;
  }
}

function readUser(json) {
  let user = JSON.parse(json);
  if (!user.age) throw new PropertyRequiredError("age");
  if (!user.name) throw new PropertyRequiredError("name");
  return user;
}

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No property: name
    alert(err.name); // PropertyRequiredError
    alert(err.property); // name
  } else if (err instanceof SyntaxError) {
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // 알려지지 않은 에러는 재던지기함
  }
}
```

```javascript
// 기본 에러 클래스 MyError
class MyError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
  }
}

class ValidationError extends MyError {}

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.property = property;
  }
}

alert(new PropertyRequiredError("field").name); // PropertyRequiredError
```

### 예외 감싸기

예외 감싸기는 다음 순서로 진행

1. 데이터 읽기와 같은 포괄적인 에러를 대변하는 새로운 클래스 `ReadError`를 만듦
2. 함수 `readUser` 발생한 `ValidationError`, `SyntaxError` 등의 에러는 `readUser` 내부에서 잡음
   - 이때 `ReadError`를 생성
3. `ReadError` 객체의 `cause` 프로퍼티에는 실제 에러에 대한 참조가 저장됨

## 참고

> [에러 핸들링](https://ko.javascript.info/error-handling)
