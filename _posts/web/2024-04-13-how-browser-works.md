---
title: 브라우저는 어떻게 동작하는가
date: 2024-04-13 15:21:14 +0900
last_modified_at: 2024-04-13 15:21:14 +0900
categories: [Web]
tags: [web, browser]
---

브라우저는 어떻게 동작할까요? 순서대로 알아보겠습니다.

## 개요

웹 성능에 있어 두 가지 주요한 문제

- 지연시간
- 브라우저가 대부분 싱글 쓰레드로 동작

지연시간

- 빠른 로딩을 위해 이겨내야할 중요한 문제
- 빠른 로딩을 위해 신경써야할 것에는 최대한 빠르게 요청하는 것도 포함됨
- `네트워크 지연 시간`: 네트워크를 통해 컴퓨터로 바이트를 전송하는 데 걸리는 시간

브라우저가 대부분 싱글 쓰레드로 동작

- 메인 쓰레드가 요청된 모든 작업을 수행하면서도 유저와의 상호작용에 반응할 수 있도록 보장하기 위해서는 렌더링 시간이 가장 중요
- 브라우저가 싱글 쓰레드로 동작한다는 점을 이해하고, 가능한 메인 쓰레드의 책임을 줄여주는 방식으로 웹 성능 향상을 이룰 수 있음
- 이렇게 하면 렌더링은 부드럽고, 상호작용에 대한 응답은 즉각적일 것

## 탐색(Navigation)

웹 페이지를 로딩하는 첫 단계

사용자가 주소창에 URL을 입력, 링크를 클릭, 폼을 제출하는 등의 동작을 통해 요청을 보낼 때마다 발생

웹 최적화의 목표 중 하나는 탐색이 완료될 때까지의 시간을 최소화하는 것

이상적인 조건에서 그다지 오래 걸리는 작업이 아니지만, 지연시간과 대역폭은 지연을 일으키는 적

### DNS 조회(DNS Lookup)

웹 페이지를 탐색하는 첫 단계는, 해당 페이지의 자원이 어디에 위치하는지 찾는 것

`https://example.com`을 탐색

- HTML 페이지는 IP 주소가 `93.184.216.34`인 서버에 위치
- 만약 이 사이트를 한번도 방문한 적이 없다면 DNS 조회가 필요

브라우저는 DNS 조회를 요청

- 최종적으로 이름 서버에 의해 처리되고 IP 주소로 응답함
- 최초의 요청 이후에 IP는 일정 기간 동안 캐시됨
- 이름 서버에 다시 연락하는 대신 캐시에서 IP 주소를 검색해 후속 요청 속도를 높임

DNS 조회는 보통 호스트 이름 하나당 한번만 수행됨

- 하지만 DNS 조회는 요청된 페이지에서 참조하는 다른 호스트 이름에 대해서는 각각 수행해야 함
- 만약 글꼴, 이미지, 스크립트, 광고 그리고 다른 자원들이 서로 다른 호스트 이름을 가지고 있다면, DNS 조회는 각각에 대해서 모두 수행되어야 함
- 이는 특히 모바일 네트워크 환경에서 성능에 문제가 될 수 있음
- 사용자가 모바일 환경에 있을 때, 각각의 DNS 조회는 휴대폰에서 셀 타워에 가야하고, 셀 타워에서 권위 있는 DNS 서버에 도달해야함
- 휴대폰과 셀 타워, 그리고 이름 서버의 거리에 따라서 상당한 지연시간이 생길 수도 있음

### TCP 핸드셰이크(TCP Handshake)

IP 주소를 알고난 후, 브라우저는 서버와 TCP 3방향 핸드셰이크를 통해 연결을 설정

데이터를 전송하기 전에(주로 HTTPS를 통해서) 통신하려는 두 주체(이 경우에는 브라우저와 웹 서버)가 TCP 소켓 연결을 위한 매개변수를 주고받을 수 있도록 만들어진 방식

TCP의 3방향 핸드셰이크 기술

- "SYN-SYN-ACK"로 불리기도 함
  - 더 정확히는 SYN, SYN-ACK, ACK
