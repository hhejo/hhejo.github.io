---
title: 모던 JavaScript 튜토리얼 09 - 클래스 1
date: 2023-11-07 20:11:03 +0900
last_modified_at: 2023-11-18 10:54:44 +0900
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
alert(User === User.prototype.constructor); // true
alert(User.prototype.sayHi); // alert(this.name);
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

### 클래스는 단순한 편의 문법이 아닙니다

어떤 사람들은 `class`라는 키워드 없이도 클래스 역할을 하는 함수를 선언할 수 있기 때문에 클래스는 편의 문법에 불과하다고 얘기함

편의 문법(syntactic sugar, 문법 설탕)

- 기능은 동일하나 기존 문법을 쉽게 읽을 수 있게 만든 문법

```javascript
// class User와 동일한 기능을 하는 순수 함수를 만들어보기
// 1. 생성자 함수 만들기
function User(name) {
  this.name = name;
}
// 모든 함수의 프로토타입은 constructor 프로퍼티를 기본으로 갖고 있기 때문에
// constructor 프로퍼티를 명시적으로 만들 필요가 없음
// 2. prototype에 메서드를 추가
User.prototype.sayHi = function () {
  alert(this.name);
};

let user = new User("John");
user.sayHi();
```

순수 함수로 클래스 역할을 하는 함수를 선언하는 방법과 `class` 키워드를 사용하는 방법의 결과는 거의 같음

두 방법의 중요한 차이

1. `class`로 만든 함수에는 특수 내부 프로퍼티인 `[[IsClassConstructor]]: true`가 이름표처럼 붙음
   - 자바스크립트는 다양한 경우에 `[[IsClassConstructor]]: true`를 활용
   - 클래스 생성자를 `new`와 함께 호출하지 않으면 에러가 발생하고 이때 `[[IsClassConstructor]]: true`가 사용됨
2. 클래스에 정의된 메서드는 열거할 수 없음(non-enumerable)
   - 클래스의 `prototype` 프로퍼티에 추가된 메서드의 `enumerable` 플래그는 `false`
   - `for..in`으로 객체를 순회할 때 메서드는 순회 대상에서 제외하고자 하는 경우가 많으므로 이 특징은 유용함
3. 클래스는 항상 엄격 모드로 실행됨(`use strict`)
   - 클래스 생성자 안 코드 전체에는 자동으로 엄격 모드가 적용됨
4. 이외에도 `class`를 사용하면 다양한 기능이 따라옴

```javascript
class User {
  constructor() {}
}
alert(typeof User); // User의 타입은 함수이지만 그냥 호출할 수 없음
User(); // TypeError: Class constructor User cannot be invoked without 'new'
alert(User); // class User { ... }
```

- 클래스 생성자를 문자열로 형변환하면 "class..."로 시작하는 문자열이 되는데 이때도 `[[IsClassConstructor]]: true`가 사용됨

### 클래스 표현식

함수처럼 클래스도 다른 표현식 내부에서 정의, 전달, 반환, 할당할 수 있음

클래스 표현식

```javascript
let User = class {
  sayHi() {
    alert("안녕하세요.");
  }
};
```

기명 함수 표현식(Named Function Expression)과 유사하게 클래스 표현식에서 이름을 붙일 수 있음

- 클래스 표현식에 이름을 붙이면 이 이름은 오직 클래스 내부에서만 사용 가능

```javascript
// 기명 클래스 표현식(Named Class Expression)
// 명세서에는 없는 용어이지만 기명 함수 표현식과 유사하게 동작
let User = class MyClass {
  sayHi() {
    alert(MyClass); // 클래스 안에서만 MyClass라는 이름 사용 가능
  }
};
new User().sayHi();
alert(MyClass); // 에러
```

필요에 따라 클래스를 동적으로 생성할 수 있음

```javascript
function makeClass(phrase) {
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
let User = makeClass("안녕하세요.");
new User().sayHi();
```

### getter와 setter

