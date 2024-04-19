---
title: 모던 JavaScript 튜토리얼 23 - 네트워크 요청 3
date: 2024-01-19 08:25:17 +0900
last_modified_at: 2024-04-20 07:23:25 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, fetch, url]
---

Fetch API, URL objects, XMLHttpRequest

## Fetch API

```javascript
let promise = fetch(url, {
  method: "GET", // POST, PUT, DELETE 등
  headers: {
    "Content-Type": "text/plain;charset=UTF-8" // content type 헤더 값은 일반적으로 요청 본문에 따라 자동으로 설정됨
  },
  body: undefined, // string, FormData, Blob, BufferSource, 혹은 URLSearchParams
  referrer: "about:client", // ""로 Referer 헤더를 보내지 않거나 혹은 현재 오리진의 url을 전송
  referrerPolicy: "no-referrer-when-downgrade", // no-referrer, origin, same-origin...
  mode: "cors", // same-origin, no-cors
  credentials: "same-origin", // omit, include
  cache: "default", // no-store, reload, no-cache, force-cache 혹은 only-if-cached
  redirect: "follow", // manual, error
  integrity: "", // "sha256-abcdef1234567890" 같은 해시
  keepalive: false, // true
  signal: undefined, // 요청을 중단하는 AbortController
  window: window // null
});
```

### referrer, referrerPolicy

`referrer`

- `fetch`가 HTTP `Referer` 헤더를 설정하는 방법을 제어하는 옵션
- 일반적으로 해당 헤더는 자동으로 설정되며 요청한 페이지의 URL을 포함
- 대부분의 시나리오에서는 전혀 중요하지 않으며, 때로는 보안 목적으로 제거하거나 단축하는 것이 합리적
- `referrer` 옵션을 사용하면 현재 오리진 내에서 `Referer`를 설정하거나 제거 가능

```javascript
// 현재 https://javascript.info에 있다고 가정
fetch("/page", {
  referrer: "", // Referer 헤더 없이 전송
  referrer: "https://javascript.info/anotherpage" // 현재 오리진 내에서 다른 URL 설정. Referer 헤더를 설정할 수 있지만 현재 오리진 내에서만 가능
});
```

`referrerPolicy`

- `Referer`에 대한 일반 규칙을 설정
- 요청은 3가지 유형으로 나뉨
  - 동일한 출처로 요청
  - 다른 출처로 요청
  - HTTPS에서 HTTP로 요청 (안전한 프로토콜에서 안전하지 않은 프로토콜로)
- 정확한 `Referer` 값을 설정할 수 있는 `referrer` 옵션과 달리, `referrerPolicy`는 브라우저에 각 요청 유형에 대한 일반 규칙을 알려줌

Referrer Policy 사양에 설명된 가능한 값

- `no-referrer-when-downgrade`: 기본값. 전체 `Referer`는 HTTPS에서 HTTP(덜 안전한 프로토콜)로 요청을 보내지 않는 한 항상 전송됨
- `no-referrer`: 절대 `Referer`를 전송하지 않음
- `origin`: 전체 페이지 URL이 아닌 `Referer`의 오리진만 전송 (`http://site.com/path` 대신 `http://site.com`)
- `origin-when-cross-origin`: 전체 `Referer`를 동일한 오리진으로 보내지만, 크로스 오리진 요청의 경우 오리진 부분만 보냄 (위와 같음)
- `same-origin`: 동일한 오리진에는 전체 `Referer`를 보내지만 크로스 오리진 요청에 대해서는 `Referer`를 보내지 않음
- `strict-origin`: 오리진만 보내고 HTTPS에서 HTTP 요청에 대해서는 `Referer`를 보내지 않음
- `strict-origin-when-cross-origin`: 동일한 오리진의 경우 전체 `Referer`를 보내고, 크로스 오리진에 대해서는 HTTPS -> HTTP 요청이 아닌 한 오리진만 보내고 아무것도 보내지 않음
- `unsafe-url`: HTTPS -> HTTP 요청이더라도 항상 `Referer`에 전체 url을 보냄

