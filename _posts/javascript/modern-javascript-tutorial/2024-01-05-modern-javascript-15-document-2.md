---
title: 모던 JavaScript 튜토리얼 15 - 문서 2
date: 2024-01-05 09:34:27 +0900
last_modified_at: 2024-04-14 15:29:50 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, node, attribute, property, dataset, style, class]
---

주요 노드 프로퍼티, 속성과 프로퍼티, 문서 수정하기, 스타일과 클래스

## 주요 노드 프로퍼티

### DOM 노드 클래스

DOM 노드

- 프로토타입을 기반으로 상속 관계를 갖는 일반 자바스크립트 객체
- 모든 DOM 노드는 공통 조상으로부터 생성되어 공통된 프로퍼티와 메서드 지원
- 종류에 따라 각각 다른 프로퍼티 지원
- DOM 노드는 종류에 따라 대응하는 내장 클래스가 다름

`console.dir(elem)`, `console.log(elem)`

- `elem`이 자바스크립트 객체이면 대개 같은 결과 출력
- `elem`이 DOM 요소이면,
- `console.log(elem)`: 요소의 DOM 트리 출력
- `console.dir(elem)`: 요소를 DOM 객체처럼 취급해 출력. 프로퍼티 확인 쉬움

```
[ EventTarget ]

[ Node ]

[ Text ] [ Element ] [ Comment ]

[ HTMLElement ] [ SVGElement ]

[ HTMLInputElement ] [ HTMLBodyElement ] [ HTMLAnchorElement ]
```

`EventTarget`

- 루트에 있는 추상(abstract) 클래스. 이 클래스의 객체는 실제로 생성되지 않음
- 이벤트 관련 기능 제공. 모든 DOM 노드의 베이스에 있기 때문에 DOM 노드에서 이벤트 사용 가능

`Node`

- 추상 클래스로, 해당하는 객체는 절대 생성되지 않음
- DOM 노드의 베이스 역할. 공통 DOM 노드 프로퍼티와 주요 트리 탐색 기능 제공
  - `parentNode`, `nextSibling`, `childNodes` 등
- `Text`, `Element`, `Comment` 클래스에게 상속

`Element`

- DOM 요소를 위한 베이스 클래스. 요소 전용 탐색을 도와주는 프로퍼티, 메서드가 기반함
  - `nextElementSibling`, `children`, `getElementsByTagName`, `querySelector` 등
- `SVGElement`, `XMLElement`, `HTMLElement` 클래스의 베이스 역할 (브라우저는 XML, SVG도 지원)

`HTMLElement`

- HTML 요소 노드의 베이스 클래스. HTML 요소 메서드와 getter, setter 제공
- 각 태그에 해당하는 클래스에 고유한 프로퍼티와 메서드 지원

예시: `<input>` 요소

- HTMLInputElement 클래스를 기반으로 생성되고 아래 클래스들에서 프로퍼티와 메서드를 상속받음
- `HTMLInputElement`: 입력 관련 프로퍼티 제공
- `HTMLElement`: HTML 요소 메서드와 getter, setter 제공
- `Element`: 요소 노드 메서드 제공
- `Node`: 공통 DOM 노드 프로퍼티 제공
- `EventTarget`: 이벤트 관련 기능 제공
- `Object`: 일반 객체 메서드 제공

DOM 노드 클래스 이름, 상속 여부 확인하기

- `constructor.name`, `toString`
- `instanceof`

```javascript
alert(document.body.constructor.name); // HTMLBodyElement
alert(document.body); // [object HTMLBodyElement]

alert(document.body instanceof HTMLBodyElement); // true
alert(document.body instanceof HTMLElement); // true
alert(document.body instanceof Element); // true
alert(document.body instanceof Node); // true
alert(document.body instanceof EventTarget); // true
```

Interface Description Language(IDL)

- 명세서에서는 IDL을 이용해 DOM 클래스를 설명

### 'nodeType' 프로퍼티

- `nodeType`: DOM 노드의 타입을 알아내는 구식 프로퍼티
- 각 노드 타입은 상수값을 가짐. 타입을 변경하는 데 사용은 불가

```html
<body>
  <script>
    let elem = document.body;
    alert(elem.nodeType); // 1. 요소 노드
    alert(elem.firstChild.nodeType); // 3. 텍스트 노드
    alert(document.nodeType); // 9. 문서 객체
  </script>
</body>
```

