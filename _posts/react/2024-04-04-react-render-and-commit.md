---
title: React의 렌더링과 커밋
date: 2024-04-04 17:23:45 +0900
last_modified_at: 2024-04-04 17:23:45 +0900
categories: [React]
tags: [react, trigger, render, commit]
---

React가 컴포넌트를 업데이트하고 화면에 표시하는 단계에 대해 알아보겠습니다.

## React의 렌더링과 커밋

컴포넌트가 화면에 표시되기 전에 React에 의해 렌더링되어야 함

React는 렌더링 결과가 지난번과 같으면 DOM을 건드리지 않음

### React 앱의 화면 업데이트 단계 구성

UI를 요청하고 제공하는 프로세스는 세 단계로 구성

1. Trigger: 렌더링 트리거 (손님의 주문을 주방으로 전달)
2. Render: 컴포넌트 렌더링 (주방에서 주문 준비)
3. Commit: DOM에 커밋 (테이블 위에 주문 배치)

### 1단계: 렌더링 트리거

컴포넌트가 렌더링되는 데는 두 가지 이유가 있음

1. 컴포넌트의 초기 렌더링
2. 컴포넌트(또는 상위 항목 중 하나)의 상태가 업데이트됨

초기 렌더링

- 앱이 시작되면 초기 렌더링을 트리거해야 함
- 대상 DOM 노드로 `createRoot`를 호출한 다음 컴포넌트로 `render`를 호출하면 완료됨

```javascript
import App from "./App.js";
import { createRoot } from "react-dom/client";
const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

상태 업데이트 시 다시 렌더링

- 컴포넌트가 처음 렌더링되면 `set` 함수로 상태를 업데이트해 추가 렌더링을 트리거할 수 있음
- 컴포넌트의 상태를 업데이트하면 렌더링이 자동으로 대기열에 추가됨

### 2단계: React가 컴포넌트를 렌더링

렌더링을 트리거한 후 React는 컴포넌트를 호출해 화면에 표시할 내용을 파악

- 렌더링은 React가 컴포넌트를 호출하는 것
- 초기 렌더링 시 React는 루트 컴포넌트를 호출
- 후속 렌더링의 경우 React는 상태 업데이트가 렌더링을 트리거한 함수 컴포넌트를 호출
- 이 프로세스는 재귀적
- 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 해당 컴포넌트를 다음에 렌더링하고 해당 컴포넌트도 무언가를 반환하면 해당 컴포넌트를 다음에 렌더링하는 식
- 더 이상 중첩된 컴포넌트가 없고 React가 화면에 표시해야 할 내용을 정확히 알 때까지 프로세스가 계속됨

### 3단계: React는 DOM에 변경 사항을 커밋

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정

- 초기 렌더링의 경우 React는 `appendChild()` DOM API를 사용해 생성된 모든 DOM 노드를 화면에 배치
- 재렌더링의 경우 React는 DOM이 최신 렌더링 출력과 일치하도록 최소한의 필수 작업(렌더링 중에 계산됨)을 적용

React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경

### 에필로그: 브라우저 페인트

브라우저 렌더링

- 렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 화면을 다시 칠함
- 이 프로세스를 브라우저 렌더링이라 함
  - 문서 전체에서 혼동을 피하기 위해 페인팅이라 부르겠음

## 참고

> [Render and Commit](https://react.dev/learn/render-and-commit)

> [리액트 Virtual DOM과 diffing](https://joong-sunny.github.io/react/react2/)

> [React의 렌더 단계와 커밋 단계](https://www.moonkorea.dev/React-%EB%A0%8C%EB%8D%94%EB%8B%A8%EA%B3%84-%EC%BB%A4%EB%B0%8B%EB%8B%A8%EA%B3%84)

> [재조정](https://www.moonkorea.dev/React-%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%9E%AC%EC%A1%B0%EC%A0%95)

> [React가 고유한 key를 권장하는 이유](https://www.moonkorea.dev/React-%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%9E%AC%EC%A1%B0%EC%A0%95%EA%B3%BC-key)
