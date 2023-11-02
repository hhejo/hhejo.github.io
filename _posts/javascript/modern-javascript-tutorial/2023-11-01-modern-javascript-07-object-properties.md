---
title: 모던 JavaScript 튜토리얼 07 - 객체 프로퍼티 설정
date: 2023-11-01 21:26:07 +0900
last_modified_at: 2023-11-02 07:45:23 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

프로퍼티 플래그와 설명자, 프로퍼티 getter와 setter

## 프로퍼티 플래그와 설명자

객체 프로퍼티 추가 구성 옵션에 대해 알아보기

### 프로퍼티 플래그

객체 프로퍼티는 값(value)과 함께 플래그(flag)라 불리는 특별한 속성 세 가지를 가짐

- `writable`
  - `true`: 값 수정 가능
  - `false`: 읽기만 가능
- `enumerable`
  - `true`: 반복문으로 나열 가능
  - `false`: 반복문으로 나열 불가
- `configurable`
  - `true`: 프로퍼티 삭제, 플래그 수정 가능
  - `false`: 프로퍼티 삭제, 플래그 수정 불가

지금까지 해왔던 평범한 방식으로 프로퍼티를 생성하면, 프로퍼티 플래그는 모두 `true`가 됨

- `true`로 설정된 플래그는 언제든지 수정 가능

특정 프로퍼티에 대한 정보를 모두 얻기

```javascript
let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName);
```

- `obj`: 정보를 얻고자 하는 객체
- `propertyName`: 정보를 얻고자 하는 객체 내 프로퍼티
- 프로퍼티 설명자(descriptor) 객체 반환
  - 프로퍼티 값, 세 플래그에 대한 정보 가짐

```javascript
let user = { name: "John" };
let descriptor = Object.getOwnPropertyDescriptor(user, "name");

alert(JSON.stringify(descriptor, null, 2));
/*
{
  "value": "John",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```

플래그 변경하기

```javascript
Object.defineProperty(obj, propertyName, descriptor);
```

- `obj`, `propertyName`: 설명자를 적용하고 싶은 객체와 객체 프로퍼티
- `descriptor`: 적용하고자 하는 프로퍼티 설명자
- 객체에 해당 프로퍼티가 있으면 플래그를 원하는대로 변경해줌
- 프로퍼티가 없으면 인수로 넘겨받은 정보를 이용해 새로운 프로퍼티 생성
  - 이때 플래그 정보가 없으면 플래그 값은 자동으로 `false`

```javascript
let user = {};
Object.defineProperty(user, "name", { value: "John" });
let descriptor = Object.getOwnPropertyDescriptor(user, "name");
alert(JSON.stringify(descriptor, null, 2));
/*
}
  "value": "John",
  "writable": false,
  "enumerable": false,
  "configurable": false
}
*/
```

평범한 방식으로 객체 프로퍼티를 만들었을 때와 `defineProperty`를 이용해 프로퍼티를 만들었을 때의 가장 큰 차이점은 플래그에 있음

- `defineProperty`를 사용해 프로퍼티를 만든 경우
  - `descriptor`에 플래그 값을 명시하지 않으면 플래그 값이 자동으로 `false`가 됨
  - 플래그 값을 `true`로 설정하려면 `descriptor`에 `true`라고 명시해야 함

### writable 플래그

`writable` 플래그로 특정 프로퍼티에 값을 쓰지 못하도록(non-writable) 할 수 있음

```javascript
"use strict";
let user = { name: "John" };
Object.defineProperty(user, "name", { writable: false });
console.log(user); // { name: "John" }
console.log(Object.getOwnPropertyDescriptor(user, "name"));
/*
{
  value: "John",
  writable: false,
  enumerable: true,
  configurable: true
}
*/
user.name = "test"; // TypeError: Cannot assign to read only property 'name' of object '#<Object>'
```

