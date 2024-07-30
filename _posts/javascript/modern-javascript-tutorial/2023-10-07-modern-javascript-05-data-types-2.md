---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 2
date: 2023-10-07 08:19:33 +0900
last_modified_at: 2024-07-30 16:17:02 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, array, iterable, array-like]
---

배열, 배열과 메서드, iterable 객체

## 배열

배열

- 순서가 있는 컬렉션을 저장할 때 사용

객체

- 순서를 고려하지 않고 만들어진 자료구조
- 새로운 프로퍼티를 기존 프로퍼티 사이에 끼워넣기 불가능

### 배열 선언

- 배열의 요소의 자료형에는 제약이 없고 함수도 가능
- `array[index]`: 배열 내 특정 요소를 얻거나 수정하고 새로운 요소 추가도 가능
- `length` 프로퍼티: 배열의 요소 개수 반환
- trailing 쉼표 `,`: 배열의 마지막 요소는 객체와 마찬가지로 쉼표로 끝날 수 있음
  - 모든 줄의 생김새가 유사해지기 때문에 요소를 넣거나 빼기 쉬워짐

```javascript
let arr = new Array();
let arr = [];
```

```javascript
let fruits = ["사과", "오렌지", "자두"];
fruits[2] = "배"; // 수정
fruits[3] = "레몬"; // 추가
alert(fruits.length); // 4
let arr = [
  "사과",
  { name: "John" },
  true,
  function () {
    alert("Hello");
  }
];
alert(arr[1].name); // John
arr[3](); // Hello
```

### pop·push와 shift·unshift

- `push`: 배열 맨 끝에 요소 추가
  - 여러 개 한번에 추가 가능
  - `arr.push(...)`는 `arr.[arr.length] = ...`와 같은 효과
  - 배열의 새로운 길이 반환
- `pop`: 배열 맨 끝 요소 추출
- `shift`: 배열 제일 앞 요소를 꺼내 제거한 후 남은 요소들을 앞으로 밀음
- `unshift`: 배열 제일 앞에 요소 추가
  - 여러 개 한번에 추가 가능

### 배열의 내부 동작 원리

- 배열은 특별한 종류의 객체
  - 키가 숫자이고, 본질은 객체
- 배열은 객체이므로 원하는 프로퍼티를 객체처럼 추가할 수 있음
- 그러면 자바스크립트 엔진이 배열을 일반 객체처럼 다루게 됨
- 배열을 다룰 때만 적용되는 최적화 기법이 동작하지 않아 배열 특유의 이점이 사라짐
  - `arr.test = 5` 같이 숫자가 아닌 값을 프로퍼티 키로 사용하는 경우
  - `arr[0]`과 `arr[1000]`만 추가하고 그 사이에 아무 요소도 없는 경우
  - `arr[1000]`, `arr[999]` 같이 요소를 역순으로 채우는 경우

### 성능

`push`와 `pop`은 빠르지만, `shift`와 `unshift`는 느림

### 반복문

- `for`문: 인덱스를 사용해 순회
  - 배열을 순회할 때 쓰는 가장 오래된 방법
- `for...of`: 현재 요소의 인덱스는 얻을 수 없고 값만 얻을 수 있음
  - 문법이 간단함
- `for...in`
  - 배열은 객체형에 속하므로 사용할 수 있지만 좋지 않음
  - 모든 프로퍼티(키가 숫자가 아닌 프로퍼티 포함)를 대상으로 순회
  - 객체와 함께 사용할 때 최적화되어 있어서 배열에 사용하면 객체에 사용하는 것 대비 10~100배 느림

```javascript
let arr = [1, 2, 3];
for (let i = 0; i < arr.length; i++) alert(arr[i]);
for (let item of arr) alert(item);
```

### 'length' 프로퍼티

- 배열을 조작하면 `length` 프로퍼티 자동 갱신
- 배열 내 요소의 개수가 아니고, 가장 큰 인덱스에 1을 더한 값
- 수동으로 값을 감소시키면 배열이 잘릴 수 있음
  - `arr.length = 0`으로 배열 비움

```javascript
let arr = [1, 2, 3, 4, 5];
arr.length = 9;
alert(arr); // 1,2,3,4,5,,,, -> [1, 2, 3, 4, 5, 비어 있음 × 4]
arr.length = 2;
alert(arr); // 1,2
arr.length = 5; // 원래 길이 복구
alert(arr[3]); // undefined
arr.length = 0; // 배열 비우기
```

### new Array()

`new Array(num)`