### nodeName과 tagName으로 태그 이름 확인하기

- DOM 노드의 태그 이름 확인
- `tagName`: 요소 노드에만 존재하는 프로퍼티
- `nodeName`: 모든 노드에 존재하는 프로퍼티
  - 요소 노드를 대상으로 하면 `tagName`과 같음
  - 텍스트 노드, 주석 노드 등에서는 노드 타입을 나타내는 문자열 반환

태그 이름은 항상 대문자

- XML 모드는 제외. 브라우저에서 HTML과 XML을 처리하는 모드는 다름
- 헤더가 `Content-Type: application/xml+xhtml`인 HTML 문서를 받으면 XML 모드로 문서 처리
- 웹 페이지는 대개 HTML 모드로 처리됨. HTML 모드에서는 `tagName`, `nodeName`이 대문자로 변경
- XML 모드에서는 케이스가 그대로 유지되지만 XML 모드는 요즘에는 거의 사용되지 않음

```html
<body>
  <!-- 주석 -->
  <script>
    alert(document.body.nodeName); // BODY
    alert(document.body.tagName); // BODY
    // 주석 노드를 대상으로 두 프로퍼티 비교
    alert(document.body.firstChild.tagName); // undefined (요소가 아님)
    alert(document.body.firstChild.nodeName); // #comment
    // 문서 노드를 대상으로 두 프로퍼티 비교
    alert(document.tagName); // undefined (요소가 아님)
    alert(document.nodeName); // #document
  </script>
</body>
```

### innerHTML로 내용 조작하기

- `innerHTML`: 요소 안의 HTML을 문자열 형태로 받을 수 있고 수정도 할 수 있는 프로퍼티
- 요소 노드에만 사용 가능하고 문법이 틀린 HTML을 넣으면 브라우저가 자동으로 수정
- `<script>` 태그를 삽입하면 해당 태그는 HTML의 일부가 되나 실행은 되지 않음

`elem.innerHTML += "추가 html"`

- 요소에 HTML 추가가 아니라 내용을 덮어씀
- `elem.innerHTML = elem.innerHTML + "추가 html"`의 축약
- 기존 내용을 완전히 삭제하고 기존 내용과 새로운 내용을 합친 새로운 내용을 밑바닥부터 씀
- 이미지 등의 리소스 전부 다시 로딩
  - 텍스트와 이미지가 많았다면 버벅이고, 입력 태그에 입력한 값이 있었다면 없어짐 등의 부작용
- `innerHTML` 말고 HTML을 추가하는 방법으로 부작용을 없앨 수 있음

```html
<body>
  <p>p 태그</p>
  <div>div 태그</div>
  <script>
    alert(document.body.innerHTML); // 내용 읽기
    document.body.innerHTML = "새로운 BODY!"; // 교체
  </script>
</body>
```

### outerHTML로 요소의 전체 HTML 보기

- `outerHTML`: 요소 전체 HTML이 담긴 프로퍼티
- `innerHTML`에 요소 자체를 더한 것이라 봐도 됨
- `innerHTML`과 달리 `outerHTML`로 HTML을 쓰면 요소 자체가 바뀌지 않고 DOM 안의 요소를 교체
- `outerHTML`에 하는 할당 연산이 DOM 요소를 수정하지 않기 때문
  - `outerHTML` 연산의 대상으로, 아래에서는 변수 `div`
- 할당 연산은 요소를 DOM에서 제거하고 새로운 HTML 조각을 넣음

```html
<div id="elem">Hello <b>World</b></div>
<div>Hello, world!</div>
<script>
  alert(elem.outerHTML); // <div id="elem">Hello <b>World</b></div>
  let div = document.querySelector("div");
  div.outerHTML = "<p>새로운 요소</p>";
  alert(div.outerHTML); // <div>Hello, world!</div>
</script>
```

- `div.outerHTML = ...`이 하는 일
- 문서에서 `div` 삭제를 삭제하고 새로운 HTML 조각인 `<p>...</p>`를 삭제 후 생긴 공간에 삽입
- `div`에는 여전히 기존값이 저장되어 있고 새로운 HTML 조각은 어디에도 저장되어 있지 않음

### nodeValue/data로 텍스트 노드 내용 조작하기

