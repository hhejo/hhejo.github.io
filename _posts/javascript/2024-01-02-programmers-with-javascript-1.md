---
title: 프로그래머스 with JavaScript (1)
date: 2024-01-02 07:25:44 +0900
last_modified_at: 2024-01-02 07:25:44 +0900
categories: [JavaScript, Programmers]
tags: [javascript, algorithm]
---

배열에서 특정 원소의 개수

## 배열에서 특정 원소의 개수 구하기

### 1. for

```javascript
let arr = [1, 2, 1, 2, 3, 3, 4, 5, 5, 5, 7];
let cnt = 0;
for (let el of arr) {
  if (el === 4) cnt += 1;
}
```

### 2. filter

```javascript
let arr = [1, 2, 1, 2, 3, 3, 4, 5, 5, 5, 7];
let cnt = arr.filter((el) => el === 4).length;
```

### 3. reduce

```javascript
let arr = [1, 2, 1, 2, 3, 3, 4, 5, 5, 5, 7];
let cnt = arr.reduce((cnt, el) => cnt + (el === 4), 0);
```
