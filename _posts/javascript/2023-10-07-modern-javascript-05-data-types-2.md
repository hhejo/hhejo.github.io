---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 2
date: 2023-10-05 08:19:33 +0900
last_modified_at: 2023-10-14 07:55:14 +0900
categories: [JavaScript]
tags: [javascript]
---

배열, 배열 메서드, iterable

## 배열

순서가 있는 컬렉션을 저장할 때 사용

객체는 순서를 고려하지 않고 만들어진 자료구조로 새로운 프로퍼티를 기존 프로퍼티 사이에 끼워넣기 불가능

### 배열 선언

```javascript
let arr = new Array();
let arr = [];
```

대괄호 안에 인덱스를 넣어 배열 내 특정 요소를 얻거나, 수정할 수 있음

새로운 요소를 배열에 추가할 수도 있음

`length`

- 배열의 요소 개수 반환

배열의 요소의 자료형에 제약은 없음

```javascript
let fruits = ["사과", "오렌지", "자두"];
fruits[2] = "배"; // 수정
fruits[3] = "레몬"; // 추가
alert(fruits.length); // 4
let arr = [
  "사과",
  { name: "이보라" },
  true,
  function () {
    alert("안녕하세요.");
  }
];
alert(arr[1].name); // 이보라
arr[3](); // 안녕하세요.
```

trailing 쉼표

- 배열의 마지막 요소는 객체와 마찬가지로 쉼표로 끝날 수 있음
- 모든 줄의 생김새가 유사해지기 때문에 요소를 넣거나 빼기 쉬워짐

### pop·push와 shift·unshift

`push`

- 배열 맨 끝에 요소 추가
- 여러 개를 한번에 추가 가능
- `arr.push(...)`는 `arr.[arr.length] = ...`와 같은 효과
- 배열의 새로운 길이 반환

`pop`

- 배열 맨 끝 요소 추출

`shift`

- 배열 제일 앞 요소를 꺼내 제거한 후 남은 요소들을 앞으로 밀음

`unshift`

- 배열 제일 앞에 요소 추가
- 여러 개를 한번에 추가 가능

### 배열의 내부 동작 원리

- 배열은 특별한 종류의 객체(키가 숫자이고, 본질은 객체)
- 배열은 객체이므로 원하는 프로퍼티를 객체처럼 추가할 수 있음
- 그러나 그렇게 작성하면 자바스크립트 엔진이 배열을 일반 객체처럼 다루게 되어, 배열을 다룰 때만 적용되는 최적화 기법이 동작하지 않아 배열 특유의 이점이 사라짐
  - `arr.test = 5` 같이 숫자가 아닌 값을 프로퍼티 키로 사용하는 경우
  - `arr[0]`과 `arr[1000]`만 추가하고 그 사이에 아무 요소도 없는 경우
  - `arr[1000]`, `arr[999]` 같이 요소를 역순으로 채우는 경우

### 성능

`push`와 `pop`은 빠르지만, `shift`와 `unshift`는 느림

### 반복문

`for`문

- 배열을 순회할 때 쓰는 가장 오래된 방법으로 순회 시에 인덱스 사용

`for...of`문

- 현재 요소의 인덱스는 얻을 수 없고 값만 얻을 수 있지만 문법이 간단
- 배열은 객체형에 속하므로 `for...in`을 사용할 수 있지만 좋지 않음
  - 모든 프로퍼티를 대상으로 순회하기 때문에 키가 숫자가 아닌 프로퍼티도 순회 대상에 포함
  - 배열이 아니라 객체와 함께 사용할 때 최적화되어 있어서 배열에 사용하면 객체에 사용하는 것 대비 10~100배 느림

### 'length' 프로퍼티

배열에 조작을 가하면 `length` 프로퍼티 자동 갱신

`length` 프로퍼티는 배열 내 요소의 개수가 아니라 가장 큰 인덱스에 1을 더한 값

수동으로 조작해 값을 감소시키면 배열이 잘릴 수 있음

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

숫자형 인수 하나를 넣어 `new Array()`를 호출하면 배열이 만들어지는데, 이 배열에는 요소가 없는 반면 길이는 인수와 같아짐

