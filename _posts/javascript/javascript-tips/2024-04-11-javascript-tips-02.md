---
title: JavaScript 팁 모음 ! (02)
date: 2024-04-11 16:27:00 +0900
last_modified_at: 2024-04-11 16:27:00 +0900
categories: [JavaScript, Javascript-Tips]
tags: [javascript]
---

JavaScript 팁을 모아봤습니다.

## JavaScript 팁 모음 ! (02)

조건부로 속성 추가하기

```javascript
const isValid = false;
const age = 18;
const person = {
  id: "ab32",
  name: "Krina",
  ...(isValid && { isActive: true }),
  ...((age >= 18 || isValid) && { cart: 0 })
};
console.log(person); // { id: "ab32", name: "Krina", cart: 0 }
```

동적 키를 사용해 객체 분해

```javascript
const productData = { id: "23", name: "Laptop" };
const extractKey = "name";
const { [extractKey]: data } = productData;
console.log(data); // Laptop
```

배열에서 falsy 값 제거

- `Boolean` 사용하기

```javascript
const fruitList = ["apple", null, "mango", undefined, ""];
const filterFruitList = fruitList.filter(Boolean);
console.log(filterFruitList); // [ "apple", "mango" ]
const isAnyFruit = fruitList.some(Boolean);
console.log(isAnyFruit); // true
```

배열에서 중복 값 제거

- `Set` 사용하기

```javascript
const fruitList = ["apple", "mango", "apple", "grapes"];
const uniqFruitList = [...new Set(fruitList)];
console.log(uniqFruitList); // [ "apple", "mango", "grapes" ]
```

숫자를 문자열로 쉽게 변환

- 문자열은 앞에 `+`를 붙여주기
- 숫자에 빈 문자열을 더해 문자열로 만들기

```javascript
const personId = 324743;
console.log(personId + "", typeof (personId + "")); // personId string
```

## 참고

> [11 Useful Modern JavaScript Tips](https://medium.com/dhiwise/11-useful-modern-javascript-tips-9736962ed2cd)