- 생성된 배열에는 요소가 없으나 길이는 인수와 같아짐
- 대괄호로 배열을 생성하는 것을 추천

```javascript
let arr = new Array(2);
alert(arr[0]); // undefined
alert(arr.length); // 2
arr = new Array("사과", "배", "오렌지");
```

### 다차원 배열(Multidimensional array)

이차원 배열 만들기

```javascript
let graph = new Array(3).fill(0).map(() => new Array(3).fill(0));
graph = Array.from(Array(3), () => new Array(3).fill(0));
console.log(graph); // [ [ 0, 0, 0 ], [ 0, 0, 0 ], [ 0, 0, 0 ] ]
```

### toString

- 요소를 쉼표로 구분한 문자열 반환
- 배열에는 `Symbol.toPrimitive`나 `valueOf` 메서드가 없음

```javascript
let arr = [1, 2, 3];
alert(arr); // 1,2,3
alert(arr.toString()); // 1,2,3
alert(String(arr) === "1,2,3"); // true
alert([] + 1); // 1
alert([1] + 1); // 11
alert([1, 2] + 1); // 1,21
alert([1] + [2, 3]); // 12,3
```

### 예제

배열 컨텍스트에서 함수 호출하기

- `arr[2]()`: `obj`가 `arr`이고, `method`는 `2`인 `obj[method]()`를 호출하는 것과 문법적으로 동일
- 즉 `arr[2]`에 있는 함수가 객체 메서드처럼 호출됨
- 따라서 `arr[2]`는 `arr`을 참조하는 `this`를 받고, 배열을 출력

```javascript
let arr = ["a", "b"];
arr.push(function () {
  console.log(this);
});
arr[2](); // a,b,function(){...}
```

## 배열과 메서드

### 요소 추가·제거 메서드

- `delete`: 해당 키의 값만 지우고 빈 공간은 채우지 않아 배열의 길이는 변하지 않음
  - 객체형에 속하기 때문에 사용은 가능
- `arr.splice(index[, deleteCount, elem1, ..., elemN])`: 요소 추가, 삭제, 교체
  - `index`: 시작할 요소 인덱스. 음수이면 배열 끝에서부터 셈
  - `deleteCount`: 제거할 요소의 개수. `0`이면 제거하지 않음
  - `elem1, ..., elemN`: 배열에 추가할 요소
  - 삭제된 요소로 구성된 배열 반환
- `arr.slice([start], [end])`: 인수 없이 호출하면 `arr`의 복사본 생성
  - `start`부터 `end` 전까지의 요소를 복사한 새로운 배열 반환. `start`, `end`는 음수 가능
- `arr.concat(arg1, arg2...)`: 기존 배열에 요소 추가
  - `arr`의 모든 요소와 `arg1`, `arg2`, ...의 모든 요소를 모은 새로운 배열 반환
  - 인수 `argN`이 배열이면 배열의 모든 요소 복사
  - 객체나 유사 배열 객체는 분해되지 않고 통째로 복사
- `Symbol.isConcatSpreadable`
  - 인자로 받은 유사 배열 객체에 특수한 프로퍼티 `Symbol.isConcatSpreadable`이 있는 경우
  - 이 객체를 배열처럼 취급해서 객체 전체가 아닌 객체 프로퍼티의 값이 더해짐

```javascript
// delete
let arr = ["I", "go", "home"];
delete arr[1];
alert(arr[1]); // undefined
alert(arr.length); // 3. arr = ["I", , "home]

// splice
let arr = ["I", "study", "JavaScript", "right", "now"];
arr.splice(1, 1); // ["study"]
arr; // ["I", "JavaScript"]
arr.splice(0, 2, "Let's", "dance"); // ["I", "JavaScript"]
arr; // ["Let's", "dance", "right", "now"]
let arr2 = [1, 2, 5];
arr2.splice(-1, 0, 3, 4); // []
alert(arr2); // 1,2,3,4,5

// slice
let chars = ["t", "e", "s", "t"];
alert(chars.slice(1, 3)); // e,s
alert(chars.slice(-2)); // s,t
let arr = [1, 2, 3];
let copy = arr.slice(); // [1, 2, 3]
alert(copy); // 1,2,3
copy.push(4);
alert(arr); // 1,2,3
arr.splice(); // []
arr.slice(); // [1, 2, 3]
let arr2 = [...arr];
let arr3 = arr.slice();

// concat
let arr = [1, 2];
arr.concat(3, 4); // [1, 2, 3, 4]
arr.concat([5, 6], 7, [8, 9]); // [1, 2, 5, 6, 7, 8, 9]
arr; // [1, 2]
let arr2 = [1, 2, 3];
let arrayLike = { 0: "something", length: 1 };
alert(arr2.concat(arrayLike)); // 1,2,[object Object]

// Symbol.isConcatSpreadable
let arr = [1, 2];
let arrayLike = {
  0: "a",
  1: "b",
  [Symbol.isConcatSpreadable]: true,
  length: 2
};
alert(arr.concat(arrayLike)); // 1,2,a,b

// splice와 concat
let arr = [1, 2];
arr.splice(0, 0, 3, 4, [5, 6]); // []
arr; // [3, 4, [5, 6], 1, 2]
let arr2 = [1, 2];
arr2.concat(3, 4, [5, 6]); // [1, 2, 3, 4, 5, 6]
arr2; // [1, 2]
```

