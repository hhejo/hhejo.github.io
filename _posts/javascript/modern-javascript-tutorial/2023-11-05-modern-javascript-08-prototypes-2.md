---
title: 모던 JavaScript 튜토리얼 08 - 프로토타입과 프로토타입 상속 2
date: 2023-11-05 11:54:16 +0900
last_modified_at: 2023-11-15 18:34:54 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

내장 객체의 프로토타입, 프로토타입 메서드와 `__proto__`가 없는 객체

## 내장 객체의 프로토타입

`prototype` 프로퍼티는 자바스크립트 내부에서도 광범위하게 사용됨

모든 내장 생성자 함수에서 `prototype` 프로퍼티를 사용

모든 내장 객체는 다음 규칙을 따름

- 메서드는 프로토타입에 저장됨(`Array.prototype`, `Object.prototype`, `Date.prototype` 등)
- 객체 자체에는 데이터만 저장함(배열의 요소, 객체의 프로퍼티, 날짜 등)

원시값 또한 래퍼 객체의 프로토타입에 `Number.prototype`, `String.prototype`, `Boolean.prototype` 같은 메서드를 저장함

- `undefined`, `null`은 예외

내장 프로토타입은 수정 가능

- 내장 프로토타입의 메서드를 빌려와 새로운 메서드를 만드는 것도 가능
- 내장 프로토타입 변경은 되도록 하지 않아야 함

### Object.prototype

```javascript
let obj = {};
alert(obj); // [object Object]
```

- `[object Object]` 문자열을 생성하는 코드는 어디에 있나?
- `obj = new Object()`를 줄이면 `obj = {}`가 됨
- `Object`는 내장 객체 생성자 함수
  - 이 생성자 함수의 `prototype`은 `toString`을 비롯한 다양한 메서드가 구현되어 있는 거대한 객체를 참조

```
Object               Object.prototype
[     ] -prototype-> [ constructor: Object ]
                     [ toString: function  ]
                     [ ...                 ]
                                ^
                                | [[Prototype]]
                     obj = new Object()
                     [                     ]
```

- 따라서 `obj.toString()`을 호출하면 `Object.prototype`에서 해당 메서드를 가져옴

```javascript
let obj = {};
alert(obj.__proto__ === Object.prototype); // true
alert(obj.toString === obj.__proto__.toString); // true
alert(obj.toString === Object.prototype.toString); // true
```

`Object.prototype` 위쪽에는 `[[Prototype]]` 체인이 없음

```javascript
alert(Object.prototype.__proto__); // null
alert(Object.prototype.prototype); // undefined
```

### 다양한 내장 객체의 프로토타입

`Array`, `Date`, `Function`을 비롯한 내장 객체들 역시 프로토타입에 메서드를 저장해 놓음

배열 `[1, 2, 3]`을 만들면 `new Array()`의 디폴트 생성자가 내부에서 동작하여 `Array.prototype`이 배열 `[1, 2, 3]`의 프로토타입이 되고 개발자는 `Array.prototype`을 통해 배열 메서드를 사용할 수 있음

- 이런 내부 동작은 메모리 효율을 높여줌

명세서에서는 모든 내장 프로토타입의 상속 트리 꼭대기에는 `Object.prototype`이 있어야 한다고 규정

- 모든 것은 객체를 상속받는다고 하기도 함

```
                                       null
                                        ^
                                        | [[Prototype]]
                            Object.prototype
                            [ toString: function   ]
                            [ other object methods ]
          ^                              ^                             ^
          | [[Prototype]]                | [[Prototype]]               | [[Prototype]]
Array.prototype             Function.prototype              Number.prototype
[ slice: function    ]      [ call: function         ]      [ toFixed: function    ]
[ other array methods]      [ other function methods ]      [ other number methods ]
     ^                                   ^                    ^
     | [[Prototype]]                     | [[Prototype]]      | [[Prototype]]
[ 1, 2, 3 ]                 [ function f(args) { ... } ]    [ 5 ]
```

```javascript
let arr = [1, 2, 3];
alert(arr.__proto__ === Array.prototype); // true
alert(arr.__proto__.__proto__ === Object.prototype); // true
alert(arr.__proto__.__proto__.__proto__); // null
```

```javascript
alert(Array.prototype.__proto__ === Object.prototype); // true
```

체인 상의 프로토타입에는 중복 메서드가 있을 수 있음

`Array.prototype`에는 요소 사이에 쉼표를 넣어 요소 전체를 합친 문자열을 반환하는 자체 메서드 `toString`이 있음

- 그런데 `Object.prototype`에도 메서드 `toString`이 있음
- 중복 메서드가 있을 때는 체인 상에서 가까운 곳에 있는 메서드가 사용됨

```javascript
let arr = [1, 2, 3];
alert(arr); // 1,2,3. Array.prototype.toString의 결과
```

```
Object.prototype
[ toString: function ]
[ ...                ]
           ^
           | [[Prototype]]
Array.prototype
[ toString: function ]
[ ...                ]
     ^
     | [[Prototype]]
[ 1, 2, 3 ]
```

`console.dir`를 사용해 내장 객체의 상속 관계를 확인할 수 있음

배열이 아닌 다른 내장 객체들 또한 같은 방법으로 동작

- 함수는 내장 객체 `Function`의 생성자를 사용해 만들어짐
- `call`, `apply`를 비롯한 함수에서 사용할 수 있는 메서드는 `Function.prototype`에서 받아옴
- 함수에도 `toString`이 구현되어 있음

