---
title: 모던 JavaScript 튜토리얼 04 - 객체 기본 1
date: 2023-10-02 12:01:57 +0900
last_modified_at: 2023-11-19 08:08:16 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

객체, 참조에 의한 객체 복사, 가비지 컬렉션, 메서드와 this

## 객체(Object)

객체는 몇 가지 특수한 기능을 가진 연관 배열(associative array)

프로퍼티(property)

- `키(key): 값(value)` 쌍으로 구성
- `key`: 프로퍼티 이름. 문자형만 가능
- `value`: 모든 자료형(원시형, 객체 등 데이터 집합이나 복잡한 개체(entity)) 가능
- 객체 안에 프로퍼티 여러 개가 들어갈 수 있음

빈 객체를 생성하는 두 가지 방법

1. 객체 리터럴(object literal) `{}`
2. 객체 생성자 `new Object()`

```javascript
let user = {}; // 객체 리터럴
let user = new Object(); // 객체 생성자
```

순수 객체(plain object)

- 일반 객체는 순수 객체라고 불림

일반 객체 외에도 다양한 객체 존재

- `Array`, `Date`, `Error`, ...

### 리터럴과 프로퍼티

프로퍼티 값 읽기

- 점 표기법(dot notation) `.`
- 대괄호 표기법(square bracket notation) `[]`

프로퍼티 삭제하기

- `delete` 연산자

```javascript
let user = { age: 30, "like birds": true };
user.name = "John"; // 점 표기법으로 프로퍼티 값 읽고 쓰기
alert(user["like birds"]); // 대괄호 표기법으로 프로퍼티 값 읽기
delete user.age; // 프로퍼티 삭제
key = "name";
alert(user[key]); // John
alert(user.key); // undefined, (user["key"]와 같음)
```

상수 객체는 수정될 수 있음

```javascript
const user = { name: "John" }; // user의 값을 고정하지만 내용은 수정 가능
user.name = "Pete";
```

### 계산된 프로퍼티

`{ [key]: value }`

- 객체를 만들 때 객체 리터럴 안의 프로퍼티 키를 대괄호로 둘러 쌈

```javascript
let fruit = prompt("과일 이름", "apple");
let bag = { [fruit]: 5 };
// 아래 두 줄보다 간결함
// let bag = {};
// bag[fruit] = 5;
alert(bag.apple); // fruit에 'apple'이 할당되었다면 5 출력
```

복잡한 표현식 작성 가능

```javascript
let bag = { [fruit + "Computers"]: 5 };
alert(bag.appleComputers); // 5
```

### 단축 프로퍼티

프로퍼티 값 단축 구문(property value shorthand)

- 프로퍼티 값을 기존 변수에서 받아와 사용
- 이런 경우가 많음

```javascript
function makeUser(name, age) {
  return { name, age }; // { name: name, age: age };
}
let user = makeUser("John", 30);
```

### 프로퍼티 이름의 제약 사항

예약어를 키로 사용 가능

- `for`, `let`, `return` 등

```javascript
let obj = { for: 1, let: 2, return: 3 };
alert(obj.for + obj.let + obj.return); // 6
```

문자형, 심볼형이 아닌 값은 문자열로 자동 형 변환

```javascript
let obj = { 0: "test" }; // 0은 "0"으로 자동 형 변환됨
alert(obj["0"]); // test
alert(obj[0]); // test (동일한 프로퍼티)
```

객체 프로퍼티 키의 `__proto__`는 특별 취급

```javascript
let obj = {};
obj.__proto__ = 5;
alert(obj.__proto__); // [object Object] (값이 변하지 않았음)
```

### 'in' 연산자로 프로퍼티 존재 여부 확인하기

존재하지 않는 프로퍼티에 접근하기

- 에러가 발생하지 않고 `undefined` 반환

```javascript
"use strict";
let user = {};
user.name; // undefined
```

`===`

- 키가 존재하는지, 존재하지 않는지 `===` 비교로 알 수 없음

```javascript
let user = {};
alert(user.noSuchProperty === undefined); // true
```

- 키 `noSuchProperty`가 존재하고 그 값이 `undefined`
- 키 `noSuchProperty`가 존재하지 않음

`"key" in object`

- 키의 존재 여부를 정확히 알 수 있음

```javascript
let obj = { test: undefined };
alert(obj.test); // undefined (1)
alert("test" in obj); // true (2)
```

- (1) 키 `test`가 존재하고 값이 `undefined`인지, 키 `test`가 존재하지 않는지 모름
- (2) 키가 존재하는지 정확히 알 수 있음

### 'for...in' 반복문

```javascript
for (key in object) {...}
```

