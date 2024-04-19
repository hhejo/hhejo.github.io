---
title: input 태그의 속성을 사용해 입력을 제어하고 label 같이 사용하기
date: 2024-04-19 10:22:18 +0900
last_modified_at: 2024-04-19 10:22:18 +0900
categories: [HTML]
tags: [html, input, label]
---

input 태그의 여러 속성을 사용해 자바스크립트에 대한 의존을 줄여요. label도 같이 사용합시다.

## input 태그의 입력 관련 속성

`pattern`

- 정규식을 사용할 수 있음

`required`

- 입력하지 않고 전송하는 경우를 방지

`minlength`, `maxlength`

- 최소, 최대 입력 크기 제어

`placeholder`

- 입력창에 데이터가 없다면 나타나는 텍스트

## input 태그의 기타 유용한 속성

`size`

- 문자 개수를 단위로 해서 입력창의 길이 설정

`title`

- 커서를 올려 툴팁을 보여줌
- 요청한 형식과 입력이 다를 경우 표시되는 경고창에 `title` 내용도 출력됨

`type="hidden"`

- 데이터를 전송하지만 사용자에게는 보이지 않음

```html
<input type="hidden" id="postId" value="1234" />
```

`disabled`

- 읽기만 가능하고 변경은 불가능
- `<form>`으로 값이 전송되지 않음

`readonly`

- 읽기만 가능하고 변경은 불가능
- `<form>`으로 값이 전송됨

## label과 같이 사용하기

### label

`<label>` 태그는 `<input>` 태그를 라벨링해 의미를 좀 더 명확히 나타낼 수 있음

### label과 input을 연결하는 두 가지 방법

1. `<label>`의 `for` 속성값과 `<input>`의 `id`값을 같게 설정
2. `<label>` 안에 `<input>`을 위치시킴

## 예시: 전화번호 입력하기

```html
<form onsubmit="alert('Success')">
  <label for="phone">전화번호</label>
  <input
    type="tel"
    id="phone"
    placeholder="010-0000-0000"
    pattern="\d*"
    minlength="10"
    maxlength="11"
    title="하이픈(-) 없이 붙여서 입력"
    size="11"
    required
  />
</form>
```

## 참고

> [HTML/CSS 입력폼 유효성 검사에 필요한 input태그 pattern 속성](https://shurimp.tistory.com/46)

> [Form에서 label과 input 요소의 연결 및 title 사용법](https://hohoya33.tistory.com/41)

> [<input>: The Input (Form Input) element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)

> [<input> 타입, 속성 총정리 ](https://velog.io/@forest0501/input-타입-속성-총정리)
