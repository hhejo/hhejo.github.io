---
title: React 렌더링 동작
date: 2024-04-14 12:20:27 +0900
last_modified_at: 2024-04-14 12:20:27 +0900
categories: [React]
tags: [react]
---

ddd
dd
d
d
d
d
drafts에서 옮기기

React 렌더링 동작을 자세하게 알아보겠습니다.

## What is "Rendering"?

렌더링이란?

- React가 컴포넌트에 현재 props와 상태의 조합을 기반으로 UI의 해당 부분이 어떻게 생겼으면 좋겠는지 설명하도록 요청하는 프로세스

### Rendering Process Overview

렌더링 프로세스 개요

1. 렌더링 과정에서, React는 컴포넌트 트리의 루트에서 시작하여 아래쪽으로 반복하여 업데이트가 필요한 것으로 플래그가 지정된 모든 컴포넌트를 탐색
2. 플래그가 지정된 각 컴포넌트에 대해 React는 다음을 호출
   - 함수 컴포넌트의 경우 `FunctionComponent(props)`
   - 클래스 컴포넌트의 경우 `classComponentInstance.render()`
3. render pass의 다음 단계를 위해 렌더링 출력을 저장

컴포넌트의 렌더링 출력(render outupt)

- 일반적으로 JSX 구문으로 작성되며,
- JS가 컴파일되고 배포를 준비할 때 `React.createElement()` 호출로 변환됨

`createElement`

- UI의 의도된 구조를 설명하는 일반 JS 객체인 React 엘리먼트를 반환

```javascript
// JSX 문법
return <MyComponent a={42} b="testing">Text here</MyComponent>

// 이 호출로 변환됨
return React.createElement(MyComponent, { a: 42, b: "testing" }, "Text here")

// 이 요소 객체가 됨
{ type: MyComponent, props: { a: 42, b: "testing" }, children: ["Text here"] }

// 그리고 내부적으로 React는 실제 함수를 호출해 렌더링함
let elements = MyComponent({ ...props, children });
```

```javascript
// HTML 같은 "host components"의 경우
return <button onClick={() => {}}>Click Me</button>

//
React.createElement("button", { onClick }, "Click Me")

// 최종적으로
{ type: "button", props: { onClick }, children: ["Click Me"] }
```

조정(reconciliation)

1. 전체 컴포넌트 트리에서 렌더링 출력을 수집
2. React는 새로운 객체 트리를 diff하고,
   - 이 객체 트리(tree of objects)를 흔히 가상 돔(virtual DOM)이라 함
3. 실제 DOM을 현재 원하는 출력처럼 보이게 하기 위해 적용해야 하는 모든 변경 사항의 목록을 수집
4. 이러한 차이점 및 계산 프로세스를 조정(reconciliation)이라 함
5. 그런 다음 React는 계산된 모든 변경 사항을 하나의 동기식 시퀀스(synchronous sequence)로 DOM에 적용

> React 팀은 최근 몇 년 동안 가상 DOM이라는 용어를 경시해왔음

> 저는 "virtual DOM"이라는 용어를 사용하지 않았으면 좋겠습니다.
> 2013년에는 사람들이 React가 모든 렌더링에서 DOM 노드를 생성한다고 생각했기 때문에 이 용어가 의미가 있었습니다. 하지만 오늘날 사람들은 거의 그렇게 생각하지 않습니다.
> "virtual DOM"은 DOM 문제에 대한 해결 방법처럼 들립니다. 하지만 React는 그런 것이 아닙니다.
> React는 "value UI"입니다. 핵심 원칙은 UI가 문자열이나 배열처럼 값이라는 것입니다.
> 변수에 보관하고, 전달하고, 자바스크립트 제어 흐름을 사용하는 등의 작업을 할 수 있습니다.
> DOM에 변경 사항을 적용하지 않기 위해 다른 방식을 사용하는 것이 아니라 표현력이 핵심입니다.
> 예를 들어 `<Message recipientId={10} />`은 DOM이 아닙니다.
> 개념적으로는 지연 함수 호출을 나타냅니다. `Message.bind(null, { recipientId: 10 })`

### Render and Commit Phases

렌더링 및 커밋 단계

React 팀은 이 작업을 개념적으로 두 단계로 나눔

1. 렌더링 단계(render phase)에는 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업이 포함됨
2. 커밋 단계(commit phase)는 이러한 변경 사항을 DOM에 적용하는 프로세스

React가 커밋 단계에서 DOM을 업데이트한 후에는 요청된 DOM 노드와 컴포넌트 인스턴스를 가리키도록 모든 참조를 적절히 업데이트

그런 다음 `componentDidMount` 및 `componentDidUpdate` 클래스 lifecycle 메서드와 `useLayoutEffect` 훅을 동기적으로 실행

그런 다음 React는 짧은 timeout을 설정하고, 타임아웃이 만료되면 모든 `useEffect` 훅을 실행

이 단계를 "Passive Effects" 단계라고도 함

React 18은 `useTransition`과 같은 "동시 렌더링"(Concurrent Rendering) 기능을 추가

이를 통해 React는 브라우저가 이벤트를 처리할 수 있도록 렌더링 단계에서 작업을 일시중지할 수 있음

React는 나중에 해당 작업을 재개하거나, 버리거나, 적절하게 다시 계산

render pass가 완료되면 React는 여전히 커밋 단계를 한 단계로 동기화하여 실행함

여기서 이해해야 할 핵심 부분은 "rendering"은 "DOM 업데이트"와는 다르며, 컴포넌트는 결과적으로 눈에 보이는 변화 없이 렌더링될 수 있다는 것

React가 컴포넌트를 렌더링할 때

- 컴포넌트는 지난번과 동일한 렌더링 출력을 반환할 수 있으므로 변경할 필요가 없음
- 동시 렌더링에서 React는 여러 번 렌더링할 수 있지만 다른 업데이트가 현재 수행중인 작업을 무효화하면 매번 렌더링 출력을 버림

## How Does React Handle Renders?

## 참고

> [Blogged Answers: A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/)