- 엄격모드에서만 에러 발생
- 비 엄격모드에서는 에러가 발생하지 않지만 값을 변경하는 것도 불가능
  - 플래그에서 정한 규칙을 위반하는 행위는 에러 없이 그냥 무시됨

아래 예시는 위 예시와 동일하게 동작

- `defineProperty` 메서드로 프로퍼티를 만듦

```javascript
"use strict";
let user = {};
Object.defineProperty(user, "name", {
  value: "John",
  // writable: true,
  enumerable: true,
  configurable: true
});
user.name = "Hello"; // TypeError: Cannot assign to read only property 'name' of object '#<Object>'
```

### enumerable 플래그

객체 내장 메서드 `toString`은 열거가 불가능(non-enumerable)하기 때문에 `for..in` 사용 시 나타나지 않음

- 커스텀 `toString`을 추가하면 `for..in`에 나타남

```javascript
let user = {
  name: "John",
  toString() {
    return this.name;
  }
};
for (let key in user) alert(key); // name, toString
```

특정 프로퍼티의 `enumerable` 플래그 값을 `false`로 설정하면 `for..in` 반복문에 나타나지 않음

```javascript
let user = {
  name: "John",
  toString() {
    return this.name;
  }
};
Object.defineProperty(user, "toString", { enumerable: false });
for (let key in user) alert(key); // name
alert(Object.keys(user)); // name
```

- 열거 불가능한 프로퍼티는 `Object.keys`에도 배제됨

### configurable 플래그

구성 가능하지 않음을 나타내는 플래그(non-configurable flag)인 `configurable: false`는 몇몇 내장 객체나 프로퍼티에 기본으로 설정됨

어떤 프로퍼티의 `configurable` 플래그가 `false`로 설정되어 있다면, 해당 프로퍼티는 객체에서 지울 수 없음

```javascript
"use struct";
let descriptor = Object.getOwnPropertyDescriptor(Math, "PI");
alert(JSON.stringify(descriptor, null, 2));
/*
{
  "value": 3.141592653589793,
  "writable": false,
  "enumerable": false,
  "configurable": false
}
*/
Math.PI = 3; // TypeError: Cannot assign to read only property 'PI' of object '#<Object>'
```

`configurable` 플래그를 `false`로 설정하면 돌이킬 수 없음

- `defineProperty`를 써도 값을 `true`로 되돌릴 수 없음

`configurable: false`가 생성하는 제약 사항

- `configurable` 플래그 수정 불가
- `enumerable` 플래그 수정 불가
- `writable: false`의 값을 `true`로 변경 불가(`true`를 `false`로 변경하는 것은 가능)
- 접근자 프로퍼티 `get/set`을 변경할 수 없음(새로 만드는 것은 가능)

```javascript
let user = {};
Object.defineProperty(user, "name", {
  value: "John",
  writable: false,
  configurable: false
});
user.name = "Pete"; // Error
delete user.name; // Error
Obejct.defineProperty(user, "name", { value: "Pete" }); // Error
Object.defineProperty(user, "name", { writable: true }); // Error
```

non-configurable과 non-writable은 다름

- `configurable` 플래그가 `false`여도 `writable` 플래그가 `true`이면 프로퍼티 값 변경 가능
- `configurable: false`는 플래그 값 변경이나 프로퍼티 삭제를 위해 만들어짐
  - 프로퍼티 값 변경을 막기 위해 만들어진 것이 아님

### Object.defineProperties

프로퍼티 여러 개를 한번에 정의 가능

```javascript
Object.defineProperties(obj, {
  prop1: descriptor1,
  prop2: descriptor2,
  ...
});
```

```javascript
Object.defineProperties(user, {
  name: { value: "John", writable: false },
  surname: { value: "Smith", writable: false }
});
```

### Object.getOwnPropertyDescriptors

프로퍼티 설명자를 전부 한꺼번에 가져오기 가능

- `Object.defineProperties`와 함께 사용하면 객체 복사 시 플래그도 함께 복사 가능

