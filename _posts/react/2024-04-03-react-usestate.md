---
title: React의 useState에 대해 자세히 알아보기
date: 2024-04-03 17:23:45 +0900
last_modified_at: 2024-04-03 17:23:45 +0900
categories: [React]
tags: [react, usestate, state, hook]
---

React, useState, state, hook

## React의 useState에 대해 자세히 알아보기

### 지역 변수(local variable)

- 일반적인 지역 변수는 렌더링 간에 유지되지 않음
- React가 컴포넌트를 다시 렌더링할 때, 처음부터 렌더링
- 지역 변수에 대한 변경 사항을 고려하지 않음
- 아래 예시에서 `count`는 변경되지 않음

```javascript
const App = () => {
  let count = 0;
  const handleClick = () => (count += 1);
  return (
    <>
      <div>count</div>
      <button onClick={handleClick}>Click</button>
    </>
  );
};
```

새 데이터로 컴포넌트를 업데이트하려면, 다음 두 가지 작업이 수행되어야 함

1. 렌더링 간에 데이터를 유지
2. React를 트리거해 세 데이터로 구성 요소를 렌더링(re-rendering)

### State

- 컴포넌트의 메모리
- 시간이 지남에 따라 변하는 데이터
- 모든 컴포넌트에는 state를 추가할 수 있고, 업데이트 가능
- 각 컴포넌트의 state는 독립적
- state는 snapshot처럼 동작
- 상태를 업데이트해도 다음 상태가 이전 상태와 같으면, React는 업데이트를 무시
  - `Object.is`를 통해 비교

### useState

```javascript
import { useState } from "react";

const [state, setState] = useState(initialState);
```

- `useState` 훅
- `state`: 렌더링 간에 데이터를 유지하는 상태 변수
- `setState`: 변수를 업데이트하고 React를 트리거해 컴포넌트를 다시 렌더링하는 state setter 함수
- `set` 함수를 호출해도 이미 실행중인 코드의 현재 상태는 변경되지 않음
  - 다음 렌더링부터 `useState`가 반환할 내용에만 영향을 미침

이전 상태를 기반으로 상태 업데이트

```javascript
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}

function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
```

상태의 객체 및 배열 업데이트

- React에서 state는 읽기 전용으로 간주되므로 기존 객체를 변경하기보다는 상태를 교체해야 함

```javascript
form.firstName = "Taylor"; // X

setForm({ ...form, firstName: "Taylor" }); // O
```

초기 상태 재생성 방지

- React는 초기 상태를 한 번 저장하고 다음 렌더링에서는 이를 무시

```javascript
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
}
```

- `createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만, 여전히 모든 렌더링에서 이 함수를 호출하고 있음

```javascript
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
}
```

- 함수 자체를 전달하면, React는 초기화 중에만 해당 함수를 호출
- 개발 모드에서 React는 initializer가 순수하다는 것을 확인하기 위해 두 번 호출할 수 있음

키를 사용하여 상태 재설정

- 컴포넌트에 다른 `key`를 전달해 컴포넌트의 상태를 재설정할 수 있음
- `key`가 변경하면 React는 컴포넌트(하위 자식 컴포넌트 모두)를 처음부터 다시 생성하므로 해당 상태가 재설정됨

### Hook

- Hook(훅)은 React가 렌더링되는 동안에만 사용 가능한 특별한 함수
- 이를 통해 다른 React 기능에 연결할 수 있음(hook into)
- 훅들의 이름은 `use`로 시작함
- 훅은 컴포넌트의 최상위 수준이나, 자체 훅에서만 호출 가능

### React의 특징

React는 상태 업데이트를 일괄 처리함

모든 이벤트 핸들러가 실행되고 `set` 함수가 호출된 후 화면을 업데이트

- 단일 이벤트 중에 여러 번 다시 렌더링되는 것을 방지

React는 Strict 모드에서 우발적인 불순물을 찾는 데 도움을 주기 위해, 업데이터 함수(updater function)를 두 번 호출

- 이는 개발 전용 동작이며 프로덕션에는 영향을 주지 않음
- 업데이터 함수가 순수하다면(되어야 하는 대로) 동작에 영향을 주지 않음
- 호출 중 하나의 결과는 무시될 것

## 참고

> [React 다시 한번 useState를 파헤쳐보자.](https://velog.io/@hjthgus777/%EB%8B%A4%EC%8B%9C-%ED%95%9C%EB%B2%88-useState-%EB%A5%BC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90)

> [useState](https://react.dev/reference/react/useState)

> [리액트 깊이알아보는 useState, 리렌더링핵심 작동원리](https://joong-sunny.github.io/react/react1/)

> [React State 특징 & 동작원리](https://www.datoybi.com/react-state/)

> [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

> [React useState 호출 시 무조건 렌더링 된다?](https://khj930410.tistory.com/213)
