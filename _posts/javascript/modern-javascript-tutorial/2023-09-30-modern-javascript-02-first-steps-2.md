---
title: 모던 JavaScript 튜토리얼 02 - 자바스크립트 기본 2
date: 2023-09-30 09:56:48 +0900
last_modified_at: 2024-01-17 13:14:20 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    variable,
    constant,
    data-type,
    typeof,
    alert,
    prompt,
    confirm,
    type-conversion,
    operator,
    if-else,
    nullish,
    while,
    for,
    switch,
    function
  ]
---

변수와 상수, 자료형, alert, prompt, confirm을 이용한 상호작용, 형 변환, 기본 연산자와 수학, 비교 연산자, if와 ?를 사용한 조건 처리, 논리 연산자, nullish 병합 연산자 ??, while과 for 반복문, switch문, 함수, 함수 표현식, 화살표 함수 기본

## 변수와 상수

### 변수(variable)

데이터를 저장할 때 쓰이는 이름이 붙은 저장소

`let`

- `let` 키워드로 변수 생성
- `$`, `_`도 변수명 가능
- 같은 변수를 여러 번 선언하면 에러 발생

```javascript
let varName = 1;
let message = "Hello, world!";
let $ = 1,
  _ = 2;
alert($ + _); // 3
```

`var`

- `var` 키워드로도 변수 생성
- 오래된 방식

`use strict`가 없으면 값을 할당해 변수를 생성할 수 있음

```javascript
num = 5;
alert(num); // 5
```

```javascript
"use strict";
num = 5; // error: num is not defined
```

### 상수

- `const` 키워드로 상수 생성
- 대문자 상수: 하드 코딩한 값의 별칭을 만들 때 활용

```javascript
const pageLoadTime = 0.2;
const COLOR_ORANGE = "#FF7F00";
```

## 자료형

동적 타입(dynamically typed) 언어

- 자바스크립트는 동적 타입 언어
- 자료의 타입은 있지만 변수에 저장되는 값의 타입은 언제든지 바꿀 수 있음
- 자바스크립트의 변수는 자료형에 관계 없이 모든 자료형일 수 있음

자바스크립트의 8가지 기본 자료형

- Number, BigInt, String, Boolean, Null, Undefined, Symbol
- Object

### 숫자형(Number)

- 정수 및 부동소수점(floating point number) 숫자를 나타냄
- 숫자형: `-(2^53 - 1)` ~ `2^53 - 1` 표현 가능
  - `-9,007,199,254,740,991` ~ `9,007,199,254,740,991`
- 특수 문자 값(special numeric value) 포함
  - `Infinity`, `-Infinity`, `NaN`
- 자바스크립트에서는 말이 안 되는 수학 연산을 해도 스크립트가 에러를 발생시키지 않고 `NaN`을 반환하며 연산 종료

```javascript
let num = 123;
num = 1.2345;
let inf = 1 / 0; // Infinity
let notNum = "숫자가 아님" - 2; // NaN
alert("Hello" - 2); // NaN
```

### BigInt

- 정수 리터럴 끝에 `n`을 붙임
- 길이에 상관 없이 정수 표현 가능

```javascript
const bigInt = 1234567890123456789012345678901234567890n;
```

### 문자형(String)

- 큰따옴표, 작은따옴표, 백틱으로 묶음
- 큰따옴표와 작은따옴표는 차이 없음
- 백틱으로 묶고 `${...}` 안에 원하는 변수나 표현식을 문자열 안에 삽입 가능
- 자바스크립트는 글자형을 지원하지 않음
  - 글자형: 글자(character) 하나를 저장할 때 쓰이는 자료형

```javascript
let name = "John";
let x = `1 + 1 = ${1 + 1}`;
```

### 불린형(Boolean)

- `true`, `false` 두 가지 값만 가짐
- 비교 결과를 저장할 때도 사용

```javascript
let nameFieldChecked = true;
let ageFieldChecked = false;
let isGreater = 4 > 1; // true
```

### null 값