```javascript
let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));
```

```javascript
for (let key in user) clone[key] = user[key];
```

- 위 방법은 플래그는 복사하지 않음
- 플래그 정보도 복사하려면 `Object.defineProperties`를 사용

`for..in`을 사용해 객체를 복사하면 심볼형 프로퍼티도 배제됨

- `Object.getOwnPropertyDescriptors`는 심볼형 프로퍼티를 포함한 프로퍼티 설명자 전체를 반환

### 객체 수정을 막아주는 다양한 메서드

프로퍼티 설명자는 특정 프로퍼티 하나를 대상으로 함

한 객체 내 프로퍼티 전체를 대상으로 하는 제약 사항을 만드는 메서드들

`Object.preventExtensions(obj)`

- 객체에 새로운 프로퍼티를 추가할 수 없게 함

`Object.seal(obj)`

- 새로운 프로퍼티 추가나 기존 프로퍼티 삭제를 막음
- 프로퍼티 전체에 `configurable: false`를 설정하는 것과 동일한 효과

`Object.freeze(obj)`

- 새로운 프로퍼티 추가나 기존 프로퍼티 삭제, 수정을 막음
- 프로퍼티 전체에 `configurable: false`, `writable: false`를 설정하는 것과 동일한 효과

`Object.isExtensible(obj)`

- 새로운 프로퍼티를 추가하는 게 불가능한 경우 `false`, 그렇지 않은 경우 `true` 반환

`Object.isSealed(obj)`

- 프로퍼티 추가, 삭제가 불가능하고 모든 프로퍼티가 `configurable: false`이면 `true` 반환

`Object.isFrozen(obj)`

- 프로퍼티 추가, 삭제, 변경이 불가능하고 모든 프로퍼티가 `configurable: false`, `writable: false`이면 `true` 반환

## 프로퍼티 getter와 setter

객체의 프로퍼티는 두 종류로 나뉨

1. 데이터 프로퍼티(data property)

- 지금까지 사용한 모든 프로퍼티

2. 접근자 프로퍼티(accessor property)
   - 본질은 함수
   - 값을 획득(get)하고 설정(set)하는 역할
   - 외부 코드에서는 함수가 아닌 일반적인 프로퍼티로 보임

### getter와 setter

접근자 프로퍼티는 getter(획득자), setter(설정자) 메서드로 표현됨

객체 리터럴 안에서 getter와 setter 메서드는 `get`과 `set`으로 나타냄

```javascript
let obj = {
  get propName() {
    // getter, obj.propName을 실행할 때 실행되는 코드
  },
  set propName(value) {
    // setter, obj.propName = value를 실행할 때 실행되는 코드
  }
};
```

getter 메서드

- `obj.propName`을 이용해 프로퍼티를 읽으려 할 때 실행

setter 메서드

- `obj.propName = value`로 프로퍼티에 값을 할당하려 할 때 실행

```javascript
let user = {
  name: "John",
  surname: "Smith",
  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};
alert(user.fullName); // John Smith
```

바깥 코드에서 접근자 프로퍼티를 일반 프로퍼티처럼 사용 가능

접근자 프로퍼티를 사용하면 함수처럼 호출하지 않고, 일반 프로퍼티에서 값에 접근하는 것처럼 평범하게 프로퍼티 값을 얻을 수 있음

- 나머지 작업은 getter 메서드가 뒷단에서 처리함

```javascript
"use strict";
user.fullName = "Test"; // TypeError: Cannot set property fullName of #<Object> which has only a getter
```

- 엄격모드가 아니면 에러가 발생하지 않고 무시됨

```javascript
// "use strict";
let user = {
  name: "John",
  surname: "Smith",
  get fullName() {
    return `${this.name} ${this.surname}`;
  },
  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }
};
user.fullName = "Alice Cooper";
alert(user.name); // Alice
alert(user.surname); // Cooper
```

