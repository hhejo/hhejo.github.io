---
title: HTTP 메시지
date: 2024-05-24 09:28:34 +0900
last_modified_at: 2024-05-31 07:12:09 +0900
categories: [Web]
tags: [web, http, message]
---

HTTP 메시지에 대해 알아보겠습니다.

## HTTP 메시지

HTTP 메시지

- 서버와 클라이언트 간에 데이터가 교환되는 방식
- ACSII로 인코딩된 여러 줄의 텍스트 정보

메시지 타입

- 요청(request): 클라이언트가 서버로 전달해서 서버의 액션이 일어나게 함
- 응답(response): 요청에 대한 서버의 답변

HTTP 요청과 응답의 구조

1. 시작 줄(start-line): 항상 한 줄. 실행되어야 할 요청 또는 요청 수행에 대한 성공 혹은 실패가 기록됨
2. HTTP 헤더: 옵션으로 들어감. 요청에 대한 설명 혹은 메시지 본문에 대한 설명
3. 빈 줄(blank line): 요청에 대한 모든 메타 정보가 전송되었음을 알림
4. 요청과 관련된 내용 혹은 응답과 관련된 문서. 본문의 존재 유무 및 크기는 첫 줄과 HTTP 헤더에 명시됨

### HTTP 요청

시작 줄

1. HTTP 메서드
   - 서버가 수행해야 할 동작 (GET, POST, PUT, HEAD, OPTIONS 등)
2. 요청 타겟
   - URL 또는 프로토콜, 포트, 도메인의 절대 경로
   - 요청 컨텍스트에 의해 특정지어짐
   - HTTP 메서드에 따라 요청 타겟 포맷이 달라짐
3. HTTP 버전
   - 응답 메시지에서 써야 할 HTTP 버전
   - 메시지의 남은 구조를 정의

요청 타겟 포맷

- origin 형식(절대 경로)
  - 끝에 `?`와 쿼리 문자열이 따라옴
  - GET, POST, HEAD, OPTIONS 메서드와 함께 사용
  - `POST / HTTP/1.1`
  - `GET /background.png HTTP/1.0`
  - `HEAD /test.html?query=alibaba HTTP/1.1`
  - `OPTIONS /anypage.html HTTP/1.0`
- absolute 형식(완전한 URL)
  - 프록시에 연결하는 경우 대부분 GET과 함께 사용
  - `GET http://developer.mozilla.org/ko/docs/Web/HTTP/Messages HTTP/1.1`
- authority 형식
  - 도메인 이름 및 옵션 포트(`:`가 앞에 붙음)로 이루어진 URL의 인증 컴포넌트
  - HTTP 터널을 구축하는 경우에만 CONNECT와 함께 사용
  - `CONNECT developer.mozilla.org:80 HTTP/1.1`
- asterisk 형식
  - 별표(`*`) 하나로 서버 전체를 나타냄
  - `OPTIONS * HTTP/1.1`

헤더

- 대소문자 구분 없는 문자열 다음에 콜론(`:`)이 붙으며 그 뒤에 오는 값은 헤더에 따라 달라짐
- 헤더는 값까지 포함해 한 줄
- General 헤더: 메시지 전체에 적용
- Request 헤더: 요청의 내용을 좀 더 구체화, 컨텍스트 제공, 조건에 따른 제약 사항, 요청 내용 수정 등
- Representation 헤더: 메시지 데이터의 원래 형식과 적용된 인코딩 설명(메시지에 본문이 있는 경우에만)

본문

- GET, HEAD, DELETE, OPTIONS처럼 리소스를 가져오는 요청은 보통 본문이 필요 없음
- 일부 요청은 업데이트를 하기 위해 서버에 데이터를 전송(보통 POST 요청)

본문의 2가지 종류

- 헤더 두 개(`Content-Type`, `Content-Length`)로 정의된 단일 파일로 구성되는 단일-리소스 본문(single-resource bodies)
- 각각 서로 다른 정보를 담고 있는 멀티파트 본문으로 구성되는 다중 리소스 본문(보통 HTML 폼과 관련)

### HTTP 응답

상태 줄(status line)

1. 보통 `HTTP/1.1` 프로토콜 버전
2. 요청의 성공 여부를 나타내는 상태 코드
3. 상태 코드에 대한 정보 제공 목적의 짧은 상태 텍스트

```
HTTP/1.1 404 Not Found.
```

헤더

- 대소문자를 구분하지 않는 문자열 다음에 콜론`:`이 오며 그 뒤에 오는 값은 헤더에 따라 다름
- General 헤더: 메시지 전체에 적용
- Response 헤더: 상태 줄에 포함되지 않은 서버에 대한 추가 정보를 제공
- Representation 헤더: 메시지 데이터의 원래 형식과 적용된 인코딩을 설명(메시지에 본문이 있는 경우에만)

본문

- `Content-Type`, `Content-Length` 두 개의 헤더로 길이가 알려진 하나의 파일로 구성된 단일-리소스 본문(single-resource bodies)
- `Transfer-Encoding`이 `chunked`인 청크로 나뉘어 길이를 모르는 하나의 파일로 구성된 단일-리소스 본문
- 서로 다른 정보를 담고 있는 멀티파트 본문으로 이루어진 다중 리소스 본문(상대적으로 보기 힘듦)

### HTTP/2 프레임

HTTP/1.x 메시지의 몇 가지 성능 결함

- 본문은 압축이 되나 헤더는 압축 불가
- 연속된 메시지들은 비슷한 헤더를 가지나 메시지마다 반복되어 전송됨
- 다중전송(multiplexing) 불가
  - 서버 하나에 연결을 여러 개 열어야 하고, 적극적인(warm) TCP 연결이 소극적인(cold) TCP 연결보다 효율적

HTTP/2

- HTTP/1.x 메시지를 프레임으로 나눠 스트림에 끼워 넣음
- 데이터와 헤더 프레임이 분리되었기 때문에 헤더 압축 가능
- 멀티플렉싱: 스트림 여러 개를 하나로 묶을 수 있어 기저에서 수행되는 TCP 연결이 좀 더 효율적

HTTP 프레임

- HTTP/1.1 메시지와 기저가 되는 전송 프로토콜 사이의 HTTP/2에서 추가된 단계
- HTTP 프레임 때문에 개발자들이 API를 바꿔야 할 필요는 없음
- 브라우저와 서버 둘 다 모두 HTTP 프레임을 쓸 수 있다면, HTTP/2가 활성화되고 사용됨

## 참고

[HTTP 메시지](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages)
