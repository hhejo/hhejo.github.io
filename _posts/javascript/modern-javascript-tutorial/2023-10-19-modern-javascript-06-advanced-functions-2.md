---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 2
date: 2023-10-19 12:31:06 +0900
last_modified_at: 2023-12-06 10:07:29 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

변수의 유효 범위와 클로저, 오래된 var

## 변수의 유효 범위와 클로저

자바스크립트는 함수 지향 언어

- 함수를 동적으로 생성
- 생성한 함수를 다른 함수에 인수로 전달
- 생성된 곳이 아닌 곳에서 함수 호출

함수 내부에서 함수 외부에 있는 변수에 접근 가능

- 함수가 생성된 이후 외부 변수가 변경되는 경우
  - 함수는 새로운 값을 가져올 것 ( )
  - 생성 시점 이전의 값을 가져올 것 ( )
- 매개변수를 통해 함수를 넘기고, 이 함수를 멀리 떨어진 코드에서 호출하는 경우
  - 함수가 호출된 곳을 기준으로 외부 변수에 접근할 것 ( )

`let`, `const`로 선언한 변수는 동일하게 동작

### 코드 블록

코드 블록 `{...}`

- 블록 안에서 선언한 변수는 블록 안에서만 사용 가능
- 수행하는 코드를 한데 묶음
- 블록 안에는 작업 수행에만 필요한 변수가 들어감
- `if`, `for,` `while`에서도 동일
  - `for (let i = 0; ...)`에서 `let i`는 코드 블록 밖에 있으나, 블록에 속하는 코드로 취급

```javascript
// 1
{
  let message = "안녕하세요.";
  alert(message);
}
{
  let message = "안녕히 가세요."; // 둘 다 블록이 없었다면 에러 발생 (동일한 이름의 변수 선언)
  alert(message);
}

// 2
{
  let message = "안녕하세요";
  alert(message); // 안녕하세요
}
alert(message); // ReferenceError: message is not defined
```

```javascript
if (true) {
  let phrase = "안녕하세요!";
  alert(phrase); // 안녕하세요!
}
alert(phrase); // ReferenceError: phrase is not defined

for (let i = 0; i < 3; i++) {
  alert(i); // 0, 1, 2
}
alert(i); // ReferenceError: i is not defined
```

### 중첩 함수

중첩 함수(nested function)

- 함수 내부에서 선언한 함수
- 새로운 객체의 프로퍼티 형태나 중첩 함수 그 자체로 반환될 수 있음
  - 반환된 중첩 함수는 어디서든 호출해 사용 가능
  - 외부 변수에 접근 가능
- 코드를 정돈하는 데에 사용 가능

```javascript
function sayHiBye(firstName, lastName) {
  function getFullName() {
    return firstName + " " + lastName;
  }
  alert("Hello, " + getFullName());
  alert("Bye, " + getFullName());
}
```

호출될 때마다 다음 숫자를 반환하는 함수

- counter를 여러 개 만들었을 때, 이 함수들은 서로 독립적
- 함수와 중첩 함수 내 count 변수에는 어떤 값이 할당?

```javascript
function makeCounter() {
  let count = 0;
  return function () {
    return count++;
  };
}
let counter = makeCounter();
alert(counter()); // 0
alert(counter()); // 1
alert(counter()); // 2
let counter2 = makeCounter();
alert(counter2()); // 0
```

### 렉시컬 환경

**단계 1. 변수**

렉시컬 환경(Lexical Environment)

- 내부 숨김 연관 객체(internal hidden associated object)
- 자바스크립트에서 렉시컬 환경을 가지는 것들
  - 실행 중인 함수
  - 코드 블록 `{...}`
  - 스크립트 전체
- 명세서에서 자바스크립트가 어떻게 동작하는지 설명하는 데 쓰이는 이론상의 객체
  - 렉시컬 환경은 명세서에만 존재
  - 코드를 사용해 직접 렉시컬 환경을 얻거나 조작하는 것은 불가능
  - 자바스크립트 엔진들은 명세서에 언급된 사항을 준수하면서 엔진 고유의 방법을 사용해 렉시컬 환경 최적화

렉시컬 환경 객체의 구성

1. 환경 레코드(Environment Record)
   - 모든 지역 변수를 프로퍼티로 저장하고 있는 특수 내부 객체
   - `this` 값과 같은 기타 정보도 저장
   - 변수는 환경 레코드의 프로퍼티
   - 변수를 가져오거나 변경하는 것은 환경 레코드의 프로퍼티를 가져오거나 변경하는 것
