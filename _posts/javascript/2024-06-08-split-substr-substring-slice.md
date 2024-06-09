---
title: JavaScript 문자열 자르기 메서드
date: 2024-06-08 09:03:16 +0900
last_modified_at: 2024-06-09 11:45:04 +0900
categories: [JavaScript]
tags: [javascript]
---

split, substr, substring, slice

## split

`seperator`로 문자열 분리해 배열로 리턴

```javascript
str.split(seperator[, limit])
```

- `seperator`: 구분자
- `limit`: 분리할 문자열의 개수
  - `0`을 전달하면 빈 배열 리턴

## substr

특정 인덱스에서 원하는 길이만큼 잘라서 문자열로 리턴

```javascript
str.substr(start[, length])
```

- `start`: 시작 인덱스
  - 음수이면 뒤부터 셈
- `length`: 길이만큼 자름

## substring

시작 인덱스에서 끝 인덱스 전까지 문자열 잘라서 리턴

```javascript
str.substring(start, end);
```

- `start`: 시작 인덱스
- `end`: 끝 인덱스
- 음수를 전달하면 빈 문자열 리턴

## slice

`substring`과 비슷하나 인자로 음수 전달 가능

```javascript
str.slice([begin[, end]])
```

- `begin`: 시작 인덱스
- `end`: 시작 인덱스

## 참고

[Javascript - 문자열 자르기(split, substr, substring, slice)](https://umanking.github.io/2022/07/17/javascript-string-substring/)
