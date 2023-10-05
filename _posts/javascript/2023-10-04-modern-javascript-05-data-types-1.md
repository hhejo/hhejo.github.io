---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 1
date: 2023-10-04 20:23:45 +0900
last_modified_at: 2023-10-04 20:23:45 +0900
categories: [JavaScript]
tags: [javascript]
---

원시값의 메서드, 숫자형, 문자열

## 원시값의 메서드

- 자바스크립트는 원시값에도 메서드를 호출할 수 있어 객체처럼 다룰 수 있음
- 원시값: string, number, bigint, boolean, symbol, null, undefined
- 객체: 프로퍼티에 다양한 종류의 값 저장. 함수도 객체의 일종. 원시값보다 무겁고 추가 자원 사용

### 원시값을 객체처럼 사용하기

- 원시값은 원시값 그대로 남겨 단일 값 형태 유지
- 문자열, 숫자, 불린, 심볼의 메서드와 프로퍼티에 접근할 수 있도록 언어 차원에서 허용
- 이를 위해 원시값이 메서드나 프로퍼티에 접근하려 하면 추가 기능을 제공해주는 특수한 객체, 원시 래퍼 객체(object wrapper)를 생성하고 곧 삭제됨
- 각 래퍼 객체는 원시 타입에 따라 다양함. 원시 자료형의 이름을 차용해 `String`, `Number`, `Boolean`, `Symbol`이라 부름

```javascript
let str = "Hello";
alert(str.toUpperCase()); // HELLO
```

- 문자열 str은 원시값이므로 원시값의 프로퍼티(toUpperCase)에 접근하는 순간 특별한 객체 생성
- 이 객체는 문자열의 값을 알고 있고, toUpperCase()와 같은 유용한 메서드를 가짐
- 메서드가 실행되고 새로운 문자열 반환
- 특별한 객체는 파괴되고, 원시값 str만 남음
- 자바스크립트 엔진이 최적화를 도와줌

```javascript
let n = 1.23456;
alert(n.toFixed(2)); // 1.23
```

### String, Number, Boolean을 생성자로 쓰지 않기

- 하위 호환성을 위해 남겨둠
- 그러나 혼동이 발생할 수 있기 때문에 래퍼 객체를 만드는 것은 추천하지 않음

```javascript
alert(typeof 0); // number
alert(typeof new Number(0)); // object
```

```javascript
let zero = new Number(0);
if (zero) {
  alert("그런데 여러분은 zero가 참이라는 것에 동의하시나요!?!"); // 객체는 논리 평가 시에 참
}
```

- `new`를 붙이지 않고 `String`, `Number`, `Boolean`을 사용하는 것은 괜찮음
- `new` 없이 사용하면 형의 원시값으로 변경해줌

```javascript
let num = Number("123"); // 123
```

- `null`, `undefined`는 래퍼 객체도 없고 메서드도 없음

```javascript
alert(null.test); // error
```

## 숫자형

- 일반적인 숫자(2^53 ~ -2^53)는 배정밀도 부동소수점 숫자(double precision floating point number)로 알려진 64비트 형식의 IEEE-754에 저장
- 임의의 길이를 가진 정수는 BigInt 숫자로 나타냄

### 숫자를 입력하는 다양한 방법

```javascript
let billion = 1e9; // 10억. 1과 9개의 0
alert(7.3e9); // 73억
let ms = 1e-6; // 0.000001
alert(0xff); // 255
alert(0xff); // 255
let a = 0b11111111; // 255
let b = 0o377; // 255
alert(a == b); // true
```

### toString(base)

`num.toString(base)`

- `base` 진법으로 `num`을 표현하고 이를 문자형으로 변환하고 반환
- `base`는 `2` ~ `36`까지 가능하고 `10`이 기본값

```javascript
let num = 255;
alert(num.toString(16)); // ff
alert(num.toString(2)); // 11111111
```

