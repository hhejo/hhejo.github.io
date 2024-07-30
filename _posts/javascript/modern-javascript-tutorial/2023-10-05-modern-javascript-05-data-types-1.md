---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 1
date: 2023-10-05 20:23:45 +0900
last_modified_at: 2024-07-30 16:15:17 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, primitive, number, string]
---

원시값의 메서드, 숫자형, 문자열

## 원시값의 메서드

자바스크립트는 원시값에도 메서드 호출 가능

- 원시값은 객체가 아니지만 객체처럼 다룰 수 있음
- 원시값(7개): number, string, bigint, boolean, symbol, null, undefined
- 객체: 프로퍼티에 다양한 종류의 값 저장
  - 원시값보다 무겁고 내부 구조 유지를 위해 추가 자원 사용
  - 함수도 객체의 일종이며 함수를 프로퍼티로 저장 가능
  - 자바스크립트는 날짜, 오류, HTML 요소(HTML element) 등을 다룰 수 있게 해주는 다양한 내장 객체 제공

### 원시값을 객체처럼 사용하기

원시값에 메서드를 사용하고 싶음

- 원시값은 원시값 그대로 남겨 단일 값 형태 유지
- string, number, boolean, symbol의 메서드와 프로퍼티에 접근할 수 있도록 언어 차원에서 허용
- 원시값이 메서드나 프로퍼티에 접근하려 하면 원시 래퍼 객체가 생성되고 곧 삭제됨
  - 원시 래퍼 객체(object wrapper): 추가 기능을 제공해주는 특수한 객체
  - 각 래퍼 객체는 원시 자료형의 이름을 차용해 `String`, `Number`, `Boolean`, `Symbol`
- 자바스크립트 엔진이 최적화를 도움
- 원시 래퍼 객체를 만들지 않고도 마치 원시 래퍼 객체를 생성한 것처럼 동작하게끔 해줌

```javascript
let str = "Hello";
alert(str.toUpperCase()); // HELLO
let n = 1.23456;
alert(n.toFixed(2)); // 1.23
```

1. 문자열 str의 원시값의 프로퍼티(toUpperCase)에 접근하는 순간 특별한 객체 생성됨
   - 이 객체는 문자열의 값을 알고 있고, 메서드를 가짐
2. 메서드가 실행되고 새로운 문자열 반환
3. 특별한 객체는 파괴되고, 원시값 str만 남음

`String`, `Number`, `Boolean`을 생성자로 쓰지 않기

- 혼동이 발생할 수 있기 때문에 래퍼 객체를 만드는 것은 추천하지 않음
- 하위 호환성을 위해 남겨짐
- `new` 없이 사용하면 형의 원시값으로 변경해줌
  - `String(number)`, `Number(string)`

```javascript
alert(typeof 0); // number
alert(typeof new Number(0)); // object
let zero = new Number(0);
if (zero) alert("zero는 참"); // 객체는 논리 평가 시 항상 참
let num = Number("123"); // 123. 올바른 사용법
```

`null`, `undefined`

- 래퍼 객체도 없고 메서드도 없음

```javascript
alert(null.test); // error
```

### 예제

문자열에 프로퍼티를 추가할 수 있을까요?

```javascript
let str = "Hello";
str.test = 5;
alert(str.test);
```

- `str`의 프로퍼티에 접근하면 래퍼 객체 생성
- 엄격 모드: `TypeError`
  - 래퍼 객체를 수정하려 할 때 에러 발생
- 비 엄격 모드: `undefined`
  - 에러가 발생하지 않고 래퍼 객체에 프로퍼티 `test` 추가
  - 그런데 래퍼 객체는 바로 삭제되기 때문에 마지막 줄이 실행될 때는 프로퍼티 `test`를 찾을 수 없음
- 따라서 원시값은 추가 데이터를 저장할 수 없음

## 숫자형

모던 자바스크립트는 숫자를 나타내는 두 가지 자료형을 지원

