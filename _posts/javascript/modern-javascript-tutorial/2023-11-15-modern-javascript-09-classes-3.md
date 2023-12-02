---
title: 모던 JavaScript 튜토리얼 09 - 클래스 3
date: 2023-11-15 10:03:12 +0900
last_modified_at: 2023-12-02 13:10:56 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

내장 클래스 확장하기, 'instanceof'로 클래스 확인하기, 믹스인

## 내장 클래스 확장하기

배열, 맵 같은 내장 클래스도 확장 가능

```javascript
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
}
let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false
let filteredArr = arr.filter((item) => item >= 10);
alert(filteredArr); // 10, 50
alert(filteredArr.isEmpty()); // false
```

- `PowerArray`는 기본 `Array`를 상속 받음
- `filter`, `map` 등의 내장 메서드가 상속 받은 클래스인 `PowerArray`의 인스턴스(객체)를 반환
  - 이 객체를 구현할 때는 내부에서 객체의 `constructor` 프로퍼티를 사용
  - 따라서 아래와 같은 관계를 가짐

```javascript
arr.constructor === PowerArray;
```

- `arr.filter()`가 호출될 때, 내부에서는 기본 `Array`가 아닌 `arr.constructor`를 기반으로 새로운 배열이 만들어지고, 여기에 필터 후 결과가 담김
  - `PowerArray`에 구현된 메서드를 사용할 수 있는 장점이 있음
- 동작 방식을 변경할 수도 있음
  - 특수 정적 getter인 `Symbol.species`를 클래스에 추가할 수 있음
  - `Symbol.species`가 있으면 `map`, `filter` 등의 메서드를 호출할 때 만들어지는 개체의 생성자를 지정할 수 있음
  - 원하는 생성자를 반환하기만 하면 됨
- `map`이나 `filter` 같은 내장 메서드가 일반 배열을 반환하도록 하려면 `Symbol.species`가 `Array`를 반환하도록 해주면 됨

```javascript
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
  // 내장 메서드는 반환 값에 명시된 클래스를 생성자로 사용
  static get [Symbol.species]() {
    return Array;
  }
}
let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false
// filter는 arr.constructor[Symbol.species]를 생성자로 사용해 새로운 배열 생성
let filteredArr = arr.filter((item) => item >= 10);
// filteredArr는 PowerArray가 아닌 Array의 인스턴스
alert(filteredArr.isEmpty()); // TypeError: filteredArr.isEmpty is not a function
```

`Map`, `Set` 같은 컬렉션도 위와 같이 동작

- `Symbol.species`를 사용

### 내장 객체와 정적 메서드 상속

내장 객체는 `Object.keys`, `Array.isArray` 등의 자체 정적 메서드를 가짐

네이티브 클래스들은 서로 상속 관계를 맺음

- `Array`는 `Object`를 상속받음

일반적으로는 한 클래스가 다른 클래스를 상속받으면 정적 메서드와 그렇지 않은 메서드 모두를 상속받음

그러나 내장 클래스는 다름

- 내장 클래스는 정적 메서드를 상속받지 못함

`Array`와 `Date`는 모두 `Object`를 상속받기 때문에 두 클래스의 인스턴스에서는 `Object.prototype`에 구현된 메서드를 사용할 수 있음

그런데 `Array.[[Prototype]]`과 `Date.[[Prototype]]`은 `Object`를 참조하지 않기 때문에 `Array.keys()`나 `Date.keys()` 같은 정적 메서드를 인스턴스에서 사용할 수 없음

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
                                          ^
                                          | [[Prototype]]
                                new Date()
                                [ 1 Jan 2019 ]
