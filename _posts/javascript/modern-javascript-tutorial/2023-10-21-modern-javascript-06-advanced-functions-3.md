---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 3
date: 2023-10-21 13:07:22 +0900
last_modified_at: 2024-04-07 07:14:33 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [javascript, global-object, function, function-object, function-property, nfe]
---

전역 객체, 객체로서의 함수와 기명 함수 표현식, new Function 문법

## 전역 객체

전역(global)

- 자바스크립트에서 전역의 의미: 함수 안에 있지 않은(not inside the function)

전역 객체

- 어디서나 사용 가능한 변수, 함수 생성 가능
- 언어나 자체 호스트 환경에 기본 내장되어 있는 경우가 많음
  - 브라우저 환경은 `window`, Node.js 환경은 `global`
- 전역 객체 이름을 `globalThis`로 표준화하자는 내용이 최근 자바스크립트 명세에 추가됨
- 전역 객체의 모든 프로퍼티는 직접 접근 가능 (하위 호환성)

전역 변수

- 브라우저에서 `var`로 선언한 전역 함수나 전역 변수는 전역 객체의 프로퍼티가 됨
  - `let`, `const` 제외
  - 모듈을 사용하는 모던 자바스크립트는 이런 방식을 지원하지 않음
- 되도록 사용하지 않는 것이 좋음
- 중요한 변수라서 모든 곳에서 사용할 수 있게 하려면, 전역 객체에 직접 프로퍼티를 추가

```javascript
alert("Hello");
window.alert("Hello"); // 위와 동일하게 동작

var gVar = 5;
alert(window.gVar); // 5 (전역 객체 window의 프로퍼티가 됨)
let gLet = 5;
alert(window.gLet); // undefined

window.obj = { name: "test" };
alert(obj.name); // test
alert(window.obj.name); // 지역 변수 obj가 있다면 충돌 없이 전역 객체 window에서 명시적으로 가져올 수 있음
```

### 폴리필 사용하기

폴리필

- 전역 객체를 이용해 현재 사용중인 브라우저가 최신 자바스크립트 기능을 지원하는지 여부 확인
- 명세에는 있는 기능이지만 해당 기능을 지원하지 않는 오래된 브라우저를 사용하고 있는 경우,
- 직접 함수를 만들어 전역 객체에 추가하는 방식으로 폴리필 생성

```javascript
if (!window.Promise) {
  alert("구식 브라우저 사용중");
  window.Promise = "모던 자바스크립트에서 지원하는 기능을 직접 구현..";
}
```

## 객체로서의 함수와 기명 함수 표현식

객체로서의 함수

- 함수를 호출할 수 있음
  - 함수는 호출이 가능한(callable) 행동 객체
- 함수에 프로퍼티를 추가·제거하거나 참조를 통해 전달 가능
  - `name`, `length` 등
- 자바스크립트에서 함수는 값

### name 프로퍼티

`name` 프로퍼티

- 함수의 이름 반환
- 익명 함수라도 자동으로 이름이 할당되고 기본값으로 이름을 할당해도 동일
- contextual name: 이름이 없는 함수의 이름을 지정할 때는 컨텍스트에서 이름을 가져옴
  - 자바스크립트 명세서에 정의된 이 기능
- 객체 메서드도 `name`으로 이름을 가져올 수 있음
  - 함수처럼 자동 할당 되지 않음
  - 적절한 이름을 추론할 수 없으면 `name`에 빈 문자열이 저장됨

```javascript
function sayHi() {}
alert(sayHi.name); // sayHi

let func1 = function () {};
alert(func1.name); // func1 (익명 함수도 자동으로 이름 할당)
let func2 = function originalFunc() {};
alert(func2.name); // originalFunc

function testFunc(printName = function () {}) {
  alert(printName.name);
}
testFunc(); // printName (기본값으로 이름 할당)

let user = {
  sayHi() {},
  sayBye: function () {}
};
alert(user.sayHi.name); // sayHi (메서드 이름)
alert(user.sayBye.name); // sayBye (메서드 이름)

let arr = [function () {}];
alert(arr[0].name); // <빈 문자열>
```

### length 프로퍼티

