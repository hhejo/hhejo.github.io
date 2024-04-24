---
title: 모던 JavaScript 튜토리얼 23 - 네트워크 요청 2
date: 2024-01-17 08:25:17 +0900
last_modified_at: 2024-04-24 12:34:04 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, cors, safe-request, unsafe-request, preflight, credential]
---

CORS

## CORS

`fetch` 요청 실패

- `fetch`로 요청을 보내게 될 사이트가 현재 접속 사이트와 다르다면 요청이 실패할 수 있음
- 요청이 왜 실패하는지 알기 위해서는 오리진(origin)이라는 핵심 개념을 알아야 함
- 오리진: 도메인·프로토콜·포트 세 가지에 의해 결정됨

```javascript
try {
  await fetch("http://example.com");
} catch (err) {
  alert(err); // TypeError: Failed to fetch
}
```

Cross-Origin Request

- 크로스 오리진 요청, 교차 출처 요청
- 도메인이나 서브도메인, 프로토콜, 포트가 다른 곳에 요청을 보내는 것
- 크로스 오리진 요청을 보내려면, 리모트 오리진에서 전송받은 특별한 헤더가 필요
- 이러한 정책을 CORS라고 부름
  - Cross-Origin Resource Sharing, 크로스 오리진 리소스 공유

### 왜 CORS가 필요한가에 대한 짧은 역사

CORS는 악의를 가진 해커로부터 인터넷을 보호하기 위해 만들어짐

- 과거 수 년 동안, 한 사이트의 스크립트에서 다른 사이트에 있는 컨텐츠에 접근할 수 없다는 제약이 있었음
- 이러한 간단하지만 강력한 규칙은 인터넷 보안을 위한 근간
- 보안 규칙 덕분에 해커가 만든 웹 사이트 `hacker.com`에서 `gmail.com`에 있는 메일 박스에 접근할 수 없던 것
- 사람들은 이런 제약 덕분에 안전하게 인터넷을 사용할 수 있었음

그런데 이 당시의 자바스크립트는 네트워크 요청을 보낼 수 있을 만한 메서드를 지원하지 않았음

- 당시 자바스크립트는 웹 페이지를 꾸미기 위한 토이 랭귀지 수준
- 하지만 많은 웹 개발자들이 강력한 기능을 원하기 시작하면서 위와 같은 제약을 피해 다른 웹 사이트에 요청을 보내기 위한 트릭들을 만들기 시작함

`<form>` 사용하기

- 트릭 중 하나로, 개발자들은 `<form>` 안에 `<iframe>`을 넣어 `<form>`을 전송해 현재 사이트에 남아있으면서 네트워크 요청을 전송
- 이 당시에는 네트워크 관련 메서드가 없었지만 폼은 어디든 데이터를 보낼 수 있다는 특징을 이용해 폼으로 다른 사이트에 GET, POST 요청을 보냈었음
- 하지만 다른 사이트에서 `<iframe>`에 있는 컨텐츠를 읽는 것은 금지되었기 때문에 응답을 읽는 것은 불가능했음
- 개발자들은 iframe과 페이지 양쪽에 특별한 스크립트를 심어 이런 제약 역시 피할 수 있는 트릭을 만들었음
- 이렇게 iframe을 사용한 트릭은 오리진이 다른 사이트 간에도 양방향 통신이 가능하도록 했음

```html
<!-- 폼 target -->
<iframe name="iframe"></iframe>
<!-- 자바스크립트를 사용해 폼을 동적으로 생성하고 보냄-->
<form target="iframe" method="POST" action="http://another.com/...">...</form>
```

스크립트 사용하기

- 제약을 피하기 위한 또 다른 트릭으로, `script` 태그의 `src` 속성값에는 도메인 제약이 없기 때문에 이 특징을 사용하면 어디서든 스크립트를 실행할 수 있음
- `script` 태그를 이용해 `<script src="http://another.com/...">` 형태로 `another.com`에 데이터를 요청하게 되면,
- JSONP(JSON with Padding)라 불리는 프로토콜을 사용해 데이터를 가져오게 됨
- 이런 꼼수를 쓰면 보안 규칙을 어기지 않으면서도 양방향으로 데이터를 전달할 수 있음. 양쪽에서 동의한 상황이라면 해킹도 아님
- 아직도 이런 방식을 사용해 통신을 하는 서비스가 있음. 이 방식은 오래된 브라우저도 지원
- 그러던 와중에 브라우저측 자바스크립트에도 네트워크 관련 메서드가 추가됨
- 처음 네트워크 요청 메서드가 등장했을 때에는 크로스 오리진 요청이 불가능했지만 긴 논의 끝에 크로스 오리진 요청을 허용하기로 결정
- 대신 크로스 오리진 요청은 서버에서 명시적으로 크로스 오리진 요청을 허가했다는 것을 알려주는 특별한 헤더를 전송받았을 때만 가능하도록 제약을 받게 됨