```javascript
let user = { name: "John", age: 30, isAdmin: true };
for (let key in user) {
  alert(key); // name, age, isAdmin
  alert(user[key]); // John, 30, true
}
```

객체 정렬 방식

- 객체는 특별한 방식으로 정렬됨

정수 프로퍼티

- 변형 없이 정수에서 왔다 갔다 할 수 있는 문자열

```javascript
String(Math.trunc(Number("49"))); // 49, 정수 프로퍼티 o
String(Math.trunc(Number("+49"))); // 49, 정수 프로퍼티 x
String(Math.trunc(Number("1.2"))); // 1, 정수 프로퍼티 x
```

- 키가 정수인 경우
  - 자동 정렬
- 키가 정수가 아닌 경우
  - 작성된 순서대로 프로퍼티 나열

## 참조에 의한 객체 복사

객체는 참조에 의해(by reference) 저장되고 복사됨

- 변수에 객체 자체가 아닌 메모리 상의 주소인 참조가 저장됨

원시값은 값 그대로 저장, 할당되고 복사됨

참조에 의한 비교

- 객체 비교 `obj1 == obj2`(`obj1 === obj2`)
  - 동등 연산자 `==`와 일치 연산자 `===`는 동일하게 동작
  - 피연산자인 두 객체가 동일한 객체이면 `true` 반환
- 대소 비교 `obj1 > obj2`, 원시값과의 비교 `obj == 5`
  - 객체가 원시형으로 변환됨

```javascript
let a = {};
let b = a;
alert(a == b); // true
alert(a === b); // true
let c = {};
let d = {};
alert(a == b); // false
```

### 객체 복사, 병합과 Object.assign

새로운 객체를 생성한 후, 기존 객체의 프로퍼티들을 순회해 원시 수준까지 복사하면 됨

- 자바스크립트는 객체 복사 내장 메서드를 지원하지 않음

```javascript
let user = { name: "John", age: 30 };
let clone = {};
for (let key in user) clone[key] = user[key];
```

`Object.assign`

```javascript
Object.assign(dest, [src1, src2, ...])
```

- `dest`: 목표 객체
- `src1, ..., srcN`: 복사할 객체
- 각 프로퍼티를 복사하고 `dest` 반환
- 동일한 이름을 가진 프로퍼티가 있으면 덮어씀

```javascript
let user = { name: "John", age: 30 };
let clone = Object.assign({}, user); // { name: "John", age: 30 }
user.canView = false; // { name: "John", age: 30, canView: false }
let permissions1 = { canView: true };
let permissions2 = { canEdit: true };
Object.assign(user, permissions1, permissions2);
user; // { name: "John", age: 30, canView: true, canEdit: true }
```

### 중첩 객체 복사

깊은 복사를 해야 하기 때문에 주의

## 가비지 컬렉션

자바스크립트 엔진이 필요 없는 원시값, 객체, 함수 등을 찾아내 삭제하는 방법

### 가비지 컬렉션 기준

도달 가능성(reachability) 개념을 사용해 메모리 관리

- 도달 가능한(어떻게든 접근하거나 사용할 수 있는) 값은 메모리에서 삭제되지 않음

루트(root)

- 태생부터 도달 가능하기 때문에 명백한 이유 없이 삭제되지 않는 값
  1. 현재 함수의 지역변수와 매개변수
  2. 중첩 함수의 체인에 있는 함수에서 사용되는 변수와 매개변수
  3. 전역변수, 기타 등등
- 루트가 참조하는 값이나 체이닝으로 루트에서 참조할 수 있는 값은 도달 가능한 값

### 간단한 예시

```javascript
let user = { name: "John" }; // user에 객체 참조 값 저장
```

```
<global> -user-> Object
                 name: "John"
```

`user`의 값을 다른 값으로 덮어쓰면 참조가 사라짐

```javascript
user = null;
```

```
<global>      Object
user: null    name: "John"
```

### 도달할 수 없는 섬

객체들이 연결되어 섬 같은 구조를 형성했는데, 이 섬에 도달할 방법이 없는 경우

- 섬을 구성하는 객체 전부가 메모리에서 삭제됨

```javascript
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;
  return {
    father: man,
    mother: woman
  };
}
let family = marry({ name: "John" }, { name: "Ann" });
family = null; // 이제 도달할 수 없음
```

### 내부 알고리즘

가비지 컬렉션 기본 알고리즘 `mark-and-sweep`

1. 가비지 컬렉터는 루트(root) 정보를 수집하고 이를 기억(mark)
2. 루트가 참조하고 있는 모든 객체를 방문하고 이것들을 mark
3. mark된 모든 객체에 방문하고 그 객체들이 참조하는 객체도 mark
   - 한번 방문한 객체는 전부 mark하기 때문에 같은 객체를 다시 방문하지 않음
