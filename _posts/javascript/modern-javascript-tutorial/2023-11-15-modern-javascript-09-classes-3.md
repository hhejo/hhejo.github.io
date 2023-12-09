---
title: 모던 JavaScript 튜토리얼 09 - 클래스 3
date: 2023-11-15 10:03:12 +0900
last_modified_at: 2023-12-09 10:59:32 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

내장 클래스 확장하기, 'instanceof'로 클래스 확인하기, 믹스인

## 내장 클래스 확장하기

내장 클래스도 확장 가능

- 배열, 맵 등

`Array`를 상속받는 `PowerArray`

- 내장 메서드(`map`, `filter` 등)가 상속받은 클래스인 `PowerArray`의 인스턴스(객체)를 반환
- `PowerArray`에 구현된 메서드를 사용할 수 있는 장점이 있음
- 이 객체를 구현할 때는 내부에서 객체의 `constructor` 프로퍼티를 사용
- 따라서 `arr.constructor === PowerArray`의 관계를 가짐
- `arr.filter` 호출 시, 내부에서 `arr.constructor`를 기반으로 새로운 배열이 생성되고 filter 결과가 담김
  - 기본 `Array`에 기반하지 않음

`Symbol.species`

- 특수 정적 getter
- 클래스에 추가해 동작 방식 변경 가능
- 메서드(`map`, `filter` 등)를 호출할 때 만들어지는 개체의 생성자 지정
  - 원하는 생성자를 반환하면 됨
- `Symbol.species`가 `Array`를 반환하면 내장 메서드(`map`, `filter` 등)가 일반 배열을 반환
- `Map`, `Set` 같은 컬렉션도 위와 같이 동작
  - `Symbol.species`를 사용

```javascript
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
}
let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false
let filtered = arr.filter((item) => item >= 10); // [ 10, 50 ]
alert(filtered.isEmpty()); // false

class PowerArray2 extends Array {
  isEmpty() {
    return this.length === 0;
  }
  static get [Symbol.species]() {
    return Array; // 내장 메서드는 반환값에 명시된 클래스를 생성자로 사용
  }
}
let arr2 = new PowerArray2(1, 2, 5, 10, 50);
alert(arr2.isEmpty()); // false
let filtered2 = arr2.filter((item) => item >= 10); // filter는 arr2.constructor[Symbol.species]를 생성자로 사용해 새로운 배열 생성
alert(filtered2.isEmpty()); // TypeError: filtered2.isEmpty is not a function
```

### 내장 객체와 정적 메서드 상속

내장 클래스와 정적 메서드 상속

- 내장 객체는 자체 정적 메서드를 가짐
  - `Object.keys`, `Array.isArray` 등
- 네이티브 클래스들은 서로 상속 관계를 맺음
- 한 클래스가 다른 클래스를 상속받는 경우
  - 일반적으로는 정적 메서드와 그렇지 않은 메서드 모두를 상속받음
- 그러나 내장 클래스는 정적 메서드를 상속받지 못함
- `Array`와 `Date`는 모두 `Object`를 상속받음
- 두 클래스의 인스턴스에서 `Object.prototype`에 구현된 메서드 사용 가능
- 그런데 `Array.[[Prototype]]`과 `Date.[[Prototype]]`은 `Object`를 참조하지 않음
- 그래서 인스턴스에서 정적 메서드(`Array.keys`, `Date.keys` 등) 사용 불가
- 내장 객체 간의 상속과 `extends`를 사용한 상속의 가장 큰 차이점이 여기에 있음

```
Object                          Object.prototype
[ defineProperty ] -prototype-> [ constructor: Object      ]
[ keys           ]              [ toString: function       ]
[ ...            ]              [ hasOwnProperty: function ]
                                             ^
                                             | [[Prototype]]
Date                            Date.prototype
[ now   ]     -prototype->      [ constructor: Date  ]
[ parse ]                       [ toString: function ]
[ ...   ]                       [ getDate: function  ]
Date와 Object는 서로 독립적               ^
                                          | [[Prototype]]
                                new Date()
                                [ 1 Jan 2019 ]
```

## 'instanceof'로 클래스 확인하기

### instanceof 연산자

`instanceof` 연산자

```javascript
obj instanceof Class;
```

- obj가 `Class`에 속하거나 `Class`를 상속받는 클래스에 속하면 `true` 반환
- 상속 관계도 확인 가능
- 인수의 타입에 따라 이를 다르게 처리하는 다형적인(polymorphic) 함수를 만드는 데 사용하기도 함
- 생성자 함수, 내장 클래스에도 사용 가능
- `instanceof` 연산자는 프로토타입 체인을 거슬러 올라가며 인스턴스 여부나 상속 여부를 확인
- 정적 메서드 `Symbol.hasInstance`를 사용해 직접 확인 로직을 설정 가능

```javascript
class Rabbit {}
let rabbit = new Rabbit();
alert(rabbit instanceof Rabbit); // true

function Rabbit2() {}
alert(new Rabbit2() instanceof Rabbit2); // true

let arr = [1, 2, 3]; // Array는 프로토타입 기반으로 Object를 상속받음
alert(arr instanceof Array); // true
alert(arr instanceof Object); // true
```

`obj instanceof Class`의 대략의 동작 알고리즘

1. 클래스에 정적 메서드 `Symbol.hasInstance`가 구현된 경우
   - `obj instanceof Class`가 실행될 때 `Class[Symbol.hasInstance](obj)` 호출되고 `true`나 `false` 반환
   - `instanceof`의 동작 커스터마이징 가능
