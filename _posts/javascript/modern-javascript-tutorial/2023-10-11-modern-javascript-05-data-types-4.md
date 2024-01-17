---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 4
date: 2023-10-11 15:17:49 +0900
last_modified_at: 2024-01-17 22:10:06 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

구조 분해 할당

## 구조 분해 할당

구조 분해 할당(destructuring assignment)

- 함수에 객체나 배열에 저장된 데이터 전체가 아닌 일부만 필요한 경우
- 함수의 매개변수가 많거나, 매개변수 기본값이 필요한 경우
- 객체나 배열을 변수로 분해할 수 있게 해주는 특별한 문법
- 어떤 것을 복사한 이후에 변수로 분해(destructurize)
  - 분해 대상이 수정, 파괴되는 것이 아님

### 배열 분해하기

- 배열을 변수로 분해
- 반환 값이 배열인 메서드와 사용
- 쉼표를 사용하여 필요하지 않은 배열 요소 무시
- 할당 연산자 우측에는 모든 이터러블 가능
- 할당 연산자 좌측에는 할당할 수 있는(assignables) 것이라면 어떤 것이든 가능
- `.entries()`로 반복하기
- 두 변수에 저장된 값 교환하기

```javascript
let [var1, var2] = [item1, item2];
```

```javascript
// 배열 분해
let arr = ["John", "Smith"];
// let firstName = arr[0];
// let surname = arr[1];
let [firstName, surname] = arr; // 코드 양 감소
alert(firstName); // John
alert(surname); // Smith

// 메서드가 반환한 배열 분해
let [firstName, surname] = "John Smith".split(" ");

// 쉼표로 특정 요소 무시
let [a, , c] = ["apple", "banana", "coconut", "date"]; // banana, date 무시
alert(c); // coconut

// 이터러블 분해
let [a, b, c] = "abc";
let [one, two, three] = new Set([1, 2, 3]);

// 객체에 할당
let user = {};
[user.name, user.surname] = "John Smith".split(" ");
user; // { name: "John", surname: "Smith" }

// .entries() 분해
let user = { name: "John", age: 30 };
for (let [key, value] of Object.entries(user)) alert(`${key}:${value}`);

// in을 사용하면 제대로 안 나옴
let user2 = new Map();
user2.set("name", "John").set("age", 30);
for (let [key, value] of user2) alert(`${key}:${value}`);

// 변수 값 교환
let guest = "Jane";
let admin = "Pete";
[guest, admin] = [admin, guest];
let [a, b, c] = [1, 2, 3];
[a, b, c] = [b, c, a];
```

### '...'로 나머지 요소 가져오기

배열 앞쪽에 위치한 값 몇개만 필요하고 그 이후 이어지는 나머지 값은 한데 모아 저장하는 경우

```javascript
let [fruit1, fruit2, ...rest] = ["apple", "banana", "coconut", "date"];
fruit1, fruit2, rest; // "apple", "banana", ["coconut", "date"]
```

### 기본값

기본값(default value) 설정

- 할당하고자 하는 변수의 개수가 분해하고자 하는 배열의 길이보다 큰 경우
  - 에러가 발생하지 않고 `undefined` 할당
- `=`을 이용해 할당할 값이 없을 때 기본으로 할당할 값 설정
- 복잡한 표현식이나 함수 호출도 가능

```javascript
let [a, b] = [];
a, b; // undefined, undefined

let [name = "Guest", surname = "Anonymous"] = ["Julius"];
name, surname; // "Julius", "Anonymous"

let [apple, banana = prompt("바나나")] = ["Apple"];
apple, banana; // Apple(배열에서 받아온 값), prompt에서 받아온 값
```

### 객체 분해하기

- 할당 연산자 우측에 분해하고자 하는 객체를, 좌측에 상응하는 객체의 프로퍼티 작성
- `{객체 프로퍼티: 목표 변수}`
  - 좌측 패턴에 콜론 `:`을 사용해 객체 프로퍼티를 프로퍼티 키와 다른 이름을 가진 변수에 저장 가능
- 프로퍼티가 없는 경우를 대비해 `=`을 사용해 기본값을 설정할 수 있음
  - 표현식이나 함수 호출을 기본값으로 할 수도 있음
- 콜론 `:`과 할당 연산자 `=`를 동시에 사용할 수 있음
- 프로퍼티가 많은 복잡한 객체에서 원하는 정보만 뽑아올 수도 있음

