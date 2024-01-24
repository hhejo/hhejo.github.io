---
title: 모던 JavaScript 튜토리얼 17 - UI 이벤트
date: 2024-01-05 21:12:15 +0900
last_modified_at: 2024-01-24 07:17:48 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, mouse, drag-and-drop, pointer, keyboard, scroll]
---

마우스 이벤트, Moving the mouse: mouseover/out, mouseenter/leave, 드래그 앤 드롭과 마우스 이벤트, Pointer events, Keyboard: keydown and keyup, Scrolling

## 마우스 이벤트

마우스 이벤트는 핸드폰이나 태블릿 같은 다른 장치에서도 발생

### 마우스 이벤트 종류

- `mousedown`, `mouseup`
- `mouseover`, `mouseout`
- `mousemove`
- `click`
- `dbclick`
- `contextmenu`

### 마우스 이벤트 순서

- 사용자가 단 하나의 동작을 했어도 실행되는 이벤트는 여러 개일 수 있음
- 마우스 왼쪽 버튼 클릭: `mousedown`, `mouseup`, `click` 순서대로 이벤트 발생
- 마우스 오른쪽 버튼 클릭: `mousedown`, `contextmenu`, `mouseup` 순서대로 이벤트 발생
- 동작은 하나지만 여러 이벤트가 실행될 때 실행 순서는 내부 규칙에 따라 결정

### 마우스 버튼

- `button` 프로퍼티: 클릭과 연관된 이벤트가 정확히 어떤 버튼에서 이벤트가 발생했는지 알려줌
- `click`, `contextmenu`는 왼쪽, 오른쪽 버튼을 눌렀을 때 발생하기 때문에 `button`을 잘 사용하지 않음
- `mousedown`, `mouseup`은 마우스 버튼 어디서나 발생하기 때문에 해당 이벤트의 핸들러에 `event.button`을 명시해야 함
- `event.button` 프로퍼티 값
  - 왼쪽(주요 버튼): `0`
  - 가운데(보조 버튼): `1`
  - 오른쪽(두 번째 버튼): `2`
  - X1(뒤로 가기 버튼): `3`
  - X2(앞으로 가기 버튼): `4`
- `buttons`: 여러 개의 버튼을 한꺼번에 눌렀을 때 해당 버튼들에 대한 정보를 정수 형태로 저장

### shift, alt, ctrl, meta 프로퍼티

- 마우스 이벤트는 이벤트가 발생할 때 함께 누른 보조키가 무엇인지를 알려주는 프로퍼티를 지원
  - `shiftKey`, `ctrlKey`: Shift 키, Ctrl 키
  - `altKey`: Alt 키(MacOS에서 Opt 키)
  - `metaKey`: MacOs에서 Cmd 키
- 마우스 이벤트가 발생하는 도중에 해당 키가 함께 눌러졌다면, 프로퍼티 값은 `true`가 됨
- MacOS에서는 `Ctrl` 키와 마우스 왼쪽 버튼을 함께 누르면 마우스 오른쪽 버튼을 누른 것으로 해석하기 때문에 `Ctrl` 대신 `Cmd`를 사용
- `if (event.ctrlKey || event.metaKey)`: 모든 사용자가 OS에 상관 없이 동일한 경험을 하게 함

### clientX, clientY와 pageX, pageY

- 마우스 이벤트는 두 종류의 좌표 정보를 지원
- `clientX/clientY`: 웹 문서 왼쪽 위를 기준으로 하는 클라이언트 좌표. 페이지가 스크롤 되어도 안 변함
- `pageX/pageY`: 창 왼쪽 위를 기준으로 하는 페이지 좌표. 페이지가 스크롤 되면 변함

### mousedown 이벤트와 선택 막기

- 불필요하게 글자가 선택되는 것을 막기
- `mousedown` 이벤트가 발생할 때 나타나는 브라우저 기본 동작
- `return false`로 기본 동작을 막았어도 글자를 선택할 수 있는 방법은 여전히 있음
- 복사, 붙여넣기를 막으려면 `onCopy` 이벤트 활용

