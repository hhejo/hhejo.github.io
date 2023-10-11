---
title: 모던 JavaScript 튜토리얼 04 - 객체 기본 2
date: 2023-10-03 20:13:24 +0900
last_modified_at: 2023-10-11 20:35:59 +0900
categories: [JavaScript]
tags: [javascript]
---

new 연산자, 생성자 함수, 메서드, 옵셔널 체이닝 ?., 심볼, 숨김 프로퍼티, 전역 심볼, 객체를 원시형으로 변환하기, Symbol.toPrimitive, toString, valueOf

## new 연산자와 생성자 함수

객체 리터럴 `{...}`은 객체 여러 개 만들기 불편함

`new` 연산자와 생성자 함수로 유사한 객체를 여러 개 만들 수 있음

### 생성자 함수(constructor function)

생성자 함수와 일반 함수에 기술적 차이는 없으나, 생성자 함수는 아래 관례를 따름

- 함수 이름의 첫 글자는 대문자로 시작
- 반드시 `new` 연산자를 붙여 실행

생성자를 이용해 재사용할 수 있는 객체 생성 코드 구현 가능

모든 함수는 생성자 함수가 될 수 있음. 어떤 함수라도 `new`를 붙여 실행하면 생성자 알고리즘 실행

```javascript
function User(name) {
  // this = {}; (빈 객체가 암시적으로 생성됨)
  this.name = name;
  this.isAdmin = false;
  // return this; (this가 암시적으로 반환됨)
}
let user = new User("보라"); // 아래 코드와 동일하게 동작
let user = { name: "보라", isAdmin: false };
```

### 익명 생성자 함수

재사용은 막으면서 코드 캡슐화 가능

```javascript
let user = new (function () {
  this.name = "John";
  this.isAdmin = false;
})();
```

### new.target과 생성자 함수

`new.target` 프로퍼티를 사용하면 함수가 `new`와 함께 호출되었는지 아닌지 알 수 있음

일반적인 방법으로 함수 호출

- `new.target`이 `undefined` 반환

`new`와 함께 호출

- `new.target`은 함수 자체를 반환

```javascript
function User() {
  alert(new.target);
}
User(); // undefined
new User(); // function User { ... }
```

### 생성자와 return문

생성자 함수에는 보통 `return`문이 없음

반환해야 할 것들은 모두 `this`에 저장되고, `this`는 자동으로 반환되기 때문

`return`문이 있으면

- 객체를 `return`한다면 `this` 대신 객체 반환
- 원시형을 `return`한다면 `return`문이 무시됨

즉, `return` 뒤에 객체가 오면 생성자 함수는 해당 객체를 반환하고, 이외의 경우는 `this` 반환

`return`문이 있는 생성자 함수는 거의 없음

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

인수가 없는 생성자 함수는 괄호를 생략해 호출 가능하지만 괄호 사용을 권장

```
let user = new User;
let user = new User(); // 위 코드와 동일하게 동작
```

### 생성자 내 메서드

생성자 함수를 사용하면 매개변수를 이용해 객체 내부를 자유롭게 구성할 수 있음

메서드도 더할 수 있음

`class` 문법을 사용하면 생성자 함수를 사용하는 것과 마찬가지로 복잡한 객체 생성 가능

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

옵셔널 체이닝(optional chaining) `?.`으로 프로퍼티가 없는 중첩 객체를 에러 없이 안전하게 접근

연산자가 아닌 함수나 대괄호와 함께 동작하는 특별한 문법 구조체(syntax construct)

```javascript
let user = {}; // 주소 정보 없음
alert(user.address.street); // TypeError: Cannot read properties of undefined (reading 'street')
```

```javascript
let html = document.querySelector(".my-element").innerHTML; // querySelector(...) 호출 결과가 null인 경우 에러 발생
```

### 옵셔널 체이닝의 등장 이전

`&&` 연산자를 사용해 에러 방지

```javascript
let user = {};
alert(user && user.address && user.address.street); // undefined (에러가 발생하지 않음)
```

### 옵셔널 체이닝의 등장

`?.`은 앞의 평가 대상이 `undefined`나 `null`이면 평가를 멈추고 `undefined`를 반환

```javascript
let user = {};
alert(user?.address?.street); // undefined (에러가 발생하지 않음)
```

