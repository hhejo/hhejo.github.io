---
title: React의 Side Effect와 useEffect에 대해 알아보기
date: 2024-04-02 14:42:17 +0900
last_modified_at: 2024-04-07 15:06:32 +0900
categories: [React]
tags: [react, side-effect, pure-function, useeffect]
---

React, Side Effect, Pure Function, useEffect 이들의 관계

## React의 Side Effect와 useEffect에 대해 알아보기

### Side Effect와 Pure Function

`Side Effect`

- 부수 효과, 의도하지 않은 효과를 의미

```javascript
let foo = "Hello";
function bar() {
  foo = "world";
}
bar(); // bar 함수는 side effect를 발생시킴
```

`Pure Function`

- 순수 함수
- 오직 함수의 입력만이 함수의 출력 결과에 영향을 주는 함수
- 항상 같은 입력에 대해 출력은 항상 같아야 함
- fetch API로 네트워크 요청을 하는 경우, 이는 순수 함수가 아님
  - 서버에 AJAX 요청을 할 때마다 응답이 달라질 수 있기 때문

### React에서의 Side Effect

- React 컴포넌트가 렌더링 된 이후, 비동기적으로 처리되어야 하는 부수적인 효과들
- 네트워크 서버에 데이터 요청하기
- 브라우저 API 사용하기 (`window`, `document` 등)
- `setTimeout`, `setInterval` 등

### useEffect

```javascript
useEffect(setup, dependencies?);
useEffect(() => {}, []);
```

- `setup`: effect의 로직이 포함된 함수. 선택적으로 cleanup 함수를 반환
  - `async` 함수는 안 됨
- `dependencies`: `setup` 코드 내부에서 참조되는 모든 반응 값들의 배열
  - 빈 배열은 컴포넌트가 렌더링 되었을 때 단 한 번만 실행
  - 배열을 아예 주지 않는다면 매 렌더링마다 실행
  - `useEffect`에서 `state`를 업데이트 한다면, React가 컴포넌트를 다시 렌더링하기 때문에 무한 루프에 빠짐
- `useEffect`는 cleanup 함수를 반환해 컴포넌트가 언마운트될 때 정리함

`useEffect` 예시

```javascript
import { useState, useEffect } from "react";

const App = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    fetch(URL)
      .then((res) => setCount(res.data))
      .catch(console.error);
  }, []);

  return <>{count}</>;
};
```

cleanup 함수로 반환하지 않으면 문제가 생길 수 있음

```javascript
useEffect(() => {
  setInterval(() => setCount(), count); // cleanup 함수가 없다면 interval은 계속 남아 메모리 누수
}, []);
```

cleanup 함수 반환하기

```javascript
useEffect(() => {
  const interval = setInterval(() => setCount(), count);
  return () => {
    clearInterval(interval);
  }; // cleanup 함수로 정리
}, []);
```

## 참고

> [useEffect](https://react.dev/reference/react/useEffect)

> [그래서 useEffect는 언제 쓰는건데요?](https://velog.io/@sucream/%EA%B7%B8%EB%9E%98%EC%84%9C-useEffect%EB%8A%94-%EC%96%B8%EC%A0%9C-%EC%93%B0%EB%8A%94%EA%B1%B4%EB%8D%B0%EC%9A%94)

> [Side Effect (부수 효과)와 Pure Function(순수 함수)](https://next-block.tistory.com/entry/Side-Effect-%EB%B6%80%EC%88%98%ED%9A%A8%EA%B3%BC)

> [React Side Effect란?](https://velog.io/@yes3427/React-Side-Effect)

> [React Side effect(사이드 이펙트)란? | 부수 효과, useEffect](https://choar816.tistory.com/163)