```html
...
<b ondblclick="alert('클릭!')" onmousedown="return false">
  여기를 더블클릭해주세요.
</b>
...
```

```html
<div
  oncopy="alert('불법 복제를 예방하기 복사 기능을 막아놓았습니다!');return false"
>
  Lorem ipsum dolor sit amet, consectetur adipiscing elit. Praesent convallis
  ultrices lacus ut dictum. Class aptent taciti sociosqu ad litora torquent per
  conubia nostra, per inceptos himenaeos.
</div>
```

## Moving the mouse: mouseover/out, mouseenter/leave

마우스가 요소 사이를 이동할 때 발생하는 이벤트

### Events mouseover/mouseout, relatedTarget

- `mouseover` 이벤트: 마우스 포인터가 요소 위에 올 때 발생. `relatedTarget` -> `target`
  - `event.target`: 마우스가 온 요소
  - `event.relatedTarget`: 마우스가 나온 요소
- `mouseout` 이벤트: 마우스 포인터가 요소에서 떠날 때 발생. `target` -> `relatedTarget`
  - `event.target`: 마우스가 나온 요소
  - `event.relatedTarget`: 포인터 아래의 새로운 요소
- `relatedTarget`: `mouseover`, `mouseout`에 있는 특별한 프로퍼티로 `target` 프로퍼티를 보완
  - 마우스가 한 요소에서 다른 요소로 이동하면, 그 중 하나는 `target`, 하나는 `relatedTarget`이 됨
  - `null`값은 마우스가 다른 요소가 아닌 창 밖에서 나왔거나 창 밖으로 떠났다는 것을 의미
  - `null`일 때 `event.relatedTarget.tagName`처럼 쓰면 에러 발생

### Skipping Elements

- 마우스 포인터는 중간 요소(또는 그 중 일부)를 건너뛸 수 있음
- 방문자가 마우스를 매우 빠르게 움직이면 일부 DOM 요소를 건너뛸 수 있음
- 포인터가 창 밖에서 페이지 중앙 안쪽으로 바로 점프할 수 있음
- `mouseover`가 트리거 되면 `mouseout`이 반드시 있음

### Mouseout when leaving for a child

- 브라우저 논리에 따르면, 마우스 커서는 언제든지 단일 요소 위에만 있을 수 있음
- 가장 많이 중첩된 요소이자 z-인덱스 기준으로 맨 위에 있는 요소
- `#parenet`에서 `#child`로 마우스 이동하면 `#parent`에서 두 가지 이벤트를 볼 수 있음
- target이 parent인 `mouseout`, target이 child인 `mouseover` (이벤트 버블링)

```html
<div id="parent">
  <div id="child">...</div>
</div>
```

### Events mouseenter and mouseleave

- `mouseenter`: 포인터가 요소에 들어갈 때 트리거
- `mouseleave`: 포인터가 요소를 떠날 때 트리거
- `mouseenter`, `mouseleave` 이벤트가 `mouseover`, `mouseout`과 다른 점
- 요소 내부의 하위 항목 간 전환은 계산되지 않고 이벤트가 버블링되지 않음

### Event delegation

## 드래그 앤 드롭과 마우스 이벤트

드래그(drag)와 드롭(drop)

- 사용자와 컴퓨터 간 상호작용을 도와주는 훌륭한 도구
- `dragstart`, `dragend` 등의 특수한 이벤트 존재

### 드래그 앤 드롭 알고리즘

1. `mousedown`에서는 움직임이 필요한 요소 준비
   - 이때 기존 요소의 복사본을 만들거나 해당 요소에 클래스를 추가하는 등 원하는 형태로 작업 가능
2. 이후 `mousemove`에서 `position:absolute`의 `left·top`을 변경
3. `mouseup`에서는 드래그 앤 드롭 완료와 관련된 모든 작업을 수행

### 올바른 위치 지정

### 잠재적 드롭 대상(드롭 가능)

## Pointer events

포인터 이벤트

- 마우스, 펜/스타일러스, 터치스크린과 같은 다양한 포인팅 장치의 입력을 처리하는 현대적인 방법

### The brief history

### Pointer event types

### Pointer event properties

### Multi-touch

### Event: pointercancel

