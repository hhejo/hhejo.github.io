---
title: 모던 JavaScript 튜토리얼 08 - 프로토타입과 프로토타입 상속 1
date: 2023-11-03 06:05:33 +0900
last_modified_at: 2023-11-03 06:05:33 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

프로토타입 상속, 함수의 prototype 프로퍼티

## 프로토타입 상속

개발을 하다 보면 기존에 있는 기능을 가져와 확장해야 하는 경우가 생김

`user`: 사람에 관한 프로퍼티와 메서드를 가진 객체

- `admin`, `guest`: 유사하지만 약간의 차이가 있는 객체
- `user`에 약간의 기능을 얹어 `admin`, `guest` 객체를 만들고 싶음

프로토타입 상속(prototypal inheritance)를 이용해 실현

- 자바스크립트 언어의 고유 기능

### `[[Prototype]]`

자바스크립트의 객체는 명세서에 명명한 `[[Prototype]]`이라는 숨김 프로퍼티를 가짐

- 이 숨김 프로퍼티 값은 `null` 혹은 다른 객체에 대한 참조
- 다른 객체를 참조하는 경우, 참조 대상을 프로토타입(prototype)이라 부름
- `[[Prototype]]`이 참조하는 객체는 프로토타입이라 함

```
prototype object
[               ]
        ^
        | [[Prototype]]
object  |
[               ]
```

`object`에서 프로퍼티를 읽으려고 하는데 해당 프로퍼티가 없으면, 자바스크립트는 자동으로 프로토타입에서 프로퍼티를 탐색

- 이런 동작 방식을 프로그래밍에서 `프로토타입 상속`이라 부름
- 언어 차원에서 지원하는 편리한 기능이나 개발 테크닉 중 프로토타입 상속에 기반해 만들어진 것들이 많음

`[[Prototype]]` 프로퍼티

- 내부 프로퍼티
- 숨김 프로퍼티
- 하지만 다양한 방법을 사용해 개발자가 값 설정 가능

`__proto__`

- 특별한 이름
- `__proto__`를 사용해 `[[Prototype]]` 값을 설정할 수 있음
- `__proto__`는 `[[Prototype]]`과 다름
  - `[[Prototype]]`용 getter(획득자)이자 setter(설정자)임
- 하위 호환성 때문에 여전히 `__proto__`를 사용할 수 있지만, 비교적 근래에 작성된 스크립트에서는 `__proto__` 대신 함수 `Object.getPropertyOf`나 `Object.setPropertyOf`를 써서 프로토타입을 획득(get)하거나 설정(set)함
- `__proto__`는 브라우저 환경에서만 지원하도록 자바스크립트 명세서에서 규정함
  - 실상은 서버 사이드를 포함한 모든 호스트 환경에서 `__proto__`를 지원

```javascript
let animal = { eats: true };
let rabbit = { jumps: true };
rabbit.__proto__ = animal; // animal이 rabbit의 프로토타입이 되도록 설정
alert(rabbit.eats); // true
alert(rabbit.jumps); // true
```

- 객체 `rabbit`에서 프로퍼티를 얻고 싶은데 해당 프로퍼티가 없다면, 자바스크립트는 자동으로 `animal`이라는 객체에서 프로퍼티를 얻음

```
animal
[ eats: true  ]
       ^
       | [[Prototype]]
rabbit |
[ jumps: true ]
```

- `rabbit`의 프로토타입은 `animal`
- `rabbit`은 `animal`을 상속받음
- `rabbit`에서도 `animal`에 구현된 유용한 프로퍼티와 메서드를 사용할 수 있음

상속 프로퍼티(inherited property)

- 프로토타입에서 상속받은 프로퍼티

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
rabbit.walk(); // 동물이 걷습니다.
```

길어진 프로토타입 체인

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
longEar.walk(); // 동물이 걷습니다.
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

프로토타입 체이닝의 제약 사항

1. 순환 참조(circular reference) 불가
   - `__proto__`를 이용해 닫힌 형태로 다른 객체를 참조하면 에러 발생
2. `__proto__`의 값은 객체나 `null`만 가능
   - 다른 자료형은 무시됨

객체에는 오직 하나의 `[[Prototype]]`만 있을 수 있음

- 객체는 두 개의 객체를 상속받을 수 없음

### 프로토타입은 읽기 전용이다

프로토타입은 프로퍼티를 읽을 때만 사용

프로퍼티를 추가, 수정, 삭제하는 연산은 객체에 직접 해야 함

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
rabbit.walk(); // 토끼가 깡충깡충 뜁니다.
```

- `rabbit.walk()`를 호출하면 프로토타입에 있는 메서드가 실행되지 않음
  - 객체 `rabbit`에 직접 추가한 메서드가 실행됨

