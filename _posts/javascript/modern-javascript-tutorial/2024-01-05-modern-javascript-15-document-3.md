---
title: 모던 JavaScript 튜토리얼 15 - 문서 3
date: 2024-01-05 15:07:55 +0900
last_modified_at: 2024-04-15 16:55:05 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, geometry, offset, sroll, browser, coordinate]
---

요소 사이즈와 스크롤, 브라우저 창 사이즈와 스크롤, 좌표

## 요소 사이즈와 스크롤

자바스크립트는 요소의 너비, 높이 같은 기하 정보 관련 프로퍼티 지원

### 샘플 요소

- `margin`: 요소 자체에 포함되지 않고 관련 특수 자바스크립트 프로퍼티도 없음
- 요소 내 텍스트가 길어 넘치면 브라우저가 이 텍스트들을 `padding-bottom`에 표시. 이는 정상적인 동작
- 스크롤 바: 몇몇 브라우저는 컨텐츠 영역 너비(content width) 일부를 빌려 스크롤 바를 위치시킴

### 기하 프로퍼티

- 기하 프로퍼티(geometry property): 값은 숫자이고 단위는 픽셀

### offsetParent와 offsetLeft, offsetTop

- offset: 요소가 화면에서 차지하는 영역 전체 크기를 나타냄
  - 요소의 너비, 높이, 패딩, 스크롤바, 테두리를 합친 크기. 마진 미포함
- `offsetParent`: 해당 요소를 렌더링할 때 좌표 계산에 사용되는 가장 가까운 조상 요소의 참조 반환. 아래 셋 중 하나
  - CSS `position` 프로퍼티가 `absolute`, `relative`, `fixed`, `sticky`인 가장 가까운 조상 요소
  - `<td>`, `<th>`, `<table>`
  - `<body>`
- `offsetParent`가 `null`이 되는 경우
  - 화면에 보이지 않는 요소 (CSS `display` 프로퍼티가 `none`, 문서 내에 있지 않은 요소)
  - `<body>`와 `<html>`
  - `position` 프로퍼티가 `fixed`
- `offsetLeft`, `offsetTop`: `offsetParent`를 기준으로 각각 요소가 오른쪽, 아래로 얼마나 떨어져 있는지 나타냄

### offsetWidth와 offsetHeight

- `offsetWidth`, `offsetHeight`: 각각 요소 가장 바깥 부분이 차지하는 너비와 높이 정보를 제공하는 프로퍼티
- 패딩, 테두리(border)를 포함한 요소 전체의 사이즈 정보 제공

화면에 표시되지 않는 요소

- 요소 혹은 이 요소의 조상 요소 중 어떤 것이든 CSS `display` 프로퍼티가 `none`
- 문서에 삽입하기 전의 요소이거나 문서 내 해당 요소가 없음
- 모든 기하 프로퍼티 값 `0`
- `offsetParent` 프로퍼티 값 `null`
- 기하 프로퍼티는 보이는 요소(displayed element)를 대상으로만 계산됨
- `function isHidden(el) { return !el.offsetWidth && !el.offsetHeight; }`로 요소의 숨김 상태 여부 파악
  - 요소가 화면에 있으나 사이즈가 `0`일 때도 `true` 반환 (비어있는 `<div>` 등)

### clientTop과 clientLeft

- `clientTop`, `clientLeft`: 테두리(border) 두께 측정 가능. 테두리는 요소 내에 있음
- 테두리 높이, 너비와 정확히 일치하지 않고 정확히는 테두리 바깥을 기준으로 한 테두리 안 상대 좌표를 나타냄
- 오른쪽에서 왼쪽으로 글이 전개되는 언어일 때 차이가 드러남
  - 아랍어, 히브리어는 스크롤바가 왼쪽에 나타나고 `clientLeft`에 스크롤바에 너비 포함

### clientWidth와 clientHeight

- `clientWidth`, `clientHeight`: 테두리 안 영역의 사이즈. 컨텐츠 너비, 패딩 포함. 스크롤바 너비는 제외
- 패딩 없는 `clientWidth`, `clientHeight`는 테두리와 스크롤바 안쪽의 컨텐츠 영역의 너비, 높이와 정확히 일치
- `contentWidth`: 컨텐츠 너비. 패딩 제외

### scrollWidth와 scrollHeight

- `scrollWidth`, `scrollHeight`: `clientWidth`, `clientHeight`와 유사하나 스크롤바에 의해 감춰진 영역도 포함
- 감춰진 영역이 있다면 훨씬 커짐
- 요소 크기를 컨텐츠가 차지하는 만큼 늘리고자 할 때 사용 가능

```javascript
element.style.height = `${element.scrollHeight}px`; // 컨텐츠가 차지하는 높이만큼 요소 높이를 늘림
```