4. 루트에서 도달 가능한 모든 객체를 방문할 때까지 위 과정을 반복
5. mark되지 않은 모든 객체를 메모리에서 삭제

### 최적화 기법

generation collection(세대별 수집)

- 객체를 새로운 객체와 오래된 객체로 분류
- 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모가 없어짐
- 따라서 이것들을 새로운 객체로 분류한 후 가비지 컬렉터가 메모리에서 제거
- 일정 시간 이상 살아남은 객체는 오래된 객체로 분류하고 가비지 컬렉터가 덜 감시

incremental collection(점진적 수집)

- 방문해야 할 객체가 많다면 모든 객체를 한번에 방문하고 mark하는 데 시간과 리소스가 많이 소모되어 실행 속도 저하
- 이를 개선하기 위해 가비지 컬렉션을 여러 부분으로 분리한 후 각 부분을 별도로 수행
- 분리하고 변경 사항을 추적하는 데 추가 작업이 필요하나, 긴 지연을 짧은 지연 여럿으로 분산 가능

idle-time collection(유휴 시간 수집)

- CPU가 유휴 상태일 때만 가비지 컬렉션을 실행해 가비지 컬렉터가 실행에 주는 영향 최소화

## 메서드와 this

실제 존재하는 개체(entity)를 표현하고자 할 때 객체를 생성

- 사용자, 주문 등

객체 프로퍼티에 함수를 할당

- 객체에게 행동할 수 있는 능력 부여

### 메서드 만들기

메서드(method)

- 객체 프로퍼티에 할당된 함수

```javascript
let user = {
  sayHi: function () {}
};
```

메서드 단축 구문

- 일반적인 방법과 완전히 동일하지는 않음
- 객체 상속 관련 미묘한 차이 존재

```javascript
let user = {
  sayHi() {}
};
```

이미 정의된 함수를 붙일 수도 있음

```javascript
let user = { name: "John", age: 30 };
user.sayHi = function () {
  alert("안녕하세요!");
};
user.sayHi(); // 안녕하세요!
```

```javascript
function sayHi() { ... }
user.sayHi = sayHi;
user.sayHi(); // 안녕하세요!
```

객체 지향 프로그래밍(object-oriented programming, OOP)

- 객체를 사용해 개체를 표현하는 방식

### 메서드와 this

메서드 내부에서 `this` 키워드를 사용해 객체 내부의 정보에 접근

- 점 앞의 `this`는 메서드를 호출할 때 사용된 객체를 나타냄

```javascript
let user = {
  name: "John",
  age: 30,
  sayHi() {
    alert(this.name); // this는 현재 객체를 나타냄
  }
};
user.sayHi(); // John
```

`this`를 사용하지 않고 외부 변수를 참조해 객체에 접근 가능

- 그러나 해당 변수가 덮어써지면 원치 않는 값을 참조해 에러가 나타날 수 있음

```javascript
let user = {
  name: "John",
  age: 30,
  sayHi() {
    alert(user.name); // TypeError: Cannot read properties of null (reading 'name')
  }
};
let admin = user;
user = null;
admin.sayHi();
```

### 자유로운 this

자바스크립트에서는 모든 함수에 `this` 사용 가능

```javascript
function sayHi() {
  alert(this.name); // 에러가 발생하지 않음
}
sayHi(); // undefined
```

```javascript
"use strict";
function sayHi() {
  console.log(this.name); // this가 undefined이므로 에러 발생
}
sayHi(); // TypeError: Cannot read properties of undefined (reading 'name')
```

`this` 값은 런타임에 결정되며 컨텍스트에 따라 달라짐

- 동일한 함수라도 다른 객체에서 호출했다면 `this`가 참조하는 값이 달라짐

```javascript
let user = { name: "John" };
let admin = { name: "Admin" };
function sayHi() {
  alert(this.name);
}
user.f = sayHi;
admin.f = sayHi;
user.f(); // John (this == user)
admin.f(); // Admin (this == admin)
admin["f"](); // Admin (점과 대괄호는 동일하게 동작)
```

### 객체 없이 호출하기

객체가 없어도 함수 호출 가능

엄격 모드가 아닐 때는 `this`가 전역 객체를 참조

- 브라우저에서는 `window` 전역 객체
- Node.js에서는 `global` 전역 객체
- 전역 객체 참조는 대개 실수인 경우가 많음

```javascript
function sayHi() {
  alert(this);
}
sayHi(); // [object Window] (엄격 모드 x)
```

엄격 모드일 때는 `this`가 `undefined`가 됨

- `this`가 `undefined`이기 때문에 `this.name` 같이 접근하려고 하면 에러 발생