2. 외부 렉시컬 환경(Outer Lexical Environment)에 대한 참조
   - 외부 코드와 연관

예시 1

```javascript
//////////////////////// Lexical Environment
let phrase = "Hello"; // [ phrase: "Hello" ] -outer-> null
alert(phrase);
```

- 렉시컬 환경이 전역 렉시컬 환경 하나만 존재
  - 전역 렉시컬 환경(Global Lexical Environment)
    - 스크립트 전체와 관련된 렉시컬 환경
- `[ phrase: "Hello" ]`
  - 변수가 저장되는 환경 레코드
- `-outer->`
  - 외부 렉시컬 환경에 대한 참조
  - 전역 렉시컬 환경은 외부 참조를 갖지 않아 `null`을 가리킴
- 코드가 실행되고 실행 흐름이 이어지면서 렉시컬 환경 변화

예시 2

```
// execution start     [ phrase: <uninitialized> ] -outer-> null
let phrase;         // [ phrase: undefined       ]
phrase = "Hello";   // [ phrase: "Hello"         ]
phrase = "Bye";     // [ phrase: "Bye"           ]
```

1. 스크립트가 시작되면 스크립트 내에서 선언한 변수 전체가 렉시컬 환경에 올라감(pre-populated)
   - 이때 변수의 상태는 `uninitialized`가 됨
     - `uninitialized`: 특수 내부 상태(special internal state)
   - 자바스크립트 엔진은 `uninitialized` 상태의 변수를 인지
   - 그러나 `let`을 만나기 전까지는 해당 변수 참조 불가능
2. `let phrase` 나타남
   - 아직 값을 할당하기 전이기 때문에 프로퍼티 값은 `undefined`
   - `phrase`는 이 시점 이후부터 사용 가능
3. `phrase`에 값이 할당됨
4. `phrase`의 값이 변경됨

**단계 2. 함수 선언문**

함수 선언문(function declaration)

- 함수는 변수와 마찬가지로 값
- 그러나 함수 선언문으로 선언한 함수는 일반 변수와는 달리 바로 초기화됨
- 그래서 렉시컬 환경이 만들어지는 즉시 사용 가능
  - 선언되기 전에도 함수 사용 가능
  - 변수는 `let`을 만나 선언이 될 때까지 사용 불가
- 함수 표현식은 해당하지 않음
  - 함수 표현식(function expression): 함수를 변수에 할당

```javascript
// execution start     [ phrase: <uninitialized> ] -outer-> null
//                     [ say: function           ]
let phase = "Hello";
function say(name) {
  alert(`${phrase}, ${name}`);
}
```

**단계 3. 내부와 외부 렉시컬 환경**

함수 호출, 실행

- 새로운 렉시컬 환경 자동 생성
- 호출 중인 함수를 위한 내부 렉시컬 환경
  - 함수 호출시 넘겨받은 매개 변수
  - 함수의 지역 변수
- 내부 렉시컬 환경이 가리키는 외부 렉시컬 환경
  - 외부 렉시컬 환경에 대한 참조

코드에서 변수에 접근할 때

- 내부 렉시컬 환경을 검색 범위로 지정
- 내부 렉시컬 환경에서 원하는 변수를 찾지 못하는 경우
- 내부 렉시컬 환경이 참조하는 외부 렉시컬 환경으로 검색 범위 확장
- 검색 범위가 전역 렉시컬 환경으로 확장될 때까지 반복
  - 엄격 모드: 전역 렉시컬 환경에 도달할 때까지 변수를 찾지 못하면 에러 발생
  - 비 엄격 모드: 정의되지 않은 변수에 값을 할당하려고 하면 에러가 발생하는 대신 새로운 전역 변수가 생성됨
    - 하위 호환성을 위해 남아있는 기능

```javascript
// 외부 렉시컬 환경 (전역 렉시컬 환경)
let phrase = "Hello";
function say(name) {
  // 내부 렉시컬 환경 (함수 say)
  alert(`${phrase}, ${name}`);
}
say("John"); // Hello, John
```

```
Lexical Environment of the call
[ name: "John" ]                  -outer-> [ say: function   ] -outer-> null
                                           [ phrase: "Hello" ]
```

**단계 4. 함수를 반환하는 함수**

```javascript
function makeCounter() {
  let count = 0;
  return function () {
    return count++;
  };
}
let counter = makeCounter();
```

