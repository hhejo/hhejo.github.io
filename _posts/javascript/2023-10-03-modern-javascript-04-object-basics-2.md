---
title: 모던 JavaScript 튜토리얼 04 - 객체 기본 2
date: 2023-10-03 20:13:24 +0900
last_modified_at: 2023-10-03 20:13:24 +0900
categories: [JavaScript]
tags: [javascript]
---

sdkfjasldfdjslk

## new 연산자와 생성자 함수

- 객체 리터럴 `{...}`을 사용해 객체를 쉽게 생성할 수 있지만, 여러 개 만들기 불편
- 유사한 객체를 여러 개 만들어야 할 때 유용한 `new` 연산자와 생성자 함수

### 생성자 함수(constructor function)

- 생성자 함수와 일반 함수에 기술적 차이는 없으나, 생성자 함수는 아래 관례를 따름
  - 함수 이름의 첫 글자는 대문자로 시작
  - 반드시 `new` 연산자를 붙여 실행
- 생성자를 이용해 재사용할 수 있는 객체 생성 코드 구현 가능
- 모든 함수는 생성자 함수가 될 수 있음. 어떤 함수라도 `new`를 붙여 실행하면 생성자 알고리즘 실행

```javascript
function User(name) {
  // this = {}; (빈 객체가 암시적으로 생성됨)
  this.name = name;
  this.isAdmin = false;
  // return this; (this가 암시적으로 반환됨)
}
let user = new User("보라"); // 아래 코드와 동일하게 동작
let user = {
  name: "보라",
  isAdmin: false
};
```

### 익명 생성자 함수

- 재사용은 막으면서 코드 캡슐화 가능

```javascript
let user = new (function () {
  this.name = "John";
  this.isAdmin = false;
})();
```

### new.target과 생성자 함수

- `new.target` 프로퍼티를 사용하면 함수가 `new`와 함께 호출되었는지 아닌지 알 수 있음
- 일반적인 방법으로 함수 호출: `new.target`이 `undefined` 반환
- `new`와 함께 호출: `new.target`은 함수 자체를 반환

```javascript
function User() {
  alert(new.target);
}
User(); // undefined
new User(); // function User { ... }
```

### 생성자와 return문

- 생성자 함수에는 보통 `return`문이 없음
- 반환해야 할 것들은 모두 `this`에 저장되고, `this`는 자동으로 반환되기 때문
- `return`문이 있으면
  - 객체를 `return`한다면 `this` 대신 객체 반환
  - 원시형을 `return`한다면 `return`문이 무시됨
- 즉, `return` 뒤에 객체가 오면 생성자 함수는 해당 객체를 반환하고, 이외의 경우는 `this` 반환
- `return`문이 있는 생성자 함수는 거의 없음

```javascript
function BigUser() {
  this.name = "원숭이";
  return { name: "고릴라" }; // this가 아닌 새로운 객체 반환
}
alert(new BigUser().name); // 고릴라
```

```javascript
function SmallUser() {
  this.name = "원숭이";
  return; // this를 반환
}
alert(new SmallUser().name); // 원숭이
```

### 괄호 생략하기

- 인수가 없는 생성자 함수는 괄호를 생략해 호출 가능
- 그래도 괄호 쓰는 것을 권장

```
let user = new User;
let user = new User(); // 위 코드와 동일하게 동작
```

### 생성자 내 메서드

- 생성자 함수를 사용하면 매개변수를 이용해 객체 내부를 자유롭게 구성할 수 있음
- 메서드도 더할 수 있음
- `class` 문법을 사용하면 생성자 함수를 사용하는 것과 마찬가지로 복잡한 객체 생성 가능

```javascript
function User(name) {
  this.name = name;
  this.sayHi = function () {
    alert("제 이름은 " + this.name + "입니다.");
  };
}
let bora = new User("이보라");
bora.sayHi(); // 제 이름은 이보라입니다.
```

## 옵셔널 체이닝 '?.'

- 옵셔널 체이닝(optional chaining) `?.`으로 프로퍼티가 없는 중첩 객체를 에러 없이 안전하게 접근
- 연산자가 아닌 함수나 대괄호와 함께 동작하는 특별한 문법 구조체(syntax construct)

```javascript
let user = {}; // 주소 정보 없음
alert(user.address.street); // TypeError: Cannot read properties of undefined (reading 'street')
```

```javascript
// querySelector(...) 호출 결과가 null인 경우 에러 발생
let html = document.querySelector(".my-element").innerHTML;
```

### 옵셔널 체이닝의 등장 이전

- `&&` 연산자를 사용해 에러 방지

```javascript
let user = {};
alert(user && user.address && user.address.street); // undefined (에러가 발생하지 않음)
```

### 옵셔널 체이닝의 등장

- `?.`은 앞의 평가 대상이 `undefined`나 `null`이면 평가를 멈추고 `undefined`를 반환
- ```javascript
  let user = {};
  alert(user?.address?.street); // undefined (에러가 발생하지 않음)
  ```
- `user`가 `null`이나 `undefined`가 아닌 경우, `user.address` 프로퍼티가 존재하지 않는다면 에러 발생
- ```javascript
  let user = null;
  alert(user?.address); // undefined
  alert(user?.address.street); // undefined
  ```
