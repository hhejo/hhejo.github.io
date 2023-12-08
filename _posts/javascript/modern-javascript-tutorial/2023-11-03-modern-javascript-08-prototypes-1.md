---
title: 모던 JavaScript 튜토리얼 08 - 프로토타입과 프로토타입 상속 1
date: 2023-11-03 06:05:33 +0900
last_modified_at: 2023-12-07 08:09:24 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

프로토타입 상속, 함수의 prototype 프로퍼티

## 프로토타입 상속

프로토타입 상속(prototypal inheritance)

- 기존에 있는 기능을 가져와 확장해야 하는 경우
- `user`: 사람에 관한 프로퍼티와 메서드를 가진 객체
- `admin`, `guest`: 유사하지만 약간의 차이가 있는 객체
- `user`에 약간의 기능을 얹어 `admin`, `guest` 객체를 만들고 싶음
- 프로토타입 상속을 이용해 실현
- 자바스크립트 언어의 고유 기능

### `[[Prototype]]`

프로토타입(prototype)

- `[[Prototype]]`이 참조하는 객체

`[[Prototype]]` 프로퍼티

- 자바스크립트의 객체는 명세서에 명명한 `[[Prototype]]` 프로퍼티를 가짐
- 내부 프로퍼티, 숨김 프로퍼티
- 값은 `null` 혹은 다른 객체에 대한 참조
- 다양한 방법으로 값 설정 가능
- 객체에는 오직 하나의 `[[Prototype]]`만 존재
  - 두 개의 객체를 상속받을 수 없음

프로토타입 상속

- `object`에서 프로퍼티를 읽는데 해당 프로퍼티가 없는 경우
  - 자바스크립트는 자동으로 프로토타입에서 프로퍼티를 탐색
- 이런 동작 방식을 프로그래밍에서 `프로토타입 상속`이라 부름
- 언어 차원에서 지원하는 편리한 기능이나 개발 테크닉 중 프로토타입 상속에 기반해 만들어진 것들이 많음

`__proto__`

- `[[Prototype]]`용 getter이자 setter
  - `__proto__`는 `[[Prototype]]`과는 다름
- 하위 호환성 때문에 여전히 `__proto__` 사용 가능
- 브라우저 환경에서만 지원하도록 자바스크립트 명세서에 규정
  - 사실 서버 사이드를 포함한 모든 호스트 환경에서 지원
- `Object.getPropertyOf`, `Object.setPropertyOf`으로 프로토타입을 get하거나 set하는 것이 좋음

상속 프로퍼티(inherited property)

- 프로토타입에서 상속받은 프로퍼티

프로토타입 체이닝의 제약 사항

1. 순환 참조(circular reference) 불가
   - `__proto__`를 이용해 닫힌 형태로 다른 객체를 참조하면 에러 발생
2. `__proto__`의 값은 객체나 `null`만 가능
   - 다른 자료형은 무시됨

```
prototype object
[               ]
        ^
        | [[Prototype]]
object  |
[               ]
```

```
animal
[ eats: true  ] rabbit의 prototype
       ^
       | [[Prototype]]
rabbit |
[ jumps: true ] animal을 상속받음
```

```javascript
let animal = {
  eats: true,
  walk() {
    alert("동물이 걷습니다.");
  }
};
let rabbit = {
  jumps: true
};
rabbit.__proto__ = animal;
rabbit.walk(); // 동물이 걷습니다.
alert(rabbit.eats); // true
alert(rabbit.jumps); // true
```

```javascript
let animal = {
  eats: true,
  walk() {
    alert("동물이 걷습니다.");
  }
};
let rabbit = {
  jumps: true,
  __proto__: animal
};
let longEar = {
  earLength: 10,
  __proto__: rabbit
};
longEar.walk(); // 동물이 걷습니다. 프로토타입 체인
alert(longEar.jumps); // true
```

```
animal
[ eats: true     ]
[ walk: function ]
        ^
        | [[Prototype]]
rabbit  |
[ jumps: true    ]
        ^
        | [[Prototype]]
longEar |
[ earLength: 10  ]
```