- `nodeValue`, `data` 프로퍼티
- 텍스트 노드 같은 다른 타입의 노드에 `innerHTML`과 유사한 역할
- 두 프로퍼티는 거의 유사하나 명세서상의 작은 차이가 있음
- `innerHTML` 프로퍼티는 요소 노드에만 사용 가능

```html
<body>
  안녕하세요.
  <!-- 주석 -->
  <script>
    let text = document.body.firstChild;
    alert(text.data); // 안녕하세요.
    let comment = text.nextSibling;
    alert(comment.data); // 주석
  </script>
</body>
```

```html
<!-- if isAdmin -->
<div>관리자로 로그인하였습니다!</div>
<!-- /if -->
```

- `data`로 주석 노드의 내용을 읽고 삽입된 지시사항을 처리하면 유용

### textContent로 순수한 텍스트만

- `textContent`: 요소 내의 텍스트에 접근해 태그는 제외하고 텍스트만 추출
- 텍스트를 안전한 방법으로 쓸 수 있기 때문에 유용

```html
<div id="news">
  <h1>주요 뉴스!</h1>
  <p>화성인이 지구를 침공하였습니다!</p>
</div>
<script>
  alert(news.textContent); // 주요 뉴스! 화성인이 지구를 침공하였습니다!
</script>
```

`innerHTML`, `innerText`, `textContent`

- `innerHTML`: `Element`의 속성
  - 사용자가 입력한 문자열이 HTML 형태로 태그와 함께 저장
- `innerText`: `Element`의 속성
  - 사용자에게 보이는 텍스트 값을 가져옴
- `textContent`: `Node`의 속성
  - `innerText`와 달리 해당 노드가 가진 텍스트 값을 그대로 읽음
  - 사용자가 입력한 문자열이 순수 텍스트 형태로 저장
  - 태그를 구성하는 특수문자들이 문자열로 처리되어 예상치 못한 HTML이 사이트에 침투하는 것을 방지

### hidden 프로퍼티

- `hidden`: 요소를 보여줄지 말지 지정
- HTML의 `hidden` 속성, JS의 `hidden` 프로퍼티
- 기술적으로 `style="display:none"`과 동일

```html
<div>아래 두 div를 숨겨봅시다.</div>
<div hidden>HTML의 hidden 속성 사용하기</div>
<div id="elem">자바스크립트의 hidden 프로퍼티 사용하기</div>
<script>
  elem.hidden = true;
</script>
```

### 기타 프로퍼티

- `value`: `<input>`, `<select>`, `<textarea>`의 값 저장
- `href`: `<a href="...">`의 href 속성값 저장
- `id`: id 속성값 저장
- 기타 등등
- `console.dir(elem)`으로 해당 요소에서 지원하는 프로퍼티 목록 확인 가능
- 대부분의 표준 HTML 속성은 대응하는 DOM 프로퍼티를 가지나 HTML 요소와 DOM 프로퍼티가 항상 같은 것은 아님

### 예제

노드 타입 맞추기

```html
<html>
  <body>
    <script>
      alert(document.body.lastChild.nodeType); // 1
    </script>
  </body>
</html>
```

- `<script>`가 실행되는 시점에는 브라우저가 `<script>` 아래 문서를 처리하지 못했음
- 가장 마지막 노드는 `<script>` 자기 자신

주석 안의 태그

```html
<script>
  let body = document.body;
  body.innerHTML = "<!--" + body.tagName + "-->";
  alert(body.firstChild.data); // BODY
</script>
```

1. `<body>`의 컨텐츠가 `<!--BODY-->`로 대체됨
2. 주석이 유일한 자식 노드가 됨
3. 주석 노드의 `data` 프로퍼티에는 주석 내용(`"BODY"`)이 저장됨

DOM 계층 구조와 'document'

- `document`는 `HTMLDocument` 클래스의 인스턴스
- `HTMLDocument` 클래스는 DOM 계층 구조에서 어디에 있을까?

```javascript
alert(document); // [object HTMLDocument]
alert(document.constructor.name); // HTMLDocument
alert(HTMLDocument.prototype.constructor === HTMLDocument); // true
alert(HTMLDocument.prototype.constructor.name); // HTMLDocument
alert(HTMLDocument.prototype.__proto__.constructor.name); // Document
alert(HTMLDocument.prototype.__proto__.__proto__.constructor.name); // Node
```

