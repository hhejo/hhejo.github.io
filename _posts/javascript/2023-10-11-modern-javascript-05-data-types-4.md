---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 4
date: 2023-10-11 15:17:49 +0900
last_modified_at: 2023-10-15 08:12:46 +0900
categories: [JavaScript]
tags: [javascript]
---

구조 분해 할당, Date, JSON

## 구조 분해 할당

함수에 객체나 배열에 저장된 데이터 전체가 아닌 일부만 필요한 경우

함수의 매개변수가 많거나 매개변수 기본값이 필요한 경우

구조 분해 할당(destructuring assignment)

- 객체나 배열을 변수로 분해할 수 있게 해주는 특별한 문법
- 어떤 것을 복사한 이후에 변수로 분해(destructurize)
- 분해 대상이 수정, 파괴되는 것이 아님

### 배열 분해하기

배열을 변수로 분해하기

```javascript
let arr = ["Bora", "Lee"];
// let firstName = arr[0];
// let surname = arr[1];
let [firstName, surname] = arr; // 코드 양 감소
alert(firstName); // Bora
alert(surname); // Lee
```

반환 값이 배열인 메서드와 사용하기

```javascript
let [firstName, surname] = "Bora Lee".split(" ");
```

쉼표를 사용하여 필요하지 않은 배열 요소 무시하기

```javascript
let [a, , c] = ["apple", "banana", "coconut", "date"]; // banana, date 무시
alert(c); // coconut
```

할당 연산자 우측엔 모든 이터러블 가능

```javascript
let [a, b, c] = "abc";
let [one, two, three] = new Set([1, 2, 3]);
```

할당 연산자 좌측엔 할당할 수 있는(assignables) 것이라면 어떤 것이든 가능

```javascript
let user = {};
[user.name, user.surname] = "Bora Lee".split(" ");
user; // { name: "Bora", surname: "Lee" }
```

`.entries()`로 반복하기

```javascript
let user = { name: "John", age: 30 };
for (let [key, value] of Object.entries(user)) alert(`${key}:${value}`);
```

```javascript
let user = new Map();
user.set("name", "John").set("age", 30);
for (let [key, value] of user) alert(`${key}:${value}`);
```

두 변수에 저장된 값 교환하기

```javascript
let guest = "Jane";
let admin = "Pete";
[guest, admin] = [admin, guest];
let [a, b, c] = [1, 2, 3];
[a, b, c] = [b, c, a];
```

### '...'로 나머지 요소 가져오기

배열 앞쪽에 위치한 값 몇개만 필요하고 그 이후 이어지는 나머지 값은 한데 모아 저장하는 경우

```javascript
let [fruit1, fruit2, ...rest] = ["apple", "banana", "coconut", "date"];
fruit1, fruit2, rest; // "apple", "banana", ["coconut", "date"]
```

### 기본값

할당하고자 하는 변수의 개수가 분해하고자 하는 배열의 길이보다 크면 에러가 발생하지 않고 `undefined` 할당

```javascript
let [firstName, surname] = [];
firstName, surname; // undefined, undefined
```

`=`을 이용해 할당할 값이 없을 때 기본으로 할당할 값인 기본값(default value) 설정

```javascript
let [name = "Guest", surname = "Anonymous"] = ["Julius"];
name, surname; // "Julius", "Anonymous"
```

복잡한 표현식이나 함수 호출도 기본값 가능

```javascript
let [surname = prompt("성 입력"), name = prompt("이름 입력")] = ["김"];
surname, name; // "김"(배열에서 받아온 값), prompt에서 받아온 값
```

### 객체 분해하기

할당 연산자 우측에 분해하고자 하는 객체를, 좌측에 상응하는 객체의 프로퍼티 작성

```javascript
let {var1, var2} = {var1:..., var2:...}
```

```javascript
let options = { title: "Menu", width: 100, height: 200 };
let { title, width, height } = options;
// let { width, height, title } = options; 순서 상관 없음
```

객체 프로퍼티를 프로퍼티 키와 다른 이름을 가진 변수에 저장 가능

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
