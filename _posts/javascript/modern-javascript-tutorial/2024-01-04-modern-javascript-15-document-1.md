---
title: 모던 JavaScript 튜토리얼 15 - 문서 1
date: 2024-01-04 18:41:59 +0900
last_modified_at: 2024-01-23 08:29:12 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    host,
    window,
    dom,
    bom,
    node,
    element,
    document,
    getelement,
    queryselector
  ]
---

브라우저 환경과 다양한 명세서, DOM 트리, DOM 탐색하기, getElement*, querySelector*로 요소 검색하기

## 브라우저 환경과 다양한 명세서

자바스크립트

- 본래 웹 브라우저에서 사용하려고 만든 언어
- 이후 다양한 사용처와 플랫폼을 지원하는 언어로 변모

호스트 환경(host environment)

- 호스트: 자바스크립트가 돌아가는 플랫폼
- 각 플랫폼은 해당 플랫폼에 특정되는 기능을 제공
  - 자바스크립트 명세서에서 이를 부르는 명칭
- 랭귀지 코어(ECMAScript)에 더해 플랫폼에 특정되는 객체, 함수 제공
  - 웹 브라우저는 웹 페이지를 제공하기 위한 수단을 제공하고 Node.js는 서버 사이드 기능 제공

window

- 호스트 환경이 웹 브라우저일 때 최상단의 루트 객체. 자바스크립트 코드의 전역 객체
- 브라우저 창(browser window)을 대변하고 제어할 수 있는 메서드 제공
  - DOM: document, ...
  - BOM: navigator, screen, location, frames, history, XMLHttpRequest, ...
  - JavaScript: Object, Array, Function, ...

### 문서 객체 모델(DOM)

문서 객체 모델(Document Object Model, DOM)

- 웹 페이지 내의 모든 컨텐츠를 수정 가능한 객체로 나타냄
- DOM은 브라우저만을 위한 모델이 아님. 브라우저가 아닌 곳에서도 DOM을 사용하는 경우가 있음
- `document`: 페이지의 기본 진입점 역할 객체로, 페이지 내 무엇이든 변경·생성 가능

CSS 객체 모델(CSS Object Model, CSSOM)

- CSS 규칙과 스타일시트를 객체로 나타냄
- 이 객체를 어떻게 읽고 쓸 수 있을지에 대한 설명을 담은 별도의 명세서
- 문서에 쓰이는 스타일 규칙을 수정할 때 DOM과 함께 사용

### 브라우저 객체 모델(BOM)

브라우저 객체 모델(Browser Object Model, BOM)

- 문서 이외의 모든 것을 제어하기 위해 브라우저(호스트 환경)가 제공하는 추가 객체. HTML 명세서의 일부

## DOM 트리

DOM

- DOM에 따르면 모든 HTML 태그는 객체
- 태그 내의 문자(text) 역시 객체
- 이런 모든 객체는 자바스크립트를 통해 접근 가능
- 중첩 태그(nested tag): 태그 하나가 감싸고 있는 자식 태그

### DOM 예제

```html
<!DOCTYPE html>
<html>
  <head>
    <title>사슴에 관하여</title>
  </head>
  <body>
    사슴에 관한 진실.
  </body>
</html>
```

```
▾ HTML
  ▾ HEAD
    #text ↵␣␣␣␣
    ▾ TITLE
      #text 사슴에 관하여
    #text ↵␣␣
  #text ↵␣␣
  ▾ BODY
    #text ↵␣␣사슴에 관한 진실.
```

- 트리에 있는 노드는 모두 객체
- `<html>`은 루트 노드이고 `<head>`, `<body>`는 루트 노드의 자식

요소 노드(element node), 요소(element)

- 태그는 요소 노드이고 트리 구조를 구성

텍스트 노드(text node)

