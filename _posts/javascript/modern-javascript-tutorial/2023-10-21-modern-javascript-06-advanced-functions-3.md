---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 3
date: 2023-10-21 13:07:22 +0900
last_modified_at: 2023-11-30 09:38:26 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

전역 객체, 객체로서의 함수와 기명 함수 표현식, new Function 문법

## 전역 객체

전역 객체

- 어디서나 사용 가능한 변수나 함수 생성 가능
- 언어나 자체 호스트 환경에 기본 내장되어 있는 경우가 많음
  - 브라우저 환경: `window`
  - Node.js 환경: `global`
- 전역 객체 이름을 `globalThis`로 표준화하자는 내용이 최근 자바스크립트 명세에 추가됨

전역 객체의 모든 프로퍼티는 직접 접근 가능

```javascript
alert("Hello");
window.alert("Hello"); // 위와 동일하게 동작
```

브라우저에서 `let`이나 `const`가 아닌 `var`로 선언한 전역 함수나 전역 변수는 전역 객체의 프로퍼티가 됨

- 하위 호환성 때문에 아래와 같이 전역 객체를 사용해도 동작은 하지만 권장하지 않음
- 모듈을 사용하는 모던 자바스크립트는 이런 방식을 지원하지 않음

```javascript
var gVar = 5;
alert(window.gVar); // 5. var로 선언한 변수는 전역 객체 window의 프로퍼티가 됨
let gLet = 5;
alert(window.gLet); // undefined
```

중요한 변수라서 모든 곳에서 사용할 수 있게 하려면, 전역 객체에 직접 프로퍼티를 추가하는 것이 나음

```javascript
window.currentUser = { name: "John" };
alert(currentUser.name); // John
alert(window.currentUser.name); // 지역 변수 currentUser가 있다면 충돌 없이 전역 객체 window에서 명시적으로 가져올 수 있음
```

전역 변수는 되도록 사용하지 않는 것이 좋음

함수를 만들 때는 외부 변수나 전역 변수를 사용하는 것보다 input 변수를 받고 이를 이용해 output을 만들어내게 해야 테스트도 쉽고 에러도 덜 발생

### 폴리필 사용하기

전역 객체를 이용해 현재 사용중인 브라우저가 최신 자바스크립트 기능을 지원하는지 여부 확인 가능

```javascript
if (!window.Promise) {
  alert("구식 브라우저 사용중");
}
```

- 구식 브라우저는 `Promise` 객체를 지원하지 않음

명세에는 있는 기능이지만 해당 기능을 지원하지 않는 오래된 브라우저를 사용하고 있다면, 직접 함수를 만들어 전역 객체에 추가하는 방식으로 폴리필을 만들 수 있음

```javascript
if (!window.Promise) {
  window.Promise = "모던 자바스크립트에서 지원하는 기능을 직접 구현..";
}
```

자바스크립트에서 전역(global)이라는 용어는 '함수 안에 있지 않은(not inside the function)'이라고 생각하면 좋음

전역에 변수, 함수 등을 정의하게 되면 전역 객체(브라우저에서 window 객체, Node.js에서 global 객체)를 통해 해당 변수나 함수에 접근할 수 있음

## 객체로서의 함수와 기명 함수 표현식

자바스크립트에서 함수는 값으로 취급

- 모든 값은 자료형을 갖고, 함수는 객체

함수는 호출이 가능한(callable) 행동 객체

- 함수를 호출할 수 있고, 객체처럼 함수에 프로퍼티를 추가·제거하거나 참조를 통해 전달 가능

### name 프로퍼티

name 프로퍼티를 사용해 함수의 이름을 가져올 수 있음

```javascript
function sayHi() {}
alert(sayHi.name); // sayHi
```

익명 함수라도 자동으로 이름이 할당됨

```javascript
let sayHi = function () {};
alert(sayHi.name); // sayHi
let sayHi = function func() {};
alert(sayHi.name); // func
```

기본값을 사용해 이름을 할당해도 동일

```javascript
function f(sayHi = function () {}) {
  alert(sayHi.name);
}
f(); // sayHi
```

contextual name

- 자바스크립트 명세서에 정의된 이 기능을 contextual name이라 부름
- 이름이 없는 함수의 이름을 지정할 때는 컨텍스트에서 이름을 가져옴

객체 메서드의 이름도 name 프로퍼티로 가져올 수 있음

