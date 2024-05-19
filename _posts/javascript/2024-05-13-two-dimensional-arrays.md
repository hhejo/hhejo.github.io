---
title: 자바스크립트로 1차원, 2차원 배열 생성하기
date: 2024-05-13 19:13:08 +0900
last_modified_at: 2024-05-18 10:10:44 +0900
categories: [JavaScript]
tags: [javascript, array]
---

1차원과 2차원 배열을 생성하는 다양한 방법들을 정리합니다

## 1차원 배열

### 1. 배열 리터럴

```javascript
let arr = [1, 2, 3]; // [1, 2, 3]
```

### 2. 배열 생성자 Array, of, fill

```javascript
let arr = new Array(1, 2, 3); // [1, 2, 3]
let arr2 = new Array(3); // [empty x 3]
let arr3 = Array.of(3); // [3]
let arr4 = new Array(3).fill(0); // [0, 0, 0]
```

- 배열 생성자를 사용할 때 매개변수가 1개이면 생성할 배열의 크기를 의미
- `Array.of()` 메서드로 단일 요소 배열 생성 가능
- `Array.fill()` 메서드로 비어있는 요소를 채울 수 있음

### 3. from

```javascript
let arr = Array.from({ length: 3 }, () => 0); // [0, 0, 0];
```

- `Array.from()` 메서드에 배열이나 이터러블 객체를 삽입
- map 함수를 두 번째 인수로 넣어 배열 요소 초기화

### 4. ..., map

```javascript
let arr = [...Array(3)].map(() => 0); // [0, 0, 0]
```

## 2차원 배열

### 1. 중첩 루프

```javascript
function create2DArray(m, n) {
  let arr = new Array(m);
  for (let i = 0; i < m; i++) {
    arr[i] = new Array(n);
    for (let j = 0; j < n; j++) {
      arr[i][j] = 0;
    }
  }
  return arr;
}
```

### 2. Array.from

```javascript
function create2DArray(m, n) {
  return Array.from({ length: m }, () => Array.from({ length: n }, () => 0));
}
```

### 3. Array.fill, map

```javascript
function create2DArray(m, n) {
  return Array(m)
    .fill()
    .map(() => Array(n).fill(0));
}
```

### 4. ..., map

```javascript
function create2DArray(m, n) {
  return [...Array(m)].map(() => Array(n).fill(0));
}
```

## 참고

[Mastering JavaScript: Multiple Ways to Generate a Two-Dimensional Array](https://dev.to/yanhaijing/mastering-javascript-multiple-ways-to-generate-a-two-dimensional-array-cpg)