### 프로토타입은 읽기 전용이다

프로토타입은 읽기 전용

- 프로토타입은 프로퍼티를 읽을 때만 사용
- 프로퍼티를 추가, 수정, 삭제하는 연산은 객체에 직접 해야 함

```javascript
let animal = {
  eats: true,
  walk() {} // rabbit은 이제 이 메서드를 사용하지 않음
};
let rabbit = {
  __proto__: animal
};
rabbit.walk = function () {
  alert("토끼가 깡충깡충 뜁니다.");
};
rabbit.walk(); // 토끼가 깡충깡충 뜁니다. (프로토타입에 있는 메서드가 실행되지 않음)
```

```
animal
[ eats: true     ]
[ walk: function ]
         ^
         | [[Prototype]]
rabbit   |
[ walk: function ]
```

접근자 프로퍼티(accessor property) 상속

- setter 함수로 접근자 프로퍼티에 값을 할당하는 경우
- 상속받은 객체에 접근자 프로퍼티가 추가되지 않음
  - `admin`에 접근자 프로퍼티 `fullName`이 추가되지 않음

```javascript
let user = {
  name: "John",
  sername: "Smith",
  set fullName(value) {
    [this.name, this.surname] = value.split(" "); // admin.fullName = 으로 호출하면 this=admin
  },
  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};
let admin = {
  __proto__: user,
  isAdmin: true
};
alert(admin.fullName); // John Smith
admin.fullName = "Alice Cooper"; // setter 함수가 실행되고 admin에 name, surname 프로퍼티 추가됨
alert(admin.fullName); // Alice Cooper. setter에 의해 추가된 admin의 프로퍼티(name, surname)에서 값을 가져옴
alert(user.fullName); // John Smith. 본래 user에 있었던 프로퍼티 값
```

### this가 나타내는 것

`this`

- 프로토타입에 영향 받지 않음
- 메서드를 객체에서, 프로토타입에서 호출했는지 상관 없음
- `this`는 언제나 `.` 앞에 있는 객체
- 상속받은 메서드를 사용하더라도 객체는 프로토타입이 아닌 자신의 상태를 수정
- 객체 하나에 메서드를 많이 구현하고, 여러 객체에서 이 커다란 객체를 상속받게 하는 경우가 많음

```javascript
let animal = {
  walk() {
    if (!this.isSleeping) alert("동물이 걸어갑니다.");
  },
  sleep() {
    this.isSleeping = true;
  }
};
let rabbit = {
  name: "하얀 토끼",
  __proto__: animal
};
rabbit.sleep(); // rabbit에 새로운 프로퍼티 isSleeping을 추가하고 그 값을 true로 변경함
alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined. 프로토타입에는 isSleeping이라는 프로퍼티가 없음
```

```
animal
[ walk: function       ]
[ sleep: function      ] 메서드는 공유
           ^
           | [[Prototype]]
rabbit     |
[ name: "White Rabbit" ]
[ isSleeping: true     ] 객체의 상태는 공유되지 않음
```

### for..in 반복문

`for..in`

- 상속 프로퍼티도 순회 대상에 포함

`obj.hasOwnProperty(key)`

- 상속 프로퍼티를 순회 대상에서 제외 가능
- `key`의 프로퍼티가 상속 프로퍼티이면 `false` 반환
- `obj`에 직접 구현되어 있는 프로퍼티이면 `true` 반환

객체의 키-값을 순회하는 메서드

- 대부분은 상속 프로퍼티를 제외하고 동작
- 해당 객체에서 정의한 프로퍼티만 연산 대상에 포함
- `Object.keys`, `Object.values`

`Object.prototype` 객체

- 객체 리터럴로 선언하면 자동으로 `Obejct.prototype`을 상속받음

```javascript
let animal = { eats: true };
let rabbit = { jumps: true, __proto__: animal };
alert(Object.keys(rabbit)); // jumps
for (let prop in rabbit) alert(prop); // jumps, eats
```