- `#text`: 요소 내의 문자. 새 줄(new line), 공백(space)을 포함한 문자열만 담음
- 공백을 지우면 텍스트 노드도 사라짐. 자식 노드를 가질 수 없어 트리의 끝에서 leaf 노드가 됨
- 문자열 양 끝 공백과 공백만 있는 텍스트 노드는 개발자 도구에서 보이지 않아 화면을 덜 차지함
- 아래 두 예외를 제외하고, 문서 내 공백이 있다면 다른 문자와 마찬가지로 텍스트 노드가 됨
- 텍스트 노드 생성의 2가지 예외
  1. 역사적인 이유로 `<head>` 이전의 공백과 새 줄은 무시됨
  2. `</body>` 뒤에 뭔가를 넣어도 자동으로 `body` 안쪽으로 옮겨져 `</body>` 뒤에는 공백이 있을 수 없음
     - HTML 명세서에서 모든 컨텐츠는 `body` 안쪽에 있어야 함
- 공백 없는 텍스트 노드만으로 HTML 문서를 구성하면 아래처럼 됨

```
<!DOCTYPE HTML>
<html><head><title>사슴에 관하여</title></head><body>사슴에 관한 진실.</body></html>
```

```
▾ HTML
  ▾ HEAD
    ▾ TITLE
      #text 사슴에 관하여
  ▾ BODY
    #text 사슴에 관한 진실.
```

### 자동 교정

- 브라우저는 기형적인 HTML을 만나면 DOM 생성과정에서 HTML을 자동으로 교정
  - 문서에 있는 에러, 닫는 태그가 없는 에러 등
- 최상위 태그는 항상 `<html>`로, 없다면 자동으로 문서 최상위에 삽입. `<body>`도 동일한 방식 적용
- 테이블에는 언제나 `<tbody>` 존재
  - DOM 명세서에는 테이블에 반드시 `<tbody>`가 있어야 한다고 명시
  - HTML에서는 `<tbody>`를 생략할 수 있지만 브라우저가 자동으로 DOM에 `<tbody>` 생성

### 기타 노드 타입

주석 노드(comment node)

- 주석도 노드가 되고 DOM에 추가됨
- 화면에는 출력되지 않으나 HTML에 있다면 반드시 DOM 트리에 추가되어야 하는 규칙 때문

```html
<!-- comment -->
```

```
#comment comment
```

HTML 안의 모든 것은 DOM을 구성

- HTML 문서 최상단의 `<!DOCTYPE...>` 지시자도 DOM 노드가 되고 `<html>` 바로 위에 위치
- `document` 객체 또한 DOM 노드이고 주석도 마찬가지

노드 타입

- 총 12가지. 실무에서 다루는 노드는 주로 4가지
- `document` 노드: DOM의 진입점
- element 노드: HTML 태그에서 생성되며 DOM 트리를 구성하는 블록
- text 노드, comment 노드

## DOM 탐색하기

`document` 객체

- DOM에 수행하는 모든 연산은 `document` 객체에서 시작
- DOM에 접근하기 위한 진입점으로, 이를 통과하면 어떤 노드에도 접근 가능

```
                [ document ]
                      ^
                      |
        [ document.documentElement ] <HTML>
                      ^
                      |
              [ document.body ] (if inside body)
------------------------------------------------
                      ^
                      |
                [ parentNode ]
                      ^
                      |
previousSibling <- [ <DIV> ] -> nextSibling
                  childNodes
firstChild                         lastChild
```

- `document.documentElement`(=`<html>`): `document`를 제외하고 DOM 트리 꼭대기에 있는 문서 노드
- `document.body`(=`<body>`): `<body>` 요소에 해당하는 DOM 노드
- `document.head`(=`<head>`): `<head>` 접근 가능

`document.body`가 `null`인 경우

- 브라우저가 스크립트를 읽는 도중에 아직 읽지 않은 요소. 존재하지 않아 스크립트에서 접근 불가
- `document.body`를 읽기 전, `<head>` 안의 스크립트에서는 `document.body`에 접근 불가
- DOM에서 `null` 값은 존재하지 않거나, 해당하는 노드가 없음을 의미

```html
<html>
  <head>
    <script>
      alert("HEAD: " + document.body); // null. 아직 <body>에 해당하는 노드 생성 전
    </script>
  </head>
  <body>
    <script>
      alert("BODY: " + document.body); // HTMLBodyElement
    </script>
  </body>
</html>
```

