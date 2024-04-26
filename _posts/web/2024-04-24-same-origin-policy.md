---
title: Same-Origin Policy에 대하여
date: 2024-04-24 12:09:45 +0900
last_modified_at: 2024-04-26 12:09:45 +0900
categories: [Web]
tags: [web, same-origin-policy]
---

동일 출처 정책이란 정확히 무엇일까요?

## 동일 출처 정책 (Same-origin policy)

한 출처(origin)에서 로드한 문서 또는 스크립트가 다른 출처의 리소스와 상호작용하는 방식을 제한하는 중요한 보안 메커니즘

잠재적 악성 문서를 격리하여 가능한 공격 경로를 줄이는 데 도움이 됨

- 인터넷의 악성 웹사이트가 브라우저에서 JS를 실행하여, 사용자가 로그인한 타사 웹메일 서비스 또는 회사 인트라넷에서 데이터를 읽고 해당 데이터를 공격자에게 전달하는 것을 방지

## Origin의 정의

프로토콜, 포트, 호스트가 모두 동일한 경우 두 URL의 오리진이 동일

- scheme/host/port tuple 혹은 tuple이라고도 함

### 상속된 오리진

`about:blank` 또는 `javascript:` URL이 있는 페이지에서 실행되는 스크립트

- 해당 URL이 포함된 문서의 출처를 상속함
- 이러한 유형의 URL에는 출처 서버에 대한 정보가 포함되어 있지 않기 때문
- `about:blank`는 부모 스크립트가 콘텐츠를 작성하는 새 빈 팝업 창의 URL로 자주 사용됨
  - `Window.open()` 메커니즘을 통해
- 이 팝업에는 자바스크립트도 포함되어 있으면, 해당 스크립트는 팝업을 만든 스크립트와 동일한 오리진을 상속받음

`data:` URL

- 새롭고 비어있는 보안 컨텍스트를 가져옴

### 파일 오리진

`file:///` 스키마

- 모던 브라우저는 `file:///` 스키마를 사용하여 로드된 파일의 오리진을 불투명한 오리진으로 처리
- 즉 파일에 같은 폴더에 있는 다른 파일이 포함되어 있으면 동일한 오리진에서 온 것으로 간주하지 않고 CORS 오류를 트리거할 수 있음

## 교차 출처 네트워크 접근 (Cross-origin network access)

동일 출처 정책

- `XMLHttpRequest`나 `<img>` 요소를 사용하는 경우와 같이 서로 다른 두 출처 간의 상호작용을 제어
- 이러한 상호작용은 일반적으로 세 가지 범주로 분류됨
- 교차 출처 쓰기는 보통 허용함
  - 링크, 리다이렉트, 양식 제출 등 (일부 HTTP 요청은 preflight를 요구)
- 교차 출처 삽입은 보통 허용함
- 교차 출처 읽기는 보통 허용하지 않음
  - 하지만 종종 교차 출처 삽입 과정에서 읽기 권한이 누출됨
  - 삽입된 이미지의 크기, 삽입된 스크립트의 작업 또는 삽입된 리소스의 가용성을 읽을 수 있음

교차 출처로 삽입할 수 있는 리소스의 일부 예시

- `<script src="..."></script>`를 사용하는 자바스크립트
  - 구문 오류에 대한 세부 정보는 동일 출처 스크립트에서만 사용 가능
- `<link rel="stylesheet" href="...">`로 적용된 CSS
  - CSS의 완화된 구문 규칙으로 인해 교차 출처 CSS에는 올바른 `Content-Type` 헤더가 요구됨
  - 브라우저는 MIME 유형이 올바르지 않고, 리소스가 유효한 CSS 구성으로 시작하지 않는 교차 출처 로드인 경우 스타일시트 로드를 차단
- `<img>`로 표시하는 이미지
- `<video>`와 `<audio>`로 재생하는 미디어
- `<object>`와 `<embed>`로 삽입하는 외부 리소스
- `@font-face`로 적용하는 글꼴
  - 일부 브라우저는 교차 출처를 허용하지만 다른 브라우저는 동일 출처를 요구할 수도 있음
- `<iframe>`으로 삽입하는 모든 것
  - 사이트는 `X-Frame-Options` 헤더를 사용하여 출처 간 프레이밍 방지 가능

### 교차 출처 접근을 허용하는 방법

CORS를 사용하여 교차 출처의 접근을 허용

- CORS는 서버가 브라우저에 콘텐츠 로드를 허용하는 다른 호스트를 지정할 수 있도록 하는 HTTP의 일부

### 교차 출처 접근을 막는 방법

교차 출처 쓰기 방지

- CSRF(Cross-Site Request Forgery) 토큰이라고 요청에서 추츨할 수 없는 토큰을 확인해야 함
- 이 토큰이 필요한 페이지의 교차 출처 읽기를 막아야 함

교차 출처 읽기 방지

- 리소스를 삽입할 수 없도록 설정해야 함
- 리소스 삽입 과정에서 일부 정보가 새어 나가므로 삽입을 방지하는 경우가 많음

교차 출처 삽입 방지

- 리소스가 앞서 나열된 삽입 가능 형식 중 하나로 해석되지 않도록 해야 함
- 브라우저는 `Content-Type` 헤더를 준수하지 않을 수 있음

## 교차 출처 스크립트 API 접근

특정 자바스크립트 API를 사용하면 문서가 서로를 직접 참조 가능

- `iframe.contentWindow`
- `window.parent`
- `window.open`
- `window.opener`
- 두 문서의 원본이 동일하지 않은 경우 이러한 참조는 `Window`와 `Location` 객체에 대해 매우 제한된 액세스를 제공

서로 다른 출처의 문서 간 통신

- `window.postMessage`

## 교차 출처 데이터 저장소 접근

Web Storage, IndexedDB

- 브라우저에 저장된 데이터에 대한 액세스는 출처별로 구분됨
- 각 출처는 별도의 저장소를 가지며, 한 출처의 자바스크립트는 다른 출처에 속한 저장소에서 읽거나 쓸 수 없음

쿠키

- 출처에 대한 별도의 정의를 사용
- 페이지는 상위 도메인이 공개 접미사가 아닌 한 자체 도메인 또는 상위 도메인에 대한 쿠키 설정 가능
- 쿠키를 설정할 때 `Domain`, `Path`, `Secure`, `HttpOnly` 플래그를 사용해 가용성 제한 가능
- 쿠키를 읽을 때 쿠키가 어디에서 설정되었는지 알 수 없음
- 안전한 https 연결만 사용하더라도 보이는 모든 쿠키는 안전하지 않은 연결을 사용하여 설정되었을 수 있음

## 참고

> [Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)

> [SOP(Same-origin policy) 란 무엇일까?](https://dongwooklee96.github.io/post/2021/03/23/sopsame-origin-policy-란-무엇일까.html)

> [Same-Origin Policy(동일 출처 정책)은 무엇인가?](https://codingmoon.io/posts/interview-questions/describe-same-origin-policy/)

> [CORS가 대체 무엇일까? (feat. SOP)](https://hudi.blog/sop-and-cors/)