```
animal
[ eats: true     ]
[ walk: function ]
         ^
         | [[Prototype]]
rabbit   |
[ walk: function ]
```

그러나 접근자 프로퍼티(accessor property)는 setter 함수를 사용해 프로퍼티에 값을 할당하므로, 접근자 프로퍼티에 값을 할당(`(**)`)하면 객체(`admin`)에 프로퍼티(`fullName`)가 추가되는 게 아니라 setter 함수가 호출되면서 위 예시와 다르게 동작

```javascript
let user = {
  name: "John",
  sername: "Smith",
  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  },
  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};
let admin = {
  __proto__: user,
  isAdmin: true
};
alert(admin.fullName); // John Smith (*)
admin.fullName = "Alice Cooper"; // (**). setter 함수가 실행됨
alert(admin.fullName); // Alice Cooper. setter에 의해 추가된 admin의 프로퍼티(name, surname)에서 값을 가져옴
alert(user.fullName); // John Smith. 본래 user에 있었던 프로퍼티 값
```

- 프로토타입 `user`에는 getter 함수 `get fullName`이 있기 때문에 `(*)`로 표시한 줄에서는 `get fullName`이 호출됨
- 프로타입에 이미 setter 함수(`set fullName`)가 정의되어 있기 때문에 `(**)`로 표시한 줄의 할당 연산이 실행되면 객체 `user`에 프로퍼티가 추가되는 게 아니라 프로토타입에 있는 setter 함수가 호출됨

### this가 나타내는 것

`set fullName(value)` 본문의 `this`에는 어떤 값이 들어가는가

- 프로퍼티 `this.name`과 `this.surname`에 값을 쓰면 그 값이 `user`에 저장되는지 `admin`에 저장되는지

`this`는 프로토타입에 영향을 받지 않음

- 메서드를 객체에서, 프로토타입에서 호출했는지 상관 없이 `this`는 언제나 `.` 앞에 있는 객체

`admin.fullName=`으로 setter 함수를 호출할 때, `this`는 `user`가 아닌 `admin`이 됨

객체 하나를 만들고 여기에 메서드를 많이 구현해 놓은 다음, 여러 객체에서 이 커다란 객체를 상속받게 하는 경우가 많기 때문에 이런 특징을 잘 알아야 함

상속받은 메서드를 사용하더라도 객체는 프로토타입이 아닌 자신의 상태를 수정

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
[ sleep: function      ]
           ^
           | [[Prototype]]
rabbit     |
[ name: "White Rabbit" ]
[ isSleeping: true     ]
```

- 메서드는 공유되지만 객체의 상태는 공유되지 않음

### for..in 반복문

`for..in`은 상속 프로퍼티도 순회 대상에 포함시킴

```javascript
let animal = { eats: true };
let rabbit = { jumps: true, __proto__: animal };
alert(Object.keys(rabbit)); // jumps
for (let prop in rabbit) alert(prop); // jumps, eats
```

`obj.hasOwnProperty(key)`

- 상속 프로퍼티를 순회 대상에서 제외할 수 있음
- `key`에 대응하는 프로퍼티가 상속 프로퍼티가 아니고 `obj`에 직접 구현되어 있는 프로퍼티일 때만 `true`를 반환

```javascript
let animal = { eats: true };
let rabbit = { jumps: true, __proto__: animal };

for (let prop in rabbit) {
  let isOwn = rabbit.hasOwnProperty(prop);
  if (isOwn) alert(`객체 자신의 프로퍼티: ${prop}`); // jumps
  else alert(`상속 프로퍼티: ${prop}`); // eats
}
```

```
               null
                 ^
                 | [[Prototype]]
Object.prototype |
[ toString: function       ]
[ hasOwnProperty: function ]
[ ...                      ]
              ^
              | [[Prototype]]
animal        |
[ eats: true               ]
              ^
              | [[Prototype]]