|                       값                       | To same origin | To another origin | HTTPS -> HTTP |
| :--------------------------------------------: | :------------: | :---------------: | :-----------: |
|                 `no-referrer`                  |       -        |         -         |       -       |
| `no-referrer-when-downgrade` 혹은 `""`(기본값) |      full      |       full        |       -       |
|                    `origin`                    |     origin     |      origin       |    origin     |
|           `origin-when-cross-origin`           |      full      |      origin       |    origin     |
|                 `same-origin`                  |      full      |         -         |       -       |
|                `strict-origin`                 |     origin     |      origin       |       -       |
|       `strict-origin-when-cross-origin`        |      full      |      origin       |       -       |
|                  `unsafe-url`                  |      full      |       full        |     full      |

사이트 외부에서 알 수 없는 URL 구조를 가진 어드민 영역이 있다고 가정

- `fetch`를 전송하면 기본적으로 항상 페이지의 전체 URL과 함께 `Referer` 헤더를 보냄 (HTTPS -> HTTP 요청하는 경우에는 `Referer`가 없음)
- 예: `Referer: https://javascript.info/admin/secret/paths`

```javascript
fetch("https://another.com/page", {
  referrerPolicy: "origin-when-cross-origin" // Referer: https://javascript.info
});
```

- 다른 웹 사이트에서 URL 경로가 아닌 원본 부분만 알고 싶다면 다음 옵션을 설정할 수 있음
- 모든 `fetch` 호출에 이를 넣을 수 있음
  - 모든 요청을 수행하고 내부에서 `fetch`를 사용하는 프로젝트의 자바스크립트 라이브러리에 통합할 수도 있음
- 기본 동작과 비교했을 때 유일한 차이점은 다른 오리진 가져오기에 대한 요청의 경우 경로 없이 URL의 원본 부분만 보낸다는 것
  - 예: `https://javascript.info`
- 오리진에 대한 요청의 경우에도 전체 `Referer`를 얻음
  - 디버깅 목적에 유용할 수 있음

Referer 정책은 `fetch`에만 국한되지 않음

- 사양에 설명된 Referer 정책은 `fetch` 전용이 아니라 더 전역적
- 특히, Referrer-Policy HTTP 헤더 또는 링크 단위로 `<a rel="noreferrer">`를 사용하여 전체 페이지에 대한 기본 정책을 설정 가능

### mode

- `mode` 옵션: 가끔 크로스 오리진 요청을 방지하는 안전 장치
- `"cors"`: 기본값. CORS에 설명된 대로 크로스 오리진 요청이 허용됨
- `"same-origin"`: 크로스 오리진 요청은 금지됨
- `"no-cors"`: 단순 교차 출처 요청만 허용됨
- `fetch` URL이 서드파티에서 올 경우 유용할 수 있으며 교차 출처 기능을 제한하기 위해 전원 끄기 스위치를 원할 때 유용함

### credentials

- `credentials` 옵션: `fetch`가 요청과 함께 쿠키 및 HTTP-Authorization 헤더를 보내야 하는지 여부를 지정
- `"same-origin"`: 기본값. 크로스 오리진 요청을 보내지 않음
- `"include"`: 항상 보냄. 자바스크립트가 응답에 접근하려면 크로스 오리진 서버의 `Accept-Control-Allow-Credentials`가 필요
- `"omit"`: 동일한 오리진 요청의 경우에도 절대 보내지 않음

### cache

- 기본적으로 `fetch` 요청은 표준 HTTP-caching을 사용
- 즉, `Expires`, `Cache-Control` 헤더를 존중하고 `If-Modified-Since` 등을 보냄. 일반 HTTP 요청과 마찬가지로
- `cache` 옵션: HTTP-cache를 무시하거나 사용법을 미세 조정
- `"default"`: `fetch`는 표준 HTTP-cache 규칙과 헤더를 사용
- `"no-store"`: HTTP-cache를 완전히 무시
  - `If-Modified-Since`, `If-None-Match`, `If-Unmodified-Since`, `If-Match` 혹은 `If-Range` 헤더를 설정하면 이 모드가 기본값이 됨
