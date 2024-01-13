---
title: 모던 JavaScript 튜토리얼 18 - 폼과 폼 조작
date: 2024-01-05 22:10:47 +0900
last_modified_at: 2024-01-13 09:42:00 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

폼 프로퍼티와 메서드, focus와 blur, 이벤트: change, input, cut, copy, paste, submit 이벤트와 메서드

## 폼 프로퍼티와 메서드

폼(form) 조작에 사용되는 요소에는 특별한 프로퍼티와 이벤트가 많음

### 폼과 요소 탐색하기

`document.forms`

- 특수한 컬렉션으로 폼(form)이 이것의 구성원
- 이름과 순서가 있는 기명 컬렉션(named collection)

```javascript
document.forms.my; // 이름이 'my'인 폼
document.forms[0]; // 문서 내의 첫 번째 폼
```

`form.elements`

- 폼의 요소 얻기

```html
<form name="my">
  <input name="one" value="1" />
  <input name="two" value="2" />
</form>
<script>
  let form = document.forms.my; // <form name="my"> 요소
  let elem = form.elements.one; // <input name="one"> 요소
  alert(elem.value); // 1
</script>
```

`form.elements[name]`

- 이름이 같은 요소 여러 개 다루는 경우
- `form.elements[name]`은 컬렉션이 됨
- 라디오 버튼 등

```html
<form>
  <input type="radio" name="age" value="10" />
  <input type="radio" name="age" value="20" />
</form>
<script>
  let form = document.forms[0];
  let ageElems = form.elements.age;
  alert(ageElems[0]); // [object HTMLInputElement]
</script>
```

`fieldset`

- 하위 폼처럼 쓰임
- 폼은 하나 이상의 `<fieldset>` 요소를 포함 가능
- `elements` 프로퍼티 지원

```html
<body>
  <form id="form">
    <fieldset name="userFields">
      <legend>info</legend>
      <input name="login" type="text" />
    </fieldset>
  </form>
  <script>
    alert(form.elements.login); // <input name="login">
    let fieldset = form.elements.userFields;
    alert(fieldset); // HTMLFieldSetElement
    alert(fieldset.elements.login == form.elements.login); // true. 이름을 사용해 form과 fieldset 모두에서 input을 구할 수 있음
  </script>
</body>
```

`form.name`

- 짧은 표기법
- `form[index/name]`으로도 요소에 접근 가능
  - `form.elements.login` 대신 `form.login`
- 요소에 접근해 `name` 속성을 변경해도 변경 전 이름을 계속 사용할 수 있는 문제가 있음
  - 새로운 이름도 물론 사용 가능
  - 그러나 폼 요소의 이름을 변경하는 일은 드물어 보통은 문제되지 않음

```html
<form id="form">
  <input name="login" />
</form>
<script>
  alert(form.elements.login == form.login); // true, 동일한 <input>
  form.login.name = "username"; //  input의 name 속성을 변경
  // form.elements에는 name 속성 변경이 반영됨
  alert(form.elements.login); // undefined
  alert(form.elements.username); // input
  // form은 새로운 이름과 이전 이름을 모두 인식
  alert(form.username == form.login); // true
</script>
```

### element.form으로 역참조하기

`element.form`

- 모든 요소는 `element.form`으로 폼에 접근 가능

```html
<form id="form">
  <input type="text" name="login" />
</form>
<script>
  let login = form.login; // 폼 -> 요소
  alert(login.form); // HTMLFormElement. 요소 -> 폼
</script>
```

### 폼 요소

input과 textarea

- 요소의 값 얻기
- `input.value`(string)
- `input.checked`(boolean)
- `textarea.value`
- `textarea.innerHTML`: 페이지를 처음 열 당시의 HTML만 저장됨
  - 최신 값을 구할 수 없기 때문에 사용하지 말 것

```javascript
input.value = "New value";
textarea.value = "New text";
input.checked = true; // 체크박스나 라디오 버튼에서 사용 가능
```

select와 option

- `select.options`: `<option>` 하위 요소를 담은 컬렉션
- `select.value`: 현재 선택된 `<option>` 값
- `select.selectedIndex`: 현재 선택된 `<option>`의 인덱스
- `<select>`의 값 설정하는 세 가지 방법
  1. 조건에 맞는 `<option>` 하위 요소를 찾아 `option.selected` 속성을 `true`로 설정
     - 가장 확실
  2. `select.value`를 원하는 값으로 설정
  3. `select.selectedIndex`를 원하는 option 번호로 설정
     - 2, 3번 방법이 더 편리
- `multiple` 속성이 있는 경우 option을 다중 선택 가능
  - 쓰게 된다면 첫 번째 방법으로 `<option>` 하위 요소에 있는 `selected` 프로퍼티를 추가·제거해야 함

```html
<select id="select">
  <option value="apple">Apple</option>
  <option value="pear">Pear</option>
  <option value="banana">Banana</option>
</select>
<script>
  // 세 가지 코드의 실행 결과는 모두 동일
  select.options[2].selected = true;
  select.selectedIndex = 2;
  select.value = "banana";
</script>
```