1. 클래스에 `Symbol.hasInstance`가 구현되지 않은 경우
   - 대부분의 클래스에는 구현되어 있지 않아 일반적인 로직이 사용됨
   - `obj instanceof Class`는 `Class.prototype`이 `obj` 프로토타입 체인 상의 프로토타입 중 하나와 일치하는지 확인

```javascript
class Animal {
  static [Symbol.hasInstance](obj) {
    if (obj.canEat) return true;
  }
}
let obj = { canEat: true };
alert(obj instanceof Animal); // true
```

```
obj.__proto__ === Class.prototype?
obj.__proto__.__proto__ === Class.prototype?
obj.__proto__.__proto__.__proto__ === Class.prototype?
...
이 중 하나라도 true라면 true 반환
그렇지 않고 체인 끝에 도달하면 false 반환
```

```javascript
class Animal {}
class Rabbit extends Animal {}
let rabbit = new Rabbit();
alert(rabbit instanceof Animal); // true
// rabbit instanceof Animal을 실행했을 때 Animal.prototype과 비교되는 대상들
alert(rabbit.__proto__ === Rabbit.prototype); // true
alert(rabbit.__proto__.__proto__ === Animal.prototype); // true
```

```
      null
        ^
        | [[Prototype]]
Object.prototype
[              ]
        ^
        | [[Prototype]]
Animal.prototype
[              ]
        ^
        | [[Prototype]]
Rabbit.prototype
[              ]
        ^
        | [[Prototype]]
rabbit  |
[              ]
```

`objA.isPrototypeOf(objB)`

- `objA`가 `objB`의 프로토타입 체인 상 어딘가에 있으면 `true` 반환
- `obj instanceof Class`는 `Class.prototype.isPrototypeOf(obj)`와 동일
- `isPrototypeOf`는 `Class` 생성자를 제외하고 포함 여부를 검사하는 점이 특이함
- 검사 시 프로토타입 체인과 `Class.prototype`만 고려
- 이런 특징은 객체 생성 후 `prototype` 프로퍼티가 변경되는 경우 특이한 결과를 초래

```javascript
function Rabbit() {}
let rabbit = new Rabbit();
Rabbit.prototype = {}; // 프로토타입이 변경됨
alert(rabbit instanceof Rabbit); // false (더 이상 Rabbit이 아님)
```

### 보너스: 타입 확인을 위한 Object.prototype.toString

일반 객체를 문자열로 변화하면 `[object Object]`가 됨

- `toString`의 구현 방식 때문에 `[object Object]`가 됨
- `toString`의 숨겨진 기능을 사용하면 확장 `typeof`, `instanceof`의 대안을 만들 수 있음
- 명세서에 따르면 객체에서 내장 `toString`을 추출해 모든 값을 대상으로 실행 가능
- 호출 결과는 값에 따라 달라짐
  - 숫자형: `[object Number]`
  - 불린형: `[object Boolean]`
  - null: `[object Null]`
  - undefined: `[object Undefined]`
  - 배열: `[object Array]`
  - 그외: 커스터마이징 가능
- `toString` 알고리즘은 내부적으로 `this`를 검사하고 상응하는 결과를 반환

```javascript
let obj = {};
alert(obj); // [object Object]
alert(obj.toString()); // [object Object]

let objectToString = Object.prototype.toString;
let arr = [];
alert(objectToString.call(arr)); // [object Array]

let s = Object.prototype.toString;
alert(s.call(123)); // [object Number]
alert(s.call(null)); // [object Null]
alert(s.call(alert)); // [object Function]
```

`Symbol.toStringTag`

- 특수 객체 프로퍼티
- `toString`의 동작 커스터마이징 가능
- 대부분의 호스트 환경은 자체 객체에 이와 유사한 프로퍼티를 구현
- 호스트 환경 고유 객체의 `Symbol.toStringTag` 값은 `[object ...]`로 싸여진 값과 동일
- `typeof` 연산자의 강력한 변형들(`toString`, `toStringTag`)은 원시 자료형 뿐만 아니라 내장 객체에서도 사용 가능
- 커스터마이징도 가능
- 내장 객체의 타입 확인을 넘어서 타입을 문자열 형태로 받고 싶다면 `instanceof` 대신 `{}.toString.call`을 사용

```javascript
let user = {
  [Symbol.toStringTag]: "User"
};
alert({}.toString.call(user)); // [object User]

alert(window[Symbol.toStringTag]); // Window
alert(XMLHttpRequest.prototype[Symbol.toStringTag]); // XMLHttpRequest
alert({}.toString.call(window)); // [object Window]
alert({}.toString.call(new XMLHttpRequest())); // [object XMLHttpRequest]
```

|               |                      동작 대상                      |    반환값    |
| :-----------: | :-------------------------------------------------: | :----------: |
|   `typeof`    |                       원시형                        |    문자열    |
| `{}.toString` | 원시형, 내장 객체, `Symbol.toStringTag`가 있는 객체 |    문자열    |
| `instanceof`  |                        객체                         | true나 false |

### 예제

이상한 instanceof

- `a`는 `B()`를 통해 생성하지 않았는데 `instanceof`가 `true`를 반환하는 이유
- `instanceof`는 평가 시, 함수는 고려하지 않고 평가 대상의 `prototype`을 고려
- 평가 대상의 `prototype`이 프로토타입 체인 상에 있는 프로토타입과 일치하는지 여부를 고려
- `a.__proto__ == B.__proto__`이므로 `instanceof`는 `true`를 반환
- `instanceof`의 내부 알고리즘에 의해 `prototype`은 생성자 함수가 아닌 타입을 정의함

```javascript
function A() {}
function B() {}
A.prototype = B.prototype = {};
let a = new A();
alert(a instanceof B); // true
```

## 믹스인

## 참고

> [클래스](https://ko.javascript.info/classes)
