---
title: 모던 JavaScript 튜토리얼 18 - 폼과 폼 조작
date: 2024-01-05 22:10:47 +0900
last_modified_at: 2024-01-05 22:10:47 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

폼 프로퍼티와 메서드, focus와 blur, 이벤트: change, input, cut, copy, paste, submit 이벤트와 메서드

## 폼 프로퍼티와 메서드

폼(form) 조작에 사용되는 요소에는 특별한 프로퍼티와 이벤트가 많음

### 폼과 요소 탐색하기

`document.forms`

- 특수한 컬렉션
- 폼(form)이 이것의 구성원
- 이름과 순서가 있는 기명 컬렉션(named collection)
- 개발자는 이 이름이나 순서를 사용해 문서 내의 폼에 접근 가능

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
    // 이름을 사용해 form과 fieldset 모두에서 input을 구할 수 있음
    alert(fieldset.elements.login == form.elements.login); // true
  </script>
</body>
```

짧은 표기법: `form.name`

- `form[index/name]`으로도 요소에 접근 가능
  - `form.elements.login` 대신 `form.login`
- `form.name`을 사용한 표기법은 잘 작동하나 요소에 접근해 `name` 속성을 변경해도 변경 전 이름을 계속 사용할 수 있는 문제가 있음
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

모든 요소는 `element.form`으로 폼에 접근 가능

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

폼 조작에 사용되는 요소들

input과 textarea

- 요소의 값 얻기
- `input.value`(string)
- `input.checked`(boolean)
- `textarea.value`
  - `textarea.innerHTML`에는 페이지를 처음 열 당시의 HTML만 저장됨
  - 최신 값을 구할 수 없기 때문에 사용하지 말 것

```javascript
input.value = "New value";
textarea.value = "New text";
input.checked = true; // 체크박스나 라디오 버튼에서 사용 가능
```

select와 option

- `<select>` 요소의 세 가지 중요 프로퍼티
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

- 사용자가 폼 요소를 클릭하거나 Tab 키를 눌러 요소로 이동하면 해당 요소가 포커스(focus) 됨
- 요소를 포커싱한다는 것은 일반적으로 '여기에 데이터를 입력할 준비를 하라'는 것을 의미

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

## 이벤트: change, input, cut, copy, paste

## submit 이벤트와 메서드

## 참고

> [폼과 폼 조작](https://ko.javascript.info/forms-controls)