`user`가 `null`이나 `undefined`가 아닌 경우, `user.address` 프로퍼티가 존재하지 않는다면 에러 발생

```javascript
let user = null;
alert(user?.address); // undefined
alert(user?.address.street); // undefined
```

변수 `user`가 선언되어 있지 않으면 에러 발생

```javascript
user?.address; // ReferenceError: user is not defined
```

`?.`는 왼쪽 평가 대상에 값이 없으면 평가 멈춤

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

`?.`은 읽기나 삭제하기에는 사용할 수 있지만 쓰기에는 사용할 수 없음

```javascript
user?.name = "Violet"; // SyntaxError: Invalid left-hand side in assignment
// undefined = 'Violet'이 되기 때문
```

## 심볼형

객체 프로퍼티 키로 문자형과 심볼형만 허용

### 심볼(symbol)

`Symbol()`

유일한 식별자(unique identifier)를 만들고 싶을 때 사용

유일성을 보장하기 때문에 동일 심볼을 여러 개 만들어도 각 심볼값은 다름

심볼에 붙이는 설명(심볼 이름)은 어떤 것에도 영향을 주지 않는 이름표 역할

```javascript
let id = Symbol(); // id는 심볼
let id = Symbol("id"); // 'id'라는 설명이 붙는 심볼 id
```

```javascript
let id1 = Symbol("id");
let id2 = Symbol("id");
alert(id1 == id2); // false
```

심볼형 값은 다른 자료형으로 암시적 형 번환(자동 형 변환)이 되지 않음

```javascript
let id = Symbol("id");
alert(id); // TypeError: Cannot convert a Symbol value to a string
alert(id.toString()); // Symbol(id)
alert(id.description); // id
```

### 숨김(hidden) 프로퍼티

외부 코드에서 접근이 불가능하고 값도 덮어쓸 수 없는 프로퍼티

```javascript
let user = { name: "John" }; // 서드파티 코드에서 가져온 객체
let id = Symbol("id");
user[id] = 1;
alert(user[id]); // 1, 심볼을 키로 사용해 데이터에 접근
```

- `user`는 서드파티에서 가져 온 객체이므로 함부로 새로운 프로퍼티를 추가할 수 없음
- 심볼은 서드파티 코드에서 접근할 수 없기 때문에, 심볼을 사용하면 서드파티 코드가 모르게 `user`에 식별자 부여 가능

```javascript
let id = Symbol("id");
user[id] = "제 3 스크립트 id 값";
```

```javascript
let user = { name: "John" };
user.id = "스크립트 id 값";
user.id = "제3 스크립트 id 값"; // 의도치 않게 덮어 씀
```

객체 리터럴 `{...}`을 사용해 객체를 만든 경우, 대괄호를 사용해 심볼형 키를 만들어야 함

```javascript
let id = Symbol("id");
let user = { name: "John", [id]: 123 };
```

심볼은 `for...in`에서 배제됨

`Object.keys()`에서도 키가 심볼인 프로퍼티는 배제됨

심볼형 프로퍼티 숨기기(hiding symbolic property)

```javascript
let id = Symbol("id");
let user = { name: "John", age: 30, [id]: 123 };
for (let key in user) alert(key); // name, age
alert(Object.keys(user)); // name,age
```

`Object.assign`은 키가 심볼인 프로퍼티를 배제하지 않고 객체 내 모든 프로퍼티를 복사

```javascript
let id = Symbol("id");
let user = { [id]: 123 };
let clone = Object.assign({}, user);
alert(clone[id]); // 123
```

### 전역 심볼

전역 심볼 레지스트리 안에 있는 심볼

전역 심볼 레지스트리(global symbol registry)를 이용해 이름이 같은 심볼이 같은 개체를 가리킬 수 있음

`Symbol.for(key)`를 사용해 레지스트리 안에 있는 심볼을 읽거나, 새로운 심볼을 생성

해당 메서드를 호출하면 이름이 `key`인 심볼을 반환. 조건에 맞는 심볼이 레지스트리 안에 없으면 새로운 심볼 `Symbol(key)`를 만들고 레지스트리 안에 저장

```javascript
let id = Symbol.for("id");
let idAgain = Symbol.for("id");
alert(id === idAgain); // true
```