```
[ Node ]
[ Element ]     [ Document ]     [ CharacterData ] [ Attr ]
[ HTMLElement ] [ HTMLDocument ] [ Text ]
                                 [ Comment ]
```

## 속성과 프로퍼티

브라우저가 웹 페이지를 만날 때

1. HTML을 읽음(파싱, parsing)
2. DOM 객체 생성
3. 요소 노드에서 대부분의 표준 속성(attribute)은 DOM 객체의 프로퍼티(property)가 됨
   - 그러나 속성-프로퍼티가 항상 일대일 매핑되지는 않음

HTML 속성

- HTML 안에 쓰임
- 문자열만 가능
- 대소문자 가리지 않음

DOM 프로퍼티

- DOM 객체 안에 쓰임
- 모든 값 가능
- 대소문자 가림
- HTML 속성보다 DOM 프로퍼티를 사용하는 것을 권장

### DOM 프로퍼티

- 자신만의 DOM 프로퍼티 생성 가능
- 일반 자바스크립트 객체처럼 행동
- 어떤 값이든 가능
- 대소문자를 가림

```javascript
document.body.myData = { name: "Caesar", title: "Imperator" };
alert(document.body.myData.title); // Imperator
document.body.sayTagName = function () {
  alert(this.tagName);
};

document.body.sayTagName(); // BODY. sayTagName의 'this'에 document.body가 저장됨
Element.prototype.sayHi = function () {
  alert(`Hello, I'm ${this.tagName}`);
};

document.documentElement.sayHi(); // Hello, I'm HTML
document.body.sayHi(); // Hello, I'm BODY
```

### HTML 속성

- HTML 속성의 특징
  - HTML 태그는 복수의 속성을 가질 수 있음
  - 대소문자를 가리지 않음
  - 값은 항상 문자열
- HTML 표준 속성
  - 브라우저가 HTML을 파싱해 DOM 객체를 만들 때 HTML 표준 속성을 인식하고 이것을 사용해 DOM 프로퍼티 생성
- 비표준 속성
  - 비표준 속성은 프로퍼티로 전환되지 않음. 이에 매핑하는 DOM 프로퍼티가 생성되지 않음
  - 한 요소에서는 표준인 속성이 다른 요소에서는 표준이 아닐 수 있음

```html
<body id="test" something="non-standard">
  <script>
    alert(document.body.id); // test
    alert(document.body.something); // undefined (비표준 속성은 프로퍼티로 전환되지 않음)
  </script>
</body>

<body id="body" type="...">
  <input id="input" type="text" />
  <script>
    alert(input.type); // text
    alert(body.type); // undefined (type은 body의 표준 속성이 아니므로 DOM 프로퍼티가 생성되지 않음)
  </script>
</body>
```

비표준 속성에도 접근할 수 있는 메서드들

- `elem.hasAttribute(name)`: 속성 존재 여부 확인
- `elem.getAttibute(name)`: 속성값 가져오기
- `elem.setAttribute(name, value)`: 속성값 변경
- `elem.removeAttribute(name)`: 속성값 삭제
- `elem.attributes`: 모든 속성값 읽기
  - 내장 클래스 `Attr`을 구현한 객체들이 담긴 열거 가능한 컬렉션 반환
  - 각 객체에는 `name`, `value` 프로퍼티 존재
- `outerHTML`로 직접 추가한 속성을 포함한 모든 속성을 볼 수 있음

```html
<body something="non-standard">
  <script>
    alert(document.body.getAttribute("something")); // non-standard. 비표준 속성에 접근
  </script>
</body>
```

### 프로퍼티-속성 동기화

표준 속성이 변하는 경우

- 대응하는 프로퍼티는 자동 갱신
- 몇몇 경우를 제외하고 프로퍼티가 변하면 속성도 자동 갱신
  - `input.value`처럼 동기화가 속성에서 프로퍼티 방향으로만 일어나는 경우
  - 속성 `value` 수정 -> 프로퍼티 수정
  - 프로퍼티 수정 -> 속성 수정 X
  - 유저의 어떤 행동 때문에 `value`가 수정되었을 때, 속성에 저장된 값을 가져와 수정 전의 원래 값으로 복구 가능

```html
<input />
<script>
  let input = document.querySelector("input");
  input.setAttribute("id", "id"); // 속성 추가 -> 프로퍼티 갱신
  alert(input.id); // id
  input.id = "newId"; // 프로퍼티 변경 -> 속성 갱신
  alert(input.getAttribute("id")); // newId

  input.setAttribute("value", "text"); // 속성 추가 -> 프로퍼티 갱신
  alert(input.value); // text
  input.value = "newValue"; // 프로퍼티를 변경해도 속성이 갱신되지 않음
  alert(input.getAttribute("value")); // text. 갱신되지 않음
