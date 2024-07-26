---
title: 모던 JavaScript 튜토리얼 02 - 자바스크립트 기본 1
date: 2023-09-29 12:58:37 +0900
last_modified_at: 2024-07-26 18:23:34 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, script, statement, semicolon, comment, use-strict]
---

Hello, world!, 코드 구조, 엄격 모드

## Hello, world!

### script 태그

- `<script>` 태그를 이용해 자바스크립트 프로그램을 HTML 문서 대부분의 위치에 삽입 가능

```html
<script>
  alert("Hello, world!");
</script>
```

### 모던 마크업

- `<script>` 태그에는 몇 가지 속성(attribute)이 있으나, 요즘에는 잘 사용하지 않음

`type` 속성

- 이전에는 필수로 명시했으나, 현재(모던 자바스크립트)는 아님
- 모던 HTML 표준에서 의미가 바뀌어 자바스크립트 모듈에 사용할 수 있는 속성

```html
<script type="text/javascript"></script>
```

`language` 속성

- 현재 사용하고 있는 스크립트 언어를 나타냄
- 자바스크립트가 기본 언어이므로 사용할 필요 없음

```html
<script language=""></script>
```

스크립트 전후에 위치한 주석

- `<script>` 태그를 처리하지 못하는 브라우저가 해당 스크립트를 읽지 못하게 하려고 사용
- 모던 자바스크립트에서는 사용하지 않음

```
<script type="text/javascript"><!--
    ...
//--></script>
```

### 외부 스크립트

`src` 속성

- 자바스크립트 코드의 양이 많다면 파일로 나눠 저장
- 분해한 파일들을 `src` 속성을 이용해 HTML에 삽입
- 절대 경로, 상대 경로, 혹은 URL 전체를 사용할 수 있음
- `src` 속성이 있는 경우, 태그 내부의 코드 무시
- 스크립트가 아주 간단할 때만 HTML 안에 직접 작성하기

```html
<!-- 절대 경로 -->
<script src="/path/to/script.js"></script>
<!-- 상대 경로 -->
<script src="/js/script1.js"></script>
<script src="/js/script2.js"></script>
<!-- URL 전체 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js"></script>
<!-- src 속성이 있어 태그 내부의 코드 무시 -->
<script src="file.js">
  alert(1); // 무시됨
</script>
```

스크립트를 별도의 파일에 작성하기

- 브라우저가 스크립트를 다운받아 캐시에 저장하기 때문에 성능 향상
- 여러 페이지에서 동일한 스크립트를 사용하는 경우
  - 브라우저는 페이지가 바뀔 때마다 스크립트를 새로 다운받지 않고 캐시로부터 스크립트를 가져와 사용
- 스크립트 파일을 한 번만 다운받으면 되어 트래픽 절약, 웹 페이지 속도 향상

## 코드 구조

### 문(statement)

- 어떤 작업을 수행하는 문법 구조(syntax structur)와 명령어(command)를 의미

### 세미콜론(semicolon)

- 줄 바꿈이 있다면 세미콜론 생략 가능

세미콜론 자동 삽입(automatic semicolon insertion)

- 자바스크립트는 줄 바꿈이 있으면 이를 암시적 세미콜론으로 해석
- 자바스크립트는 대괄호 `[...]` 앞에는 세미콜론이 있다고 가정하지 않음
- 세미콜론을 사용하는 것을 권장

### 주석(comment)

- 자바스크립트 엔진은 주석을 무시하기 때문에 주석의 위치는 실행에 영향 없음

```javascript
// 한 줄 주석

/*
여러 줄
주석
*/
```

## 엄격 모드

ECMAScript(ES5)

- 새로운 기능이 추가되고 기존 기능 중 일부가 변경됨
- 변경사항 대부분은 ES5의 기본 모드에서는 활성화되지 않도록 설계됨
- 대신 `"use strict"` 지시자로 엄격 모드(strict mode)를 활성화했을 때만 이 변경사항 적용

### use strict

- `"use strict"` 지시자가 스크립트 최상단에 오면 스크립트 전체가 모던 방식으로 동작
- `"use strict"` 위에는 주석만 사용 가능
- 일단 적용되면 취소할 수 없음
- 함수 본문 맨 앞에 와 해당 함수만 엄격 모드로 실행 가능
- 브라우저 콘솔에서는 기본적으로 적용되어 있지 않기 때문에, 직접 입력해 적용
- 클래스, 모듈을 사용해 구성한다면 `"use strict"`를 생략해도 자동으로 적용

```html
"use strict";
<!-- 여기서부터 모던 방식으로 실행  -->
```

## 참고

> [자바스크립트 기본](https://ko.javascript.info/first-steps)
