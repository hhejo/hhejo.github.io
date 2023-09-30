---
title: 모던 JavaScript 튜토리얼 02 - 자바스크립트 기본 2
date: 2023-09-30 09:56:48 +0900
last_modified_at: 2023-09-30 09:56:48 +0900
categories: [JavaScript]
tags: [javascript]
---

변수, 상수, 자료형, 브라우저 상호작용, 형 변환, 연산자, 반복문, switch문, 함수

## 변수

- `let` 키워드로 변수 생성
- `var`는 오래된 방식
- `$`, `_`도 변수명으로 사용 가능

## 상수

`const` 키워드로 상수 생성

## 자료형

자바스크립트에는 8가지 기본 자료형이 있음

### 숫자형

- 정수 및 부동소수점 숫자를 나타냄
- 특수 문자 값(special numeric value) `Infinity`, `-Infinity`, `NaN` 포함
- 숫자형으로 `-(2^53 - 1)` ~ `2^53 - 1`을 나타낼 수 있음

### BigInt

- 길이에 상관 없이 정수를 나타낼 수 있음
- 정수 리터를 끝에 `n`을 붙임

```javascript
const bigInt = 1234567890123456789012345678901234567890n;
```

### 문자형

- 큰따옴표, 작은따옴표, 백틱으로 묶음
- 백틱으로 묶고 `${...}` 안에 원하는 변수나 표현식을 문자열 안에 쉽게 넣을 수 있음

```javascript
let x = `1 + 1 = ${1 + 1}`;
```

### 불린형

`true`, `false` 두 가지 값만 가짐

### null 값

존재하지 않는 값, 비어 있는 값, 알 수 없는 값을 나타냄

### undefined 값

- 값이 할당되지 않은 상태를 나타냄
- 변수는 선언했지만, 값을 할당하지 않았다면 자동으로 `undefined`가 할당됨
- 비어있거나 알 수 없는 상태라는 것을 명시적으로 나타내려면 `null` 권장

```javascript
let age; // undefined
```

### 객체와 심볼

- 객체(object)형을 제외한 다른 자료형은 원시(primitive) 자료형
- 심볼(symbol)형은 객체의 고유한 식별자(unique identifier)를 만들 때 사용

### typeof 연산자

- 인수의 자료형을 반환
- 연산자 형태: `typeof x`
- 함수 형태: `typeof(x)`

```javascript
typeof Math; // object
typeof null; // object
typeof alert; // function
```

1. Math는 수학 연산을 제공하는 내장 객체
2. null은 고유한 자료형을 가지는 특수 값으로 객체가 아니지만, 하위호환성을 유지하기 위해 오류를 수정하지 않음
3. typeof는 피연산자가 함수면 function을 반환하는데 함수형은 따로 없음

## 상호작용

브라우저 환경에서 사용되는 최소한의 사용자 인터페이스 기능 `alert`, `prompt`, `confirm`

### alert

사용자가 확인 버튼을 누를 때까지 메시지를 보여주는 창을 계속 띄움

```javascript
alert("Hello");
```

메시지가 있는 작은 창을 모달(modal) 창이라 함

### prompt

```javascript
result = prompt(title, [default]);
```

- `title`: 사용자에게 보여줄 문자열
- `default`: 입력 필드의 초깃값
- 사용자가 입력을 취소한 경우 `null`을 반환

### confirm

```javascript
result = confirm(question);
```

- `question`: 질문 문자열
- 사용자가 확인 버튼을 누르면 `true`, 그 외의 경우는 `false` 반환

## 형 변환 (type conversion)

### 문자형으로 변환

`String(value)` 함수

### 숫자형으로 변환

```javascript
alert("6" / "2"); // 수학과 관련된 함수와 표현식에서 자동으로 발생
```

- `Number(value)` 함수
- 숫자 이외의 글자가 들어간 문자열을 숫자형으로 변환하면 `NaN`이 됨
- `undefined` -> `NaN`
- `null` -> `0`
- `true, false` -> `1`, `0`
- `string` -> 문자열의 처음과 끝 공백 제거됨. 제거 후 남아있는 문자열이 없다면 `0`, 그렇지 않다면 문자열에서 숫자를 읽음. 변환에 실패하면 `NaN`

### 불린형으로 변환

- `Boolean(value)` 함수
- `0`, `빈 문자열`, `null`, `undefined`, `NaN` -> `false`
- 이외의 값은 `true`로 변환됨