```html
<select id="select" multiple>
  <option value="blues" selected>Blues</option>
  <option value="rock" selected>Rock</option>
  <option value="classic">Classic</option>
</select>
<script>
  let selected = Array.from(select.options)
    .filter((option) => option.selected)
    .map((option) => option.value); // 선택한 값 전체
  alert(selected); // blues,rock
</script>
```

option 생성자

```javascript
option = new Option(text, value, defaultSelected, selected);
```

- 잘 사용되지 않음

...

## focus와 blur

focus

- 요소로 이동하면 해당 요소 포커스
  - 사용자가 폼 요소 클릭
  - Tab 키를 눌러 요소로 이동
- 일반적으로 '여기에 데이터를 입력할 준비를 하라'는 것을 의미

`autofocus`

- HTML 속성
- 이 속성이 있는 요소는 페이지가 로드된 후 자동으로 포커싱 됨

blur

- 요소가 포커스를 잃는 순간
- 대개 데이터 입력이 완료된 것을 의미
- 포커싱이 해제되는 순간 데이터를 체크하거나 저장하기 위해 서버에 요청을 보내는 등의 코드를 실행할 수 있음

### focus, blur 이벤트

`focus`, `blur` 이벤트

- 요소가 포커스를 받을 때, 잃을 때 발생

```html
<style>
  .invalid {
    border-color: red;
  }
  #error {
    color: red;
  }
</style>

이메일: <input type="email" id="input" />
<div id="error"></div>

<script>
  input.onblur = function () {
    if (!input.value.includes("@")) {
      // @ 유무를 이용해 유효한 이메일 주소가 아닌지 체크
      input.classList.add("invalid");
      error.innerHTML = "올바른 이메일 주소를 입력하세요.";
    }
  };
  input.onfocus = function () {
    if (this.classList.contains("invalid")) {
      // 사용자가 새로운 값을 입력하려고 하므로 에러 메시지를 지움
      this.classList.remove("invalid");
      error.innerHTML = "";
    }
  };
</script>
```

### focus, blur 메서드

`elem.focus()`, `elem.blur()`

- 요소에 포커스를 주거나 제거

```html
<style>
  .error {
    background: red;
  }
</style>

이메일: <input type="email" id="input" />
<input
  type="text"
  style="width:220px"
  placeholder="이메일 형식이 아닌 값을 입력하고 여기에 포커스를 주세요."
/>

<script>
  input.onblur = function () {
    if (!this.value.includes("@")) {
      // 이메일이 아니면 에러 출력
      this.classList.add("error");
      input.focus(); // input 필드 포커스
    } else {
      this.classList.remove("error");
    }
  };
</script>
```

- `onblur`는 요소가 포커스를 잃고 난 후에 발생
- `onblur` 안에서 `event.preventDefault()`를 호출해 포커스를 잃게 하는 것을 막을 수 없음

자바스크립트로 인한 포커스 손실

- 포커스는 손실은 많은 이유로 발생
- 자바스크립트 자체가 이를 유발할 수 있음
  - `alert`는 포커스를 자신에게 옮기고 해제되면 되돌아감
  - 요소가 DOM에서 제거되면 포커스 손실이 발생하고, 다시 삽입돼도 포커스가 돌아오지 않음
- 이러한 기능으로 `focus`, `blur` 핸들러가 오작동할 수 있음
- 주의해서 사용할 것

### tabindex를 사용해서 모든 요소 포커스 하기

요소의 포커스

- 대다수의 요소는 기본적으로 포커싱을 지원하지 않음
  - `<div>`, `<span>`, `<table>`, ...
  - `elem.focus()`가 동작하지 않고 `focus`, `blur` 이벤트 트리거 되지 않음
- 사용자가 웹 페이지와 상호작용할 수 있게 도와주는 요소는 `focus`,`blur`를 지원
  - `<button>`, `<input>`, `<select>`, `<a>`, ...

`tabindex` HTML 속성

- `tabindex`가 있는 요소는 종류와 상관 없이 포커스 가능
- 속성값은 숫자
- Tab 키를 눌러 요소 사이를 이동할 때의 순서
- `tabindex`가 없는 요소는 문서 내 순서에 따라 포커스 이동

`tabindex` 주의 사항

- `tabindex`가 `0`인 요소
  - `tabindex` 속성이 없는 것처럼 동작
  - 포커스를 이동시킬 때 `tabindex`가 `1`보다 크거나 같은 요소보다 나중에 포커스 받음
  - `tabindex="0"`은 포커스 가능하게 만들지만 포커스 순서는 기본 순서 그대로 유지하고 싶을 때 사용
  - 요소의 포커스 우선 순위를 일반 `<input>`과 같아지도록 함
- `tabindex`가 `-1`인 요소
  - 스크립트로만 포커스 하고 싶은 요소에 사용
  - Tab 키를 사용하면 이 요소는 무시되지만 `elem.focus()` 메서드를 사용하면 잘 포커싱 됨
