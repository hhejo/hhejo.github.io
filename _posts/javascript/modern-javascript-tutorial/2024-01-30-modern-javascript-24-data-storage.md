---
title: 모던 JavaScript 튜토리얼 24 - 브라우저에 데이터 저장하기 1
date: 2024-01-30 11:37:09 +0900
last_modified_at: 2024-04-22 11:32:17 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, cookie, localstorage, sessionstorage]
---

쿠키와 document.cookie, localStorage와 sessionStorage

## 쿠키와 document.cookie

쿠키

- 브라우저에 저장되는 작은 크기의 문자열로, RFC 6265 명세에서 정의한 HTTP 프로토콜의 일부
- 주로 웹 서버에 의해 만들어짐
- 서버가 HTTP 응답 헤더의 `Set-Cookie`에 내용을 넣어 전달하면, 브라우저는 이 내용을 자체적으로 브라우저에 저장. 이것이 쿠키
- 브라우저는 사용자가 쿠키를 생성하도록 한 동일 서버(사이트)에 접속할 때마다 쿠키의 내용을 `Cookie` 요청 헤더에 넣어 함께 전달
- 쿠키는 클라이언트 식별과 같은 인증에 가장 많이 쓰임
  1. 사용자가 로그인하면 서버는 HTTP 응답 헤더의 `Set-Cookie`에 담긴 세션 식별자(session identifier) 정보를 사용해 쿠키를 설정
  2. 사용자가 동일 도메인에 접속하려고 하면 브라우저는 HTTP `Cookie` 헤더에 인증 정보가 담긴 고윳값(세션 식별자)을 함께 실어 서버에 요청을 보냄
  3. 서버는 브라우저가 보낸 요청 헤더의 세션 식별자를 읽어 사용자를 식별
- `document.cookie` 프로퍼티로 브라우저에서도 쿠키에 접근

### 쿠키 읽기

- `document.cookie`: `name=value` 쌍으로 구성. 각 쌍은 `;`로 구분. 쌍 하나는 하나의 독립된 쿠키
- `;`을 기준으로 `document.cookie`의 값을 분리해 원하는 쿠키를 찾음

### 쿠키 쓰기

- `document.cookie`에 직접 값 작성 가능
- `cookie`는 데이터 프로퍼티가 아닌 접근자 프로퍼티
- `document.cookie`에 값을 할당하면 브라우저는 이 값을 받아 해당 쿠키를 갱신. 이때, 다른 쿠키의 값은 변경되지 않음
- 즉, `document.cookie=` 연산은 모든 쿠키를 덮어쓰지 않고 명시된 쿠키의 값만 갱신
- 쿠키의 이름과 값에는 특별한 제약이 없어 모든 글자가 허용됨
- 하지만 형식의 유효성을 일관성 있게 유지하기 위해 반드시 내장 함수 `encodeURIComponent`를 사용해 이름과 값을 이스케이프 처리

```javascript
document.cookie = "user=John"; // 이름이 user인 쿠키의 값만 갱신
alert(document.cookie); // 모든 쿠키 보여주기

let name = "my name"; // 공백 포함
let value = "John Smith"; // 공백 포함
document.cookie = encodeURIComponent(name) + "=" + encodeURIComponent(value);
alert(document.cookie); // ...; my%20name=John%20Smith (공백이 인코딩 처리됨)
```

쿠키의 한계

- 쿠키에는 몇 가지 제약 사항이 있음
- `encodeURIComponent`로 인코딩한 이후의 `name=value` 쌍은 `4KB`를 넘을 수 없음
- `4KB` 용량을 넘는 정보는 쿠키에 저장 불가
- 도메인 하나당 저장할 수 있는 쿠키의 개수는 20여 개 정도로 한정됨
- 개수는 브라우저에 따라 조금씩 다름

쿠키의 옵션

- 쿠키에는 몇 가지 옵션이 있음
- 몇몇 옵션은 아주 중요하기 때문에 꼭 지정해야 함
- 옵션은 `key=value` 뒤에 나열하고 `;`으로 구분

```javascript
document.cookie = "user=John; path=/; expires=Tue, 19 Jan 2038 03:14:07 GMT";
```

### path

- `path=/mypath`
- URL path(경로)의 접두사로, 이 경로의 하위 경로에 있는 페이지만 쿠키에 접근 가능
- 절대 경로여야 하고 (미지정 시) 기본값은 현재 경로
- `path=/admin` 옵션으로 설정한 쿠키는 `/admin`과 `/admin/something`에서는 볼 수 있지만 `/home`이나 `adminpage`에서는 볼 수 없음
- 특별한 경우가 아니라면 `path` 옵션을 `path=/` 같이 루트로 설정해 웹 사이트위 모든 페이지에서 쿠키에 접근할 수 있도록 함