- 변수 `user`가 선언되어 있지 않으면 에러 발생
- ```javascript
  user?.address; // ReferenceError: user is not defined
  ```
- `?.`는 왼쪽 평가 대상에 값이 없으면 평가 멈춤

### ?.()와 ?.[]

```javascript
let user1 = {
  admin() {
    alert("관리자 계정입니다.");
  }
};
let user2 = {};
user1.admin?.(); // 관리자 계정입니다.
user2.admin?.();
```

```javascript
let user1 = { firstName: "Violet" };
let user2 = null;
let key = "firstName";
alert(user1?.[key]); // Violet
alert(user2?.[key]); // undefined
alert(user1?.[key]?.something?.not?.existing); // undefined
```

```javascript
delete user?.name; // user가 존재하면 user.name을 삭제
```

## 심볼형

- 객체 프로퍼티 키로 문자형과 심볼형만 허용

### 심볼(symbol)

- 유일한 식별자(unique identifier)를 만들고 싶을 때 사용
- 유일성을 보장하기 때문에 동일 심볼을 여러 개 만들어도 각 심볼값은 다름
- 심볼에 붙이는 설명(심볼 이름)은 어떤 것에도 영향을 주지 않는 이름표 역할
- 심볼형 값은 다른 자료형으로 암시적 형 번환(자동 형 변환)이 되지 않음

```javascript
let id = Symbol(); // id는 심볼
let id = Symbol("id"); // 'id'라는 설명이 붙는 심볼 id
```

```javascript
let id1 = Symbol("id");
let id2 = Symbol("id");
alert(id1 === id2); // false
```

```javascript
let id = Symbol("id");
alert(id); // TypeError: Cannot convert a Symbol value to a string
alert(id.toString()); // Symbol(id)
alert(id.description); // id
```

### '숨김' 프로퍼티

- 외부 코드에서 접근이 불가능하고 값도 덮어쓸 수 없는 프로퍼티

```javascript
let user = { name: "John" }; // 서드파티 코드에서 가져온 객체
let id = Symbol("id");
user[id] = 1;
alert(user[id]); // 심볼을 키로 사용해 데이터에 접근
```

```javascript
let id = Symbol("id");
user[id] = "제 3 스크립트 id 값";
```

```javascript
let user = { name: "John" };
user.id = "스크립트 id 값";
user.id = "제3 스크립트 id 값"; // 의도치 않게 덮어 씀
```

- 객체 리터럴 `{...}`을 사용해 객체를 만든 경우, 대괄호를 사용해 심볼형 키를 만들어야 함

```javascript
let id = Symbol("id");
let user = { name: "John", [id]: 123 };
```

- 심볼은 `for...in`에서 배제됨
- `Object.assign`은 키가 심볼인 프로퍼티를 배제하지 않고 객체 내 모든 프로퍼티를 복사

### 전역 심볼

- 전역 심볼 레지스트리 안에 있는 심볼: 전역 심볼
- 전역 심볼 레지스트리(global symbol registry)를 이용해 이름이 같은 심볼이 같은 개체를 가리킬 수 있음
- `Symbol.for(key)`를 사용해 레지스트리 안에 있는 심볼을 읽거나, 새로운 심볼을 생성
- 해당 메서드를 호출하면 이름이 `key`인 심볼을 반환. 조건에 맞는 심볼이 레지스트리 안에 없으면 새로운 심볼 `Symbol(key)`를 만들고 레지스트리 안에 저장

```javascript
let id = Symbol.for("id");
let idAgain = Symbol.for("id");
alert(id === idAgain); // true
```

- `Symbol.keyFor(sym)`를 사용하면 이름을 얻을 수 있음
- 전역 심볼 레지스트리를 뒤져서 해당 심볼의 이름을 얻어냄
- 전역 심볼이 아닌 인자가 넘어오면 `undefined`를 반환

```javascript
// 이름을 이용해 심볼을 찾음
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");
// 심볼을 이용해 이름을 얻음
alert(Symbol.keyFor(sym)); // name
alert(Symbol.keyFor(sym2)); // id
```

- 전역 심볼이 아닌 모든 심볼은 `description` 프로퍼티가 있음
- 일반 심볼에서 이름을 얻고 싶으면 `description` 프로퍼티 사용

```javascript
let globalSymbol = Symbol.for("name");
let localSymbol = Symbol("name");
alert(Symbol.keyFor(globalSymbol)); // name, 전역 심볼
alert(Symbol.keyFor(localSymbol)); // undefined, 전역 심볼이 아님
alert(localSymbol.description); // name
```

### 시스템 심볼

- 자바스크립트 내부에서 사용되는 심볼
- 객체를 미세 조정할 수 있음
- `Symbol.hasInstance`, `Symbol.isConcatSpreadable`, `Symbol.iterator`, `Symbol.toPrimitive`, ...

## 객체를 원시형으로 변환하기

- 객체는 논리 평가시 `true`를 반환
- 객체끼리 빼는 연산을 할 때나 수학 관련 함수를 적용할 때 숫자형으로의 형 변환 발생
- 문자형으로의 형 변환은 대개 `alert(obj)` 같이 객체를 출력하려고 할 때 발생

### ToPrimitive

## 참고

> [객체: 기본](https://ko.javascript.info/object-basics)