rabbit        |
[ jumps: true              ]
```

- `rabbit`은 `animal`을 상속받음
- `animal`은 `Object.prototype`을 상속받음
- `Object.prototype`은 `null`을 상속받음
- `animal`을 객체 리터럴 방식으로 선언했기 때문에 `Object.prototype`을 상속받음
- `hasOwnProperty`는 `Object.prototype.hasOwnProperty`에서 옴
- `hasOwnProperty`의 `enumerable` 플래그가 `false`이기 때문에 열거 불가능한 프로퍼티

키-값을 순회하는 메서드 대부분은 상속 프로퍼티를 제외하고 동작

- `Object.keys`, `Object.values` 같이 객체의 키-값을 대상으로 무언가를 하는 메서드 대부분은 상속 프로퍼티를 제외하고 동작
- 프로토타입에서 상속받은 프로퍼티는 제외하고, 해당 객체에서 정의한 프로퍼티만 연산 대상에 포함

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

- 모던 엔진에서는 객체에서 프로퍼티를 가져오는 것과 프로토타입에서 프로퍼티를 가져오는 것 사이에 성능적인 차이가 없음
- 모던 엔진은 프로퍼티가 어디서 발견됐는지 기억했다가 다음 요청 시 이 정보를 재사용
  - `pocket.glasses` 예
  - 엔진은 `glasses`가 발견된 곳(`head`)을 기억하고 있다가 다음 요청부터는 이 프로퍼티가 발견된 곳에서 검색을 시작
- 모던 엔진은 뭔가 변화가 생기면 내부 캐시를 변경해줄 정도로 똑똑하기 때문에 최적화를 안전하게 수행

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

```javascript
let hamster = {
  stomach: [],
  eat(food) {
    this.stomach.push(food);
  }
};
let speedy = {
  __proto__: hamster
};
let lazy = {
  __proto__: hamster
};

speedy.eat("apple");
console.log(speedy); // {}
console.log(speedy.stomach); // [ "apple" ]
console.log(lazy); // {}
console.log(lazy.stomach); // [ "apple" ]
```

```javascript
let hamster = {
  stomach: [],
  eat(food) {
    this.stomach = [food]; // this.stomach.push(food) 대신 food를 this.stomach에 할당
  }
};
let speedy = {
  __proto__: hamster
};
let lazy = {
  __proto__: hamster
};

speedy.eat("apple");
console.log(speedy); // { stomach: [ "apple" ] }
console.log(speedy.stomach); // [ "apple" ]
console.log(lazy); // {}
console.log(lazy.stomach); // []
```

`speedy.eat("apple")`을 호출하면 생기는 일

1. 메서드 `speedy.eat`은 프로토타입 `hamster`에서 발견되는데, 점 앞에는 객체 `speedy`가 있으므로 `this`에는 `speedy`가 할당되어 메서드가 실행됨
2. `this.stomach.push()`를 실행하면 프로퍼티 `stomach`을 찾아서 여기에 `push`를 호출해야 함. 그런데 `this`인 `speedy`에는 프로퍼티 `stomach`이 없음
3. `stomach`을 찾기 위해 프로토타입 체인을 거슬러 올라가보니 `hamster`에 `stomach`이 있는 것을 발견
4. `push` 메서드는 프로토타입 `hamster`에 있는 `stomach`을 대상으로 동작하여 프로토타입에 `food`가 추가됨

- 모든 햄스터가 하나의 `stomach`을 공유하는 이유는 이런 동작 방식 때문
- `lazy.stomach.push()`, `speedy.stomach.push()`를 호출했을 때 모두 프로퍼티 `stomach`은 프로토타입에서 발견되어 새로운 데이터는 `stomach`에 추가됨
- 문제를 해결하려면 `push` 메서드가 아닌 `this.stomach=`을 사용해 데이터를 할당하면 됨
- `this.stomach=`은 객체 자체에 해당 프로퍼티를 추가하지 프로토타입 체인에서 `stomach`을 찾지 않기 때문에 의도한 대로 잘 동작함
- 햄스터가 각자의 `stomach`을 가지게 해도 문제를 해결할 수 있음

```javascript
let hamster = {
  stomach: [],
  eat(food) {
    this.stomach.push(food);
  }
};
let speedy = {
  __proto__: hamster,
  stomach: []
};
let lazy = {
  __proto__: hamster,
  stomach: []
};

speedy.eat("apple");
console.log(speedy); // { stomach: [ "apple" ] }
console.log(speedy.stomach); // [ "apple" ]
console.log(lazy); // { stomach: [] }
console.log(lazy.stomach); // []
```

- 문제의 `stomach`처럼 특정 객체의 상태를 설명하는 프로퍼티는 조상 객체가 아닌 객체 자체에 정의하는 것이 이런 문제를 차단할 수 있는 일반적인 방법

## 함수의 prototype 프로퍼티

리터럴이 아닌 생성자 함수를 사용해 객체를 만든 경우 프로토타입이 어떻게 동작하는가

생성자 함수로 객체를 만들었을 때 리터럴 방식과 다른 점

- 생성자 함수의 프로토타입이 객체인 경우에 `new` 연산자를 사용해 만든 객체는 생성자 함수의 프로토타입 정보를 사용해 `[[Prototype]]`을 설정함

자바스크립트가 만들어졌을 당시에는 프로토타입 기반 상속이 자바스크립트의 주요 기능 중 하나였음

### 함수의 디폴트 프로퍼티 prototype과 constructor 프로퍼티

## 참고

> [프로토타입과 프로토타입 상속](https://ko.javascript.info/prototypes)