```

- `Date`와 `Object`를 직접 이어주는 링크가 없고 서로 독립적
- `Date.prototype`만 `Object.prototype`을 상속받음

내장 객체 간의 상속과 `extends`를 사용한 상속의 가장 큰 차이점이 여기에 있음

## 'instanceof'로 클래스 확인하기

`instanceof` 연산자를 사용해 객체가 특정 클래스에 속하는지 아닌지 확인 가능

- 상속 관계도 확인해줌
- 인수의 타입에 따라 이를 다르게 처리하는 다형적인(polymorphic) 함수를 만드는 데 사용하기도 함

### instanceof 연산자

```javascript
obj instanceof Class;
```

- obj가 `Class`에 속하거나 `Class`를 상속받는 클래스에 속하면 `true` 반환

```javascript
class Rabbit {}
let rabbit = new Rabbit();
alert(rabbit instanceof Rabbit); // true
```

생성자 함수에서도 사용 가능

```javascript
function Rabbit() {}
alert(new Rabbit() instanceof Rabbit); // true
```

내장 클래스에도 사용 가능

```javascript
let arr = [1, 2, 3];
alert(arr instanceof Array); // true
alert(arr instanceof Object); // true
```

- `Array`는 프로토타입 기반으로 `Object`를 상속받음

`instanceof` 연산자는 보통 프로토타입 체인을 거슬러 올라가며 인스턴스 여부나 상속 여부를 확인

정적 메서드 `Symbol.hasInstance`를 사용하면 직접 확인 로직을 설정할 수도 있음

`obj instanceof Class`의 대략의 동작 알고리즘

1. 클래스에 정적 메서드 `Symbol.hasInstance`가 구현되어 있으면, `obj instanceof Class` 문이 실행될 때, `Class[Symbol.hasInstance](obj)`가 호출됨
   - 호출 결과는 `true`나 `false`
   - 이런 규칙을 기반으로 `instanceof`의 동작을 커스터마이징할 수 있음

```javascript
class Animal {
  static [Symbol.hasInstance](obj) {
    if (obj.canEat) return true;
  }
}
let obj = { canEat: true };
alert(obj instanceof Animal); // true
```

2. 그런데 대부분의 클래스에는 `Symbol.hasInstance`가 구현되어 있지 않음
   - 이럴 때는 일반적인 로직이 사용됨
   - `obj instanceof Class`는 `Class.prototype`이 `obj` 프로토타입 체인 상의 프로토타입 중 하나와 일치하는지 확인함
   - 비교는 차례 차례 진행됨

```
obj.__proto__ === Class.prototype?
obj.__proto__.__proto__ === Class.prototype?
obj.__proto__.__proto__.__proto__ === Class.prototype?
...
이 중 하나라도 true라면 true 반환
그렇지 않고 체인 끝에 도달하면 false 반환
```

- 위 예시에서 `rabbit.__proto__ === Rabbit.prototype`가 `true`이므로 `instanceof`는 `true`를 반환
- 상속받은 클래스를 사용하는 경우에는 두 번째 단계에서 일치 여부가 확인됨

```javascript
class Animal {}
class Rabbit extends Animal {}
let rabbit = new Rabbit();
alert(rabbit instanceof Animal); // true
// rabbit.__proto__ === Rabbit.prototype
// rabbit.__proto__.__proto__ === Animal.prototype
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

- `rabbit instanceof Animal`을 실행했을 때 `Animal.prototype`과 비교되는 대상들

`objA.isPrototypeOf(objB)`

- `objA`가 `objB`의 프로토타입 체인 상 어딘가에 있으면 `true`를 반환하는 메서드
- `obj instanceof Class`는 `Class.prototype.isPrototypeOf(obj)`와 동일

`isPrototypeOf`는 `Class` 생성자를 제외하고 포함 여부를 검사하는 점이 특이

- 검사 시, 프로토타입 체인과 `Class.prototype`만 고려
- 이런 특징은 객체 생성 후 `prototype` 프로퍼티가 변경되는 경우 특이한 결과를 초래하기도 함

```javascript
function Rabbit() {}
let rabbit = new Rabbit();
Rabbit.prototype = {}; // 프로토타입이 변경됨
alert(rabbit instanceof Rabbit); // false (더 이상 Rabbit이 아님)
```

