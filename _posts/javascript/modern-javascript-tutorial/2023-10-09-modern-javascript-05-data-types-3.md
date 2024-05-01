---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 3
date: 2023-10-09 08:23:28 +0900
last_modified_at: 2024-05-01 11:33:21 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, map, set, weakmap, weakset, object]
---

맵과 셋, 위크맵과 위크셋, Object.keys, values, entries

## 맵과 셋

객체

- 키가 있는 컬렉션을 저장

배열

- 순서가 있는 컬렉션을 저장

### 맵(Map)

- 키가 있는 데이터를 저장
- 키에 다양한 자료형을 허용(객체도 키로 허용)
  - 키를 문자형으로 자동 변환하는 객체와 달리 키의 타입을 변환하지 않고 그대로 유지
- `SameValueZero` 알고리즘을 사용해 값의 등가 여부를 확인
  - 일치 연산자 `===`와 유사하나 `NaN`과 `NaN`을 같다고 취급하는 것이 차이
  - `NaN`도 키로 사용 가능
- `new Map()`: 새로운 맵 생성
  - 인수로 각 요소가 `[키, 값]` 쌍인 배열 전달 가능
- `map.set(key, value)`: `key`를 이용해 `value`를 저장
  - 호출할 때마다 맵 자신을 반환하기 때문에 체이닝(chaining) 가능
- `map.get(key)`: `key`에 해당하는 값을 반환. 존재하지 않으면 `undefined` 반환
  - `map[key]`로 접근하면 `map`이 일반 객체로 취급되기 때문에 여러 제약 발생
- `map.has(key)`: `key`가 존재하면 `true`, 존재하지 않으면 `false` 반환
- `map.delete(key)`: `key`에 해당하는 값 삭제. 없어도 상관 없음
- `map.clear()`: 맵 안의 모든 요소 제거
- `map.size`: 요소의 개수 반환
- `map.toString`: 문자열 `"[object Map]"` 반환
- `map.valueOf`: `map`과 같음

```javascript
let map = new Map();
map.set("1", "str1").set(1, "num1").set(true, "bool1");
map.get(1); // num1
map.get("1"); // str1
alert(map.size); // 3

let john = { name: "John" };
let visitsCountMap = new Map();
let visitsCountObj = {};
visitsCountMap.set(john, 123);
visitsCountObj[john] = 123;
visitsCountMap.get(john); // 123
visitsCountObj["[object Object]"]; // 123

let map2 = new Map([
  ["1", "str1"],
  [1, "num1"],
  [true, "bool1"]
]); // 각 요소가 [key, value]인 배열 전달
alert(map2.get("1")); // str1

let map3 = new Map();
map3.set("1", "str1").set(1, "num1").set(true, "bool1"); // 체이닝

// map[key]로 사용하지 않기
map[key] = 2; // X
map.set(key, value); // O
map[key]; // X
map.get(key); // O

// map.toString(), map.valueOf()
map.toString(); // "[object Map]"
map.valueOf(); // map이 나옴
```

### 맵 요소에 반복 작업하기

- `map.keys()`: 각 요소의 키를 모은 이터러블 객체 반환
- `map.values()`: 각 요소의 값을 모은 이터러블 객체 반환
- `map.entries()`: 요소의 `[key, value]`를 한 쌍으로 하는 이터러블 객체 반환
  - `entries()` 없이 `map`만 써도 동일
- `map.forEach(fn)`: 배열과 유사한 forEach 메서드
- 객체는 프로퍼티 순서를 기억하지 못하지만 맵은 값이 삽입된 순서대로 순회 가능
- `Array.from`과 같이 사용해 이터러블 객체를 배열로 변환

```javascript
recipeMap.forEach((value, key, map) => {});
```

```javascript
let recipeMap = new Map([
  ["cucumber", 500],
  ["tomatoes", 350],
  ["onion", 50]
]);
for (let vegetable of recipeMap.keys()) alert(vegetable); // cucumber, tomatoes, onion
for (let amount of recipeMap.values()) alert(amount); // 500, 350, 50
// for (let entry of recipeMap.entries()) {}
for (let entry of recipeMap) alert(entry); // cucumber,500, tomatoes,350, onion,50

// Array.from()와 .keys(), .values(), .entries()
let map = new Map();
map.set("1", "hello").set("a", 4);
let arr = Array.from(map); // [["1", "hello"], ["a", 4]]
arr = Array.from(map.keys()); // ["1", "a"]
arr = Array.from(map.values()); // ["hello", 4]
```