### forEach로 반복 작업하기

`arr.forEach`

```javascript
arr.forEach(function (item, index, array) {...})
```

- 주어진 함수를 배열 요소 각각에 대해 실행
- 인수로 넘겨준 함수의 반환값은 무시됨

```javascript
["Bilbo", "Gandalf", "Nazgul"].forEach(alert); // Bilbo, Gandalf, Nazgul
["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {
  alert(`${item} is at index ${index} in ${array}`);
});
```

### 배열 탐색하기

- `arr.indexOf(item, from)`: `from`부터 시작해 `item`을 찾음
  - 찾으면 해당 요소의 인덱스 반환하고 못 찾으면 `-1` 반환
  - 요소를 찾을 때 `===` 사용
- `arr.lastIndexOf(item, from)`: 끝에서부터 요소 탐색
- `arr.includes(item, from)`: 해당 요소를 찾으면 `true` 반환
  - `NaN`도 제대로 처리하기 때문에, 정확한 위치가 아닌 배열 내 존재하는지 여부만 알고 싶을 때 유용
- `arr.find(fn)`: 객체로 이루어진 배열에서, 특정 조건에 부합하는 객체 하나를 찾는 데 유용
  - `fn`이 `true`를 반환하면 반복이 멈추고 해당 요소 반환
  - 조건에 해당하는 요소가 없으면 `undefined` 반환
- `arr.filter(fn)`: 조건을 충족하는 요소가 여러 개일 때 사용
  - 조건에 맞는 요소 전체를 담은 배열을 반환
  - 조건에 맞는 요소가 하나도 없으면 빈 배열 반환
- `arr.at(index)`: 주어진 인덱스와 일치하는 배열 요소 반환
  - `index`: 음수이면 뒤에서부터 셈
  - 범위를 벗어나면 `undefined` 반환

```javascript
let result = arr.find(function (item, index, array) {});
let results = arr.filter(function (item, index, array) {});
```

- `item`: 함수를 호출할 요소
- `index`: 요소의 인덱스
- `array`: 배열 자기 자신

```javascript
// arr.indexOf(), arr.includes()
const arr = [NaN];
alert(arr.indexOf(NaN)); // -1 (NaN === NaN 이라서 동작하지 않음)
alert(arr.includes(NaN)); // true

// arr.find()
let users = [
  { id: 1, name: "John" },
  { id: 2, name: "Pete" },
  { id: 3, name: "Mary" }
];
let user = users.find((item) => item.id === 1);
alert(user.name); // John

// arr.filter()
let users = [
  { id: 1, name: "John" },
  { id: 2, name: "Pete" },
  { id: 3, name: "Mary" }
];
let someUsers = users.filter((item) => item.id < 3);
alert(someUsers.name); // 2

// arr.at()
let arr = [1, 2, 3];
arr.at(0); // 1
arr.at(-1); // 3
arr.at(4); // undefined
arr.at(-3); // 1

// 맨 마지막 요소 가져오기 (length, slice, at)
let arr = [1, 2, 3];
arr[arr.length - 1]; // 3
arr.slice(-1)[0]; // 3
arr.at(-1); // 3
```

### 배열을 변형하는 메서드

- `arr.sort(fn)`: 배열 요소를 정렬
  - 정렬된 배열을 반환하지만 이미 배열 자체가 수정되었기 때문에 반환 값은 잘 사용하지 않음
  - 요소는 기본적으로 문자열로 취급해 정렬하기 때문에 함수를 넘겨 새로운 정렬 기준을 만듦
  - 정렬 함수의 반환 값에는 제약이 없음
  - 반환 값이 양수인 경우 첫 번째 인수가 두 번째 인수보다 '크다'를 나타냄
  - 음수인 경우 첫 번째 인수가 두 번째 인수보다 '작다'를 나타내기만 하면 됨
  - 문자열에는 `localeCompare`를 사용하는 것을 권장