</script>
```

### DOM 프로퍼티 값의 타입

- DOM 프로퍼티는 항상 문자열이 아님
  - 대부분의 프로퍼티의 값은 문자열
  - 체크 박스에 사용되는 `input.checked` 프로퍼티는 불린값 가짐
  - `style` 속성은 문자열이지만 `style` 프로퍼티는 객체
- DOM 프로퍼티 값이 문자열이더라도 속성값과 다른 경우
  - `href` 속성이 상대 URL이나 `#hash`더라도 `href` DOM 프로퍼티에는 항상 URL 전체가 저장됨
  - HTML 내에 명시된 `href` 속성을 정확히 얻고 싶다면 `getAttribute` 사용

```html
<div id="div" style="color:red;font-size:120%">Hello</div>
<script>
  alert(div.getAttribute("style")); // color:red;font-size:120%. 문자열
  alert(div.style); // [object CSSStyleDeclaration]. 객체
  alert(div.style.color); // red
</script>

<a id="a" href="#hello">link</a>
<script>
  alert(a.getAttribute("href")); // #hello. 속성
  alert(a.href); // 실행되는 사이트 주소에 따라 http://site.com/page#hello 형태로 값이 저장됨. 프로퍼티
</script>
```

### 비표준 속성, dataset

- 사용자가 직접 지정한 데이터를 HTML에서 자바스크립트로 넘기고 싶은 경우
- 자바스크립트를 사용해 조작할 HTML 요소를 표시하기 위해 사용하는 경우
- 요소에 스타일을 적용하는 경우
- 새 클래스를 추가하거나 지우는 것보다 속성을 사용해 더 쉽게 상태 변경 가능
- `dataset`: `data-`로 시작하는 속성을 사용할 수 있는 프로퍼티
  - `data-`로 시작하는 속성 전체는 개발자가 용도에 맞게 사용하도록 별도로 예약됨

```html
<div show-info="name"></div>
<div show-info="age"></div>
<script>
  let user = { name: "Pete", age: 25 };
  for (let div of document.querySelectorAll("[show-info]")) {
    let field = div.getAttribute("show-info"); // 원하는 정보를 필드 값에 입력해 줌
    div.innerHTML = user[field]; // Pete -> 'name', 25 -> 'age'
  }
</script>
```

```html
<style>
  .order[order-state="new"] {
    color: green;
  }
  .order[order-state="pending"] {
    color: blue;
  }
  .order[order-state="canceled"] {
    color: red;
  }
</style>
<div class="order" order-state="new">A new order.</div>
<div class="order" order-state="pending">A pending order.</div>
<div class="order" order-state="canceled">A canceled order.</div>
```

```html
<body data-about="Elephants">
  <div data-order-state="new"></div>
  <script>
    alert(document.body.dataset.about); // Elephants
    alert(document.body.dataset.orderState); // new
  </script>
</body>
```

## 문서 수정하기

적시에 요소를 새롭게 생성하거나 페이지에 있는 기존 컨텐츠를 수정하는 방법

### 예제: 메시지 보여주기

```html
<style>
  .alert {
    padding: 15px;
    border: 1px solid #d6e9c6;
    border-radius: 4px;
    color: #3c763d;
    background-color: #dff0d8;
  }
</style>
<div class="alert">
  <strong>안녕하세요!</strong> 중요 메시지를 확인하셨습니다.
</div>
```

자바스크립트를 이용해보기

### 요소 생성하기

```javascript
let tag = document.createElement("tag"); // 태그 이름으로 새로운 요소 노드 생성
let textNode = document.createTextNode("text"); // 텍스트를 이용해 새로운 텍스트 노드 생성
```

```javascript
let div = document.createElement("div");
div.className = "alert";
div.innerHTML = "<strong>안녕하세요!</strong> 중요 메시지를 확인하셨습니다.";
document.body.append(div); // 삽입해야 페이지에 비로소 나타남
```

