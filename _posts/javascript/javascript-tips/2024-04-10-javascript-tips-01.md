---
title: JavaScript 팁 모음 ! (01)
date: 2024-04-10 10:15:11 +0900
last_modified_at: 2024-04-10 10:15:11 +0900
categories: [JavaScript, Javascript-Tips]
tags: [javascript]
---

JavaScript 팁을 모아봤습니다.

## JavaScript 팁 모음 ! (01)

DOM을 효율적으로 조작

- DOM에 한 번 접근하고, 나중에 사용할 수 있도록 변수에 캐시
- `$`를 사용해 DOM 참조 나타내기

```javascript
let $todos = document.querySelector(".todos");
```

전역 변수 사용을 피하기

- 지역 변수는 함수가 완료된 후 삭제되나, 전역 변수는 창이 닫힐 때까지 삭제되지 않음
- 과도한 전역 변수는 프로그램 속도를 저하시킬 수 있음

엄격 모드

- 기존에는 조용히 무시되던 에러들을 던짐
- JavaScript 엔진의 최적화 작업을 어렵게 만드는 실수들을 바로잡음
- 가끔씩 엄격 모드의 코드는 비 엄격 모드의 동일한 코드보다 더 빨리 작동

기본값 설정

- 특정 속성에 대한 기본값을 설정하지 않음으로써 객체가 정상적으로 작동하는 데 이러한 값이 필요하지 않다는 것을 알림

## 참고

> [15 Essential JavaScript Tips You Shouldn’t Overlook](https://medium.com/@lxs7332/15-essential-javascript-tips-you-shouldnt-overlook-54aa0e6fe997)