- 존재하지 않는(nothing) 값, 비어 있는(empty) 값, 알 수 없는(unknown) 값을 나타냄
- 자바스크립트가 아닌 다른 언어에서는 존재하지 않는 객체에 대한 참조나 널 포인터(null pointer)를 나타낼 때 사용

```javascript
let age = null; // 나이를 알 수 없거나, 그 값이 비어있음
```

### undefined 값

- 값이 할당되지 않은 상태를 나타냄
- 변수는 선언했지만 값을 할당하지 않았다면 자동으로 `undefined` 할당
- 비어있거나 알 수 없는 상태라는 것을 명시적으로 나타내려면 `null` 권장

```javascript
let age; // undefined
```

### 객체와 심볼(Object, Symbol)

원시(primitive) 자료형

- 객체형을 제외한 다른 자료형
- 문자열이든 숫자든 한 가지만 표현 가능

객체(object)형

- 데이터 컬렉션, 복잡한 개체(entity) 표현 가능

심볼(symbol)형

- 객체의 고유한 식별자(unique identifier)를 만들 때 사용

### typeof 연산자

- 인수의 자료형을 반환
- 두 가지 형태의 문법 존재
  - `typeof x`: 연산자 형태
  - `typeof(x)`: 함수 형태

```javascript
typeof Math; // object (1)
typeof null; // object (2)
typeof alert; // function (3)
```

1. Math는 수학 연산을 제공하는 내장 객체
2. null은 고유한 자료형을 가지는 특수 값으로 객체가 아니지만, 하위호환성을 유지하기 위해 오류를 수정하지 않음
3. typeof는 피연산자가 함수면 function을 반환하는데, 함수형은 따로 없고 함수는 객체형에 속함

## alert, prompt, confirm을 이용한 상호작용

- 브라우저 환경에서 사용되는 최소한의 사용자 인터페이스 기능
- `alert`, `prompt`, `confirm`

### alert

```javascript
alert("Hello");
```

- 사용자가 확인 버튼을 누를 때까지 메시지를 보여주는 창을 계속 띄움
- 모달(modal) 창: 메시지가 있는 작은 창

### prompt

```javascript
result = prompt(title, [default]);
```

- `title`: 사용자에게 보여줄 문자열
- `default`: 입력 필드의 초깃값
  - 대괄호는 필수가 아닌 선택값의 의미
- 사용자가 입력을 취소한 경우 `null` 반환

### confirm

```javascript
result = confirm(question);
```

- `question`: 질문 문자열
- 사용자가 확인 버튼을 누르면 `true`, 그 외의 경우는 `false` 반환

## 형 변환 (type conversion)

- 함수와 연산자에 전달되는 값은 대부분 적절한 자료형으로 자동 변환됨
- 혹은 전달받은 값을 의도를 갖고 원하는 타입으로 변환(명시적 변환)하는 것
- 문자형, 숫자형, 불린형으로의 변환은 자주 일어나는 형 변환

### 문자형으로 변환

- 문자형의 값이 필요할 때 발생
  - `alert`는 인수로 문자형을 받는데 다른 형의 값을 전달받으면 자동으로 변환
- `String(value)` 함수로 직접 변환

```javascript
let value = true;
alert(value); // true
value = String(value); // "true"
```

### 숫자형으로 변환

- 수학과 관련된 함수와 표현식에서 자동 발생
- `Number(value)` 함수로 직접 변환

```javascript
let num = "6" / "2";
alert(num); // 3
num = Number("123"); // 123
```

|     값      |                             결과                             |
| :---------: | :----------------------------------------------------------: |
| `undefined` |                            `NaN`                             |
|   `null`    |                             `0`                              |
|   `true`    |                             `1`                              |
|   `false`   |                             `0`                              |
|  `string`   | 문자열의 처음과 끝 공백 제거 후 남아있는 문자열이 없다면 `0` |
|             |             그렇지 않다면 문자열에서 숫자를 읽음             |
|             |                    변환에 실패하면 `NaN`                     |
|             | 숫자 이외의 글자가 들어간 문자열을 숫자형으로 변환하면 `NaN` |

