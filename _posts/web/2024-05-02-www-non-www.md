---
title: www와 비-www 중에 선택하기
date: 2024-05-02 15:15:02 +0900
last_modified_at: 2024-05-02 15:15:02 +0900
categories: [Web]
tags: [web, www, redirect, canonical]
---

www 혹은 비-www URL 중 무엇을 선택해야 할까요?

## 도메인 네임이란 무엇인가요?

도메인 네임(Domain Name)

- HTTP URL에서, 앞 부분의 `http://` 또는 `https://` 다음에 오는 첫 번째 하위 문자열
- 도메인 네임은 문서가 위치하고 있는 서버에서 호스팅됨
- 서버는 꼭 물리적인 하드웨어일 필요는 없음
- 요점은 의미적으로 하나의 도메인 이름이 하나의 단일 서버를 나타냄

## 그래서, 내 사이트 웹에 대해 하나 또는 다른 것을 선택해야만 하나요?

그렇습니다

- 하나를 선택하고 그것과 함께 결속될 필요가 있음

그렇지 않습니다

- 두 가지 모두를 가질 수는 없음
- 중요한 것은 공식 도메인을 이용해 결속되어 있고 일관되다는 것
- 이 공식 도메인을 '표준' 이름이라고 부름
- 나의 모든 절대 링크는 그 이름을 사용해야 함
- 그러나 그럼에도 나는 여전히 동작하는 다른 도메인을 가질 수 있음

나의 도메인 중 하나를 나의 표준 도메인으로 선택

- 표준 도메인이 아닌 도메인을 여전히 동작하게 만들기 위한 두 가지 기술이 아래 있음

## 표준 URL을 위한 기술들

'표준' 웹 사이트를 선택하기 위한 서로 다른 방법들

### HTTP 301 리다이렉트 사용하기

- 비표준 도메인에 대한 알맞은 HTTP 301로 응답하기 위해 서버가 HTTP 요청을 받도록 구성해야 함
- 표준 URL과 동등한 비표준 URL에 접근하도록 브라우저를 리다이렉트시킴
- 예를 들어 비-www URLs를 표준 타입으로 사용하도록 선택했다면, 모든 www URL들은 그와 동등한 www가 없는 URL로 리다이렉트되게 됨

예제

1. 서버가 `http://www.example.org/whaddup`에 대한 요청을 받음 (표준 도메인이 `example.org`인 경우)
2. 서버는 헤더인 `Location: http://example.org/whaddup`과 함께 `301` 코드로 응답
3. 클라이언트는 표준 도메인 `http://example.org/whaddup`에 대한 요청을 발급

### `<link rel="canonical">` 사용하기

- 페이지의 표준 주소가 무엇인지를 가리키기 위해 페이지에 특별한 `<link>` 요소를 추가
- 페이지를 보는 사람에게는 별다른 영향이 없지만, 검색 엔진 크롤러에게 페이지가 실제로 위치한 곳을 알려줌
- 그렇게 하면 검색 엔진이 동일한 페이지를 여러 번 색인하지 않으므로, 중복 컨텐츠 혹은 어떤 종류의 스팸으로 간주하게 될 수도 있고 심지어 검색 엔진 결과 페이지에서 나의 페이지를 제거하거나 우선순위가 낮아질 수도 있음
- 그런 태그를 추가하게 되면 두 가지 도메인에 대해 같은 내용을 서브하며 어떤 URL이 표준인지를 검색 엔진에게 알려주게 됨
- 이전 예제에서 `http://www.example.org/whaddup`은 `http://example.org/whaddup`과 동일한 내용을 서브하지만 head 태그 내 추가적인 `<link>` 요소가 자리하고 있을 것임
- 이전 경우와 다르게, 브라우저 기록은 비-www와 www URL을 개별적인 엔트리로 판단할 것

```html
<link href="http://example.org/whaddup" rel="canonical" />
```

## 페이지가 두 가지 경우 모두에 동작하도록 만드세요

## 참고

[www와 비-www URL 중에서 선택하기](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/Choosing_between_www_and_non-www_URLs)

[도메인 주소에 www 가 있고 없고의 차이](https://joonius.tistory.com/19)

[도메인의 서브도메인 와일드 카드 알아가기](https://studyforus.tistory.com/185)

[www 가 들어가는것과 아닌것의 차이는 무엇인지요?](https://xetown.com/questions/1131734)

[4가지 사례로 알아보는 올바른 캐노니컬 태그 적용 방법](https://www.twinword.co.kr/blog/how-to-apply-canonical-tag-properly/)
