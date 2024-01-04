---
title: 모던 JavaScript 튜토리얼 15 - 문서 1
date: 2024-01-04 18:41:59 +0900
last_modified_at: 2024-01-04 18:41:59 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

브라우저 환경과 다양한 명세서, DOM 트리, DOM 탐색하기, getElement*, querySelector*로 요소 검색하기

## 브라우저 환경과 다양한 명세서

자바스크립트

- 본래 웹 브라우저에서 사용하려고 만든 언어
- 이후 다양한 사용처와 플랫폼을 지원하는 언어로 변모

호스트

- 자바스크립트가 돌아가는 플랫폼

호스트 환경(host environment)

- 각 플랫폼은 해당 플랫폼에 특정되는 기능을 제공
- 자바스크립트 명세서에서 이를 부르는 명칭
- 랭귀지 코어(ECMAScript)에 더해 플랫폼에 특정되는 객체, 함수 제공
  - 웹 브라우저: 웹 페이지를 제공하기 위한 수단 제공
  - Node.js: 서버 사이드 기능 제공

호스트 환경이 웹 브라우저일 때 사용할 수 있는 기능 예시

- window
  - DOM: document, ...
  - BOM: navigator, screen, location, frames, history, XMLHttpRequest, ...
  - JavaScript: Object, Array, Function, ...

window

- 루트 객체
- 자바스크립트 코드의 전역 객체
- 브라우저 창(browser window)을 대변하고 제어할 수 있는 메서드 제공

### 문서 객체 모델(DOM)

문서 객체 모델(Document Object Model, DOM)

- 웹 페이지 내의 모든 컨텐츠를 객체로 나타냄
  - 수정 가능
- `document` 객체는 페이지의 기본 진입점 역할
- DOM은 브라우저만을 위한 모델이 아님

CSS 객체 모델(CSS Object Model, CSSOM)

- CSS 규칙과 스타일시트를 객체로 나타냄
- 이 객체를 어떻게 읽고 쓸 수 있을지에 대한 설명을 담은 별도의 명세서
- 문서에 쓰이는 스타일 규칙을 수정할 때 DOM과 함께 사용

### 브라우저 객체 모델(BOM)

브라우저 객체 모델(Browser Object Model, BOM)

- 문서 이외의 모든 것을 제어하기 위해 브라우저(호스트 환경)가 제공하는 추가 객체
- HTML 명세서의 일부

## DOM 트리

DOM에 따르면 모든 HTML 태그는 객체

중첩 태그(nested tag)

- 태그 하나가 감싸고 있는 자식 태그

태그 내의 문자(text) 역시 객체

이런 모든 객체는 자바스크립트를 통해 접근 가능

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
- `<html>`은 루트 노드
- `<head>`, `<body>`는 루트 노드의 자식

요소 노드(element node)

- 혹은 그냥 요소
- 태그는 요소 노드
- 트리 구조를 구성

텍스트 노드(text node)

- 요소 내의 문자
- `#text`
- 문자열만 담음
- 자식 노드를 가질 수 없음
- 트리의 끝에서 leaf 노드가 됨
- 새 줄(newline), 공백(space) 역시 텍스트 노드

텍스트 노드 생성의 예외

1. 역사적인 이유로 `<head>` 이전의 공백과 새 줄은 무시됨
2. HTML 명세서에서 모든 컨텐츠는 `body` 안쪽에 있어야 함
   - `</body>` 뒤에 무언가를 넣어도 자동으로 `body` 안쪽으로 옮겨져 `</body>` 뒤에는 공백이 있을 수 없음

- 두 예외를 제외하고, 문서 내 공백이 있다면 다른 문자와 마찬가지로 텍스트 노드가 됨
- 공백을 지우면 텍스트 노드도 사라짐

```
<!DOCTYPE HTML>
<html><head><title>사슴에 관하여</title></head><body>사슴에 관한 진실.</body></html>
```

- 공백 없는 텍스트 노드만으로 HTML 문서를 구성하려면 위처럼 작성해야 함

```
▾ HTML
  ▾ HEAD
    ▾ TITLE
      #text 사슴에 관하여
  ▾ BODY
    #text 사슴에 관한 진실.
```

문자열 양 끝 공백과 공백만 있는 텍스트 노드는 개발자 도구에서 보이지 않음

### 자동 교정

- 기형적인 HTML을 만나면 브라우저는 DOM 생성과정에서 HTML을 자동으로 교정
- 최상위 태그는 항상 `<html>`이어야 하는데 없으면 자동으로 문서 최상위에 넣어줌
- `<body>`도 동일한 방식 적용
- DOM 생성과정에서 브라우저는 문서에 있는 에러 등, 닫는 태그가 없는 에러 등을 자동으로 처리
- 테이블에는 언제나 `<tbody>` 존재
  - 브라우저가 자동으로 DOM에 `<tbody>` 생성