```javascript
// 1. 날씨 데이터를 처리하는데 사용되는 함수를 선언
function gotWeather({ temperature, humidity }) {
  alert(`temperature: ${temperature}, humidity: ${humidity}`);
}
// 2.
let script = document.createElement("script");
script.src = `http://another.com/weather.json?callback=gotWeather`;
document.body.append(script);
// 3. 서버
gotWeather({ temperature: 25, humidity: 78 });
```

0. 날씨 정보가 저장되어 있는 `http://another.com`에서 데이터를 가져와야 한다고 가정
1. 먼저 서버에서 받아온 데이터를 소비하는 전역 함수 `gotWeather`를 선언
2. `src="http://another.com/weather..."`를 속성으로 갖는 `<script>` 태그를 만듦
   - 1에서 만든 함수를 URL 매개변수 `callback`의 값으로 사용
3. 리모트 서버 `another.com`은 클라이언트에서 필요한 날씨 데이터와 함께 `gotWeather(...)`를 호출하는 스크립트를 동적으로 생성
4. 리모트 서버에서 받아온 스크립트가 로드 및 실행되면 `gotWeather`가 호출됨
   - `gotWeather`는 현재 페이지에서 만든 함수이고 서버에서 받은 데이터도 있기 때문에 원하는 결과를 얻을 수 있음

### 안전한 요청

크로스 오리진 요청은 크게 두 가지 종류로 구분됨

- 안전한 요청(safe request)
  - 안전한 메서드, 안전한 헤더를 만족하는 요청
  - 그 외의 요청 대비 만들기 쉬움
- 그 외의 요청(안전한 요청이 아닌 요청)

안전한 요청은 다음과 같은 두 가지 조건 모두를 충족하는 말 그대로 안전한 요청

- 안전한 메서드(safe method): `GET`, `POST`, `HEAD`를 사용한 요청
- 안전한 헤더(safe header): 다음 목록에 속하는 헤더
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type`: 값이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`
- 두 조건을 모두 충족하지 않는 요청은 안전하지 않은(unsafe) 요청으로 취급됨
  - `PUT` 메서드
  - 헤더에 `API-Key`가 명시된 요청 등

안전한 요청과 그렇지 않은 요청의 근본적인 차이

- 특별한 방법을 사용하지 않고도 `<form>`이나 `<script>`를 사용해 요청을 만들 수 있음
- 아주 오래된 웹 서버라도 안전한 요청은 당연히 처리할 수 있어야 하는 것
- 표준이 아닌 헤더가 들어있거나 안전하지 않은 메서드(`DELETE` 등)를 사용한 요청은 안전한 요청이 될 수 없음
- 아주 오래전에는 자바스크립트를 사용해 이런 요청을 보내는 것이 불가능했음
- 따라서 연식이 오래된 서버는 이런 요청을 받으면 '웹 페이지는 이런 요청을 보낼 수 없었기 때문에' 뭔가 특별한 곳에서 요청이 왔을 거라 해석했었음
- 그런데 시간이 지나고 개발자가 자바스크립트를 사용해 안전하지 않은 요청을 보낼 수 있게 되자, 브라우저는 안전하지 않은 요청을 서버에 전송하기 전에 'preflight' 요청을 먼저 전송해 '서버가 크로스 오리진 요청을 받을 준비가 되어있는지를 확인'함
- 이때 서버에서 크로스 오리진 요청은 허용하지 않는다는 정보를 담은 헤더를 브라우저에 응답하면 안전하지 않은 요청은 서버로 전송되지 않음

### CORS와 안전한 요청

`https://javascript.info/page`에서 `https://anywhere.com/request`에 요청을 보낸다고 가정