클래스도 getter, setter, 계산된 프로퍼티(computed property)를 지원함

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
let user = new User("보라");
alert(user.name);
user = new User(""); // 이름이 너무 짧습니다.
```

- getter, setter는 `User.prototype`에 정의됨

### 계산된 메서드 이름 `[...]`

대괄호 `[...]`를 이용해 계산된 메서드 이름(computed method name)을 만들기

```javascript
class User {
  ["say" + "Hi"]() {
    alert("Hello");
  }
}
new User.sayHi();
```

### 클래스 필드

클래스 필드(class field)라는 문법으로 어떤 종류의 프로퍼티도 클래스에 추가할 수 있음

- 클래스 필드는 최근에 더해진 기능

```javascript
class User {
  name = "보라";
  sayHi() {
    alert(`${this.name}님 안녕하세요!`);
  }
}
new User().sayHi(); // 보라님 안녕하세요!
```

클래스를 정의할 때 `<프로퍼티 이름> = <값>`을 써주면 간단히 클래스 필드를 만들 수 있음

`User.prototype`이 아닌 개별 객체에만 클래스 필드가 설정됨

```javascript
class User {
  name = "보라";
}
let user = new User();
alert(user.name); // 보라
alert(User.prototype.name); // undefined
```

클래스 필드에는 복잡한 표현식이나 함수 호출 결과를 사용할 수 있음

```javascript
class User {
  name = prompt("이름을 알려주세요.", "보라");
}
let user = new User();
alert(user.name); // 보라
```

클래스 필드로 바인딩 된 메서드 만들기

- 자바스크립트에서 `this`는 동적으로 결정됨
- 객체 메서드를 여기저기 전달해 전혀 다른 컨텍스트에서 호출하게 되면 `this`는 메서드가 정의된 객체를 참조하지 않음

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
setTimeout(button.click, 1000); // undefined
```

잃어버린 `this`(losing `this`)

- `this`의 컨텍스트를 알 수 없게 되는 문제

두 방법을 사용해 해결 가능

1. `setTimeout(() => button.click(), 1000)` 같이 래퍼 함수를 전달하기
2. 생성자 안 등에서 메서드를 객체에 바인딩하기

두 방법 말고 클래스 필드를 사용해도 문제 해결 가능

```javascript
class Button {
  constructor(value) {
    this.value = value;
  }
  click = () => {
    alert(this.value);
  };
}
let button = new Button("안녕하세요.");
setTimeout(button.click, 1000); // 안녕하세요.
```

클래스 필드 `click = () => { ... }`는 각 `Button` 객체마다 독립적인 함수를 만들어주고 이 함수의 `this`를 해당 객체에 바인딩시켜줌

따라서 개발자는 `button.click`을 아무 곳에나 전달할 수 있고, `this`에는 항상 의도한 값이 들어감

클래스 필드의 이런 기능은 브라우저 환경에서 메서드를 이벤트 리스너로 설정해야 할 때 유용

단점

- 화살표 함수는 기본적으로 prototype을 생성하지 않기 때문에 `class명.prototype`으로 접근이 불가능
  - 테스트 케이스 작성 시 `spyOn(class명.prototype, method명)`으로 mocking할 수 없음
  - 프로토타입이 없으니 상속도 안 됨
- 특정 컨텍스트에 this를 binding 하려는 명확한 목적을 가지고 사용하는 경우를 제외하고 가급적 화살표 함수를 class 내에서 사용하는 것을 지양하는 것이 더 좋을 수 있음

## 클래스 상속

클래스 상속을 사용하면 클래스를 다른 클래스로 확장할 수 있음

- 기존에 존재하던 기능을 토대로 새로운 기능을 만들 수 있음

### extends 키워드

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

클래스 확장 문법

`class Child extends Parent`

```javascript
class Rabbit extends Animal {
  hide() {
    alert(`${this.name}이/가 숨었습니다!`);
  }
}
let rabbit = new Rabbit("흰 토끼");
rabbit.run(5); // 흰 토끼은/는 속도 5로 달립니다.
rabbit.hide(); // 흰 토끼이/가 숨었습니다!
```

```
Animal                       Animal.prototype
[     ] -prototype->         [ constructor: Animal ]
                             [ run: function       ]
                             [ stop: function      ]
                                        ^
                                        | [[Prototype]] <- extends
Rabbit                       Rabbit.prototype
[ constructor ] -prototype-> [ constructor: Rabbit ]
                                        ^
                                        | [[Prototype]]
                              new Rabbit
                             [ name: "White Rabbit ]
```

