---
title: 모던 JavaScript 튜토리얼 09 - 클래스 1
date: 2023-11-07 20:11:03 +0900
last_modified_at: 2024-04-10 14:50:28 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    class,
    class-field,
    inheritance,
    extends,
    overriding,
    super,
    homeobject
  ]
---

클래스와 기본 문법, 클래스 상속

## 클래스와 기본 문법

클래스

- 객체 지향 프로그래밍에서 특정 객체를 생성하기 위해 변수와 메서드를 정의하는 일종의 틀
- 객체를 정의하기 위한 상태(멤버 변수), 메서드(함수)로 구성

`new Function`, `class`

- 실무에서는 동일한 종류의 객체를 여러 개 생성하는 경우가 많음
- `new` 연산자와 생성자 함수로 객체 생성
- 모던 자바스크립트에 도입된 클래스(class) 문법으로 객체 생성
- 객체 지향 프로그래밍에서 사용되는 다양한 기능을 자바스크립트에서도 사용 가능

### 기본 문법

클래스 생성 기본 문법

- 메서드 사이에는 쉼표가 없음
- 클래스와 관련된 표기법은 객체 리터럴 표기법과 차이가 있음
- `constructor()`: 객체의 기본 상태를 설정하는 생성자 메서드
  - `new`에 의해 자동으로 호출되고 특별한 절차 없이 객체를 초기화

```javascript
class MyClass {
  constructor() {}
  method1() {}
  method2() {}
  ...
}
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
let user = new User("John"); // 새로운 객체 생성 -> constructor가 넘겨받은 인수와 함께 자동 실행됨
user.sayHi(); // John
```

### 클래스란

- 자바스크립트에서의 클래스는 새롭게 창안한 개체(entity)가 아닌, 함수의 한 종류

`class User {...}` 문법 구조가 하는 일

- 이름이 `User`인 함수를 생성하고, 생성자 메서드 `constructor`에서 함수 본문 가져옴
- 생성자 메서드가 없으면 본문이 비워진 채로 함수 생성
- `User.prototype`에 클래스 내에서 정의한 메서드 저장
- 객체의 메서드를 호출하면 prototype 프로퍼티를 통해 가져옴

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
alert(User === User.prototype.constructor); // true
alert(User.prototype.sayHi); // alert(this.name);
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

```
class User 선언 결과

User                                 User.prototype
[ constructor(name) { ]              [ sayHi: function   ]
[   this.name = name; ] -prototype-> [ constructor: User ]
[ }                   ]              [                   ]
```

### 클래스는 단순한 편의 문법이 아닙니다

- `class` 키워드 없이도 클래스 역할을 하는 함수 선언 가능
- 그렇기 때문에 클래스는 그저 편의 문법이라고 생각할 수 있지만, 클래스는 편의 문법이 아님
- 편의 문법(syntactic sugar): 기능은 동일하나 기존 문법을 쉽게 읽을 수 있게 만든 문법

`class User`와 동일한 기능을 하는 순수 함수

- 모든 함수의 프로토타입은 `constructor` 프로퍼티를 기본으로 가짐
- `constructor` 프로퍼티를 명시적으로 만들 필요 없음
- 순수 함수로 클래스 역할을 하는 함수를 선언하는 방법의 결과와
- `class` 키워드를 사용하는 방법의 결과는 거의 같음

`class`로 만든 함수

1. 특수 내부 프로퍼티 `[[IsClassConstructor]]: true`가 이름표처럼 붙음
   - 클래스 생성자를 `new`와 함께 호출하지 않으면 에러 발생
   - 클래스 생성자를 문자열로 형변환 시 `"class..."`로 시작하는 문자열이 됨
   - 이때 `[[IsClassConstructor]]: true`가 활용됨
2. 클래스에 정의된 메서드는 열거 불가(non-enumerable)
   - 클래스의 `prototype` 프로퍼티에 추가된 메서드의 `enumerable` 플래그는 `false`
   - `for..in`으로 객체 순회 시 메서드는 제외되어 유용
3. 항상 엄격 모드로 실행: 클래스 생성자 안 코드 전체에는 자동으로 엄격 모드 적용
4. 이외에도 `class`를 사용하면 다양한 기능이 따라옴