- `arr.reverse()`: 요소를 역순으로 정렬
  - 역순으로 정렬된 배열 반환
- 이 아래는 배열을 변형하지 않음
- `arr.map(fn)`: 배열 요소 전체를 대상으로 함수를 호출하고, 그 결과를 배열로 반환
  - 유용성과 사용 빈도가 아주 높음
- `arr.join(glue)`: `glue`를 배열 요소 사이에 붙여 하나의 문자열을 만들어 반환
- `arr.reduce(fn)`
  - 배열을 기반으로 값 하나를 도출할 때 사용
  - `accumulator`: 이전 함수의 호출 결과
    - 다음 함수를 호출할 때 첫 번째 인수로 사용됨 (previousValue)
  - `initial`: 함수 최초 호출 시 사용되는 초깃값
    - 없으면 배열의 첫 번째 요소를 초깃값으로 사용하고 두 번째 요소부터 함수를 호출
    - 배열이 비어있다면 에러 발생
  - 마지막 함수까지 호출되면 `accumulator`의 값이 반환 값이 됨
- `arr.reduceRight(fn)`: 동일하나 배열의 오른쪽부터 연산을 수행
- `str.split(delim)`: 구분자(delimiter) `delim`을 기준으로 문자열을 쪼갬
  - `delim`이 빈 문자열 `""`이면 `str`을 글자 단위로 분리
  - 두 번째 인수로 숫자를 받아 배열의 길이 제한

```javascript
let result = arr.map(function (item, index, array) {});
```

```javascript
let value = arr.reduce(function (accumulator, item, index, array) {}, [
  initial
]);
```

```javascript
// arr.sort()
let arr = [1, 2, 15];
arr.sort();
alert(arr); // 1,15,2
function compareNumeric(a, b) {
  if (a > b) return 1;
  if (a == b) return 0;
  if (a < b) return -1;
}
arr.sort(compareNumeric);
alert(arr); // 1,2,15
let arr2 = [1, 2, 15];
arr2.sort((a, b) => a - b);
alert(arr2); // 1,2,15
let countries = ["Österreich", "Andorra", "Vietnam"];
alert(countries.sort((a, b) => (a > b ? 1 : -1))); // Andorra, Vietnam, Österreich (제대로 정렬이 되지 않음)
alert(countries.sort((a, b) => a.localeCompare(b))); // Andorra,Österreich,Vietnam (제대로 정렬됨)

// arr.reverse()
let arr = [1, 2, 3, 4, 5];
arr.reverse(); // [5, 4, 3, 2, 1]
alert(arr); // 5,4,3,2,1

// arr.map()
let lengths = ["Bilbo", "Gandalf", "Nazgul"].map((item) => item.length);
alert(lengths); // 5,7,6

// arr.join()
let arr = ["Bilbo", "Gandalf", "Nazgul"];
let str = arr.join(";");
alert(str); // Bilbo;Gandalf;Nazgul

// arr.reduce()
let arr = [1, 2, 3, 4, 5];
let result = arr.reduce((acc, cur) => acc + cur, 0);
alert(result); // 15
let arr2 = [];
arr2.reduce((acc, cur) => acc + cur); // TypeError: Reduce of empty array with no initial value
arr2.reduce((acc, cur) => acc + cur, 0); // 0

// str.split()
let names = "Bilbo, Gandalf, Nazgul";
let arr = names.split(", ");
for (let name of arr) alert(`${name}에게 보내는 메시지`);

// 문자열 쪼개기
let str = "test";
alert(str.split("")); // t,e,s,t
[..."123"]; // ["1", "2", "3"]
"abc".split(""); // ["a", "b", "c"]
```

### Array.isArray로 배열 여부 알아내기

`Array.isArray`

- 자바스크립트에서 배열은 독립된 자료형으로 취급되지 않고 객체형에 속함
- `typeof`로는 일반 객체와 배열 구분 불가
- `Array.isArray(value)`로 구분

```javascript
alert(typeof {}); // object
alert(typeof []); // object
alert(Array.isArray({})); // false
alert(Array.isArray([])); // true
```

### 배열 메서드와 'thisArg'

`thisArg`