클래스 `Rabbit`을 사용해 만든 객체는 `rabbit.hide()` 같은 `Rabbit`에 정의된 메서드에도 접근할 수 있고, `rabbit.run()` 같은 `Animal`에 정의된 메서드에도 접근 가능

키워드 `extends`는 프로토타입을 기반으로 동작

`extends`는 `Rabbit.prototype.[[Prototype]]`을 `Animal.prototype`으로 설정

- `Rabbit.prototype`에서 메서드를 찾지 못하면 `Animal.prototype`에서 메서드를 가져옴

엔진은 다음 절차를 따라 메서드 `rabbit.run`의 존재를 확인

1. 객체 `rabbit`에 `run`이 있나 확인. 없음
2. `rabbit`의 프로토타입인 `Rabbit.prototype`에 메서드가 있나 확인. 없음
3. `extends`를 통해 관계가 만들어진 `Rabbit.prototype`의 프로토타입, `Animal.prototype`에 메서드가 있나 확인. 찾음

자바스크립트의 내장 객체는 프로토타입을 기반으로 상속 관계를 맺음

- `Date.prototype.[[Prototype]]`이 `Object.prototype`인 것처럼
- `Date` 객체에서 일반 객체 메서드를 사용할 수 있는 이유

`extends` 뒤에 표현식이 올 수도 있음

- 클래스 문법은 `extends` 뒤에 표현식이 와도 처리해줌

```javascript
function f(phrase) {
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
class User extends f("Hello") {}
new User.sayHi(); // Hello
```

- `class User`는 `f("Hello")`의 반환 값을 상속받음
- 조건에 따라 다른 클래스를 상속받고 싶을 때 유용

### 메서드 오버라이딩

특별한 사항이 없으면 `class Rabbit`은 `class Animal`에 있는 메서드를 그대로 상속받음

그런데 `Rabbit`에서 `stop()` 등의 메서드를 자체적으로 정의하면, 상속받은 메서드가 아닌 자체 메서드가 사용됨

```javascript
class Rabbit extends Animal {
  stop() {} // rabbit.stop()을 호출하면 Animal의 것이 아닌 이 메서드가 사용됨
}
```

- 부모 메서드를 토대로 일부 기능만 변경하고 싶을 때
- 부모 메서드의 기능을 확장하고 싶을 때
- 이럴 때 커스텀 메서드를 만들어 작업하게 되는데, 이미 커스텀 메서드를 만들었더라도 이 과정 전·후에 부모 메서드를 호출하고 싶은 경우

키워드 `super`

- `super.method(...)`는 부모 클래스에 정의된 메서드, `method`를 호출
- `super(...)`는 부모 생성자를 호출하는데, 자식 생성자 내부에서만 사용 가능

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

화살표 함수에는 `super`가 없습니다

- 화살표 함수는 `super`를 지원하지 않음
- `super`에 접근하면 `super`를 외부 함수에서 가져옴

```javascript
class Rabbit extends Animal {
  stop() {
    setTimeout(() => super.stop(), 1000); // 1초 후 부모 stop을 호출
    // setTimeout(function () { super.stop(); }, 1000);
  }
}
```

- 화살표 함수의 `super`는 `stop()`의 `super`와 같아서 위 예시는 의도한 대로 동작
- `setTimeout` 안에서 일반 함수를 사용했다면 에러 발생

```javascript
setTimeout(function () {
  super.stop();
}, 1000);
```

### 생성자 오버라이딩

지금까지는 `Rabbit`에 자체 `constructor`가 없었음

명세서에 따르면, 클래스가 다른 클래스를 상속받고 `constructor`가 없는 경우에는 비어있는 `constructor`가 만들어짐

```javascript
class Rabbit extends Animal {
  // 자체 생성자가 없는 클래스를 상속받으면 자동으로 만들어짐
  constructor(...args) {
    super(...args);
  }
}
```

생성자는 기본적으로 부모 `constructor`를 호출

- 이때 부모 `constructor`에도 인수를 모두 전달
- 클래스에 자체 생성자가 없는 경우에는 이런 일이 모두 자동으로 일어남

`Rabbit`에 커스텀 생성자 추가하기

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
}