```javascript
let { var1, var2 } = { var1: ..., var2: ... }
```

```javascript
// 객체 분해
let options = { title: "Menu", width: 100, height: 200 };
let { width, height, title } = options; // 순서 상관 없음

// 다른 이름 지정
let options = { title: "Menu", width: 100, height: 200 };
let { width: w, height: h, title } = options;
w, h, title; // "Menu", 100, 200

// 기본값 설정
let options = { title: "Menu" };
let {
  width = prompt("width?"), // prompt 창에 입력한 값
  height = 200, // 200
  title = prompt("title?") // Menu (prompt 창 x)
} = options;

// 콜론과 할당 연산자 동시 사용
let options = { title: "Menu" };
let { width: w = 100, height: h = 200, title } = options;
w, h, title; // 100, 200, "Menu"

// 특정 프로퍼티만 추출
let options = { title: "Menu", width: 100, height: 200 };
let { title } = options;
alert(title); // Menu
```

### 나머지 패턴 '...'

분해하려는 객체의 프로퍼티 개수가 할당하려는 변수의 개수보다 많을 때 나머지를 할당

```javascript
let options = { title: "Menu", width: 100, height: 200 };
let { title, ...rest } = options;
title, rest; // "Menu", { width: 100, height: 200 }
```

### `let` 없이 사용하기

- `let`으로 새로운 변수를 선언하지 않고 기존에 있던 변수에 분해한 값을 할당할 수 있음
- 할당문을 괄호 `(...)`로 감싸야 함
- `{...}`를 코드 블록으로 인식하기 때문에 `(...)`로 감싸 표현식으로 해석하게 함

```javascript
let title, width, height;
({ title, width, height } = { title: "Menu", width: 200, height: 100 });
title, width, height; // "Menu", 200, 100

let x = 0;
({ num: x } = { name: "test", num: 10 });
x; // 10
```

### 중첩 구조 분해(nested destructuring)

객체나 배열이 다른 객체나 배열을 포함하는 경우, 좀 더 복잡한 패턴을 사용하면 추출할 수 있음

```javascript
let options = {
  size: { width: 100, height: 200 },
  items: ["Cake", "Donut"],
  extra: true
};
let {
  size: { width, height },
  items: [item1, item2],
  title = "Menu"
} = options;
title, width, height, item1, item2; // "Menu", 100, 200, "Cake", "Donut"
// size, items의 전용 변수는 없음. extra도 마찬가지
alert(size); // ReferenceError: size is not defined
alert(items); // ReferenceError: items is not defined
alert(extra); // ReferenceError: extra is not defined
```

### 똑똑한 함수 매개변수

함수의 매개변수가 많은 경우

- 이중 상당수가 선택적으로 쓰이면 넘겨주는 인수의 순서가 틀려 문제가 발생할 수 있음
- 대부분의 매개변수에 기본 값이 설정되어 있어 굳이 인수를 넘겨주지 않아도 되는 경우에 문제 발생
- 매개변수가 더 많아질수록 더 가독성 저하

```javascript
function showMenu(title = "Untitled", width = 200, height = 100, items = []) {}
showMenu("My Menu", undefined, undefined, ["Item1", "Item2"]);
```

구조 분해 사용

- 매개변수 모두를 객체에 모아 함수에 전달
- 함수가 전달받은 객체를 분해하여 변수에 할당하게 함
- 중첩 객체와 콜론 조합 가능
- 모든 인수에 기본 값을 할당하려면 빈 객체를 명시적으로 전달해야 함
  - 빈 객체를 `{}`를 인수 전체의 기본 값으로 만들어 에러를 방지할 수 있음

```javascript
function({ incomingProperty: varName = defaultValue }) {}
```

```javascript
let options = { title: "My menu", items: ["Item1", "Item2"] };
function showMenu({
  title = "Untitled",
  width: w = 100,
  height: h = 200,
  items: [item1, item2]
}) {
  alert(`${title} ${w} ${h}`); // My Menu 100 200
  alert(item1); // Item1
  alert(item2); // Item2
}
showMenu(options);

showMenu({}); // 모든 인수에 기본 값 할당
showMenu(); // TypeError: Cannot read properties of undefined (reading 'title')

function showMenu({ title = "Menu", width = 100, height = 200 } = {}) {
  alert(`${title} ${width} ${height}`);
}
showMenu(); // Menu 100 200
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