- 숫자를 대상으로 메서드를 호출하고 싶다면 점 두 개 사용
- 한 개만 사용하면 소수부로 인식하기 때문
- 괄호로 둘러싸고 점 한 개만 사용할 수도 있음

```javascript
alert((123456).toString(36)); // 2n9c
```

```
alert(123456..toString(36)); // 2n9c
```

### 어림수 구하기

`Math.floor`

- 소수점 첫째 자리에서 내림(버림)

`Math.ceil`

- 소수점 첫째 자리에서 올림

`Math.round`

- 소수점 첫째 자리에서 반올림

`Math.trunc`

- 소수부 무시

|      | `Math.floor` | `Math.ceil` | `Math.round` | `Math.trunc` |
| :--: | :----------: | :---------: | :----------: | :----------: |
| 3.1  |      3       |      4      |      3       |      3       |
| 3.6  |      3       |      4      |      4       |      3       |
| -1.1 |      -2      |     -1      |      -1      |      -1      |
| -1.6 |      -2      |     -1      |      -2      |      -1      |

1.2345를 소수점 두 번째 자릿수까지만 남기기

1. 곱하기와 나누기: 10의 거듭제곱 수를 곱한 후 어림수 내장 함수를 호출하고 처음 곱한 수를 다시 나눔
2. `toFixed(n)`: 소수점 n번째 수까지의 어림수를 구한 후 문자열로 반환. n이 소수부의 길이보다 크면 끝에 0 추가됨

### 부정확한 계산

- 64비트 중 52비트는 숫자를 저장하는 데 사용되고, 11비트는 소수점 위치(정수는 0), 1비트는 부호를 저장하는 데 사용
- 숫자가 너무 커지면 `Infinity`로 처리

정밀도 손실(loss of precision)

```javascript
alert(0.1 + 0.2); // 0.30000000000000004
```

```javascript
alert(9999999999999999); // 10000000000000000
```

두 종류의 0: `0`, `-0`

### isNaN과 isFinite

`isNaN(value)`

- 인수를 숫자로 변환한 다음 `NaN`인지 테스트
- `NaN`은 자기 자신을 포함하여 그 어떤 값과도 같지 않기 때문에 `isNaN`이 필요

```javascript
alert(isNaN(NaN)); // true
alert(isNaN("str")); // true
alert(NaN === NaN); // false
```

`isFinite(value)`

- 인수를 숫자로 변환하고 변환한 숫자가 `NaN`, `Infinity`, `-Infinity`가 아닌 일반 숫자인 경우 `true` 반환
- 문자열이 일반 숫자인지 검증하는 데 사용
- 빈 문자열이나 공백만 있는 문자열은 `0`으로 취급

```javascript
alert(isFinite("15")); // true
alert(isFinite("str")); // false (NaN)
alert(isFinite(Infinity)); // false (Infinity)
```

`Object.is(a, b)`

- `NaN`을 대상으로 비교할 때 `Object.is(NaN, NaN) === true`
- `0`과 `-0`이 다르게 취급되어야 할 때 `Object.is(0, -0) === false`
- 두 가지를 제외하고 `Object.is(a, b)`와 `a === b`의 결과는 같음

### parseInt와 parseFloat

`+`, `Number()`

- 피연산자가 숫자가 아니면 형 변환 실패
- 문자열의 처음 또는 끝에 공백이 있어서 공백을 무시할 때가 아니면 엄격한 규칙 적용

`parseInt`, `parseFloat`

- 불가능할 때까지 문자열에서 숫자를 읽고, 읽는 도중 오류가 발생하면 이미 수집된 숫자를 반환
- 두 번째 인수로 원하는 진수를 지정할 수 있음. `parseInt(str, radix)`

```javascript
alert(+"100px"); // NaN
alert(parseInt("100px")); // 100
alert(parseFloat("12.5em")); //12.5
alert(parseInt("12.3")); // 12
alert(parseFloat("12.3.4")); // 12.3
alert(parseInt("a123")); // NaN
```

### 기타 수학 함수

`Math.random()`

