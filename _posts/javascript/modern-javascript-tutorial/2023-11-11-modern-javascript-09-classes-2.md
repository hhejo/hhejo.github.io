---
title: 모던 JavaScript 튜토리얼 09 - 클래스 2
date: 2023-11-11 09:22:46 +0900
last_modified_at: 2024-04-10 15:01:32 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, static, protected, private]
---

정적 메서드와 정적 프로퍼티, private, protected 프로퍼티와 메서드

## 정적 메서드와 정적 프로퍼티

정적 메서드(static method)

- 클래스 함수 자체에 메서드를 설정 가능 (`prototype`이 아님)
- 클래스 안에서 `static` 키워드를 붙여 생성
- 어떤 특정한 객체가 아닌 클래스에 속한 함수를 구현하고자 할 때 주로 사용
- 메서드를 프로퍼티 형태로 직접 할당하는 것과 동일한 일을 함
- `User.staticMethod()`가 호출될 때 `this`의 값은 클래스 생성자인 `User` 자체가 됨
- 예로 정적 메서드는 데이터베이스 관련 클래스에도 사용됨
  - 항목 검색, 저장, 삭제 등을 수행 (`Database.remove({ id: 12345 })`)

```javascript
class User {
  static staticMethod() {
    alert(this === User);
  }
}
User.staticMethod(); // ture

class User2 {}
User2.staticMethod = function () {
  alert(this === User);
};
User2.staticMethod(); // true
```

객체 `Article`이 여러 개 있고, 이들을 비교해줄 함수가 필요한 경우

- `Article.compare`를 추가할 수 있음. `Article.compare`는 `article`을 비교해주는 수단
- 글 전체를 위에서 바라보며 비교를 수행
- `Article.compare`가 글 하나의 메서드가 아닌 클래스의 메서드여야 하는 이유

```javascript
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }
  static compare(articleA, articleB) {
    return articleA.date - articleB.date;
  }
}

let articles = [
  new Article("HTML", new Date(2019, 1, 1)),
  new Article("CSS", new Date(2019, 0, 1)),
  new Article("JavaScript", new Date(2019, 11, 1))
];

articles.sort(Article.compare);
alert(articles[0].title); // CSS
```

팩토리 메서드

0. 다양한 방법을 사용해 조건에 맞는 `article` 인스턴스를 만들기
1. 매개변수(`title`, `date` 등)를 이용해 관련 정보가 담긴 `article` 생성
   - 생성자를 사용해 구현 가능
2. 오늘 날짜를 기반으로 비어있는 `article` 생성
   - 클래스에 정적 메서드를 만들어 구현 가능
3. 기타 등등

```javascript
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }
  static createTodays() {
    return new this("Today's digest", new Date()); // this=Article
  }
}

let article = Article.createTodays();
alert(article.title); // Today's digest
```

### 정적 프로퍼티

- 정적 프로퍼티(static property): 일반 클래스 프로퍼티와 유사하나 앞에 `static`이 붙는다는 점만 다름
- 클래스에 프로퍼티를 직접 할당하는 것과 동일하게 동작

```javascript
class Article {
  static publisher = "Ilya Kantor";
}
alert(Article.publisher); // Ilya Kantor

class Article2 {}
Article2.publisher = "Ilya Kantor";
alert(Article2.publisher); // Ilya Kantor
```

### 정적 프로퍼티와 메서드 상속

- 정적 프로퍼티와 정적 메서드는 상속됨
- `Animal.compare`와 `Animal.planet`은 상속됨
  - 각각 `Rabbit.compare`와 `Rabbit.planet`에서 접근 가능
- `Rabbit.compare`를 호출하면 `Animal.compare`가 호출됨. 프로토타입 덕분에 가능
- `extends` 키워드는 `Rabbit`의 `[[Prototype]]`이 `Animal`을 참조하도록 해줌

```javascript
class Animal {
  static planet = "지구";
  constructor(name, speed) {
    this.speed = speed;
    this.name = name;
  }
  run(speed = 0) {
    this.speed += speed;
    alert(`${this.name}가 속도 ${this.speed}로 달립니다.`);
  }
  static compare(animalA, animalB) {
    return animalA.speed - animalB.speed;
  }
}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name}가 숨었습니다!`);
  }
}

let rabbits = [new Rabbit("흰 토끼", 10), new Rabbit("검은 토끼", 5)];
rabbits.sort(Rabbit.compare);
rabbits[0].run(); // 검은 토끼가 속도 5로 달립니다.
alert(Rabbit.planet); // 지구
```

```
Animal                   Animal.prototype
[ compare ] -prototype-> [ constructor: Animal ]
[ planet? ]              [ run: function       ]
     ^                              ^
     | [[Prototype]]                | [[Prototype]]
Rabbit                   Rabbit.prototype
[         ] -prototype-> [ constructor: Rabbit ]
                         [ hide: function      ]
                                    ^
                                    | [[Prototype]]
                         rabbit     |
                         [ name: "White Rabbit" ]