`Symbol.keyFor(sym)`를 사용하면 이름을 얻을 수 있음

전역 심볼 레지스트리를 뒤져서 해당 심볼의 이름을 얻어냄

전역 심볼이 아닌 인자가 넘어오면 `undefined`를 반환

```javascript
// 이름을 이용해 심볼을 찾음
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");
// 심볼을 이용해 이름을 얻음
alert(Symbol.keyFor(sym)); // name
alert(Symbol.keyFor(sym2)); // id
```

전역 심볼이 아닌 모든 심볼은 `description` 프로퍼티가 있음

일반 심볼에서 이름을 얻고 싶으면 `description` 프로퍼티 사용

```javascript
let globalSymbol = Symbol.for("name");
let localSymbol = Symbol("name");
alert(Symbol.keyFor(globalSymbol)); // name, 전역 심볼
alert(Symbol.keyFor(localSymbol)); // undefined, 전역 심볼이 아님
alert(localSymbol.description); // name
```

### 시스템 심볼

자바스크립트 내부에서 사용되는 심볼

객체를 미세 조정할 수 있음

`Symbol.hasInstance`, `Symbol.isConcatSpreadable`, `Symbol.iterator`, `Symbol.toPrimitive`, ...

## 객체를 원시형으로 변환하기

객체는 논리 평가시 `true`를 반환

객체끼리 빼는 연산을 할 때나 수학 관련 함수를 적용할 때 숫자형으로의 형 변환 발생

문자형으로의 형 변환은 대개 `alert(obj)` 같이 객체를 출력하려고 할 때 발생

### ToPrimitive

특수 객체 메서드를 사용해 숫자형이나 문자형으로의 형 변환을 원하는 대로 조절 가능

hint(목표로 하는 자료형)라 불리는 값이 객체 형 변환의 세 종류의 기준

`string`

- `alert` 함수 같이 문자열을 기대하는 연산을 수행할 때는 hint가 `string`이 됨
- ```javascript
  alert(obj); // 객체를 출력하려고 함
  anotherObj[obj] = 123; // 객체를 프로퍼티 키로 사용하고 있음
  ```

`number`

- 수학 연산을 적용하려 할 때, hint는 `number`가 됨
- ```javascript
  let num = Number(obj); // 명시적 형 변환
  let n = +obj;
  let delta = date1 - date2;
  let greater = user1 > user2;
  ```

`default`

- 연산자가 기대하는 자료형이 확실치 않을 때, hint는 `default`가 됨 (아주 드물게 발생)
- 이항 덧셈 연산자 `+`는 인수가 객체일 때 hint가 `default`가 됨
- 동등 연산자 `==`를 사용해 객체-문자형, 객체-숫자형, 객체-심볼형끼리 비교할 때도 객체를 어떤 자료형으로 바꿔야 할지 확신이 안 서므로 hint는 `default`가 됨
- ```javascript
  let total = obj1 + obj2;
  if (obj == 1) { ... }
  ```
- `<`, `>` 역시 피연산자에 문자형과 숫자형 둘 다를 허용하는데, 이 연산자들은 hint를 `number`로 고정해 hint가 `default`가 되는 일이 없음. 하위 호환성 때문
- `Date` 객체를 제외한 모든 내장 객체는 hint가 `default`인 경우와 `number`인 경우를 동일하게 처리
- `boolean` hint는 존재하지 않음. 모든 객체는 `true`로 평가됨

```javascript
alert(!!{}); // true
alert(!![]); // true
alert(!!""); // false
alert(!!" "); // true
```

자바스크립트는 형 변환이 필요할 때, 아래와 같은 알고리즘에 따라 원하는 메서드를 찾고 호출

1. 객체에 `obj[Symbol.toPrimitive](hint)` 메서드가 있는지 찾고, 있다면 호출. `Symbol.toPrimitive`는 시스템 심볼로, 심볼형 키로 사용됨
2. 1에 해당하지 않고 hint가 `string`이라면 `obj.toString()`이나 `obj.valueOf()`를 호출 (존재하는 메서드만 실행됨)
3. 1과 2에 해당하지 않고, hint가 `number`나 `default`라면 `obj.valueOf()`나 `obj.toString()`을 호출 (존재하는 메서드만 실행됨)

