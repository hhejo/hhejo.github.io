---
title: Reflow와 Repaint란? 그리고 성능
date: 2024-04-20 07:31:17 +0900
last_modified_at: 2024-04-20 07:31:17 +0900
categories: [Web]
tags: [web, browser, reflow, repaint]
---

리플로우와 리페인트에 대해 알아보고, 발생을 최소화해 성능을 향상시킬 수 있는 방법을 알아보겠습니다.

## Reflow와 Repaint

요소가 시각적으로 변경되었을 때, 변화를 계산하여 화면에 다시 그려주는 작업

- DOM이 시각적으로 변경됨
- `reflow`가 발생하여 렌더 트리가 재생성됨
- 생성된 렌더 트리를 기반으로 요소를 화면에 그리는 `repaint`가 발생

### Reflow

- 요소의 너비, 높이, 위치 등이 변경되어 렌더 트리를 재생성하는 작업

reflow는 비용이 큰 작업

- 특정 요소에서 reflow가 발생하면, 주변 요소(부모, 자식 형제)에도 영향을 주기 때문

reflow 발생 시점

- 브라우저(윈도우) 리사이징 (뷰포트 변화로 Global Layout에 영향)
- Javascript로 DOM 관련 메서드를 실행하거나, DOM의 속성에 접근할 때
  - 접근만 해도 reflow가 발생하는 DOM 속성이 있음
- DOM 요소의 기하학적 속성이 변경될 때 (width, height 등)
- 폰트의 변화 (height 계산에 영향을 주어 Global Layout에 영향)
- 스타일 추가, 체거 (레이아웃을 변경)
- 내용 변화 (입력창에 텍스트 입력 등)
- class, style attribute의 동적 변화

`reflow`는 flow 과정을 다시 하는 작업

- `flow` 작업은 곧 `layout`
- `reflow` 작업을 한다는 것은 요소의 위치, 크기 계산을 다시 한다는 것

### Repaint

- 변경된 요소를 화면에 그리는 작업

repaint 발생 시점

- reflow가 발생했을 때
- 요소의 스타일이 변경될 때 (색상, 배경색 등)
- `visibility` 속성이 변경될 때 (`hidden` <-> `visible`)

repaint는 비용이 reflow보다 적은 작업

- 요소의 스타일이 변경되었을 때 발생하기 때문

`repaint`는 paint 작업을 다시 하는 것

- `paint` 작업은 요소에 스타일을 적용하는 과정
- `repaint` 작업을 한다는 것은 요소의 스타일을 재적용한다는 것

### 브라우저 렌더링의 일부 과정을 다시 한다는 것

결국 reflow와 repaint가 자주 일어나지 않아야 성능이 하락하지 않음

## Reflow, Repaint 과정 눈으로 확인해보기

1. 개발자도구 -> 성능 -> 기록 버튼 클릭
2. 기록이 되는 동안 요소의 모양, 색상을 변경해보기
3. 기록을 중단하고 결과를 보면 reflow, repaint가 발생한 것을 확인 가능

다른 방법

1. 개발자도구 -> 렌더링(아래의 탭)
2. 페인트 플래시 클릭
3. 레이아웃 변경 지역 클릭

## Reflow, Repaint 줄이기

### 1. 애니메이션 요소의 위치를 absolute로 지정

- 요소의 위치가 변경되어 주변 요소가 영향을 받으면 reflow가 여러번 발생
- 애니메이션이 적용된 요소를 `position: absolute`로 설정해 주변 요소에 영향을 주지 않게 하기

### 2. display를 none으로 설정

요소의 여러 스타일이 수정되어야 하는 경우

- `display: none`으로 렌더 트리에서 제외
- 스타일을 변경
- `display: block`으로 렌더 트리에 추가

### 3. DOM 속성 변경 코드 모으기

Javascript로 DOM의 여러 속성을 변경할 때, 코드 순서에 따라 reflow 횟수 줄이기 가능

