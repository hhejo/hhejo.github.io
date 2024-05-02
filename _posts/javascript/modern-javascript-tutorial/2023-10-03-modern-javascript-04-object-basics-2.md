---
title: 모던 JavaScript 튜토리얼 04 - 객체 기본 2
date: 2023-10-03 20:13:24 +0900
last_modified_at: 2024-04-30 07:28:16 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    new,
    constructor-function,
    optional-chaining,
    symbol,
    toprimitive
  ]
---

new 연산자와 생성자 함수, 메서드, 옵셔널 체이닝 ?., 심볼형, 객체를 원시형으로 변환하기

## new 연산자와 생성자 함수

객체 리터럴 `{...}`

- 객체 여러 개 생성하기 번거로움

`new` 연산자와 생성자 함수

- 유사한 객체를 쉽게 여러 개 만들 수 있음

### 생성자 함수(constructor function)

생성자 함수(constructor function)

- 일반 함수와의 기술적 차이는 없음
- 생성자 함수는 아래 관례를 따름
  1. 함수 이름의 첫 글자는 대문자로 시작
  2. 반드시 `new` 연산자를 붙여 실행
- 모든 함수는 생성자 함수가 될 수 있음
  - 어떤 함수라도 `new`를 붙여 실행하면 생성자 알고리즘 실행
- 생성자를 이용해 재사용할 수 있는 객체 생성 코드 구현

```javascript
function User(name) {
  // this = {}; (빈 객체가 암시적으로 생성됨)
  this.name = name;
  this.isAdmin = false;
  // return this; (this가 암시적으로 반환됨)
}
let user = new User("John");
// let user = { name: "John", isAdmin: false }; // 위 코드와 동일하게 동작
```

익명 생성자 함수

- 재사용은 막으면서 코드를 캡슐화

```javascript
let user = new (function () {
  this.name = "John";
  this.isAdmin = false;
})();
```

### new.target과 생성자 함수

`new.target` 프로퍼티

- 반환값으로 함수가 `new`와 함께 호출되었는지 아닌지 알 수 있음
- 일반적인 방법으로 함수 호출(in regular mode): `undefined` 반환
- `new`와 함께 호출(in constructor mode): 함수 자체를 반환

```javascript
function User() {
  alert(new.target);
}
User(); // undefined
new User(); // function User { ... }
```

```javascript
function User(name) {
  if (!new.target) {
    return new User(name); // new 없이 호출해도 new 붙여줌
  }
  this.name = name;
}
let bora = User("John"); // new User를 쓴 것처럼 변환
```

### 생성자와 return문

생성자 함수에는 보통 `return`문이 없음

- 반환해야 할 것들은 모두 `this`에 저장되고, `this`는 자동으로 반환되기 때문
- `return`문이 있는 생성자 함수는 거의 없음

생성자 함수에 `return`문이 있을 때

- 객체를 리턴하는 경우
  - `this` 대신 해당 객체 반환
- 이외(원시형 등) 리턴하는 경우
  - `return`문 무시

```javascript
function BigUser() {
  this.name = "원숭이";
  return { name: "고릴라" }; // this가 아닌 새로운 객체 반환
}
alert(new BigUser().name); // 고릴라

function SmallUser() {
  this.name = "원숭이";
  return; // this 반환
}
alert(new SmallUser().name); // 원숭이
```

생성자 함수의 괄호 생략하기

- 인수가 없는 생성자 함수는 괄호를 생략해 호출 가능
- 괄호 사용을 권장

```
let user = new User;
let user = new User(); // 위 코드와 동일하게 동작
```

### 생성자 내 메서드

생성자 함수를 사용하면 매개변수를 이용해 객체 내부를 자유롭게 구성할 수 있음

- `this`에 프로퍼티, 메서드 추가 가능
- `class` 문법을 사용하면 생성자 함수를 사용하는 것과 마찬가지로 복잡한 객체 생성 가능

```javascript
function User(name) {
  this.name = name;
  this.sayHi = function () {
    alert(`My name is ${this.name}.`);
  };
}
let user = new User("John");
user.sayHi(); // My name is John.
```

### 예제

함수 두 개로 동일한 객체 만들기

```javascript
let obj = {};
function A() {
  return obj;
}
function B() {
  return obj;
}
alert(new A() == new B()); // true
```

## 옵셔널 체이닝 '?.'

### 옵셔널 체이닝이 필요한 이유

- 존재하지 않는 프로퍼티에 접근해 에러가 발생할 수 있음
- `&&` 연산자 사용: 실제 해당 객체나 프로퍼티가 있는지 확인 가능하나 코드가 길어짐