### Pointer capturing

## Keyboard: keydown and keyup

### Teststand

### Keydown and keyup

- `keydown`, `keyup` 이벤트: 키가 눌렸을 때, 키를 놓을 때 발생
- `event.key`: `key` 프로퍼티로 문자를 얻음
  - 레이아웃에 따른 키 처리 시 유용
- `event.code` 프로퍼티: `code` 프로퍼티로 물리적 키 코드를 얻음
  - 언어 전환 후에도 단축키가 작동하려면 유용

`Z`를 `Shift`와 같이 누르거나 같이 누르지 않는 경우

|    키     | `event.key` |        `event.code`         |
| :-------: | :---------: | :-------------------------: |
|     Z     | `z`(소문자) |           `KeyZ`            |
| Shift + Z | `Z`(대문자) |           `KeyZ`            |
|    F1     |    `F1`     |            `F1`             |
| Backspace | `Backspace` |         `Backspace`         |
|   Shift   |   `Shift`   | `ShiftRight` or `ShiftLeft` |

기타 키 코드

- `Key<letter>`: 문자 키
- `Digit<number>`: 숫자 키
- 특수 키는 그들의 이름으로 코드됨
  - `Enter`, `Backspace`, `Tab` 등

### Auto-repeat

- 키를 충분히 오랫동안 누르고 있으면 '자동 반복'이 시작됨
- `keydown` 트리거는 계속 반복되고, 놓으면 마침내 `keyup`이 표시됨
- 자동 반복에 의해 트리거된 이벤트의 경우, 이벤트 객체의 `event.repeat` 속성이 `true`가 됨

### Default actions

- `keydown`의 기본 동작을 방지하면 OS 기반 특수 키를 제외한 대부분이 취소될 수 있음

### Legacy

- `keypress`, `keyCode`, `charCode`, `which`

## Scrolling

`scroll` 이벤트

- 페이지 또는 요소 스크롤에 반응
- `window`와 스크롤 가능한 요소 모두에서 작동
- 사용자가 문서에서 어디에 있는지에 따라 추가 컨트롤이나 정보 표시·숨기기 가능
- 사용자가 페이지 끝까지 아래로 스크롤하면 더 많은 데이터를 로드

```javascript
window.addEventListener("scroll", function () {
  document.getElementById("showScroll").innerHTML = window.pageYOffset + "px";
}); // 현재 스크롤 표시
```

### Prevent scrolling

- 스크롤 할 수 없는 항목 만들기
- 스크롤이 이미 발생한 후에 트리거되기 때문에 `onscroll` 리스너에서 `event.preventDefault()`로 스크롤 방지 불가
- 스크롤을 발생시키는 이벤트에 의해 `event.preventDefault()`로 스크롤 방지 가능
  - pageUp, pageDown의 `keydown`에 대한 이벤트 등
- 스크롤을 시작하는 방법은 다양하기 때문에 CSS `overflow` 프로퍼티를 사용하는 것이 안정적

### 예제

Endless page

스크롤의 두 가지 중요한 기능

- 스크롤은 탄력적(elastic)
  - 일부 브라우저/장치에서는 문서 시작 또는 끝 부분을 조금 넘어 스크롤할 수 있음
  - 아래에 빈 공간이 표시되면 문서가 자동으로 정상으로 '돌아옴(bounce back)'
- 스크롤은 부정확함(impresice)
  - 페이지 끝으로 스크롤하면 실제로 실제 문서 하단에서 0-50px 정도 떨어져 있을 수 있음
- 따라서 '끝까지 스크롤'한다는 것은 방문자가 문서 끝에서 100px 이상 떨어져 있지 않다는 것을 의미해야 함
- 실생활에서 우리는 더 많은 메시지나 더 많은 상품을 보여주고 싶을 수 있음

Up/down button

Load visible images

- 처음에 모든 이미지가 `placeholder.svg`
- 사용자가 이미지를 볼 수 있는 위치로 페이지가 스크롤 되면 `src`의 이미지에서 `data-src`의 이미지로 변경되고 로드

## 참고

> [UI 이벤트](https://ko.javascript.info/event-details)
