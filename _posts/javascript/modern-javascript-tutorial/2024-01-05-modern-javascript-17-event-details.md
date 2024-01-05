---
title: 모던 JavaScript 튜토리얼 17 - UI 이벤트
date: 2024-01-05 21:12:15 +0900
last_modified_at: 2024-01-05 21:12:15 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

마우스 이벤트, Moving the mouse: mouseover/out, mouseenter/leave, 드래그 앤 드롭과 마우스 이벤트, Pointer events, Keyboard: keydown and keyup, Scrolling

## 마우스 이벤트

마우스 이벤트는 핸드폰이나 태블릿 같은 다른 장치에서도 발생

### 마우스 이벤트 종류

`mousedown`, `mouseup`

`mouseover`, `mouseout`

`mousemove`

`click`

`dbclick`

`contextmenu`

### 마우스 이벤트 순서

사용자가 단 하나의 동작을 했어도 실행되는 이벤트는 여러 개일 수 있음

- 마우스 왼쪽 버튼 클릭: `mousedown`, `mouseup`, `click` 순서대로 이벤트 발생
- 동작은 하나지만 여러 이벤트가 실행될 때 실행 순서는 내부 규칙에 따라 결정

### 마우스 버튼

`button`

- 클릭과 연관된 이벤트는 정확히 어떤 버튼에서 이벤트가 발생했는지 알려주는 `button` 프로퍼티를 지원
- `click`, `contextmenu`는 왼쪽, 오른쪽 버튼을 눌렀을 때 발생하기 때문에 `button` 프로퍼티를 잘 사용하지 않음

`event.button` 프로퍼티 값

- 왼쪽(주요 버튼): `0`
- 가운데(보조 버튼): `1`
- 오른쪽(두 번째 버튼): `2`
- X1(뒤로 가기 버튼): `3`
- X2(앞으로 가기 버튼): `4`

`buttons`

- 여러 개의 버튼을 한꺼번에 눌렀을 때 해당 버튼들에 대한 정보를 정수 형태로 저장

### shift, alt, ctrl, meta 프로퍼티

마우스 이벤트는 이벤트가 발생할 때 함께 누른 보조키가 무엇인지를 알려주는 프로퍼티를 지원

보조키 별로 지원하는 프로퍼티들

- `shiftKey`: Shift 키
- `altKey`: Alt 키(MacOS에서 Opt 키)
- `ctrlKey`: Ctrl 키
- `metaKey`: MacOs에서 Cmd 키
- 마우스 이벤트가 발생하는 도중에 해당 키가 함께 눌러졌다면, 프로퍼티 값은 `true`가 됨

MacOS에서는 `Ctrl` 대신 `Cmd`를 사용할 것

### clientX, clientY와 pageX, pageY

마우스 이벤트는 두 종류의 좌표 정보를 지원

- 클라이언트 좌표: `clientX`, `clientY`
- 페이지 좌표: `pageX`, `pageY`

### mousedown 이벤트와 선택 막기

불필요하게 글자가 선택되는 것을 막기

- `mousedown` 이벤트가 발생할 때 나타나는 브라우저 기본 동작을 막는 방법
- 다른 방법도 존재
- 기본 동작을 막았음에도 불구하고 글자를 선택할 수 있는 방법은 여전히 있음

복사, 붙여넣기 막기

- `onCopy` 이벤트 활용

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

### Events mouseover/mouseout, relatedTarget

### Skipping Elements

### Mouseout when leaving for a child

### Events mouseenter and mouseleave

### Event delegation

## 드래그 앤 드롭과 마우스 이벤트

### 드래그 앤 드롭 알고리즘

### 올바른 위치 지정

### 잠재적 드롭 대상(드롭 가능)

## Pointer events

### The brief history

### Pointer event types

### Pointer event properties

### Multi-touch

### Event: pointercancel

### Pointer capturing

## Keyboard: keydown and keyup

### Teststand

### Keydown and keyup

### Auto-repeat

### Default actions

### Legacy

## Scrolling

### Prevent scrolling

## 참고

> [UI 이벤트](https://ko.javascript.info/event-details)