### 삽입 메서드

- `node.append(노드 or 문자열)`: node 끝에 노드나 문자열 삽입
- `node.prepend(노드 or 문자열)`: node 맨 앞에 노드나 문자열 삽입
- `node.before(노드 or 문자열)`: node 이전에 노드나 문자열 삽입
- `node.after(노드 or 문자열)`: node 다음에 노드나 문자열 삽입
- `node.replaceWith(노드 or 문자열)`: node를 새로운 노드나 문자열로 대체
- 문자열을 넘기면 자동으로 텍스트 노드 생성. `elem.textContent`처럼 안전한 방법으로 삽입됨
  - HTML이 아닌 넘긴 문자열 그 형태로 삽입됨. 특수문자는 이스케이프 처리
- 복수의 노드와 문자열 한번에 삽입 가능

```html
<ol id="ol">
  <li>0</li>
  <li>1</li>
  <li>2</li>
</ol>

<script>
  ol.before("before"); // <ol> 앞에 문자열 'before' 삽입
  ol.after("after"); // <ol> 뒤에 문자열 'after 삽입

  let liFirst = document.createElement("li");
  liFirst.innerHTML = "prepend";
  ol.prepend(liFirst); // <ol>의 첫 항목으로 liFirst를 삽입

  let liLast = document.createElement("li");
  liLast.innerHTML = "append";
  ol.append(liLast); // <ol>의 마지막 항목으로 liLast를 삽입
</script>
```

```
before
  1. prepend
  2. 0
  3. 1
  4. 2
  5. append
after
```

```html
before
<ol id="ol">
  <li>prepend</li>
  <li>0</li>
  <li>1</li>
  <li>2</li>
  <li>append</li>
</ol>
after
```

### insertAdjacentHTML/Text/Element

- HTML 그 자체를 삽입하고 싶은 경우 사용. `elem.innerHTML`을 사용한 것처럼 태그가 정상적으로 동작
- `elem.insertAdjacentHTML(where, html)`: `elem`을 기준으로 하는 상대 위치 `where`에 HTML 문자열 `html`을 삽입
  - `'beforebegin'`: `elem` 바로 앞
  - `'afterbegin'`: `elem`의 첫 번째 자식 요소 바로 앞
  - `'beforeend'`: `elem`의 마지막 자식 요소 바로 다음
  - `'afterend'`: `elem` 바로 다음
  - 이스케이프 처리되지 않고 그대로 삽입
- `elem.insertAdjacentText(where, text)`: HTML 대신 `text`를 문자 그대로 삽입
- `elem.insertAdjacentElement(where, elem)`: 요소를 삽입

### 노드 삭제하기

- `node.remove()`
- 요소 노드를 다른 곳으로 옮길 때, 기존에 있던 노드를 지울 필요 없음
- 모든 노드 삽입 메서드는 자동으로 기존에 있던 노드를 삭제하고 새로운 곳으로 노드를 옮김

### cloneNode로 노드 복제하기

- `elem.cloneNode(true)`: `elem`의 깊은 복제본 생성. 속성 전부와 자손 요소 전부 복사
- `false`를 전달하면 후손 노드 없이 `elem`만 복제

### DocumentFragment

```javascript
let fragment = new DocumentFragment();
```

- 특별한 DOM 노드 타입
- 여러 노드로 구성된 그룹을 감싸 다른 곳으로 전달하게 해주는 래퍼처럼 동작
- 문서에 있는 다른 노드를 `DocumentFragment`에 추가하고 그것을 문서 어딘가에 삽입하는 경우
- `DocumentFragment`는 사라지고 `DocumentFragment`에 추가한 노드만 남음

### 구식 삽입·삭제 메서드

- 하위 호환성을 위해 남은 구식 DOM 조작 메서드
- `parentElem.appendChild(node)`: `parentElem`의 마지막 자식으로 `node` 추가
- `parentElem.insertBefore(node, nextSibling)`: `parentElem` 안의 `nextSibling` 앞에 `node` 추가
- `parentElem.replaceChild(node, oldChild)`: `parentElem`의 자식 노드 중 `oldChild`를 `node`로 교체
- `parentElem.removeChild(node)`: `parentElem`에서 `node`를 삭제
- `node`가 `parentElem`의 자식이라는 가정 하
- 위 메서드들은 모두 삽입하거나 삭제한 노드 반환