```

`Rabbit extends Animal`은 두 개의 `[[Prototype]]` 참조를 생성

1. 함수 `Rabbit`은 프로토타입을 통해 함수 `Animal`을 상속 받음
2. `Rabbit.prototype`은 프로토타입을 통해 `Animal.prototype`을 상속 받음
3. 이런 과정이 있기 때문에 일반 메서드 상속과 정적 메서드 상속이 가능

```javascript
class Animal {}
class Rabbit extends Animal {}
alert(Rabbit.__proto__ === Animal); // true
alert(Rabbit.prototype.__proto__ === Animal.prototype); // true
```

### 예제

Object 상속을 명시적으로 해주는 경우와 아닌 경우의 결과의 차이?

- Object를 상속받는 클래스
  - 객체는 보통 `Object.prototype`을 상속받고 일반 객체 메서드에 접근 가능
- `class Rabbit extends Object` vs. `class Rabbit`
- 상속 받는 클래스의 생성자는 `super()`를 반드시 호출해야 함
- 그렇지 않으면 `this`가 정의되지 않음
- `super()`를 추가해도 두 방식은 다른 점이 있음
- `extends` 문법은 두 개의 프로토타입을 설정
  - 생성자 함수의 `prototype` 사이(일반 메서드용)
  - 생성자 함수 자체(정적 메서드용)

```javascript
// class Rabbit
class Rabbit {
  constructor(name) {
    this.name = name;
  }
}
let rabbit = new Rabbit("Rab");
alert(rabbit.hasOwnProperty("name")); // true

// class Rabbit extends Object
class Rabbit2 extends Object {
  constructor(name) {
    // super(); // 없으면 에러 발생
    this.name = name;
  }
}
let rabbit2 = new Rabbit2("Rab");
alert(rabbit2.hasOwnProperty("name")); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

`class Rabbit`의 경우

- `extends Object`가 없으면 `Rabbit.__proto__`는 `Object`로 설정되지 않음
- `Rabbit`에서 `Object`의 정적 메서드 사용 불가

```javascript
class Rabbit {}
alert(Rabbit.prototype.__proto__ === Object.prototype); // true
alert(Rabbit.__proto__ === Object); // false
alert(Rabbit.__proto__ === Function.prototype); // true (모든 함수의 기본 프로토타입)
alert(Rabbit.getOwnPropertyNames({ a: 1, b: 2 })); // TypeError: Rabbit.getOwnPropertyNames is not a function
```

`class Rabbit extends Object`의 경우

- `Rabbit`을 통해 `Object`의 정적 메서드에 접근 가능

```javascript
class Rabbit extends Object {}
alert(Rabbit.prototype.__proto__ === Object.prototype); // true
alert(Rabbit.__proto__ === Object); // true
alert(Rabbit.getOwnPropertyNames({ a: 1, b: 2 })); // a,b. 보통은 Object.getOwnPropertyNames로 호출
```

```
class Rabbit             class Rabbit extends Object

Function.prototype       Function.prototype
[ call: function ]       [ call: function ]
[ bind: function ]       [ bind: function ]
        ^                        ^
        | [[Prototype]]          | [[Prototype]]
Rabbit  |                Object
[ constructor    ]       [ constructor    ]
                                 ^
                                 | [[Prototype]]
                         Rabbit  |
                         [ constructor    ]
```

그냥 클래스를 정의하기 vs. 명시적으로 `Object`를 상속해 클래스를 정의하기

- 내장 객체, `Object`의 생성자는 `Object.__proto__ === Function.prototype` 관계를 가짐
- `Function.prototype`에 정의된 일반 함수 메서드는 두 경우에 모두 사용 가능
- `Object.prototype === Function.prototype.__proto__`

|               class Rabbit                |        class Rabbit extends Object        |
| :---------------------------------------: | :---------------------------------------: |
|                     -                     | 생성자에서 `super()`를 반드시 호출해야 함 |
| `Rabbit.__proto__ === Function.prototype` |       `Rabbit.__proto__ === Object`       |

## private, protected 프로퍼티와 메서드

내부 인터페이스와 외부 인터페이스를 구분짓기

- 객체 지향 프로그래밍에서 가장 중요한 원리 중 하나

### 내부 인터페이스와 외부 인터페이스

객체 지향 프로그래밍에서 프로퍼티와 메서드는 두 그룹으로 분류됨

- 내부 인터페이스(internal interface)
  - 동일한 클래스 내의 다른 메서드에서는 접근 가능
  - 클래스 밖에서는 접근할 수 없는 프로퍼티와 메서드
- 외부 인터페이스(external interface)
  - 클래스 밖에서도 접근 가능한 프로퍼티와 메서드

자바스크립트에는 두 가지 타입의 객체 필드(프로퍼티와 메서드)가 있음

- public: 어디서든지 접근할 수 있으며 외부 인터페이스를 구성
  - 지금까지 다룬 프로퍼티와 메서드는 모두 public