- 두 컴퓨터 간 TCP 세션을 협상하고 시작하기 위해서 TCP가 3개의 메시지를 전달하기 때문
- 이는 요청이 보내지기 전에 3개의 추가적인 메시지가 컴퓨터 사이에 주고받아진다는 의미

### TLS 협상(TLS Negotiation)

HTTPS를 이용한 보안성 있는 연결을 위한 또 다른 핸드셰이크

통신 암호화에 쓰일 암호를 결정하고, 서버를 확인하고, 실제 데이터 전송 전에 안전한 연결이 이루어지도록 함

이를 위해서 자원에 대한 실제 요청 전에 클라이언트에서 서버로 3번 더 왕복해야 함

```
---www.example.com--> DNS
<---93.184.216.34----

------SYN----->
<---SYN-ACK---- Site
------ACK----->

---------ClientHello-------->
<--ServerHello&Certificate---
---------ClientKey----------> Site
<---------Finished-----------
----------Finished---------->
```

보안성 있는 연결

- 연결에 보안성을 더하는 것은 페이지 로딩을 더디게 함
- 하지만 보안성 있는 연결은 지연시간이라는 비용을 낼만큼 충분히 가치가 있음
- 브라우저와 웹 서버 사이에 전송되는 데이터가 제 3자에 의해서 해독될 수 없게 되기 때문

8번의 왕복이 있은 후에, 브라우저는 마침내 요청을 할 수 있음

## 응답(Response)

웹 서버와 연결 성립

- 웹 서버로 한번 연결이 성립되고 나면, 브라우저는 유저 대신에 초기 HTTP GET request를 보냄
- 웹 사이트는 대개 HTML 파일을 요청
- 서버가 요청을 받으면, 관련 응답 헤더와 함께 HTML의 내용을 응답
- 이 초기 요청에 대한 응답은 수신된 첫 바이트 데이터를 포함하고 있음
- 구문 분석되는 중에 브라우저가 링크를 만날 때까지 링크가 걸린 자원들은 요청되지 않음

TTFB(Time To First Byte)

- 사용자가 요청을 보내고 HTML의 첫 패킷을 받는 데 걸린 시간
  - 링크를 클릭하는 등의 방식의 요청
- 첫 번째 컨텐츠 청크는 일반적으로 14KB 크기의 데이터

### TCP 슬로우 스타트(TCP Slow Start) / 14kb rule

네트워크 통신의 속도를 조절하는 알고리즘

- TCP 슬로우 스타트(TCP Slow Start)
- 이 알고리즘에 의해 첫 응답 패킷은 14kb로 정해짐
- 네트워크의 최대 대역폭을 파악할 수 있을 때까지 점진적으로 데이터의 전송량을 증가시킴

TCP 슬로우 스타트 방식

- 첫 패킷을 받고난 이후에, 서버는 다음 패킷의 사이즈를 두 배인 28kb로 늘림
- 뒤 이은 패킷의 크기도 미리 정의한 임계치에 다다르거나, 혼잡의 징후가 나타나기 전까지 2배씩 커짐

첫 페이지의 로딩에 관련한 14kb 법칙

- 초기 응답의 크기가 14kb인 이유는 TCP 슬로우 스타트 때문
- 웹 최적하를 할 때 초기 14kb 응답을 염두해야하는 것도 이 때문
- TCP 슬로우 스타트는 혼잡을 피하기 위해서 네트워크의 용량에 적당한 전송 속도를 찾고자 점진적으로 속도를 높여나감

### 혼잡 제어(Congestion Control)

서버가 TCP 패킷으로 데이터를 보내면서, 사용자의 클라이언트는 확인 응답(acknowledgements, ACKs)을 보내면서 데이터의 수신을 확인해줌

연결은 하드웨어나 네트워크 상태에 따라서 제한된 용량만을 가짐

만약 서버가 패킷을 너무 빠르게 보내게 되면, 그 패킷들을 무시될 것

- 즉, 확인 응답이 없을 것임

서버는 이를 누락된 확인 응답으로 파악함

혼잡 제어 알고리즘은 보내진 패킷의 흐름과 확인 응답을 바탕으로 전송 속도를 결정함

## 구문 분석(Parsing)

