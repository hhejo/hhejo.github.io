---
title: JavaScript로 2차원 배열 생성하기
date: 2024-04-07 11:13:42 +0900
last_modified_at: 2024-04-08 09:41:57 +0900
categories: [Algorithm]
tags: [algorithm, javascript, array]
---

JavaScript는 2차원 배열을 어떻게 생성할 수 있을까요?

## JavaScript로 2차원 배열 생성하기

for문을 이용해 생성할 수도 있지만, 더욱 간단한 방법이 있습니다.

### Array.from으로 2차원 배열 생성하기

```javascript
const [r, c] = [5, 5]; // 행, 열
let arr;
// Array
arr = Array.from(Array(r), () => Array(c).fill(0)); // Array()
arr = Array.from({ length: r }, () => Array(c).fill(0)); // 유사 배열 객체
```

### Array로 2차원 배열 생성하기

```javascript
const [r, c] = [5, 5];
let arr;
arr = Array(r)
  .fill(0)
  .map(() => Array(c).fill(0));
```