### scrollLeft와 scrollTop

- `scrollLeft`, `scrollTop`: 가로 스크롤이 오른쪽, 세로 스크롤이 아래로 움직임에 따라 가려진 영역의 너비와 높이 나타냄
- `scrollTop`은 세로 스크롤바에 의해 가려져 보이지 않는 위쪽 컨텐츠의 높이가 됨
- `scrollLeft`와 `scrollTop`은 수정 가능
- 스크립트로 프로퍼티를 수정하면 자동으로 요소 내 스크롤이 움직임

```javascript
elem.scrollTop += 10; // 버튼에 추가하면 버튼 클릭 시 스크롤바가 10px씩 내려감
```

### CSS를 사용해 너비와 높이를 얻지 마세요

- `getComputedStyle` 대신 기하 프로퍼티로 너비와 높이 정보를 얻기
- CSS `width`와 `height`는 다른 CSS 프로퍼티의 영향을 받음
  - 요소의 너비와 높이 계산 방법을 지정하는 `box-sizing`이 대표적 예
- CSS `width`와 `height`는 `auto`일 수 있음
  - 인라인 요소(inline element) 등
  - 자바스크립트는 정확한 `px` 값이 있어야 계산을 하기 때문에 `auto`는 쓸모 없음
- 스크롤바가 있으면 의도한 대로 동작하지 않는 코드가 있는 경우
  - 스크롤바가 컨텐츠 영역을 차지하는 몇몇 브라우저에서 발생
  - 컨텐츠가 실제 차지하는 영역이 CSS로 설정한 너비보다 좁은데,
  - `clientWidth`와 `clientHeight`는 이를 고려해 클라이언트 요소가 차지하는 공간을 계산
  - 그런데 `getComputedStyle(elem).width`는 브라우저 간 차이 존재
    - Chrome은 스크롤바 너비를 제외한 진짜 내부 너비를 반환
    - Firefox는 스크롤바를 무시하고 CSS로 설정한 너비를 반환
- `getComputedStyle`과 기하 프로퍼티의 차이
  - 자바스크립트를 이용해 `getComputedStyle(...).width`로 값을 얻고자 할 때만 발생

```javascript
let elem = document.body;
alert(getComputedStyle(elem).width); // CSS가 적용된 elem의 너비
```

```html
<span id="elem">안녕하세요!</span>
<script>
  alert(getComputedStyle(elem).width); // auto
</script>
```

### 예제

What's the scroll from the bottom?

```javascript
let scrollBottom = elem.scrollHeight - elem.scrollTop - elem.clientHeight;
```

What is the scrollbar width?

- 표준 스크롤바 너비를 반환하는 코드를 작성
- 윈도우의 경우 `12px`에서 `20px`
- 브라우저가 이를 위한 공간을 예약하지 않으면 `0px`
  - 스크롤바가 텍스트 위에 반투명하게 표시되는 경우도 발생
- 스크롤이 있지만 테두리, 패딩 없는 요소 생성
  - 전체 너비 `offsetWidth`와 내부 컨텐츠 영역 너비 `clientWidth`의 차이가 정확히 스크롤바 너비가 됨

```javascript
let div = document.createElement("div");
div.style.overflowY = "scroll";
div.style.width = "50px";
div.style.height = "50px";
document.body.append(div); // document에 삽입하지 않으면 크기는 0
let scrollWidth = div.offsetWidth - div.clientWidth; // 전체 너비 - 내부 컨텐츠 영역 너비
div.remove();
alert(scrollWidth);
```

CSS `width`와 `clientWidth`의 차이

- `elem.clientWidth`: 숫자형 반환
  - 컨텐츠, 패딩 포함 영역의 너비(표준 `box-sizing`과 연관)
  - 브라우저 종류에 상관 없이 항상 동일하게 스크롤바 영역이 있는 경우에는 스크롤바 너비 제외
- `getComputedStyle(elem).width`: 끝에 `px`가 붙은 문자열 반환
  - 인라인 요소의 너비를 구할 떄 `"auto"`와 같은 숫자가 아닌 값을 반환할 수도 있음
  - CSS `width`는 패딩을 제외한 컨텐츠 영역의 너비
- 페이지에 스크롤바가 있고 브라우저가 스크롤바 영역을 따로 처리하는 경우
  - 스크롤바 영역은 컨텐츠를 담을 수 없음
  - 어떤 브라우저는 CSS `width`를 계산할 때 스크롤바 너비를 제외하고 어떤 브라우저는 스크롤바 영역을 포함

## 브라우저 창 사이즈와 스크롤

`document.documentElement`