- 크로스 오리진 요청을 보낼 경우 브라우저는 항상 `Origin`이라는 헤더를 요청에 추가함
- 헤더는 다음과 같은 형태가 됨

```
GET /request
Host: anywhere.com
Origin: https://javascript.info
...
```

- `Origin` 헤더에는 요청이 이뤄지는 페이지 경로(/page)가 아닌 오리진(도메인·프로토콜·포트) 정보가 담기게 됨
- 서버는 요청 헤더에 있는 `Origin`을 검사하고,
- 요청을 받아들이기로 동의한 상태라면 특별한 헤더 `Access-Control-Allow-Origin`을 응답에 추가
- 이 헤더에는 허가된 오리진(위 예시에서는 `https://javascript.info`)에 대한 정보나 `*`이 명시됨
- 이때 응답 헤더 `Access-Control-Allow-Origin`에 오리진 정보나 `*`이 들어있으면 응답은 성공하고 그렇지 않으면 실패하게 됨

이 과정에서 브라우저는 중재인의 역할을 함

1. 브라우저는 크로스 오리진 요청 시 `Origin`에 값이 제대로 설정, 전송되었는지 확인
2. 브라우저는 서버로부터 받은 응답에 `Access-Control-Allow-Origin`이 있는지를 확인해서 서버가 크로스 오리진 요청을 허용하는지 아닌지를 확인
   - 응답 헤더에 `Access-Control-Allow-Origin`이 있다면 자바스크립트를 사용해 응답에 접근할 수 있고 아니라면 에러가 발생

```
JavaScript                              Browser                              Server

     -fetch()->
                                                       -HTTP request->
                                               Origin: https://javascript.info
     <-if the header allows, then success-
                                                       <-HTTP response-
                                                Access-Control-Allow-Origin: *
                                                 (or https://javascript.info)
```

- 서버에서 크로스 오리진 요청을 허용한 경우, preflight 요청에 대한 응답은 다음과 같은 형태

```
200 OK
Content-Type: text/html; charset=UTF-8
Access-Control-Allow-Origin: https://javascript.info
```

### 응답 헤더

- 크로스 오리진 요청이 이뤄진 경우, 자바스크립트는 기본적으로 '안전한' 응답 헤더로 분류되는 헤더에만 접속 가능

안전한 응답 헤더는 다음과 같음

- `Cache-Control`
- `Content-Language`
- `Content-Type`
- `Expires`
- `Last-Modified`
- `Pragma`
- 이외의 응답 헤더에 접근하면 에러 발생
- 위 리스트에 `Content-Length` 헤더는 없음
  - `Content-Length`는 응답 본문 크기 정보를 담고 있는 헤더
  - 무언가를 다운로드하는데, 다운로드가 몇 퍼센트나 진행되었는지 확인하려면 이 헤더에 접근할 수 있어야 함
  - 그런데 이 헤더에 접근하려면 특별한 권한이 필요

`Access-Control-Expose-Headers`

- 자바스크립트로 안전하지 않은 응답 헤더에 접근하려면 서버에서 `Access-Control-Expose-Headers`라는 헤더를 보내줘야만 함
- `Access-Control-Expose-Headers`에는 자바스크립트 접근을 허용하는 안전하지 않은 헤더 목록이 담겨 있고 각 헤더는 콤마로 구분됨
- `Access-Control-Expose-Headers` 헤더가 응답 헤더에 있어야 자바스크립트로 응답 헤더의 `Content-Length`와 `API-Key`를 읽을 수 있음

```
200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 12345
API-Key: 2c9de507f2c54aa1
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Expose-Headers: Content-Length,API-Key
```

### 안전하지 않은 요청

- 요즘에는 요청에 `GET`, `POST`뿐만 아니라 `PATCH`, `DELETE` 메서드도 사용 가능
- 그런데 과거에는 웹 페이지에서 `GET`, `POST` 이외의 HTTP 메서드를 사용해 요청을 보낼 수 있을 거라는 상상조차 할 수 없었음
- 아직까지도 이런 메서드를 다룰 수 없는 웹 서버도 꽤 있음
- 이런 서버들은 `GET`, `POST` 이외의 메서드를 사용한 요청이 오면 '이건 브라우저가 보낸 요청이 아니야'라고 판단하고 접근 권한을 확인함
- 이런 혼란스러운 상황을 피하고자 브라우저는 안전하지 않은 요청이 이뤄지는 경우, 서버에 바로 요청을 보내지 않고 'preflight' 요청이라는 사전 요청을 서버에 보내 권한지 있는지 확인함