- 함수를 호출하는 대부분의 배열 메서드는 `thisArg`라는 매개변수를 옵션으로 받을 수 있음
  - find, filter, map 등. sort는 제외
- `thisArg`는 `func`의 `this`

```javascript
arr.find(func, thisArg);
arr.filter(func, thisArg);
arr.map(func, thisArg);
```

```javascript
let army = {
  minAge: 18,
  maxAge: 27,
  canJoin(user) {
    return user.age >= this.minAge && user.age < this.maxAge;
  }
};
let users = [{ age: 16 }, { age: 20 }, { age: 23 }, { age: 30 }];
let soldiers = users.filter(army.canJoin, army); // army.canJoin 호출 시 참을 반환해주는 user를 찾음
alert(soldiers.length); // 2
alert(soldiers[0].age); // 20
alert(soldiers[1].age); // 23
```

- `thisArg`에 `army`를 지정하지 않고 단순히 `users.filter(army.canJoin)`을 사용한 경우
  - `army.canJoin`은 단독 함수처럼 취급되고 함수 본문 내 `this`는 `undefined`가 되어 에러 발생
- `users.filter(user => army.canJoin(user))`를 사용할 수도 있음
  - `thisArg`를 사용하는 방식(`users.filter(army.canJoin, army)`)이 좀 더 이해하기 쉬워 더 자주 사용됨

### 이외의 배열 메서드

- `arr.some(fn)`, `arr.every(fn)`: 모든 요소를 대상으로 함수 `fn` 호출
  - `some`: 반환 값을 `true`로 만드는 요소가 하나라도 있는지 확인
  - `every`: 모든 요소가 함수의 반환 값을 `true`로 만드는지 확인
  - 조건을 충족하면 `true`, 그렇지 않으면 `false` 반환
- `arr.fill(value, start, end)`: `start`부터 `end` 전까지 `value`로 채움
- `arr.copyWithin(target, start, end)`: `start`부터 `end`까지 요소를 복사하고 `target`에 붙여 넣음
  - 기존 요소가 있다면 덮어씀

## iterable 객체

- 배열을 일반화한 객체
- 반복 가능한(iterable) 객체
- 이터러블 개념을 사용하면 어떤 객체에든 `for..of` 반복문 적용 가능
- 배열은 대표적인 이터러블이고 문자열도 이터러블

### Symbol.iterator

객체를 이터러블로 만들기

0. 객체에 `Symbol.iterator`(특수 내장 심볼) 메서드 추가
1. `for..of`는 시작되자마자 `Symbol.iterator`를 호출
   - 없으면 에러 발생
   - `Symbol.iterator`는 반드시 이터레이터를 반환해야 함
   - iterator: 메서드 `next`가 있는 객체
2. 이후 `for..of`는 반환된 객체(이터레이터)만을 대상으로 동작
3. `for..of`에 다음 값이 필요하면 `for..of`는 이터레이터의 `next()` 메서드를 호출
4. `next()`의 반환 값은 `{ done: Boolean, value: any }`와 같은 형태여야 함
   - `done`: `true`이면 반복 종료, `false`이면 `value`에 다음 값 저장됨

```javascript
let notIterable = { from: 1, to: 5 };
for (let num of notIterable) alert(num); //TypeError: range is not iterable

// range를 이터러블로 만들기
let range = { from: 1, to: 5 };
range[Symbol.iterator] = function () {
  return {
    current: this.from,
    last: this.to,
    next() {
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};
for (let num of range) alert(num); // 1, 2, 3, 4, 5
```

이터러블 객체의 핵심

- 관심사의 분리(Separation of Concern, SoC)
- `range`에는 메서드 `next()`가 없음
- 대신 `range[Symbol.iterator]()`를 호출해서 만든 이터레이터 객체와 이 객체의 메서드 `next()`에서 반복에 사용될 값 생성
- 이를 통해 이터레이터 객체와 반복 대상인 객체 분리 가능

이터레이터 객체와 반복 대상 객체 합치기

- `range` 자체를 이터레이터로 만들면 코드가 더 간단해짐
- `range[Symbol.iterator]()`가 객체 `range` 자체를 반환
  - 필수 메서드 `next()` 있음
  - `this.current`에 반복이 얼마나 진행되었는지를 나타내는 값도 저장됨
- 코드가 짧아지나 두 개의 `for..of` 반복문을 하나의 객체에 동시에 사용할 수 없음
  - 이터레이터(객체 자신)가 하나뿐이어서 두 반복문이 반복 상태를 공유하기 때문
  - 그러나 동시에 두 개의 `for..of`를 사용하는 것은 비동기 처리에서도 흔하지는 않음