- 객체 메서드 이름은 함수처럼 자동 할당이 되지 않음
- 적절한 이름을 추론하는 게 불가능한 상황에는 name 프로퍼티에 빈 문자열이 저장됨
  - 엔진이 이름을 설정할 수 없어서 name 프로퍼티의 값이 빈 문자열이 됨
  - 실무에서 대부분의 함수는 이름이 있으므로 잘 발생하지 않는 상황

```javascript
let user = {
  sayHi() {},
  sayBye: function () {}
};
alert(user.sayHi.name); // sayHi
alert(user.sayBye.name); // sayBye
```

```javascript
let arr = [function () {}];
alert(arr[0].name); // <빈 문자열>
```

### length 프로퍼티

내장 프로퍼티 `length`는 함수 매개변수의 개수를 반환

- 나머지 매개변수는 매개변수 개수에 포함되지 않음

```javascript
function f1(a) {}
function f2(a, b) {}
function many(a, b, ...more) {}
alert(f1.length); // 1
alert(f2.length); // 2
alert(many.length); // 2
```

다른 함수 안에서 동작하는 함수의 타입을 검사(type introspection)할 때도 사용됨

```javascript
function ask(question, ...handlers) {
  let isYes = confirm(question);
  for (let handler of handlers) {
    if (handler.length == 0) {
      if (isYes) handler();
    } else {
      handler(isYes);
    }
  }
}
ask(
  "질문 있으신가요?",
  () => alert("OK를 선택하셨습니다."),
  (result) => alert(result)
);
// OK를 클릭한 경우, 핸들러 두 개를 모두 호출
// Cancel을 클릭한 경우, 두 번째 핸들러만 호출
```

다형성(polymorphism)

- 인수의 종류에 따라 인수를 다르게 처리하는 방식
- 다양한 형태의 동작
- 하나 이상의 동작을 하는 함수

### 커스텀 프로퍼티

함수에 자체적으로 만든 프로퍼티 추가 가능

```javascript
function sayHi() {
  alert("Hi");
  sayHi.counter++;
}
sayHi.counter = 0;
sayHi(); // Hi
sayHi(); // Hi
alert(`호출 횟수: ${sayHi.counter}회`); // 호출 횟수: 2회
```

프로퍼티는 변수가 아님

- `sayHi.counter = 0`과 같이 함수에 프로퍼티를 할당해도 함수 내에 지역 변수 `counter`가 만들어지지 않음
- `counter` 프로퍼티와 `let counter`는 전혀 관계 없음

프로퍼티를 저장하는 것처럼 함수를 객체처럼 다룰 수 있지만, 이는 실행에 아무 영향을 끼치지 않음

변수는 함수 프로퍼티가 아니고 함수 프로퍼티는 변수가 아니기 때문

- 둘 사이에는 공통점이 없음

클로저는 함수 프로퍼티로 대체 가능

```javascript
function makeCounter() {
  // let count = 0;
  function counter() {
    return counter.count++;
  }
  counter.count = 0;
  return counter;
}
let counter = makeCounter();
alert(counter()); // 0
alert(counter()); // 1
counter.count = 10; // 외부에서 접근 가능
```

- 이제 `count`는 외부 렉시컬 환경이 아닌 함수 프로퍼티에 바로 저장됨
- 이렇게 함수 프로퍼티에 정보를 저장하는 게 클로저를 사용하는 것보다 나은 방법인가?

`count` 값이 외부 변수에 저장되어 있는 경우 두 방법의 차이가 드러남

- 클로저를 사용한 경우 외부 코드에서 `count`에 접근할 수 없음
  - 오직 중첩 함수 내에서만 `count` 값 수정 가능
- 함수 프로퍼티를 사용해 `count`를 함수에 바인딩시킨 경우 외부에서 값 수정 가능
- 목적에 따라 구현 방법을 선택하면 됨

### 기명 함수 표현식

기명 함수 표현식(Named Function Expression, NFE)

- 이름이 있는 함수 표현식을 나타내는 용어

일반 함수 표현식

```javascript
let sayHi = function (who) {
  alert(`Hello, ${who}`);
};
```

이름 붙이기

```javascript
let sayHi = function func(who) {
  alert(`Hello, ${who}`);
};
sayHi("John"); // Hello, John
```

- 이름 `func`를 붙임

