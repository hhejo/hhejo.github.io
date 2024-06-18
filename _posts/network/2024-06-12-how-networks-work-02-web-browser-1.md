---
title: 1%의 네트워크 원리 (02) - 웹 브라우저가 메시지를 생성 1
date: 2024-06-12 07:12:30 +0900
last_modified_at: 2024-06-17 12:36:29 +0900
categories: [Network]
tags: [network]
---

HTTP 리퀘스트 메시지 작성

# 1%의 네트워크 원리 (02) - 웹 브라우저가 메시지를 생성 1

1. HTTP 리퀘스트 메시지 작성
   - 사용자가 브라우저에 URL 입력
   - 브라우저가 URL 해석
   - 리퀘스트 메시지 생성(HTTP 프로토콜)
2. 웹 서버의 IP 주소를 DNS 서버에 조회
   - OS에 의뢰해 웹 서버에 송신
   - 상대의 IP 주소 필요
   - URL 안의 웹 서버의 도메인명을 DNS 서버에 조회 - IP 주소 조사
3. 전 세계의 DNS 서버가 연대
   - DNS 서버가 IP 주소를 조사
   - 전 세계의 수만 대의 DNS 서버들이 연대
4. 프로토콜 스택에 메시지 송신을 의뢰
   - 메시지를 웹 서버에 송신하도록 OS에 의뢰

## 01 HTTP 리퀘스트 메시지를 작성

### 1 탐험 여행은 URL 입력부터 시작

URL

- Uniform Resource Locator
- `http://`, `file:`, `mailto:`, `ftp:`, ...

HTTP

- HyperText Transfer Protocol
- 프로토콜: 통신 동작의 규칙을 정한 것

FTP

- File Transfer Protocol

도메인명

- `www.cyber.co.kr`
- 마침표로 구분하여 표현하는 이름

### 2 브라우저는 먼저 URL을 해독

URL의 요소

```
http://웹서버명/디렉토리명/.../파일명
```

- `http:`: 프로토콜 - 데이터 출처에 엑세스하는 방법
- `//`: 나중에 이어지는 문자열이 서버의 이름임을 나타냄
- `웹서버명`
- (이 아래는 생략 가능)
- `/`
- `디렉토리명`
- `/`
- `파일명`

```
http://www.lab.cyber.co.kr/dir1/file1.html
```

### 3 파일명을 생략한 경우

대부분의 서버가 `index.html` 또는 `default.htm`이라는 파일명 설정

```
http://www.lab.cyber.co.kr/dir/
http://www.lab.cyber.co.kr/
http://www.lab.cyber.co.kr
http://www.lab.cyber.co.kr/whatisthis
```

1. 루트 디렉토리(`/`) 아래 `dir/` 디렉토리
2. 루트 디렉토리(`/`)
3. 루트 디렉토리가 생략됨
4. `whatisthis`라는 파일 혹은 디렉토리

### 4 HTTP의 기본 개념

리퀘스트 메시지

- 무엇을: URI
- 어떻게 해서: Method

응답 메시지

- ...스테이터스

URI

- Uniform Resource Identifier

HTTP 버전: `1.0`, `1.1`

메서드

- GET
  - 많아야 수백 바이트 전송
- POST
- HEAD
- OPTIONS
- PUT
- DELETE
- TRACE
- CONNECT

### 5 HTTP 리퀘스트 메시지 생성

- 브라우저는 URL을 해독
- 웹 서버, 파일명을 판단
- 이것을 바탕으로 HTTP 리퀘스트 메시지 생성

리퀘스트 메시지

1. 리퀘스트 라인(1줄): `메서드 공백 URI 공백 HTTP버전`
2. 메시지 헤더 + 공백 1줄
3. 메시지 본문

응답 메시지

1. 스테이터스 라인(1줄): `HTTP버전 공백 스테이터스코드 공백 응답문구`
2. 메시지 헤더 + 공백 1줄
3. 메시지 본문(바이너리 데이터)

헤더의 종류

- 제너럴 헤더
- 리퀘스트 헤더
- 응답 헤더
- 엔티티 헤더

### 6 리퀘스트 메시지를 보내면 응답이 되돌아옴

스테이터스 코드

- 1XX: 처리의 경과 상황 등을 통지
- 2XX: 정상 종료
- 3XX: 무언가 다른 조치 필요
- 4XX: 클라이언트측 오류
- 5XX: 서버측 오류

한 개의 리퀘스트에 한 개의 응답

## 참고

성공과 실패를 결정하는 1%의 네트워크 원리
