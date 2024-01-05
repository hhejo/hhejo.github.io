---
title: 모던 JavaScript 튜토리얼 16 - 이벤트
date: 2024-01-05 16:33:33 +0900
last_modified_at: 2024-01-05 16:33:33 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

브라우저 이벤트 소개, 버블링과 캡처링, 이벤트 위임, 브라우저 기본 동작, 커스텀 이벤트 디스패치

## 브라우저 이벤트 소개

이벤트(event)

- 무언가 일어났다는 신호
- 모든 DOM 노드는 이런 신호를 만들어냄
- 이벤트는 DOM에만 한정되지는 않음

### 이벤트 핸들러

- 이벤트가 발생했을 때 실행되는 함수(handler)
- 여러 가지 방법으로 할당 가능

HTML 속성

- HTML 안의 `on<event>` 속성에 핸들러 할당 가능
- 브라우저가 속성값을 이용해 새로운 함수 생성
- 생성된 함수를 DOM 프로퍼티에 할당
- 코드가 길다면 함수를 만들어 호출하는 것을 추천
- 대소문자를 구분하지 않음
  - 속성값은 대개 `onclick` 같이 소문자로 작성

```html
<input value="클릭해 주세요." onclick="alert('클릭!')" type="button" />

<script>
  function countRabbits() {
    for (let i = 1; i <= 3; i++) alert(`토끼 ${i}마리`);
  }
</script>
<input type="button" onclick="countRabbits()" value="토끼를 세봅시다!" />
```

DOM 프로퍼티

- DOM 프로퍼티 `on<event>`를 사용해도 핸들러 할당 가능
- HTML 속성을 사용해 만든 핸들러와 동일하게 동작
- 대소문자 구분
  - `elem.onclick`은 괜찮으나 `elem.ONCLICK`은 불가

```html
<input id="elem" type="button" value="클릭해 주세요." />
<script>
  elem.onclick = function () {
    alert("감사합니다.");
  };
</script>
```

`onclick` 프로퍼티는 단 하나만 존재

- 복수의 이벤트 핸들러 할당 불가
- 하나 더 추가하면 기존 핸들러는 덮어써짐
- `elem.onclick = null` 같이 핸들러 제거 가능

### this로 요소에 접근하기

핸들러 내부에 쓰인 `this`

- 핸들러가 할당된 요소

### 자주 하는 실수

이미 존재하는 함수를 직접 핸들러에 할당하는 경우

- 괄호 없이 할당해야 함 (함수를 반환하지 않는다면)

```javascript
function sayThanks() {
  alert("감사합니다!");
}
elem.onclick = sayThanks; // sayThanks()가 아님
```

HTML 속성값에는 괄호가 있어야 함

- 브라우저는 속성값을 읽고, 속성값을 함수 본문으로 하는 핸들러 함수를 만들기 때문

```html
<input type="button" id="button" onclick="sayThanks()" />
```

```javascript
button.onclick = function () {
  sayThanks();
};
```

`setAttribute`로 핸들러를 할당하지 말 것

```javascript
document.body.setAttribute("onclick", function () {
  alert(1);
}); // 속성은 항상 문자열이기 때문에 함수가 문자열이 되어 <body>를 클릭하면 에러 발생
```

### addEventListener

HTML 속성, DOM 프로퍼티를 이용한 이벤트 핸들러 할당 방식의 공통 문제점

- 하나의 이벤트에 복수의 핸들러 할당 불가

```javascript
element.addEventListener(event, handler, [options]);
```

- `event`: 이벤트 이름
- `handler`: 핸들러 함수
- `options`: 아래 프로퍼티를 갖는 객체
  - `once`: `true`이면 이벤트가 트리거될 때 리스너가 자동으로 삭제됨
  - `capture`: 어느 단계에서 이벤트를 다뤄야 하는지 알려주는 프로퍼티
    - 호환성 유지를 위해 `options`를 객체가 아닌 `false/true`로 할당하는 것도 가능
    - 이는 `{capture: false/true}`와 동일
  - `passive`: `true`이면 리스너에서 지정한 함수가 `preventDefault()`를 호출하지 않음
- 여러번 호출하면 핸들러를 여러개 붙일 수 있음

