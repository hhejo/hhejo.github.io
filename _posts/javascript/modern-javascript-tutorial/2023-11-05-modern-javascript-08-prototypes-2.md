---
title: 모던 JavaScript 튜토리얼 08 - 프로토타입과 프로토타입 상속 2
date: 2023-11-05 11:54:16 +0900
last_modified_at: 2023-11-16 10:54:14 +0900
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

`__proto__`는 브라우저를 대상으로 개발하고 있다면 다소 구식이기 때문에 더는 사용하지 않는 것이 좋음

- 표준에도 관련 내용이 명시됨

아래와 같은 모던한 메서드들 사용 권장

`Object.create(proto, [descriptors])`

- `[[Prototype]]`이 `proto`를 참조하는 빈 객체를 생성

`Object.getPrototypeOf(obj)`

- `obj`의 `[[Prototype]]`을 반환

`Object.setPrototypeOf(obj, proto)`

- `obj`의 `[[Prototype]]`이 `proto`가 되도록 설정

```javascript
let animal = { eats: true };
let rabbit = Object.create(animal);
alert(rabbit.eats); // true
alert(Object.getPrototypeOf(rabbit) === animal); // true
Object.setPrototypeOf(rabbit, {}); // rabbit의 프로토타입을 {}으로 바꿈
```

```javascript
let animal = { eats: true };
let rabbit = Object.create(animal, { jumps: { value: true } });
alert(rabbit.jumps); // true
```

`Object.create`를 사용하면 `for..in`을 사용해 프로퍼티를 복사하는 것보다 더 효과적으로 객체 복제 가능

```javascript
let clone = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

- 열거 가능한 프로퍼티
- 열거 불가능한 프로퍼티
- 데이터 프로퍼티
- getter, setter
- 등등 모든 프로퍼티가 복제됨
- `[[Prototype]]`도 복제됨

### 비하인드 스토리

`[[Prototype]]`을 다루는 방법은 다양함

역사적인 이유가 있음

- 생성자 함수의 `prototype` 프로퍼티는 아주 오래전부터 사용됨
- 2012년 명세서에 `Object.create` 추가됨
  - `Object.create`를 사용하면 주어진 프로토타입으로 객체 생성 가능
  - 프로토타입을 얻거나 설정하는 것은 불가능
  - 그래서 브라우저는 비표준 접근자인 `__proto__`를 구현해 언제나 프로토타입을 얻거나 설정할 수 있게 함
- 이후 2015년에 `Object.setPrototypeOf`와 `Object.getPrototypeOf`가 표준에 추가되면서 `__proto__`와 동일한 기능을 수행할 수 있게 됨
  - 그런데 이 시점에는 `__proto__`를 사용하는 곳이 너무 많아서 `__proto__`는 사실상 표준(de-facto standard)이 됨
  - 이 내용은 명세서의 부록 B(Annex B)에 추가되어 있음
  - 부록 B의 내용은 브라우저 이외의 호스트 환경에서는 선택사항이라는 것을 의미
- 이런 역사적인 이유 때문에 지금은 여러 방식을 원하는 대로 쓸 수 있게 됨

속도가 중요하다면 기존 객체의 `[[Prototype]]`을 변경하지 마라

- 원한다면 언제나 `[[Prototype]]`을 얻거나 수정할 수 있음
- 하지만 대개는 객체를 생성할 때만 `[[Prototype]]`을 설정하고 이후에는 수정하지 않음
  - `rabbit`이 `animal`을 상속받도록 설정한 후에는 상속 관계를 잘 변경하지 않음
- 자바스크립트 엔진은 이런 시나리오를 토대로 최적화됨
- `Object.setPrototypeOf`나 `obj.__proto__=`를 써서 프로토타입을 그때그때 바꾸는 연산은 객체 프로퍼티 접근 관련 최적화를 망치기 때문에 성능에 나쁜 영향을 미침
- `[[Prototype]]`을 바꾸는 것이 어떤 결과를 초래할지 확실히 알거나 속도가 전혀 중요하지 않은 경우가 아니라면 `[[Prototype]]`을 바꾸지 않는 것을 권장

### 아주 단순한 객체

`__proto__`가 나쁜 이유

객체는 키-값 쌍이 있는 연관 배열로도 사용할 수 있음

- 커스텀 사전을 만드는 것과 같이 사용자가 직접 입력한 키를 가지고 객체를 만들다 보면 사소한 결함이 발견됨
- 다른 문자열은 괜찮지만 `"__proto__"`라는 문자열은 문자열 키로 사용할 수 없음

```javascript
let obj = {};
let key = prompt("입력하고자 하는 key", "__proto__");
obj[key] = "...값...";
alert(obj[key]); // "...값..."이 아닌 [object Object]가 출력됨
```

- 프롬프트 창에 `__proto__`를 입력하면 값이 제대로 할당되지 않음
- `__proto__` 프로퍼티는 특별한 프로퍼티이기 때문에 항상 객체이거나 `null`이어야 함
- 키가 무엇이 되었든 키-값 쌍을 저장하려고 하는데 키가 `__proto__`일 때 값이 제대로 저장되지 않는 것은 버그
- 할당 값이 객체일 때는 프로토타입이 바뀔 수 있다는 치명적인 버그가 발생할 수 있음
  - 프로토타입이 바뀌면 예상치 못한 일이 발생할 수 있음
  - 대개 프로토타입이 중간에 바뀌는 시나리오는 배제하고 개발을 진행
  - 이런 고정관념 때문에 프로토타입이 중간에 바뀌면서 발생한 버그는 그 원인을 쉽게 찾지 못함
  - 서버 사이드에서 자바스크립트를 사용할 때는 이런 버그가 취약점이 되기도 함

객체 대신 맵을 사용해 예방할 수 있음

- 그런데 자바스크립트를 만든 사람들이 아주 오래 전부터 이런 문제를 고려했기 때문에 객체를 써도 문제를 예방할 수 있음

```
Object               Object.prototype
[     ] -prototype-> [ ...                     ]
                     [ get __proto__: function ]
                     [ set __proto__: function ]
                                  ^
                                  | [[Prototype]]
                      obj         |
                     [                         ]