- `elem.tabIndex` 프로퍼티를 사용해도 `tabindex` 속성을 사용한 것과 동일 효과

```html
첫 번째 항목을 클릭하고 Tab 키를 눌러보면서 포커스 된 요소 순서를 눈여겨보세요.
참고로 탭을 많이 누르면 예시 밖으로 포커스가 이동하니, 주의하세요.
<ul>
  <li tabindex="1">일</li>
  <li tabindex="0">영</li>
  <li tabindex="2">이</li>
  <li tabindex="-1">음수 일</li>
</ul>
<style>
  li {
    cursor: pointer;
  }
  :focus {
    outline: 1px dashed green;
  }
</style>
```

- `1` -> `2` -> `0` 순서로 이동
- `-1`은 무시됨

### focusin과 focusout을 사용해 이벤트 위임하기

`focus`, `blur` 이벤트

- 버블링 되지 않음
- 이벤트 위임 효과를 주는 방법 2가지
  1. `focus`, `blur`는 버블링 되지 않지만 캡처링은 됨
  2. `focusin`, `focusout` 사용하기

`focusin`, `focusout` 이벤트

- `focus`, `blur`와 동일하나 버블링 됨
- `elem.addEventListener` 방식으로 핸들러 추가
- `on<event>` 방식으로 추가하면 안 됨

## 이벤트: change, input, cut, copy, paste

데이터가 변경될 때 실행되는 다양한 이벤트

### 이벤트: change

`chage` 이벤트

- `onchange` 속성
- 요소 변경이 끝나면 발생
- 텍스트 입력 요소는 요소 변경이 끝날 때가 아니라 포커스를 잃을 때 이벤트 발생
- `select`, `input type=checkbox`, `input type=radio`의 경우, 선택 값이 변경된 직후 이벤트 발생

### 이벤트: input

`input` 이벤트

- `oninput` 속성
- 사용자가 값을 수정할 때마다, 수정하자마자 발생
- 키보드 이벤트와 달리, 어떤 방법으로든 값을 변경할 때 발생
  - 마우스로 글자 붙여넣기
  - 음성인식 기능으로 글자 입력
- 수정이 일어날 때마다 이벤트를 실행하고 싶다면 사용
- <-, -> 키 같이 값을 변경시키지 않는 키보드 입력, 동작에는 반응하지 않음
- 그 무엇도 `oninput`을 막을 수 없음
  - `event.preventDefault()`로 기본 동작을 막아도 값이 수정되면 그 즉시 `input` 이벤트 발생
  - `event.preventDefault()`를 써봤자 효과 없음

### 이벤트: cut, copy, paste

`cut`, `copy`, `paste` 이벤트

- `oncut`, `oncopy`, `onpaste` 속성
- 값을 잘라내기·복사하기·붙여넣기 할 때 발생
- `ClipboardEvent` 클래스의 하위 클래스
- `event.clipboardData` 프로퍼티로 클립보드에 저장된 데이터 읽고 쓰기 가능
  - `event.clipboardData.getData('text/plain')`
- 복사하거나 붙여넣기 한 데이터에 접근 가능
- `event.preventDefault()`로 기본 동작을 막아 아무것도 복사·붙여넣기 할 수 없게 가능
- 텍스트뿐만 아니라 파일 등 모든 것 가능
- 클립보드는 전역 OS 레벨
- 대부분의 브라우저는 안전을 위해 특정 사용자 동작의 범위에서만 클립보드의 읽기·쓰기에 대한 접근 허용
- 모든 브라우저에서 `dispatchEvent`를 사용해 커스텀 클립보드 이벤트 생성을 금지
  - Firefox 제외

## submit 이벤트와 메서드

### submit 이벤트

`submit` 이벤트

- 폼을 제출할 때 트리거 됨
- 폼을 서버로 전송하기 전에 내용 검증 시, 폼 전송 취소 시 사용

폼을 전송하는 2가지 방법

- `<input type="submit">`, `<input type="image">` 클릭
- 인풋 필드에서 Enter 키 누르기
  - `click` 이벤트도 트리거 됨
- 두 방법 모두 `submit` 이벤트 트리거

input 필드에서 Enter 키를 눌러 폼을 전송하면 `<input type="submit">`에 있는 `click` 이벤트가 트리거 됨

```html
<form onsubmit="return false">
  <input
    type="text"
    size="30"
    value="여기에 포커스를 준 다음에 Enter 키 누르기"
  />
  <input
    type="submit"
    value="제출"
    onclick="alert('클릭 이벤트가 트리거 되었습니다!')"
  />
</form>
```

### submit 메서드

`form.submit()`

- 자바스크립트로 직접 폼을 서버에 전송
- 호출된 다음에는 `submit` 이벤트 생성되지 않음
  - 개발자가 호출했다면, 스크립트에서 이미 필요한 모든 조치를 했다고 가정하기 때문

## 참고

> [폼과 폼 조작](https://ko.javascript.info/forms-controls)