- 루트 문서 요소. `<html>` 태그와 상응하는 요소로 다양한 메서드 지원
- 브라우저 창이 차지하는 너비, 높이
- 스크롤 때문에 보이지 않는 영역을 포함해 문서 전체가 차지하는 너비, 높이
- 자바스크립트로 페이지 스크롤하기

### 브라우저 창의 너비와 높이

- `document.documentElement.clientWidth`, `document.documentElement.clientHeight`: 창이 차지하는 너비와 높이
  - 스크롤바가 차지하는 공간을 제외하고 계산하기 때문에 사용 권장
  - 눈에 보이는 문서에서 컨텐츠가 실제로 들어가게 될 영역의 너비와 높이 반환
  - 창 사이즈가 필요한 경우는 스크롤바 안쪽에 무언가를 그리거나 위치시킬 때가 많음
- `window.innerWidth`, `window.innerHeight`: 스크롤바가 차지하는 영역을 포함해 계산

HTML에 항상 `DOCTYPE`을 명시

- 기하 관련 프로퍼티는 HTML에 문서에 `<!DOCTYPE HTML>`이 명시되어 있지 않으면 이상하게 동작할 수 있음

### 문서의 너비와 높이

- `document.documentElement`의 `scrollWidth`, `scrollHeight`로 문서 전체 크기 계산
- 전체 페이지를 대상으로 했을 때, `document.documentElement`의 프로퍼티들은 예상한 대로 동작하지 않음
- 정확한 문서 전체 높이 값을 얻으려면 여섯 프로퍼티가 반환하는 값 중 최댓값을 골라야 함
- 오래 전부터 있던 이상한 계산법

```javascript
let scrollHeight = Math.max(
  document.body.scrollHeight,
  document.documentElement.scrollHeight,
  document.body.offsetHeight,
  document.documentElement.offsetHeight,
  document.body.clientHeight,
  document.documentElement.clientHeight
);
alert("스크롤에 의해 가려진 부분을 포함한 전체 문서 높이: " + scrollHeight);
```

### 스크롤 정보 얻기

- `scrollLeft`, `scrollTop`: DOM 요소의 현재 스크롤 상태를 구하는 프로퍼티
  - 스크롤에 의해 가려진 영역에 대한 정보
  - 대부분의 브라우저에서 문서의 스크롤 상태를 구할 수 있음
- `window.pageXOffset`, `window.pageYOffset`: 브라우저 상관 없이 스크롤 정보를 구할 수 있음. 읽기만 가능

```javascript
alert("세로 스크롤에 의해 가려진 위쪽 영역 높이: " + window.pageYOffset); // 2305
alert("가로 스크롤에 의해 가려진 왼쪽 영역 너비: " + window.pageXOffset); // 0
```

### scrollTo, scrollBy로 스크롤 상태 변경하기

- `scrollTop`, `scrollLeft`: 일반 요소의 스크롤 상태 변경
- `document.documentElement`의 `scrollTop`, `scrollLeft`: 페이지 전체의 스크롤 상태 변경
  - Safari는 `document.body`의 `scrollTop`, `scrollLeft` 사용
- `window.scrollBy(x, y)`, `window.scrollTo(pageX, pageY)`: 브라우저 종류에 상관 없이 동일한 동작을 보장
- `scrollBy(x, y)`: 페이지의 스크롤 상태를 현재 포지션을 기준으로 상대적으로 조정
- `scrollTo(pageX, pageY)`: 절대 좌표를 기준으로 페이지 스크롤 상태 변경
  - `(pageX, pageY)`: 눈에 보이는 영역의 왼쪽 위 모서리의 좌표가 문서 전체의 왼쪽 위 모서리를 기준으로 한 좌표
  - `scrollLeft`, `scrollTop`을 변경한 것처럼 움직임
- 자바스크립트로 스크롤을 움직이려면 DOM이 완전히 만들어진 상태여야 함
- `<head>`에 있는 스크립트에서 페이지 전체의 스크롤을 움직이려 하면 잘 동작하지 않을 수 있음

```javascript
window.scrollBy(0, 10); // 스크롤 10px 내림
window.scrollTo(0, 0); // 문서 스크롤 상태를 처음으로 되돌리기
```

### scrollIntoView

- `elem.scrollIntoView(top)`: 전체 페이지 스크롤이 움직여 `elem`이 눈에 보이는 상태로 변경됨
- `top`이 `true`(디폴트): `elem`이 창 제일 위에 보이도록 스크롤 상태가 변경됨
  - `elem`의 위쪽 모서리가 창의 위쪽 모서리와 일치
- `top`이 `false`: `elem`이 창 가장 아래에 보이도록 스크롤 상태가 변경됨
  - `elem`의 아래쪽 모서리가 창의 아래쪽 모서리와 일치