```

- `__proto__`는 객체의 프로퍼티가 아니가 `Object.prototype`의 접근자 프로퍼티
- `obj.__proto__`를 읽거나 쓸 때는 이에 대응하는 getter, setter가 프로토타입에서 호출되고 `obj`는 `[[Prototype]]`을 통해 getter와 setter에 겁근
- `__proto__`는 `[[Prototype]]`에 접근하기 위한 수단이지 `[[Prototype]]` 그 자체가 아님

간단한 트릭을 써 객체가 연관 배열의 역할을 다 할 수 있게 하기

```javascript
let obj = Object.create(null);
let key = prompt("입력하고자 하는 key", "__proto__");
obj[key] = "...값...";
alert(obj[key]); // "...값..."이 제대로 출력됨
```

- `Object.create(null)`로 프로토타입이 없는 빈 객체 생성(`[[Prototype]]`이 `null`인 객체)

```
   null
     ^
     | [[Prototype]]
obj  |
[         ]
```

`Object.create(null)`로 객체를 만들면 `__proto__` getter와 setter를 상속받지 않음

- `__proto__`는 평범한 데이터 프로퍼티처럼 처리됨

프로토타입이 없는 빈 객체

- 아주 단순한(very plain) 객체
- 순수 사전식(pure dictionary) 객체
- 일반 객체 `{...}` 보다 훨씬 단순함
- 내장 메서드가 없다는 단점이 있음
  - 객체를 연관 배열로 쓸 때는 이런 단점이 문제가 되지 않음
  - 객체 관련 메서드 대부분은 `Object.keys(obj)` 같이 `Object.something(...)` 형태
  - 이 메서드들은 프로토타입에 있는 게 아니기 때문에 아주 단순한 객체에도 사용 가능

```javascript
let obj = Object.create(null);
alert(obj); // TypeError: Cannot convert object to primitive value
```

```javascript
let chineseDictionary = Object.create(null);
chineseDictionary.hello = "你好";
chineseDictionary.bye = "再见";
alert(Object.keys(chineseDictionary)); // hello,bye
```

`Object.keys(obj)`, `Object.values(obj)`, `Object.entries(obj)`

- `obj` 내 열거 가능한 프로퍼티 키, 값, 키-값 쌍을 담은 배열 반환

`Object.getOwnPropertySymbols(obj)`

- `obj` 내 심볼형 키를 담은 배열 반환

`Object.getOwnPropertyNames(obj)`

- `obj` 내 문자형 키를 담은 배열 반환

`Reflect.ownKeys(obj)`

- `obj` 내 키 전체를 담은 배열 반환

`obj.hasOwnProperty(key)`

- 상속받지 않고 `obj` 자체에 구현된 키 중 이름이 `key`인 것이 있으면 `true` 반환

객체의 프로퍼티를 반환하는 메서드들은 객체가 직접 소유한 프로퍼티만 반환

상속 프로퍼티는 `for..in`을 사용해 얻을 수 있음

### 예시

사전에 toString 추가하기

- 쉼표로 구분된 키 목록을 반환하는 `dictionary.toString()` 메서드 추가
- `toString`은 객체 위의 `for..in`에 나타나지 않게

```javascript
let dictionary = Object.create(null, {
  toString: {
    value() {
      return Object.keys(this).join();
    }
  }
});

dictionary.apple = "Apple";
dictionary.__proto__ = "test";

for (let key in dictionary) alert(key);
alert(dictionary.toString());
```

호출 간의 차이점

```javascript
function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype.sayHi = function () {
  alert(this.name);
};

let rabbit = new Rabbit("Rabbit");

rabbit.sayHi(); // Rabbit
Rabbit.prototype.sayHi(); // undefined
Object.getPrototypeOf(rabbit).sayHi(); // undefined
rabbit.__proto__.sayHi(); // undefined
```

## 참고

> [프로토타입과 프로토타입 상속](https://ko.javascript.info/prototypes)