```javascript
// 1. 생성자 함수 만들기
function User(name) {
  this.name = name;
}
// 2. prototype에 메서드 추가
User.prototype.sayHi = function () {
  alert(this.name);
};
let user = new User("John");
user.sayHi();

class User {
  constructor() {}
}
alert(typeof User); // User의 타입은 함수이지만 그냥 호출할 수 없음
User(); // TypeError: Class constructor User cannot be invoked without 'new'
alert(User); // class User { ... }
```

### 클래스 표현식

- 클래스도 함수처럼 다른 표현식 내부에서 정의, 전달, 반환, 할당 가능
- 필요에 따라 동적으로 클래스 생성 가능
- 클래스 표현식에 붙은 이름: 오직 클래스 내부에서만 사용 가능
  - 기명 함수 표현식(Named Function Expression)과 유사
  - 기명 클래스 표현식(Named Class Expression)은 명세서에는 없는 용어

```javascript
let ClassExpression = class {
  sayHi() {}
};

function makeClass(phrase) {
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
let User = makeClass("안녕하세요."); // 동적으로 클래스 생성
new User().sayHi(); // 안녕하세요.

// 클래스 표현식에 이름 붙이기
let User = class MyClass {
  sayHi() {
    alert(MyClass); // 클래스 안에서만 이름 MyClass 사용 가능
  }
};
new User().sayHi();
alert(MyClass); // 에러
```

### getter와 setter

- 클래스도 getter, setter, 계산된 프로퍼티(computed property) 지원
- getter, setter는 해당 프로토타입에 정의됨

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  get name() {
    return this._name;
  }
  set name(value) {
    if (value.length < 4) {
      alert("이름이 너무 짧습니다.");
      return;
    }
    this._name = value;
  }
}
let user = new User("Alice");
alert(Object.getOwnPropertyNames(user)); // _name
alert(user.name); // Alice
user = new User(""); // 이름이 너무 짧습니다.
```

### 계산된 메서드 이름 `[...]`

- 계산된 메서드 이름(computed method name)
- 대괄호 `[...]`로 생성

```javascript
class User {
  ["say" + "Hi"]() {
    alert("Hello");
  }
}
new User.sayHi(); // Hello
```

### 클래스 필드

- 클래스 필드(class field): 클래스를 정의할 때 `<프로퍼티 이름> = <값>` 작성
- `User.prototype`이 아닌 개별 객체에만 클래스 필드가 설정됨
- 복잡한 표현식, 함수 호출 결과 등 어떠한 종류의 프로퍼티도 클래스에 추가 가능
- 클래스 필드는 최근에 추가된 기능

```javascript
class User {
  name = "John"; // 클래스 필드
  sayHi() {
    alert(`Hello, ${this.name}!`);
  }
}
new User().sayHi(); // Hello, John!
let user = new User();
alert(user.name); // John
alert(User.prototype.name); // undefined. 개별 객체에만 클래스 필드 설정됨

class User2 {
  name = prompt("이름을 입력하세요.", "John"); // 함수 호출 결과를 받는 클래스 필드
}
let user2 = new User();
alert(user2.name); // John
```

잃어버린 `this`(losing `this`)

- `this`의 컨텍스트를 알 수 없게 되는 문제. 자바스크립트에서 `this`는 동적으로 결정됨
- 객체 메서드를 전달해 전혀 다른 컨텍스트에서 호출할 때, `this`는 메서드가 정의된 객체를 참조하지 않음

```javascript
class Button {
  constructor(value) {
    this.value = value;
  }
  click() {
    alert(this.value);
  }
}
let button = new Button("안녕하세요");
alert(Object.getOwnPropertyNames(button)); // value
setTimeout(button.click, 1000); // undefined