### Object.entries: 객체를 맵으로 바꾸기

`Object.entries()`

- 객체의 키-값 쌍을 요소(`[key, value]`)로 갖는 배열 반환
  - 이터러블이 아닌 배열 반환
- 문자열도 인수로 전달 가능

```javascript
let obj = { name: "John", age: 30 };
Object.entries(obj); // [["name", "John"], ["age", 30]]
let map = new Map(Object.entries(obj)); // {'name' => 'John', 'age' => 30}
alert(map.get("name"));
Object.entries([1, 2, 3]); // [['0', 1], ['1', 2], ['2', 3]]
Object.entries("abc"); // [['0', 'a'], ['1', 'b'], ['2', 'c']]
```

### Object.fromEntries: 맵을 객체로 바꾸기

`Object.fromEntries()`

- 각 요소가 `[key, value]` 쌍인 배열을 객체로 변환
- 이터러블 객체도 인수로 받음

```javascript
let prices = Object.fromEntries([
  ["banana", 1],
  ["orange", 2],
  ["meat", 4]
]);
prices; // { banana: 1, orange: 2, meat: 4 }
alert(prices.orange); // 2

let map = new Map();
map.set("banana", 1).set("orange", 2).set("meat", 4);
let obj = Object.fromEntries(map); // Object.fromEntries(map.entries())
obj; // { banana: 1, orange: 2, meat: 4 }
alert(obj.orange); // 2
```

### 셋(Set)

- 중복을 허용하지 않는 값을 모은 컬렉션
- 키가 없는 값이 저장됨
- 값의 유일무이함을 확인하는 데 최적화
- `new Set(iterable)`: 셋 생성. 이터러블 객체를 전달받으면(주로 배열) 그 안의 값을 복사해 셋에 넣음
- `set.add(value)`: 값을 추가하고 셋 자신을 반환. 이미 있는 값은 적용되지 않음
- `set.delete(value)`: 호출 시점에 셋에 값이 있어서 제거에 성공하면 `true`, 아니면 `false` 반환
- `set.has(value)`: 셋에 값이 존재하면 `true`, 아니면 `false` 반환
- `set.clear()`: 셋을 비움
  `set.size`: 셋에 몇 개의 값이 있는지 셈

```javascript
let set = new Set();
let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };
set.add(john);
set.add(pete);
set.add(mary);
set.delete(john); // true
set.delete(john); // false
set.add(john).add(mary);
alert(set.size); // 3
for (let user of set) {
  alert(user.name); // John, Pete, Mary
}
```

### 셋의 값에 반복 작업하기

- `set.keys()`: 셋의 모든 값을 포함하는 이터러블 객체 반환
- `set.values()`: `set.keys()`와 동일한 작업
  - 맵과의 호환성을 위한 메서드
- `set.entries()`: 셋의 각 값을 이용해 만든 `[value, value]` 배열을 포함하는 이터러블 객체 반환
  - 맵과의 호환성을 위한 메서드
- `for..of`
- `set.forEach(fn)`
  - 맵과의 호환성을 위해 `valueAgain`이 있어 맵을 셋으로, 셋을 맵으로 교체하기 쉬워짐

```javascript
set.forEach((value, valueAgain, set) => {});
```

```javascript
let set = new Set(["oranges", "apples", "bananas"]);
for (let value of set) alert(value);
set.forEach((value, valueAgain, set) => {
  alert(value);
});
```

### 배열에서 중복 요소 제거하기

- Map이나 Set을 `Array.from`과 같이 쓰기

```javascript
let values = ["a", "b", "a", "b", "b", "b", "a", "a", "c"];
function unique(arr) {
  return Array.from(new Set(arr));
}
let newArr = unique(values); // ["a", "b", "c"]
```

## 위크맵과 위크셋

자바스크립트는 도달 가능한(추후 사용될 가능성이 있는) 값을 메모리에 유지