### 불린형으로 변환

- 논리 연산을 수행할 때 발생
- `Boolean(value)` 함수로 직접 변환

|                     값                      |  결과   |
| :-----------------------------------------: | :-----: |
| `0`, `빈 문자열`, `null`,`undefined`, `NaN` | `false` |
|                  이외의 값                  | `true`  |

```javascript
alert(Boolean("")); // false
alert(Boolean(" ")); // true
alert(Boolean(0)); // false
alert(Boolean("0")); // true
```

## 기본 연산자와 수학

피연산자(operand)

- 연산자가 연산을 수행하는 대상
- 인수(argument)로 불리기도 함

단항(unary) 연산자

- 피연산자를 하나만 받는 연산자

이항(binary) 연산자

- 피연산자를 두 개를 받는 연산자

자바스크립트에서 지원하는 수학 연산자

- `+`, `-`, `*`, `/`, `%`, `**`

### 이항 연산자 +와 문자열 연결

이항 연산자 `+`의 피연산자로 문자열이 전달되는 경우

- 덧셈이 아닌 문자열을 병합(연결)
- 피연산자 중 하나가 문자열이면 다른 하나도 문자열로 변환

다른 산술 연산자는 오직 숫자형의 피연산자만 다룸

- 피연산자가 숫자형이 아닌 경우 그 형을 숫자형으로 변환

```javascript
alert("1" + 2); // 12
alert(2 + "1"); // 21
alert(2 + 2 + "1"); // 41
alert(6 - "2"); // 4
alert("6" / "2"); // 3
```

### 단항 연산자 +와 숫자형으로의 변환

덧셈 연산자 `+`는 이항 연산자 뿐만 아니라 단항 연산자로도 활용

- 피연산자가 숫자가 아닌 경우 숫자형으로 변환

```javascript
alert(+true); // 1
alert(+""); // 0
let apples = "2",
  oranges = "3";
alert(apples + oranges); // 23
alert(+apples + +oranges); // 5
```

### 연산자 우선순위

하나의 표현식에 둘 이상의 연산자가 있는 경우, 실행 순서는 연산자의 우선순위(precedence)에 의해 결정

### 할당 연산자

할당(assignment) 연산자 `=`

- `x = value`를 호출하면 `value`가 `x`에 쓰이고 `value` 반환
- 여러 개를 연결할 수 있음(체이닝)

```javascript
let a = 1;
let b = 2;
let c = 3 - (a = b + 1);
alert(a); // 3
alert(c); // 0

let x, y, z;
x = y = z = 2 + 2; // x, y, z는 4
```

### 복합 할당 연산자

- 변수에 연산자를 적용하고 그 결과를 같은 변수에 저장
- `+=`, `*=`, `-=`, `/=`

### 증가·감소 연산자

- 증가(increment) 연산자 `++`, 감소(decrement) 연산자 `--`
- 전위형, 후위형으로 나뉨

### 비트 연산자

- bitwise operator
- 인수를 32비트 정수로 변환해 이진 연산 수행
- `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`

### 쉼표 연산자

- comma operator `,`
- 여러 표현식을 한 줄에 평가할 수 있게 하고 마지막 표현식의 평가 결과만 반환

```javascript
let a = (1 + 2, 3 + 4); // 7
```

## 비교 연산자

### 불린형 반환

비교 연산자는 불린형 값을 반환

```javascript
alert(2 > 1); // true
alert(2 == 1); // false
alert(2 != 1); // true
```

### 문자열 비교

사전순으로 문자열 비교(유니코드 순)

```javascript
alert("Z" > "A"); // true
alert("Glow" > "Glee"); // true
alert("Bee" > "Be"); // true
```

### 다른 형을 가진 값 간의 비교

비교하려는 값들의 자료형이 다르면 자바스크립트는 이 값들을 숫자형으로 변환

```javascript
alert("2" > 1); // true
alert("01" == 1); // true
alert(true == 1); // true
alert(false == 0); // true
```