### 'document.write'에 대한 첨언

- `document.write(html)`
- `html`이 페이지 그 자리에 즉시 추가됨
- DOM도 없고 그 어떤 표준도 존재하지 않았던 과거에 만들어진 메서드. 표준에 정의된 메서드가 아님
- 페이지를 불러오는 도중에만 작동
- 페이지가 다 로드된 후 `document.write`를 호출하면 기존에 있던 문서 내용이 사라짐
  - 그래서 페이지 로드가 다 끝난 후에는 사용 불가
- 브라우저는 HTML을 읽는(파싱하는) 도중에 `document.write(HTML)`를 만나면 텍스트 형식의 HTML을 마치 원래 페이지에 있었던 것처럼 해석
  - DOM 구조가 완성되기 전에 페이지에 내용이 삽입되어 중간에 DOM 조작을 하지 않기 때문에 속도가 아주 빨라지는 장점
  - 엄청나게 많은 글자를 HTML에 동적으로 추가해야 하는데 아직 페이지를 불러오는 중이고, 속도가 필요한 상황이면 유용
  - 하지만 이런 상황은 드물어서 잘 쓰지 않음

```html
<p>페이지 어딘가...</p>
<script>
  document.write("<b>자바스크립트를 사용해 Hello 입력</b>");
</script>
<p>끝</p>
```

```html
<p>일 초 후, 이 페이지의 내용은 전부 교체됩니다.</p>
<script>
  setTimeout(() => document.write("<b>...사라졌습니다.</b>"), 1000); // 페이지 로드가 끝난 후이므로 기존 내용 사라짐
</script>
```

### 예제

createTextNode vs innerHTML vs textContent

1. `elem.append(document.createTextNode(text))`
2. `elem.innerHTML = text`
3. `elem.textContent = text` (1과 3은 같은 동작을 수행)

요소 삭제하기

```html
<ol id="elem">
  <li>Hello</li>
  <li>World</li>
</ol>
<script>
  function clear(elem) {
    for (let i = 0; i < elem.childNodes.length; i++)
      elem.childNodes[i].remove();
  }
  clear(elem); // elem 내부 리스트를 삭제
</script>
```

- 잘못된 방법
- `remove()`는 `elem.childNodes`를 변화시키기 때문에 반복문을 실행할 때마다 `0`번째 인덱스부터 시작해야 함
- 그러나 `i`는 계속해서 증가하기 때문에 일부 원소들을 지나침
- `for..of` 반복문도 동일한 문제가 있음

```javascript
function clear(elem) {
  while (elem.firstChild) {
    elem.firstChild.remove();
  }
}

function clear(elem) {
  elem.innerHTML = "";
}
```

- 올바른 방법 2가지

왜 'aaa'가 남아 있을까요

```html
<table id="table">
  aaa
  <tr>
    <td>Test</td>
  </tr>
</table>
<script>
  alert(table); // table은 삭제할 테이블의 id
  table.remove(); // 삭제해도 문서 안에 aaa가 남아있음
</script>
```

- 주어진 HTML이 잘못되었기 때문
- 브라우저는 이를 자동으로 고침
- 그러나 명세서에 따르면 `<table>` 안에는 표와 관련된 특정 태그만이 존재할 수 있어 텍스트가 있어서는 안 됨
- 따라서 브라우저는 `aaa`를 `<table>` 앞에 추가
- 그래서 테이블을 삭제해도 텍스트가 남아있음

## 스타일과 클래스

요소에 스타일을 적용할 수 있는 방법 두 가지

1. CSS에 클래스를 만들고 요소에 클래스 추가하기
   - `<div class="...">`
2. 프로퍼티를 `style`에 바로 쓰기
   - `<div style="...">`

자바스크립트로 클래스와 `style` 프로퍼티 둘 다 수정 가능

- `style`보다 CSS 클래스 수정을 더 우선시해야 함
- `style`은 클래스를 다룰 수 없을 때만 사용할 것
- 자바스크립트를 사용해 요소의 좌표를 동적으로 계산하고 사용할 때 `style`을 사용하면 좋음
- CSS 관련 스타일을 명시한 클래스를 만들고, 자바스크립트로 이 클래스를 요소에 추가하는 방식 권장

### className과 classList