// 클래스 필드로 바인딩된 메서드를 사용해 문제 해결하기
class Button2 {
  constructor(value) {
    this.value = value;
  }
  click = () => alert(this.value); // 클래스 필드
}
let button2 = new Button2("안녕하세요");
alert(Object.getOwnPropertyNames(button2)); // click,value
setTimeout(button2.click, 1000); // 안녕하세요
```

해결 방법

1. 래퍼 함수를 전달하기: `setTimeout(() => button.click(), 1000)`
2. 생성자 내부 등에서 메서드를 객체에 바인딩하기
3. 클래스 필드를 사용하기: `class Button { click = () => { ... } }`

클래스 필드로 `this` 문제 해결하기

- 클래스 필드 `click = () => { ... }`는 각 `Button` 객체마다 독립적인 함수를 생성
- 이 함수의 `this`를 해당 객체에 바인딩해 `button.click`을 아무 곳에나 전달 가능
  - `this`에는 항상 의도한 값이 들어감
- 클래스 필드의 이런 기능은 브라우저 환경에서 메서드를 이벤트 리스너로 설정해야 할 때 유용

단점

- 화살표 함수는 기본적으로 `prototype`을 생성하지 않음
  - `class명.prototype`으로 접근 불가
  - 테스트 케이스 작성 시 `spyOn(class명.prototype, method명)`으로 mocking 불가
- 프로토타입이 없으니 상속도 안 됨
- 가급적 화살표 함수를 class 내에서 사용하는 것을 지양하는 것이 더 좋을 수 있음
  - 특정 컨텍스트에 `this`를 바인딩 하려는 명확한 목적을 가지고 사용하는 경우는 제외

## 클래스 상속

클래스 상속

- 클래스를 다른 클래스로 확장 가능
- 기존에 존재하던 기능을 토대로 새로운 기능 생성

### extends 키워드

```javascript
class Child extends Parent {}
```

- 클래스 확장 문법 `extends`
- 키워드 `extends`는 프로토타입을 기반으로 동작
- `extends` 뒤에 표현식을 작성할 수 있어 조건에 따라 다른 클래스를 상속받고 싶을 때 유용
- `extends`는 `Child.prototype.[[Prototype]]`을 `Parent.prototype`으로 설정
- `Child.prototype`에서 메서드를 찾지 못하면 `Parent.prototype`에서 메서드 탐색
- `Child`의 객체는 `Child`의 메서드와 `Parent`의 메서드 접근 가능

```javascript
function f(phrase) {
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
class User extends f("Hello") {} // f("Hello")의 반환 값을 상속받음
new User.sayHi(); // Hello
```

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed = speed;
    alert(`${this.name}은/는 속도 ${this.speed}로 달립니다.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name}이/가 멈췄습니다.`);
  }
}
let animal = new Animal("동물");
```

```
Animal               Animal.prototype
[     ] -prototype-> [ constructor: Animal ]
                     [ run: function       ]
                     [ stop: function      ]
                                ^
                                | [[Prototype]]
                     new Animal |
                     [ name: "My animal"   ]
```

```javascript
class Rabbit extends Animal {
  hide() {
    alert(`${this.name}이/가 숨었습니다!`);
  }
}
let rabbit = new Rabbit("흰 토끼");
rabbit.run(5); // 흰 토끼은/는 속도 5로 달립니다.
rabbit.hide(); // 흰 토끼이/가 숨었습니다!
alert(rabbit.__proto__ === Rabbit.prototype); // true
alert(rabbit.__proto__.__proto__ === Animal.prototype); // true
alert(rabbit.__proto__.__proto__.__proto__ === Object.prototype); // true
```

```
Animal                       Animal.prototype
[             ] -prototype-> [ constructor: Animal  ]
                             [ run: function        ]
                             [ stop: function       ]
                                        ^
                                        | [[Prototype]] <- extends
Rabbit                       Rabbit.prototype
[ constructor ] -prototype-> [ constructor: Rabbit  ]
                                        ^
                                        | [[Prototype]]
                             new Rabbit
                             [ name: "White Rabbit" ]
```

엔진이 메서드 `rabbit.run`의 존재를 확인하는 절차

1. 객체 `rabbit`에서 `run` 탐색
2. `rabbit`의 프로토타입인 `Rabbit.prototype`에서 탐색
3. `Animal.prototype`에서 탐색하고 찾음
   - `extends`를 통해 관계가 만들어진 `Rabbit.prototype`의 프로토타입

자바스크립트의 내장 객체

- 프로토타입을 기반으로 상속 관계를 맺음
- `Date.prototype.[[Prototype]]`이 `Object.prototype`인 것처럼
- `Date` 객체에서 일반 객체 메서드를 사용할 수 있는 이유