- 0 이상 1 미만의 난수 반환

`Math.max(a, b, c, ...)`

- 인수 중 최댓값을 반환

`Math.min(a, b, c, ...)`

- 인수 중 최솟값을 반환

`Math.pow(n, power)`

- `n`을 `power`번 거듭제곱한 값을 반환

## 문자열

- 텍스트 형식의 데이터는 길이에 상관 없이 문자열 형태로 저장
- 페이지 인코딩 방식과 상관 없이 항상 `UTF-16` 형식을 따름

### 따옴표

- 작은따옴표, 큰따옴표, 백틱

템플릿 리터럴(template literal)

- 표현식을 `${...}`로 감싸고 이를 백틱으로 감싸는 방식
- 백틱을 사용하면 여러 줄 문자열 작성 가능

템플릿 함수(template function)

- 첫 번째 백틱 바로 앞에 함수 이름을 써주면 해당 함수는 백틱 안의 문자열 조각이나 표현식 평가 결과를 인수로 받아 자동으로 호출됨 (태그드 템플릿(tagged template))
- ```
  func`string`
  ```

### 특수 기호

`\`과 함께 사용

### 문자열의 길이

`length` 프로퍼티에 문자열의 길이가 저장됨

```javascript
alert("My\n".length); // 3
```

### 특정 글자에 접근하기

`[]`

- 접근하려는 위치에 글자가 없는 경우 `undefined` 반환

`str.charAt(pos)`

- 접근하려는 위치에 글자가 없는 경우 빈 문자열 반환

`for...of`

- 문자열을 구성하는 글자를 대상으로 반복 작업
- ```javascript
  for (let char of "Hello") {
    alert(char);
  }
  ```

### 문자열의 불변성

문자열은 수정할 수 없음

### 대·소문자 변경하기

`toLowerCase()`, `toUpperCase()`

- ```javascript
  alert("Interface".toUpperCase()); // INTERFACE
  alert("Interface".toLowerCase()); // interface
  alert("Interface"[0].toLowerCase()); // i
  ```

### 부분 문자열 찾기

`str.indexOf(substr, pos)`

- 문자열 `str`의 `pos`에서부터 시작해 부분 문자열 `substr`이 어디 있는지 찾음
- 찾으면 위치를 반환하고 그렇지 않으면 `-1` 반환

`str.lastIndexOf(substr, pos)`

- 문자열 끝에서부터 부분 문자열을 찾음
- 반환되는 부분 문자열 위치는 문자열 끝이 기준

`str.includes(substr, pos)`

- `str`에 `substr`이 있는지 없는지에 따라 `true`, `false` 반환

`str.startsWith(substr)`, `str.endsWith(substr)`

- 특정 문자열로 시작하는지, 특정 문자열로 끝나는지 여부 확인

### 부분 문자열 추출하기

`str.slice(start [, end])`

- `start`부터 `end` 전까지 반환
- `end`가 없다면 끝까지 반환
- 음수라면 문자열 끝에서부터 카운팅 시작
- 주로 많이 씀

`str.substring(start [, end])`

- `start`와 `end` 사이 반환
- `slice`와 유사하지만 `start`가 `end`보다 커도 괜찮음
- 음수는 0으로 취급

`str.substr(start [, length])`

- `start`에서부터 시작해 `length` 개의 글자 반환
- 끝 위치 대신 길이를 기준으로 문자열을 추출
- 음수 허용

### 문자열 비교하기

`str.codePointAt(pos)`

- `pos`에 위치한 글자의 코드 반환

`String.fromCodePoint(code)`

- 숫자 형식의 `code`에 대응하는 글자 생성

`str.localeCompare(str2)`

- `str`이 `str2`보다 작으면 음수 반환
- `str`이 `str2`보다 크면 양수 반환
- `str`이 `str2`와 같으면 0 반환

### 문자열 심화

- 서로게이트 쌍(surrogate pair)
- 발음 구별 기호
- 유니코드 정규화

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