```javascript
let user = {}; // 주소 정보 없음
alert(user.address.street); // TypeError: Cannot read properties of undefined (reading 'street')
let html = document.querySelector(".my-element").innerHTML; // querySelector(...) 호출 결과가 null인 경우 에러 발생
alert(user && user.address && user.address.street); // undefined (에러가 발생하지 않으나 코드가 길어짐)
```

### 옵셔널 체이닝의 등장

옵셔널 체이닝(optional chaining) `?.`

- 프로퍼티가 없는 중첩 객체를 에러 없이 안전하게 접근
- 앞의 평가 대상이 `undefined`, `null`이면 평가를 멈추고 `undefined` 반환
- 연산자가 아닌 함수나 대괄호와 함께 동작하는 특별한 문법 구조체(syntax construct)

```javascript
let user = {};
alert(user?.address?.street); // undefined
let user2 = null;
alert(user2?.address); // undefined
let user3 = {};
alert(user3?.address.street); // TypeError: Cannot read properties of undefined (reading 'street')
user4?.address; // ReferenceError: user4 is not defined
```

### 단락 평가

`?.`는 왼쪽 평가 대상에 값이 없으면 평가 멈춤

```javascript
let user = null;
let x = 0;
user?.sayHi(x++);
alert(x); // 0
```

### ?.()와 ?.[]

`?.`

- 연산자가 아님
- 함수나 대괄호와 함께 동작하는 특별한 문법 구조체
- `delete`와 조합 가능
- 읽기나 삭제하기에는 사용할 수 있지만 쓰기에는 사용 불가
  - `undefined = value`가 되기 때문

```javascript
let user1 = {
  admin() {
    alert("관리자 계정입니다.");
  }
};
let user2 = {};
user1.admin?.(); // 관리자 계정입니다.
user2.admin?.();

let user3 = { firstName: "John" };
let user4 = null;
let key = "firstName";
alert(user3?.[key]); // John
alert(user4?.[key]); // undefined
alert(user3?.[key]?.something?.not?.existing); // undefined

delete user?.name; // user가 존재하면 user.name을 삭제

user?.name = "John"; // SyntaxError: Invalid left-hand side in assignment
```

## 심볼형

객체 프로퍼티 키

- 문자형
- 심볼형

### 심볼(symbol)

`Symbol()`

- 유일한 식별자(unique identifier)를 만들고 싶을 때 사용
- 유일성 보장: 동일 심볼을 여러 개 만들어도 각 심볼값은 다름
- 심볼 이름: 심볼에 붙이는 설명
  - 어떤 것에도 영향을 주지 않는 이름표 역할
- 심볼형 값은 다른 자료형으로 암시적 형 번환(자동 형 변환)이 되지 않음
  - `symbol.toString()`: 문자열로 출력
  - `symbol.description`: 설명을 보임(프로퍼티)

```javascript
let id = Symbol(); // id는 심볼
let id = Symbol("id"); // 'id'라는 설명이 붙는 심볼 id

let id1 = Symbol("id");
let id2 = Symbol("id");
alert(id1 == id2); // false

let id3 = Symbol("id");
alert(id3); // TypeError: Cannot convert a Symbol value to a string
alert(id3.toString()); // Symbol(id)
alert(id3.valueOf()); // TypeError: Cannot convert a Symbol value to a string
alert(id3.description); // id
```

### 숨김(hidden) 프로퍼티

- 외부 코드에서 접근이 불가능하고 값도 덮어쓸 수 없는 프로퍼티
- 심볼은 서드파티 코드에서 접근할 수 없음
- 심볼을 사용하면 서드파티 코드가 모르게 객체에 식별자 부여 가능

```javascript
let user = { name: "John" }; // 서드파티 코드에서 가져온 객체
user.id = "스크립트 id 값";
user.id = "제3 스크립트 id 값"; // 일반적인 프로퍼티를 사용 시 의도치 않게 덮어 씀
let id = Symbol("id");
user[id] = "제 3 스크립트 id 값"; // 덮어쓰지 않음(심볼을 키로 사용해 데이터에 접근)
let user2 = { name: "Pete", [id]: 123 }; // [id]가 아닌 id로 쓰면 심볼 id가 아닌 문자열 "id"가 키가 됨
```

심볼형 프로퍼티 숨기기(hiding symbolic property)

- 아래 방법들에 대해 드러나지 않음
- `for...in`
- `Object.keys()`, `Object.values()`, `Object.entries()`
- 외부 스크립트나 라이브러리
  - 심볼형 키를 가진 프로퍼티에 접근 불가

```javascript
let id = Symbol("id");
let user = { name: "John", age: 30, [id]: 123 };
for (let key in user) alert(key); // name, age
alert(Object.keys(user)); // name,age
```