```javascript
let arr = new Array(2);
alert(arr[0]); // undefined
alert(arr.length); // 2
arr = new Array("사과", "배", "오렌지");
```

대괄호로 배열을 생성하는 것을 추천

### 다차원 배열(Multidimensional array)

배열의 요소가 배열

### toString

요소를 쉼표로 구분한 문자열 반환

배열엔 `Symbol.toPrimitive`나 `valueOf` 메서드가 없음

```javascript
let arr = [1, 2, 3];
alert(arr); // 1,2,3
alert(String(arr) === "1,2,3"); // true
alert([] + 1); // 1
alert([1] + 1); // 11
alert([1, 2] + 1); // 1,21
```

## 배열과 메서드

### 요소 추가·제거 메서드

객체형에 속하기 때문에 `delete`를 사용할 수 있으나 배열의 길이는 변하지 않음

- 해당 키에 상응하는 값을 지우기 때문
- 빈 공간을 채우지는 않음

```javascript
let arr = ["I", "go", "home"];
delete arr[1];
alert(arr[1]); // undefined
alert(arr.length); // 3
```

`arr.splice(index[, deleteCount, elem1, ..., elemN])`

- 요소 추가, 삭제, 교체 가능
- `index`: 조작을 가할 첫 번째 요소를 가리키는 인덱스
- `deleteCount`: 제거할 요소의 개수
- `elem1, ..., elemN`: 배열에 추가할 요소
- 삭제된 요소로 구성된 배열 반환
- `deleteCount`를 `0`으로 설정하면 요소를 제거하지 않으면서 새로운 요소 추가 가능
- 음수 인덱스 가능(배열 끝에서부터 셈)

```javascript
let arr = ["I", "study", "JavaScript"];
arr.splice(1, 1); // ["study"]
arr; // ["I", "JavaScript"]
```

```javascript
let arr = ["I", "study", "JavaScript", "right", "now"];
arr.splice(0, 3, "Let's", "dance"); // ["I", "study", "JavaScript"]
arr; // ["Let's", "dance", "right", "now"]
```

```javascript
let arr = ["I", "study", "JavaScript"];
arr.splice(2, 0, "complex", "language"); // []
arr; // ["I", "study", "complex", "language", "JavaScript"]
```

```javascript
let arr = [1, 2, 5];
arr.splice(-1, 0, 3, 4); // []
alert(arr); // 1,2,3,4,5
```

`arr.slice([start], [end])`

- `start` 인덱스부터 `end` 인덱스까지(`end` 제외)의 요소를 복사한 새로운 배열 반환
- 둘 다 음수 가능
- `str.slice`와 다르게 서브 문자열 대신 서브 배열을 반환
- `arr.slice()`로 인수를 하나도 넘기지 않고 호출해 `arr`의 복사본 생성 가능

```javascript
let arr = ["t", "e", "s", "t"];
alert(arr.slice(1, 3));
alert(arr.slice(-2));
```

```javascript
let arr = [1, 2, 3];
let arr2 = arr.slice(); // [1, 2, 3]
alert(arr2); // 1,2,3
arr2.push(4);
alert(arr); // 1,2,3
```

```javascript
let arr = [1, 2, 3];
arr.splice(); // []
arr.slice(); // [1, 2, 3]
```

`arr.concat(arg1, arg2...)`

- 기존 배열의 요소를 사용해 새로운 배열을 만들거나 기존 배열에 요소 추가
- 인수 개수에 제한이 없고 인수로 배열이나 값이 옴
- `arr`에 속한 모든 요소와 `arg1`, `arg2` 등에 속한 모든 요소를 한데 모은 새로운 배열 반환
- 인수 `argN`이 배열일 경우 배열의 모든 요소가 복사됨
- 객체가 인자로 넘어오면(배열처럼 보이는 유사 배열 객체라도) 객체는 분해되지 않고 통으로 복사
- 인자로 받은 유사 배열 객체에 특수한 프로퍼티 `Symbol.isConcatSpreadable`이 있으면 이 객체를 배열처럼 취급해서 객체 전체가 아닌 객체 프로퍼티의 값이 더해짐

```javascript
let arr = [1, 2];
arr.concat(3, 4); // [1, 2, 3, 4]
arr.concat([5, 6], 7, [8, 9]); // [1, 2, 5, 6, 7, 8, 9]
arr; // [1, 2]
```