### childNodes, firstChild, lastChild로 자식 노드 탐색하기

- 자식 노드(child node, children): 바로 아래의 자식 요소
- 후손 노드(decendants node): 중첩 관계에 있는 모든 요소. 자식 노드, 자식 노드의 자식 노드 등
- `childNodes` 컬렉션
  - 텍스트 노드를 포함한 모든 자식 노드가 담김
  - 반복 가능한 유사 배열 객체인 컬렉션. `for..of` 사용 가능
  - 배열 메서드는 사용할 수 없지만 `Array.from`을 적용하면 사용 가능
- `firstChild`, `lastChild`: 첫 번째, 마지막 자식 노드에 빠르게 접근할 수 있는 프로퍼티
- `elem.hasChildNodes()`: 자식 노드의 존재 여부 검사

DOM 컬렉션

- 읽는 것만 가능. DOM을 변경하려면 다른 메서드 사용
  - `childNodes[i] = ...`로 자식 노드 교체 불가
- 몇몇 예외사항을 제외하고 거의 모든 DOM 컬렉션은 DOM의 현재 상태를 반영
  - `elem.childNodes`를 참조하는 도중, DOM에 새로운 노드가 추가·삭제되면, 변경사항이 컬렉션에도 자동으로 반영
- 컬렉션에 `for..in`을 사용하면 거의 사용되지 않는 추가 프로퍼티까지 순회하기 때문에 `for..of` 사용 권장

```html
<html>
  <body>
    <div>시작</div>
    <ul>
      <li>항목</li>
    </ul>
    <div>끝</div>
    <script>
      for (let i = 0; i < document.body.childNodes.length; i++)
        alert(document.body.childNodes[i]); // Text, DIV, Text, UL, ... , SCRIPT
    </script>
    <!-- 스크립트 실행 시점에는 브라우저가 읽지 못하는 부분 -->
    ...추가 내용...
  </body>
</html>
```

```javascript
elem.childNodes[0] === elem.firstChild; // 첫 번째 자식 노드
elem.childNodes[elem.childNodes.length - 1] === elem.lastChild; // 마지막 자식 노드
```

### 형제와 부모 노드

- 형제 노드(sibling node): 같은 부모를 가진 노드
- `nextSibling`, `previousSibling`: 다음, 이전 형제 노드에 관한 정보를 나타내는 프로퍼티
- `parentNode`: 부모 노드에 대한 정보를 나타내는 프로퍼티

```javascript
alert(document.body.parentNode === document.documentElement); // true. <html>은 <body>의 부모 노드
alert(document.head.nextSibling); // HTMLBodyElement. <body>는 <head>의 다음 형제 노드
alert(document.body.previousSibling); // HTMLHeadElement. <head>는 <body>의 이전 형제 노드
```

### 요소 간 이동

DOM 요소 노드 탐색

```
                        [ document ]
                              ^
                              |
                [ document.documentElement ] <HTML>
                              ^
                              |
                      [ document.body ] (if inside body)
------------------------------------
                              ^
                              |
                      [ parentElement ]
                              ^
                              |
previousElementSibling <- [ <DIV> ] -> nextElementSibling
                           children
firstElementChild                         lastElementChild
```

- 실무에서는 텍스트 노드, 주석 노드는 거의 다루지 않고 요소 노드를 조작하는 작업이 대부분
- 위에서 배운 프로퍼티들은 모든 종류의 노드 참조. 아래 프로퍼티들은 요소 노드를 참조
- `children`: 해당 요소의 자식 노드 중 요소 노드
- `firstElementChild`, `lastElementChild`: 첫 번째 자식 요소 노드, 마지막 자식 요소 노드
- `previousElementSibling`, `nextElementSibling`: 형제 요소 노드
- `parentElement`: 부모 요소 노드
  - `parentNode`는 종류에 상관 없이 부모 노드 반환
  - 두 프로퍼티는 아래 상황을 제외하고 대개 같은 노드를 반환