### domain

- `domain=site.com`
- 쿠키에 접근 가능한 domain(도메인)을 지정
- 몇 가지 제약이 있어서 아무 도메인이나 지정할 수 없음
- `domain` 옵션에 아무 값도 넣지 않았다면 쿠키를 설정한 도메인에서만 접근 가능
- `site.com`에서 설정한 쿠키는 `other.com`에서 얻을 수 없음
- 서브 도메인(subdomain)인 `forum.site.com`에서도 쿠키 정보를 얻을 수 없다는 제약 사항이 하나 더 있음
- 서브 도메인이나 다른 도메인에서 쿠키에 접속할 방법은 없음
- `site.com`에서 생성한 쿠키를 `other.com`에서 절대 전송받을 수 없음
- 이런 제약 사항은 안정성을 높이기 위해 만들어짐
- 민감한 데이터가 저장된 쿠키는 관련 페이지에서만 볼 수 있도록 하기 위함
- 사실 서브 도메인에서 쿠키 정보를 얻는 방법이 있음
- `site.com`에서 쿠키를 설정할 때 `domain` 옵션에 루트 도메인인 `domain=site.com`을 명시적으로 설정하면 됨

```javascript
document.cookie = "user=John"; // site.com에서 쿠키를 설정
alert(document.cookie); // 찾을 수 없음 (site.com의 서브도메인 forum.site.com에서 user 쿠키에 접근하려 함)

document.cookie = "user=John; domain=site.com"; // site.com에서 서브 도메인(*.site.com) 어디서든 쿠키에 접속하게 설정할 수 있음
alert(document.cookie); // user=John 쿠키 확인 가능
```

### expires와 max-age

- `expires`(유효 일자)나 `max-age`(만료 기간) 옵션이 지정되어 있지 않으면 브라우저가 닫힐 때 쿠키도 함께 삭제됨
- 이런 쿠키를 세션 쿠키(session cookie)라고 부름
- `expires`나 `max-age` 옵션을 설정하면 브라우저를 닫아도 쿠키가 삭제되지 않음

`expires=Tue, 19 Jan 2038 03:14:07 GMT`

- 브라우저는 설정된 유효 일자까지 쿠키를 유지하다가 해당 일자가 도달하면 쿠키를 자동으로 삭제
- 쿠키의 유효 일자는 반드시 GMT 포맷으로 설정
- `date.toUTCString`을 사용하면 해당 포맷으로 쉽게 변경할 수 있음
- `expires` 옵션을 과거로 지정하면 쿠키는 삭제됨

```javascript
let date = new Date(Date.now() + 86400e3);
date = date.toUTCString();
document.cookie = "user=John; expires=" + date; // 유효기간이 하루인 쿠키 생성
```

`max-age=3600`

- `max-age`는 `expires` 옵션의 대안으로, 쿠키 만료 기간을 설정할 수 있게 해줌
- 현재부터 설정하고자 하는 만료일시까지의 시간을 초로 환산한 값을 설정
- 0이나 음수값을 설정하면 쿠키는 바로 삭제됨

```javascript
document.cookie = "user=John; max-age=3600"; // 1시간 뒤에 쿠키가 삭제됨
document.cookie = "user=John; max-age=0"; // 만료 기간을 0으로 지정하여 쿠키를 바로 삭제함
```

### secure

- `secure` 옵션을 설정하면 HTTPS로 통신하는 경우에만 쿠키가 전송됨
- `secure` 옵션이 없으면 기본 설정이 적용됨
- `http://site.com`에서 설정(생성)한 쿠키를 `https://site.com`에서 읽을 수 있고,
- `https://site.com`에서 설정(생성)한 쿠키도 `http://site.com`에서 읽을 수 있음
- 쿠키는 기본적으로 도메인만 확인하지 프로토콜을 따지지 않기 때문
- 하지만 `secure` 옵션이 설정된 경우, `https://site.com`에서 설정한 쿠키는 `http://site.com`에서 접근할 수 없음
- 쿠키에 민감한 내용이 저장되어 있어 암호화되지 않은 HTTP 연결을 통해 전송되는 것을 원치 않는다면 이 옵션을 사용하면 됨

```javascript
document.cookie = "user=John; secure"; // https://로 통신하고 있다고 가정. 설정한 쿠키는 HTTPS 통신 시에만 접근 가능
```

### samesite

## 참고

> [브라우저에 데이터 저장하기](https://ko.javascript.info/data-storage)