```javascript
let arr = [1, 2, 3];
let arrayLike = { 0: "something", length: 1 };
alert(arr.concat(arrayLike)); // 1,2,[object Object]
```

```javascript
let arr = [1, 2];
let arrayLike = {
  0: "a",
  1: "b",
  [Symbol.isConcatSpreadable]: true,
  length: 2
};
alert(arr.concat(arrayLike)); // 1,2,a,b
```

### forEach로 반복 작업하기

`arr.forEach`

- 주어진 함수를 배열 요소 각각에 대해 실행
- 인수로 넘겨준 함수의 반환값은 무시됨

```javascript
arr.forEach(function (item, index, array) {...});
```

```javascript
["Bilbo", "Gandalf", "Nazgul"].forEach(alert); // Bilbo, Gandalf, Nazgul
["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {
  alert(`${item} is at index ${index} in ${array}`);
});
// Bilbo is at index 0 in Bilbo,Gandalf,Nazgul
// Gandalf is at index 1 in Bilbo,Gandalf,Nazgul
// Nazgul is at index 2 in Bilbo,Gandalf,Nazgul
```

### 배열 탐색하기

`arr.indexOf(item, from)`

- `from`부터 시작해 `item`을 찾음
- 요소를 발견하면 해당 요소의 인덱스 반환
- 발견하지 못하면 `-1` 반환
- 요소를 찾을 때 `===`를 사용

`arr.lastIndexOf(item, from)`

- 검색을 끝에서부터 시작

`arr.includes(item, from)`

- 해당하는 요소를 발견하면 `true` 반환
- `NaN`도 제대로 처리하기 때문에, 정확한 위치가 아닌 배열 내 존재하는지 여부만 알고 싶을 때 좋음

```javascript
const arr = [NaN];
alert(arr.indexOf(NaN)); // -1 (NaN === NaN 이라서 동작하지 않음)
alert(arr.includes(NaN)); // true
```

`arr.find(fn)`

- 객체로 이루어진 배열에서, 특정 조건에 부합하는 객체를 찾는 데 유용
- `arr.findIndex`도 동일한 일을 하지만, 요소를 반환하지 않고 인덱스를 반환
- 실무에선 객체로 구성된 배열을 많이 다루기 때문에 유용

```javascript
let result = arr.find(function (item, index, array) {
  // true가 반환되면 반복이 멈추고 해당 요소 반환
  // 조건에 해당하는 요소가 없으면 undefined 반환
});
```

- `item`: 함수를 호출할 요소
- `index`: 요소의 인덱스 (잘 쓰이지 않음)
- `array`: 배열 자기 자신

```javascript
let users = [
  { id: 1, name: "John" },
  { id: 2, name: "Pete" },
  { id: 3, name: "Mary" }
];
let user = users.find((item) => item.id === 1);
alert(user.name); // John
```

`arr.filter(fn)`

- 조건을 충족하는 요소가 여러 개일 때 사용
- 조건에 맞는 요소 전체를 담은 배열을 반환
- `find`는 하나, `filter`는 여러 개 탐색

```javascript
let results = arr.filter(function (item, index, array) {
  // 조건을 충족하는 요소는 results에 순차적으로 더해짐
  // 조건을 충족하는 요소가 하나도 없으면 빈 배열 반환
});
```

```javascript
let users = [
  { id: 1, name: "John" },
  { id: 2, name: "Pete" },
  { id: 3, name: "Mary" }
];
let someUsers = users.filter((item) => item.id < 3);
alert(someUsers.name); // 2
```

### 배열을 변형하는 메서드

`arr.map(fn)`

- 배열 요소 전체를 대상으로 함수를 호출하고, 그 결과를 배열로 반환

```javascript
let result = arr.map(function (item, index, array) {
  // 요소 대신 새로운 값 반환
});
```

```javascript
let lengths = ["Bilbo", "Gandalf", "Nazgul"].map((item) => item.length);
alert(lengths); // 5,7,6
```

`arr.sort(fn)`