### 보너스: 타입 확인을 위한 Object.prototype.toString

일반 객체를 문자열로 변화하면 `[object Object]`가 됨

```javascript
let obj = {};
alert(obj); // [object Object]
alert(obj.toString()); // [object Object]
```

이렇게 `[object Object]`가 되는 이유는 `toString`의 구현 방식 때문

그런데 `toString`에는 `toString`을 더 강략하게 만드는 기능이 숨겨져 있음

`toString`의 숨겨진 기능을 사용하면 확장 `typeof`, `instanceof`의 대안을 만들 수 있음

명세서에 따르면 객체에서 내장 `toString`을 추출하는 것이 가능

이렇게 추출한 메서드는 모든 값을 대상으로 실행할 수 있음

호출 결과는 값에 따라 달라짐

- 숫자형: `[object Number]`
- 불린형: `[object Boolean]`
- null: `[object Null]`
- undefined: `[object Undefined]`
- 배열: `[object Array]`
- 그외: 커스터마이징 가능

```javascript
let objectToString = Object.prototype.toString;
let arr = [];
alert(objectToString.call(arr)); // [object Array]
```

- `toString` 알고리즘은 내부적으로 `this`를 검사하고 상응하는 결과를 반환

```javascript
let s = Object.prototype.toString;
alert(s.call(123)); // [object Number]
alert(s.call(null)); // [object Null]
alert(s.call(alert)); // [object Function]
```

Symbol.toStringTag

- 특수 객체 프로퍼티 `Symbol.toStringTag`를 사용하면 `toString`의 동작을 커스터마이징할 수 있음

```javascript
let user = {
  [Symbol.toStringTag]: "User"
};
alert({}.toString.call(user)); // [object User]
```

대부분의 호스트 환경은 자체 객체에 이와 유사한 프로퍼티를 구현함

```javascript
alert(window[Symbol.toStringTag]); // Window
alert(XMLHttpRequest.prototype[Symbol.toStringTag]); // XMLHttpRequest
alert({}.toString.call(window)); // [object Window]
alert({}.toString.call(new XMLHttpRequest())); // [object XMLHttpRequest]
```

- 호스트 환경 고유 객체의 `Symbol.toStringTag` 값은 `[object ...]`로 싸여진 값과 동일
- `typeof` 연산자의 강력한 변형들(`toString`, `toStringTag`)은 원시 자료형 뿐만 아니라 내장 객체에서도 사용 가능
- 커스터마이징도 가능
- 내장 객체의 타입 확인을 넘어서 타입을 문자열 형태로 받고 싶다면 `instanceof` 대신 `{}.toString.call`을 사용할 수 있음

|               |                      동작 대상                      |    반환값    |
| :-----------: | :-------------------------------------------------: | :----------: |
|   `typeof`    |                       원시형                        |    문자열    |
| `{}.toString` | 원시형, 내장 객체, `Symbol.toStringTag`가 있는 객체 |    문자열    |
| `instanceof`  |                        객체                         | true나 false |

### 예제

이상한 instanceof

- `a`는 `B()`를 통해 생성하지 않았는데 `instanceof`가 `true`를 반환하는 이유

```javascript
function A() {}
function B() {}
A.prototype = B.prototype = {};
let a = new A();
alert(a instanceof B); // true
```

- `instanceof`는 평가 시, 함수는 고려하지 않고 평가 대상의 `prototype`을 고려
- 평가 대상의 `prototype`이 프로토타입 체인 상에 있는 프로토타입과 일치하는지 여부를 고려
- `a.__proto__ == B.__proto__`이므로 `instanceof`는 `true`를 반환
- `instanceof`의 내부 알고리즘에 의해 `prototype`은 생성자 함수가 아닌 타입을 정의함

## 믹스인

## 참고

> [클래스](https://ko.javascript.info/classes)