```javascript
"use strict";
function sayHi() {
  alert(this);
}
sayHi(); // undefined (엄격 모드 o)
```

자유로운 `this`가 만드는 결과

- bound `this`
  - `this`는 항상 메서드가 정의된 객체를 참조할 것이라는 개념
  - 다른 언어를 사용하던 개발자의 혼동
- 자바스크립트에서 `this`는 런타임에 결정
- 메서드가 어디에서 정의되었는지에 상관 없이, `this`는 점 앞의 객체가 무엇인가에 따라 자유롭게 결정
- 장점: 함수(메서드)를 하나만 만들어 여러 객체에서 재사용 가능
- 단점: 유연해서 실수하기 쉬움

### this가 없는 화살표 함수

화살표 함수는 일반 함수와는 달리 고유한 `this`를 가지지 않음

- 화살표 함수에서 `this`를 참조하는 경우
  - 화살표 함수가 아닌 평범한 외부 함수에서 `this` 값을 가져옴
- 별개의 `this`가 만들어지는 건 원하지 않고, 외부 컨텍스트에 있는 `this`를 이용하고 싶은 경우 화살표 함수가 유용

```javascript
let user = {
  firstName: "보라",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow(); // arrow의 this는 외부 함수 user.sayHi()의 this가 됨
  }
};
user.sayHi(); // 보라
```

```javascript
let user = {
  firstName: "bora",
  sayHi() {
    function notArrow() {
      alert(this.firstName); // undefined
    }
    notArrow();
  }
};
user.sayHi();
```

```javascript
let user = {
  firstName: "보라",
  sayHi() {
    function notArrow() {
      return alert(this); // [object Window] (전역 Window 객체)
    }
    notArrow();
    let arrow = () => alert(this); // [object Object] (user 객체)
    arrow();
  }
};
user.sayHi();
```

```javascript
let user = {
  firstName: "보라",
  sayHi() {
    function f() {
      let arrow = () => alert(this.firstName);
      arrow();
    }
    f();
  }
};
user.sayHi(); // undefined
```

### 예제

객체 리터럴에서 this 사용하기

- `this` 값을 설정할 때는 객체 정의가 사용되지 않음
- `this` 값은 호출 시점에 결정됨

```javascript
"use strict";
function makeUser() {
  return { name: "John", ref: this };
}
let user = makeUser();
alert(user.ref.name); // TypeError: Cannot read properties of undefined
```

- `makeUser()` 내 `this`는 `undefined`가 됨
  - 메서드로써 호출된 게 아니라 함수로써 호출되었기 때문
- `this` 값은 전체 함수가 됨
  - 코드 블록과 객체 리터럴은 여기에 영향을 주지 않음
- 따라서 `ref: this`는 함수의 현재 `this` 값을 가져옴

```javascript
"use strict";
function makeUser() {
  return this;
}
alert(makeUser().name); // TypeError: Cannot read properties of undefined
```

- `this`의 값이 `undefined`가 되는 함수
- `alert(makeUser().name)`와 `alert(user.ref.name)`의 결과가 같음

```javascript
function makeUser() {
  return {
    name: "John",
    ref() {
      return this;
    }
  };
}
let user = makeUser();
alert(user.ref().name); // John
```

- `user.ref()`가 메서드가 되고 `this`는 `.` 앞의 객체가 되기 때문에 에러가 발생하지 않음

```javascript
let obj = {
  name: "obj",
  ref: this,
  f() {
    console.log(this);
    return this;
  }
};
console.log(obj.ref.name); // undefined
obj.ref; // {}
let temp = obj.f(); // { name: "test", ref: {}, f: [Function: f] }
console.log(temp); // { name: "test", ref: {}, f: [Function: f] }
```

```javascript
function func() {
  console.log(this);
  return this;
}
func(); // Object global
let temp = {};
temp.f = func;
temp.f(); // { f: [Function: func] }
```

계산기 만들기

```javascript
let calculator = {
  sum() {
    return this.a + this.b;
  },
  mul() {
    return this.a * this.b;
  },
  read() {
    this.a = +prompt("a:", 0);
    this.b = +prompt("b:", 0);
  }
};
calculator.read();
alert(calculator.sum());
alert(calculator.mul());
```

체이닝

- 메서드가 객체 자신을 반환하면 체이닝을 만들 수 있음

`return this;`

```javascript
let ladder = {
  step: 0,
  up() {
    this.step++;
    return this;
  },
  down() {
    this.step--;
    return this;
  },
  showStep() {
    alert(this.step);
    return this;
  }
};
ladder.up().up().down().up().down().showStep(); // 1
```

## 참고

> [객체: 기본](https://ko.javascript.info/object-basics)