```
[ <empty> ] -outer-> [ count: 0 ] -outer-> [ makeCounter: function ] -outer-> null
                                           [ counter: function     ]
```

- `makeCounter()` 호출할 때마다 새로운 렉시컬 환경 객체가 생성됨
- 여기에 `makeCounter`를 실행하는 데 필요한 변수들 저장됨
- `makeCounter()` 실행
  - 중첩 함수(`function () { return count++; }`) 생성
  - 생성되기만 하고 실행은 되지 않은 상태
- `counter.[[Environment]]`
  - `{ count: 0 }`이 있는 렉시컬 환경에 대한 참조가 저장됨
- `counter()`를 호출하면 각 호출마다 새로운 렉시컬 환경이 생성됨
  - 이 렉시컬 환경은 `counter.[[Environment]]`에 저장된 렉시컬 환경을 외부 렉시컬 환경으로서 참조

`[[Environment]]` 프로퍼티

- 모든 함수는 함수가 생성된 곳의 렉시컬 환경을 기억
- 함수는 `[[Environment]]`를 가짐
- 숨김 프로퍼티
- 함수가 만들어진 곳의 렉시컬 환경에 대한 참조가 저장됨
- 호출 장소와 상관 없이 함수가 자신이 태어난 곳을 기억할 수 있는 이유
- 함수가 생성될 때 딱 한 번 값이 세팅되고 영원히 변하지 않음

실행 흐름이 중첩 함수의 본문으로 넘어올 때 (`count` 변수 필요)

- 익명 중첩 함수`function() { return count++; }`의 자체 렉시컬 환경에서 변수 탐색
  - 지역 변수가 없어 해당 렉시컬 환경이 비어 있음 (`<empty>`)
- `counter()`의 렉시컬 환경이 참조하는 외부 렉시컬 환경 탐색
  - `count`를 탐색하고 발견
- `count++`가 실행되면서 `count` 값이 1 증가
  - 변숫값 갱신은 변수가 저장된 렉시컬 환경에서 이뤄짐
- `counter()`를 여러 번 호출하면 `count` 변수가 증가하는 이유

```
[ <empty> ] -outer-> [ count: 1 ] -outer-> [ makeCounter: function ] -outer-> null
                                           [ counter: function     ]
```

클로저(closure)

- 외부 변수를 기억하고, 이 외부 변수에 접근할 수 있는 함수
- 자바스크립트에서는 모든 함수가 자연스럽게 클로저가 됨
  - 예외도 있음
- 자바스크립트의 함수는 숨김 프로퍼티인 `[[Environment]]`를 이용해 자신이 어디서 만들어졌는지 기억
  - 함수 본문에서는 `[[Environment]]`를 사용해 외부 변수에 접근

클로저가 무엇인가?

- 클로저의 정의
- 자바스크립트에서 모든 함수가 클로저인 이유
- `[[Environment]]` 프로퍼티와 렉시컬 환경의 동작 방식

### 가비지 컬렉션

함수 호출이 끝날 때

- 함수에 대응하는 렉시컬 환경이 메모리에서 제거됨
- 함수와 관련된 변수들 모두 사라짐
- 그래서 함수 호출이 끝나면 관련 변수를 참조할 수 없음
- 자바스크립트에서 모든 객체는 도달 가능한 상태일 때만 메모리에 유지됨

호출이 끝난 후에도 여전히 도달 가능한 중첩 함수가 있는 경우

- 이때는 이 중첩 함수의 `[[Environment]]` 프로퍼티에 외부 함수 렉시컬 환경에 대한 정보가 저장됨
  - 도달 가능한 상태가 됨
- 함수 호출은 끝났지만 렉시컬 환경이 메모리에 유지되는 이유

```javascript
function f() {
  let value = 123;
  return function () {
    alert(value);
  };
}
let g = f(); // g.[[Environment]]에 f() 호출 시 만들어지는 렉시컬 환경 정보가 저장됨
```

이렇게 중첩 함수를 사용할 때 주의할 점

- `f()`를 여러 번 호출하고 그 결과를 어딘가에 저장하는 경우, 호출 시 만들어지는 각 렉시컬 환경 모두가 메모리에 유지됨
- 렉시컬 환경 객체는 다른 객체와 마찬가지로 도달할 수 없을 때 메모리에서 삭제됨
- 해당 렉시컬 환경 객체를 참조하는 중첩 함수가 하나라도 있으면 사라지지 않음
- 중첩 함수가 메모리에서 삭제되고 난 후에야 이를 감싸는 렉시컬 환경(그리고 그 안의 변수 `value`)도 메모리에서 제거됨