```javascript
element.removeEventListener(event, handler, [options]);
```

- 핸들러 삭제
- 삭제는 동일한 함수만 가능
- 변수에 핸들러 함수를 저장해 놓지 않으면 핸들러 삭제 불가

```javascript
elem.addEventListener("click", () => alert("감사합니다!"));
// ....
elem.removeEventListener("click", () => alert("감사합니다!")); // 삭제 불가

function handler() {
  alert("감사합니다!");
}
input.addEventListener("click", handler);
// ....
input.removeEventListener("click", handler); // 삭제됨
```

```html
<input id="elem" type="button" value="클릭해 주세요." />
<script>
  function handler1() {
    alert("감사합니다!");
  }
  function handler2() {
    alert("다시 한번 감사합니다!");
  }
  elem.onclick = () => alert("안녕하세요."); // 안녕하세요.
  elem.addEventListener("click", handler1); // 감사합니다!
  elem.addEventListener("click", handler2); // 다시 한번 감사합니다!
</script>
```

DOM 프로퍼티에 할당할 수 없는 몇몇 이벤트가 있음

- `addEventListener`를 써야함

```javascript
document.onDOMContentLoaded = function () {
  alert("DOM이 완성되었습니다."); // 이 얼럿창은 절대 뜨지 않음
};

document.addEventListener("DOMContentLoaded", function () {
  alert("DOM이 완성되었습니다."); // 이 얼럿창은 제대로 뜸
});
```

### 이벤트 객체

이벤트 객체(event object)

- 이벤트가 발생하면 브라우저는 이벤트 객체를 생성
- 여기에 이벤트에 관한 상세한 정보를 넣음
- 핸들러에 인수 형태로 전달

이벤트 객체에서 지원하는 프로퍼티 중 일부

- `event.type`: 이벤트의 타입
- `event.currentTarget`: 이벤트를 처리하는 요소
  - 화살표 함수를 사용해 핸들러를 만드는 경우, 다른 곳에 바인딩 하지 않은 경우
    - `this`가 가리키는 값과 같게 됨
  - 화살표 함수를 사용했거나, 함수를 다른 곳에 바인딩한 경우
    - `event.currentTarget`을 사용해 이벤트가 처리되는 요소 정보를 얻을 수 있음
- `event.clientX/clientY`: 포인터 관련 이벤트에서 커서의 상대 좌표(모니터 기준이 아닌 브라우저 화면 기준)

```html
<input type="button" onclick="alert(event.type)" value="이벤트 타입" />
```

- HTML 핸들러 안에서도 이벤트 객체 접근 가능
- 브라우저는 속성을 읽고 `function (event) { alert(event.type); }` 같은 핸들러를 생성하기 때문

### 객체 형태의 핸들러와 handleEvent

`handleEvent`

- `addEventListener`는 함수 뿐만 아니라 객체를 이벤트 핸들러로 할당 가능
- 이벤트 발생 시 객체에 구현한 `obj.handleEvent(event)` 메서드 호출
  - 이벤트 타입을 정확히 명시해야 함
- 클래스도 사용 가능
- `handleEvent`가 모든 이벤트를 처리할 필요는 없음
  - 이벤트 관련 메서드를 `handleEvent`에서 호출해 사용할 수도 있음

```html
<button id="elem">클릭해 주세요.</button>
<script>
  let obj = {
    handleEvent(event) {
      alert(event.type + " 이벤트가 " + event.currentTarget + "에서 발생");
    }
  };
  elem.addEventListener("click", obj);

  class Menu {
    handleEvent(event) {
      switch (event.type) {
        case "mousedown":
          elem.innerHTML = "마우스 버튼을 눌렀습니다.";
          break;
        case "mouseup":
          elem.innerHTML += " 그리고 버튼을 뗐습니다.";
          break;
      }
    }
  }
  let menu = new Menu();
  elem.addEventListener("mousedown", menu);
  elem.addEventListener("mouseup", menu);
</script>
```