주의

```javascript
let a = 0;
alert(Boolean(a)); // false
let b = "0";
alert(Boolean(b)); // true
alert(a == b); // true
```

### 일치 연산자

동등 연산자(equality operator) `==`

- `0`과 `flase` 구별 불가

일치 연산자(strict equality operator) `===`

- 형 변환 없이 값 비교
- 비교할 때는 일치 연산자 `===` 권장

```javascript
alert(0 == false); // true
alert("" == false); // true
alert(0 === false); // false
```

### null이나 undefined와 비교하기

일치 연산자 `===`로 비교

- 두 값의 자료형이 다르기 때문에 `false` 반환

동등 연산자 `==`로 비교

- 특별한 규칙이 적용돼 `true` 반환

산술 연산자나 기타 비교 연산자 `>`, `<`, `>=`, `<=`로 비교

- `null` -> `0`
- `undefined` -> `NaN`

```javascript
alert(null === undefined); // false
alert(null == undefined); // true
```

### null vs 0

```javascript
alert(null > 0); // false (1)
alert(null == 0); // false (2)
alert(null >= 0); // true (3)
```

`==`와 `<`, `>`, `<=`, `>=`의 동작 방식이 다르기 때문

1. `null`이 숫자형으로 변환되어 `0`이 됨
2. `==`은 피연산자가 `undefined`나 `null`일 때 형 변환을 하지 않음
   - `undefined`와 `null`을 비교하는 경우에만 `true`를 반환
   - 그 이외의 경우(`null`이나 `undefined`를 다른 값과 비교할 때)는 무조건 `false`를 반환
3. `null`이 숫자형으로 변환되어 `0`이 됨

### 비교가 불가능한 undefined

```javascript
alert(undefined > 0); // false (1)
alert(undefined < 0); // false (2)
alert(undefined == 0); // false (3)
```

1. `undefined`가 `NaN`(숫자형)으로 변환되고, `NaN`이 피연산자인 경우 비교 연산자는 항상 `false`를 반환
2. `undefined`가 `NaN`(숫자형)으로 변환되고, `NaN`이 피연산자인 경우 비교 연산자는 항상 `false`를 반환
3. `undefined`는 `null`이나 `undefined`와 같고, 그 이외의 값과는 같지 않기 때문에 `false`를 반환

## if와 '?'를 사용한 조건 처리

### if문

`if(...)`문의 괄호 안의 조건식의 결과가 `true`이면 코드 블록 실행

```javascript
if () {
  // ...
}
```

### 불린형으로의 변환

falsy 값

- `0`, `""`, `null`, `undefined`, `NaN`

truthy 값

- 이외의 값들

### else절

```javascript
if () {
  // ...
} else {
  // ...
}
```

### else if로 복수 조건 처리하기

```javascript
if () {
  // ...
} else if () {
  // ...
} else {
  // ...
}
```

### 조건부 연산자 '?'

자바스크립트의 유일한 삼항(ternary) 연산자

```javascript
let result = condition ? value1 : value2;
```

## 논리 연산자

- `||`, `&&`, `!`
- `if`를 `||`나 `&&`로 대체 가능
- `!!`로 값을 불린형으로 변경 가능

```javascript
let x = 1;
x > 0 && alert("0보다 크다");

alert(!!null); // false
alert(!!"non-empty string"); // true
```

단락 평가(short circuit evaluation)

## nullish 병합 연산자 '??'

`a ?? b`

- nullish 병합 연산자 (nullish coalescing operator) `??`
- 여러 피연산자 중 값이 확정되어 있는 변수를 찾을 수 있음
- a가 `null`도 아니고 `undefined`도 아니면 a
- 그 외의 경우는 b
- 즉, a가 `null`이거나 `undefined`이면 b를 얻음

```javascript
let x = a !== null && a !== undefined ? a : b;
let y = a ?? b;

let firstName = null;
let lastName = null;
let nickname = "바이올렛";
alert(firstName ?? lastName ?? nickName ?? "익명의 사용자"); // 바이올렛
```

