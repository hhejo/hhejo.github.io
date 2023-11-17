---
title: 모던 JavaScript 튜토리얼 09 - 클래스 1
date: 2023-11-07 20:11:03 +0900
last_modified_at: 2023-11-17 10:54:44 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

클래스와 기본 문법, 클래스 상속

## 클래스와 기본 문법

클래스

- 객체 지향 프로그래밍에서 특정 객체를 생성하기 위해 변수와 메서드를 정의하논 일종의 틀
- 객체를 정의하기 위한 상태(멤버 변수)와 메서드(함수)로 구성됨

실무에서는 사용자나 물건 같이 동일한 종류의 객체를 여러 개 생성하는 경우가 많음

- `new function`(`new` 연산자와 생성자 함수)을 사용할 수 있음
- 모던 자바스크립트에 도입된 클래스(class) 문법을 사용해 객체 지향 프로그래밍에서 사용되는 다양한 기능을 자바스크립트에서도 사용 가능

### 기본 문법

클래스 생성 기본 문법

```javascript
class MyClass {
  constructor() {}
  method1() {}
  method2() {}
  ...
}
```

- `new MyClass()`를 호출하면 내부에서 정의한 메서드가 들어 있는 객체가 생성됨

`constructor()`

- 객체의 기본 상태를 설정해주는 생성자 메서드
- `new`에 의해 자동으로 호출됨
- 특별한 절차 없이 객체를 초기화할 수 있음

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}

let user = new User("John");
user.sayHi();
```

`new User("John")`을 호출할 때 일어나는 일

- 새로운 객체가 생성됨
- 넘겨받은 인수와 함께 `constructor`가 자동으로 실행됨
  - 이때 인수 `"John"`이 `this.name`에 할당됨

메서드 사이에는 쉼표가 없음

- 클래스와 관련된 표기법은 객체 리터럴 표기법과 차이가 있음

### 클래스란

클래스는 자바스크립트에서 새롭게 창안한 개체(entity)가 아님

자바스크립트에서 클래스는 함수의 한 종류

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}
alert(typeof User); // function
```

`class User {...}` 문법 구조가 하는 일

- `User`라는 이름을 가진 함수를 만듦
  - 함수 본문은 생성자 메서드 `constructor`에서 가져옴
  - 생성자 메서드가 없으면 본문이 비워진 채로 함수가 만들어짐
- `sayHi` 같은 클래스 내에서 정의한 메서드를 `User.prototype`에 저장함

`new User`를 호출해 객체를 만들고, 객체의 메서드를 호출하면 메서드를 prototype 프로퍼티를 통해 가져옴

`class User` 선언 결과는 아래 그림이 됨

```
User                                 User.prototype
[ constructor(name) { ]              [ sayHi: function   ]
[   this.name = name; ] -prototype-> [ constructor: User ]
[ }                   ]              [                   ]
```

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}
alert(typeof User); // function
```

## 클래스 상속

## 참고

> [클래스](https://ko.javascript.info/classes)