```javascript
let animal = { eats: true };
let rabbit = { jumps: true, __proto__: animal };
for (let prop in rabbit) {
  let isOwn = rabbit.hasOwnProperty(prop); // Object.prototype.hasOwnProperty에서 옴
  if (isOwn) alert(`객체 자신의 프로퍼티: ${prop}`); // jumps
  else alert(`상속 프로퍼티: ${prop}`); // eats
}
```

```
               null
                 ^
                 | [[Prototype]]
Object.prototype |
[ toString: function       ] null을 상속
[ hasOwnProperty: function ]
[ ...                      ]
              ^
              | [[Prototype]]
animal        |
[ eats: true               ] Object.prototype을 상속
              ^
              | [[Prototype]]
rabbit        |
[ jumps: true              ] animal을 상속
```

### 예제

프로토타입 이해하기

```javascript
let animal = {
  jumps: null
};
let rabbit = {
  __proto__: animal,
  jumps: true
};

console.log(rabbit.jumps); // true
delete rabbit.jumps;
console.log(rabbit.jumps); // null
delete animal.jumps;
console.log(rabbit.jumps); // undefined
```

검색 알고리즘

모던 엔진

- 객체에서 프로퍼티를 가져오는 것
- 프로토타입에서 프로퍼티를 가져오는 것
- 모던 엔진에서는 둘 사이에 성능적인 차이가 없음
  - 해당 프로퍼티가 발견된 곳을 기억
  - 다음 요청부터는 그곳에서 검색을 시작
- 모던 엔진은 뭔가 변화가 생기면 내부 캐시를 변경해줄 정도로 똑똑함
  - 최적화를 안전하게 수행

```javascript
let head = { glasses: 1 };
let table = { pen: 3 };
let bed = { sheet: 1, pillow: 2 };
let pockets = { money: 2000 };

pockets.__proto__ = bed;
bed.__proto__ = table;
table.__proto__ = head;

console.log(pockets.pen); // 3
console.log(bed.glasses); // 1
```

어디에 프로퍼티가 추가될까요

```javascript
let animal = {
  eat() {
    this.full = true;
  }
};
let rabbit = {
  __proto__: animal
};

rabbit.eat(); // full 프로퍼티는 animal, rabbit 어디에 생성되나?
console.log(animal); // { eat: [Function: eat] }
console.log(rabbit); // { full: true }
```

왜 햄스터 두 마리 모두 배가 꽉 찼을까요

- `speedy.eat("apple")`을 호출하면 생기는 일
  1. 메서드 `eat`은 프로토타입 `hamster`에서 발견되고 `this`에 `speedy`가 할당되어 메서드 실행
  2. `this.stomach.push(food)` 실행하나 `speedy`에는 `stomach`이 없음
  3. 프로토타입 체인에서 `stomach` 발견
  4. 프로토타입 `hamster`의 `stomach`에 `food` 추가
- 이런 동작 방식 때문에 모든 햄스터가 하나의 `stomach`을 공유
- `lazy.stomach.push()`, `speedy.stomach.push()`를 호출한 경우
  - 프로토타입에서 `stomach`이 발견되고 여기에 새로운 데이터 추가
- 문제를 해결하는 2가지 방법
  - `this.stomach=`을 사용해 객체 자체에 해당 프로퍼티 추가
  - 햄스터가 각자의 `stomach`을 가지게 하면 `push` 사용 가능

```javascript
let hamster = {
  stomach: [], // 특정 객체의 상태를 설명하는 프로퍼티 stomach
  eat(food) {
    this.stomach.push(food);
    // this.stomach = [food];
  }
};
let speedy = {
  __proto__: hamster
  // stomach: [] // 객체 자체에 정의
};
let lazy = {
  __proto__: hamster
  // stomach: [] // 객체 자체에 정의
};

speedy.eat("apple");
console.log(speedy); // {}
console.log(speedy.stomach); // [ "apple" ]
console.log(lazy); // {}
console.log(lazy.stomach); // [ "apple" ]
```