구문 분석

- 브라우저가 네트워크를 통해 받은 데이터를 DOM이나 CSSOM으로 바꾸는 단계
- 브라우저가 첫 번째 데이터의 청크를 받으면, 수신된 정보를 구문 분석하기 시작
- 이는 렌더러가 화면에 페이지를 그리는 데 사용됨

DOM

- 브라우저는 마크업을 내부적으로 DOM으로 표현
- DOM은 공개되어 있고, 자바스크립트의 다양한 API를 통해 조작 가능

요청된 HTML의 페이지의 크기가 초기 패킷의 크기인 14kb보다 크더라도, 브라우저는 구문 분석을 시작하고 가지고 있는 데이터 수준에서 렌더링을 시도함

이것이 웹 성능 최적화에서 브라우저가 렌더링을 하는데 필요한 모든 것, 아니면 적어도 페이지의 템플릿(첫 렌더링에 필요한 HTML이나 CSS)만이라도 첫 14kb에 포함해야 하는 이유임

하지만 화면에 렌더링하기 전에 HTML, CSS, Javascript를 구문 분석해야 함

### DOM 트리 구축(Building the DOM tree)

중요한 렌더링 경로(Critical Rendering Path)를 다섯 가지 단계로 설명

첫 단계는 HTML을 처리하여 DOM 트리를 만드는 것

HTML 구문 분석은 토큰화와 트리 생성을 포함

HTML 토큰은 시작 및 종료 태그 그리고 속성 이름 및 값을 포함

만약 문서가 잘 구성되어 있다면 구문 분석은 명확하고 빠르게 이루어짐

구문 분석기는 토큰화된 입력을 분석하여 DOM 트리를 만듦

DOM 트리

- DOM 트리는 문서의 내용을 설명
- html 요소는 시작하는 태그이고 DOM 트리의 루트 노드
- 트리는 다른 태그간의 관계와 계층을 반영
- 다른 태그에 감싸져 있는 태그는 자식 노드
- DOM 노드의 개수가 많아질 수록, DOM 트리를 만드는 데 더 오랜 시간이 걸림

구문 분석기가 이미지와 같은 논 블로킹 자원을 발견하면, 브라우저는 해당 자원을 요청하고 분석을 계속함

구문 분석은 CSS 파일을 만났을 때도 지속될 수 있음

하지만 `async`나 `defer` 같은 설정이 되어 있지 않은 `<script>` 태그는 렌더링을 막고, HTML의 분석을 중지시킴

브라우저의 프리로드 스캐너가 이 작업을 가속화하지만, 과도한 스크립트는 여전히 주요한 병목구간이 될 수 있음

### 프리로드 스캐너(Preload scanner)

브라우저가 DOM 트리를 만드는 프로세스는 메인 쓰레드를 차지함

그렇기 때문에, 프리로드 스캐너는 사용 가능한 컨텐츠를 분석하고 CSS, Javascript, 웹 폰트 같이 우선순위가 높은 자원을 요청함

프리로드 스캐너 덕에 구문 분석기가 외부 자원에 대한 참조를 찾아 요청하기까지 기다리지 않아도 됨

프리로드 스캐너가 자원을 뒤에서 미리 요청함

그래서 구문 분석기가 요청되는 자원에 다다를 때 쯤이면 이미 그 자원들을 전송받고 있거나 이미 전송받은 후일 것

프리로드 스캐너가 제공하는 최적화는 블록킹을 줄여줌

```html
<link rel="stylesheet" src="styles.css" />
<script src="myscript.js" async></script>
<img src="myimage.jpg" alt="image description" />
<script src="anotherscript.js" async></script>
```

- 메인 쓰레드가 HTML과 CSS를 분석하고 있을 때, 프리로드 스캐너는 스크립트와 이미지를 찾아 다운로드하기 시작할 것
- Javascript의 분석과 실행 순서가 중요하지 않고 스크립트가 프로세스를 막지 않도록 하려면 `async` 속성이나 `defer` 속성을 추가하면 됨

CSS를 다운로드하는 것은 HTML 분석이나 다운로드를 막지 않음

하지만 Javascript의 실행은 막음