preflight 요청

- `OPTIONS` 메서드를 사용하고, 본문은 비어있고 두 헤더가 함께 들어감
- `Access-Control-Request-Method` 헤더: 안전하지 않은 요청에서 사용하는 메서드 정보가 담김
- `Access-Control-Request-Headers` 헤더: 안전하지 않은 요청에서 사용하는 헤더 목록이 담김. 각 헤더는 쉼표로 구분됨

안전하지 않은 요청을 허용하기로 합의한 경우

- 서버는 본문이 비어있고 상태 코드가 200인 응답을 다음과 같은 헤더와 함께 브라우저로 보냄
- `Access-Control-Allow-Origin`: `*`이나 요청을 보낸 오리진이어야 함
- `Access-Control-Allow-Method`: 허용된 메서드 정보가 담김
- `Access-Control-Allow-Headers`: 허용된 헤더 목록이 담김
- `Access-Control-Max-Age`: 퍼미션 체크 여부를 몇 초간 캐싱해 놓을지를 명시
  - 이렇게 퍼미션 정보를 캐싱해 놓으면 브라우저는 일정 기간 동안 preflight 요청을 생략하고 안전하지 않은 요청을 보낼 수 있음

```
JavaScript                                   Browser                                   Server

     -fetch()->                              [ Preflight ]
                                             -OPTIONS->
                                             Origin                                              (1)
                                             Access-Control-Request-Method
                                             Access-Control-Request-Headers

                                                                   <-200 OK-
                                                                   Access-Control-Allow-Origin
                                                                   Access-Control-Allow-Method   (2)
                                                                   Access-Control-Allow-Headers
                                                                   Access-Control-Max-Age
                                             [ if allowed ]
                                             -Main HTTP request->                                (3)
                                             Origin

                                                                   <-Main HTTP response-         (4)
                                                                   Access-Control-Allow-Origin

     <-if allowed: success, otherwise error-
```

실제 안전하지 않은 크로스 오리진 요청이 어떻게 이뤄지는지 예시

```javascript
let response = await fetch("https://site.com/service.json", {
  method: "PATCH", // 안전하지 않은 요청으로 분류되는 이유 1
  headers: {
    "Content-Type": "application/json", // 안전하지 않은 요청으로 분류되는 이유 2
    "API-Key": "secret" // 안전하지 않은 요청으로 분류되는 이유 3
  }
});
```

0. 안전하지 않은 요청으로 분류되는 이유 세 가지 존재
1. `PATCH` 메서드 사용
2. `Content-Type`이 `application/x-www-form-urlencoded`나 `multipart/form-data`, `text/plain`이 아님
3. 비표준 헤더 `API-Key`를 사용

1단계: preflight 요청

```
OPTIONS /service.json
Host: site.com
Origin: https://javascript.info
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Content-Type,API-Key
```

- 본 요청을 보내기 전에 브라우저는 자체적으로 위와 같은 preflight 요청을 보냄
- 메서드: `OPTIONS`
- 경로: 본 요청과 동일한 경로(`/service.json`)
- 크로스 오리진 특수 헤더
  - `Origin`: 본 요청의 오리진
  - `Access-Control-Request-Method`: 본 요청에서 사용하는 메서드
  - `Access-Control-Request-Headers`: 본 요청에서 사용하는 안전하지 않은 헤더 목록(콤마로 구분)

2단계: preflight 응답

- 서버는 상태 코드 200과 함께 다음과 같은 헤더를 담은 응답을 보냄
- `Access-Control-Allow-Methods: PATCH`
- `Access-Control-Allow-Headers: Content-Type,API-key`
- 이렇게 응답이 와야 비로소 본 요청을 보낼 수 있음. 그렇지 않으면 에러 발생
- 미래에 `PATCH` 이외의 메서드와 다양한 헤더를 허용할 수 있도록 세팅하려면 `Access-Control-Allow-Methods`와 `Access-Control-Allow-Headers`에 원하는 메서드와 헤더를 추가해 놓으면 됨

```
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Methods: PUT,PATCH,DELETE
Access-Control-Allow-Headers: API-Key,Content-Type,If-Modified-Since,Cache-Control
Access-Control-Max-Age: 86400
```