1. 일반적인 숫자(`2^53` 미만, `-2^53` 초과)
   - 배정밀도 부동소수점 숫자(double precision floating point number)로 알려진 64비트 형식의 IEEE-754에 저장
2. 임의의 길이를 가진 정수
   - BigInt로 나타냄

### 숫자를 입력하는 다양한 방법

자바스크립트에서 지원하는 진법은 3가지

- 2, 8, 16 진수
- 이외의 진법을 사용하려면 `parseInt` 사용

```javascript
let billion = 1e9; // 10억 (1과 9개의 0)
alert(7.3e9); // 73억
let ms = 1e-6; // 0.000001
alert(0xff); // 255 (16진수)
// alert(0xFF); // 255 (대소문자 상관 없음)
let a = 0b11111111; // 255 (2진수)
let b = 0o377; // 255 (8진수)
alert(a == b); // true
```

### toString(base)

`num.toString(base)`

- `base`진법으로 `num`을 표현하고 문자형으로 변환해 반환
- `base`: `2~36` (기본값 `10`)
- `base`가 `36`이면 `0..9`, `A..Z`를 이용해 숫자를 표현하므로 숫자로 된 긴 식별자를 짧게 줄여 유용

```javascript
let num = 255;
alert(num.toString(16)); // ff
alert(num.toString(2)); // 11111111
alert((123456).toString(36)); // 2n9c
```

숫자를 대상으로 메서드 호출하기

- `..`: 점 한 개만 사용하면 소수부로 인식함
- `().`

```javascript
// alert(123456..toString(36));
alert((123456).toString(36)); // 2n9c
```

### 어림수 구하기(rounding)

|      | `Math.floor` | `Math.ceil` | `Math.round` | `Math.trunc` |
| :--: | :----------: | :---------: | :----------: | :----------: |
| 3.1  |      3       |      4      |      3       |      3       |
| 3.6  |      3       |      4      |      4       |      3       |
| -1.1 |      -2      |     -1      |      -1      |      -1      |
| -1.6 |      -2      |     -1      |      -2      |      -1      |

- `Math.floor`: 소수점 첫째 자리에서 내림(버림)
- `Math.ceil`: 소수점 첫째 자리에서 올림
- `Math.round`: 소수점 첫째 자리에서 반올림
- `Math.trunc`: 소수부 무시

`toFixed(n)`

- 소수점 n번째 수까지의 어림수를 구하고 문자열로 반환
- `n`이 소수부의 길이보다 크면 끝에 `0` 추가

```javascript
let num = 12.34;
alert(num.toFixed(1)); // 12.3
alert(num.toFixed(5)); // 12.34000
num = 12.36;
alert(num.toFixed(1)); // 12.4 (가장 가까운 값으로 올림 혹은 버림. Math.round와 유사)

// 다시 숫자형으로 바꾸기
alert(+num.toFixed(1)); // +
alert(Number(num.toFixed(1))); // Number()

// 그냥 계산으로 구하기
let num2 = 1.23456;
alert(Math.floor(num2 * 100) / 100); // 1.23456 -> 123.456 -> 123 -> 1.23
```

### 부정확한 계산

숫자는 내부적으로 64비트 형식 IEEE-754으로 표현

- 숫자를 저장하려면 정확히 64비트 필요
- 52비트: 숫자 저장
- 11비트: 소수점 위치(정수는 0)
- 1비트: 부호 저장
- 숫자가 너무 커지면 64비트 공간이 넘쳐서 `Infinity`로 처리됨

정밀도 손실(loss of precision)

