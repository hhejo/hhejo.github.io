---
title: CORS에 대해 자세하게 알아보기!
date: 2024-04-26 15:50:11 +0900
last_modified_at: 2024-04-26 15:50:11 +0900
categories: [Web]
tags: [web, cors]
---

CORS 총정리

## CORS 없는 교차 출처 접근 (Cross-origin access without CORS)

```html
<script src="…"></script>
<link rel="stylesheet" href="…" />
<iframe src="…"></iframe>
<video src="…"></video>
<audio src="…"></audio>
```

- 이와 같은 API를 사용하면 다른 웹사이트에 요청을 하고 상대 사이트의 동이 없이도 특정 방식으로 응답을 처리할 수 있음
- 이는 1994년 HTTP 쿠키의 등장으로 복잡해짐
- HTTP 쿠키는 자격 증명(credentials)이라고 부르는 집합의 일부가 되었음
- 여기에는 TLS 클라이언트 인증서와 HTTP 인증 사용 시 `Authorization` 요청 헤더에 자동으로 들어가는 상태도 포함됨

Credentials (자격 증명)

- 자격 증명을 사용하면 서버가 여러 요청에 걸쳐 특정 사용자에 대한 상태를 유지할 수 있음
- 트위터에서 내 피드 표시
- 은행에서 내 계좌를 표시하는 등의 방식이 바로 이 인증
- 위의 방법 중 하나를 사용하여 다른 사이트의 콘텐츠를 요청하면 다른 사이트의 자격 증명도 함께 전송됨
- 지난 몇 년 동안 이로 인해 엄청난 보안 문제가 발생함

## 교차 출처 리소스 공유 (Cross-Origin Resource Sharing)

추가 HTTP 헤더를 사용하여, 한 출처에서 실행중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제

웹 애플리케이션은 리소스가 자신의 출처(도메인, 프로토콜, 포트)와 다를 때 교차 출처 HTTP 요청을 실행

교차 출처 요청의 예시

- `https://domain-a.com`의 프론트엔드 자바스크립트 코드가 `XMLHttpRequest`를 사용하여 `https://domain-b.com/data.json`을 요청

보안 상의 이유로 브라우저는 스크립트에서 시작한 교차 출처 HTTP 요청을 제한

- `XMLHttpRequest`와 Fetch API는 동일 출처 정책을 따름
- 즉 이 API를 사용하는 웹 애플리케이션은 자신의 출처와 동일한 리소스만 불러올 수 있음
- 다른 출처의 리소스를 불러오려면 그 출처에서 올바른 CORS 헤더를 포함한 응답을 반환해야 함

CORS 체제는 브라우저와 서버 간의 안전한 교차 출처 요청 및 데이터 전송을 지원

최신 브라우저는 `XMLHttpRequest` 또는 Fetch와 같은 API에서 CORS를 사용하여 교차 출처 HTTP 요청의 위험을 완화

## CORS를 사용하는 요청

교차 출처 공유 표준은 다음과 같은 경우에 사이트 간 HTTP 요청을 허용

- `XMLHttpRequest`와 Fetch API의 호출
- 웹 폰트(CSS 내 `@font-face`에서 교차 도메인 폰트 사용 시)
  - 서버가 허용된 웹사이트에서만 교차 사이트 로드 및 사용할 수 있는 트루타입 글꼴을 배포할 수 있도록 함
- WebGL 텍스쳐
- `drawImage()`를 사용해 캔버스에 그린 이미지/비디오 프레임
- 이미지로부터 추출하는 CSS Shapes

### 기능적 개요

교차 출처 리소스 공유 표준

- 웹브라우저에서 해당 정보를 읽는 것이 허용된 출처를 서버에서 설명할 수 있는 새로운 HTTP 헤더를 추가함으로써 동작

추가적으로 서버 데이터에 부수 효과(side effect)를 일으킬 수 있는 HTTP 요청 메서드에 대해, CORS 명세는 브라우저가 요청을 `OPTIONS` 메서드로 프리플라이트(preflight, 사전 전달)하여 지원하는 메서드를 요청하고, 서버의 허가가 떨어지면 실제 요청을 보내도록 요구하고 있음

또한 서버는 클라이언트에게 요청에 인증정보(쿠키, HTTP 인증)를 함께 보내야 한다고 알려줄 수도 있음

CORS 실패는 오류의 원인

- 보안상의 이유로 자바스크립트에서는 오류의 상세 정보에 접근할 수 없음
- 알 수 있는 것은 오류가 발생했다는 사실 뿐
- 정확히 어떤 것이 실패했는지 알아내려면 브라우저의 콘솔을 봐야 함

## 접근 제어 시나리오 예제

교차 출처 리소스 공유가 동작하는 방식을 보여주는 세 가지 시나리오