```javascript
let range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    this.current = this.from;
    return this;
  },
  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    } else {
      return { done: true };
    }
  }
};
for (let num of range) alert(num); // 1, 2, 3, 4, 5
```

무한 개의 이터레이터

- `range.to`가 `Infinity`이면 `range`가 무한대가 됨
- `next`가 값을 계속 반환하는 것은 정상적인 동작
  - `next`에는 제약 사항 없음
- `break`를 사용해 언제든 반복을 멈출 수 있음

### 문자열은 이터러블입니다

- `for..of`: 문자열의 각 글자를 순회
- 서로게이트 쌍(surrogate pair)에도 잘 동작

```javascript
for (let char of "test") alert(char); // t, e, s, t
```

### 이터레이터를 명시적으로 호출하기

- `for..of`를 사용하는 것보다 반복 과정을 더 잘 통제
- 반복 과정을 여러 개로 쪼갤 수 있음
  - 반복을 시작했다가 잠시 멈춰 다른 작업을 하다가 다시 반복을 시작

```javascript
let str = "Hello";
let iterator = str[Symbol.iterator](); // for (let char of str) {...} 와 동일한 작업을 함
while (true) {
  let { done, value } = iterator.next();
  if (done) break;
  alert(value); // H, e, l, l, o
}
```

### 이터러블과 유사 배열

이터러블(iterable)

- 메서드 `Symbol.iterator`가 구현된 객체
- 이터러블 객체라고 해서 유사 배열 객체는 아님
- 배열이 아니기 때문에 `push`, `pop` 등의 메서드를 지원하지 않음

유사 배열(array-like)

- 인덱스, `length 프로퍼티`가 있어 배열처럼 보이는 객체
- 유사 배열 객체라고 해서 이터러블 객체는 아님
- 배열이 아니기 때문에 `push`, `pop` 등의 메서드를 지원하지 않음

문자열

- 이터러블 객체: `for..of` 사용
- 유사 배열 객체: 숫자 인덱스, `length` 프로퍼티 있음
- 이터러블 객체이면서 유사 배열 객체

```javascript
let arrayLike = { 0: "Hello", 1: "World", length: 2 };
// Symbol.iterator 없으므로 에러 발생
for (let item of arrayLike) {...} // TypeError: arrayLike is not iterable
```

### Array.from

`Array.from(obj[, mapFn, thisArg])`

- 이터러블이나 유사 배열을 받아 진짜 `Array` 생성
- 인수가 이터러블이나 유사 배열이면 새로운 배열 생성하고, 객체의 모든 요소를 새롭게 만든 배열로 복사
- `mapFn`: 각 요소를 대상으로 `mapFn` 적용하고 반환된 값을 새로운 배열에 추가
- `thisArg`: 각 요소의 `this`를 지정
- 서로게이트 쌍
  - `str.split`과 달리 문자열 자체가 가진 이터러블 속성을 이용해 동작하기 때문에 `for..of`처럼 서로게이트 쌍에도 제대로 적용됨
  - 내장 순수 메서드는 서로게이트 쌍을 지원하지 않음
  - 서로게이트 쌍을 처리할 수 있는 `slice` 직접 구현 가능

```javascript
// array-like -> array
let arrayLike = { 0: "Hello", 1: "World", length: 2 };
let arr = Array.from(arrayLike);
alert(arr.pop()); // World

// iterable -> array
let range = { ..iterable.. };
let arr2 = Array.from(range);
alert(arr2); // 1,2,3,4,5
let arr3 = Array.from(range, (num) => num * num);
alert(arr3); // 1,4,9,16,25

// 서로게이트 쌍
let str = "𝒳😂";
let chars = Array.from(str);
alert(char[0]); // 𝒳
alert(char[1]); // 😂
alert(chars.length); // 2
// 위 코드는 아래 코드와 기술적으로 동일하게 동작
let chars = []; // Array.from 내부에서는 아래와 동일한 반복문 동작
for (let char of str) chars.push(char);

// 서로게이트 쌍을 처리하는 커스텀 slice
function slice(str, start, end) {
  return Array.from(str).slice(start, end).join("");
}
let str = "𝒳😂𩷶";
alert(slice(str, 1, 3)); // 😂𩷶 (커스텀 slice)
alert(str.slice(1, 3)); // �� 쓰레깃값 출력 (영역이 다른 특수 값)
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
