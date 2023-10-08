---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 3
date: 2023-10-06 08:23:28 +0900
last_modified_at: 2023-10-06 08:23:28 +0900
categories: [JavaScript]
tags: [javascript]
---

맵, 셋, aaaaaaaaa

## 맵과 셋

- 객체: 키가 있는 컬렉션을 저장
- 배열: 순서가 있는 컬렉션을 저장

### 맵(Map)

- 키가 있는 데이터를 저장(객체와 유사)
- 키에 다양한 자료형을 허용(객체와 차이)
- 객체도 키로 혀용
- 객체는 키를 문자형으로 자동 변환하지만 맵은 키의 타입을 변환하지 않고 그대로 유지
- `SameValueZero`라 불리는 알고리즘을 사용해 값의 등가 여부를 확인
  - 이 알고리즘은 일치 연산자 `===`와 거의 유사하지만 `NaN`과 `NaN`을 같다고 취급하기 때문에 일치 연산자와 차이가 있음
  - 따라서 `NaN`도 키로 쓸 수 있음

`new Map()`

- 새로운 맵 생성

`map.set(key, value)`

- `key`를 이용해 `value`를 저장
- 호출할 때마다 맵 자신을 반환하기 때문에 체이닝(chaining)할 수 있음
- ```javascript
  map.set("1", "str1").set(1, "num1").set(true, "bool1");
  ```

`map.get(key)`

- `key`에 해당하는 값을 반환
- `key`가 존재하지 않으면 `undefined`를 반환
- `[]`(`map[key]`)를 사용해 접근할 키의 값에 접근하게 되면 `map`이 일반 객체로 취급되기 때문에 좋지 않음

`map.has(key)`

- `key`가 존재하면 `true`를 반환
- `key`가 존재하지 않으면 `false`를 반환

`map.delete(key)`

- `key`에 해당하는 값을 삭제

`map.clear()`

- 맵 안 의 모든 요소를 제거

`map.size`

- 요소의 개수를 반환

```javascript
let map = new Map();
map.set("1", "str1");
map.set(1, "num1");
map.set(true, "bool1");
alert(map.get(1)); // num1
alert(map.get("1")); // str1
alert(map.size); // 3
```

```javascript
let john = { name: "John" };
let visitsCountMap = new Map();
let visitsCountObj = {};
visitsCountMap.set(john, 123);
visitsCountObj[john] = 123;
alert(visitsCountMap.get(john)); // 123
alert(visitsCountObj["[object Object]"]); // 123
```

### 맵 요소에 반복 작업하기

`map.keys()`

- 각 요소의 키를 모은 반복 가능한(iterable) 객체를 반환

`map.values()`

- 각 요소의 값을 모은 이터러블 객체를 반환

`map.entries()`

- 요소의 `[key, value]`를 한 쌍으로 하는 이터러블 객체를 반환
- 해당 객체는 `for..of` 반복문에 사용
- `entries()` 없이 `map`만 써도 동일

```javascript
let recipeMap = new Map([
  ["cucumber", 500],
  ["tomatoes", 350],
  ["onion", 50]
]);
for (let vegetable of recipeMap.keys()) {
  alert(vegetable); // cucumber, tomatoes, onion
}
for (let amount of recipeMap.values()) {
  alert(amount); // 500, 350, 50
}
// for (let entry of recipeMap.entries()) {}
for (let entry of recipeMap) {
  alert(entry); // cucumber,500, tomatoes,350, onion,50
}
```

- 객체는 프로퍼티 순서를 기억하지 못하지만 맵은 값이 삽입된 순서대로 순회

`map.forEach(fn)`

- 배열과 유사하게 해당 메서드 지원

```javascript
recipeMap.forEach((value, key, map) => {
  alert(`${key}: ${value}`);
});
```

### Object.entries: 객체를 맵으로 바꾸기

- 각 요소가 키-값 쌍인 배열이나 이터러블 객체를 초기화 용도로 맵에 전달해 새로운 맵 생성 가능

```javascript
let map = new Map([
  ["1", "str1"],
  [1, "num1"],
  [true, "bool1"]
]);
alert(map.get("1")); // str1
```

`Object.entries(obj)`

- 객체를 맵으로 변경
- 객체의 키-값 쌍을 요소(`[key, value]`)로 갖는 배열 반환
- 반환 값을 `Object.fromEntries()`의 인수로 사용하면 좋음

```javascript
let obj = { name: "John", age: 30 };
let map = new Map(Object.entries(obj)); // [["name", "John"], ["age", 30]]
alert(map.get("name"));
```

`Object.fromEntries()`

- 맵을 객체로 변경
- 각 요소가 `[key, value]` 쌍인 배열을 객체로 변환
- 자료가 맵에 저장되어 있는데 서드파티 코드에서 자료를 객체 형태로 넘겨받길 원할 때 사용
- 인수로 `Object.entries()`의 반환 값을 사용하면 좋음
- 이터러블 객체를 인수로 받기 때문에 꼭 배열을 인수로 전달할 필요 없음

```javascript
let prices = Object.fromEntries([
  ["banana", 1],
  ["orange", 2],
  ["meat", 4]
]);
// prices = { banana: 1, orange: 2, meat: 4 }
alert(prices.orange); // 2
```

```javascript
let map = new Map();
map.set("banana", 1);
map.set("orange", 2);
map.set("meat", 4);
let obj = Object.fromEntries(map.entries());
// obj = { banana: 1, orange: 2, meat: 4 }
alert(obj.orange); // 2
```

```javascript
let obj = Object.fromEntries(map); // .entries() 생략해도 동일
```

### 셋(Set)

- 중복을 허용하지 않는 값을 모아놓은 특별한 컬렉션
- 키가 없는 값이 저장됨
- 값의 유일무이함을 확인하는 데 최적화되어 있음

`new Set(iterable)`

- 셋 생성
- 이터러블 객체를 전달받으면(대개 배열을 전달받음) 그 안의 값을 복사해 셋에 넣음

`set.add(value)`

- 값을 추가하고 셋 자신을 반환
- 동일한 값이 있다면 중복은 적용되지 않음

`set.delete(value)`

- 값을 제거
- 호출 시점에 셋 내에 값이 있어서 제거에 성공하면 `true`, 아니면 `false` 반환

`set.has(value)`

- 셋 내에 값이 존재하면 `true`, 아니면 `false` 반환

`set.clear()`

- 셋을 비움

`set.size`

- 셋에 몇 개의 값이 있는지 셈

```javascript
let set = new Set();
let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };
set.add(john);
set.add(pete);
set.add(mary);
set.add(john);
set.add(mary);
alert(set.size); // 3
for (let user of set) {
  alert(user.name); // John, Pete, Mary
}
```

### 셋의 값에 반복 작업하기

`for..of`

`set.forEach(fn)`

```javascript
set.forEach((value, valueAgain, set) => {});
```

- 맵과의 호환성을 위해 `valueAgain`이 있음
- 맵을 셋으로, 셋을 맵으로 교체하기 쉬워짐

```javascript
let set = new Set(["oranges", "apples", "bananas"]);
for (let value of set) alert(value);
set.forEach((value, valueAgain, set) => {
  alert(value);
});
```

`set.keys()`

- 셋 내의 모든 값을 포함하는 이터러블 객체를 반환

`set.values()`

- `set.keys()`와 동일한 작업을 함
- 맵과의 호환성을 위한 메서드

`set.entries()`

- 셋 내의 각 값을 이용해 만든 `[value, value]` 배열을 포함하는 이터러블 객체를 반환
- 맵과의 호환성을 위한 메서드

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
