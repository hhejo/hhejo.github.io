---
title: 모던 JavaScript 튜토리얼 23 - 네트워크 요청 1
date: 2024-01-16 20:04:41 +0900
last_modified_at: 2024-01-16 20:04:41 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

fetch, FormData 객체, Fetch: Download progress, Fetch: Abort

## fetch

자바스크립트를 사용하면 필요할 때 서버에 네트워크 요청을 보내고 새로운 정보를 받아오는 일을 할 수 있음

네트워크 요청은 다음과 같은 경우에 이뤄짐

- 주문 전송
- 사용자 정보 읽기
- 서버에서 최신 변경분 가져오기
- 등등

그런데 이 모든 것들은 페이지 새로고침 없이도 가능

AJAX

- Asynchronous JavaScript And XML
- 비동기적 JavaScript와 XML
- 서버에서 추가 정보를 비동기적으로 가져올 수 있게 해주는 포괄적인 기술을 나타내는 용어
- 만들어진 지 오래 됨
  - AJAX에 XML이 포함된 이유

AJAX 이외에 서버에 네트워크 요청을 보내고 정보를 받아올 수 있는 방법은 다양함

그 중 하나가 fetch

`fetch`

- 모던하고 다재다능한 메서드
- 대부분의 모던 브라우저가 지원

```javascript
let promise = fetch(url, [options]);
```

- `url`: 접근하고자 하는 URL
- `options`: 선택 매개변수. method나 header 등을 지정 가능
  - 아무것도 넘기지 않으면 요청은 `GET` 메서드로 진행되어 `url`로부터 컨텐츠가 다운로드 됨
- `fetch()`를 호출하면 브라우저는 네트워크 요청을 보내고 프라미스가 반환됨
- 반환되는 프라미스는 `fetch()`를 호출하는 코드에서 사용됨

응답은 대개 두 단계를 거쳐 진행됨

- 먼저, 서버에서 응답 헤더를 받자마자 `fetch` 호출 시 반환받은 `promise`가 내장 클래스 Response의 인스턴스와 함께 이행 상태가 됨
- 이 단계는 아직 본문(body)이 도착하기 전이지만, 개발자는 응답 헤더를 보고 요청이 성공적으로 처리되었는지 아닌지를 확인할 수 있음
- 네트워크 문제나 존재하지 않는 사이트에 접속하려는 경우 같이 HTTP 요청을 보낼 수 없는 상태에서는 프라미스는 거부 상태가 됨

HTTP 상태는 응답 프로퍼티를 사용해 확인 가능

- `status`: HTTP 상태 코드
- `ok`: 불린 값. HTTP 상태 코드가 200과 299 사이인 경우 `true`

```javascript
let response = await fetch(url);
if (response.ok) {
  let json = await response.json();
} else {
  alert(`HTTP-Error: ${response.status}`);
}
```

두 번째 단계에서는 추가 메서드를 호출해 응답 본문을 받음

`response`에는 프라미스를 기반으로 하는 다양한 메서드가 있어 이 메서드들로 다양한 형태의 응답 본문 처리 가능

- `response.text()`: 응답을 읽고 텍스트를 반환
- `response.json()`: 응답을 JSON 형태로 파싱
- `response.formData()`: 응답을 `formData` 객체 형태로 반환
- `response.blob()`: 응답을 Blob 형태로 반환
- `response.arrayBuffer()`: 응답을 ArrayBuffer 형태로 반환
- `response.body`: ReadableStream 객체. 응답 본문을 청크 단위로 읽을 수 있음

```javascript
let url =
  "https://api.github.com/repos/javascript-tutorial/ko.javascript.info/commits";
let response = await fetch(url);
let commits = await response.json(); // 응답 본문을 읽고 JSON 형태로 파싱
alert(commits[0].author.login);
```

```javascript
fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
)
  .then((response) => response.json())
  .then((commits) => alert(commits[0].author.login));
```

```javascript
let response = await fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
);
let text = await response.text(); // 응답 본문을 텍스트 형태로 읽기
alert(text.slice(0, 80) + "...");
```

```javascript
let response = await fetch("/article/fetch/logo-fetch.svg");
let blob = response.blob(); // 응답을 Blob 객체 형태로 다운로드
let img = document.createElement("img"); // 다운로드받은 Blob을 담을 <img>
img.style = "position:fixed;top:10px;left:10px;width:100px";
document.body.append(img);
img.src = URL.createObjectURL(blob); // 이미지를 화면에 보여줌
setTimeout(() => {
  img.remove();
  URL.revokeObjectURL(img.src);
}, 3000); // 3초 후 이미지 숨김
```

- fetch 명세서 우측 상단에 있는 로고(바이너리 데이터) 가져오기

본문을 읽을 때 사용되는 메서드는 딱 하나만 사용 가능

- `response.text()`를 사용해 응답을 얻었다면 본문의 컨텐츠는 모두 처리된 상태이기 때문에 `response.json()`은 동작하지 않음

```javascript
let text = await response.text(); // 응답 본문이 소비됨
let parsed = await response.json(); // 실패
```

### 응답 헤더

응답 헤더

- `response.headers`에 맵과 유사한 형태로 저장됨
- 맵은 아니지만 맵과 유사한 메서드를 지원
- 이 메서드들로 헤더 일부만 추출하거나 헤더 전체 순회 가능

```javascript
let response = await fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
);
alert(response.headers.get("Content-Type")); // application/json; charset=utf-8 (헤더 일부를 추출)
for (let [key, value] of response.headers) alert(`${key} = ${value}`); // 헤더 전체를 순회
```

### 요청 헤더

요청 헤더

- `headers` 옵션을 사용해 `fetch`에 요청 헤더를 설정할 수 있음
- `headers`에는 아래와 같이 다양한 헤더 정보가 담긴 객체를 넘기게 됨

```javascript
let response = fetch(protectedUrl, {
  headers: { Authentication: "secret" }
});
```

`headers`를 사용해 설정할 수 없는 헤더도 있음

- `Accept-Charset`, `Accept-Encoding`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`
- `Connection`
- `Content-Length`
- `Cookie`, `Cookie2`
- `Date`
- `DNT`
- `Expect`
- `Host`
- `Keep-Alive`
- `Origin`
- `Referer`
- `TE`
- `Trailer`
- `Transfer-Encoding`
- `Upgrade`
- `Via`
- `Proxy-*`
- `Sec-*`

이런 제약은 HTTP를 목적에 맞고 안전하게 사용할 수 있도록 하려고 만들어짐

금지 목록에 있는 헤더는 브라우저만 배타적으로 설정·관리할 수 있음

### POST 요청

.

## 참고

> [네트워크 요청](https://ko.javascript.info/network)