```javascript
// BAD: reflow가 3번 발생
const el1 = document.querySelector(".first");
el1.style.width = "10px";
const el2 = document.querySelector(".second");
el2.style.width = "10px";
const el3 = document.querySelector(".third");
el3.style.width = "10px";

// GOOD: reflow가 1번 발생
const el1 = document.querySelector(".first");
const el2 = document.querySelector(".second");
const el3 = document.querySelector(".third");
el1.style.width = "10px";
el2.style.width = "10px";
el3.style.width = "10px";
```

브라우저의 reflow 처리 방식

- 브라우저는 변경할 요소가 있을 때, 즉시 처리하지 않고 큐에 저장
- 일정한 시간이 지나거나, 큐에 변경 작업이 쌓였을 때 reflow 실행
- DOM의 수정을 모아두면 이 수정 코드가 큐에 쌓여있다가 한번에 처리되기 때문에 reflow 횟수를 줄일 수 있음

### 4. reflow 유발 함수의 호출을 제한

reflow를 발생시키는 함수나 속성을 매번 호출하지 않고 변수에 저장

```javascript
// BAD
for (let i = 0; i < 10; i++) {
  el.style.width = target.offsetWidth + 10; // 반복문 실행마다 매번 호출
}

// GOOD
const { offsetWidth } = target; // 한 번 호출해서 변수에 저장
for (let i = 0; i < 10; i++) {
  el.style.width = offsetWidth + 10;
}
```

### 5. CSS로 스타일을 한번에 변경

DOM의 여러 스타일을 변경해야 한다면, 스타일을 CSS 클래스로 정의해두고 한번에 변경

```javascript
// BAD
el.style.width = "10px";
el.style.height = "10px";
el.style.borderRadius = "5px";
el.style.backgroundColor = "red";
el.style.left = "20px";
```

```html
<!-- GOOD -->
<body>
  <script>
    el.className = "small";
  </script>
</body>
<style>
  .small {
    width: 10px;
    height: 10px;
    border-radius: 5px;
    background-color: red;
    left: 20px;
  }
</style>
```

### 6. 가상 DOM 사용

React, Vue 등이 알아서 관리해줌

- 가상 DOM은 생성한 DOM을 저장했다가, DOM이 변경되었다면 메모리에 저장했던 DOM과 현재 변경된 DOM을 비교해 변경된 부분만 실제 DOM에 반영
- 모든 DOM에 영향을 주지 않고 변경이 필요한 DOM만 바꿔주기 때문에 불필요한 재렌더링을 막음

더 많은 최적화 방법은 여기 [Reflow 원인과 마크업 최적화 Tip](https://lists.w3.org/Archives/Public/public-html-ig-ko/2011Sep/att-0031/Reflow_____________________________Tip.pdf)

## 참고

> [리플로우, 리페인트와 브라우저 렌더링 알아보기](https://mong-blog.tistory.com/entry/리플로우-리페인트와-브라우저-렌더링-알아보기)

> [Reflow, Repaint 와 7가지 렌더링 최적화방법](https://ekimnida.tistory.com/45)

> [Reflow, Repaint가 진짜 일어나고 있을까?](https://sypear.tistory.com/32)

> [Gecko Reflow Visualization - mozilla.org](https://www.youtube.com/watch?v=ZTnIxIA5KGw)

> [Reflow, Repaint을 알아보자!](https://velog.io/@young_pallete/Reflow-Repaint을-알아보자)

> [What forces layout / reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)

> [CSS Triggers List – What Kind of Changes You Can Make](https://csstriggers.com)

> [Understanding Reflow and Repaint in the browser](https://dev.to/gopal1996/understanding-reflow-and-repaint-in-the-browser-1jbg)

> [Reflow 원인과 마크업 최적화 Tip](https://lists.w3.org/Archives/Public/public-html-ig-ko/2011Sep/att-0031/Reflow_____________________________Tip.pdf)