- 모든 예제는 지원하는 브라우저에서 교차 출처 요청을 생성할 수 있는 `XMLHttpRequest`를 사용

### 단순 요청 (Simple requests)

CORS preflight를 트리거하지 않는, 다음 조건을 모두 충족하는 요청

- `Fetch` 명세(CORS를 정의한)는 이 용어를 사용하지는 않음
- 다음 중 하나의 메서드
- 유저 에이전트가 자동으로 설정한 헤더(예를 들어 `Connection`, `User-Agent`, Fetch 명세에서 "forbidden header name"으로 정의한 헤더) 외에, 수동으로 설정할 수 있는 헤더는 오직 Fetch 명세에서 "CORS-safelisted request-header"로 정의한 헤더뿐임
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type` (아래의 추가 요구 사항에 유의)
- `Content-Type` 헤더는 다음의 값들만 허용
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `text/plain`
- 요청에 사용된 `XMLHttpRequestUpload` 객체에는 이벤트 리스너가 등록되어 있지 않음
  - 이들은 `XMLRequest.upload` 프로퍼티를 사용하여 접근
- 요청에 `ReadableStream` 객체가 사용되지 않음

요청 헤더의 `Origin`을 보면, `https://foo.example`로부터 요청이 왔다는 것을 알 수 있음

`Origin: https://foo.example`

서버는 이에 대한 응답으로 `Access-Control-Allow-Origin` 헤더를 전송

가장 간단한 접근 제어 프로토콜은 `Origin` 헤더와 `Access-Control-Allow-Origin`을 사용하는 것

`Access-Control-Allow-Origin: *`

- 모든 도메인에서 접근 가능

`Access-Control-Allow-Origin: https://foo.example`

- 오직 `https://foo.example`의 요청만 리소스에 대한 접근 허용

### 프리플라이트 요청 (preflighted requests)

먼저 `OPTIONS` 메서드를 통해 다른 도메인의 리소스로 HTTP 요청을 보내 실제 요청이 전송하기에 안전한지 확인함

cross-origin 요청은 유저 데이터에 영향을 줄 수 있기 때문에 이와 같이 미리 전송(preflighted)함

프리플라이트 요청

- `Origin`
- `Access-Control-Request-Method`
- `Access-Control-Request-Headers`

응답

- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Headers`
- `Access-Control-Max-Age`

preflight request가 완료되면 실제 요청을 전송

### 자격 증명을 포함한 요청

기본적으로 cross-site `XMLHttpRequest`나 Fetch 호출에서 브라우저는 자격 증명을 보내지 않음

브라우저는 `Access-Control-Allow-Credentials: true` 헤더가 없는 응답을 거부

자격 증명 요청에 응답할 때 서버는 반드시 `*` 와일드카드를 지정하는 대신, `Access-Control-Allow-Origin` 헤더 값에 출처를 지정해야 함

## HTTP 응답 헤더

### Access-Control-Allow-Origin

```
Access-Control-Allow-Origin: <origin> | *
Access-Control-Allow-Origin: *
Access-Control-Allow-Origin: https://mozilla.org
```

- 단일 출처를 지정하여 브라우저가 해당 출처가 리소스에 접근하도록 허용
- `*` 와일드카드는 브라우저의 origin에 상관 없이 모든 리소스에 접근하도록 허용(자격 증명이 없는 요청)
- 서버가 와일드카드 대신에 하나의 origin을 지정하는 경우, 서버는 `Vary` 응답 헤더에 `Origin`을 포함해야 함 (`Vary: Origin`)
  - 서버 응답이 `Origin` 요청 헤더에 따라 다르다는 것을 클라이언트에 알려줌

### Access-Control-Expose-Headers

```
Access-Control-Expose-Headers: <header-name>[, <header-name>]*
Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
```

- 브라우저가 접근할 수 있는 헤더를 서버의 화이트리스트에 추가할 수 있음
- 위 예시에서 `X-My-Custom-Header`와 `X-Another-Custom-Header` 헤더가 브라우저에 드러남

### Access-Control-Max-Age

```
Access-Control-Max-Age: <delta-seconds>
```

- preflight request 결과를 캐시할 수 있는 시간
- `delta-seconds`: 결과를 캐시할 수 있는 시간(초)

### Access-Control-Allow-Credentials

```
Access-Control-Allow-Credentials: true
```

- `credentials` 플래그가 true일 때 요청에 대한 응답을 표시할 수 있는지를 나타냄
- simple `GET` requests는 preflighted되지 않으므로 credentials 있는 리소스를 요청하면, 이 헤더가 리소스와 함께 반환되지 않음
- 이 헤더가 없으면 브라우저에서 응답을 무시하고 웹 컨텐츠로 반환되지 않음

### Access-Control-Allow-Methods

```
Access-Control-Allow-Methods: <method>[, <method>]*
```

- 리소스에 접근할 때 허용되는 메서드를 지정
- preflight request에 대한 응답으로 사용됨

### Access-Control-Allow-Headers

```
Access-Control-Allow-Headers: <header-name>[, <header-name>]*
```

- preflight request에 대한 응답으로 사용됨
- 실제 요청시 사용할 수 있는 HTTP 헤더를 나타냄

## HTTP 요청 헤더

### Origin

```
Origin: <origin>
```

- 요청이 시작된 서버를 나타내는 URI
- 경로 정보는 포함하지 않고, 오직 서버 이름만 포함
- `null`도 올 수 있음
- 접근 제어 요청에는 항상 `Origin` 헤더가 전송됨

### Access-Control-Request-Method

```
Access-Control-Request-Method: <method>
```

- 실제 요청에서 어떤 HTTP 메서드를 사용할지 서버에 알려주기 위해 preflight request할 때 사용

### Access-Control-Request-Headers

```
Access-Control-Request-Headers: <field-name>[, <field-name>]*
```

- 실제 요청에서 어떤 HTTP 헤더를 사용할지 서버에 알려주기 위해 preflight request할 때 사용

## 정리

COR (Cross-Origin Request, 교차 출처 요청)

- 도메인이나 서브도메인, 프로토콜, 포트가 다른 곳에 요청을 보내는 것
- 리모트 오리진에서 전송받은 특별한 헤더가 있어야 요청 전송 가능

CORS (Cross-Origin Resource Sharing, 교차 출처 리소스 공유)

- 이러한 정책
- 추가 HTTP 헤더를 사용하여, 한 출처에서 실행 중인 웹 앱이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제

Cross-Origin Request 구분

- 안전한 요청
- 그 외의 요청

안전한 요청

- 안전한 메서드 `GET`, `POST`, `HEAD`
- 안전한 헤더
  - `Accept`, `Accept-Language`, `Content-Language`
  - `Content-Type`이고 값이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`

