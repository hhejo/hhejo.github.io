---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 1
date: 2023-10-17 14:30:02 +0900
last_modified_at: 2024-05-02 15:30:41 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, recursion, linked-list, rest-parameters, spread-syntax]
---

재귀와 스택, 나머지 매개변수와 스프레드 문법

## 재귀와 스택

재귀(recursion)

- 함수가 자기 자신을 호출하는 경우
- 큰 목표 작업 하나를 동일하면서 간단한 작업 여러 개로 나눌 수 있을 때 유용한 프로그래밍 패턴
- 목표 작업을 간단한 동작 하나와 목표 작업을 변형한 작업으로 단순화 시킬 수 있을 때 사용
- 특정 자료구조를 다뤄야 할 때 사용

### 두 가지 사고방식

```javascript
pow(x, n); // x를 n제곱하는 함수
```

1 반복적인 사고: `for` 루프

```javascript
function pow(x, n) {
  let result = 1;
  for (let i = 0; i < n; i++) result *= x;
  return result;
}
```

2 재귀적인 사고: 작업을 단순화하고 자기 자신을 호출

```javascript
function pow(x, n) {
  if (n == 1) return x;
  return x * pow(x, n - 1);
}
```

```
pow(2, 4) = 2 * pow(2, 3)
pow(2, 3) = 2 * pow(2, 2)
pow(2, 2) = 2 * pow(2, 1)
pow(2, 1) = 1
```

- `n == 1`: 명확한 결괏값을 즉시 도출
  - 재귀의 베이스(base) (`pow(x, 1)`은 `x`)
- `n != 1`: 재귀 단계
  - `n`이 `1`이 될 때까지 이어짐(자기 자신을 계속 호출)

재귀 깊이(recursion depth)

- 가장 처음 하는 호출을 포함한 중첩 호출의 최대 개수
  - `pow(x, n)`의 재귀 깊이는 `n`
- 자바스크립트 엔진은 최대 재귀 깊이를 제한
  - 10,000개는 허용, 그 이상은 엔진에 따라 상이
- 제한을 완화하기 위해 엔진 내부에서 자동으로 tail calls optimization 수행(주로 간단한 경우에)

### 실행 컨텍스트와 스택

실행 컨텍스트(execution context)

- 함수 실행에 대한 세부 정보를 담고 있는 내부 데이터 구조
  - 제어 흐름의 현재 위치, 변수의 현재 값, `this`의 값(여기서는 다루지 않음) 등
- 함수 호출 1회당 정확히 하나의 실행 컨텍스트 생성
- 메모리를 차지하기 때문에 반복문을 사용하면 대개 함수 호출의 비용(메모리 사용)이 절약됨
- 실행 컨텍스트 스택(execution context stack): 중지된 함수와 연관된 실행 컨텍스트를 저장하는 특별한 자료구조

함수 내부의 중첩 호출 시

- 현재 함수의 실행 일시 중지
- 중지된 함수와 연관된 실행 컨텍스트는 실행 컨텍스트 스택 최상단에 기록
- 중첩 호출(서브 호출)을 위한 새로운 컨텍스트 생성
- 중첩 호출 실행하고 완료 후 종료
- 실행 컨텍스트 스택에서 일시 중단한 함수의 실행 컨텍스트 꺼냄(pop)
- 중단한 함수의 실행 재개

재귀

- 재귀 깊이: 스택에 들어가는 실행 컨텍스트의 수의 최댓값과 동일
- 재귀의 장점
  - 코드가 짧아짐, 코드 이해도 상승, 유지보수 향상
- 재귀 코드를 반복문 코드로 재작성해도 큰 개선이 없는 경우
  - 조건에 따라 함수가 다른 재귀 서브 호출을 하고 그 결과를 합칠 때
  - 분기문이 복잡하게 얽혀있을 때

### 재귀적 순회

재귀는 재귀적 순회(recursive traversal)를 구현할 때 유용

재귀를 이용해 모든 임직원의 급여를 더한 값 구하기

1. 임직원 배열을 가진 단순한 부서
   - 반복문으로 급여 합계 구함