- `0.1 + 0.2`의 경우,
- 10진법으로는 쉽게 표현되는 `0.1`, `0.2` 같은 분수는 이진법으로 표현하면 무한 소수가 됨
- `1/3`은 무한 소수 `0.33333(3)`이 됨
- 2진법 체계에서 `2`의 거듭제곱으로 나눈 값은 잘 동작하지만 `1/10` 같이 `2`의 거듭제곱이 아닌 값으로 나누면 무한 소수가 됨
- 10진법에서 `1/3`을 정확히 나타낼 수 없듯이, 2진법을 사용해 `0.1` 또는 `0.2`를 정확하게 저장하는 방법은 없음
- IEEE-754에서는 가능한 가장 가까운 숫자로 반올림하는 방법을 사용해 이런 문제를 해결
- 두 숫자 `0.1`, `0.2`를 합하면 정밀도 손실이 더해짐
- `(0.1).toFixed(20)`의 경우,
- 숫자를 저장할 때는 64비트가 사용되는데, 이 중 실제 숫자를 저장하는 데 사용되는 52비트에 위 숫자를 저장하기엔 공간이 부족
- 최소 유효 숫자(the least significant digit)가 손실됨
- 자바스크립트는 숫자 손실이 일어나도 오류를 발생시키지 않고 적절한 포맷으로 숫자를 맞춤

```javascript
alert(1e500); // Infinity
alert(0.1 + 0.2 == 0.3); // false
alert(0.1 + 0.2); // 0.30000000000000004
alert((0.1).toFixed(20)); // 0.10000000000000000555
alert(9999999999999999); // 10000000000000000 (수가 증가함)
```

`0`, `-0` 두 개의 0

- 자바스크립트 내부에서 숫자를 표현하는 방식 때문에 발생하는 현상
- 자바스크립트에서는 숫자의 부호가 단일 비트에 저장됨
- 0을 포함한 모든 숫자에 부호를 설정할 수도, 설정하지 않을 수도 있음
- 대부분의 연산은 `0`과 `-0`을 동일하게 취급

### isNaN과 isFinite

`isNaN(value)`

- 인수를 숫자로 변환한 결과가 `NaN`이면 `true`를 반환하고, 아니면 `false` 반환
- `NaN`은 자기 자신을 포함하여 그 어떤 값과도 같지 않음

```javascript
alert(isNaN(NaN)); // true
alert(isNaN("str")); // true
alert(NaN == NaN); // false
alert(isNaN(undefined)); // true
alert(isNaN(null)); // false
```

`isFinite(value)`

- 인수를 숫자로 변환한 결과가 일반 숫자인 경우 `true`를 반환하고, `NaN`, `Infinity`, `-Infinity`이면 `false` 반환
- 문자열이 일반 숫자인지 검증하는 데 사용
- 빈 문자열 `""`이나 공백만 있는 문자열 `"  "`은 `isFinite`를 포함한 모든 숫자 관련 내장 함수에서 `0`으로 취급

```javascript
alert(isFinite("15")); // true
alert(isFinite("str")); // false (NaN)
alert(isFinite(Infinity)); // false (Infinity)
alert(isFinite(NaN)); // false (NaN)
alert(isFinite("")); // true
alert(isFinite(" ")); // true
alert(isFinite(" 1 ")); // true
alert(isFinite("+1")); // true
alert(isFinite("1px")); // false
let num = +prompt("숫자를 입력하세요.", "");
alert(isFinite(num)); // 숫자가 아닌 값, Infinity, -Infinity를 입력하면 false
```

`Object.is(a, b)`

- `NaN`을 대상으로 비교할 때: `Object.is(NaN, NaN) === true`
- `0`과 `-0`이 다르게 취급되어야 할 때: `Object.is(0, -0) === false`
- 두 가지를 제외하고 `Object.is(a, b)`와 `a === b`의 결과는 같음
- 비교 결과가 정확해야 하는 경우 사용
- `Object.is`에서 사용되는 비교 방식은 명세서에서 SameValue라고 불림

```javascript
alert(Object.is(NaN, NaN)); // true
alert(NaN === NaN); // false
alert(Object.is(-0, +0)); // false
alert(-0 === +0); // true
```

### parseInt와 parseFloat

`+`, `Number()`