- 이렇게 서버에서 preflight 응답이 오면 브라우저는 `Access-Control-Allow-Method`에 `PATCH`가 있는 것을 확인하고,
- 이어서 `Access-Control-Allow-Headers`에 `Content-Type`과 `API-Key`가 있는 것을 확인함
- 둘 다 있는 것을 확인했기 때문에 이제 브라우저는 본 요청을 서버에 보냄
- 참고로 `Access-Control-Max-Age` 헤더가 응답으로 오면 preflight 허용 여부가 헤더와 함께 캐싱되기 때문에 브라우저는 헤더 값에 명시한 초동안 preflight 요청을 보내지 않음
- 위 예시처럼 응답이 온 경우에는 하루동안 preflight 요청을 전송하지 않고 바로 본 요청이 전송됨

3단계: 실제 요청

- preflight 요청이 성공적으로 이뤄진 후에야 브라우저는 본 요청을 보냄
- 지금부터의 프로세스는 안전한 요청이 이뤄질 때의 절차와 동일함
- 본 요청은 크로스 오리진 요청이기 때문에 `Origin` 헤더가 붙음

```
PATCH /service.json
Host: site.com
Content-Type: application/json
API-Key: secret
Origin: https://javascript.info
```

4단계: 실제 응답

- 서버에서는 본 응답에 `Access-Control-Allow-Origin` 헤더를 반드시 붙여줘야 함
- preflight 요청이 성공했더라도

```
Access-Control-Allow-Origin: https://javascript.info
```

- 이 모든 과정이 끝나야 자바스크립트를 사용해 실제 응답을 읽을 수 있음

preflight 요청

- preflight 요청은 '무대 밖에서' 일어나기 때문에 자바스크립트를 사용해 관찰할 수는 없음
- 자바스크립트로는 본 요청의 응답을 받는 일만 할 수 있음
- 서버에서 크로스 오리진 요청을 허용하지 않는 경우에는 에러가 발생

### 자격 증명

- 자바스크립트로 크로스 오리진 요청을 보내는 경우, 기본적으로 쿠키나 HTTP 인증 같은 자격 증명(credential)이 함께 전송되지 않음
- HTTP 요청의 경우 대개 쿠키가 함께 전송되는데, 자바스크립트를 사용해 만든 크로스 오리진 요청은 예외
- 따라서 `fetch('http://another.com')`를 사용해 요청을 보내도 `another.com` 관련 쿠키가 함께 전송되지 않음
- 이런 예외가 생긴 이유는 자격 증명과 함께 전송되는 요청의 경우 영향력이 강하기 때문
- 크로스 오리진 요청 시 자격 증명을 함께 전송할 수 있으면 사용자 동의 없이 자바스크립트로 민감한 정보에 접근할 수 있게 됨
- 그럼에도 불구하고 서버에서 이를 허용하고 싶다면, 자격 증명이 담긴 헤더를 명시적으로 허용하겠다는 세팅을 서버에 해줘야 함
- `fetch` 메서드에 자격 증명 정보를 함께 전송하려면 다음과 같이 `credentials: "include"` 옵션을 추가하면 됨

```javascript
fetch("http://another.com", { credentials: "include" });
```

- 이렇게 옵션을 추가하면 `fetch`로 요청을 보낼 때 `another.com`에 대응하는 쿠키가 함께 전송됨
- 자격 증명 정보가 담긴 요청을 서버에서 받아들이기로 동의했다면,
- 서버는 응답에 `Access-Control-Allow-Origin` 헤더와 함께 `Access-Control-Allow-Credentials: true` 헤더를 추가해서 보냄

```
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Credentials: true
```

- 자격 증명이 함께 전송되는 요청을 보낼 때는 `Access-Control-Allow-Origin`에 `*`을 쓸 수 없음
- 위 예시에서처럼 `Access-Control-Allow-Origin`에는 정확한 오리진 정보만 명시되어야 함
- 이런 제약이 있어야 어떤 오리진에서 요청이 왔는지에 대한 정보를 서버가 신뢰할 수 있기 때문

### 요약

브라우저 관점에서의 크로스 오리진 요청