```html
<button id="elem">클릭해 주세요.</button>
<script>
  class Menu {
    handleEvent(event) {
      let method = "on" + event.type[0].toUpperCase() + event.type.slice(1); // mousedown -> onMousedown
      this[method](event);
    }
    onMousedown() {
      elem.innerHTML = "마우스 버튼을 눌렀습니다.";
    }
    onMouseup() {
      elem.innerHTML += " 그리고 버튼을 뗐습니다.";
    }
  }
  let menu = new Menu();
  elem.addEventListener("mousedown", menu);
  elem.addEventListener("mouseup", menu);
</script>
```

## 버블링과 캡처링

### 버블링

이벤트 버블링(event bubbling)

- 한 요소에 이벤트가 발생하면, 이 요소에 할당된 핸들러가 동작
- 이어서 부모 요소의 핸들러가 동작
- 가장 최상단의 조상 요소를 만날 때까지 이 과정이 반복되면서, 요소 각각에 할당된 핸들러가 동작
- 거의 모든 이벤트는 버블링됨

### event.target

부모 요소의 핸들러

- 이벤트가 정확히 어디서 발생했는지 등에 대한 자세한 정보를 얻을 수 있음

타깃 요소(target)

- 이벤트가 발생한 가장 안쪽의 요소
- `event.target`을 이용해 접근 가능

`event.target`과 `this`(=`event.currentTarget`)의 차이

- `event.target`: 실제 이벤트가 시작된 타깃 요소
  - 버블링이 진행되어도 변하지 않음
  - 예: 폼 안쪽 실제 클릭한 요소를 가리킴
- `this`: 현재 요소
  - 현재 실행 중인 핸들러가 할당된 요소를 참조
  - 예: 폼 요소의 핸들러가 동작했기 때문에 폼 요소를 가리킴
    - 폼 요소를 정확히 클릭했을 때는 `event.target`과 `this`가 같음

### 버블링 중단하기

이벤트 버블링

- 타깃 이벤트에서 시작
- `<html>` 요소를 거쳐 `document` 객체를 만날 때까지 각 노드에서 모두 발생
- 몇몇 이벤트는 `window` 객체까지 거슬러 올라감

`event.stopPropagation()`

- 이벤트를 완전히 처리하고 난 후 버블링 중단
- 한 요소의 특정 이벤트를 처리하는 핸들러가 여러개인 경우, 핸들러 중 하나가 버블링을 멈추더라도 나머지 핸들러는 여전히 동작
- 위쪽으로 일어나는 버블링은 막아주지만, 다른 핸들러들이 동작하는 것은 막지 못함

```html
<body onclick="alert(`버블링은 여기까지 도달하지 못합니다.`)">
  <button onclick="event.stopPropagation()">클릭해 주세요.</button>
</body>
```

`event.stopImmediatePropagation()`

- 버블링을 멈추고 요소에 할당된 다른 핸들러의 동작도 막음
- 요소에 할당된 특정 이벤트를 처리하는 핸들러 모두가 동작하지 않음

꼭 필요한 경우를 제외하고 버블링 막지 않기

- 이벤트 버블링을 막아야 하는 경우는 거의 없음

### 캡처링

표준 DOM 이벤트에서 정의한 이벤트 흐름

1. 캡처링 단계
   - 이벤트가 하위 요소로 전파되는 단계
2. 타깃 단계
   - 이벤트가 실제 타깃 요소에 전달되는 단계
   - 타깃 단계는 별도로 처리되지는 않고, 캡처링과 버블링 단계의 핸들러는 타깃 단계에서 트리거 됨
3. 버블링 단계
   - 이벤트가 상위 요소로 전파되는 단계

캡처링(capturing) 단계를 이용해야 하는 경우는 거의 없음

캡처링 단계에서 이벤트를 잡으려면 `addEventListener`의 `capture` 옵션을 `true`로 설정

```javascript
elem.addEventListener(..., {capture: true});
elem.addEventListener(..., true); // true만 써도 됨
```

`capture` 옵션

- `false`: 디폴트 값으로, 핸들러는 버블링 단계에서 동작
- `true`: 핸들러는 캡처링 단계에서 동작

`event.eventPhase`

- 현재 발생 중인 이벤트 흐름의 단계를 알 수 있는 프로퍼티
- 반환되는 정숫값에 따라 이벤트 흐름의 현재 실행 단계 파악
- 핸들러를 통해 흐름 단계를 알 수 있기 때문에 잘 쓰이지 않음