```javascript
function f() {
  let value = Math.random();
  return function () {
    alert(value);
  };
}
let arr = [f(), f(), f()]; // 3개의 렉시컬 환경이 만들어지고, 각 렉시컬 환경은 메모리에서 삭제되지 않음
let g = f(); // g가 살아있는 동안에는 연관 렉시컬 환경도 메모리에 유지됨
g = null; // 도달할 수 없는 상태가 되었으므로 메모리에서 삭제됨
```

최적화 프로세스

- 함수가 살아있는 동안엔 이론상으론 모든 외부 변수 역시 메모리에 유지됨
- 실제로는 자바스크립트 엔진이 이를 지속해서 최적화
- 자바스크립트 엔진은 변수 사용을 분석하고 외부 변수가 사용되지 않는다고 판단되면 이를 메모리에서 제거
- 디버깅 시 최적화 과정에서 제거된 변수를 사용할 수 없다는 점은 V8 엔진의 주요 부작용

```javascript
function f() {
  let value = Math.random();
  function g() {
    debugger; // Uncaught ReferenceError: value is not defined가 출력됨
  }
  return g;
}
let g = f();
g();
```

- 이론상으로는 `value`에 접근할 수 있어야 하지만 최적화 대상이 되어서 에러 발생

```javascript
let value = "이름이 같은 다른 변수";
function f() {
  let value = "가장 가까운 변수";
  function g() {
    debugger; // 콘솔에 alert(value);를 입력하면 "이름이 같은 다른 변수"가 출력됨
  }
  return g;
}
let g = f();
g();
```

### 예제

```javascript
let name = "John";
function sayHi() {
  alert("Hi, " + name);
}
name = "Pete";
sayHi(); // Hi, Pete
```

- 함수는 현재 상태 그대로 외부 변수를 가져오며 가장 최근 값을 사용
- 이전 변수 값은 어디에도 저장되지 않음
- 함수가 변수를 원하면 자체 렉시컬 환경이나 외부 렉시컬 환경에서 현재 값을 가져옴

```javascript
function makeWorker() {
  let name = "Pete";
  return function () {
    alert(name);
  };
}
let name = "John";
let work = makeWorker();
work(); // Pete
```

- `let name = "Pete"`가 없었다면 `let name = "John"`을 사용

```javascript
function makeCounter() {
  let count = 0;
  return function () {
    return count++;
  };
}
let counter = makeCounter();
let counter2 = makeCounter();

alert(counter()); // 0
alert(counter()); // 1

alert(counter2()); // 0
alert(counter2()); // 1
```

- 두 함수는 각각 다른 `makeCounter` 호출에 의해 생성됨
- 두 함수는 독립적인 렉시컬 환경을 가짐
  - 각 함수는 자신만의 `count`를 가짐

```javascript
function Counter() {
  let count = 0; // this.count = 0;
  this.up = function () {
    return ++count; // return ++this.count;
  };
  this.down = function () {
    return --count; // return --this.count;
  };
}
let counter = new Counter();
alert(counter.up()); // 1
alert(counter.up()); // 2
alert(counter.down()); // 1
alert(counter.count); // undefined
```

- 생성자 함수의 두 중첩 함수는 동일한 외부 렉시컬 환경에서 생성됨
  - 같은 `count` 변수를 공유
- `count` 변수에 직접 접근할 수 없음

```javascript
"use strict";
let phrase = "Hello";
if (true) {
  let user = "John";
  function sayHi() {
    alert(`${phrase}, ${user}`);
  }
}
sayHi(); // ReferenceError: sayHi is not defined
```

- `sayHi`는 `if`문 안에서 정의했기 때문에, 그 안에서만 접근 가능
- 엄격 모드가 아니면 잘 출력됨

```javascript
function sum(a) {
  return function (b) {
    return a + b; // a는 외부 렉시컬 환경에서 가져옴
  };
}
alert(sum(1)(2)); // 3
alert(sum(5)(-1)); // 4
```

- 두 번째 괄호가 동작하려면 첫 번째 괄호는 반드시 함수를 반환해야 함

```javascript
let x = 1;
function func() {
  console.log(x); // 지역 변수 x는 함수 시작부터 엔진이 알고 있으나 let을 만나기 전까지 uninitialized(unusable) 상태
  let x = 2; // 이게 없어지면 1이 출력됨
}
func(); // ReferenceError: Cannot access 'x' before initialization
```

