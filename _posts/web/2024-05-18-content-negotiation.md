---
title: 콘텐츠 협상
date: 2024-05-18 09:35:16 +0900
last_modified_at: 2024-05-23 14:21:03 +0900
categories: [Web]
tags: [web, content-negotiation]
---

콘텐츠 협상에 대해 알아보겠습니다.

## 콘텐츠 협상

HTTP에서의 콘텐츠 협상(Content negotiation)

- 동일한 URI에서 리소스의 서로 다른 버전을 제공하기 위해 사용하는 메커니즘
- 사용자 에이전트가 사용자에게 제일 잘 맞는 것이 무엇인지를 명시할 수 있음
- 문서의 언어, 이미지 포맷, 적절한 컨텐츠 인코딩

### 콘텐츠 협상의 원칙

리소스

- 특정 문서
- 클라이언트가 리소스를 내려받기를 원하면 그것의 URL을 사용해 요청
- 리소스들은 특유의 URL을 가짐

프레젠테이션

- 리소스가 제공하는 여러 변형들. 각각의 변형을 프레젠테이션이라 함
- 서버는 리소스가 제공하는 여러 변형들 중 하나를 선택하기 위해 이런 URL을 사용
- 클라이언트에게 해당 리소스의 특정 프레젠테이션을 반환

콘텐츠 협상

- 리소스가 호출됐을 때 특정 프레젠테이션을 선택하는 방법을 결정

클라이언트와 서버 간의 협상 방식 2가지

1. 서버 주도 협상(주도적인 협상)
   - 클라이언트가 보내는 특정 HTTP 헤더를 이용하는 방법
   - 특정 종류의 리소스에 대한 표준 협상 방법
2. 에이전트 주도 협상(리액티브 협상)
   - 서버에 의해 전달되는 `300` 혹은 `406` HTTP 응답 코드를 이용하는 방법
   - 폴백 메커니즘으로써 사용됨

### 서버 주도 콘텐츠 협상

서버 주도 콘텐츠 협상(주도적인 협상)

- 리소스의 특정 프레젠테이션에 동의하기 위한 가장 일반적인 방법
- 브라우저는 URL을 이용해 몇 개의 HTTP 헤더를 전송
  - 혹은 사용자 에이전트라면 어떤 다른 종류든지
- 이런 헤더들은 사용자의 우선적인 선택을 나타냄
- 서버는 그것들을 힌트로써 사용하며 클라이언트로 서브하기 위한 최선의 컨텐츠를 선택

HTTP/1.1 표준은 서버 주도 협상을 시작하는 표준 헤더 목록을 정의

- `Accept`
- `Accept-Charset`
- `Accept-Encoding`
- `Accept-Language`

서버는 실제로 콘텐츠 협상에 있어 어떤 헤더가 사용될 지(더 엄밀히 말하자면 연관된 응답 헤더) 가리키기 위해 `Vary` 헤더를 사용하므로 캐시는 최적으로 동작

서버 주도 콘텐츠 협상의 몇 가지 결점

- 서버는 브라우저에 대한 전체적인 지식을 가지고 있지 않음. 클라이언트가 선택하는 리액티브 콘텐츠 협상과 달리 서버 선택은 항상 다소 임의적
- 클라이언트에 의한 정보는 상당히 장황하며 사생활 침해에 대한 위협을 가짐
- 주어진 리소스의 몇몇 프레젠테이션이 전송되므로, 샤드된 캐시들은 덜 효율적이며 서버 구현은 좀 더 복잡해짐

### `Accept` 헤더

`Accept` 헤더

- 에이전트가 처리하고자 하는 미디어 리소스의 MIME 타입
- MIME 타입을 쉼표로 구분
- 각각 품질 인자와 함께 나열

### `Accept-CH` 헤더

`Accept-CH` 헤더

- 적합한 응답을 선택하기 위해 서버가 사용할 수 있는 설정 데이터를 나열
- `DPR`: 클라이언트 기기의 픽셀 비율
- `Viewport-Width`: CSS 픽셀에서의 레이아웃 뷰포트
- `Width`: 물리적인 픽셀에서의 리소스 너비(이미지의 고유 사이즈)

### `Accept-Charset` 헤더

`Accept-Charset` 헤더

- 사용자 에이전트가 어떤 종류의 캐릭터 인코딩을 이해할 수 있는지를 서버에 알림
- UTF-8이 잘 지원되고 있고 선호되는 방식
- 대부분의 브라우저들은 해당 헤더를 생략

### `Accept-Encoding` 헤더

`Accept-Encoding` 헤더

- 수용 가능한(압축을 지원하는) 컨텐츠 인코딩을 정의
- 값은 인코딩 값의 우선 순위를 가리키는 q 인자 목록
- 선언된 것이 없다면 기본값 `identidy`는 가장 낮은 우선 순위

### `Accept-Language` 헤더

`Accept-Language` 헤더

- 사용자가 선호하는 언어를 가리킴
- 이 값은 신뢰할 수 없음
- 서버에서 선택한 언어가 아닌 다른 언어를 선택할 수 있는 방법을 제공해야 함(사이트 상에 언어 메뉴 제공 등)
- 사용자가 서버가 선택한 언어를 재정의하고 나면, 사이트는 더 이상 언어 감지를 사용해서는 안 되며 명시적으로 선택된 언어를 인정해야 함

### `User-Agent` 헤더

`User-Agent` 헤더

- 요청을 전송하는 브라우저를 식별
- 공백 문자로 구분된 제품 토큰(product tokens)과 코멘트 목록을 포함
- 제품 토큰: 브라우저 이름 뒤에 `/`와 버전 번호가 오는 이름
- 코멘트: 둥근 괄호를 경계로 하는 자유 문자열

### `Vary` 응답 헤더

`Vary` 응답 헤더

- 웹 서버에 의한 응답 내로 전달됨
- 서버 주도 콘텐츠 협상의 과정 중에 서버에 의해 사용되는 헤더들의 목록을 나타냄
- 결정 기준 캐시를 알리기 위해 필요
- `*`: 서버 주도 콘텐츠 협상이 적합한 컨텐츠 선택을 위해 헤더로 전달되지 않은 정보도 사용한다는 의미

### 에이전트 주도 협상

서버 주도 협상의 결점

- 확장하기에 불리
- 협상 내에서 사용하는 기능 당 한 가지 헤더가 존재해야 함
- 헤더의 전송은 반드시 모든 요청 상에서 이루어져야 함

에이전트 주도 협상(리액티브 협상)

- 애매모호한 요청과 맞닥뜨렸을 경우, 서버는 사용 가능한 대체 리소스들에 대한 링크를 포함하고 있는 페이지를 회신
- 사용자는 해당 리소스들을 표시하고 사용하려는 리소스를 선택

## 참고

[콘텐츠 협상](https://developer.mozilla.org/ko/docs/Web/HTTP/Content_negotiation)