핸들러를 제거할 때, `removeEventListener`가 같은 단계에 있어야 함

- `addEventListener(..., true)`로 핸들러를 할당했다면
- `removeEventListener(..., true)`를 사용해 지워야 함

같은 요소와 같은 단계에 설정한 리스너는 설정한 순서대로 동작

```javascript
elem.addEventListener("click", (e) => alert(1)); // 첫 번째로 트리거
elem.addEventListener("click", (e) => alert(2));
```

## 이벤트 위임

이벤트 위임(event delegation)

- 캡처링과 버블링을 활용해 강력한 이벤트 핸들링 패턴인 이벤트 위임 구현 가능
- 비슷한 방식으로 여러 요소를 다뤄야 할 때 사용
- 요소마다 핸들러를 할당하지 않고, 요소의 공통 조상에 이벤트 핸들러를 단 하나만 할당해도 여러 요소를 한꺼번에 다룰 수 있음
- 공통 조상에 할당한 핸들러에서 `event.target`을 이용하면 실제 어디서 이벤트가 발생했는지 알 수 있음

### 이벤트 위임 활용하기

### '행동' 패턴

이벤트 위임은 요소에 선언적 방식으로 행동(behavior)을 추가할 때 사용할 수도 있음

- 이때는 특별한 속성과 클래스를 사용
- 행동 패턴은 두 부분으로 구성
  1. 요소의 행동을 설명하는 커스텀 속성을 요소에 추가
  2. 문서 전체를 감지하는 핸들러가 이벤트를 추적하게 함
     - 1에서 추가한 속성이 있는 요소에서 이벤트가 발생하면 작업을 수행

`document` 객체에 핸들러를 할당할 때 `document.onclick` 사용 금지

- 충돌을 일으킬 가능성 있음
- `addEventListener` 사용 권장

## 브라우저 기본 동작

상당수 이벤트는 발생 즉시 브라우저에 의해 특정 동작을 자동으로 수행

- 링크 클릭 시 해당 URL로 이동
- 폼 전송 버튼 클릭 시 서버에 폼 전송
- 마우스 버튼을 누른 채로 글자 위에서 커서를 움직이면 글자 선택

### 브라우저 기본 동작 막기

브라우저 기본 동작을 취소할 수 있는 두 가지 방법

- `event` 객체 사용하기
  - `event` 객체에 구현된 `event.preventDefault()` 메서드 사용
- 핸들러가 `addEventListener`가 아닌 `on<event>`를 사용해 할당된 경우, `false`를 반환하게 해 기본 동작 막기
  - 핸들러에서 `false`를 반환하는 것은 예외 상황
  - 이벤트 핸들러에서 반환된 값은 대개 무시되는데, `on<event>`를 사용해 할당한 핸들러에서 `false`를 반환하는 것은 단 하나의 예외 상황
  - 이외의 값들은 반환돼도 무시됨

```html
<a href="/" onclick="return false">이동하지 않음 1</a>
<a href="/" onclick="event.preventDefault()">이동하지 않음 2</a>
```

`<button>` 태그가 아닌 `<a>` 태그를 쓰는 이유

- 많은 사람들이 마우스 오른쪽 버튼 클릭 후 새 창에서 열기를 클릭해 링크를 열기 때문
- `<button>`이나 `<span>`을 쓰면 이 기능을 사용할 수 없음
- 검색 엔진은 인덱싱(색인)을 하는 동안 `<a href="...">` 링크를 따라감

후속 이벤트

- 어떤 이벤트들은 순차적으로 발생
- 이런 이벤트들은 첫 번째 이벤트를 막으면 두 번째 이벤트가 일어나지 않음

### addEventListener의 'passive' 옵션

`addEventListener`의 `passive: true` 옵션