- `"reload"`: HTTP-cache(있는 경우)에서 결과를 가져오지 말고 응답으로 캐시를 채움 (응답 헤더가 허용하는 경우)
- `"no-cache"`: 캐시된 응답이 있으면 조건부 요청을 생성하고, 그렇지 않으면 일반 요청을 생성. 응답으로 HTTP-cache 채우기
- `"force-cache"`: 오래되었더라도 HTTP-cache의 응답을 사용. HTTP-cache에 응답이 없으면 일반 HTTP 요청을 수행하고 정상적으로 동작
- `"only-if-cached"`: 오래되었더라도 HTTP-cache의 응답을 사용. HTTP-cache에 응답이 없으면 에러. `mode`가 `"same-origin"`일 때만 작동

### redirect

- 일반적으로, `fetch`는 301, 302와 같은 HTTP 리디렉션을 투명하게 따름
- `redirect` 옵션으로 이를 변경할 수 있음
- `"follow"`: 기본값. HTTP-redirects를 따름
- `"error"`: HTTP-redirect 경우에 오류
- `"manual"`: HTTP 리디렉션을 따르지 않지만 `response.url`이 새 URL이 되고, `response.redirected`가 `true`가 되어 새 URL로 수동으로 리디렉션을 수행할 수 있음 (필요한 경우)

### integrity

- `integrity` 옵션: 응답이 미리 알려진 checksum과 일치하는지 확인
- 사양에 설명된 대로 지원되는 해시 함수는 SHA-256, SHA-284 및 SHA-512이며 브라우저에 따라 다른 함수가 있을 수 있음
- 예를 들어, 파일을 다운로드 하는데 SHA-256 체크섬이 "abcdef"인 것을 알고 있는 경우 (실제 체크섬은 더 긺)

```javascript
fetch("http://site.com/file", { integrity: "sha256-abcdef" });
```

- 그러면 `fetch`는 자체적으로 SHA-256을 계산하고 이를 문자열과 비교. 일치하지 않는 경우 에러 발생

### keepalive

- `keepalive` 옵션: 요청이 시작된 웹 페이지보다 오래 지속될 수 있음(outlive)을 나타냄
- 예를 들어, 현재 방문자가 페이지를 어떻게 사용하는지에 대한 통계를 수집해 사용자 경험을 분석하고 개선할 수 있음
  - 마우스 클릭, 조회한 페이지 조각 등
- 방문자가 페이지를 떠날 때, 서버에 데이터를 저장하고 싶으면 `window.onunload` 이벤트를 사용할 수 있음

```javascript
window.onunload = function () {
  fetch("/analytics", {
    method: "POST",
    body: "statistics",
    keepalive: true
  });
};
```

- 일반적으로 문서가 언로드되면 관련된 모든 네트워크 요청은 중단됨
- 그러나 `keepalive` 옵션은 브라우저가 페이지를 떠난 후에도 백그라운드에서 요청을 수행하도록 지시
- 따라서 이 옵션은 요청이 성공하는 데 필수적

여기에는 몇 가지 제한사항이 있음

- 메가바이트 전송 불가. `keepalive` 요청 본문 제한은 `64kb`
  - 방문에 대한 많은 통계를 수집해야 하는 경우 정기적으로 패킷을 보내 마지막 `onunload` 요청에 대한 공간이 많이 남지 않도록 해야 함
  - 이 제한은 모든 `keepalive` 요청에 함께 적용됨
  - 즉, 여러 개의 `keepalive` 요청을 병렬로 수행할 수 있지만, 본문 길이의 합이 `64kb`를 초과해서는 안 됨
- 문서가 언로드되면 서버 응답을 처리할 수 없음
  - 위 예시에서 `fetch`는 `keepalive` 때문에 성공하지만 후속 기능은 동작하지 않음
  - 통계 전송과 같은 대부분의 경우 서버는 데이터를 수락하고 일반적으로 그러한 요청에 대해 빈 응답을 보내기 때문에 문제가 되지 않음

## URL objects

내장 `URL` 객체

- URL 생성 및 구분 분석을 위한 편리한 인터페이스를 제공
- 정확히 `URL` 객체를 요구하는 네트워킹 메서드는 없으며 문자열이면 충분
- 따라서 기술적으로는 `URL`을 사용할 필요가 없지만 때로는 정말 도움이 될 수 있음