- 안전한 크로스 오리진 요청과 그렇지 않은 크로스 오리진 요청으로 나뉨
- 안전한 요청은 다음 조건을 모두 충족하는 요청
  - 메서드: `GET`, `POST`, `HEAD`
  - 헤더: `Accept`, `Accept-Language`, `Content-Language`
  - `Content-Type` 값이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`
- 안전한 요청은 아주 오래전부터 `<form>`이나 `<script>` 태그를 사용해도 가능했던 요청
- 안전하지 않은 요청은 브라우저에서는 보낼 수 없었던 요청

실무 관점에서 두 요청의 차이

- 안전한 요청은 `Origin` 헤더와 함께 바로 요청이 전송됨
- 안전하지 않은 요청은 브라우저에서 본 요청이 이뤄지기 전에 preflight 요청이라 불리는 사전 요청을 보내 퍼미션 여부를 물음

안전한 요청은 다음과 같은 절차를 따름

- -> 오리진 정보가 담긴 `Origin` 헤더와 함께 브라우저가 요청을 전송
- <- 자격 증명이 없는 요청의 경우(기본), 서버는 아래와 같은 응답을 전송
  - `Origin` 값과 동일하거나 `*`인 `Access-Control-Allow-Origin`
- <- 자격 증명이 있는 요청의 경우, 서버는 아래와 같은 응답을 전송
  - `Origin` 값과 동일한 `Access-Control-Allow-Origin`
  - 값이 `true`인 `Access-Control-Allow-Credentials`

자바스크립트를 사용해 `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, `Pragma`를 제외한 응답헤더에 접근하는 경우

- 응답 헤더의 `Access-Control-Expose-Headers`에 접근을 허용하는 헤더가 명시돼 있어야 함

안전하지 않은 요청의 절차는 다음과 같음

- -> 브라우저는 동일한 URL에 `OPTIONS` 메서드를 사용한 preflight 요청을 전송. 이때 헤더에 다음과 같은 정보가 들어감
  - `Access-Control-Request-Method`: 본 요청의 메서드 정보가 담김
  - `Access-Control-Request-Headers`: 본 요청의 헤더 정보가 담김
- <- 서버는 상태 코드 200과 아래와 같은 헤더를 담은 응답을 전송
  - `Access-Control-Allow-Method`: 허용되는 메서드 목록이 담김
  - `Access-Control-Allow-Headers`: 허용되는 헤더 목록이 담김
  - `Access-Control-Max-Age`: 몇 초간 preflight 요청 없이 크로스 오리진 요청을 바로 보낼지에 대한 정보가 담김
- 이후에는 본 요청이 전송되고 절차는 안전한 요청과 동일

### 예제

왜 오리진이 필요할까요

- HTTP 헤더를 구성하는 `Referer`는 주로 네트워크 요청을 처음 시작한 페이지의 url 정보를 담음
- `http://javascript.info/some/url`에서 `http://google.com`에 요청을 보냈다고 가정한 헤더

```
Accept: */*
Accept-Charset: utf-8
Accept-Encoding: gzip,deflate,sdch
Connection: keep-alive
Host: google.com
Origin: http://javascript.info
Referer: http://javascript.info/some/url
```

- 헤더에 `Referer`와 `Origin` 정보가 모두 들어감
- `Referer`에 더 많은 정보가 있는데 왜 `Origin`을 따로 두었는지
- 헤더에 `Referer`나 `Origin`이 없을 수도 있는데 이런 경우에는 어떤 일이 발생하는지
- 간혹 `Referer`가 없는 경우가 있기 때문에 `Origin`이 필요
- HTTPS 프로토콜을 사용해 HTTP 페이지를 요청하는 경우같이 보안 수준이 높은 방법을 사용해 보안 수준이 낮은 페이지에 접근할 때는 `Referer`가 없음
- 이 외에도 컨텐츠 보안 정책이 적용돼 `Referer` 정보가 누락되는 경우도 있음
- `fetch`에는 `Referer` 전송을 막는 옵션이 있음
- 동일한 사이트 내에서 `Referer` 수정을 허용하는 옵션도 있음
- 명세서에 따르면 `Referer`는 HTTP 헤더에서 선택 사항
- 이렇게 `Referer`는 신뢰할 수 없는 정보를 담고 있을 확률이 높기 때문에 명세서에 `Origin`이 추가됨
- 크로스 요청 시 브라우저는 `Origin`이 제대로 전송되는 것을 보장

## 참고

> [네트워크 요청](https://ko.javascript.info/network)