- 브라우저에게 `preventDefault()`를 호출하지 않겠다로 알리는 역할
- 필요한 이유
- 모바일 기기에는 사용자가 스크린에 손가락을 대고 움직일 때 발생하는 `touchmove`와 같은 이벤트 존재
- 이런 이벤트는 기본적으로 스크롤링(scrolling)을 발생시킴
- 그런데 핸들러의 `preventDefault()`를 사용하면 스크롤링을 막을 수 있음
- 브라우저는 스크롤링을 발생시키는 이벤트를 감지했을 때 먼저 모든 핸들러를 처리
- 이때 `preventDefault`가 어디에서도 호출되지 않았다고 판단되면 그제서야 스크롤링을 진행
- 이 과정에서 불필요한 지연 발생
- 화면이 덜덜 떨리는 현상 발생
- `passive: true` 옵션은 핸들러가 스크롤링을 취소하지 않을 것이라는 정보를 브라우저에게 알려주는 역할
- 이 정보를 바탕으로 브라우저는 화면을 최대한 자연스럽게 스크롤링할 수 있게 하고 이벤트는 적절히 처리됨
- 몇몇 브라우저에서 `touchstart`와 `touchmove` 이벤트의 `passive`는 기본값이 `true`
  - Firefox, Chrome 등

### event.defaultPrevented

`event.defaultPrevented`

- 기본 동작을 막은 경우 값이 `true`
- 막지 않은 경우 값이 `false`
- `event.stopPropagation()` 대신 `event.defaultPrevented`를 사용해 이벤트가 적절히 처리되었다고 다른 이벤트에게 알릴 수도 있음

유스 케이스

```html
<p>문서 레벨 컨텍스트 메뉴(event.defaultPrevented를 확인함)</p>
<button id="elem">버튼 레벨 컨텍스트 메뉴</button>
<script>
  elem.oncontextmenu = function (event) {
    event.preventDefault();
    // event.stopPropagation(); // 의도한 대로 동작하나 대가가 너무 큼
    alert("버튼 컨텍스트 메뉴");
  };
  document.oncontextmenu = function (event) {
    if (event.defaultPrevented) return; // 이제 버튼 우클릭 해도 해당 이벤트만 발생
    event.preventDefault();
    alert("문서 컨텍스트 메뉴");
  };
</script>
```

`event.stopPropagation()`과 `event.preventDefault()`

- `event.stopPropagation()`과 `return false`로 알려진 `event.preventDefault()`는 명백히 다른 메서드
- 두 메서드는 연관성이 없음

## 커스텀 이벤트 디스패치

커스텀 이벤트(custom event)

- 자바스크립트로 이벤트를 직접 생성 가능
- 그래픽 컴포넌트(graphic component)를 만들 때 사용
- 새로운 커스텀 이벤트뿐만 아니라 내장 이벤트를 직접 만들 수도 있음
  - `click`, `mousedown` 등
  - 테스팅을 자동화할 때 유용

### Event의 생성자

내장 이벤트 클래스

- DOM 요소 클래스와 같이 계층 구조를 형성
- 내장 이벤트 클래스 계층의 꼭대기에는 `Event` 클래스가 있음

```javascript
let event = new Event(type[, options]);
```

- `Event` 객체 생성
- `type`: 이벤트 타입을 나타내는 문자열
  - 내장 이벤트, 커스텀 이벤트
- `options`: 두 개의 선택 프로퍼티가 있는 객체
  - `bubbles: true/false`: `true`인 경우 이벤트가 버블링 됨
  - `cancelable: true/false`: `true`인 경우 브라우저 기본 동작이 실행되지 않음
  - 두 프로퍼티는 기본적으로 `false`

### dispatchEvent

`elem.dispatchEvent(event)`

- 이벤트 객체를 생성한 다음에 `dispatchEvent`를 호출해 요소에 있는 이벤트를 반드시 '실행'시켜줘야 함
- '일을 처리하다' 라는 뜻의 dispatch
- 이렇게 이벤트를 실행시켜줘야 핸들러가 일반 브라우저 이벤트처럼 이벤트에 반응

```html
<button id="elem" onclick="alert('클릭!');">자동으로 클릭 되는 버튼</button>
<script>
  let event = new Event("click");
  elem.dispatchEvent(event); // 클릭!. 클릭하지 않아도 일단 실행됨
</script>
```

`event.isTrusted`

- 이벤트가 스크립트를 통해 생성한 이벤트인지 '진짜' 사용자가 만든 이벤트인지 알 수 있음
- `true`이면 사용자 액션을 통해 만든 이벤트라는 것을 의미
- `false`이면 해당 이벤트가 스크립트를 통해 생성되었다는 것

### 커스텀 이벤트 버블링 예시

