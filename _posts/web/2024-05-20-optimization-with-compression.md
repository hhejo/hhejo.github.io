---
title: gzip 압축으로 사이트 최적화하기
date: 2024-05-20 10:49:30 +0900
last_modified_at: 2024-05-23 08:24:09 +0900
categories: [Web]
tags: [web, gzip, deflate]
---

사이트를 최적화하는 방법 중 하나인 압축에 대해 알아보겠습니다.

## 압축으로 사이트 최적화하기

압축

- 대역폭 절약
- 사이트 속도 향상

### 예시

HTTP Request와 Response

- `http://www.yahoo.com/index.html` 파일 요청의 경우

```
1. Browser Request                  2. Web Server Finds File
                          ---1KB-->
GET /index.html HTTP/1.1            /var/www/.../index.html
                                            |
                                        Read File
                                            |
4. Browser Displays Page            3. Server Response
                          <-100KB-- HTTP/1.x 200 OK
                                    <html>...</html>
```

문제점

- 작동은 하지만 효율적이지 않음
- 100KB은 많은 텍스트 양이며, HTML은 중복되는 요소가 많음
- 대부분의 태그는 동일하게 닫는 태그를 가짐
- 어떤 식으로든 웹에서는 HTML 문서를 사용
- 파일이 너무 큰 경우 압축을 해야 함

이전까지 사용했던 평범한 index.html 대신 .zip 파일(index.html.zip)을 브라우저에 전송

- 대역폭, 다운로드 시간 절약
- 로드 시간 절약

Compress HTTP Response

```
1. Browser Request                  2. Web Server Finds File
GET /index.html HTTP/1.1  ---1KB-->
Accept-encoding: gzip               /var/www/.../index.html
                                            |
                                         Zip File
                                            |
4. Browser Decompresses &           3. Server Response
    Displays Page         <--10KB-- HTTP/1.x 200 OK
                                    Content-encoding: gzip
                                    <compressed file>
```

### 브라우저와 서버

- 압축된 파일을 전송해도 좋다는 것을 브라우저와 서버가 모두 알고 있어야 함

브라우저

- 압축된 콘텐츠를 허용한다는 내용을 헤더에 담아 서버에 알림
- `Accept-Encoding: gzip, deflate`
- 브라우저에 의한 요청일 뿐 요구는 아님

서버

- 콘텐츠가 실제로 압축된 경우 압축 방식을 헤더에 응답을 보냄
- `Content-Encoding: gzip`
- 서버가 해당 응답 헤더를 전송하지 않았다면 파일은 압축되지 않았다는 의미
- 서버가 콘텐츠를 압축하지 않기를 원할 수 있음

### 서버 세팅

- 우리는 브라우저를 통제할 수 없음
- 브라우저는 `Accept-Encoding: gzip, deflate` 헤더를 전송할 수도 있고 안 할 수도 있음
- 브라우저가 압축된 콘텐츠를 처리할 수 있다면, 콘텐츠를 압축해서 반환하도록 서버를 구성해 대역폭을 절약하는 것이 좋음

IIS(Internet Information Services)

- IIS의 경우 HTTP 설정에서 압축을 활성화
- 마이크로소프트 윈도우를 사용하는 서버들을 위한 인터넷 기반 서비스들의 모임

Apache

- `mod_deflate`: 표준. 설정이 쉬움
- `mod_gzip`: 콘텐츠를 미리 압축할 수 있어 더 강력함
- 두 가지 경우 모두 Apache에서 브라우저가 `Accept-Encoding` 헤더를 보내는지 확인해 압축 또는 정규 버전을 전송

### 압축 검증

온라인에서 확인

- 온라인 gzip 테스트 사이트를 사용해 페이지가 압축되었는지 확인

브라우저에서 확인

- Chrome 관리자 도구 - 네트워크 탭
- 페이지 새로고침 후 페이지 자체의 네트워크 줄 클릭
- 헤더의 `content-encoding: gzip`은 콘텐츠가 압축된 상태로 전송되었음을 의미

### 주의사항

이전 브라우저

- 몇몇 브라우저는 여전히 압축 콘텐츠에 대해 오류가 발생할 수 있음

이미 압축된 콘텐츠

- 대부분의 이미지, 음악, 비디오는 이미 압축됨
- 다시 압축하기 위해 시간을 낭비할 필요는 없음
- HTML, CSS, JavaScript만 압축하면 됨

CPU-로드

- 즉석에서 콘텐츠를 압축하면 CPU 시간이 사용되고 대역폭이 절약됨
- 정적 콘텐츠를 미리 압축하고 압축된 버전으로 전송하는 방법이 있음

## 참고

[How To Optimize Your Site With GZIP Compression](https://betterexplained.com/articles/how-to-optimize-your-site-with-gzip-compression/)

[WEB, 번역 Gzip 압축으로 사이트 최적화하는 방법](https://velog.io/@ss-won/FE-번역-Gzip-압축으로-사이트-최적화하는-방법)

[인터넷 정보 서비스](https://ko.wikipedia.org/wiki/인터넷_정보_서비스)