### 메서드 오버라이딩

- 특별한 사항이 없다면 `class Child`는 `class Parent`에 있는 메서드를 그대로 상속받음
- 그런데 `Child`에서 `Parent`의 메서드를 자체적으로 정의하면, 상속받은 메서드가 아닌 자체 메서드가 사용됨
- 이럴 때 커스텀 메서드를 만들어 작업
  - 부모 메서드를 토대로 일부 기능만 변경하고 싶을 때
  - 부모 메서드의 기능을 확장하고 싶을 때
  - 이미 커스텀 메서드를 만들었더라도 이 과정 전·후에 부모 메서드를 호출하고 싶은 경우?
- `super` 키워드
  - `super.method(...)`: 부모 클래스에 정의된 메서드 `method`를 호출
  - `super(...)` 부모 생성자를 호출
  - 자식 생성자 내부에서만 사용 가능
  - 화살표 함수에는 `super`가 없음. `super`에 접근하면 `super`를 외부 함수에서 가져옴

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed = speed;
    alert(`${this.name}가 속도 ${this.speed}로 달립니다.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name}가 멈췄습니다.`);
  }
}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name}가 숨었습니다!`);
  }
  stop() {
    super.stop(); // 부모 클래스의 stop을 이용해 멈춤
    this.hide(); // 숨음
  }
}

let rabbit = new Rabbit("흰 토끼");
rabbit.run(5); // 흰 토끼가 속도 5로 달립니다.
rabbit.stop(); // 흰 토끼가 멈췄습니다. 흰 토끼가 숨었습니다!
```

```javascript
class Rabbit extends Animal {
  stop() {
    setTimeout(function () {
      super.stop(); // SyntaxError: 'super' keyword  unexpected  here
    }, 1000);
  }
}

class Rabbit2 extends Animal {
  stop() {
    setTimeout(() => super.stop(), 1000); // 화살표 함수의 super는 stop()의 super와 같음
  }
}
```

### 생성자 오버라이딩

- 클래스가 다른 클래스를 상속받는데 `constructor`가 없으면 비어있는 `constructor`가 생성됨
- 생성자는 기본적으로 부모 `constructor`를 호출하고, 이때 부모 `constructor`에도 인수를 모두 전달
- 클래스에 자체 생성자가 없는 경우에는 이런 일이 모두 자동으로 발생

```javascript
class Child extends Parent {
  // 자체 생성자가 없는 클래스를 상속받으면 자동으로 생성됨
  constructor(...args) {
    super(...args);
  }
}
```

상속 클래스의 생성자 함수

- 특수 내부 프로퍼티 `[[ConstructorKind]]: "derived"`가 이름표처럼 붙음
- 자바스크립트는 상속 클래스의 생성자 함수(derived constructor)와 그렇지 않은 생성자 함수를 구분

일반 클래스의 생성자 함수와 상속 클래스의 생성자 함수 간 차이

- 일반 클래스가 `new`와 함께 실행될 때
  - 빈 객체가 생성되고 `this`에 이 객체를 할당
- 상속 클래스의 생성자 함수가 실행될 때
  - 일반 클래스에서 일어난 일이 일어나지 않음
  - 빈 객체가 생성되고 부모 클래스의 생성자가 `this`에 이 객체를 할당해주는 것을 기대
- 상속 클래스의 생성자에서는 반드시 `super`를 호출해 부모 생성자를 실행해 주어야 함
  - 그렇지 않으면 `this`가 될 객체가 만들어지지 않아 에러 발생
- `super(...)`는 `this`를 사용하기 전에 반드시 호출해야 함

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
}

