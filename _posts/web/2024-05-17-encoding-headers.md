---
title: 세 가지 Encoding 헤더들
date: 2024-05-17 10:04:28 +0900
last_modified_at: 2024-05-20 10:04:28 +0900
categories: [Web]
tags: [web, accept-encoding, content-encoding, transfer-encoding]
---

Accept-Encoding, Content-Encoding, Transfer-Encoding 헤더에 대해 알아보겠습니다.

## Accept-Encoding

`Accept-Encoding` 요청 헤더

- 클라이언트가 이해 가능한 컨텐츠 인코딩을 알림
- 컨텐츠 협상을 사용하여 서버는 제안된 내용 중 하나를 선택하고 사용하며 `Content-Encoding`을 이용해 선택된 것을 클라이언트에 알림

서버가 응답의 본문을 압축하지 않으려 하는 경우

- 전송되어야 할 데이터가 이미 압축되어 있고, 두번째 압축이 전송해야 할 데이터를 더 작게 만들지 못할 경우
  - 이미지 포맷 압축 등
- 서버에 과부하가 걸리고 압축 요구사항에 의해 초래된 연산의 오버헤드를 감당할 수 없는 경우

### 문법

```
Accept-Encoding: gzip
Accept-Encoding: compress
Accept-Encoding: deflate
Accept-Encoding: br
Accept-Encoding: identity
Accept-Encoding: *

// Multiple algorithms, weighted with the quality value syntax:
Accept-Encoding: deflate, gzip;q=1.0, *;q=0.5
```

- `gzip`
- `compress`
- `deflate`
- `identity`
- `*`
- `;q=`

### 예제

```
Accept-Encoding: gzip
Accept-Encoding: gzip, compress, br
Accept-Encoding: br;q=1.0, gzip;q=0.8, *;q=0.1
```

## Content-Encoding

`Content-Encoding` 응답 헤더

- 미디어 타입을 압축하기 위해 사용
- 헤더의 값은 개체 본문에 어떤 추가적인 컨텐츠 인코딩이 적용될지를 나타냄
- `Content-Type` 헤더에 의해 참조되는 미디어 타입을 얻도록 디코드하는 방법을 클라이언트가 알게 해줌

### 문법

```
Content-Encoding: gzip
Content-Encoding: deflate
Content-Encoding: compress
Content-Encoding: identity
Content-Encoding: br
```

- `gzip`
- `deflate`
- `compress`: 오늘날의 브라우저에서는 사용되지 않음
- `identity`
- `br`

### gzip을 이용해 압축하기

클라이언트 측에서 HTTP 요청 내에 함께 전송될 압축 스킴 목록을 알림

```
Accept-Encoding: gzip, deflate
```

서버는 사용한 스킴을 응답

```
Content-Encoding: gzip
```

## Transfer-Encoding

`Transfer-Encoding` 헤더

- 사용자에게 entity를 안전하게 전송하기 위해 사용하는 인코딩 형식을 지정
- hop-by-hop 헤더로, 리소스 자체가 아닌 두 노드 사이에 메시지를 적용
- 다중-노드 연결의 각각의 세그먼트는 `Transfer-Encoding`의 값을 다르게 사용할 수 있음
- 전체 연결에 있어 데이터를 압축하고자 한다면 end-to-end 헤더인 `Content-Encoding` 헤더를 대신 사용할 것

### 문법

```
Transfer-Encoding: chunked
Transfer-Encoding: compress
Transfer-Encoding: deflate
Transfer-Encoding: gzip
Transfer-Encoding: identity

// 어떤 값들은 쉼표로 구분하여 나열될 수 있습니다
Transfer-Encoding: gzip, chunked
```

- `chunked`: 데이터가 일련의 청크 내에서 전송됨. `Content-Length` 헤더는 이 경우 생략됨
  - 각 청크의 앞부분에 현재 청크의 길이가 16진수 형태로 옴
  - 그 뒤에는 `\r\n`이 오고 그 다음에 청크 자체가 오며 그 뒤에는 다시 `\r\n`이 옴
- `compress`
- `deflate`
- `gzip`
- `identity`

### 청크 분할 인코딩

- 더 많은 양의 데이터가 클라이언트에 전송되고 요청이 완전히 처리되기 전까지는 응답의 전체 크기를 알지 못하는 경우에 유용
- DB 쿼리의 결과가 될 큰 HTML 테이블을 생성하는 경우
- 큰 이미지를 전송하는 경우

청크 분할 응답

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

7\r\n
Mozilla\r\n
9\r\n
Developer\r\n
7\r\n
Network\r\n
0\r\n
\r\n
```

## 참고

[Accept-Encoding](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Accept-Encoding)

[Content-Encoding](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Encoding)

[Transfer-Encoding](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Transfer-Encoding)