### 단항 연산자 +와 숫자형으로의 변환

피연산자가 숫자가 아닌 경우 숫자형으로 변환됨

```javascript
alert(+true); // 1
alert(+""); // 0
```

### 할당 연산자

```javascript
let a = 1;
let b = 2;
let c = 3 - (a = b + 1);
alert(a); // 3
alert(c); // 0
```

### 일치 연산자

- 동등 연산자(equality operator) `==`는 `0`과 `flase` 구별 불가
- 일치 연산자(strict equality operator) `===`는 형 변환 없이 값 비교

```javascript
alert(0 == false); // true
alert("" == false); // true
alert(0 === false); // false
```

### null이나 undefined와 비교하기

```javascript
alert(null === undefined); // false
alert(null == undefined); // true
```

- `<`, `>`, `<=`, `>=`를 사용하여 비교하면 `null`은 `0`, `undefined`는 `NaN`으로 변환됨

### null vs 0

```javascript
alert(undefined > 0); // false
alert(undefined == 0); // false
alert(undefined >= 0); // true
```

### 비교가 불가능한 undefined

```javascript
alert(undefined > 0); // false
alert(undefined < 0>); // false
alert(undefined == 0); // false
```

### 조건부 연산자 '?'

```javascript
let result = condition ? value1 : value2;
```

### 논리연산자

- `||`, `&&`, `!`
- `if`를 `||`나 `&&`로 대체하는 방식도 있음
- `!!`을 사용해 값을 불린형으로 변경 가능

```javascript
let x = 1;
x > 0 && alert("0보다 크다");
```

```javascript
alert(!!null); // false
alert(!!"non-empty string"); // true
```

### nullish 병합 연산자 '??'

- nullish 병합 연산자(nullish coalescing operator) `??`로 여러 피연산자 중 값이 확정되어 있는 변수를 찾을 수 있음
- `a ?? b`: a가 null도 아니고 undefined도 아니면 a. 그 외의 경우는 b

```javascript
x = a ?? b;
x = a !== null && a !== undefined ? a : b;
let firstName = null;
let lastName = null;
let nickname = "바이올렛";
alert(firstName ?? lastName ?? nickName ?? "익명의 사용자"); // 바이올렛
```

### '??'와 '||'의 차이

- `||`는 첫 번째 truthy 값 반환
- `??`는 첫 번째 정의된(defined) 값을 반환
- `null`, `undefined`, `0`을 구분해 다룰 때 중요

```javascript
height = height ?? 100; // height에 값이 정의되지 않은 경우, 100 할당
```

```javascript
let height = 0;
alert(height || 100); // 100
alert(height ?? 100); // 0
```

- 높이처럼 `0`이 할당될 수 있는 변수를 사용해 기능을 개발할 땐 `||`보다 `??`이 적합

## 반복문

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
    if (condition) break outer;
  }
}
```

- 레이블을 별도에 줄에 쓸 수도 있음

```javascript
outer:
for (condition) {
  ...
}
```

## switch문

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

- break문을 만날 때까지 실행하기 때문에 잘 써야 함

## 함수

함수는 프로그램을 구성하는 주요 구성 요소(building block)

### 함수 선언(function declaration)

```javascript
function name(parameter1, ...) {
  ...
}
```

- 함수 호출시 매개변수에 인수를 전달하지 않으면 그 값은 `undefined`가 됨
- 원하지 않는다면 기본값(default value)를 설정하면 됨

```javascript
function showMessage(text = "no text given") {}
```

- return문이 없거나 return 지시자만 있는 함수는 `undefined`를 반환

### 함수 표현식(function expression)

```javascript
let sayHi = function() {
  ...
};
```

### 콜백 함수

- 함수를 함수의 인수로 전달하고, 필요하다면 인수로 전달한 그 함수를 나중에 호출(called back)하는 것이 콜백 함수의 개념
- 이름 없이 선언한 함수는 익명 함수(anonymous function)라 부름

### 함수 표현식 vs 함수 선언문

- 함수 표현식은 실제 실행 흐름이 해당 함수에 도달했을 때 함수를 생성
- 함수 선언문은 함수 선언문이 정의되기 전에도 호출 가능
- 엄격 모드에서 함수 선언문이 코드 블록 내에 위치하면 해당 함수는 블록 내 어디서든 접근 가능. 블록 밖에서는 접근 불가

### 화살표 함수(arrow function)

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