- 피연산자가 숫자가 아니면 형 변환 실패
- 문자열의 처음 또는 끝에 공백이 있어서 공백을 무시할 때가 아니면 엄격한 규칙 적용
- 공백 문자열 `' '` -> `0`

`parseInt()`, `parseFloat()`

- 불가능할 때까지 문자열에서 숫자를 읽고, 읽는 도중 오류가 발생하면 이미 수집된 숫자를 반환
- `parseInt(str, radix)`: 원하는 진수를 `radix`에 지정
- 공백 문자열 `' '` -> `NaN`

```javascript
alert(+"100px"); // NaN
alert(+" 1 "); // 1
alert(parseInt("100px")); // 100
alert(parseFloat("12.5em")); //12.5
alert(parseInt("12.3")); // 12
alert(parseFloat("12.3.4")); // 12.3
alert(parseInt("a123")); // NaN

+" "; // 0
Number(" "); // 0
parseInt(" "); // NaN
+" 1 "; // 1
Number(" 1 "); // 1
parseInt(" 1 "); // 1
+" 10px"; // NaN
Number(" 10px"); // NaN
parseInt(" 10px"); // 10
+" 3 4 "; // NaN
Number(" 3 4 "); // NaN
parseInt(" 3 4 "); // 3
```

### 기타 수학 함수

- `Math.random()`: 0 이상 1 미만의 난수 반환
- `Math.max(a, b, c, ...)`: 인수 중 최댓값 반환
- `Math.min(a, b, c, ...)`: 인수 중 최솟값 반환
- `Math.pow(n, power)`: `n`을 `power`번 거듭제곱한 값 반환

## 문자열

텍스트 형식의 데이터는 길이에 상관 없이 문자열 형태로 저장

- 자바스크립트에서 문자열은 페이지 인코딩 방식과 상관 없이 항상 `UTF-16` 형식을 따름

### 따옴표

작은따옴표, 큰따옴표, 백틱

템플릿 리터럴(template literal)

- 표현식을 `${...}`로 감싸고 이를 백틱으로 감싸는 방식
- 백틱으로 여러 줄 문자열 작성 가능

템플릿 함수(template function)

- 첫 번째 백틱 바로 앞에 함수 이름을 써줌
- 해당 함수는 백틱 안의 문자열 조각이나 표현식 평가 결과를 인수로 받아 자동으로 호출됨
- 태그드 템플릿(tagged template)

```
function func(x) {
  alert(x);
}
func`string` // string
```

### 특수 기호