```javascript
alert(document.documentElement.parentNode); // document (document.documentElement의 부모는 document)
alert(document.documentElement.parentElement); // null (document 노드는 요소 노드가 아님)

while ((elem = elem.parentElement)) alert(elem); // <html>까지 거슬러 올라가지만 document까지는 가지 않음
```

```html
<html>
  <body>
    <div>시작</div>
    <ul>
      <li>항목</li>
    </ul>
    <div>끝</div>
    <script>
      for (let elem of document.body.children) alert(elem); // DIV, UL, DIV, SCRIPT
    </script>
    ...
  </body>
</html>
```

|       종류        | 모든 노드에 적용 가능 | 요소 노드에만 적용 가능 |
| :---------------: | :-------------------: | :---------------------: |
|     부모 노드     |      parentNode       |      parentElement      |
|     자식 노드     |      childNodes       |        children         |
| 첫 번째 자식 노드 |      firstChild       |    firstElementChild    |
| 마지막 자식 노드  |       lastChild       |    lastElementChild     |
|  이전 형제 노드   |    previousSibling    | previousElementSibling  |
|  다음 형제 노드   |      nextSibling      |   nextElementSibling    |

### 테이블 탐색하기

테이블

- 일부 DOM 요소 노드는 편의를 위해 기본 프로퍼티 외에 추가적인 프로퍼티 지원
- `<table>` 요소
  - 기본 프로퍼티 외에 다음 프로퍼티 지원
  - `table.rows`: `<tr>` 요소를 담은 컬렉션을 참조
  - `table.caption`, `table.tHead`, `table.tFoot`: `<caption>`, `<tHead>`, `<tFoot>` 요소 참조
  - `table.tBodies`: `<tbody>` 요소를 담은 컬렉션 참조
    - 표준에 따르면 테이블 내에 여러 개의 `<tbody>`가 존재할 수 있고 최소 하나의 `<tbody>` 필요
    - HTML 문서에는 `<tbody>`가 없더라도 브라우저는 `<tbody>`를 DOM에 자동으로 추가
- `<thead>`, `<tfoot>`, `<tbody>` 요소
  - `tbody.rows`, `thead.rows`, `tfoot.rows`: `<tr>` 요소 컬렉션을 참조
- `<tr>` 요소
  - `tr.cells`: 주어진 `<tr>` 안의 모든 `<td>`, `<th>`를 담은 컬렉션을 반환
  - `tr.sectionRowIndex`
    - 주어진 `<tr>`이 `<thead>`, `<tbody>`, `<tfoot>` 안쪽에서 몇 번째 줄에 위치하는지를 나타내는 인덱스 반환
  - `tr.rowIndex`: `<table>` 내에서 해당 `<tr>`이 몇 번째 줄인지를 나타내는 숫자 반환
- `<td>`, `<th>` 요소
  - `td.cellIndex` ,`th.cellIndex`: `<td>`나 `<th>`가 속한 `<tr>`에서 해당 셀이 몇 번째인지를 나타내는 숫자 반환

### 예제

`document.body.lastElementChild.lastElementChild`

- 마지막 자식 요소의 마지막 자식 요소 가져오기

형제 노드에 관한 질문

- `elem.lastChild.nextSibling`은 항상 `null`
- `elem.children[0].previousSibling`은 항상 `null`이 아님
  - `elem.children[0]`은 요소 노드 중 첫 번째 자식 노드를 나타내기 때문
  - `previousSibling`은 텍스트 노드가 될 수도 있음
- 두 경우 모두 자식 노드가 없는 경우 에러 발생

## getElement*, querySelector*로 요소 검색하기

상대 위치를 이용하지 않으면서 웹 페이지 내에서 원하는 요소 노드에 접근하는 방법

### document.getElementById 혹은 id를 사용해 요소 검색하기

`document.getElementById(id)`

```javascript
let elem = document.getElementById("id");
```

- `document` 객체를 대상으로 해당 `id` 속성을 가진 요소 노드를 찾음
- 문서 노드가 아닌 다른 노드에는 호출 불가
- 문서 내 요소의 `id` 속성값은 중복되면 `id`를 이용해 검색하는 메서드의 동작이 예측 불가해짐
- 요소 `id` 속성값을 따서 자동으로 전역변수가 선언됨
  - 동일한 이름을 가진 변수가 선언되면 사용 불가
  - 하위 호환성을 위해 남긴 동작이기 때문에 요소 접근 시 `getElementById` 같은 메서드 권장

