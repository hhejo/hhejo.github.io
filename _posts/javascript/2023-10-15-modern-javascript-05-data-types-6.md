---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 6
date: 2023-10-15 10:20:13 +0900
last_modified_at: 2023-10-16 12:32:48 +0900
categories: [JavaScript]
tags: [javascript]
---

JSON

## JSON과 메서드

객체를 문자열로 전환해야 할 때

- 네트워크를 통해 객체를 전송
- 로깅 목적으로 객체를 출력
- 전환된 문자열에 원하는 객체 프로퍼티가 포함되어야 함

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

- 개발 과정에서 프로퍼티가 추가, 삭제, 수정될 수 있는데 매번 `toString`을 수정한다면 번거로움
- 반복문을 돌릴 수 있지만 중첩 객체 등으로 객체가 복잡한 경우는 까다로움

### JSON.stringify

JSON(JavaScript Object Notation)

- 값이나 객체를 나타내는 범용 포맷(RFC 4627 표준에 정의됨)
- 자바스크립트에서 사용할 목적으로 만들어진 포맷
- 라이브러리를 사용하면 자바스크립트가 아닌 언어에서도 JSON을 다룰 수 있어 JSON을 데이터 교환 목적으로 많이 사용

`JSON.stringify`

- 객체를 JSON으로 변환
- 원시값에도 적용 가능
  - 객체 `{...}`, 배열 `[...]`, 원시형(문자형, 숫자형, 불린형 값(`true`, `false`), `null`)

`JSON.parse`

- JSON을 객체로 변환

```javascript
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
/* JSON으로 인코딩된 객체:
{
  "name": "John",
  "age": 30,
  "isAdmin": false,
  "courses": ["html","css","js"],
  "wife": null
}
*/
```

변환된 문자열

- JSON으로 인코딩된(JSON-encoded) 객체
- 직렬화 처리된(serialized) 객체
- 문자열로 변환된(stringified) 객체
- 결집된(marshalled) 객체
- 객체는 이렇게 문자열로 변환된 후에야 네트워크를 통해 전송하거나 저장소에 저장할 수 있음

JSON으로 인코딩된 객체는 일반 객체와 다른 특징을 가짐

- 문자열은 큰따옴표로만 감싸야 함(작은따옴표, 백틱 불가)
- 객체 프로퍼티 이름은 큰따옴표로 감싸야 함

`JSON.stringify`는 원시값에도 적용 가능

```javascript
alert(JSON.stringify(1)); // 1
alert(JSON.stringify(`test`)); // "test"
alert(JSON.stringify(true)); // true
alert(JSON.stringify([1, 2, 3])); // [1,2,3]
```

`JSON.stringify`는 자바스크립트 특유의 객체 프로퍼티를 처리할 수 없음

- JSON은 데이터 교환을 목적으로 만들어진 언어에 종속되지 않는 포맷이기 때문
- `JSON.stringify` 호출 시 무시되는 프로퍼티
  - 함수 프로퍼티(메서드)
  - 심볼형 프로퍼티(키가 심볼인 프로퍼티)
  - 값이 `undefined`인 프로퍼티

```javascript
let user = {
  sayHi() { // 무시
    alert("Hello");
  }
  [Symbol('id')]: 123, // 무시
  something: undefined // 무시
};
alert(JSON.stringify(user)); // {}
```

중첩 객체도 알아서 문자열로 변환

```javascript
let meetup = {
  title: "Conference",
  room: { number: 23, participants: ["john", "ann"] }
};
alert(JSON.stringify(meetup));
/*
{
  "title":"Conference",
  "room":{"number":23,"participants":["john","ann"]},
}
*/
```

그러나 순환 참조가 있으면 원하는 대로 객체를 문자열로 바꿀 수 없음

```javascript
let room = { number: 23 };
let meetup = { title: "Conference", participants: ["john", "ann"] };
meetup.place = room; // meetup이 room을 참조
room.occupiedBy = meetup; // room이 meetup을 참조
JSON.stringify(meetup); // TypeError: Converting circular structure to JSON
```

### replacer로 원하는 프로퍼티만 직렬화하기

```javascript
let json = JSON.stringify(value[, replacer, space])
```

- `value`: 인코딩 하려는 값
- `replacer`: JSON으로 인코딩하길 원하는 프로퍼티가 담긴 배열. 또는 매핑 함수(`function(key, value)`)
- `space`: 서식 변경 목적으로 사용할 공백 문자 수

```javascript
let room = { number: 23 };
let meetup = {
  title: "Conference",
  participants: [{ name: "john" }, { name: "Alice" }],
  place: room
};
room.occupiedBy = meetup;
alert(JSON.stringify(meetup, ["title", "participants"]));
// 'name'을 넣지 않으면 가져오지 못함
// {"title":"Conference","participants":[{},{}]}
alert(
  JSON.stringify(meetup, ["title", "participants", "place", "name", "number"])
);
/*
{
  "title":"Conference",
  "participants":[{"name":"John"},{"name":"Alice"}],
  "place":{"number":23}
}
*/
```