### Creating a URL

```javascript
new URL(url, [base]);
```

- 새 `URL` 객체를 생성하는 구문
- `url`: 전체 URL 또는 유일한 경로(기본이 설정된 경우 아래 참조)
- `base`: 선택적 기본 URL. 설정되어 있고 `url` 인수에 경로만 있는 경우 URL은 기본을 기준으로 생성됨
- 기존 URL과 관련된 경로를 기반으로 새 URL 생성 가능
- URL 객체를 사용해 해당 구성 요소에 즉시 접근. 쉽게 URL 구문 분석 가능

```javascript
// 동일한 두 URL
let url1 = new URL("https://javascript.info/profile/admin");
let url2 = new URL("/profile/admin", "https://javascript.info");
alert(url1); // https://javascript.info/profile/admin
alert(url2); // https://javascript.info/profile/admin

// 기존 URL과 관련된 경로를 기반으로 새 URL 생성
let url = new URL("https://javascript.info/profile/admin");
let newUrl = new URL("tester", url);
alert(newUrl); // https://javascript.info/profile/tester

// URL 객체를 사용해 해당 구성 요소에 즉시 접근
let url = new URL("https://javascript.info/url");
alert(url.protocol); // https:
alert(url.host); // javascript.info
alert(url.pathname); // /url
```

```
-------------------------------href--------------------------------
------------origin------------
               -----host------
protocol       hostname   port   pathname      search         hash
 https:   //   site.com : 8080  /path/page  ?p1=v1&p2=v2...   #hash
```

- `href`: 전체 URL로 `url.toString()`과 같음
- `protocol`: `:`로 끝남
- `search`: `?`로 시작하는 매개변수 문자열
- `hash`: `#`로 시작
- HTTP 인증이 존재하는 경우 `user`와 `password` 프로퍼티가 있을 수 있음
  - `http://login:password@site.com`. 거의 사용되지 않음

`URL` 객체를 문자열 대신 네트워킹(및 대부분의 다른) 메서드에 전달할 수 있음

- `URL` 객체를 `fetch` 혹은 `XMLHttpRequest`나 URL 문자열이 예상되는 거의 모든 곳에서 사용할 수 있음
- 일반적으로 `URL` 객체는 문자열 대신 모든 메서드에 전달될 수 있음
- 대부분의 메서드는 `URL` 객체를 전체 URL이 포함된 문자열로 변환을 수행하기 때문

### SearchParams "?..."

주어진 검색 매개변수를 사용해 URL을 생성한다고 가정

- 매개변수에 공백, 라틴 문자가 아닌 문자 등이 포함된 경우 매개변수를 인코딩해야 함

```javascript
new URL("https://google.com/search?query=JavaScript");
```

- 이에 대한 URL 프로퍼티, URLSearchParams 타입의 객체 `url.searchParams`가 있음
- 검색 매개변수에 대한 편리한 방법을 제공
- `append(name, value)`: `name` 매개변수를 추가
- `delete(name)`: `name` 매개변수를 제거
- `get(name)`: `name` 매개변수를 얻음
- `getAll(name)`: 동일한 `name` 매개변수를 모두 가져옴
- `has(name)`: `name` 매개변수가 있는지 확인
- `set(name, value)`: 매개변수를 설정하거나 대체
- `sort()`: 이름별로 매개변수를 정렬. 거의 필요하지 않음
- `Map`과 유사하게 반복 가능

```javascript
let url = new URL("https://google.com/search");
url.searchParams.set("q", "test me!"); // 공백, !를 포함한 매개변수 추가
alert(url); // https://google.com/search?q=test+me%21
url.searchParams.set("tbs", "qdr:y"); // :을 포함한 매개변수 추가
alert(url); // https://google.com/search?q=test+me%21&tbs=qdr%3Ay (매개변수는 자동으로 인코딩됨)
// iterate over search parameters (decoded)
for (let [name, value] of url.searchParams) alert(`${name}=${value}`); // q=test me!, tbs=qdr:y
```

### Encoding

...

## XMLHttpRequest

...

## 참고

> [네트워크 요청](https://ko.javascript.info/network)