- private: 클래스 내부에서만 접근할 수 있으며, 내부 인터페이스를 구성할 때 쓰임
- protected
  - private과 비슷하나 자손 클래스에서도 접근 가능
  - 내부 인터페이스를 만들 때 유용
  - 자손 클래스의 필드에 접근해야 하는 경우가 많아 private 필드보다 더 광범위하게 사용됨
  - 자바스크립트 이외의 다수 언어에서 protected 필드를 지원
  - protected를 사용하면 편리한 점이 많아 자바스크립트에서 지원하지 않지만 모방해서 사용

### 프로퍼티 보호하기

protected 프로퍼티

- 앞에 밑줄 `_`이 붙음
- 자바스크립트에서 강제한 사항은 아님
- 밑줄은 프로그래머들 사이에서 외부 접근이 불가능한 프로퍼티나 메서드를 나타낼 때 사용

```javascript
class CoffeeMachine {
  _waterAmount = 0; // protected 프로퍼티(사실은 public)
  set waterAmount(value) {
    if (value < 0) throw new Error("물의 양은 음수가 될 수 없습니다.");
    this._waterAmount = value;
  }
  get waterAmount() {
    return this._waterAmount;
  }
  constructor(power) {
    this._power = power; // public 프로퍼티
    alert(`전력량이 ${power}인 커피머신을 만듭니다.`);
  }
}
let coffeeMachine = new CoffeeMachine(100);
coffeeMachine.waterAmount = -10; // Error: 물의 양은 음수가 될 수 없습니다.
```

### 읽기 전용 프로퍼티

- 프로퍼티를 생성할 때만 값을 할당할 수 있고, 그 이후에는 값을 절대 수정하지 말아야 하는 경우 활용
- setter 없이 getter만 만들어 읽기 전용 프로퍼티 생성

```javascript
class CoffeeMachine {
  // ...
  constructor(power) {
    this._power = power;
  }
  get power() {
    return this._power;
  }
}
let coffeeMachine = new CoffeeMachine(100);
alert(`전력량이 ${coffeeMachine.power}인 커피머신을 만듭니다.`);
coffeeMachine.power = 10; // TypeError: Cannot set property power of #<CoffeeMachine> which has only a getter
```

getter와 setter 함수

- `get`, `set` 문법을 사용하는 방법보다 `get.../set...` 형식의 함수가 선호됨
- 다소 길어보이나 함수를 선언하면 다수의 인자를 받을 수 있어 좀 더 유용함

```javascript
class CoffeeMachine {
  _waterAmount = 0;
  setWaterAmount(value) {
    if (value < 0) throw new Error("물의 양은 음수가 될 수 없습니다.");
    this._waterAmount = value;
  }
  getWaterAmount() {
    return this._waterAmount;
  }
}
new CoffeeMachine().setWaterAmouont(100);
```

protected 필드는 상속됨

- `class MegaMachine extends CoffeeMachine`로 클래스를 상속 받는 경우
- 새로운 클래스의 메서드에서 `this._waterAmount`나 `this._power`로 프로퍼티에 접근 가능
- private 필드와 다르게 자연스러운 상속 가능

### private 프로퍼티

private 프로퍼티와 메서드

- `#`으로 시작해 private 필드를 의미
- 클래스 안에서만 접근 가능하고 클래스 외부나 자손 클래스에서는 접근 불가
- private 필드는 `this[name]`로 사용 불가
- private 필드는 public 필드와 상충하지 않음
  - private 프로퍼티 `#waterAmount`와 public 프로퍼티 `waterAmount`를 동시에 가질 수 있음
- private 필드는 protected 필드와는 달리 언어 자체에 의해 강제된다는 점이 장점
- 제안(proposal) 목록에 등재된 문법으로, 명세서에 등재되기 직전 상태

```javascript
class CoffeeMachine {
  #waterLimit = 200;
  #waterAmount = 0;
  #checkWater(value) {
    if (value < 0) throw new Error("물의 양은 음수가 될 수 없습니다.");
    if (value > this.#waterLimit) throw new Error("물이 용량을 초과합니다.");
  }
  get waterAmount() {
    return this.#waterAmount;
  }
  set waterAmount(value) {
    if (value < 0) throw new Error("물의 양은 음수가 될 수 없습니다.");
    this.#waterAmount = value;
  }
}

let coffeeMachine = new CoffeeMachine();
coffeeMachine.#checkWater(); // SyntaxError: Private field '#checkWater' must be declared in an enclosing class
coffeeMachine.#waterLimit = 1000; // SyntaxError: Private field '#waterLimit' must be declared in an enclosing class

let machine = new CoffeeMachine();
machine.waterAmount = 100;
alert(machine.#waterAmount); // SyntaxError: Private field '#waterAmount' must be declared in an enclosing class

class MegaCoffeeMachine extends CoffeeMachine {
  method() {
    alert(this.#waterAmount); // SyntaxError: Private field '#waterAmount' must be declared in an enclosing class
  }
}
```

```javascript
class User {
  sayHi() {
    let fieldName = "name";
    alert(`Hello, ${this[fieldName]}`); // 보통은 가능하지만 private 필드는 접근 불가능
  }
}
```

## 참고

> [클래스](https://ko.javascript.info/classes)
