---
title: 프로그래머스 Level 0 with JavaScript (1)
date: 2023-10-16 22:41:39 +0900
last_modified_at: 2023-10-16 22:41:39 +0900
categories: [JavaScript, Programmers]
tags: [javascript]
---

문자열 반복해 붙이기, 대소문자 구분하기, 문자열 하나씩 출력하기, 정수를 붙이기

## 문자열 반복해 붙이기

```javascript
str.repeat(count);
```

문자열을 `count`번만큼 반복해 붙인 새로운 문자열을 반환

- `count`: 양의 정수(0 포함)

## 대소문자 구분하기

대소문자 구분 메서드가 따로 없음

아래와 같이 응용하거나, 아스키코드 값을 이용하거나, 정규 표현식을 이용

```javascript
for (let char in str) {
  if (char === char.toUpperCase()) {
    console.log(`${char}: 대문자`);
  } else {
    console.log(`${char}: 소문자`);
  }
}
```

## 문자열 하나씩 출력하기

`...`를 사용하면 문자열을 쪼갤 수 있음

```javascript
str = "example";
[...str].forEach((char) => console.log(char));
```

## 정수를 붙이기

```javascript
parseInt(12 + "" + 34); // 1234
```