- 객체에 `fullName`이라는 가상의 프로퍼티가 생김
- 읽고 쓸 수는 있지만 실제로는 존재하지 않음

### 접근자 프로퍼티 설명자

데이터 프로퍼티의 설명자와 접근자 프로퍼티의 설명자는 다름

접근자 프로퍼티에는 설명자 `value`, `writable`이 없는 대신 `get`, `set` 함수가 있음

접근자 프로퍼티는 다음 설명자를 가짐

- `get`: 인수가 없는 함수, 프로퍼티를 읽을 때 동작
- `set`: 인수가 하나인 함수, 프로퍼티에 값을 쓸 때 호출
- `enumerable`: 데이터 프로퍼티와 동일
- `configurable`: 데이터 프로퍼티와 동일

`defineProperty`에 설명자 `get`, `set`을 전달해 `fullName`을 위한 접근자 생성 가능

```javascript
let uesr = { name: "John", surname: "Smith" };
Object.defineProperty(user, "fullName", {
  get() {
    return `${this.name} ${this.surname}`;
  },
  set(value) {
    [this.name, this.surname] = value.split(" ");
  }
});
alert(user.fullName); // John Smith
for (let key in user) alert(key); // name, surname
```

프로퍼티는 접근자 프로퍼티(`get`, `set` 메서드를 가짐)나 데이터 프로퍼티(`value`를 가짐) 중 한 종류에만 속함

- 둘 다에 속할 수 없음

한 프로퍼티에 `get`과 `value`를 동시에 설정하면 에러 발생

```javascript
Object.defineProperty({}, "prop", {
  get() {
    return 1;
  },
  value: 2
}); // TypeError: Invalid property descriptor. Cannot both specify accessors and a value or writable attribute, #<Object>
```

### getter와 setter 똑똑하게 활용하기

getter와 setter를 실제 프로퍼티 값을 감싸는 래퍼(wrapper)처럼 사용하면 프로퍼티 값을 원하는 대로 통제 가능

```javascript
let user = {
  get name() {
    return this._name;
  },
  set name(value) {
    if (value.length < 4) {
      alert("입력값이 너무 짧습니다. 네 글자 이상을 입력하세요.");
      return;
    }
    this._name = value;
  }
};
user.name = "Pete";
alert(user.name); // Pete
user.name = ""; // 너무 짧은 이름을 할당
```

- 실제 값은 별도의 프로퍼티 `_name`에 저장
- 기술적으로는 외부 코드에서 `user._name`을 사용해 바로 접근할 수 있음
- 그러나 밑줄 `_`로 시작하는 프로퍼티는 객체 내부에서만 활용하고, 외부에서는 건드리지 않는 것이 관습

### 호환성을 위해 사용하기

접근자 프로퍼티는 언제 어느 때나 getter와 setter를 사용해 데이터 프로퍼티의 행동과 값을 원하는 대로 조정할 수 있게 해줘서 유용

```javascript
function User(name, age) {
  this.name = name;
  this.age = age; //
}
let john = new User("John", 25);
```

```javascript
function User(name, birthday) {
  this.name = name;
  this.birthday = birthday; //
}
let john = new User("John", new Date(1992, 6, 1));
```

- `age` 대신 `birthday`를 저장해야 해서 코드를 수정함
- 생성자 함수를 수정하면 기존 코드를 다 수정해야 함
- 특정 프로퍼티를 위한 getter를 추가해서 해결할 수 있음

```javascript
function User(name, birthday) {
  this.name = name;
  this.birthday = birthday;
  Object.defineProperty(this, "age", {
    get() {
      let todayYear = new Date().getFullYear();
      return todayYear - this.birthday.getFullYear();
    }
  });
}
let john = new User("John", new Date(1992, 6, 1));
alert(john.birthday); // birthday 사용 가능
alert(john.age); // age 역시 사용 가능
```

## 참고

> [객체 프로퍼티 설정](https://ko.javascript.info/object-properties)
