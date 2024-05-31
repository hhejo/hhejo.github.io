---
title: 전형적인 HTTP 세션
date: 2024-05-25 13:11:46 +0900
last_modified_at: 2024-05-31 08:33:46 +0900
categories: [Web]
tags: [web, http, session]
---

전형적인 HTTP 세션에 대해 알아보겠습니다

## 전형적인 HTTP 세션

HTTP와 같은 클라이언트-서버 프로토콜에서 세션의 과정

1. 클라이언트가 TCP 연결 수립(또는 전송 계층이 TCP가 아닌 다른 적당한 연결로)
2. 클라이언트는 요청을 전송하고 응답을 기다림
3. 서버는 요청에 대해 처리하고 응답을 상태 코드, 요청에 부합하는 데이터와 함께 돌려보냄

HTTP/1.1부터는 세번째 과정 이후 클라이언트가 해당 시점에 또 다른 요청을 보낼 수 있도록 연결을 닫지 않음

- 따라서 두번째, 세번째 과정이 몇 번에 걸쳐 일어날 수 있음

### 연결 수립

클라이언트는 연결을 수립

- HTTP에서 연결을 여는 것은 보통의 경우 TCP인 기본적인 전송계층 내에서 연결을 수립하는 것을 의미
- TCP 이용 시 컴퓨터 상의 HTTP 서버를 위한 기본 포트는 80(8000, 8080)
- 요청을 위한 페이지 URL은 도메인 이름과 포트 번호 둘 다 포함
- 포트 번호가 80인 경우 생략 가능

클라이언트-서버 모델

- 서버로 하여금 명시적인 요청 없이 클라이언트로 데이터를 전송하는 것을 허용하지 않음
- `XMLHttpRequest`, `Fetch` API를 통해 주기적으로 서버에 핑하기
- HTML 웹소켓 API
- 혹은 그와 유사한 프로토콜을 사용해 이런 문제에 대한 해결책으로 사용

### 클라이언트 요청 전송

사용자-에이전트

- 연결이 한번 수립된 경우 요청 전송 가능
- 사용자-에이전트는 일반적으로 웹 브라우저를 의미
- 크롤러와 같이 무엇이든 될 수 있음

클라이언트 요청

0. 세 가지 블록으로 나누어진 CRLF로 구분된 텍스트 지시자들로 이루어짐
1. 파라미터가 따르는 요청 메서드 포함
   - 문서의 경로(프로토콜, 도메인 이름을 제외한 절대 URL)
   - 사용중인 HTTP 프로토콜 버전
2. 각각 특정 헤더를 나타냄
3. 부가적인 데이터 블록
   - 주로 POST 메서드에 의해 사용

요청 예제

```
GET / HTTP/1.1
Host: developer.mozilla.org
Accept-Language: fr
```

- `http://developer.mozilla.org/`의 최상위 페이지 가져오도록 요청
- 서버에게 사용자-에이전트가 해당 페이지에 대해 프랑스어 페이지를 원한다고 알림
- `Content-Length` 헤더가 없으므로 데이터 블록은 비어있고 서버는 헤더의 마지막을 나타내는 빈 줄을 받는 즉시 요청 처리 가능

```
POST /contact_form.php HTTP/1.1
Host: developer.mozilla.org
Content-Length: 64
Content-Type: application/x-www-form-urlencoded

name=Joe%20User&request=Send%20me%20one%20of%20your%20catalogue
```

요청 메서드

- HTTP는 주어진 자원에 대해 실행되길 바라는 동작을 가리키는 요청 메서드 집합을 정의
- `GET`: 지정된 자원의 표시 요청. 데이터를 가져오는 것 외에는 아무것도 할 수 없음
- `POST`: 서버에 데이터를 전송해 서버가 상태를 바꾸도록 만듦

### 서버 응답의 구조

연결된 에이전트가 자신의 요청을 전송하고 난 뒤 웹서버가 그것을 처리하고 최종적으로 응답을 돌려보냄

서버 응답

0. 세 개의 다른 블록으로 나누어진 CRLF로 구분된 텍스트 지시자들로 이루어짐
1. 상태 줄: 상태 요청이 따르도록 사용된 HTTP 버전의 acknowledegment로 구성
2. 각각 특정 헤더를 나타냄
3. 데이터 블록(존재한다면)

응답 예제

```
HTTP/1.1 200 OK
Date: Sat, 09 Oct 2010 14:28:02 GMT
Server: Apache
Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
ETag: "51142bc1-7449-479b075b2891b"
Accept-Ranges: bytes
Content-Length: 29769
Content-Type: text/html

<!DOCTYPE html... (here comes the 29769 bytes of the requested web page)
```

- 웹페이지의 성공적인 수신

```
HTTP/1.1 301 Moved Permanently
Server: Apache/2.2.3 (Red Hat)
Content-Type: text/html; charset=iso-8859-1
Date: Sat, 09 Oct 2010 14:30:24 GMT
Location: https://developer.mozilla.org/ (this is the new link to the resource; it is expected that the user-agent will fetch it)
Keep-Alive: timeout=15, max=98
Accept-Ranges: bytes
Via: Moz-Cache-zlb05
Connection: Keep-Alive
X-Cache-Info: caching
X-Cache-Info: caching
Content-Length: 325 (the content contains a default page to display if the user-agent is not able to follow the link)

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://developer.mozilla.org/">here</a>.</p>
<hr>
<address>Apache/2.2.3 (Red Hat) Server at developer.mozilla.org Port 80</address>
</body></html>
```

- 요청 자원이 영구적으로 옮겨짐

```
HTTP/1.1 404 Not Found
Date: Sat, 09 Oct 2010 14:33:02 GMT
Server: Apache
Last-Modified: Tue, 01 May 2007 14:24:39 GMT
ETag: "499fd34e-29ec-42f695ca96761;48fe7523cfcc1"
Accept-Ranges: bytes
Content-Length: 10732
Content-Type: text/html

<!DOCTYPE html... (contains a site-customized page helping the user to find the missing resource)
```

- 요청된 자원이 존재하지 않음

HTTP 응답 상태 코드

- 특정 HTTP 요청이 성공적으로 끝났는지 아닌지 알림
- 다섯가지 계층 내로 그룹화
  - 정보를 제공하는 응답
  - 성공적인 응답
  - 리다이렉트
  - 클라이언트 오류
  - 서버 오류

## 참고

[전형적인 HTTP 세션](https://developer.mozilla.org/ko/docs/Web/HTTP/Session)