`\`과 함께 사용

### 문자열의 길이

`length` 프로퍼티

- 문자열 길이 저장됨

```javascript
alert("My\n".length); // 3
```

### 특정 글자에 접근하기

- `str[]`: 접근하려는 위치에 글자가 없으면 `undefined` 반환
- `str.at(index)`: 유효하지 않은 인덱스이면 `undefined` 반환
  - `-1`로 맨 뒤 글자 가져올 수 있어 유용
- `str.charAt(pos)`: 접근하려는 위치에 글자가 없으면 빈 문자열 `""` 반환
  - 하위 호환성을 위해 남은 메서드

```javascript
"123"[3]; // undefined
"123".charAt(3); // ""
"123".at(-1); // 3
```

### 문자열의 불변성

- 문자열은 수정 불가

```javascript
let str = "Hi";
srt[0] = "h"; // TypeError: Cannot assign to read only property '0' of string 'Hi'
```

### 대·소문자 변경하기

- `toLowerCase()`, `toUpperCase()`

```javascript
alert("Interface".toUpperCase()); // INTERFACE
alert("Interface".toLowerCase()); // interface
alert("Interface"[0].toLowerCase()); // i
```

### 부분 문자열 찾기

- `str.includes(substr[, pos])`
  - `str`에 `substr`이 있으면 `true`, 없으면 `false` 반환
- `str.startsWith(substr)`
  - `str`이 `substr`로 시작하면 `true` 반환
- `str.endsWith(substr)`
  - `str`이 `substr`로 끝나면 `true` 반환
- `str.indexOf(substr[, pos])`
  - 문자열 `str`의 `pos`에서부터 시작해 부분 문자열 `substr`이 어디 있는지 찾음
  - 찾으면 위치를 반환하고 그렇지 않으면 `-1` 반환
- `str.lastIndexOf(substr, pos)`
  - 문자열 끝에서부터 `substr` 찾음
  - 반환되는 부분 문자열 위치는 문자열 끝이 기준
- `str.search(substr)`
  - 시작 위치 지정 불가
  - 정규식 사용 가능
- 비트 NOT 연산자를 사용한 기법
  - `n`이 32비트 정수일 때, `~n`은 `-(n + 1)`
  - 부호가 있는 32비트 정수 `n` 중, `~n`을 `0`으로 만드는 경우는 `n == -1`일 때가 유일

```javascript
~2; // -3, -(2 + 1)
~1; // -2, -(1 + 1)
~0; // -1, -(0 + 1)
~-1; // 0, -(-1 + 1)
```

```javascript
let str = "Widget";
if (~str.indexOf("Widget")) alert("찾았다");
```

### 부분 문자열 추출하기

- `str.slice(start [, end])`
  - `start`부터 `end`가 있으면 `end` 전까지, 없으면 끝까지 반환
  - 음수인 경우 문자열 끝에서부터 카운팅 시작
  - 인수가 아무것도 없다면 현재와 똑같은 문자열 반환
  - 주로 많이 사용됨
- `str.substring(start [, end])`
  - `start`와 `end` 사이 반환
  - `slice`와 유사하지만 `start`가 `end`보다 커도 괜찮음
  - 음수는 0으로 취급
- `str.substr(start [, length])`
  - `start`에서부터 시작해 `length` 개의 글자 반환
  - 끝 위치 대신 길이를 기준으로 문자열을 추출
  - 음수 허용

```javascript
"123".slice(); // "123"
"123".slice(0); // "123"
"123".slice(0, "123".length); // "123"
"123".slice("123".length); // ""
"123".slice(-"123".length); // "123"
```

### 문자열 비교하기

- `str.codePointAt(pos)`
  - `pos`에 위치한 글자의 코드 반환
- `String.fromCodePoint(code)`
  - 숫자 형식의 `code`에 대응하는 글자 생성
- `str.localeCompare(str2)`
  - `str`이 `str2`보다 작으면 음수 반환
  - `str`이 `str2`보다 크면 양수 반환
  - `str`이 `str2`와 같으면 0 반환

### 문자열 심화

- 서로게이트 쌍(surrogate pair)
- 발음 구별 기호
- 유니코드 정규화

### 기타 유용한 문자열 메서드

- `str.trim()`
  - 문자열 앞과 끝의 공백 문자를 제거
- `str.repeat(n)`
  - 문자열을 `n`번 반복
- `str.replace(searchValue, replaceValue)`
  - `searchValue`: 문자열 또는 정규표현식
  - `replaceValue`: 문자열 또는 콜백 함수
- `str.replaceAll()`
  - 모두 대체
- `str.padStart(targetLength [, padString])`
  - `targetLength`: 전체 문자열 길이 지정
  - `padString`: 현재 문자열의 길이가 `targetLength`보다 짧다면 이것으로 채움
- `str.padEnd()`
  - 우측부터 적용
- `str.concat(str1, ..., strN)`
  - 문자열을 합쳐 반환

유용한 방법

- `for...of`
  - 문자열을 구성하는 글자를 대상으로 반복 작업
- `...`
  - 문자열 분해하기
- `str.split("")`
  - 인수로 빈 문자열 `""` 넣으면 한 글자씩 쪼갬

```javascript
for (let char of "Hello") alert(char); // H,e,l,l,o

[..."abc"]; // ["a", "b", "c"]

"abc".split(""); // ["a", "b", "c"]
"abc".split(); // ["abc"]
"abc".split(" "); // ["abc"]
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