class Rabbit extends Animal {
  constructor(name, earLength) {
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
}

let rabbit = new Rabbit("흰 토끼", 10); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

- 에러가 발생하는 이유
- 상속 클래스의 생성자에서는 반드시 `super(...)`를 호출해야 하는데, `super(...)`를 호출하지 않아 에러 발생
- `super(...)`는 `this`를 사용하기 전에 반드시 호출해야 함

`super(...)`를 호출해야 하는 이유

- 상속 클래스의 생성자가 호출될 때 일어나는 일
- 자바스크립트는 상속 클래스의 생성자 함수(derived constructor)와 그렇지 않은 생성자 함수를 구분함
- 상속 클래스의 생성자 함수에는 특수 내부 프로퍼티인 `[[ConstructorKind]]: "derived"`가 이름표처럼 붙음
- 일반 클래스의 생성자 함수와 상속 클래스의 생성자 함수 간 차이는 `new`와 함께 드러남
  - 일반 클래스가 `new`와 함께 실행되면, 빈 객체가 만들어지고 `this`에 이 객체를 할당함
  - 상속 클래스의 생성자 함수가 실행되면, 일반 클래스에서 일어난 일이 일어나지 않음
  - 상속 클래스의 생성자 함수는 빈 객체를 만들고 `this`에 이 객체를 할당하는 일을 부모 클래스의 생성자가 처리해주길 기대함
- 이런 차이 때문에 상속 클래스의 생성자에서는 `super`를 호출해 부모 생성자를 실행해 주어야 함
- 그렇지 않으면 `this`가 될 객체가 만들어지지 않아 에러 발생

```javascript
class Rabbit extends Animal {
  constructor(name, earLength) {
    super(name); // this를 사용하기 전에 super()를 호출하면 Rabbit의 생성자가 제대로 동작
    this.earLength = earLength;
  }
}
```

클래스 필드 오버라이딩: 까다로운 내용

- 오버라이딩은 메서드뿐만 아니라 클래스 필드를 대상으로도 적용할 수 있음
- 부모 클래스의 생성자 안에 있는 오버라이딩한 필드에 접근하려고 할 때 자바스크립트는 다른 프로그래밍 언어와는 다르게 조금 까다로움

```javascript
class Animal {
  name = "animal";
  constructor() {
    alert(this.name); // (*)
  }
}

class Rabbit extends Animal {
  name = "rabbit";
}

new Animal(); // animal
new Rabbit(); // animal
```

- `Animal`을 상속받는 `Rabbit`에서 `name` 필드를 오버라이딩 함
- `Rabbit`에는 따로 생성자가 정의되어 있지 않기 때문에 `Rabbit`을 사용해 인스턴스를 만들면 `Animal`의 생성자가 호출됨
- `new Animal()`과 `new Rabbit()`을 실행할 때 두 경우 모두 `(*)`로 표시한 줄에 있는 `alert` 함수가 실행되면서 얼럿 창에 `animal`이 출력됨
- 이를 통해 부모 생성자는 자식 클래스에서 오버라이딩한 값이 아닌, 부모 클래스 안의 필드 값을 사용한다는 사실을 알 수 있음
- 상속을 받고 필드 값을 오버라이딩했는데 새로운 값 대신 부모 클래스 안에 있는 기존 필드 값을 사용한다?
- 메서드와 비교해 생각해보기

```javascript
class Animal {
  // this.name = "animal" 대신 메서드 사용
  showName() {
    alert("animal");
  }
  constructor() {
    this.showName(); // alert(this.name); 대신 메서드 호출
  }
}

class Rabbit extends Animal {
  showName() {
    alert("rabbit");
  }
}

new Animal(); // animal
new Rabbit(); // rabbit
```

- 자식 클래스에서 부모 생성자를 호출하면 오버라이딩한 메서드가 실행되어야 함 (위 예시가 원하던 결과)
- 그런데 클래스 필드는 자식 클래스에서 필드를 오버라이딩해도 부모 생성자가 오버라이딩한 필드 값을 사용하지 않음
- 부모 생성자는 항상 부모 클래스에 있는 필드의 값을 사용

필드 초기화 순서 때문

- 클래스 필드는 다음과 같은 규칙에 따라 초기화 순서가 달라짐
  - 아무것도 상속받지 않는 베이스 클래스는 생성자 실행 이전에 초기화됨
  - 부모 클래스가 있는 경우에는 `super()` 실행 직후에 초기화됨

예시에서 `Rabbit`은 하위 클래스이고 `constructor()`가 정의되어 있지 않음

- 이런 경우 생성자는 비어있는데 그 안에 `super(...args)`만 있다고 보면 됨

따라서 `new Rabbit()`을 실행하면 `super()`가 호출되고 그 결과 부모 생성자가 실행됨

그런데 이때 하위 클래스 필드 초기화 순서에 따라 하위 클래스 `Rabbit`의 필드는 `super()` 실행 후에 초기화됨

부모 생성자가 실행되는 시점에 `Rabbit`의 필드는 아직 존재하지 않음

이런 이유로 필드를 오버라이딩했을 때 `Animal`에 있는 필드가 사용된 것

이렇게 자바스크립트는 오버라이딩 시 필드와 메서드의 동작 방식이 미묘하게 다름

이런 문제는 오버라이딩한 필드를 부모 생성자에서 사용할 때만 발생

개발 시 필드 오버라이딩이 문제가 되는 상황이 발생하면 필드 대신 메서드를 사용하거나 getter나 setter를 사용해 해결하면 됨

### super 키워드와 `[[HomeObject]]`

`super`에 대해 좀 더 깊이 파고들기

지금까지 배운 내용만으로는 `super`가 제대로 동작하지 않음

내부에서 `super`는 어떻게 동작할까?

- 객체 메서드가 실행되면 현재 객체가 `this`가 됨
- 이 상태에서 `super.method()`를 호출하면 엔진은 현재 객체의 프로토타입에서 `method`를 찾아야 함
- 이런 과정들은 어떻게 일어나나?

엔진은 현재 객체 `this`를 알기 때문에 `this.__proto__.method`를 통해 부모 객체의 `method`를 찾을 수 있을 것 같지만 아님

예시

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
    // 예상대로라면 super.eat()이 동작해야 함
    this.__proto__.eat.call(this); // (*)
  }
};