## 함수의 prototype 프로퍼티

생성자 함수로 객체 생성

- 객체 리터럴로 생성하는 것과의 차이?
- 생성자 함수의 프로토타입이 객체일 때
- `new` 연산자로 만든 객체는 생성자 함수의 프로토타입 정보를 사용해 `[[Prototype]]`을 설정
- 프로토타입 기반 상속은 초기 자바스크립트의 주요 기능 중 하나
  - 그런데 과거에는 프로토타입에 직접 접근할 수 없었음
- 생성자 함수의 `prototype` 프로퍼티를 이용
  - 많은 스크립트가 아직 이 방법을 사용하는 이유

`F.prototype` 프로퍼티

- `prototype`은 생성자 함수 `F`에 정의된 일반 프로퍼티
- 바로 앞에서 배운 프로토타입과는 이름만 같고 실제로는 다름
- 일반적인 프로퍼티
- `F.prototype`은 `new F`를 호출할 때만 사용
- `new F`를 호출할 때 만들어지는 새로운 객체의 `[[Prototype]]`을 할당해줌
- 새로운 객체가 만들어진 후에 `F.prototype` 프로퍼티가 바뀌는 경우
  - `F.prototype = <another object>`
  - `new F`로 생성한 또 다른 새로운 객체는 `<another object>`를 `[[Prototype]]`으로 가짐
- 기존에 있던 객체의 `[[Prototype]]`은 그대로 유지

```javascript
let animal = { eats: true };
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype = animal; // `new Rabbit`을 호출해 만든 새로운 객체의 `[[Prototype]]`을 `animal`로 설정
let rabbit = new Rabbit("흰 토끼"); // rabbit.__proto__ == animal
alert(rabbit.eats); // true
```

```
Rabbit               animal
[     ] -prototype-> [ eats: true            ]
                                 ^
                                 | [[Prototype]]
                     rabbit      |
                     [ name: "White Rabbit"  ] animal을 상속받음
```

생성자 함수, 객체 리터럴, 객체 생성자로 생성한 객체

```javascript
function NewFunction() {}
let fromNewFunction = new NewFunction();
console.log(fromNewFunction.__proto__.__proto__ == Object.prototype); // true
console.log(NewFunction.prototype.__proto__ == Object.prototype); // true
console.log(fromNewFunction.__proto__); // {}
console.log(NewFunction.prototype); // {}

let fromObjectLiteral = {};
console.log(fromObjectLiteral.__proto__ == Object.prototype); // true

let fromNewObject = new Object();
console.log(fromNewObject.__proto__ == Object.prototype); // true
```

```
                          Object.prototype
                          [                       ]
                                      ^                            ^                            ^
                                      | [[Prototype]]              | [[Prototype]]              | [[Prototype]]
NewFunction               {}          |                fromObjectLiteral            fromNewObject
[          ] -prototype-> [                       ]    [                       ]    [                       ]
                                      ^
                                      | [[Prototype]]
                          fromNewFunction
                          [                       ]
```

### 함수의 디폴트 프로퍼티 prototype과 constructor 프로퍼티

`prototype` 프로퍼티

- 함수의 디폴트 프로퍼티
- 모든 함수에 기본적으로 존재
- `constructor` 프로퍼티 하나만 있는 객체를 가리킴
- 일반 객체에 `prototype` 프로퍼티를 추가해도 아무런 일이 일어나지 않음
- 모든 함수는 기본적으로 `F.prototype = { constructor: F }`를 가짐

`constructor` 프로퍼티

- 함수 자신을 가리킴
- 객체의 생성자를 얻을 수 있음
- 기존에 있던 객체의 `constructor`를 사용해 새로운 객체 생성 가능
- 어떤 객체를 만들 때 어떤 생성자가 사용되었는지 알 수 없는 경우 유용
  - 객체가 서드 파티 라이브러리에서 온 경우
- 자바스크립트는 알맞은 `constructor` 값을 보장하지 않음
  - 함수에는 기본으로 `prototype`이 설정된다는 것뿐

