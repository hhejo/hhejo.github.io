---
title: 모던 JavaScript 튜토리얼 04 - 객체 기본 1
date: 2023-10-02 12:01:57 +0900
last_modified_at: 2023-10-02 12:01:57 +0900
categories: [JavaScript]
tags: [javascript]
---

객체, 리터럴, 프로퍼티, in 연산자, for...in 반복문, 객체 복사, 가비지 컬렉션, 메서드, this

## 객체(Object)

- 7개의 원시형(primitive) 자료형과 다르게 객체형은 키로 구분된 데이터 집합이나 복잡한 개체(entity)를 담을 수 있음
- 프로퍼티(property): `키(key): 값(value)` 쌍으로 구성
- 키는 문자형만, 값은 모든 자료형 가능
- 빈 객체를 만드는 방법
  1. 객체 생성자
  2. 객체 리터럴(object literal)

```javascript
let user = new Object(); // 객체 생성자 문법
let user = {}; // 객체 리터럴 문법
```

### 리터럴과 프로퍼티

```javascript
let user = {
  name: "John",
  age: 30
};
```

- 점 표기법(dot notation)을 이용해 프로퍼티 값을 읽을 수 있음
- `delete` 연산자를 사용해 프로퍼티 삭제 가능
- 여러 단어를 조합한 프로퍼티 이름을 만드는 경우 이름을 따옴표로 묶어야 함. 이 경우 대괄호 표기법(square bracket notation)을 사용할 수 있음

```javascript
let user = {
  age: 30,
  "like birds": true
};
user.name = "john";
delete user.age;
alert(user["like birds"]);
key = "name";
alert(user[key]); // john
alert(user.key); // undefined
```

### 계산된 프로퍼티

- 객체를 만들 때 객체 리터럴 안의 프로퍼티 키가 대괄호로 둘러싸인 경우

```javascript
let fruit = prompt("과일 이름", "apple");
let bag = {
  [fruit]: 5
};
```

### 단축 프로퍼티

- 프로퍼티 값을 기존 변수에서 받아와 사용하는 경우가 많음
- 프로퍼티 값 단축 구문(property value shorthand)을 사용해 코드를 짧게 줄일 수 있음

```javascript
function makeUser(name, age) {
  return {
    name: name,
    age: age
  };
}
let user = makeUser("John", 30);
alert(user.name);
```

```javascript
function makeUser(name, age) {
  return { name, age };
}
```

### 프로퍼티 이름의 제약 사항

- 예약어를 키로 사용해도 상관 없음
- 문자형이나 심볼형에 속하지 않은 값은 문자열로 자동 형변환 됨
- 객체 프로퍼티 키의 `__proto__`는 특별 취급

### 'in' 연산자로 프로퍼티 존재 여부 확인하기

- 자바스크립트는 존재하지 않는 프로퍼티에 접근해도 에러가 발생하지 않고 `undefined`를 반환
- ```javascript
  let user = {};
  alert(user.noSuchProperty === undefined); // true
  ```
- `"key" in object`
- ```javascript
  let obj = { test: undefined };
  alert(obj.test); // undefined. 키 test가 존재하고 값이 undefined인지, 키 test가 존재하지 않는지 모름
  alert("test" in obj); // true
  ```

### 'for...in' 반복문

```javascript
for (key in object) {
  ...
}
```

```javascript
let user = { name: "John", age: 30, isAdmin: true };
for (let key in user) {
  alert(key);
  alert(user[key]);
}
```

### 객체 정렬 방식

- 객체는 특별한 방식으로 정렬됨
- 정수 프로퍼티: 변형 없이 정수에서 왔다 갔다 할 수 있는 문자열
- 키가 정수인 경우, 자동 정렬
- 키가 정수가 아닌 경우, 작성된 순서대로 프로퍼티 나열

## 참조에 의한 객체 복사

- 객체는 참조에 의해(by reference) 저장되고 복사됨
- 원시값은 값 그대로

### 참조에 의한 비교

- 객체 비교시 동등 연산자 `==`와 일치 연산자 `===`는 동일하게 동작
- ```javascript
  let a = {};
  let b = a;
  alert(a == b); // true
  alert(a === b); // true
  let c = {};
  let d = {};
  alert(a == b); // false
  ```
- `obj1 > obj2` 같은 대소 비교나 `obj == 5` 같은 원시값과의 비교에선 객체가 원시형으로 변환

### 객체 복사, 병합과 Object.assign

- 자바스크립트는 객체 복사 내장 메서드를 지원하지 않기 때문에 새로운 객체를 만든 다음 기존 객체의 프로퍼티들을 순회해 원시 수준까지 프로퍼티를 복사하면 됨

```javascript
let user = { name: "John", age: 30 };
let clone = {};
for (let key in user) {
  clone[key] = user[key];
}
```

- `Object.assign`을 사용할 수도 있음
- 목표로 하는 객체 `dest`에 복사하고자 하는 객체 `src1, src2, ...`를 작성해 각 프로퍼티를 복사하고 `dest`를 반환
- 동일한 이름을 가진 프로퍼티가 있는 경우 덮어씀

```
Object.assign(dest, [src1, src2, src3, ...])
```

```javascript
let user = { name: "John", age: 30 };
let clone = Object.assign({}, user); // 반복문 없이 객체 복사
```