커스텀 이벤트 사용 시 주의할 점

- 반드시 `addEventListener`를 이용해 핸들링
  - `on<event>`는 내장 이벤트에만 해당하는 문법
- `bubbles:true`를 명시적으로 할당하지 않으면 이벤트가 버블링되지 않음
- 내장 이벤트와 커스텀 이벤트의 버블링 메커니즘은 동일
- 커스텀 이벤트에도 캡쳐링, 버블링 단계 존재

### MouseEvent, KeyboardEvent 등의 다양한 이벤트

`new Event`로 만들면 안 되고 반드시 관련 내장 클래스를 사용해 만들어야 하는 이벤트들

- 제대로 된 생성자를 사용해야만 해당 이벤트 전용 표준 프로퍼티를 명시할 수 있음
- `UIEvent`, `FocusEvent`, `MouseEvent`, `WheelEvent`, `KeyboardEvent`, ...

### 커스텀 이벤트

`new CustomEvent`

- 제대로 된 커스텀 이벤트를 만들기 위해 사용
- `Event`와 거의 유사하지만 한 가지 차이점 존재
- `CustomEvent`의 두 번째 인수에는 객체가 들어갈 수 있음
- 개발자는 이 객체에 `detail`이라는 프로퍼티를 추가해 커스텀 이벤트 관련 정보를 명시하고, 정보를 이벤트에 전달 가능
- `detail` 프로퍼티에는 어떤 데이터도 들어갈 수 있음
- 다른 이벤트 프로퍼티와 충돌을 피하기 위함
- `new CustomEvent`를 사용하면 코드 자체만으로 커스텀 이벤트라고 설명해줌

```html
<h1 id="elem">환영합니다!</h1>
<script>
  elem.addEventListener("hello", function (event) {
    alert(event.detail.name);
  }); // 추가 정보는 이벤트와 함께 핸들러에 전달됨
  elem.dispatchEvent(new CustomEvent("hello", { detail: { name: "John" } }));
</script>
```

### event.preventDefault()

커스텀 이벤트의 기본 동작

- 커스텀 이벤트를 만들고 디스패칭 해주는 코드에 원하는 동작을 넣는 경우
- 커스텀 이벤트에도 기본 동작 설정 가능
- 이벤트 기본 동작이 취소되면 `elem.dispatchEvent(event)` 호출 시 `false` 반환
- 해당 이벤트를 디스패치하는 코드에서는 이를 통해 기본 동작이 취소되어야 한다는 것을 인지
- `new CustomEvent`의 `cancelable`이 `true`로 지정되지 않으면 `event.preventDefault()`가 무시됨

### 이벤트 안 이벤트

이벤트는 대개 큐에서 처리됨

- 이벤트는 순서대로 하나의 이벤트가 처리되면 다음 이벤트가 처리됨
- 그러나 이벤트 안 `dispatchEvent`처럼 이벤트 안에 다른 이벤트가 있는 경우
- 위 규칙이 적용되지 않음

이벤트 안 이벤트

- 이벤트 안에 있는 이벤트는 즉시 처리됨
- 새로운 이벤트 핸들러가 호출되고 난 후에 현재 이벤트 핸들링이 재개됨

```html
<button id="menu">메뉴(클릭해주세요)</button>
<script>
  menu.onclick = function () {
    alert(1);
    menu.dispatchEvent(new CustomEvent("menu-open", { bubbles: true })); // 중첩 이벤트 menu-open이 document에 할당된 핸들러에서 처리됨
    alert(2);
  };
  document.addEventListener("menu-open", () => alert("중첩 이벤트")); // 1과 2 사이에 트리거됨
</script>
```

- 중첩 이벤트가 `dispatchEvent`일 때뿐만 아니라 이벤트 핸들러 안에서 다른 이벤트를 트리거하는 메서드를 호출할 때 발생
- 즉, 이벤트 안 이벤트는 동기적으로 처리됨
- 중첩 이벤트가 동기적으로 처리되길 원치 않는 경우?
  - 맨 나중에 `dispatchEvent` 넣기
  - 중첩 이벤트를 지연시간이 0인 `setTimeout`으로 감싸기

## 참고

> [이벤트 기초](https://ko.javascript.info/events)