`length` 프로퍼티

- 함수 매개변수의 개수 반환
- 나머지 매개변수는 매개변수 개수에 포함되지 않음
- 다른 함수 안에서 동작하는 함수의 타입을 검사(type introspection)할 때도 사용됨

```javascript
function f1(a) {}
function f2(a, b) {}
function many(a, b, ...more) {}
alert(f1.length); // 1
alert(f2.length); // 2
alert(many.length); // 2

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
  () => alert("OK를 선택하셨습니다."), // OK를 클릭 시 핸들러 두 개를 모두 호출
  (result) => alert(result) // Cancel을 클릭 시 두 번째 핸들러만 호출
);
```

다형성(polymorphism)

- 인수의 종류에 따라 인수를 다르게 처리하는 방식
- 다양한 형태의 동작
- 하나 이상의 동작을 하는 함수

### 커스텀 프로퍼티

함수 프로퍼티

- 함수에 커스텀 프로퍼티를 추가할 수 있음. 함수 프로퍼티는 변수가 아님
- 프로퍼티를 저장하는 것처럼, 함수를 객체처럼 다룰 수 있으나 이는 실행에 아무 영향을 끼치지 않음
  - 변수는 함수 프로퍼티가 아니고 함수 프로퍼티는 변수가 아님
- 클로저는 함수 프로퍼티로 대체 가능
- `sayHi.counter = 0`과 같이 함수에 프로퍼티를 할당해도 함수 내에 지역 변수 `counter`가 만들어지지 않음
  - `counter` 프로퍼티와 `let counter`는 전혀 관계 없음

함수 프로퍼티 vs 클로저

- 클로저를 사용한 경우
  - 외부 코드에서 `count`에 접근 불가
  - 오직 중첩 함수 내에서만 `count` 값에 접근 가능
- 함수 프로퍼티를 사용한 경우
  - `count`를 함수에 바인딩
  - 외부에서 값 수정 가능

```javascript
function sayHi() {
  alert("Hi");
  sayHi.counter++;
}
sayHi.counter = 0; // 함수 프로퍼티
sayHi(); // Hi
sayHi(); // Hi
alert(`호출 횟수: ${sayHi.counter}회`); // 호출 횟수: 2회

function makeCounter() {
  let closureCount = 0; // 클로저
  function counter() {
    closureCount++;
    counter.propertyCount++;
    return;
  }
  counter.propertyCount = 0; // 함수 프로퍼티
  return counter;
}
let counter = makeCounter();
counter();
console.log(counter.propertyCount); // 1
counter();
console.log(counter.closureCount); // undefined
```

많은 자바스크립트 라이브러리에서 함수 커스텀 프로퍼티를 잘 활용하고 있음

- 주요 함수를 하나 만들고, 여기에 다양한 헬퍼 함수를 붙이는 식으로 라이브러리를 구성
- 라이브러리 하나가 전역 변수 하나만 사용하므로 전역 공간을 더럽히지 않고 이름 충돌도 방지
  - jQuery: 이름이 `$`인 주요 함수로 구성
  - lodash: 주요 함수 `_`에 `_.clone`, `_.keyBy` 등의 프로퍼티를 추가하는 식으로 구성

### 기명 함수 표현식

기명 함수 표현식(Named Function Expression, `NFE`)

- 이름이 있는 함수 표현식
- 이름으로 함수 표현식 내부에서 자기 자신을 참조할 수 있음
- 기명 함수 표현식 외부에서는 그 이름을 사용할 수 없음
- 함수 표현식에 붙인 이름은 현재 함수만 참조하도록 명세서에 정의됨

```javascript
// 이름 func를 붙임
let sayHi = function func(who) {
  if (who) alert(`Hello, ${who}`);
  else sayHi("Guest"); // func로 호출하면 에러가 발생하지 않음
  // else func("Guest"); // 자기 자신을 참조
};
let welcome = sayHi;
sayHi = null;
welcome(); // TypeError: sayHi is not a function (함수 호출 시점에 외부 렉시컬 환경의 sayHi=null)
func(); // ReferenceError: func is not defined
```

### 예제