이름을 붙여도 위 함수는 여전히 함수 표현식

- 여전히 표현식을 할당한 형태를 유지하기 때문에 함수 선언문으로 바뀌지 않음
- 이름을 추가한다고 기존에 동작하던 기능이 동작하지 않는 일은 없음

이름을 붙이면 두 가지 변화가 생김

1. 이름을 사용해 함수 표현식 내부에서 자기 자신을 참조 가능
2. 기명 함수 표현식 외부에서는 그 이름을 사용할 수 없음

```javascript
let sayHi = function func(who) {
  if (who) alert(`Hello, ${who}`);
  else func("Guest");
};
sayHi(); // Hello, Guest
func(); // ReferenceError: func is not defined
```

- `sayHi` 대신 `func`로 호출한 이유
  - 아래와 같이 코드를 작성하면 외부 코드에 의해 `sayHi`가 변경될 수 있음
  - 함수가 `sayHi`를 자신의 외부 렉시컬 환경에서 가져오기 때문에 에러 발생
  - 지역 렉시컬 환경엔 `sayHi`가 없기 때문에 외부 렉시컬 환경에서 `sayHi`를 찾음
  - 함수 호출 시점에 외부 렉시컬 환경의 `sayHi`엔 `null`이 저장되어 있기 때문에 에러 발생
  - 함수 표현식에 이름을 붙여주면 해결 가능

```javascript
let sayHi = function (who) {
  if (who) alert(`Hello, ${who}`);
  else sayHi("Guest");
};
let welcome = sayHi;
sayHi = null;
welcome(); // TypeError: sayHi is not a function
```

```javascript
let sayHi = function func(who) {
  if (who) alert(`Hello, ${who}`);
  else func("Guest");
};
let welcome = sayHi;
sayHi = null;
welcome(); // Hello, Guest
```

- `func`라는 이름은 함수 지역 수준(function-local)에 존재하므로 외부 렉시컬 환경에서 찾지 않아도 됨
  - 외부 렉시컬 환경에서는 보이지도 않음
- 함수 표현식에 붙인 이름은 현재 함수만 참조하도록 명세서에 정의됨
- 기명 함수 표현식을 사용하면 `sayHi`나 `welcome` 같은 외부 변수의 변경과 관계 없이 `func`라는 내부 함수 이름을 사용해 언제든 함수 표현식 내부에서 자기 자신 호출 가능

많은 자바스크립트 라이브러리에서 함수 커스텀 프로퍼티를 잘 활용하고 있음

- 주요 함수를 하나 만들고 여기에 다양한 헬퍼 함수를 붙이는 식으로 구성됨
  - jQuery는 이름이 `$`인 주요 함수로 이루어짐
  - lodash는 주요 함수 `_`에 `_.clone`, `_.keyBy` 등의 프로퍼티를 추가하는 식으로 구성됨
- 함수 하나에 다양한 헬퍼 함수를 붙여 라이브러리를 만들면 라이브러리 하나가 전역 변수 하나만 사용하므로 전역 공간을 더럽히지 않음
  - 이름 충돌도 방지

### 예시

```javascript
function makeCounter() {
  let count = 0;
  function counter() {
    return count++;
  }
  counter.set = (value) => (count = value);
  counter.decrease = () => count--;
  return counter;
}
```

- 지역변수 `count`를 사용
- 추가 메서드는 함수 `counter`에 정의
- 함수 `counter`에 할당된 메서드들은 동일한 외부 렉시컬 환경을 공유하고 동일한 `count`에 접근 가능

```javascript
function sum(a) {
  let currentSum = a;
  function f(b) {
    currentSum += b;
    return f; // f()로 재귀 호출하지 않고 자신을 반환만 함
  }
  f.toString = function () {
    return currentSum;
  };
  return f;
}
alert(sum(1)(2)); // 3
alert(sum(5)(-1)(2)); // 6
alert(sum(6)(-1)(-2)(-3)); // 0
alert(sum(0)(1)(2)(3)(4)(5)); // 15
```

- `sum`은 실제로 한 번만 동작
  - 함수 `sum`은 함수 `f`를 반환

## new Function 문법

함수 표현식과 함수 선언문 이외에 함수를 만들 수도 있는 또 하나의 방법

잘 사용하는 방법은 아니지만 이 방법 외에는 대안이 없을 때 사용

### 문법