class Rabbit extends Animal {
  // 커스텀 생성자 추가
  constructor(name, earLength) {
    // super(name); // this를 사용하기 전에 super()를 호출하면 Rabbit의 생성자가 제대로 동작
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
}

let rabbit = new Rabbit("흰 토끼", 10); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

클래스 필드 오버라이딩

- 오버라이딩은 메서드뿐만 아니라 클래스 필드를 대상으로도 적용 가능
- 부모 클래스의 생성자 안에 있는 오버라이딩한 필드에 접근하려고 할 때,
- 자바스크립트는 다른 프로그래밍 언어와는 다르게 조금 까다로움

```javascript
// 1. 클래스 필드 오버라이딩: 생각한 대로 동작하지 않음
class Animal {
  name = "animal";
  constructor() {
    alert(this.name); // (*)
  }
}
class Rabbit extends Animal {
  name = "rabbit"; // 오버라이딩
}
new Animal(); // animal
new Rabbit(); // animal. 생각한 대로 출력되지 않음

// 2. 메서드 오버라이딩: 생각한 대로 동작
class Animal2 {
  showName() {
    alert("animal");
  }
  constructor() {
    this.showName(); // alert(this.name); 대신 메서드 호출
  }
}
class Rabbit2 extends Animal2 {
  showName() {
    alert("rabbit");
  }
}
new Animal2(); // animal
new Rabbit2(); // rabbit. 생각한 대로 출력됨
```

- `Rabbit`에서 `name` 필드를 오버라이딩
- `Rabbit`을 사용해 인스턴스를 만들면 `Animal`의 생성자가 호출됨
  - `Rabbit`에는 따로 생성자가 정의되어 있지 않기 때문
- `new Animal()`과 `new Rabbit()`을 실행할 때 모두 `(*)`이 실행되어 `animal` 출력
- 부모 생성자는 자식 클래스에서 오버라이딩한 값이 아닌, 부모 클래스 안의 필드 값을 사용
- 메서드 오버라이딩(2번)은 생각한 대로 출력됨
- 상속을 받고, 필드 값을 오버라이딩 했는데 새로운 값 대신 부모 클래스 안에 있는 기존 필드 값을 사용하는 이유?
- 클래스 필드 오버라이딩과 메서드 오버라이딩 동작이 다른 이유는 필드 초기화 순서 때문
- 클래스 필드는 다음 규칙에 따라 초기화 순서가 달라짐
  - 아무것도 상속받지 않는 베이스 클래스의 경우: 생성자 실행 이전에 초기화
  - 부모 클래스가 있는 경우: `super()` 실행 직후에 초기화

오버라이딩한 클래스 필드 값이 사용되지 않은 이유

- `Rabbit`은 하위 클래스이고 `constructor()`가 정의되어 있지 않음
- 이런 경우 생성자는 비어있는데, 그 안에 `super(...args)`만 있는 것과 동일
- `new Rabbit()` 실행 -> `super()` 호출 -> 부모 생성자 실행
- 하위 클래스 필드 초기화 순서에 의해 `Rabbit`의 필드는 `super()` 실행 후에 초기화
- 부모 생성자가 실행될 때 `Rabbit`의 필드는 아직 존재하지 않음
- 그래서 필드를 오버라이딩했을 때 `Animal`에 있는 필드가 사용된 것
- 이렇게 자바스크립트는 오버라이딩 시 필드와 메서드의 동작 방식이 미묘하게 다름
  - 이런 문제는 오버라이딩한 필드를 부모 생성자에서 사용할 때만 발생
  - 개발 시 필드 오버라이딩이 문제가 되는 상황이 발생할 때,
  - 필드 대신 메서드를 사용. getter나 setter를 사용

```javascript
class Parent {
  x = console.log("from Parent:", this); // 1, 3
  constructor() {
    console.log(this.x, "\n"); // 2, 4
  }
}
class Child extends Parent {
  x = console.log("from Child:", this); // 5
}
new Parent();
new Child();
```

```
from Parent: Parent {}
undefined

from Parent: Child {}
undefined

from Child: Child { x: undefined }
```

### super 키워드와 `[[HomeObject]]`

- `super`에 대해 좀 더 깊이 파고들기
- 지금까지 배운 내용만으로는 `super`가 제대로 동작하지 않음

내부에서 `super`의 동작 방식

- 객체 메서드가 실행되면, 현재 객체가 `this`가 됨
- 이 상태에서 `super.method()`를 호출하면 엔진은 현재 객체의 프로토타입에서 `method`를 찾아야 함
- 이런 과정들은 어떻게 일어나나?
- 엔진은 현재 객체 `this`를 알고 있음
- 그래서 `this.__proto__.method`를 통해 부모 객체의 `method`를 찾을 수 있을 것 같지만 아니

```javascript
let animal = {
  name: "동물",
  eat() {
    alert(`${this.name} 이/가 먹이를 먹습니다.`);
  }
};

let rabbit = {
  __proto__: animal,
  name: "토끼",
  eat() {
    this.__proto__.eat.call(this); // (*)
  }
};

rabbit.eat(); // 토끼 이/가 먹이를 먹습니다.
```

- `rabbit.__proto__`는 `animal`
- `(*)`에서 `eat`을 프로토타입 `animal`에서 가져옴
- 그리고 현재 컨텍스트에 기반해 `eat.call(this)` 호출
- 예상한 내용 `토끼 이/가 먹이를 먹습니다.`가 출력됨
- `this.__proto__.eat()`이면 현재 객체가 아닌 프로토타입의 컨텍스트에서 부모 `eat`을 실행
- `동물 이/가 먹이를 먹습니다.`가 출력되기 때문에 `.call(this)`을 사용해야 함
- 그러나 체인에 객체를 하나 더 추가하면 문제가 발생하기 시작

```javascript
let animal = {
  name: "동물",
  eat() {
    alert(`${this.name} 이/가 먹이를 먹습니다.`);
  }
};

let rabbit = {
  __proto__: animal,
  eat() {
    this.__proto__.eat.call(this); // (*)
  }
};

let longEar = {
  __proto__: rabbit,
  eat() {
    this.__proto__.eat.call(this); // (**)
  }
};

longEar.eat(); // RangeError: Maximum call stack size exceeded
```

- `longEar.eat()`이 호출될 때 어떤 일이 발생하는지 하나씩 보면 에러가 발생한 이유를 알 수 있음
- `(*)`과 `(**)`에서 `this`는 현재 객체인 `longEar`가 됨
- 모든 객체 메서드는 프로토타입 등이 아닌 현재 객체를 `this`로 가짐
- 따라서 `(*)`과 `(**)`의 `this.__proto__`에는 정확히 같은 값 `rabbit`이 할당됨
- 체인 위로 올라가지 않고 양쪽 모두에서 `rabbit.eat`을 호출하기 때문에 무한 루프에 빠짐

1. `longEar.eat()` 내부의 `(**)`에서 `rabbit.eat`을 호출하는데, 이때 `this`는 `longEar`

- ```javascript
  this.__proto__.eat.call(this); // (**). longEar.eat() 안의 this=longEar
  longEar.__proto__.eat.call(this); // 윗줄과 같음. longEar.__proto__=rabbit
  rabbit.eat.call(this); // 윗줄과 같음
  ```

2. `rabbit.eat` 내부의 `(*)`에서 체인 위쪽에 있는 호출을 전달하려 했으나 `this`가 `longEar`이기 때문에 또다시 `rabbit.eat` 호출

- ```javascript
  this.__proto__.eat.call(this); // (*). 역시 rabbit.eat() 안의 this=longEar
  longEar.__proto__.eat.call(this); // 윗줄과 같음. longEar.__proto__=rabbit
  rabbit.eat.call(this); // 윗줄과 같음
  ```

3. 이런 내부 동작 때문에 `rabbit.eat`은 체인 위로 올라가지 못하고 자기 자신을 계속 호출해 무한 루프에 빠짐

`[[HomeObject]]`

- 위 문제는 `this`만으로는 해결 불가
- 이런 문제를 해결할 수 있는 자바스크립트의 함수 전용 특수 내부 프로퍼티 `[[HomeObject]]`
- 클래스이거나 객체 메서드인 함수의 `[[HomeObject]]` 프로퍼티는 해당 객체가 저장됨
- `super`는 `[[HomeObject]]`를 이용해 부모 프로토타입과 메서드를 찾음
- `longEar.eat` 같은 객체 메서드는 `[[HomeObject]]`를 알고 있음
- 그래서 `this` 없이도 프로토타입으로부터 부모 메서드를 가져올 수 있음

```javascript
let animal = {
  name: "동물",
  eat() {
    alert(`${this.name} 이/가 먹이를 먹습니다.`); // animal.eat.[[HomeObject]] == animal
  }
};

let rabbit = {
  __proto__: animal,
  name: "토끼",
  eat() {
    super.eat(); // rabbit.eat.[[HomeObject]] == rabbit
  }
};

let longEar = {
  __proto__: rabbit,
  name: "귀가 긴 토끼",
  eat() {
    super.eat(); // longEar.eat.[[HomeObject]] == longEar
  }
};

longEar.eat(); // 귀가 긴 토끼 이/가 먹이를 먹습니다.
```

메서드는 자유롭지 않음

- 자바스크립트에서 함수는 대개 객체에 묶이지 않고 자유로움
  - `this`가 달라도 객체 간 메서드를 복사하는 것이 가능
- 그런데 `[[HomeObject]]`는 그 존재만으로도 함수의 자유도를 파괴함. 메서드가 객체를 기억하기 때문
- 개발자가 `[[HomeObject]]`를 변경할 방법은 없음. 한번 바인딩 된 함수는 더이상 변경되지 않음
- 다행히 `[[HomeObject]]`는 오직 `super` 내부에서만 유효
- 그렇기 때문에 `super`를 사용하지 않는 경우에는 메서드의 자유성이 보장됨. 객체간 복사도 가능
  - 메서드에서 `super`를 사용하면 이야기가 달라짐

```javascript
let animal = {
  sayHi() {
    alert(`나는 동물입니다.`);
  }
};

let rabbit = {
  __proto__: animal,
  sayHi() {
    super.sayHi();
  }
};

let plant = {
  sayHi() {
    alert("나는 식물입니다.");
  }
};

let tree = {
  __proto__: plant,
  sayHi: rabbit.sayHi // (*)
};

tree.sayHi(); // 나는 동물입니다.
```

- 객체 간 메서드를 잘못 복사해 `super`가 제대로 동작하지 않음
- `(*)`에서 메서드 `tree.sayHi`는 중복 코드를 방지하기 위해 `rabbit`에서 메서드를 복사해옴
- 그런데 복사해온 메서드는 `rabbit`에서 생성됨
  - 이 메서드의 `[[HomeObject]]`는 `rabbit`이고 개발자는 `[[HomeObject]]` 변경 불가
- `tree.sayHi()`의 코드 내부에는 `super.sayHi()`가 있음
- `rabbit`의 프로토타입은 `animal`이므로 `super`는 체인 위에 있는 `animal`로 올라가 `sayHi`를 찾음

```
animal                      plant
[       ]                   [ sayHi ]
    ^                           ^
    |                           |
rabbit                      tree
[ sayHi ] <-[[HomeObject]]- [ sayHi ]
```

함수 프로퍼티가 아닌 메서드 사용하기

- `[[HomeObject]]`는 클래스와 일반 객체의 메서드에서 정의됨
- 그런데 객체 메서드의 경우, 메서드를 반드시 `method()` 형태로 정의해야 `[[HomeObject]]`가 제대로 동작
  - `method: function(){}` 형태로 정의하면 안 됨
- 메서드 문법이 아닌(non-method syntax) 함수 프로퍼티를 사용하는 경우,
- `[[HomeObject]]` 프로퍼티가 설정되지 않기 때문에 상속이 제대로 되지 않음
- 자바스크립트에게 두 방법의 차이는 중요
- 화살표 함수는 `this`나 `super`를 갖지 않아 주변 컨텍스트에 잘 들어맞음

```javascript
// 메서드 문법이 아닌 함수 프로퍼티 작성
let animal = {
  eat: function () {
    // eat() {...} 대신 eat: function() {...}을 사용
  }
};

let rabbit = {
  __proto__: animal,
  eat: function () {
    super.eat();
  }
};

rabbit.eat(); // SyntaxError: 'super' keyword unexpected here. [[HomeObject]]가 없어서 에러 발생
```

### 예제

인스턴스 생성 오류

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Rabbit extends Animal {
  constructor(name) {
    // super(name);
    this.name = name;
    this.created = Date.now();
  }
}

let rabbit = new Rabbit("White Rabbit"); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
alert(rabbit.name);
```

- 자식 클래스의 생성자에서 `super()`를 호출하지 않아 에러 발생

## 참고

> [클래스](https://ko.javascript.info/classes)