```html
<div id="elem">
  <div id="elem-content">Element</div>
</div>
<script>
  // let elem = document.getElementById("elem");
  elem.style.background = "red"; // id 속성값을 딴 전역변수
  window["elem-content"]; // id에 하이픈(-)이 있으면 대괄호로 접근
  let elem = 5; // elem은 더이상 <div id="elem">을 참조하지 않고 5가 됨
</script>
```

### querySelectorAll

- `elem.querySelectorAll(css)`: CSS 선택자에 대응하는 `elem`의 자식 요소 모두 반환
- `:hover`, `:active` 등의 가상 클래스(pseudo-class) 사용 가능

```html
<ul>
  <li>1-1</li>
  <li>1-2</li>
</ul>
<ul>
  <li>2-1</li>
  <li>2-2</li>
</ul>
<script>
  let elements = document.querySelectorAll("ul > li:last-child");
  for (let elem of elements) alert(elem.innerHTML); // "1-2", "2-2"
</script>
```

### querySelector

- `elem.querySelector(css)`: 주어진 CSS 선택자에 대응하는 요소 중 첫 번째 요소 반환
- 해당하는 요소를 찾으면 검색 중단. `elem.querySelectorAll(css)[0]`보다 빠르고 코드가 짧음

### matches

- `elem.matches(css)`: 요소 `elem`이 주어진 CSS 선택자와 일치하는지 판단. DOM을 검색하지 않음
- 요소가 담겨 있는 배열 등을 순회헤 원하는 요소만 걸러내고자 할 때 유용

### closest

- `elem.closest(css)`: 자기 자신을 포함해 CSS 선택자와 일치하는 가장 가까운 조상 요소를 찾음
- DOM 트리를 한 단계씩 거슬러 올라가면서 원하는 요소 찾고, 찾으면 검색을 중단하고 해당 요소 반환
- 조상 요소(ancestor node): DOM 트리에서 특정 요소의 상위에 있는 요소
  - 부모 요소, 부모 요소의 부모 요소 등

### getElementsBy\*

- `elem.getElementsByTagName(tag)`: 주어진 태그에 해당하는 요소를 찾아 컬렉션에 담아 반환
- `elem.getElementsByClassName(className)`: class 속성값을 기준으로 요소를 찾아 컬렉션에 담아 반환
- `document.getElementsByName(name)`: name 속성값을 기준으로 문서 전체 검색

### 살아 있는 컬렉션

- `getElementsBy`로 시작하는 모든 메서드는 살아있는 컬렉션 반환
  - 문서에 변경이 있을 때마다 컬렉션이 자동 갱신되어 최신 상태를 유지
- `querySelectorAll`은 정적인 컬렉션을 반환
  - 컬렉션이 한번 확정되면 늘어나지 않음

```html
<div>첫 번째 div</div>
<script>
  let divs = document.getElementsByTagName("div");
  let divs2 = document.querySelectorAll("div");
  alert(divs.length); // 1
  alert(divs2.length); // 1
</script>
<div>두 번째 div</div>
<script>
  alert(divs.length); // 2
  alert(divs2.length); // 1
</script>
```

### 요약

|         메서드         | 검색 기준  | 호출 대상이 요소가 될 수 있음 | 컬렉션 갱신 |
| :--------------------: | :--------: | :---------------------------: | :---------: |
|     querySelector      | CSS 선택자 |               O               |      X      |
|    querySelectorAll    | CSS 선택자 |               O               |      X      |
|     getElementById     |     id     |               X               |      X      |
|   getElementsByName    |    name    |               X               |      O      |
|  getElementsByTagName  |  tag, \*   |               O               |      O      |
| getElementsByClassName |   class    |               O               |      O      |

`elemA.contains(elemB)`

- `elemB`가 `elemA`에 속하거나 `elemA == elemB`이면 참 반환

## 참고

> [문서](https://ko.javascript.info/document)