심볼형 프로퍼티 복사하기

- `Object.assign`: 키가 심볼인 프로퍼티를 배제하지 않고 객체 내 모든 프로퍼티를 복사
- `{ ...obj }`: 키가 심볼인 프로퍼티를 배제하지 않음

```javascript
let id = Symbol("id");
let user = { [id]: 123 };
let clone = Object.assign({}, user); // { ...user };
alert(clone[id]); // 123
```

### 전역 심볼

- 전역 심볼 레지스트리(global symbol registry) 안에 있는 심볼
- 이름이 같은 심볼이 같은 개체를 가리킬 수 있음
- 그냥 심볼은 이름이 같더라도 별개로 취급

`Symbol.for(key)`

- 레지스트리 안에 있는 심볼을 읽거나, 새로운 심볼을 생성
- 이름이 `key`인 심볼을 반환
- 조건에 맞는 심볼이 레지스트리 안에 없으면 새로운 심볼 `Symbol(key)`을 만들고 레지스트리 안에 저장

`Symbol.keyFor(sym)`

- 전역 심볼 레지스트리에서 해당 전역 심볼의 이름을 반환
- 전역 심볼이 아닌 인자는 `undefined` 반환

`symbol.description`

- 일반 심볼의 이름 반환
- 전역 심볼이 아닌 모든 심볼은 `description` 프로퍼티가 있음

```javascript
let id = Symbol.for("id");
let idAgain = Symbol.for("id");
alert(id === idAgain); // true

// 이름으로 심볼 찾기
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");
// 심볼로 이름 얻기
alert(Symbol.keyFor(sym)); // name
alert(Symbol.keyFor(sym2)); // id

let globalSymbol = Symbol.for("name");
let localSymbol = Symbol("name");
alert(Symbol.keyFor(globalSymbol)); // name, 전역 심볼
alert(Symbol.keyFor(localSymbol)); // undefined, 전역 심볼이 아님
alert(localSymbol.description); // name
```

### 시스템 심볼

- 자바스크립트 내부에서 사용되는 심볼
- 내장 메서드 등의 기본 동작을 원하는 대로 변경 가능
- 객체 미세 조정 가능
- `Symbol.*`로 접근
  - `Symbol.toPrimitive`
  - `Symbol.iterator`
  - `Symbol.hasInstance`
  - `Symbol.isConcatSpreadable`
  - 기타 등등

## 객체를 원시형으로 변환하기

객체 -> 원시형

- 불린형: 객체는 논리 평가 시 `true` 반환
- 숫자형: 객체끼리 빼는 연산 혹은 수학 관련 함수를 적용할 때
- 문자형: `alert(obj)` 같이 객체를 출력하려고 할 때

### ToPrimitive

특수 객체 메서드

- 숫자형이나 문자형으로의 형 변환을 원하는 대로 조절 가능
- `hint`: 목표로 하는 자료형
  - 객체 형 변환의 세 종류의 기준
  - 모든 객체는 `true`로 평가되어 `boolean` hint는 없음

3가지 종류의 `hint`

1. `string`: 문자열을 기대하는 연산을 수행할 때 (alert 함수 등)
2. `number`: 수학 연산을 적용하려 할 때
3. `default`: 연산자가 기대하는 자료형이 확실치 않을 때 (아주 드물게 발생)
   - hint가 `default`인 경우와 `number`인 경우를 동일하게 처리
     - 모든 내장 객체 (`Date` 객체 제외)
   - 이항 덧셈 연산자 `+`: 인수가 객체일 때
   - 동등 연산자 `==`: 객체-문자형, 객체-숫자형, 객체-심볼형끼리 비교할 때
     - 객체를 어떤 자료형으로 바꿔야 할지 정확하지 않음
   - 비교 연산자 `<`, `>`: hint를 `number`로 고정해 hint가 `default`가 되는 일이 없음 (하위 호환성 때문)
     - 피연산자에 문자형, 숫자형 둘 다 허용

자바스크립트 형 변환 알고리즘

1. 객체에 `obj[Symbol.toPrimitive](hint)` 메서드가 있는지 찾고, 있다면 호출
2. 1에 해당하지 않고, hint가 `string`: `obj.toString()`이나 `obj.valueOf()` 호출
   - 존재하는 메서드만 실행
3. 1과 2에 해당하지 않고, hint가 `number`나 `default`: `obj.valueOf()`나 `obj.toString()` 호출
   - 존재하는 메서드만 실행