CORS와 안전한 요청

- 요청 `Origin` 헤더에 오리진 정보가 담김
- 응답 `Access-Control-Allow-Origin` 헤더에 허가된 오리진 정보나 와일드카드 `*` 명시

안전한 응답 헤더

- 교차 출처 요청이 이뤄진 경우 자바스크립트는 기본적으로 안전한 응답 헤더로 분류되는 헤더에만 접근 가능
- `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, `Pragma`
- 이외의 응답 헤더에 접근하면 에러 발생

안전하지 않은 응답 헤더에 접근하기

- 자바스크립트로 안전하지 않은 응답 헤더에 접근하려면 `Access-Control-Expose-Headers` 헤더 사용
- `Access-Control-Expose-Headers`에 접근할 헤더 목록 명시

안전하지 않은 요청

- preflight 요청이 먼저 시도됨
- `OPTIONS` 메서드
- `Access-Control-Request-Method` 헤더: 안전하지 않은 요청에서 사용하는 메서드 정보 담김
- `Access-Control-Request-Headers` 헤더: 안전하지 않은 요청에서 사용하는 헤더 목록 담김

안전하지 않은 요청을 허용하는 경우

- `Access-Control-Allow-Origin`: 와일드카드 `*`이나 요청을 보낸 오리진
- `Access-Control-Allow-Method`: 허용된 메서드 정보 담김
- `Access-Control-Allow-Headers`: 허용된 헤더 목록 담김
- `Access-Control-Max-Age`: 퍼미션 체크 여부를 몇 초간 캐싱해 놓을지를 명시 (86400은 24시간)
- preflight 요청이 성공적으로 이뤄진 후, 안전한 요청이 이뤄질 때의 절차와 동일

자격 증명 (Credentials)

- 자바스크립트르 교차 출처 요청을 보내는 경우, 기본적으로 쿠키나 HTTP Authorization 같은 자격 증명이 함께 전송되지 않음
- HTTP 요청의 경우 대개 쿠키가 함께 전송되나, 자바스크립트로 만든 교차 출처 요청은 예외
- `fetch` API의 경우 `fetch(url, { credentials: "include" })`로 자격 증명 정보를 함께 전송
- 응답 헤더에는 `Access-Control-Allow-Credentials: true`가 포함되어야 함

## 참고

> [교차 출처 리소스 공유 (CORS)](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)

> [How to win at CORS](https://jakearchibald.com/2021/cors/)

> [The CORS Playground](https://jakearchibald.com/2021/cors/playground/)

> [CORS가 대체 무엇일까? (feat. SOP)](https://hudi.blog/sop-and-cors/)
