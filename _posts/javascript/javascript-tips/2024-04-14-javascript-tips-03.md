---
title: JavaScript 팁 모음 ! (03)
date: 2024-04-14 17:05:23 +0900
last_modified_at: 2024-04-14 17:05:23 +0900
categories: [JavaScript, Javascript-Tips]
tags: [javascript]
---

JavaScript 팁을 모아봤습니다.

## `array.length`를 루프 내에 캐싱하기

크기가 큰 배열을 처리할 때는 `array.length`를 변수에 할당해 매 루프 진입 시점마다 `array.length`를 계산하는 것을 피할 수 있음

```javascript
const length = array.length;
for (let i = 0; i < length; i++) {
  console.log(array[i]);
}
```

## NodeList를 배열로 전환하기

NodeList는 배열 메서드를 사용할 수 없음

NodeList를 배열로 변환해야 가능

```javascript
let elements = document.querySelector("p"); // NodeList
let arrayElements = [].slice.call(elements); // 배열로 변환됨
arrayElements = Array.from(elements); // 또 다른 방법
```

## JSON으로 깊은 복사하기

`JSON.stringify()`를 사용해 객체를 JSON으로 변환하고, `JSON.parse()`를 사용해 다시 파싱

큰 객체에서는 성능 문제가 발생할 수 있으므로 간단한 객체에 대해서만 수행 권장

```javascript
const person = {
  name: "Dom",
  age: 28,
  skills: ["Programming", "Cooking", "Tennis"]
};
const copied = JSON.parse(JSON.stringify(person)); // 깊은 복사
console.log(person.skills === copied.skills); // false
```

## 향상된 배열 검색

`find()` 메서드를 사용하면 테스트 함수를 전달할 수 있음

배열 내에서 테스트 함수를 통과하는 첫 번째 요소가 반환됨

```javascript
const occupations = ["Lawyer", "Doctor", "Programmer", "Chef", "Store Manager"];
const result = occupations.find((o) => o.startsWith("C"));
console.log(result); // Chef
```

## 자체 호출 함수 (Self-Invoking Functions)

자체 호출 함수는 스스로 실행되는 함수

복잡한 로직이 필요한 변수나 상수를 할당

```javascript
const someComplexValue = (() => {
  const [a, b] = [10, 20];
  if (a > b) return a * b;
  return b / a;
})();
```

## 참고

> [JavaScript - 번역 11가지 극도로 유용한 JavaScript 팁](https://chaewonkong.github.io/posts/11-useful-js-tips.html)

> [7 Must Know JavaScript Tips & Tricks 🎈]()