```javascript
let john = { name: "John" };
john = null; // 객체가 도달 가능하지 않게 되어 메모리에서 삭제
```

객체의 프로퍼티, 배열, 맵, 셋의 요소

- 자신이 속한 자료구조가 메모리에 남아있는 동안 대개 도달 가능한 값으로 취급되어 메모리에 유지

```javascript
let john = { name: "John" };
let array = [john];
john = null;
alert(JSON.stringify(array[0])); // {"name":"John"}

let john2 = { name: "John" };
let map = new Map();
map.set(john2, "...");
john2 = null;
for (let obj of map.keys()) alert(JSON.stringify(obj)); // {"name":"John"}
alert(map.size);
```

### 위크맵(WeakMap)

- 키로 쓰인 객체가 가비지 컬렉션의 대상이 됨
- 키는 반드시 객체. 원시값은 키로 사용 불가
- 위크맵의 키로 사용된 객체를 참조하는 것이 아무것도 없다면, 해당 객체는 메모리와 위크맵에서 자동으로 삭제됨
- `weakMap.get(key)`
- `weakMap.set(key, value)`
- `weakMap.delete(key)`
- `weakMap.has(key)`
- 위크맵은 맵과 다르게 반복 작업과 `keys()`, `values()`, `entries()` 메서드를 지원하지 않음
  - 가비지 컬렉션의 동작 방식 때문
  - 객체는 모든 참조를 잃게 되면 자동으로 가비지 컬렉션의 대상이 됨
  - 가비지 컬렉션이 일어나는 시점은 자바스크립트 엔진이 결정 (정확히 알 수 없음)
  - 객체가 모든 참조를 잃었을 때, 즉시 메모리에서 삭제될 수도 있고 다른 삭제 작업이 있을 때까지 대기하다가 함께 삭제될 수도 있음
    - 따라서 현재 위크맵에 요소가 몇 개 있는지 정확히 파악할 수 없음
  - 가비지 컬렉터가 한번에 메모리 청소를 할 수도 있고, 부분 부분 메모리 청소를 할 수도 있음
    - 따라서 위크맵의 요소(키/값) 전체를 대상으로 무언가를 하는 메서드는 동작 자체가 불가능

```javascript
let weakMap = new WeakMap();
let obj = {};
weakMap.set(obj, "ok");
weakMap.set("test", "whoops"); // TypeError: Invalid value used as weak map key

let john = { name: "John" };
weakMap.set(john, "...");
john = null; // john을 나타내는 객체는 이제 메모리에서 지워짐
```

### 유스 케이스: 추가 데이터

위크맵을 사용할 수 있는 경우

- 부차적인 데이터를 저장할 공간이 필요할 때 유용
- 서드파티 라이브러리와 같은 외부 코드에 속한 객체를 가지고 작업을 할 때,
- 객체에 추가할 데이터는 객체가 살아있는 동안에만 유효한 상황인 경우
- 위크맵에 원하는 데이터를 저장하면 객체(키)가 가비지 컬렉션의 대상이 될 때, 데이터도 함께 사라짐
- 애플리케이션 구조가 복잡할 때는 쓸모 없는 데이터를 수동으로 삭제하는 것이 까다롭기 때문에 위크맵은 유용

```javascript
// WeakMap 사용 전
// visitsCount.js
let visitsCountMap = new Map(); // 사용자 방문 횟수 저장
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1); // 해당 유저의 방문 횟수 증가
}
// main.js
let john = { name: "John" };
countUser(john);
john = null; // 해당 사용자가 사라져도 visitsCountMap의 키로 사용되고 있어 아직 메모리에 남아있음

// WeakMap 사용 후
// visitsCount.js
let visitsCountMap = new WeakMap(); // 위크맵으로 사용자 방문 횟수 저장
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}
let john = { name: "John" };
countUser(john);
john = null; // 사용자가 도달 가능하지 않으면 위크맵에서도 자동으로 삭제됨
```

### 유스 케이스: 캐싱

위크맵은 캐싱이 필요할 때 유용