함수에 기본으로 설정되는 `prototype` 프로퍼티 값을 다른 객체로 바꾸는 경우

- `new`로 생성된 객체에 `constructor`가 없음
- `prototype` 전체를 덮어쓰지 않기
- `prototype`에 원하는 프로퍼티를 추가, 제거하기
- 실수로 `prototype`을 덮어썼어도 `constructor` 프로퍼티를 수동으로 다시 만들면 다시 사용할 수 있음

```javascript
function Rabbit() {} // 함수를 만들기만 해도 디폴트 프로퍼티 prototype이 설정됨
// Rabbit.prototype = { constructor: Rabbit };
alert(Rabbit.prototype.constructor == Rabbit); // true
```

```
Rabbit            default "prototype"
[ prototype -]--->[              ]
[            ]<---[- constructor ]
```

```javascript
function Rabbit() {}
let rabbit = new Rabbit();
alert(Rabbit.prototype.constructor == Rabbit); // true
alert(rabbit.constructor == Rabbit); // true
alert(rabbit.__proto__.constructor == Rabbit); // true
alert(rabbit.__proto__.constructor == Rabbit.prototype.constructor); // true
alert(rabbit.__proto__.constructor == rabbit.constructor); // true
```

```
Rabbit                 default "prototype"
[     ] -prototype->   [                 ]
[     ] <-constructor- [                 ]
   ^                            ^
   |                            | [[Prototype]]
   |                   rabbit   |
   ㄴ---constructor--- [                 ]
```

```javascript
function Rabbit(name) {
  this.name = name;
  alert(name);
}
// new Rabbit으로 만든 객체 모두 constructor 프로퍼티 사용 가능
// 이때 [[Prototype]]을 거침
let rabbit = new Rabbit("흰 토끼");
let rabbit2 = new rabbit.constructor("검정 토끼");
```

```javascript
function Rabbit() {}
Rabbit.prototype = { jumps: true }; // Rabbit.prototype 전체를 덮어씀
let rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // false

function Rabbit2() {}
Rabbit2.prototype.jumps = true; // Rabbit2.prototype 전체를 덮어쓰지 않고 원하는 프로퍼티를 추가
let rabbit2 = new Rabbit2();
alert(rabbit2.constructor === Rabbit); // true. 디폴트 프로퍼티 Rabbit.prototype.constructor 유지

Rabbit.prototype = { jumps: true, constructor: Rabbit }; // Rabbit.prototype.constructor 사용 가능
rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // true
```

### 예제

prototype 변경하기

```javascript
function Rabbit() {}
Rabbit.prototype = { eats: true };
let rabbit = new Rabbit();
// Rabbit.prototype = {};
// Rabbit.prototype.eats = false;
// delete rabbit.eats;
// delete Rabbit.prototype.eats;
alert(rabbit.eats); // true, false, true, undefined
```

동일한 생성자 함수로 객체 만들기

- `new user.constructor("Pete")`는 `user`에서 `constructor`를 탐색
- 객체에서 원하는 프로퍼티를 찾지 못해 프로토타입에서 검색 시작
- `user`의 프로토타입은 `User.prototype`
  - `User.prototype`은 빈 객체(일반 객체) `{}`
  - 일반 객체의 프로토타입은 `Object.prototype`
- `Object.prototype.constructor == Object`
  - 따라서 `Object` 사용
- 결국 `let user2 = new user.constructor("Pete")`는 `let user2 = new Object("Pete")`가 됨
  - `Object`의 생성자는 인수를 무시하고 항상 빈 객체를 생성
- `let user2 = new Object("Pete")`는 `let user2 = {}`와 같음

```javascript
function User(name) {
  this.name = name;
}
User.prototype = {}; // 이 줄이 없었다면 user2.name="Pete"
let user = new User("John");
let user2 = new user.constructor("Pete");
alert(user2.name); // undefined
```

## 참고

> [프로토타입과 프로토타입 상속](https://ko.javascript.info/prototypes)