2. N개의 하위 부서가 있는 객체
   - 각 하위 부서에 속한 임직원의 급여 합계를 얻기 위해 N번의 재귀 호출
   - 최종적으로 모든 하위 부서 임직원의 급여를 더함

```javascript
let company = {
  sales: [
    { name: "John", salary: 1000 },
    { name: "Alice", salary: 1600 }
  ],
  development: {
    sites: [
      { name: "Peter", salary: 2000 },
      { name: "Alex", salary: 1800 }
    ],
    internals: [{ name: "Jack", salary: 1300 }]
  }
};

function sumSalaries(department) {
  if (Array.isArray(department))
    return department.reduce((prev, current) => prev + current.salary, 0);
  let sum = 0;
  for (let subdep of Object.values(department)) sum += sumSalaries(subdep);
  return sum;
}
alert(sumSalaries(company)); // 7700
```

### 재귀적 구조

- 자기 자신의 일부를 복제하는 형태
- HTML 태그
  - 일반 텍스트
  - HTML-주석
  - 이외의 HTML 태그
    - 일반 텍스트
    - HTML-주석
    - 이외의 HTML 태그 ...

### 연결 리스트

배열

- 요소 삽입, 삭제에 많은 비용 필요

연결 리스트

- 빠르게 삽입, 삭제 가능

```javascript
let list = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: {
        value: 4,
        next: null
      }
    }
  }
};
```

```javascript
let list = { value: 1 };
list.next = { value: 2 };
list.next.next = { value: 3 };
list.next.next.next = { value: 4 };
list.next.next.next.next = null;
```

```
     value         value         value         value
list [ 1 ] -next-> [ 2 ] -next-> [ 3 ] -next-> [ 4 ] -> null
```

나누기

```javascript
let secondList = list.next.next;
list.next.next = null;
```

```
     value         value
      list [ 1 ] -next-> [ 2 ] -next-> null
secondList [ 3 ] -next-> [ 4 ] -next-> null
```

합치기

```javascript
list.next.next = secondList;
```

맨 앞에 새로운 값 추가

```javascript
let list = { value: 1 };
list.next = { value: 2 };
list.next.next = { value: 3 };
list.next.next.next = { value: 4 };
list = { value: "new item", next: list }; // list에 새로운 value 추가
```

```
     value                  value         value         value         value
list [ "new item" ] -next-> [ 1 ] -next-> [ 2 ] -next-> [ 3 ] -next-> [ 4 ] -next-> null
```

중간 요소 제거

```javascript
list.next = list.next.next;
```

```
     value                  value         value         value
list [ "new item" ] -next-> [ 2 ] -next-> [ 3 ] -next-> [ 4 ] -next-> null

value
[ 1 ] -next-> [ 2 ] ...
```

## 나머지 매개 변수와 스프레드 문법

상당수의 자바스크립트 내장 함수는 인수의 개수 제약이 없음

1. 임의의 수의 인수를 받을 수 있음
2. 함수의 매개변수로 배열을 받을 수 있음

### 나머지 매개 변수 '...'

함수에 넘겨주는 인수의 개수에는 제약 없음

- 함수의 정의 방법과 상관 없이 받는 인수의 개수보다 더 넘겨줘도 에러가 발생하지 않음

```javascript
function sum(a, b) {
  return a + b;
}
alert(sum(1, 2, 3, 4, 5)); // 3
```

나머지 매개 변수 `...`

- 여분의 매개변수를 담을 배열 이름 앞에 붙임
- 앞부분의 매개변수는 변수로 받고 남은 매개변수는 배열로 모을 수 있음
  - 나머지 매개변수는 마지막에 위치

```javascript
function sumAll(...args) {
  let sum = 0;
  for (let arg of args) sum += arg;
  return sum;
}
alert(sumAll(1)); // 1
alert(sumAll(1, 2)); // 3
alert(sumAll(1, 2, 3, 4)); // 10

function showFood(a, b, ...args) {
  alert(`${a} ${b}`); // Apple Banana
  alert(titles[0]); // Potato
  alert(titles[1]); // Pizza
  alert(titles.length); // 2
}
showFood("Apple", "Banana", "Potato", "Pizza");
```

### arguments 객체