### 기타 노드 타입

주석 노드(comment node)

- 주석도 노드가 됨
- 화면에는 출력되지 않으나 HTML에 뭔가가 있다면 반드시 DOM 트리에 추가되어야 하는 규칙 때문에 DOM에 추가됨

```html
<!-- comment -->
```

```
#comment comment
```

HTML 안의 모든 것은 DOM을 구성

- 주석이더라도
- HTML 문서 최상단의 `<!DOCTYPE...>` 지시자 또한 DOM 노드가 됨
  - `<html>` 바로 위에 위치
- `document` 객체 또한 DOM 노드

노드 타입은 총 12가지

- 실무에서 다루는 노드는 주로 4가지
- `document` 노드
  - DOM의 진입점
- element 노드
  - HTML 태그에서 생성되며 DOM 트리를 구성하는 블록
- text 노드
- comment 노드

## DOM 탐색하기

`document` 객체

- DOM에 수행하는 모든 연산은 `document` 객체에서 시작
- DOM에 접근하기 위한 진입점
- 이를 통과하면 어떤 노드에도 접근 가능

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
[ parentNode ]
  ^
  |
previousSibling <- <DIV> -> nextSibling
                childNodes
firstChild                      lastChild
```

`document.documentElement`

- `document`를 제외하고 DOM 트리 꼭대기에 있는 문서 노드
- `<html>`

`document.body`

- `<body>` 요소에 해당하는 DOM 노드

`document.head`

- `<head>` 접근 가능

`document.body`가 `null`일 수도 있음

- 스크립트를 읽는 도중에 존재하지 않는 요소는 스크립트에서 접근 불가
- 브라우저가 아직 읽지 않았기 때문
- `document.body`를 읽지 않았으면, `<head>` 안의 스크립트에서는 `document.body`에 접근 불가
- DOM에서 `null` 값은 '존재하지 않음'이나 '해당하는 노드가 없음'을 의미

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

자식 노드(child node, children)

- 바로 아래의 자식 요소

후손 노드(decendants node)

- 중첩 관계에 있는 모든 요소
- 자식 노드, 자식 노드의 자식 노드 등

`childNodes` 컬렉션

- 텍스트 노드를 포함한 모든 자식 노드가 담김
- 배열이 아닌 반복 가능한 유사 배열 객체인 컬렉션
- `for..of` 사용 가능
- 배열이 아니기 때문에 배열 메서드 사용 불가
  - `Array.from`을 적용해 사용 가능

```html
<html>
  <body>
    <div>시작</div>
    <ul>
      <li>항목</li>
    </ul>
    <div>끝</div>
    <script>
      for (let i = 0; i < document.body.childNodes.length; i++) {
        alert(document.body.childNodes[i]); // Text, DIV, Text, UL, ... , SCRIPT
      }
    </script>
    ...추가 내용...
  </body>
</html>
```

- 스크립트 실행 시점에는 브라우저가 추가 내용은 읽지 못함

`firstChild`, `lastChild` 프로퍼티

- 첫 번째, 마지막 자식 노드에 빠르게 접근

```javascript
elem.childNodes[0] === elem.firstChild;
elem.childNodes[elem.childNodes.length - 1] === elem.lastChild;
```

`elem.hasChildNodes()`

- 자식 노드의 존재 여부 검사

DOM 컬렉션

- 읽는 것만 가능
  - `childNodes[i] = ...`로 자식 노드 교체 불가
  - DOM을 변경하려면 다른 메서드 사용
- DOM 컬렉션은 살아있음
  - DOM의 현재 상태를 반영
  - 몇몇 예외사항 제외
  - `elem.childNodes`를 참조하는 도중, DOM에 새로운 노드가 추가되거나 삭제되면, 변경사항이 컬렉션에도 자동으로 반영
- 컬렉션에 `for..in`을 사용하면 거의 사용되지 않는 추가 프로퍼티까지 순회
  - `for..of` 사용 권장

### 형제와 부모 노드

형제 노드(sibling node)

- 같은 부모를 가진 노드

`nextSibling`, `previousSibling`

- 다음, 이전 형제 노드에 관한 정보

`parentNode`

- 부모 노드에 대한 정보

```javascript
alert(document.body.parentNode === document.documentElement); // true. <html>은 <body>의 부모 노드
alert(document.head.nextSibling); // HTMLBodyElement. <body>는 <head>의 다음 형제 노드
alert(document.body.previousSibling); // HTMLHeadElement. <head>는 <body>의 이전 형제 노드
```

### 요소 간 이동

## 참고

> [문서](https://ko.javascript.info/document)