- `elem.class`
  - 과거 오래된 자바스크립트에는 `class` 같은 예약어는 객체의 프로퍼티로 사용 불가했음
  - 지금은 이런 제약사항은 없음
- `elem.className`
  - 클래스를 위한 프로퍼티로 `"class"` 속성에 대응
  - 무언가를 대입하면 전체가 바뀜
- `elem.classList`
  - 클래스 하나만 추가하거나 제거하고 싶을 경우 사용하는 프로퍼티
  - `elem.classList.add/remove("class")`: class를 추가하거나 제거
  - `elem.classList.toggle("class")`: class가 존재할 경우 class를 제거하고, 그렇지 않은 경우 추가
  - `elem.classList.contains("class")`: class 존재 여부 반환
  - 이터러블 객체이므로 `for..of` 사용 가능

```html
<body class="main page">
  <script>
    alert(document.body.className); // main page
    document.body.classList.add("article");
    alert(document.body.className); // main page article
    for (let name of document.body.classList) alert(name); // main, page, article
  </script>
</body>
```

### 요소의 스타일

- `elem.style`: 속성 `"style"`에 쓰인 값과 대응되는 객체
- 여러 단어를 이어 만든 프로퍼티는 카멜 케이스 사용

### style 프로퍼티 재지정하기

- `delete`로 프로퍼티를 삭제하는 대신 빈 문자열 할당
- 빈 문자열을 할당하면 브라우저는 마치 프로퍼티가 없었던 것처럼 CSS 클래스와 브라우저 내장 스타일을 페이지에 적용
- `style.cssText`: 문자열을 사용해 전체 스타일을 설정
  - 기존 스타일에 스타일을 추가하는 것이 아닌 전체를 교체
  - `elem.setAttribute('style', '...')`로 속성을 설정해도 같은 효과

```html
<div id="div">버튼</div>
<script>
  div.style.cssText = `color: red !important;
    background-color: yellow;
    width: 100px;
    text-align: center;
  `;
</script>
```

### 단위에 주의하기

자바스크립트로 스타일 값을 설정할 때는 단위를 붙여야 함

### getComputedStyle로 계산된 스타일 얻기

```javascript
getComputedStyle(element, [pseudo]);
```

- 프로퍼티의 결정값 반환. 프로퍼티 전체 이름이 필요
- 계산값을 얻기 위해 만들어진 오래된 메서드이나 결정값이 더 편하기 때문에 표준이 개정됨
- `element`: 값을 읽을 요소
- `pseudo`: 의사 요소가 필요할 경우 명시. 값이 없거나 빈 문자열이면 요소 자체를 의미
- `elem.style` 같이 스타일 정보가 들어 있는 객체를 반환하지만 `elem.style`과 달리 전체 CSS 클래스 정보도 함께 담김
- `:visited` 링크 관련 스타일은 숨겨짐
  - 방문한 적 있는 링크에 `:visited` CSS 의사 클래스 사용 가능
  - 자바스크립트로는 `:visited`에 적용된 스타일을 얻을 수 없음
  - 악의를 가진 페이지가 사용자가 링크를 방문했는지 여부를 테스트하고 사생활 침해를 방지하기 위함
- `style` 프로퍼티는 `"style"` 속성의 값을 읽을 때만 사용 가능
- `style` 프로퍼티만으로는 CSS 종속(CSS cascade) 값을 다룰 수 없음
- `elem.style`만으로는 CSS 클래스를 사용해 적용한 스타일을 읽을 수 없음

```html
<head>
  <style>
    body {
      color: red;
      margin: 5px;
    }
  </style>
</head>
<body>
  붉은 글씨
  <script>
    alert(document.body.style.color); // 빈 문자열
    alert(document.body.style.marginTop); // 빈 문자열

    let computedStyle = getComputedStyle(document.body);
    alert(computedStyle.marginTop); // 5px
    alert(computedStyle.color); // rgb(255, 0, 0)
  </script>
</body>
```

CSS 속성과 관련된 두 가지 개념

1. 계산값(computed style value): CSS 규칙, CSS 상속이 모두 적용된 값
   - `height:1em`, `font-size:125%` 등
2. 결정값(resolved style value): 요소에 최종적으로 적용되는 값
   - 브라우저는 계산값을 받아 고정 단위를 사용하는 절댓값으로 변환
   - `height:20px`, `font-size:16px` 등

## 참고

> [문서](https://ko.javascript.info/document)
