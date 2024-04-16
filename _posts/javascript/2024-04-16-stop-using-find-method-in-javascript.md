---
title: JavaScript의 find 메서드는 그만!
date: 2024-04-16 14:49:45 +0900
last_modified_at: 2024-04-16 14:49:45 +0900
categories: [JavaScript]
tags: [javascript, map]
---

JavaScript의 find() 메서드는 그만 사용하고, 대신에 Map을 사용하는 이유와 방법을 알아보겠습니다.

## find 메서드는 어떻게 작동하나요?

자바스크립트에서 `find()` 메서드는 배열에서 선형 검색을 수행

- 주어진 조건을 만족하는 첫 번째 요소를 반환
- 해당 요소를 찾을 때까지 모든 데이터를 읽음
- 찾고자 하는 항목이 배열의 마지막 요소인 것이 최악의 시나리오

따라서 시간 복잡도는 O(n)

- 배열의 요소의 개수가 많고, 반복하는 것이 많다면 성능이 저하될 수 있음

## 해시 맵이란 무엇이며 얼마나 빠른가요?

해시 맵은 기본적으로 키를 해당 값에 매핑할 수 있는 저장소

- 키를 제공하면 해당 키와 연관된 값을 반환
- 이러한 해시 테이블에서 동일한 키 값을 사용하여 삽입 및 삭제 작업을 수행
- 따라서 매우 빠른 스토리지 시스템임

이러한 모든 작업에는 O(1)의 시간 복잡도가 필요

## Array보다 Map을 사용하는 것이 나은 경우와 이유

프로젝트에서 종종 사용자 인터렉티브 컴포넌트에서 `find()` 메서드를 많이 사용

- 동적 경로(dynamic routes)에서 매개변수를 기반으로 데이터를 가져오기
- 특정 데이터를 조작하기
- 드롭다운 선택하기
- 입력 필드에 입력한 데이터를 검색하기 등등
- 때로는 필터링이 필요할 때도 있음

이러한 모든 작업은 선형적으로 작동하며, 시간 복잡도는 O(n)

## Array와 Map을 사용한 작업 비교

데이터 예시

```javascript
const data = [
  {
    age: 50,
    email: "atuny0@sohu.com",
    firstname: "Terry",
    lastname: "Medhurst",
    gender: "male",
    id: 1,
    username: "atuny0"
  },
  {
    age: 28,
    email: "hbingley1@plala.or.jp",
    firstname: "Sheldon",
    lastname: "Quigley",
    gender: "male",
    id: 2,
    username: "hbingley1"
  }
  // ... 1000개 이상의 많은 추가 요소들
];
```

배열로 작업하기

```javascript
// 요소 찾기
data.find((e) => e.username === "hbingley1");
// 요소 조작하기
const index = data.findIndex((e) => e.username === "hbingley1");
array[index] = newItem;
// 요소 삭제하기 (1번 방법)
const index1 = data.findIndex((e) => e.username === "hbingley1");
array.splice(index1, 1);
// 요소 삭제하기 (2번 방법)
const newArray = data.filter((e) => e.username !== "hbingley1");
// 요소 확인하기
data.includes((e) => e.username === "hbingley1");
```

맵으로 작업하기

```javascript
// Map으로 변환하기
const map = new Map(data.map((user) => [user.username, user]));
// 요소 찾기
map.get("hbingley1");
// 요소 조작하기
map.set("hbingley1", newItem);
// 요소 삭제하기
map.delete("hbingley1");
// 요소 확인하기
map.has("hbingley1");
```

## Map이란?

자바스크립트의 맵은 키를 값에 매핑하기 위한 데이터 구조를 제공

- Map은 모든 값을 키로 사용 가능
- 키는 고유해야 함(unique)

`find()`를 사용하는 대신, 데이터의 크기와 관계 없이 시간 복잡도가 O(1)인 `get()`을 사용해 맵 데이터 구조의 키와 연관된 값에 접근할 수 있음

시간 복잡도가 O(n)인 `includes()`를 사용하는 대신, 시간 복잡도가 O(1)인 `has()`로 데이터의 존재 여부를 확인할 수 있음

```javascript
let hashData = null;
const getUsers = async () => {
  try {
    const response = await fetch("https://dummyjson.com/users");
    const data = await response.json();
    hashData = new Map(data.users.map((user) => [user.username, user]));
  } catch (error) {
    console.error(error);
  }
};
```

## 참고

> [Stop Using find() Method in JavaScript](https://medium.com/@enestalayy/stop-using-find-method-in-javascript-dfdb40b10821)
