---
title: HTTP 압축
date: 2024-05-16 13:21:47 +0900
last_modified_at: 2024-05-20 08:11:39 +0900
categories: [Web]
tags: [web, http-compression]
---

HTTP 압축, 여러 압축 방식

## HTTP 압축

HTTP 압축(HTTP Compression)

- 전송 속도와 대역폭 이용을 개선하기 위해 웹 서버와 웹 브라우저에 빌드되는 기능
  - 필요로 하는 대역폭 용량을 낮춰줘 웹 사이트의 성능을 높임
- HTTP 데이터: 서버로부터 전송되기 전에 압축됨
- gzip, DEFLATE가 가장 흔한 압축 스킴

HTTP에서 수행할 수 있는 2가지 압축 방식

1. 더 낮은 단계 - 전송 인코딩 헤더 필드는 HTTP의 페이로드가 압축되었음을 지시할 수 있음
2. 더 높은 단계 - 콘텐츠 인코딩 헤더 필드는 전송, 캐시, 기타 참조되는 리소스가 압축됨을 지시할 수 있음

콤텐츠 인코딩을 사용하는 압축이 전송 인코딩보다 더 널리 지원됨

압축이 일어나는 3개의 서로 다른 계층

1. 먼저 몇 개의 파일 형식이 최적화된 특유의 방법으로 압축됨
2. 그 뒤 HTTP 계층에서 일반적인 암호화가 일어남(리소스는 끝단 간에 압축되어 전송됨)
3. 마침내 압축이 HTTP 커넥션의 두 노드 사이의 커넥션 계층에서 정의도리 수 있음

### 압축 스킴 협상

1 압축 스킴 알리기

```
GET /encrypted-area HTTP/1.1
Host: www.example.com
Accept-Encoding: gzip, deflate
```

- `Content-Encoding`의 경우 필드 내 목록은 `Accept-Encoding`으로 호칭됨
- `Transfer-Encoding`의 경우 필드 이름은 `TE`로 호칭됨

2 나가는 데이터 압축

```
HTTP/1.1 200 OK
Date: mon, 26 June 2016 22:38:34 GMT
Server: Apache/1.3.3.7 (Unix)  (Red-Hat/Linux)
Last-Modified: Wed, 08 Jan 2003 23:11:55 GMT
Accept-Ranges: bytes
Content-Length: 438
Connection: close
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
```

- 서버가 하나 이상의 압축 스킴을 지원하면, 나가는 데이터는 양측에서 모두 지원되는 하나 이상의 메서드에 의해 압축됨
- 서버는 HTTP 응답 안에 사용되는 스킴과 함께 `Content-Encoding` 또는 `Transfer-Encoding`을 추가하게 되며 쉼표로 구분

### 파일 포맷 압축

- 텍스트는 일반적으로 60%의 중복을 가짐

무손실 압축

- 압축-비압축 사이클이 복원된 데이터를 변경하지 않는 것
- 이미지에서는 `gif`, `png`가 사용

손실 압축

- 사용자가 인지하기 힘든 방법 내에서 원래의 데이터를 사이클이 변경하는 것
- 웹에서 비디오 포맷, 이미지에서 `jpeg`
- 일반적으로 무손실 압축 알고리즘보다 효율적

### 종단 간 압축

End-to-end compression

- 웹 사이트의 가장 큰 성능 이득이 발생하는 곳
- 서버에 의해 처리되고 클라이언트에 도달할 때까지 결코 변하지 않을 메시지 바디의 압축을 나타냄
- 중간 노드가 무엇이든지 바디는 건들지 않고 그대로 둠
- 모든 모던 브라우저와 서버들은 종단간 압축을 지원
- 사용할 압축 알고리즘만 유일하게 협상
- 이런 압축 알고리즘들은 텍스트에 최적화됨
- `gzip`, `br`

사전 컨텐츠 협상(proactive content negotiation)

- 사용할 알고리즘을 선택하기 위해 브라우저와 서버가 사용
- 브라우저: 브라우저가 지원하는 알고리즘, 그것의 우선순위와 함께 `Accept-Encoding` 헤더 전송
- 서버: 해당 헤더를 뽑아내 응답의 바디를 압축하는 데 사용하고, 서버가 선택한 알고리즘을 `Content-Encoding` 헤더로 브라우저에게 알림
- 적어도 `Content-Encoding`을 포함하는 하나의 `Vary` 헤더가 반드시 응답 내 해당 헤더에 붙어 전송되어야 함
- 그러한 방법으로 캐시는 리소스의 다른 표현을 캐시하는 게 가능해짐

압축

- 명확한 성능 향상을 제공
- 모든 파일에 대해 활성화하는 것이 좋으나, 이미지, 오디오, 비디오 같은 파일들은 이미 압축되어 있을 것

### Hop-by-hop 압축

- 종단 간 압축과 비슷하나 한 가지 필수적인 엘리먼트에 의한 차이가 있음
- 서버 내에서 리소스에 대한 압축이 일어나지 않고, 클라이언트와 서버 사이의 경로 상에 있는 어떤 두 노드 사이에서 메시지의 바디에 압축 발생
- 중간 노드 간의 커넥션들은 다른 압축을 적용할 수도 있음
- 이를 위해 HTTP는 종단 간 압축에서의 컨텐츠 협상과 유사한 메커니즘을 사용
- 요청을 전송하는 노드는 노드의 요청이 `TE` 헤더를 사용하고 있다는 것을 알림
- 다른 노드들은 알맞은 방법을 선택해 적용하고 `Transfer-Encoding` 헤더를 사용해 선택한 내용을 가리킴
- hop-by-hop 압축은 서버와 클라이언트에는 보이지 않으며 드물게 사용됨
- `TE` 헤더와 `Transfer-Encoding` 헤더는 리소스의 길이를 알지 못한 상태로 전송을 시작하도록 청크에 의해 응답을 전송하는 데 대부분 사용됨

## 참고

[HTTP 압축](https://ko.wikipedia.org/wiki/HTTP_압축)

[HTTP에서의 압축](https://developer.mozilla.org/ko/docs/Web/HTTP/Compression)

[Content-Encoding](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Encoding)