- 배열 요소를 정렬
- 재정렬된 배열을 반환하지만 이미 배열 자체가 수정되었기 때문에 반환 값은 잘 사용하지 않음
- 요소는 기본적으로 문자열로 취급해 정렬
- 새로운 정렬 기준을 만드려면 함수를 넘겨줘야 함

```javascript
let arr = [1, 2, 15];
arr.sort();
alert(arr); // 1,15,2
```

```javascript
function compareNumeric(a, b) {
  if (a > b) return 1;
  if (a == b) return 0;
  if (a < b) return -1;
}
let arr = [1, 2, 15];
arr.sort(compareNumeric);
alert(arr); // 1,2,15
```

- 정렬 함수의 반환 값에는 제약이 없음. 반환 값이 양수인 경우 첫 번째 인수가 두 번째 인수보다 '크다'를 나타내고, 음수인 경우 첫 번째 인수가 두 번째 인수보다 '작다'를 나타내기만 하면 됨

```javascript
let arr = [1, 2, 15];
arr.sort((a, b) => a - b);
alert(arr); // 1,2,15
```

- 문자열엔 `localeCompare`를 사용하는 것을 권장

```javascript
let countries = ["Österreich", "Andorra", "Vietnam"];
alert(countries.sort((a, b) => (a > b ? 1 : -1))); // Andorra, Vietnam, Österreich (제대로 정렬이 되지 않았습니다.)
alert(countries.sort((a, b) => a.localeCompare(b))); // Andorra,Österreich,Vietnam (제대로 정렬되었네요!)
```

`arr.reverse()`

- 요소를 역순으로 정렬
- 반환 값은 재정렬된 배열

```javascript
let arr = [1, 2, 3, 4, 5];
arr.reverse(); // [5, 4, 3, 2, 1]
alert(arr); // 5,4,3,2,1
```

`str.split(delim)`

- 구분자(delimiter) `delim`을 기준으로 문자열을 쪼갬
- 두 번째 인수로 숫자를 받아 배열의 길이를 제한해줄 수 있음
- `delim`을 빈 문자열로 지정하면 문자열을 글자 단위로 분리할 수 있음

```javascript
let names = "Bilbo, Gandalf, Nazgul";
let arr = names.split(", ");
for (let name of arr) {
  alert(`${name}에게 보내는 메시지`);
}
```

```javascript
let str = "test";
alert(str.split("")); // t,e,s,t
```

`arr.join(glue)`

- `glue`를 배열 요소 사이에 붙여 하나의 문자열을 만들어 반환

```javascript
let arr = ["Bilbo", "Gandalf", "Nazgul"];
let str = arr.join(";");
alert(str); // Bilbo;Gandalf;Nazgul
```

`arr.reduce(fn)`

- 배열을 기반으로 값 하나를 도출할 때 사용
- `accumulator`: 이전 함수의 호출 결과. 다음 함수를 호출할 때 첫 번째 인수로 사용됨 (previousValue)
- `initial`: 함수 최초 호출 시 사용되는 초깃값. 없으면 배열의 첫 번째 요소를 초깃값으로 사용하고 두 번째 요소부터 함수를 호출. 배열이 비어있다면 에러 발생
- 마지막 함수까지 호출되면 `accumulator`의 값이 반환 값이 됨

```javascript
let value = arr.reduce(
  function (accumulator, item, index, array) {
    // ...
  },
  [initial]
);
```

```javascript
let arr = [1, 2, 3, 4, 5];
let result = arr.reduce((acc, cur) => acc + cur, 0);
alert(result); // 15
```

```javascript
let arr = [];
arr.reduce((acc, cur) => acc + cur); // TypeError: Reduce of empty array with no initial value
arr.reduce((acc, cur) => acc + cur, 0); // 15
```

`arr.reduceRight(fn)`

- 동일하나 배열의 오른쪽부터 연산을 수행

### Array.isArray로 배열 여부 알아내기

자바스크립트에서 배열은 독립된 자료형으로 취급하지 않고 객체형에 속함

`typeof`로는 일반 객체와 배열을 구분할 수 없음

```javascript
alert(typeof {}); // object
alert(typeof []); // object
```

`Array.isArray(value)`로 감별 가능

```javascript
alert(Array.isArray({})); // false
alert(Array.isArray([])); // true
```

### 배열 메서드와 'thisArg'