```javascript
// hint가 string
alert(obj); // 객체를 출력하려고 함
anotherObj[obj] = 123; // 객체를 프로퍼티 키로 사용

// hint가 number
let num = Number(obj); // 명시적 형 변환
let n = +obj; // 단항 덧셈 연산
let delta = date1 - date2; // 수학 연산 (이항 덧셈 연산 제외)
let greater = user1 > user2; // 대소 비교

// hint가 default
let total = obj1 + obj2; // 이항 덧셈 연산
if (obj == 1) { ... } // 객체-숫자형 비교
```

```javascript
alert(!!{}); // true
alert(!![]); // true
alert(!!""); // false
alert(!!" "); // true

/*
{} + {} 콘솔에서 NaN
alert({} + {}) 는 [objectObject][objectObject]
[] + [] 콘솔에서 ""
alert([] + []) 는 ""
*/
```

### Symbol.toPrimitive

`Symbol.toPrimitive`

```javascript
obj[Symbol.toPrimitive] = function (hint) {};
```

- 내장 심볼(시스템 심볼)
- 심볼형 키로 사용
- 목표로 하는 자료형(hint)을 명명하는 데 사용
- 모든 종류의 형 변환 가능
- hint는 string, number, default 중 하나
- 반드시 원시값을 반환

```javascript
let user = {
  name: "John",
  money: 1000,
  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint === "string" ? `{name: "${this.name}"}` : this.money;
  }
};
alert(user); // hint: string, {name: 'John'}
alert(+user); // hint: number, 1000
alert(user + 500); // hint: default, 1500
```

### toString과 valueOf

`toString`, `valueOf`

- 심볼이 생기기 이전부터 존재한 평범한 메서드
- 형 변환을 직접 구현 가능(구식 방법)
- 반드시 원시값을 반환해야 함
  - 객체를 반환하면 그 결과는 무시됨(메서드가 처음부터 없었던 것처럼 됨)
- 일반 객체는 기본적으로 `toString`과 `valueOf`에 적용되는 다음 규칙을 따름
  - `toString`: 문자열 `"[object Object]"` 반환
  - `valueOf`: 객체 자신 반환 (메서드가 존재하지 않는 것처럼 무시됨. 역사적인 이유 때문)

객체에 `Symbol.toPrimitive`가 없는 경우

- 자바스크립트는 아래 규칙에 따라 `toString`이나 `valueOf`를 호출
  - hint가 string: toString -> valueOf (toString이 있다면 호출, 없다면 valueOf 호출)
  - 그 외: valueOf -> toString

모든 형 변환을 한 곳에서 처리해야 하는 경우

- `toString`만 구현하면 됨
- 객체에 `Symbol.toPrimitive`와 `valueOf`가 없으면 `toString`이 모든 형 변환을 처리

```javascript
let obj = { name: "test" };
alert(obj); // [object Object]
alert(obj.valueOf() === obj); // true

let user = {
  name: "John",
  money: 1000,
  toString() {
    return `{name: "${this.name}"}`; // hint=string
  },
  valueOf() {
    return this.money; // hint=number or hint=default
  }
};
alert(user); // toString, {name: "John"}
alert(+user); // valueOf, 1000
alert(user + 500); // valueOf, 1500

let user2 = {
  name: "John",
  toString() {
    return this.name;
  }
};
alert(user2); // John (toString 호출)
alert(user2 + 500); // John500 (toString 호출)
```

### 반환 타입

세 개의 메서드는 hint에 명시된 자료형으로의 형 변환을 보장해주지 않음

- `toString()`: 항상 문자열을 반환하리라는 보장이 없음
- `Symbol.toPrimitive`: hint가 `number`일 때 항상 숫자형 자료가 반환되리라는 보장이 없음
- 객체가 아닌 원시값을 반환해준다는 것만 확실

객체를 반환하는 경우

- `toString`, `valueOf`: 에러가 발생하지 않음
  - 반환값이 무시되고 메서드 자체가 존재하지 않았던 것처럼 동작
  - 과거 자바스크립트에는 에러라는 개념이 잘 정립되어 있지 않았기 때문
- `Symbol.toPrimitive`: 에러 발생
  - 무조건 원시자료를 반환해야 함

```javascript
let obj = {
  [Symbol.toPrimitive](hint) {
    return { test: "testObject" };
  }
};
console.log(+obj); // TypeError: Cannot convert object to primitive value
console.log(`${obj}`); // TypeError: Cannot convert object to primitive value
```

### 추가 형 변환

객체가 피연산자일 때

1. 객체 -> 원시형 형 변환
2. 변환 후 원시값이 원하는 형이 아닌 경우, 또다시 형 변환 발생

```javascript
let obj = {
  toString() {
    return "2";
  }
};
alert(obj * 2); // 4. "2" * 2 -> 2 * 2 -> 4
alert(obj + 2); // 22
```

## 참고

> [객체: 기본](https://ko.javascript.info/object-basics)