rabbit.eat(); // 토끼 이/가 먹이를 먹습니다.
```

- `rabbit.__proto__`는 `animal`
- `(*)`로 표시한 줄에서 `eat`을 프로토타입(`animal`)에서 가져오고 현재 컨텍스트에 기반해 `eat`을 호출
- 여기서 `.call(this)`를 주의해서 봐야 함
- `this.__proto__.eat()`만 있으면 현재 객체가 아닌 프로토타입의 컨텍스트에서 부모 `eat`을 실행하기 때문에 `.call(this)`이 있어야 함
- 예상한 내용이 얼럿 창에 출력됨

체인에 객체를 하나 더 추가하면 문제가 발생하기 시작

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
    // call을 사용해 컨텍스트를 옮겨가며 부모(animal) 메서드를 호출
    this.__proto__.eat.call(this); // (*)
  }
};

let longEar = {
  __proto__: rabbit,
  eat() {
    // longEar를 가지고 무언가를 하면서 부모(rabbit) 메서드를 호출
    this.__proto__.eat.call(this); // (**)
  }
};

longEar.eat(); // RangeError: Maximum call stack size exceeded
```

- `longEar.eat()`이 호출될 때 어떤 일이 발생하는지 하나씩 보면 에러가 발생한 이유를 알 수 있음
- `(*)`과 `(**)`로 표시한 두 줄에서 `this`는 현재 객체인 `longEar`가 됨
- 모든 객체 메서드는 프로토타입 등이 아닌 현재 객체를 `this`로 가짐
- 따라서 `(*)`과 `(**)`로 표시한 줄의 `this.__proto__`에는 정확히 같은 값 `rabbit`이 할당됨
- 체인 위로 올라가지 않고 양쪽 모두에서 `rabbit.eat`을 호출하기 때문에 무한 루프에 빠짐

1. `longEar.eat()` 내부의 `(**)`로 표시한 줄에서 `rabbit.eat`을 호출하는데, 이때 `this`는 `longEar`

```javascript
// longEar.eat() 안의 this는 longEar
this.__proto__.eat.call(this); // (**)
// 따라서 윗줄은 아래와 같아짐
longEar.__proto__.eat.call(this);
// longEar의 프로토타입은 rabbit이므로 윗줄은 아래와 같아짐
rabbit.eat.call(this);
```