함수를 호출하는 대부분의 배열 메서드(find, filter, map 등. sort는 제외)는 `thisArg`라는 매개변수를 옵션으로 받을 수 있음

`thisArg`는 `func`의 `this`가 됨

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
// army.canJoin 호출 시 참을 반환해주는 user를 찾음
let soldiers = users.filter(army.canJoin, army);
alert(soldiers.length); // 2
alert(soldiers[0].age); // 20
alert(soldiers[1].age); // 23
```

- `thisArg`에 `army`를 지정하지 않고 단순히 `users.filter(army.canJoin)`을 사용했다면 `army.canJoin`은 단독 함수처럼 취급되고, 함수 본문 내 `this`는 `undefined`가 되어 에러가 발생했을 것
- `users.filter(user => army.canJoin(user))`를 사용하면 `users.filter(army.canJoin, army)`를 대체할 수 있지만 `thisArg`를 사용하는 방식이 좀 더 이해하기 쉬워 더 자주 사용됨

## iterable 객체

배열을 일반화한 객체, 반복 가능한(iterable) 객체

이터러블 개념을 사용하면 어떤 객체에든 `for..of` 반복문 적용 가능

배열은 대표적인 이터러블

### Symbol.iterator

```javascript
let range = { from: 1, to: 5 };
for (let num of range) alert(num); // TypeError: range is not iterable
```

- `range`를 이터러블로 만드려면(`for..of`가 동작하도록 하려면) 객체에 `Symbol.iterator`(특수 내장 심볼)라는 메서드를 추가해 아래와 같은 일이 벌어지도록 만들어야 함

1. `for..of`가 시작되자마자 `for..of`는 `Symbol.iterator`를 호출
   - `Symbol.iterator`가 없으면 에러 발생
   - `Symbol.iterator`는 반드시 이터레이터(iterator, 메서드 `next`가 있는 객체)를 반환해야 함
2. 이후 `for..of`는 반환된 객체(이터레이터)만을 대상으로 동작
3. `for..of`에 다음 값이 필요하면 `for..of`는 이터레이터의 `next()` 메서드를 호출
4. `next()`의 반환 값은 `{ done: Boolean, value: any }`와 같은 형태여야 함
   - `done=true`는 반복이 종료되었음을 의미
   - `done=false`일 땐 `value`에 다음 값이 저장됨

```javascript
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

이터러블 객체의 핵심은 관심사의 분리(Separation of concern, SoC)에 있음

- `range`엔 메서드 `next()`가 없음
- 대신 `range[Symbol.iterator]()`를 호출해서 만든 이터레이터 객체와 이 객체의 메서드 `next()`에서 반복에 사용될 값 생성

이를 통해 이터레이터 객체와 반복 대상인 객체 분리 가능

이터레이터 객체와 반복 대상 객체를 합쳐서 `range` 자체를 이터레이터로 만들면 코드가 더 간단해짐

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

- `range[Symbol.iterator]()`가 객체 `range` 자체를 반환
- 반환된 객체에는 필수 메서드 `next()`가 있고 `this.current`에 반복이 얼마나 진행되었는지를 나타내는 값도 저장됨
- 코드도 더 짧아져서 이렇게 작성하는 게 좋을 때가 있음
- 하지만 두 개의 `for..of` 반복문을 하나의 객체에 동시에 사용할 수 없음
  - 이터레이터(객체 자신)가 하나뿐이어서 두 반복문이 반복 상태를 공유하기 때문
  - 그러나 동시에 두 개의 `for..of`를 사용하는 것은 비동기 처리에서도 흔하지는 않음

무한 개의 이터레이터

- `range`에서 `range.to`에 `Infinity`를 할당하면 `range`가 무한대가 될 수 있음
- 무수히 많은 의사 난수(pseudorandom numbers)를 생성하는 이터러블 객체를 만드는 것도 가능
- `next`엔 제약 사항이 없고 `next`가 값을 계속 반환하는 것은 정상적인 동작
- 위와 같은 이터러블에 `for..of` 반복문을 사용하면 끝이 없겠지만 `break`를 사용해 언제든 반복을 멈출 수 있음

### 문자열은 이터러블입니다

`for..of`는 문자열의 각 글자를 순회함