- 캐싱(caching): 시간이 오래 걸리는 작업의 결과를 저장해 연산 시간과 비용을 절약해주는 기법
  - 동일한 함수를 여러 번 호출해야 할 때, 최초 호출 시 반환된 값을 어딘가에 저장해 놓음
  - 그 다음에는 함수를 호출하는 대신 저장된 값을 사용하는 것이 예

```javascript
// cache.js
let cache = new Map(); // cache를 WeakMap으로 교체하면 됨
function process(obj) {
  if (!cache.has(obj)) {
    let result = obj; // 연산 수행
    cache.set(obj, result);
  }
  return cache.get(obj);
}
// main.js
let obj = {...}
let result1 = process(obj);
let result2 = process(obj); // 연산을 수행할 필요 없이 맵에 저장된 결과를 가져옴
obj = null; // 객체가 쓸모 없어지면
alert(cache.size); // 1 (여전히 객체가 남아 있어 메모리 낭비)
```

### 위크셋(WeakSet)

- 셋과 유사하나 객체만 저장 가능. 원시값 저장 불가
- 해당 객체가 도달 가능할 때만 메모리에 유지됨
- 위크맵처럼 복잡한 데이터를 저장하지 않고 예, 아니오 같은 간단한 답변을 얻는 용도로 사용
  - `weakSet.add()`
  - `weakSet.delete()`
  - `weakSet.has()`
  - `size`, `keys()`나 반복 작업 관련 메서드 사용 불가

```javascript
let visitedSet = new WeakSet();
let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };
visitedSet.add(john);
visitedSet.add(pete);
visitedSet.add(john);
alert(visitedSet.has(john)); // true
alert(visitedSet.has(mary)); // false
john = null; // visitedSet에 john을 나타내는 객체가 자동으로 삭제됨
```

## Object.keys, values, entries

순회(iteration)

- 일반 객체에도 순회 관련 메서드가 있지만 `keys()`, `values()`, `entries()`와 문법의 차이가 있음
- Map, Set, Array: `keys()`, `values()`, `entries()`를 쓸 수 있는 자료구조

### Object.keys, values, entries

- `Object.keys(obj)`: 객체의 키만 담은 배열 반환
- `Object.values(obj)`: 객체의 값만 담은 배열 반환
- `Object.entries(obj)`: `[키, 값]` 쌍을 담은 배열 반환
- Map, Set, Array 전용 메서드와 일반 객체용 메서드 비교
  - `map.keys()`, `set.keys()`, `arr.keys()`: 이터러블 객체 반환
  - `Object.keys(obj)`(`obj.keys()` 아님): 진짜 배열 반환
- `Object.keys()`, `Object.values()`, `Object.entries()`는 `for..in` 반복문처럼 심볼형 프로퍼티 무시
  - 심볼형 키가 필요한 경우 `Object.getOwnPropertySymbols` 사용
  - 키 전체를 배열 형태로 반환하는 `Reflect.ownKeys(obj)`를 사용해도 됨

문법이 다른 이유

- 유연성 때문
- 자바스크립트에서는 복잡한 자료구조 전체가 객체에 기반
- 객체 `data`에 자체적으로 `data.values()`라는 메서드를 구현해 사용하는 경우
  - `Object.values(data)` 같은 형태로 메서드를 호출할 수 있으면
  - 커스텀 메서드, 내장 메서드 둘 다 사용할 수 있음
- 메서드 `Object.*`를 호출하면 이터러블 객체가 아닌 객체의 한 종류를 반환
  - 하위 호환성 때문

```javascript
let user = { name: "John", age: 30 };
Object.keys(user); // ["name", "age"]
Object.values(user); // ["John", 30]
Object.entries(user); // [["name", "John"], ["age", 30]]
```

### 객체 변환하기

객체에 배열 전용 메서드 사용하기

1. `Object.entries(obj)`로 객체의 키-값 쌍인 요소를 얻음
2. 해당 배열에 `map` 등의 배열 전용 메서드를 적용
3. 반환된 배열에 `Object.fromEntries`로 배열을 다시 객체로 변환

```javascript
let prices = { banana: 1, orange: 2, meat: 4 };
let doublePrices = Object.fromEntries(
  Object.entries(prices).map(([key, value]) => [key, value * 2])
);
doublePrices; // { banana: 2, orange: 4, meat: 8 }
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
