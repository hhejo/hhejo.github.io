---
title: JavaScript에서 tilde 연산자 사용하기
date: 2024-04-27 15:01:44 +0900
last_modified_at: 2024-04-27 15:01:44 +0900
categories: [JavaScript]
tags: [javascript, tilde]
---

tilde 연산자와 double tilde 연산자로 간단하게 소수점 버리고 0으로 바꾸기

## tilde 연산자

tilde 연산자 `~`

- NOT의 비트 연산자

`~N`은 `-(N + 1)`이 됨

```javascript
let a = 5; // 00000000000000000000000000000101
console.log(~a); // -6
console.log((~a).toString(2)); // '11111111111111111111111111111010'
```

## double tilde

### 소수점 버리기

`Math.floor()` 대신 사용할 수 있고 더 빠름

- Chrome에서는 `Math.floor`와 비슷한 속도
- `parseInt`보다는 확실히 빠름

```javascript
let [a, b] = [10, 3];
console.log(a / b); // 3.3333333333333335
let c = ~~(a / b);
console.log(c); // 3
```

### undefined, null을 0으로 바꾸기

```javascript
console.log(~~undefined); // 0
console.log(~~null); // 0
```

예: `undefined + 3 -> NaN`

- `undefined`, `null`이 나올 수 있는 상황을 `~~`로 해결

```javascript
const arr = [1, 1, 1, 2, 2, 3, 3, 3, 3];

const obj1 = {};
arr.forEach((v) => {
  if (obj1[v]) obj1[v] += 1; // obj1에 키가 있는지 확인하는 작업
  else obj1[v] = 1;
});
//obj1 { 1: 3, 2: 2, 3: 4 }

const obj2 = {};
arr.forEach((v) => (obj2[v] = ~~obj2[v] + 1)); // 키가 있는지 확인하지 않고 바로 작업
//obj2 { 1: 3, 2: 2, 3: 4 }
```

## 참고

> [JS tilde(~)과 double tilde(~~)연산자](https://velog.io/@proshy/JS-tilde과-double-tilde연산자)

> [The Mysterious Double Tilde (~~) Operation](https://dev.to/asadm/the-mysterious-double-tilde-operation-mih)
