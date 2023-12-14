---
title: JavaScript에서 연속된 값의 배열 얻기
date: 2023-12-13 13:27:18 +0900
last_modified_at: 2023-12-14 09:17:29 +0900
categories: [JavaScript]
tags: [javascript, range]
---

Python에 range가 있다면 JavaScript에는?

## JavaScript의 Python range?

`[1, 2, 3, 4, 5, ...]`처럼 일정한 크기로 연속되는 숫자의 배열을 만들고 싶을 때가 있습니다.

하지만 JavaScript에는 Python의 `range` 함수 같은 함수가 없습니다.

### for 반복문 사용하기

가장 쉬운 방법은 `for` 반복문을 사용하는 것입니다.

```javascript
let arr = [];
for (let i = 1; i <= n; i++) arr.push(i);
```

이것을 Python과 비슷한 `range` 함수로 직접 만들 수 있습니다.

```javascript
function range(start, end, step = 1) {
  if (!end) [start, end] = [0, start];
  let arr = [];
  for (let i = start; i < end; i++) {
    if (!(i % step)) arr.push(i);
  }
  return arr;
}
console.log(range(1, 11)); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log(range(5)); // [0, 1, 2, 3, 4]
console.log(range(1, 11, 2)); // [2, 4, 6, 8, 10]
```

### Array 메서드 사용하기

`Array.keys`

```javascript
let arr = [...Array(5).keys()];
console.log(arr); // [0, 1, 2, 3, 4]
let arr2 = [...Array(5).keys()].map((x) => x + 1);
console.log(arr2); // [1, 2, 3, 4, 5]
```

`Array.from`

```javascript
let arr = Array.from({ length: 5 }, (_, idx) => idx);
console.log(arr); // [0, 1, 2, 3, 4]
let arr2 = Array.from({ length: 5 }, (_, idx) => idx + 1);
console.log(arr2); // [1, 2, 3, 4, 5]
```

`Array.fill`

```javascript
let arr = Array(5)
  .fill(0)
  .map((_, idx) => idx);
console.log(arr); // [0, 1, 2, 3, 4]
let arr2 = Array(5)
  .fill(0)
  .map((_, idx) => idx + 1);
console.log(arr2); // [1, 2, 3, 4, 5]
```

### String과 연속된 문자 만들기

`String.fromCharCode`와 `String.charCodeAt`을 사용합니다.

```javascript
let string = String.fromCharCode(
  ...[...Array("Z".charCodeAt() - "A".charCodeAt() + 1).keys()].map(
    (idx) => idx + "A".charCodeAt()
  )
);
console.log(string); // ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

## 참고

> [Does JavaScript have a method like "range()" to generate a range within the supplied bounds?](https://stackoverflow.com/questions/3895478/does-javascript-have-a-method-like-range-to-generate-a-range-within-the-supp)

> [연속된 값을 갖는 배열을 얻는 방법](https://velog.io/@jinyongp/%EC%97%B0%EC%86%8D%EB%90%9C-%EB%B0%B0%EC%97%B4%EC%9D%84-%EC%96%BB%EB%8A%94-%EB%B0%A9%EB%B2%95)