### 스크롤 막기

- `document.body.style.overflow = "hidden"`: 페이지의 스크롤바 위치 고정
- `document.body.style.overflow = ""`: 고정 해제
- `document.body` 요소 뿐만 아니라 다른 요소의 스크롤을 고정시킬 때도 사용 가능
- 스크롤바가 사라지는 단점
  - 스크롤바는 일정 공간을 차지. 스크롤바가 사라지면 해당 공간을 채우기 위해 컨텐츠가 갑자기 움직이는 현상 발생
  - 이렇게 되면 사용자 입장에서 이상해 보일 수 있음
- 컨텐츠 움직임 현상 해결 방법
  - 스크롤바를 고정시키기 전과 후의 `clientWidth` 값을 비교해 해당 증상을 보정해야 함
  - 스크롤바가 사라질 때는 `clientWidth` 값이 갑자기 커짐
  - 이때 스크롤바가 차지했던 영역만큼 `document.body`에 `padding`을 주면 됨
  - 컨텐츠 전체의 너비를 스크롤바가 사라지기 전과 같은 값으로 유지 가능
- 모바일 웹
  - 스크롤을 조금 내릴 경우, 전체화면 모드로 전환(도구 막대 사라짐)되면서 창 크기가 달라짐
  - 항상 실물 디바이스로 테스트하는 것이 좋음
  - 개발자 도구에서 테스트할 경우, 항상 전체화면이기 때문에 기하값 응용시 버그를 발견하기 힘들 수 있음

## 좌표

대부분의 자바스크립트 메서드는 다음 두 좌표 체계 중 하나를 이용

1. 창 기준: `clientX/clientY`
   - 창(window) 맨 위 왼쪽 모서리를 기준으로 좌표 계산
   - `position:fixed`와 유사
2. 문서 기준: `pageX/pageY`
   - 문서(document) 최상단(root)에서 문서 맨 위 왼쪽 모서리를 기준으로 좌표 계산
   - `position:absolute`를 사용하는 것과 비슷

스크롤

- 스크롤을 움직이기 전
  - 창의 맨 위 왼쪽 모서리가 문서의 맨 위 왼쪽 모서리와 정확히 일치
- 스크롤이 움직이는 경우
  - 문서가 이동하면 문서 기준 좌표는 변경되지 않음
  - 창 내 요소는 움직이기 때문에 창 기준 요소 좌표가 변경됨
- 문서가 스크롤 되었을 때
  - `pageY`: 문서 맨 위부터 계산됨. 스크롤 전, 후 값 동일
  - `clientY`: 해당 지점이 창 상단과 가까워짐. 창 기준 좌표가 변함

### getBoundingClientRect로 요소 좌표 얻기

- `elem.getBoundingClientRect()`
- `elem`을 감싸는 가장 작은 네모의 창 기준 좌표를 `DOMRect` 클래스의 객체 형태로 반환
- 좌표는 소수, 음수일 수 있음

`DOMRect`의 주요 프로퍼티

- `x`, `y`: 요소를 감싸는 네모의 창 기준 X, Y 좌표
- `width`, `height`: 요소를 감싸는 네모의 너비(음수 가능)
- `top`, `bottom`: 요소를 감싸는 네모의 위쪽 모서리, 아래쪽 모서리의 Y 좌표
- `left`, `right`: 요소를 감싸는 네모의 왼쪽 모서리, 아래쪽 모서리의 X 좌표
- `left = x`
- `top = y`
- `right = x + width`
- `bottom = y + height`
- `right`, `bottom` 좌표는 CSS `position` 프로퍼티와 다름

### elementFromPoint(x, y)

```javascript
let elem = document.elementFromPoint(x, y);
```

- 창 기준 좌표 `(x, y)`에서 가장 가까운 중첩 요소 반환
- 창 밖 좌표를 대상으로 호출하면 `null` 반환
  - 좌표 중 하나라도 음수이거나 창의 너비, 높이를 벗어나는 경우

### 요소를 창 내 특정 좌표에 고정하기

- 문서 기준 좌표와 `position:absolute`를 함께 사용해 부자연스러운 현상 개선

### 문서 기준 좌표

- 창이 아닌 문서 왼쪽 위 모서리부터 시작
- 창 기준 좌표는 `position:fixed`에 해당
- 문서 기준 좌표는 `position:absolute`와 비슷
- 두 좌표 체계를 연관시키는 수식
  - `pageY` = `clientY` + 문서에서 세로 방향 스크롤에 의해 밀려난 부분의 높이
  - `pageX` = `clientX` + 문서에서 가로 방향 스크롤에 의해 밀려난 부분의 높이

## 참고

> [문서](https://ko.javascript.info/document)