```javascript
for (let char of "test") alert(char); // t, e, s, t
```

서로게이트 쌍(surrogate pair)에도 잘 동작

### 이터레이터를 명시적으로 호출하기

```javascript
let str = "Hello";
// for (let char of str) {...} 와 동일한 작업을 함
let iterator = str[Symbol.iterator]();
while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value); // H, e, l, l, o
}
```

이터레이터를 명시적으로 호출하는 경우는 거의 없지만, 이 방법을 사용하면 `for..of`를 사용하는 것보다 반복 과정을 더 잘 통제할 수 있음

반복을 시작했다가 잠시 멈춰 다른 작업을 하다가 다시 반복을 시작하는 것과 같이 반복 과정을 여러 개로 쪼개기 가능

### 이터러블과 유사 배열

이터러블(iterable)

- 메서드 `Symbol.iterator`가 구현된 객체
- 이터러블 객체라고 해서 유사 배열 객체는 아님
- 배열이 아니기 때문에 `push`, `pop` 등의 메서드를 지원하지 않음

유사 배열(array-like)

- 인덱스와 `length` 프로퍼티가 있어서 배열처럼 보이는 객체
- 유사 배열 객체라고 해서 이터러블 객체는 아님
- 배열이 아니기 때문에 `push`, `pop` 등의 메서드를 지원하지 않음

문자열

- 이터러블 객체(`for..of`를 사용할 수 있음)
- 유사 배열 객체(숫자 인덱스와 `length` 프로퍼티가 있음)
- 이터러블 객체이면서 유사 배열 객체임

```javascript
let arrayLike = { 0: "Hello", 1: "World", length: 2 };
for (let item of arrayLike) {
  // Symbol.iterator 없으므로 에러 발생
} // TypeError: arrayLike is not iterable
```

### Array.from

이터러블과 유사 배열에 배열 메서드를 적용하고 싶음

`Array.from()`

- 이터러블이나 유사 배열을 받아 진짜 `Array`를 생성
- 배열 메서드 사용 가능
- 객체를 받아 이터러블이나 유사 배열인지 조사
- 넘겨 받은 인수가 이터러블이나 유사 배열인 경우 새로운 배열을 만들고 객체의 모든 요소를 새롭게 만든 배열로 복사

```javascript
let arrayLike = { 0: "Hello", 1: "World", length: 2 };
let arr = Array.from(arrayLike);
alert(arr.pop()); // World
```

```javascript
let arr = Array.from(range); // range는 이터러블
alert(arr); // 1,2,3,4,5
```

`Array.from(obj[, mapFn, thisArg])`

- 매핑(mapping) 함수를 선택적으로 넘겨줄 수 있음
- `mapFn`을 두 번째 인수로 넘겨주면 새로운 배열에 `obj`의 요소를 추가하기 전에 각 요소를 대상으로 `mapFn` 적용 가능
- 새로운 배열엔 `mapFn`을 적용하고 반환된 값 추가
- `thisArg`는 각 요소의 `this`를 지정

```javascript
let arr = Array.from(range, (num) => num * num);
alert(arr); // 1,4,9,16,25
```

```javascript
let str = "𝒳😂";
let chars = Array.from(str);
alert(char[0]); // 𝒳
alert(char[1]); // 😂
alert(chars.length); // 2
```

- `str.split`과 달리 문자열 자체가 가진 이터러블 속성을 이용해 동작하기 때문에 `for..of`처럼 서로게이트 쌍에도 제대로 적용됨
- 위의 예시는 아래 예시와 동일하게 동작

```javascript
let str = "𝒳😂";
let chars = []; // Array.from 내부에선 아래와 동일반 반복문이 동작
for (let char of str) chars.push(char);
```

- 서로게이트 쌍을 처리할 수 있는 `slice` 직접 구현 가능

```javascript
function slice(str, start, end) {
  return Array.from(str).slice(start, end).join("");
}
let str = "𝒳😂𩷶";
alert(slice(str, 1, 3)); // 😂𩷶
// 내장 순수 메서드는 서로게이트 쌍을 지원하지 않음
alert(str.slice(1, 3)); // �� 쓰레깃값 출력 (영역이 다른 특수 값)
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