```javascript
function makeCounter() {
  let count = 0;
  function counter() {
    return count++;
  }
  counter.set = (value) => (count = value); // counter 함수에 함수 프로퍼티로 메서드 할당
  counter.decrease = () => count--; // 동일한 외부 렉시컬 환경을 공유해 동일한 count에 접근 가능
  return counter;
}
```

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

- `sum`은 실제로 한 번만 동작하고 함수 `sum`은 함수 `f`를 반환

## new Function 문법

`new Function`

- 함수 표현식과 함수 선언문 이외에 함수를 만들 수 있는 또 하나의 방법
- 잘 사용하는 방법은 아니지만, 이 방법 외에는 대안이 없을 때 사용

### 문법

`new Function(functionBody)`

```javascript
let func = new Function([arg1, arg2, ...argN], functionBody);
```

- `arg1...argN`: 만들어지는 함수의 인수
- `functionBody`: 함수 본문
- 런타임에 받은 문자열을 사용해 함수 생성 가능
- 어떤 문자열도 함수로 변경 가능
- 서버에서 전달받은 문자열을 이용해 새로운 함수를 만들고 실행할 수 있음

```javascript
let sum = new Function("a", "b", "return a + b");
alert(sum(1, 2)); // 3
let sayHi = new Function('alert("Hello")');
sayHi(); // Hello

let str = "서버에서 동적으로 전달받은 문자열(코드 형태)";
let func = new Function(str);
func();
```

### 클로저

`new Function`으로 함수를 만드는 경우

- 함수의 `[[Environment]]` 프로퍼티가 현재 렉시컬 환경이 아닌 전역 렉시컬 환경을 참조
  - `[[Environment]]`: 함수가 만들어진 렉시컬 환경을 참조하는 특별한 프로퍼티
- 매개변수를 사용해 인수를 전달
- 외부 변수에 접근할 수 없고 오직 전역 변수에만 접근 가능
  - 실무에서 유용하게 쓰이는 특징
  - 문자열을 사용해서 함수를 만드는 경우, 서버를 비롯한 외부에서 코드를 받아올 수 있음
  - `new Function`으로 만든 함수 내부에서 외부 변수에 접근할 때, 기존 함수 선언 방식으로 작성한 함수와 동일한 동작이 보장되어야 함
  - 그런데 스크립트가 프로덕션 서버에 반영되기 전, 압축기에 의해 압축될 때 문제 발생
    - 압축기가 지역 변수 이름을 짧게 바꾸기 때문
    - 함수 내부에 `let userName`이라는 지역 변수가 있으면,
    - 압축기에 의해 `let a` 등 짧은 이름으로 대체되어 `userName` 모두가 `a`로 교체됨
    - 함수 외부에서는 함수 내부의 변수에 접근할 수 없기 때문에 문제 없음
  - 이런 동작 방식 때문에 `new Function`으로 만든 함수 내부에서 외부 변수에 접근할 때,
  - `userName`은 이미 이름이 변경되었기 때문에 찾을 수 없음
  - 압축기가 동작한 이후에 `new Function`으로 만든 함수의 내부에서 외부 렉시컬 환경에 접근할 때 문제가 발생할 수 있음
  - 함수 내부에서 외부 변수에 접근하는 것은 아키텍처 관점에서도 좋지 않고 에러에 취약
- 압축기(minifier): 스크립트에서 주석이나 여분의 공백 등을 없애 코드 크기를 줄여주는 프로그램
  - 단순히 변수를 찾아서 바꾸지 않음
  - 코드 구조를 분석해 기존에 작성한 코드의 기능을 망가뜨리지 않으면서 영리하게 제 역할을 수행

```javascript
function getFunc() {
  let value = "test";
  let func = new Function("alert(value)");
  // let func = () => alert(value); // test
  return func;
}
getFunc()(); // ReferenceError: value is not defined. value에 접근 불가
```

세 선언 방식은 동일하게 동작

```javascript
new Function("a", "b", "return a + b"); // 기본 문법
new Function("a,b", "return a + b"); // 쉼표로 구분
new Function("a , b", "return a + b"); // 쉼표와 공백으로 구분
```

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