- 존재하지 않는 변수와 초기화되지 않은 변수의 차이
- 변수는 실행이 코드 블록(또는 함수)에 진입하는 순간부터 초기화되지 않은 상태로 시작
- 해당 `let` 문이 나올 때까지 초기화되지 않은 상태로 유지
  - 기술적으로는 존재하지만 이전에는 사용할 수 없는 변수

데드 존(dead zone)

- 변수를 일시적으로 사용할 수 없는 영역
- 코드 블록의 시작 부분부터 `let` 까지

```javascript
let arr = [1, 2, 3, 4, 5, 6, 7];
function inBetween(a, b) {
  return function (x) {
    return x >= a && x <= b;
  };
}
function inArray(arr) {
  return function (x) {
    return arr.includes(x);
  };
}
alert(arr.filter(inBetween(3, 6))); // 3,4,5,6
alert(arr.filter(inArray([1, 2, 10]))); // 1,2
```

```javascript
let users = [
  { name: "John", age: 20, surname: "Johnson" },
  { name: "Pete", age: 18, surname: "Peterson" },
  { name: "Ann", age: 19, surname: "Hathaway" }
];
// 이름을 기준으로 정렬(Ann, John, Pete)
// users.sort((a, b) => (a.name > b.name ? 1 : -1));
// 나이를 기준으로 정렬(Pete, Ann, John)
// users.sort((a, b) => (a.age > b.age ? 1 : -1));
function byField(fieldName) {
  return (a, b) => (a[fieldName] > b[fieldName] ? 1 : -1);
}
users.sort(byField("name"));
users.sort(byField("age"));
```

```javascript
function makeArmy() {
  let shooters = [];
  let i = 0;
  while (i < 10) {
    // shooter 함수
    let shooter = function () {
      alert(i); // 몇 번째 shooter인지 출력해줘야 함
    };
    shooters.push(shooter);
    i++;
  }
  return shooters;
}

let army = makeArmy();
army[0](); // 0번째 shooter가 10을 출력함
army[5](); // 5번째 shooter 역시 10을 출력함
// 모든 shooter가 자신의 번호 대신 10을 출력하고 있음
```

```
let shooters = [];
shooters = [
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); },
  function () { alert(i); }
];
```

- 함수 내부에 지역 변수 `i` 가 없기 때문
- 함수가 호출되면 외부 렉시컬 환경에서 `i`를 가져옴
- `i`의 마지막 값은 `10`

```javascript
function makeArmy() {
  let shooters = [];
  for (let i = 0; i < 10; i++) {
    let shooter = function () {
      alert(i);
    };
    shooters.push(shooter);
  }
  return shooter;
}
let army = makeArmy();
army[0](); // 0
army[5](); // 5
```

- 코드 블록 `for (let i = 0; i < 10; i++) {...}`이 실행될 때마다 해당 변수 `i`를 사용하여 새로운 렉시컬 환경이 생성되므로 올바르게 작동

```
shooters = [                 // for block LexicalEnvironment
  function () { alert(i); }, // [i: 0]
  function () { alert(i); }, // [i: 1]                                makeArmy() LexicalEnvironment
  function () { alert(i); }, // [i: 2]                       -outer->
  ...,
  function () { alert(i); }, // [i: 10]
]
```

```javascript
function makeArmy() {
  let shooters = [];
  let i = 0;
  while (i < 10) {
    let j = i; // 값 복사. 현재 루프 반복에 속하는 완전한 독립 복사본
    let shooter = function () {
      alert(j);
    };
    shooters.push(shooter);
    i++;
  }
  return shooters;
}
let army = makeArmy();
army[0](); // 0
army[5](); // 5
```

## 오래된 var

`var`

- `let`으로 선언한 변수와 유사
  - 대부분의 경우에 `let`을 `var`로, `var`를 `let`으로 바꿔도 큰 문제 없이 동작
- `var`는 `let`과 `const`로 선언한 변수와는 다른 방식으로 동작
  - 초기 자바스크립트 구현 방식 때문

### var는 블록 스코프가 없음

`var`로 선언한 변수의 스코프

- 함수 스코프
- 전역 스코프
- 블록 기준으로 스코프가 생기지 않음
  - 블록 밖에서 접근 가능
  - 코드 블록 무시
  - 루프 수준의 스코프 무시

`var`는 구식 자바스크립트의 잔재