2. `rabbit.eat` 내부의 `(*)`로 표시한 줄에서 체인 위쪽에 있는 호출을 전달하려 했으나 `this`가 `longEar`이기 때문에 또다시 `rabbit.eat`이 호출됨

```javascript
// rabbit.eat() 안의 this 역시 longEar
this.__proto__.eat.call(this); // (*)
// 따라서 윗줄은 아래와 같아짐
longEar.__proto__.eat.call(this);
// longEar의 프로토타입은 rabbit이므로 윗줄은 아래와 같아짐
rabbit.eat.call(this);
```

3. 이런 내부 동작 때문에 `rabbit.eat`은 체인 위로 올라가지 못하고 자기 자신을 계속 호출해 무한 루프에 빠짐

이런 문제는 `this`만으로는 해결할 수 없음

`[[HomeObject]]`

- 이런 문제를 해결할 수 있는 자바스크립트의 함수 전용 특수 내부 프로퍼티
- 클래스이거나 객체 메서드인 함수의 `[[HomeObject]]` 프로퍼티는 해당 객체가 저장됨
- `super`는 `[[HomeObject]]`를 이용해 부모 프로토타입과 메서드를 찾음

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

longEar.eat(); // 귀가 긴 토끼 이/가 먹이를 먹습slek.
```

- `longEar.eat` 같은 객체 메서드는 `[[HomeObject]]`를 알고 있기 때문에 `this` 없이도 프로토타입으로부터 부모 메서드를 가져올 수 있음

메서드는 자유롭지 않습니다.

- 자바스크립트에서 함수는 대개 객체에 묶이지 않고 자유로움
- 이런 자유성 때문에 `this`가 달라고 객체 간 메서드를 복사하는 것이 가능
- 그런데 `[[HomeObject]]`는 그 존재만으로도 함수의 자유도를 파괴
  - 메서드가 객체를 기억하기 때문
  - 개발자가 `[[HomeObject]]`를 변경할 방법은 없기 때문에 한번 바인딩 된 함수는 더이상 변경되지 않음
- 다행스럽게도 `[[HomeObject]]`는 오직 `super` 내부에서만 유효
  - 그렇기 때문에 `super`를 사용하지 않는 경우에는 메서드의 자유성이 보장됨
  - 객체간 복사도 가능
  - 메서드에서 `super`를 사용하면 이야기가 달라짐

객체 간 메서드를 잘못 복사한 경우에 `super`가 제대로 동작하지 않는 경우

```javascript
let animal = {
  sayHi() {
    console.log(`나는 동물입니다.`);
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
    console.log("나는 식물입니다.");
  }
};

let tree = {
  __proto__: plant,
  sayHi: rabbit.sayHi // (*)
};

tree.sayHi(); // 나는 동물입니다.
```

- 잘못 출력된 이유
- `(*)`로 표시한 줄에서 메서드 `tree.sayHi`는 중복 코드를 방지하기 위해 `rabbit`에서 메서드를 복사해옴
- 그런데 복사해온 메서드는 `rabbit`에서 생성했기 때문에 이 메서드의 `[[HomeObject]]`는 `rabbit`임
  - 개발자는 `[[HomeObject]]`를 변경할 수 없음
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

- `[[HomeObject]]`는 클래스와 일반 객체의 메서드에 정의됨
- 그런데 객체 메서드의 경우 `[[HomeObject]]`가 제대로 동작하게 하려면 메서드를 반드시 `method()` 형태로 정의해야 함
  - `"method: function()"` 형태로 정의하면 안 됨
- 개발자 입장에서 두 방법의 차이는 중요하지 않을 수 있지만 자바스크립트 입장에서는 중요함
- 메서드 문법이 아닌(non-method syntax) 함수 프로퍼티를 사용해 예시를 작성해보면 다음과 같음
- `[[HomeObject]]` 프로퍼티가 설정되지 않기 때문에 상속이 제대로 되지 않음

```javascript
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

rabbit.eat(); // SyntaxError: 'super' keyword unexpected here
// [[HomeObject]]가 없어서 에러 발생
```

화살표 함수는 `this`나 `super`를 갖지 않으므로 주변 컨텍스트에 잘 들어맞음

### 예시

인스턴스 생성 오류

시계 확장하기

## 참고

> [클래스](https://ko.javascript.info/classes)