```javascript
function f() {}
alert(f.__proto__ == Function.prototype); // true
alert(f.__proto__.__proto__ == Object.prototype); // true
```

### 원시값

문자열, 숫자, 불린값은 객체가 아님

그런데 이런 원시 타입 값의 프로퍼티에 접근하려고 하면 내장 생성자 `String`, `Number`, `Boolean`을 사용하는 임시 래퍼(wrapper) 객체가 생성됨

- 임시 래퍼 객체는 이런 메서드를 제공한 후 사라짐

래퍼 객체는 보이지 않는 곳에서 만들어지는데 엔진에 의해 최적화됨

그런데 명세서에는 각 자료형에 해당하는 래퍼 객체의 메서드를 프로토타입 안에 구현해놓고 `String.prototype`, `Number.prototype`, `Boolean.prototype`을 사용해 쓰도록 규정함

`null`과 `undefined`에 대응하는 래퍼 객체는 없음

- 메서드와 프로퍼티를 이용할 수 없음
- 프로토타입도 사용할 수 없음

### 네이티브 프로토타입 변경하기

네이티브 프로토타입은 수정할 수 있음

```javascript
String.prototype.show = function () {
  alert(this);
};
"BOOM!".show(); // BOOM!
// 모든 문자열에서 해당 메서드에 접근 가능
```

네이티브 프로토타입에 새 내장 메서드를 추가하는 것은 좋지 않음

- 프로토타입은 전역으로 영향을 미치기 때문에 프로토타입을 조작하면 기존 코드와 충돌할 가능성이 큼

모던 프로그래밍에서 네이티브 프로토타입 변경을 허용하는 경우는 딱 하나, 폴리필을 만들 때

폴리필

- 자바스크립트 명세서에 있는 메서드와 동일한 기능을 하는 메서드 구현체
- 명세서에는 정의되어 있으나 특정 자바스크립트 엔진에서는 해당 기능이 구현되어있지 않을 때 폴리필을 사용
- 폴리필을 직접 구현한 후, 폴리필을 내장 프로토타입에 추가할 때만 네이티브 프로토타입을 변경할 것

```javascript
// repeat이라는 메서드가 없다고 가정
if (!String.prototype.repeat) {
  String.prototype.repeat = function (n) {
    return new Array(n + 1).join(this);
  };
}
alert("라".repeat(3)); // 라라라
```

### 프로토타입에서 메서드 빌려오기

한 객체의 메서드를 다른 객체로 복사할 때 이 기법이 사용됨

```javascript
let obj = { 0: "Hello", 1: "world!", length: 2 }; // 유사 배열 객체
obj.join = Array.prototype.join; // Array 메서드 복사
alert(obj.join(",")); // Hello,world!
```

- 내장 메서드 `join`의 내부 알고리즘은 제대로 된 인덱스가 있는지와 `length` 프로퍼티가 있는지만 확인하기 때문에 예시는 에러 없이 의도한 대로 동작
- 호출 대상이 진짜 배열인지는 상관 없음
- 다수의 내장 메서드가 이런 식으로 동작

메서드 빌리기 말고 `obj.__proto__`를 `Array.prototype`으로 설정해 배열 메서드를 상속받게 해 `obj`에서 모든 `Array` 메서드를 사용할 수 있음

```javascript
let obj = { 0: "Hello", 1: "world!", length: 2 }; // 유사 배열 객체
obj.__proto__ = Array.prototype; // Array 메서드 복사
alert(obj.join(",")); // Hello,world!
```

그런데 자바스크립트는 단일 상속만 허용하기 때문에 이 방법은 `obj`가 다른 객체를 상속받고 있을 때는 사용할 수 없음

메서드 빌리기는 여러 객체에서 필요한 기능을 가져와 섞는 것을 가능하게 해주기 때문에 유연한 개발을 가능하게 해줌

### 예시

메서드 `f.defer(ms)`를 함수에 추가하기

```javascript
function f() {
  alert("Hello!");
}

Function.prototype.defer = function (ms) {
  setTimeout(this, ms);
  // setTimeout(() => this(), ms); 이것도 잘 출력됨
};

f.defer(1000); // 1초 후 "Hello!" 출력
```

데코레이팅 `defer()`를 함수에 추가하기

```javascript
function f(a, b) {
  alert(a, b);
}

Function.prototype.defer = function (ms) {
  let f = this;
  return function (...args) {
    setTimeout(() => f.apply(this, args), ms);
  };
};

f.defer(1000)(1, 2); // 1초 후 3을 출력
```

- 객체 메서드에 대한 데코레이션 동작을 만들기 위해서 `this`를 `f.apply` 안에서 사용했음
- 그래서 래퍼 함수가 객체 메서드로써 호출된다면 `this`는 기존 메서드 `f`에 전달됨

```javascript
Function.prototype.defer = function (ms) {
  let f = this;
  return function (...args) {
    setTimeout(() => f.apply(this, args), ms);
  };
};

let user = {
  name: "John",
  sayHi() {
    alert(this.name);
  }
};

user.sayHi = user.sayHi.defer(1000);
user.sayHi();
```

## 프로토타입 메서드와 `__proto__`가 없는 객체

## 참고

> [프로토타입과 프로토타입 상속](https://ko.javascript.info/prototypes)