Javascript는 종종 요소에 영향을 주는 CSS 속성들을 조작하기 때문

### CSSOM 구축(Building the CSSOM)

중요한 렌더링 경로에서 두 번째 단계는 CSS를 처리하고 CSSOM 트리를 만드는 것

CSS 객체 모델은 DOM과 비슷

DOM과 CSSOM은 둘 다 트리 구조

둘은 각각의 독립적인 자료구조

브라우저는 CSS 규칙을 이해할 수 있고 작업을 진행할 수 있도록 스타일 맵으로 변환함

브라우저는 CSS에 있는 각각의 규칙을 읽고, 트리 노드를 만듦

CSS 선택기에 기반해서 부모 노드, 자식 노드, 형제 관계의 노드를 만들어짐

HTML이 그러한 것처럼, 브라우저는 전송받은 CSS 규칙을 작업 가능한 상태로 변환해야 함

따라서 브라우저는 HTML을 객체로 바꾼 프로세스를 CSS에 대해서 다시 한 번 함

CSSOM 트리는 사용자 에이전트의 스타일 시트를 포함함

브라우저는 노드에 적용 가능한 가장 일반적인 규칙부터 적용함

그리고 재귀적으로 더 구체적으로 적용된 규칙에 따라 계산된 스타일을 수정해감

다른 말로, 속성 값을 캐스케이드함

CSSOM을 만드는 것은 매우 매우 빠르고 현재 개발자 도구에서 고유한 색으로 표시되지 않음

개발자 도구에서 "스타일 재계산"에는 CSS를 구문 분석하고, CSSOM 트리를 만들고, 계산된 스타일을 재귀적으로 계산하는 데 드는 총 시간이 표시됨

CSSOM을 만드는 데 드는 시간은 일반적으로 한 번의 DNS 조회를 하는 시간보다 짧기 때문에 웹 성능 최적화의 관점에서 CSSOM은 성능 향상에 큰 기여를 할 수 있는 영역은 아님

### 다른 작업들(Other Processes)

Javascript 컴파일(JavaScript Compilation)

- CSS가 분석되고 CSSOM이 생성되는 동안, 프리 스캐너 덕에 Javascript 파일 같은 다른 자원도 다운로드됨
- Javascript는 해석, 컴파일, 구문 분석 및 실행됨
- 스크립트는 추상 구문 트리로 구문 분석됨
- 일부 브라우저 엔진은 추상 구문 트리를 인터프리터에게 넘김
- 그 결과 메인 쓰레드에서 실행되는 바이트코드가 생성됨
- 이것이 Javascript 컴파일 과정

접근성 트리 구축(Building the Accessibility Tree)

- 브라우저는 접근성 트리를 만듦
- 보조 장치는 이 트리를 이용해 내용을 분석하고 해석
- 접근성 객체 모델(AOM)은 DOM의 의미 버전
- 브라우저는 DOM이 업데이트 될 때 접근성 트리도 업데이트함
- 접근성 트리는 보조 기술 자체적으로 수정될 수는 없음
- AOM이 만들어지기 전까지, 화면 리더기는 컨텐츠에 접근할 수 없음

## 렌더(Render)

렌더링 과정에는 스타일, 레이아웃, 페인트, 그리고 때때로 합성이 포함됨

CSSOM과 DOM 트리는 구문 분석되는 과정에서 생성되고 렌더 트리로 합성됨

렌더 트리는 보이는 요소의 레이아웃을 계산을 함

그러고 나서 요소가 화면에 페인트됨

어떤 경우에는 컨텐츠가 자신만의 레이어를 가지도록 조작되고, 나중에 합성됨

화면의 일부분을 CPU 대신 GPU가 그리면서 메인 쓰레드의 부담이 줄고 성능이 향상됨

### 스타일(Style)

중요한 렌더링 경로에서 세 번째 단계는 DOM과 CSSOM을 합쳐 렌더 트리를 만드는 것

계산된 스타일 트리(다른 말로 렌더 트리)는 DOM 트리의 루트부터 시작하여 눈에 보이는 노드를 순회하며 만들어짐