### 중첩 객체 복사

- 깊은 복사를 해야 하기 때문에 주의

## 가비지 컬렉션

자바스크립트 엔진이 필요 없는 원시값, 객체, 함수 등을 찾아내 삭제하는 방법

### 가비지 컬렉션 기준

- 도달 가능성(reachability) 개념을 사용해 메모리 관리를 수행
- 어떻게든 접근하거나 사용할 수 있는, 즉 도달 가능한 값은 메모리에서 삭제되지 않음
- 태생부터 도달 가능하기 때문에 명백한 이유 없이 삭제되지 않는 값, 루트(root)
  1. 현재 함수의 지역변수와 매개변수
  2. 중첩 함수의 체인에 있는 함수에서 사용되는 변수와 매개변수
  3. 전역변수, 기타 등등
- 루트가 참조하는 값이나 체이닝으로 루트에서 참조할 수 있는 값은 도달 가능한 값이 됨

### 도달할 수 없는 섬

- 객체들이 연결되어 섬 같은 구조를 만드는데, 이 섬에 도달할 방법이 없는 경우, 섬을 구성하는 객체 전부가 메모리에서 삭제

### 내부 알고리즘

- 가비지 컬렉션 기본 알고리즘: `mark-and-sweep`

1. 가비지 컬렉터는 루트(root) 정보를 수집하고 이를 기억(mark)
2. 루트가 참조하고 있는 모든 객체를 방문하고 이것들을 mark
3. mark된 모든 객체에 방문하고 그 객체들이 참조하는 객체도 mark. 한번 방문한 객체는 전부 mark하기 때문에 같은 객체를 다시 방문하지 않음
4. 루트에서 도달 가능한 모든 객체를 방문할 때까지 위 과정을 반복
5. mark되지 않은 모든 객체를 메모리에서 삭제

### 최적화 기법

- generation collection(세대별 수집): 객체를 새로운 객체와 오래된 객체로 나눔. 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모가 없어지기 때문에 이것들을 새로운 객체로 구분하고 가비지 컬렉터가 메모리에서 제거. 일정 시간 이상 살아남은 객체는 오래된 객체로 분류하고 가비지 컬렉터가 덜 감시
- incremental collection(점진적 수집): 방문해야할 객체가 많다면 모든 객체를 한번에 방문하고 mark하는데 시간과 리소스가 많이 소모되어 실행 속도도 저하됨. 그래서 가비지 컬렉션을 여러 부분으로 분리한 후 각 부분을 별도로 수행. 분리하고 변경 사항을 추적하는 데 추가 작업이 필요하나 긴 지연을 짧은 지연 여럿으로 분산시킬 수 있음
- idle-time collection(유휴 시간 수집): 가비지 컬렉터는 실행에 주는 영향을 최소화하기 위해 CPU가 유휴 상태일 때에만 가비지 컬렉션을 실행

## 메서드와 this

- 객체는 사용자, 주문 등과 같이 실제 존재하는 개체(entity)를 표현하고자 할 때 생성됨
- 자바스크립트에선 객체의 프로퍼티에 함수를 할당해 객체에게 행동할 수 있는 능력 부여

### 메서드 만들기

- 메서드(method): 객체 프로퍼티에 할당된 함수
- 이미 정의된 함수를 붙일 수도 있음

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

### 메서드 단축 구문

```javascript
user = {
  sayHi: function () {
    alert("Hello");
  }
};
user = {
  sayHi() {
    alert("Hello");
  }
};
```

### 메서드와 this

- 메서드 내부에서 `this` 키워드를 사용해 객체 내부의 정보에 접근
- 점 앞의 `this`는 메서드를 호출할 때 사용된 객체를 나타냄
- `this`를 사용하지 않고 외부 변수를 참조해 객체에 접근할 수 있으나 해당 변수가 덮어써지면 원치 않는 값을 참조해 에러가 발생할 수 있음

```javascript
let user = {
  name: "John",
  age: 30,
  sayHi() {
    alert(this.name);
  }
};
user.sayHi(); // John
```

### 자유로운 this

- 자바스크립트에서는 모든 함수에 `this`를 사용할 수 있음
- `this` 값은 런타임에 결정. 컨텍스트에 따라 달라짐
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

- 객체가 없어도 함수 호출 가능
- ```javascript
  function sayHi() {
    alert(this);
  }
  sayHi(); // undefined (엄격 모드)
  ```
- `this`가 `undefined`이기 때문에 `this.name`으로 접근하려고 하면 에러 발생
- 엄격 모드가 아닐 때는 `this`가 전역 객체를 참조. 브라우저에서는 `window` 전역 객체
- 전역 객체 참조는 대개 실수인 경우가 많음
- 자바스크립트에서 `this`는 런타임에 결정. 메서드가 어디에서 정의되었는지에 상관 없이, `this`는 점 앞의 객체가 무엇인가에 따라 자유롭게 결정

### this가 없는 화살표 함수

- 화살표 함수는 일반 함수와는 달리 고유한 `this`를 가지지 않음
- 화살표 함수에서 `this`를 참조하면, 화살표 함수가 아닌 평범함 외부 함수에서 `this` 값을 가져옴
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

## 참고

> [객체: 기본](https://ko.javascript.info/object-basics)