`replacer` 함수는 프로퍼티(키, 값) 쌍 전체를 대상으로 호출되는데, 반드시 기존 프로퍼티 값을 대신하여 사용할 값을 반환해야 함

- 특정 프로퍼티를 직렬화에서 누락시키려면 반환 값을 `undefined`로 만들면 됨

```javascript
alert(
  JSON.stringify(meetup, function replacer(key, value) {
    alert(`${key}: ${value}`);
    return key === "occupiedBy" ? undefined : value;
  })
);
/* replacer 함수에서 처리하는 키:값 쌍 목록
:             [object Object]
title:        Conference
participants: [object Object],[object Object]
0:            [object Object]
name:         John
1:            [object Object]
name:         Alice
place:        [object Object]
number:       23
*/
```

- 첫 얼럿창의 `":[object Object]"` 문자열은 함수가 최초로 호출될 때 `{"": meetup}` 형태의 래퍼 객체가 만들어지기 때문
- `replacer` 함수가 가장 처음으로 처리해야 하는 `(key, value)` 쌍에서 키는 빈 문자열(`""`), 값은 변환하고자 하는 객체(`meetup`)가 됨
- `replacer` 함수를 사용해 중첩 객체 등을 포함한 객체 전체에서 원하는 프로퍼티만 선택해 직렬화 가능

### space로 가독성 높이기

중간에 삽입해 줄 공백 문자 수

로깅이나 가독성을 높이는 목적으로 사용

```javascript
let user = {
  name: "John",
  age: 25,
  roles: {
    isAdmin: false,
    isEditor: true
  }
};
alert(JSON.stringify(user, null, 2));
/* 공백 문자 두 개를 사용해 들여쓰기
{
  "name": "John",
  "age": 25,
  "roles": {
    "isAdmin": false,
    "isEditor": true
  }
}
*/
alert(JSON.stringify(user, null, 4));
/* 공백 문자 네 개를 사용해 들여쓰기
{
    "name": "John",
    "age": 25,
    "roles": {
        "isAdmin": false,
        "isEditor": true
    }
}
*/
```

### 커스텀 'toJSON'

`toJSON`

- 객체에 `toJSON` 메서드가 구현되어 있으면 객체를 JSON으로 변환할 수 있음
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
/*
{
  "title":"Conference",
  "date":"2017-01-01T00:00:00.000Z",  // Date 객체의 내장 메서드 toJSON이 호출되면서 date의 값이 문자열로 변경
  "room": {"number":23}
}
*/
```

```javascript
let room = {
  number: 23,
  toJSON() {
    return this.number;
  }
};
let meetup = { title: "Conference", room };
alert(JSON.stringify(room)); // 23
alert(JSON.stringify(meetup)); // {"title":"Conference","room":23}
```

### JSON.parse

JSON으로 인코딩된 객체를 다시 객체로 디코딩

```javascript
let value = JSON.parse(str, [reviver]);
```

- `str`: JSON 형식의 문자열
- `reviver`: 모든 `(key, value)` 쌍을 대상으로 호출되는 `function(key, value)` 형태의 함수로 값을 변경시킬 수 있음
- 중첩 객체에도 사용 가능

```javascript
let numbers = "[0, 1, 2, 3]";
numbers = JSON.parse(numbers); // [0, 1, 2, 3]
```

```javascript
let userData =
  '{ "name": "John", "age": 35, "isAdmin": false, "friends": [0,1,2,3] }';
let user = JSON.parse(userData);
```

### JSON을 만들 때 흔한 실수

```javascript
let json = `{
  name: "John",                     // 프로퍼티 이름을 큰따옴표로 감싸지 않았습니다.
  "surname": 'Smith',               // 프로퍼티 값은 큰따옴표로 감싸야 하는데, 작은따옴표로 감쌌습니다.
  'isAdmin': false                  // 프로퍼티 키는 큰따옴표로 감싸야 하는데, 작은따옴표로 감쌌습니다.
  "birthday": new Date(2000, 2, 3), // "new"를 사용할 수 없습니다. 순수한 값(bare value)만 사용할 수 있습니다.
  "friends": [0,1,2,3]              // 이 프로퍼티는 괜찮습니다.
}`;
```

JSON은 주석을 지원하지 않음. 주석을 추가하면 유효하지 않은 형식이 됨

### reviver 사용하기

서버로부터 문자열로 변환된 `meetup` 객체를 전송받았다고 가정

```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
// 역 직렬화(deserialize)로 자바스크립트 객체 생성
let meetup = JSON.parse(str);
alert(meetup.date.getDate()); // TypeError: meetup.date.getDate is not a function
```

- `meetup.date`의 값은 `Date` 객체가 아니고 문자열이기 때문에 에러 발생
- 문자열을 `Date`로 전환해줘야 한다는 것을 알려야 함
  - `reviver` 사용
  - 중첩 객체에도 적용 가능

```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
let meetup = JSON.parse(str, function (key, value) {
  if (key == "date") return new Date(value);
  return value;
});
alert(meetup.date.getDate()); // 30
```

```javascript
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

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