`<head>`와 그 자식 요소 혹은 사용자 정의 스타일 시트에 정의된 `script { display: none; }`처럼 `display: none` 스타일 속성을 가진 요소와 같이, 화면에 나타나지 않는 태그의 경우 렌더링 결과에 나타나지 않을 것이기 때문에 렌더 트리에 포함되지 않음

`visibility: hidden` 속성을 가진 요소는 자리를 차지하기 때문에 렌더 트리에 포함됨

코드 예시에서 사용자 에이전트가 기본으로 설정한 값을 오버라이드하는 정의를 하지 않았기 때문에, `script` 노드는 렌더 트리에 포함되지 않을 것

각각의 보이는 노드는 그 노드에 적용된 CSSOM 규칙이 있음

렌더 트리가 보이는 모든 노드의 내용과 계산된 스타일을 가지고 있음

DOM 트리에서 보이는 모든 노드에 관련된 스타일을 모두 맞춰보고, CSS 캐스케이드 방식에 따라서 각 노드의 계산된 스타일이 무엇일지 결정함

### 레이아웃(Layout)

중요한 렌더링 과정에서 네 번째 단계는 렌더 트리를 기반으로 각 노드의 도형 값을 계산하기 위해 레이아웃을 실행하는 것

레이아웃은 렌더 트리에 있는 모든 노드의 너비, 높이, 위치를 결정하는 프로세스

추가로 페이지에서 각 객체의 크기와 위치를 계산

리플로우는 레이아웃 이후에 있는 페이지의 일부분이나 전체 문서에 대한 크기나 위치에 대한 결정

렌더 트리가 한 번 만들어지고 나면, 레이아웃이 시작됨

렌더 트리는 (보이지 않더라도) 계산된 스타일과 함께 어떤 노드가 화면에 표시될지 식별

하지만 각 노드의 위치나 좌표를 알지는 못함

각 객체의 정확한 크기와 위치를 결정하기 위해서, 브라우저는 렌더 트리의 루트부터 시작하여 순회

웹 페이지에서 대부분은 박스 형태

다른 기기, 다른 데스크탑 설정은 제한 없이 매우 다양한 뷰 포트 크기를 가짐

레이아웃 단계에서 뷰 포트의 크기를 고려함

브라우저는 화면에 표시될 모든 다른 상자의 크기를 결정

뷰 포트의 크기를 기본으로하며, 레이아웃은 일반적으로 본문에서 시작해 모든 후손의 크기를 각 요소의 박스 모델 속성을 통해 계산

이미지와 같이 크기를 모르는 요소를 위해서 위치 표시 공간을 남겨둠

처음 노드의 사이즈와 위치가 결정되는 것을 레이아웃이라고 부름

이후에 노드의 크기와 위치를 다시 계산하는 것은 리플로우라고 부름

예제에서, 첫 레이아웃이 이미지가 오기 전에 일어난다고 가정을 하면

이미지의 크기를 선언하지 않았기 때문에, 이미지 크기를 알게 된 이후 리플로우가 한 번 있을 것임

### 페인트(Paint)

중요한 렌더링 경로에서 마지막 단계는 각 노드를 화면에 페인팅하는 것

페인팅이 처음 일어나는 것을 첫 번째 의미있는 페인트라고 부름

페인팅 혹은 레지스터화 단계에서, 브라우저는 레이아웃 단계에서 계산된 각 박스를 실제 화면의 픽셀로 변환

페인팅에서 텍스트, 색깔, 경계, 그림자 및 버튼이나 이미지 같은 대체 요소를 포함하여 모든 요소의 시각적인 부분을 화면에 그리는 작업이 포함됨

브라우저는 이 작업을 매우 빠르게 해야함

부드러운 스크롤이나 애니메이션을 위해서, 스타일 계산, 리플로우, 페인팅과 같이 메인 쓰레드를 점유하는 모든 작업은 브라우저를 16.67ms 미만만 차지해야만 함

2048 X 1536 화면에서 iPad는 화면에 페인트해야 할 3,145,000 픽셀을 가지고 있음

이는 매우 많은 픽셀이며, 이 픽셀은 매우 빠르게 페인팅되어야 함