`arguments` 객체

- 유사 배열 객체, 이터러블 객체
- 배열 메서드 사용 불가 (`map` 등)
- 함수의 인수 전체를 갖고 있어 인덱스로 각 인수에 접근
- 나머지 매개변수 문법 등장 전, 함수의 인수 전체를 얻어내는 유일한 방법
- 화살표 함수는 `arguments` 객체를 지원하지 않음
  - 화살표 함수에서 `arguments`에 접근하면 외부의 일반 함수의 `arguments` 가져옴
  - 자체 `this`도 갖지 않음

```javascript
function showFood() {
  alert(arguments.length);
  alert(arguments[0]);
  alert(arguments[1]);
  for (let arg of arguments) alert(arg);
}
showFood("Apple", "Banana"); // 2, Apple, Banana, Apple, Banana
showFood("Potato"); // 1, Potato, undefined, Potato

function f() {
  let showArg = () => alert(arguments[0]); // 외부의 일반 함수 f의 arguments
  showArg();
}
f(1); // 1
```

### 스프레드 문법 '...'

스프레드 문법 `...`

- 나머지 매개변수와 비슷해 보이나 반대 역할
- 스프레드 문법은 `for..of`와 같은 방식으로 내부에서 이터레이터를 사용해 요소 수집
- 이터러블 객체 여러 개 전달 가능
- 평범한 값과 혼합 사용 가능
- 배열 합칠 때도 활용
- 문자열에 `for..of`를 사용하면 문자열을 구성하는 문자 반환
- `Array.from`은 이터러블 객체인 문자열을 배열로 바꿔주기 때문에 동일한 작업 가능
- `Array.from(obj)`, `Array.from(iter)`: 유사 배열 객체, 이터러블 객체 둘 다에 사용 가능
  - 무언가를 배열로 바꿀 때 보편적으로 사용
- `[...iter]`: 이터러블 객체에만 사용. 유사 배열 객체는 불가

```javascript
let arr = [3, 5, 1];
alert(Math.max(arr)); // NaN
alert(Math.max(...arr)); // 5

let arr1 = [1, -2, 3, 4];
let arr2 = [8, 3, -8, 1];
alert(Math.max(...arr1, ...arr2)); // 8

let arr3 = [1, -2, 3, 4];
let arr4 = [8, 3, -8, 1];
alert(Math.max(1, ...arr3, 2, ...arr4, 25)); // 25

let arr5 = [3, 5, 1];
let arr6 = [8, 9, 15];
let merged = [0, ...arr5, 2, ...arr6];
alert(merged); // 0,3,5,1,2,8,9,15 (0, arr, 2, arr2 순서)

let str = "Hello";
alert([...str]); // H,e,l,l,o
alert(Array.from(str)); // H,e,l,l,o
```

### 배열과 객체의 복사본 만들기

`Object.assign` 말고도 스프레드 문법 `...`으로 배열과 객체 복사 가능

```javascript
let arr = [1, 2, 3];
let arrCopy = [...arr]; // 배열을 펼쳐 각 요소를 분리한 후, 매개변수 목록으로 만들고 새로운 배열에 할당
alert(JSON.stringify(arr) === JSON.stringify(arrCopy)); // true
alert(arr === arrCopy); // false (참조가 다름)
arr.push(4);
alert(arr); // 1, 2, 3, 4
alert(arrCopy); // 1, 2, 3

let obj = { a: 1, b: 2, c: 3 };
let objCopy = { ...obj }; // 객체를 펼쳐 각 요소를 분리한 후, 매개변수 목록으로 만들고 새로운 객체에 할당
alert(JSON.stringify(obj) === JSON.stringify(objCopy)); // true
alert(obj === objCopy); // false (참조가 다름)
obj.d = 4;
alert(JSON.stringify(obj)); // {"a":1,"b":2,"c":3,"d":4}
alert(JSON.stringify(objCopy)); // {"a":1,"b":2,"c":3}

let objCopy = { ...obj }; // Object.assign({}, obj);
let arrCopy = [...arr]; // Object.assign([], arr);

const object = Object.assign({}, ["a", "b"]); // { "0": "a", "1": "b" }
```

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