### '??'와 '||'의 차이

- `||`는 첫 번째 truthy 값 반환
- `??`는 첫 번째 정의된(defined) 값을 반환
- `null`, `undefined`, `0`을 구분해 다룰 때 중요
- `0`이 할당될 수 있는 변수를 사용할 땐 `||`보다 `??`이 적합

```javascript
height = height ?? 100; // height에 값이 정의되지 않았으면 100 할당

let width = 0;
alert(width || 100); // 100
alert(width ?? 100); // 0
```

## while과 for 반복문

### 'do...while' 반복문

```javascript
do {
  // ...
} while (condition);
```

### 'for' 반복문

```javascript
for (begin; condition; step) {
  // ...
}
```

### break/continue와 레이블

```javascript
labelName: for (...) {
  ...
}
```

```javascript
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (condition) break outer; // 한번에 2중 반복문 빠져나오기
  }
}
```

```javascript
outer: // 레이블을 별도의 줄에 쓸 수도 있음
for (condition) {
  ...
}
```

## switch문

break문을 만날 때까지 실행함에 주의

```javascript
switch (condition) {
  case value1:
    ...
    [break]
  case value2:
    ...
    [break]
  default:
    ...
    [break]
}
```

## 함수

프로그램을 구성하는 주요 구성 요소(building block)

### 함수 선언(function declaration)

```javascript
function name(parameter1, ...) {
  ...
}
```

- 함수 호출시 매개변수에 인수를 전달하지 않으면 그 값은 `undefined`가 됨
- 원하지 않는다면 기본값(default value)를 설정하면 됨
- return문이 없거나 return 지시자만 있는 함수는 `undefined`를 반환

```javascript
function showMessage(text = "no text given") {} // 기본값이 설정됨
```

## 함수 표현식(function expression)

자바스크립트는 함수를 특별한 종류의 값(동작을 나타내는 값)으로 취급

- 다른 언어에서처럼 특별한 동작을 하는 구조로 취급되지 않음
- 함수를 변수에 할당할 수 있음
- 함수 선언(function declaration), 함수 표현식(function expression)

```javascript
let sayHi = function() {
  ...
};
```

### 콜백 함수

함수를 함수의 인수로 전달하고, 필요하다면 인수로 전달한 그 함수를 나중에 호출(called back)하는 것이 콜백 함수의 개념

```javascript
function ask(question, yes, no) {
  if (confirm(question)) yes();
  else no();
}
// 콜백 함수(콜백)
function showOk() {
  alert("동의하셨습니다.");
}
// 콜백 함수(콜백)
function showCancel() {
  alert("취소 버튼을 누르셨습니다.");
}
ask("동의하십니까?", showOk, showCancel); // 함수의 인수로 함수를 전달
```

익명 함수(anonymous function)

- 이름 없이 선언한 함수
- 함수 표현식에 사용 가능

```javascript
function() {
  // ...
}
```

```javascript
ask(
  "동의하십니까?",
  function () {
    alert("동의하셨습니다.");
  },
  function () {
    alert("취소 버튼을 누르셨습니다.");
  }
);
```

### 함수 표현식 vs 함수 선언문

함수 표현식

- 실제 실행 흐름이 해당 함수에 도달했을 때 함수를 생성

함수 선언문

- 함수 선언문이 정의되기 전에도 호출 가능
- 엄격 모드에서 함수 선언문이 코드 블록 내에 위치하면 해당 함수는 블록 내 어디서든 접근 가능
- 블록 밖에서는 접근 불가

## 화살표 함수(arrow function)

```javascript
let func = (arg1, arg2, ...argN) => expression;

let func2 = function (arg1, arg2, ...argN) {
  return expression;
};
```

```javascript
let sum = (a, b) => a + b;
let double = (n) => n * 2;
let sayHi = () => alert("안녕하세요!");
let sum = (a, b) => {
  result = a + b;
  return result;
};
```

## 참고

> [자바스크립트 기본](https://ko.javascript.info/first-steps)