### Symbol.toPrimitive

자바스크립트엔 `Symbol.toPrimitive`라는 내장 심볼 존재

목표로 하는 자료형(hint)을 명명하는 데 사용

```javascript
obj[Symbol.toPrimitive] = function (hint) {
  // 반드시 원시값을 반환해야 함
  // hint는 string, number, default 중 하나가 될 수 있음
};
```

```javascript
let user = {
  name: "John",
  money: 1000,
  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint === "string" ? `{name: "${this.name}"}` : this.money;
  }
};
alert(user); // hint: string -> {name: 'John'}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

`[Symbol.toPrimitive](hint) {...}` 메서드 하나로 모든 종류의 형 변환을 다룰 수 있음

### toString과 valueOf

`toString`과 `valueOf`는 심볼이 생기기 이전부터 존재해왔던 평범한 메서드

이 메서드를 사용하면 구식이긴 하지만 형 변환을 직접 구현 가능

객체에 `Symbol.toPrimitive`가 없으면 자바스크립트는 아래 규칙에 따라 `toString`이나 `valueOf`를 호출

- hint가 string인 경우: toString -> valueOf 순 (toString이 있다면 toString 호출, toString이 없다면 valueOf 호출)
- 그 외: valueOf -> toString 순

이 메서드들은 반드시 원시값을 반환해야 함

toString이나 valueOf가 객체를 반환하면 그 결과는 무시됨. 메서드가 처음부터 없었던 것처럼 됨

일반 객체는 기본적으로 `toString`과 `valueOf`에 적용되는 다음 규칙을 따름

- `toString`은 문자열 `"[object Object]"`를 반환
- `valueOf`는 객체 자신을 반환 (메서드가 존재하지 않는 것 처럼 무시됨)

```javascript
let user = { name: "John" };
alert(user); // [object Object]
alert(user.valueOf() === user); // true
```

아래 예제는 출력 결과가 `Symbol.toPrimitive`를 사용한 예제와 동일

```javascript
let user = {
  name: "John",
  money: 1000,
  // hint가 string인 경우
  toString() {
    return `{name: "${this.name}"}`;
  },
  // hint가 number나 default인 경우
  valueOf() {
    return this.money;
  }
};
alert(user); // toString -> {name: "John"}
alert(+user); // valueOf -> 1000
alert(user + 500); // valueOf -> 1500
```

모든 형 변환을 한 곳에서 처리해야 하는 경우, `toString`만 구현해주면 됨

객체에 `Symbol.toPrimitive`와 `valueOf`가 없으면, `toString`이 모든 형 변환을 처리

```javascript
let user = {
  name: "John",
  toString() {
    return this.name;
  }
};
alert(user); // toString -> John
alert(user + 500); // toString -> John500
```

### 반환 타임

위에서 소개한 세 개의 메서드는 hint에 명시된 자료형으로의 형 변환을 보장해주지 않음

`toString()`이 항상 문자열을 반환하리라는 보장이 없고, `Symbol.toPrimitive`의 hint가 `number`일 때 항상 숫자형 자료가 반환되리라는 보장이 없음

객체가 아닌 원시값을 반환해준다는 것만 확실

과거의 잔재

- `toString`이나 `valueOf`가 객체를 반환해도 에러가 발생하지 않음. 다만 이때는 반환값이 무시되고, 메서드 자체가 존재하지 않았던 것처럼 동작
- 과거 자바스크립트엔 에러라는 개념이 잘 정립되어 있지 않았기 때문
- 반면에 `Symbol.toPrimitive`는 무조건 원시자료를 반환해야 함. 아니면 에러 발생

### 추가 형 변환

객체가 피연산자일 때는 다음 단계를 거쳐 형 변환 발생

1. 객체는 원시형으로 변화됨
2. 변환 후 원시값이 원하는 형이 아닌 경우엔 또다시 형 변환 발생

```javascript
let obj = {
  toString() {
    return "2";
  }
};
alert(obj * 2); // 4
```

```javascript
let obj = {
  toString() {
    return "2";
  }
};
alert(obj + 2); // 22
```

## 참고

> [객체: 기본](https://ko.javascript.info/object-basics)
