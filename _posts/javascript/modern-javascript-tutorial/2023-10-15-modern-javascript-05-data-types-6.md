---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 6
date: 2023-10-15 10:20:13 +0900
last_modified_at: 2024-01-18 07:15:28 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, json]
---

JSON

## JSON과 메서드

객체를 문자열로 전환해야 할 때

- 네트워크를 통해 객체를 전송
- 로깅 목적으로 객체를 출력
- 전환된 문자열에 원하는 객체 프로퍼티가 포함되어야 함
- `toString` 사용 가능하지만 개발 과정에서 프로퍼티가 추가, 삭제, 수정되어 매번 `toString`을 수정한다면 번거로움
- 반복문을 돌릴 수 있지만 중첩 객체 등으로 객체가 복잡한 경우는 까다로움

```javascript
let user = {
  name: "John",
  age: 30,
  toString() {
    return `{name: "${this.name}"}, age: ${this.age}`;
  }
};
alert(user); // {name: "John", age: 30}
```

### JSON.stringify

JSON(JavaScript Object Notation)

- 값이나 객체를 나타내는 범용 포맷(RFC 4627 표준에 정의됨)
- 자바스크립트에서 사용할 목적으로 만들어진 포맷
- 라이브러리를 사용하면 자바스크립트가 아닌 언어에서도 JSON을 다룰 수 있어 데이터 교환 목적으로 많이 사용
- `JSON.parse`: JSON을 객체로 변환
- `JSON.stringify`: 객체를 JSON으로 변환
  - 객체, 배열, 원시형(문자형, 숫자형, 불린값(`true`, `false`), `null`)
  - `NaN`, `Infinity`, `-Infinity`는 `null`로 변환
  - 중첩 객체도 변환하지만 순환 참조가 있으면 변환 불가
  - 함수 프로퍼티(메서드), 심볼형 프로퍼티(키가 심볼인 프로퍼티), 값이 `undefined`인 프로퍼티 무시
  - JSON은 데이터 교환을 목적으로 만들어진 언어에 종속되지 않는 포맷이기 때문에 자바스크립트 특유의 객체 프로퍼티 처리 불가
- JSON으로 인코딩된 객체는 일반 객체와 다른 특징을 가짐
  - 문자열은 큰따옴표로만 감싸야 함 (작은따옴표, 백틱 불가)
  - 객체 프로퍼티 이름은 큰따옴표로 감싸야 함

```javascript
// JSON.stringify()
let student = {
  name: "John",
  age: 30,
  isAdmin: false,
  courses: ["html", "css", "js"],
  wife: null
};
let json = JSON.stringify(student);
alert(typeof json); // string
alert(json);
/* {
  "name": "John",
  "age": 30,
  "isAdmin": false,
  "courses": ["html","css","js"],
  "wife": null
} */

// 원시값에도 사용 가능
alert(JSON.stringify(1)); // 1
alert(JSON.stringify(`test`)); // "test"
alert(JSON.stringify(true)); // true
alert(JSON.stringify([1, 2, 3])); // [1,2,3]

// 메서드, 심볼, undefined 무시
let user = {
  sayHi() { // 무시
    alert("Hello");
  }
  [Symbol('id')]: 123, // 무시
  something: undefined // 무시
};
alert(JSON.stringify(user)); // {}

// NaN, Infinity, -Infinity는 null로 변환
JSON.stringify({ age: NaN }); // {"age":null}
JSON.stringify({ age: Infinity }); // {"age":null}
JSON.stringify({ age: -Infinity }); // {"age":null}

// 중첩 객체도 변환
let meetup = {
  title: "Conference",
  room: { number: 23, participants: ["john", "ann"] }
};
alert(JSON.stringify(meetup));
/* {
  "title":"Conference",
  "room":{"number":23,"participants":["john","ann"]},
} */

// 순환 참조 문제
let room = { number: 23 };
let meetup = { title: "Conference", participants: ["john", "ann"] };
meetup.place = room; // meetup이 room을 참조
room.occupiedBy = meetup; // room이 meetup을 참조
JSON.stringify(meetup); // TypeError: Converting circular structure to JSON
```

변환된 문자열

- JSON으로 인코딩된(JSON-encoded) 객체
- 직렬화 처리된(serialized) 객체
- 문자열로 변환된(stringified) 객체
- 결집된(marshalled) 객체
- 객체는 이렇게 문자열로 변환된 후에야 네트워크를 통해 전송하거나 저장소에 저장할 수 있음

### replacer로 원하는 프로퍼티만 직렬화하기

- `JSON.stringify(value[, replacer, space])`
  - `value`: 인코딩 하려는 값
  - `replacer`: JSON으로 인코딩할 프로퍼티의 배열 또는 매핑 함수
  - `space`: 서식 변경 목적으로 사용할 공백 문자 수
- `replacer` 함수: `function(key, value)`
  - 프로퍼티(키, 값) 쌍 전체를 대상으로 호출
  - 반드시 기존 프로퍼티 값을 대신해 사용할 값 반환
  - 반환 값이 `undefined`이면 해당 프로퍼티 누락

```javascript
let json = JSON.stringify(value[, replacer, space]);
```

```javascript
let room = { number: 23 };
let meetup = {
  title: "Conference",
  participants: [{ name: "john" }, { name: "Alice" }],
  place: room
};
room.occupiedBy = meetup;
alert(JSON.stringify(meetup, ["title", "participants"])); // {"title":"Conference","participants":[{},{}]}
alert(
  JSON.stringify(meetup, ["title", "participants", "place", "name", "number"])
);
/* {
  "title":"Conference",
  "participants":[{"name":"John"},{"name":"Alice"}],
  "place":{"number":23}
} */

function replacer(key, value) {
  alert(`${key}: ${value}`);
  return key === "occupiedBy" ? undefined : value; // occupiedBy 프로퍼티 제외
}
let result = JSON.stringify(meetup, replacer);
alert(result);
```