`new Function`을 사용해 함수 생성

```javascript
let func = new Function([arg1, arg2, ...argN], functionBody);
```

- 새로 만들어지는 함수는 인수 `arg1...argN`과 함수 본문 `functionBody`로 구성

```javascript
let sum = new Function("a", "b", "return a + b");
alert(sum(1, 2)); // 3
let sayHi = new Function('alert("Hello")');
sayHi(); // Hello
```

기존의 방법과 `new Function`의 차이점

- 런타임에 받은 문자열을 사용해 함수 생성 가능
- 어떤 문자열도 함수로 변경할 수 있음
  - 서버에서 전달받은 문자열을 이용해 새로운 함수를 만들고 실행 가능

```javascript
let str = "서버에서 동적으로 전달받은 문자열(코드 형태)";
let func = new Function(str);
func();
```

### 클로저

함수는 특별한 프로퍼티 `[[Environment]]`에 저장된 정보를 이용해 자기 자신이 태어난 곳을 기억

`[[Environment]]`는 함수가 만들어진 렉시컬 환경을 참조

`new Function`을 이용해 함수를 만들면 함수의 `[[Environment]]` 프로퍼티가 현재 렉시컬 환경이 아닌 전역 렉시컬 환경을 참조

- `new Function`을 이용해 만든 함수는 외부 변수에 접근할 수 없고 오직 전역 변수에만 접근 가능

```javascript
function getFunc() {
  let value = "test";
  let func = new Function("alert(value)");
  return func;
}
getFunc()(); // ReferenceError: value is not defined
```

```javascript
function getFunc() {
  let value = "test";
  let func = function () {
    alert(value);
  };
  return func;
}
getFunc()(); // test
```

실무에서 유용하게 쓰임

문자열을 사용해서 함수를 만드는 경우, 서버를 비롯한 외부에서 코드를 받아올 수 있음

- 이렇게 만들어진 새 함수는 기존 스크립트와 문제 없이 상호작용할 수 있어야 함
- `new Function`으로 만든 새 함수 내부에서 외부 변수에 접근하려 할 때, 기존 함수 선언 방식으로 작성한 함수와 동일한 동작이 보장되어야 함

그런데 스크립트가 프로덕션 서버에 반영되기 전, 압축기(minifier)에 의해 압축될 때 문제 발생

- 압축기는 스크립트에서 주석이나 여분의 공백 등을 없애 코드 크기를 줄여주는 프로그램
- 압축기가 지역 변수 이름을 짧게 바꾸면서 문제 발생

함수 내부에 `let userName`이라는 변수가 있으면 이 지역 변수는 압축기에 의해 `let a` 등 짧은 이름으로 대체되는데, 이때 `userName` 모두가 `a`로 교체됨

- `userName`은 지역 변수이고, 함수 외부에선 함수 내부에 있는 변수에 접근할 수 없기 때문에 문제 없음

압축기는 단순히 변수를 찾아서 바꾸지 않고 코드 구조를 분석해 기존에 작성한 코드의 기능을 망가뜨리지 않으면서 영리하게 제 역할을 수행

- 이런 동작 방식 때문에 `new Function`으로 만든 함수 내부에서 외부 변수에 접근하려고 하면 `userName`은 이미 이름이 변경되었기 때문에 찾을 수 없음

압축기가 동작한 이후에 `new Function`으로 만든 함수 내부에서 외부 렉시컬 환경에 접근하려고 할 때 문제가 발생할 수 있음

함수 내부에서 외부 변수에 접근하는 것은 아키텍처 관점에서도 좋지 않고 에러에 취약

`new Function`으로 만든 함수에 무언가를 넘겨주고 싶다면 인수를 사용

`new Function`을 이용해 만든 함수의 `[[Environment]]`는 외부 렉시컬 환경이 아닌 전역 렉시컬 환경을 참조하므로 외부 변수를 사용할 수 없음

- 단점 같아 보이지만 에러를 예방해주기 때문에 장점이 됨
- 구조상으로는 매개변수를 사용해 값을 받는 게 더 나음
- 압축기에 의한 에러도 방지

세 선언 방식은 동일하게 동작

```javascript
new Function("a", "b", "return a + b"); // 기본 문법
new Function("a,b", "return a + b"); // 쉼표로 구분
new Function("a , b", "return a + b"); // 쉼표와 공백으로 구분
```

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