- 아주 오래 전의 자바스크립트에서는 블록 수준 렉시컬 환경이 만들어지지 않았음

```javascript
if (true) {
  var testVar = true; // 코드 블록 무시. testVar는 전역 변수가 됨
}
alert(testVar); // true

if (true) {
  let testLet = true; // test 변수는 블록 내에서만 접근 가능
}
alert(testLet); // ReferenceError: test is not defined

for (var i = 0; i < 10; i++) {} // 루프 수준의 스코프를 형성하지 않음
alert(i); // 10
```

```javascript
function sayHi() {
  if (true) {
    var phrase = "Hello"; // 함수 레벨 변수 phrase
  }
  alert(phrase);
}
sayHi(); // Hello
alert(phrase); // ReferenceError: phrase is not defined
```

### var는 변수의 중복 선언을 허용

변수의 중복 선언

- `let`: 한 스코프에서 같은 변수를 두 번 선언하면 에러 발생
- `var`: 같은 변수를 여러 번 중복으로 선언 가능
  - 이미 선언된 변수에 `var`를 사용하면 두 번째 선언문은 무시됨

```javascript
let letUser;
let letUser; // SyntaxError: Identifier 'letUser' has already been declared

var varUser = "Pete";
var varUser = "John"; // var는 아무것도 하지 않음
alert(varUser); // John
```

### 선언하기 전 사용할 수 있는 var

`var` 선언

- 모든 `var` 선언은 함수가 시작될 때 처리됨
- 전역에서 생성한 변수는 스크립트가 시작될 때 처리됨
- 함수 본문 내에서 `var`로 선언한 변수는 선언 위치와 상관 없이 함수 본문이 시작되는 지점에서 정의됨
  - 변수가 중첩 함수 내에서 정의되지 않아야 이 규칙이 적용됨

`var`로 선언한 변수

- 어디서든 참조 가능
- 변수에 무언가를 할당하기 전까지는 값이 `undefined`

아래 코드는 동일하게 동작

```javascript
// 1
function sayHi() {
  phrase = "Hello";
  alert(phrase);
  var phrase; // 위로 이동한 것처럼 동작
}
sayHi(); // Hello

// 2
function sayHi() {
  var phrase;
  phrase = "Hello";
  alert(phrase);
}
sayHi(); // Hello

// 3
function sayHi() {
  phrase = "Hello"; // if 내부의 var는 함수 sayHi의 시작 부분에서 처리되므로 phrase는 이미 정의된 상태
  if (false) {
    var phrase; // if 블록 안 코드는 절대 실행되지 않지만 호이스팅에 영향을 주지 않음
  }
  alert(phrase);
}
sayHi(); // Hello
```

호이스팅(hoisting)

- 변수가 끌어올려지는 현상
- `var`로 선언한 모든 변수는 함수의 최상위로 끌어올려지기(hoisted) 때문
- 할당은 호이스팅 되지 않음
- 선언은 호이스팅 됨

```javascript
// 1
function sayHi() {
  alert(phrase);
  var phrase = "Hello"; // 변수 선언(var), 변수에 값 할당(=)
}
sayHi(); // undefined. 할당은 호이스팅 되지 않음

// 2
function sayHi() {
  var phrase;
  alert(phrase);
  phrase = "Hello";
}
sayHi(); // undefined
```

- 변수 선언은 함수 실행이 시작될 때 처리됨(호이스팅)
- 할당은 호이스팅 되지 않음
  - 할당 관련 코드에서 처리됨
- 1번 코드는 2번 코드처럼 동작

즉시 실행 함수 표현식(immediately-invoked function expressions, `IIFE`)

- `var`도 블록 레벨 스코프를 갖게 함
- 함수를 괄호로 감싸면 자바스크립트가 함수 선언문이 아닌 표현식으로 인식
  - 함수 표현식은 이름이 없어도 괜찮고 즉시 호출도 가능
- 과거에 `var`만 사용 가능했을 때 고안된 방법

```javascript
(function () {
  let message = "Hello";
  alert(message);
})();
```

자바스크립트가 함수 표현식이라고 인식하게 해주는 방법들

- IIFE를 만드는 방법들

```
(function () {
  alert("함수를 괄호로 둘러싸기");
})();

(function () {
  alert("전체를 괄호로 둘러싸기");
}());

!function () {
  alert("표현식 앞에 비트 NOT 연산자 붙이기");
}();

+function () {
  alert("표현식 앞에 단항 덧셈 연산자 붙이기");
}();
```

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