```
replacer 함수에서 처리하는 키:값 쌍 목록

:             [object Object]
title:        Conference
participants: [object Object],[object Object]
0:            [object Object]
name:         John
1:            [object Object]
name:         Alice
place:        [object Object]
number:       23

```

- 첫 얼럿창에서 `":[object Object]"` 문자열이 나타난 이유
- 함수가 최초로 호출될 때 `{"": meetup}` 형태의 래퍼 객체 생성
- 키는 빈 문자열 `""`, 값은 변환하고자 하는 객체 `meetup`이 되고 이를 변환해 `":[object Object]"`가 됨

### space로 가독성 높이기

`space`

- 중간에 삽입해 줄 공백 문자 수
- 로깅이나 가독성을 높이는 목적으로 사용

```javascript
let user = {
  name: "John",
  age: 25,
  roles: {
    isAdmin: false,
    isEditor: true
  }
};
alert(JSON.stringify(user, null, 2)); // 공백 문자 두 개를 사용해 들여쓰기
/* {
  "name": "John",
  "age": 25,
  "roles": {
    "isAdmin": false,
    "isEditor": true
  }
} */
alert(JSON.stringify(user, null, 4)); // 공백 문자 네 개를 사용해 들여쓰기
/* {
    "name": "John",
    "age": 25,
    "roles": {
        "isAdmin": false,
        "isEditor": true
    }
} */
```

### 커스텀 'toJSON'

`toJSON`

- 객체에 `toJSON` 메서드가 구현되어 있으면 객체를 JSON으로 변환 가능
- `JSON.stringify`가 이런 경우를 감지하고 `toJSON`을 자동으로 호출
- 중첩 객체에도 구현하여 사용 가능

```javascript
let room = { number: 23 };
let meetup = {
  title: "Conference",
  date: new Date(Date.UTC(2017, 0, 1)),
  room
};
alert(JSON.stringify(meetup));
/* {
  "title":"Conference",
  "date":"2017-01-01T00:00:00.000Z",  // Date 객체의 내장 메서드 toJSON이 호출되어 date의 값이 문자열로 변경
  "room": {"number":23}
} */

let room2 = {
  number: 23,
  toJSON() {
    return this.number;
  }
};
let meetup = { title: "Conference", room2 };
alert(JSON.stringify(room2)); // 23 (toJSON 호출됨)
alert(JSON.stringify(meetup)); // {"title":"Conference","room2":23}
```

### JSON.parse

`JSON.parse(str, [reviver])`

- JSON으로 인코딩된 객체를 다시 객체로 디코딩
- `str`: JSON 형식의 문자열
- `reviver`: 모든 키, 값 쌍을 대상으로 호출되는 `function(key, value)` 형태의 함수. 값 변경 가능
- 중첩 객체에도 사용 가능

```javascript
let value = JSON.parse(str, [reviver]);
```

```javascript
let numbers = "[0, 1, 2, 3]";
numbers = JSON.parse(numbers); // [0, 1, 2, 3] (실제 배열)
let userData =
  '{ "name": "John", "age": 35, "isAdmin": false, "friends": [0,1,2,3] }';
let user = JSON.parse(userData); // 객체
```

### JSON을 만들 때 흔한 실수

- JSON은 주석을 지원하지 않아 주석을 추가하면 유효하지 않은 형식이 됨

```javascript
let json = `{
  name: "John",                     // 프로퍼티 이름을 큰따옴표로 감싸야 함
  "surname": 'Smith',               // 프로퍼티 값은 큰따옴표로 감싸야 하는데, 작은따옴표로 감쌈
  'isAdmin': false                  // 프로퍼티 키는 큰따옴표로 감싸야 하는데, 작은따옴표로 감쌈
  "birthday": new Date(2000, 2, 3), // "new"를 사용할 수 없음. 순수한 값(bare value)만 사용할 수 있음
  "friends": [0,1,2,3]
}`;
```

### reviver 사용하기

```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
let meetup = JSON.parse(str); // 역 직렬화(deserialize)로 자바스크립트 객체 생성
alert(meetup.date.getDate()); // TypeError: meetup.date.getDate is not a function
```

- `meetup.date`의 값은 `Date` 객체가 아니고 문자열이기 때문에 에러 발생
- `reviver`로 문자열을 `Date`로 전환해줘야 한다는 것을 알려야 함
  - 중첩 객체에도 적용 가능

```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
let meetup = JSON.parse(str, function (key, value) {
  if (key == "date") return new Date(value);
  return value;
});
alert(meetup.date.getDate()); // 30

let schedule = `{
  "meetups": [
    {"title":"Conference","date":"2017-11-30T12:00:00.000Z"},
    {"title":"Birthday","date":"2017-04-18T12:00:00.000Z"}
  ]
}`;
schedule = JSON.parse(schedule, function (key, value) {
  if (key == "date") return new Date(value);
  return value;
});
alert(schedule.meetups[1].date.getDate()); // 18
```

### 역참조 배제하기

```javascript
let room = { number: 23 };
let meetup = {
  title: "Conference",
  occupiedBy: [{ name: "John" }, { name: "Alice" }],
  place: room
};
room.occupiedBy = meetup; // 순환 참조
meetup.self = meetup; // 순환 참조

const result = JSON.stringify(meetup, function replacer(key, value) {
  return key != "" && value == meetup ? undefined : value; // key != ""로 {"":meetup} 제외
});
alert(result);
/* {
  "title": "Conference",
  "occupiedBy": [ { "name": "John" }, { "name": "Alice" } ],
  "place": { "number": 23 }
} */
```

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