첫 페인팅보다 다시 페인팅하는 것이 더 빠르게 마무리되기 위해서, 화면에 그리는 작업은 일반적으로 몇 개의 레이어로 구분됨

이것이 일어나면 합성이 필요함

페인팅은 레이아웃 트리의 요소를 레이어로 분리할 수 있음

컨텐츠를 CPU의 메인 쓰레드에서 GPU 레이어로 격상하는 것은 페인트 및 리페인트 성능을 높임

레이어를 가동시키는 구체적인 속성과 요소가 있음

요소에는 `<video>` 그리고 `<canvas>`가 포함되어 있음

구체적인 속성에는 `opacity`, 3D `transform`, `will-change` 등이 있음

자손 노드가 위의 이유 중 하나(혹은 여러 개)로 자신만의 레이어를 필요로 하는 것이 아니라면, 이 노드는 그들의 레이어에서 그들의 자손과 함께 그려짐

레이어는 성능을 향상시킴

하지만 메모리 관리 측면에서 봤을 때는 비싼 작업임

따라서 웹 성능 최적화 전략으로 과도하게 쓰이지는 않아야 함

### 합성(Compositing)

문서의 각 섹션이 다른 레이어에서 그려질 때, 섹션을 겹쳐놓으면서 그것들이 올바른 순서로 화면에 그려지는 것과 정확한 렌더링을 보장하기 위해 합성이 필요함

페이지가 계속해서 자원을 로드하면, 리플로우가 일어날 수 있음(예제에서 이미지가 늦게 도착하는 것을 떠올려보세요)

리플로우는 리페인트와 재합성을 일으킬 수 있음

이미지의 사이즈를 미리 정해놨다면 리플로우는 필요하지 않을 것임

그리고 리페인트 되야할 레이어만 다시 리페인트 하고 필요하다면 합성할 것임

하지만 예제에서는 이미지의 사이즈를 정하지 않았음!

이미지가 서버로부터 받아진 후, 렌더링 과정은 레이아웃 단계로 돌아가서 다시 시작됨

## 상호작용(Interactivity)

메인 쓰레드가 페이지를 그리는 것을 완료하면, 모든 것이 준비되었다고 생각할 수도 있음

하지만 꼭 그렇지는 않음

만약 지연된 Javascript를 다운했다면, 그리고 onload 이벤트가 발생할 때 코드가 실행된다면, 메인 쓰레드는 여전히 바쁠 것임

그래서 스크롤링, 터치 등 다른 상호작용이 불가능 할 것임

Time to Interactive (TTI) 는 DNS 조회와 SSL 연결이 이루어지는 첫 요청부터 페이지가 상호작용할 준비가 될 때까지 얼마나 걸리는지를 측정하는 단위

첫 번째 콘텐츠가 포함된 페인트 이후 페이지가 사용자와의 상호작용에 50ms 이내로 응답할 때를 상호작용 가능한 시점으로 봄

만약 메인 쓰레드가 구문 분석, 컴파일, Javascript 실행에 사용되고 있다면, 메인 쓰레드를 사용할 수 없고 따라서 사용자 상호작용에 50ms 이내로 적절하게 반응하지 못함

예제에서, 이미지는 매우 빠르게 로드됨

하지만 만약 anotherscript.js 파일이 2MB였고 사용자의 네트워크 연결이 느렸다면 어땠을까요?

이 경우에는 사용자는 페이지는 매우 빠르게 볼 수 있지만 스크립트가 다운로드되고, 분석되고 실행되기 전까지는 버벅이는 스크롤을 할 수 밖에 없을 것임

이는 좋은 사용자 경험이 아님

WebPageTest 예시에서 볼 수 있듯이, 메인 쓰레드를 점유하는 것을 피해야 함

사진

이 예시에서, DOM 컨텐츠를 로드하는 프로세스는 1.5초 이상 걸렸음

그리고 클릭이나 화면 탭에 응답하지 못하는 상태로 메인 쓰레드는 그 전체 시간동안 점유되었음

## 참고

> [웹페이지를 표시한다는 것: 브라우저는 어떻게 동작하는가](https://developer.mozilla.org/ko/docs/Web/Performance/How_browsers_work)