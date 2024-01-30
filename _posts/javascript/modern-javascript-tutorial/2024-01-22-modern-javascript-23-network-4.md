---
title: 모던 JavaScript 튜토리얼 23 - 네트워크 요청 4
date: 2024-01-22 14:25:36 +0900
last_modified_at: 2024-01-30 09:48:49 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, upload, long-polling, websocket]
---

파일 업로드 재개하기, 롱 폴링, 웹 소켓, Server Sent Events

## 파일 업로드 재개하기

## 롱 폴링

폴링

- 웹 소켓이나 server-sent event 같은 특정한 프로토콜을 사용하지 않아도 아주 간단히 서버와 지속적인 커넥션 유지 가능
- 구현이 매우 쉽고 다양한 경우에 사용

### Regular polling

- regular polling을 사용해 서버에서 새로운 정보를 아주 간단히 받을 수 있음
- 10초에 한 번씩 서버에 '안녕하세요. 저는 클라이언트인데 새로운 정보 줄 거 있나요?'라고 요청을 보내는 식
- 이에 대한 응답으로 서버는 먼저 클라이언트가 온라인 상태라는 사실을 스스로 알리고,
- 두 번째로 그 순간까지 받은 메시지 패킷을 보냄
- 효과가 있지만 단점도 있음
  1. 메시지는 요청 사이에 최대 10초의 지연을 두고 전달됨
  2. 메시지가 없더라도 사용자가 다른 곳으로 전환했거나 잠자고 있어도 서버는 10초마다 요청으로 폭격을 받음
     - 성능 측면에서 볼 때 처리하기에는 상당한 부담이 됨
- 서비스 규모가 작은 경우 폴링은 꽤 괜찮은 방식이지만 일반적인 경우에는 개선이 필요

### Long polling

- 일반 폴링보다는 더 나은 방식
- 폴링과 마찬가지로 구현이 쉽지만 지연 없이 메시지를 전달한다는 차이가 있음
- 흐름
  1. 요청이 서버로 전송됨
  2. 서버는 보낼 메시지가 있을 때까지 연결을 닫지 않음
  3. 메시지가 나타나면 서버는 메시지로 요청에 응답
  4. 브라우저는 즉시 새로운 요청을 함
- 브라우저가 요청을 보냈고 서버와의 연결이 보류중인 상황이 이 방법의 표준
- 메시지가 전달된 경우에만 연결이 다시 설정됨
- 네트워크 오류 등으로 인해 연결이 끊어지면 브라우저는 즉시 새 요청을 보냄
- 다음과 같은 클라이언트 측 구독(`subscribe`) 함수는 롱 요청을 가능하게 함

```javascript
async function subscribe() {
  let response = await fetch("/subscribe");

  if (response.status == 502) {
    // 상태 502는 연결 시간 초과(connection timeout) 에러
    // 연결이 너무 오랫동안 보류되어 원격 서버나 프록시가 연결을 닫았을 때 발생할 수 있음
    await subscribe(); // 재접속
  } else if (response.status != 200) {
    showMessage(response.statusText); // 에러 발생 시 보여주기
    await new Promise((resolve) => setTimeout(resolve, 1000)); // 1초 후
    await subscribe(); // 재접속
  } else {
    let message = await response.text(); // 메시지 표시
    showMessage(message);
    await subscribe(); // 다음 메시지를 얻기 위해 다시 subscribe() 호출
  }
}

subscribe();
```

- 롱 폴링을 구현한 함수 `subscribe`는 `fetch`를 사용해 요청을 보내고 응답이 올 때까지 기다린 다음 응답을 처리하고 스스로 다시 요청을 보냄

서버는 보류 중인 많은 연결에 대해 정상(ok)이어야 함

- 서버 아키텍처는 보류 중인 많은 연결과 함께 작동할 수 있어야 함
- 특정 서버 아키텍처는 연결당 하나의 프로세스를 실행하기 때문에 연결된 프로세스만큼 많은 프로세스가 발생하는 반면,
- 각 프로세스는 상당한 양의 메모리를 소비하기 때문에 너무 많은 연결이 있으면 모든 것을 소모하게 됨
- 이는 PHP 및 Ruby와 같은 언어로 작성된 백엔드인 경우가 많음
- Node.js를 사용해 작성된 서버에는 일반적으로 이러한 문제가 없음
- 즉, 프로그래밍 언어 문제는 아님
- PHP 및 Ruby를 포함한 대부분의 최신 언어에서는 적절한 백엔드를 구현할 수 있음
- 많은 동시 연결에서 서버 아키텍처가 제대로 작동하는지 확인하십시오

### Demo: a chat

### Area of usage

- 롱 폴링은 메시지가 드물게 발생하는 상황에서 효과적
- 메시지가 매우 자주 오면 메시지 요청-수신 차트가 톱니 모양이 됨
- 모든 메시지는 헤더, 인증 오버헤드 등과 함께 제공되는 별도의 요청
- 따라서 이 경우 Websocket 또는 Server Sent Events와 같은 다른 방법이 선호됨

## 웹소켓

웹소켓(Websocket)

- RFC 6455 명세서에 정의된 프로토콜
- 서버와 브라우저 간 연결을 유지한 상태로 데이터 교환 가능
- 이때 데이터는 패킷(packet) 형태로 전달되며, 전송은 커넥션 중단과 추가 HTTP 요청 없이 양방향으로 이뤄짐
- 이런 특징 때문에 웹소켓은 데이터 교환이 지속적으로 이뤄져야 하는 서비스에 아주 적합
  - 온라인 게임, 주식 트레이딩 시스템

### 간단한 예시

웹소켓 커넥션 만들기

```javascript
let socket = new WebSocket("ws://javascript.info");
```

- `ws`라는 특수 프로토콜을 사용하는 `new Websocket`을 호출
- `wss://`라는 프로토콜도 있는데 두 프로토콜의 관계는 HTTP와 HTTPS의 관계와 유사
- 소켓이 정상적으로 만들어지면 아래 네 개의 이벤트 사용 가능
  - `open`: 커넥션이 제대로 만들어졌을 때 발생
  - `message`: 데이터를 수신했을 때 발생
  - `error`: 에러가 생겼을 때 발생
  - `close`: 커넥션이 종료되었을 때 발생
- `socket.send(data)`: 커넥션이 만들어진 상태에서 무언가를 보냄

항상 `wss://`를 사용할 것

- `wss://`는 보안 이외에도 신뢰성(reliability) 측면에서 `ws`보다 좀 더 신뢰할만한 프로토콜
- `ws://`를 사용해 데이터를 전송하면 데이터가 암호화되어 있지 않은 채로 전송되기 때문에 데이터가 그대로 노출됨
- 그런데 아주 오래된 프록시 서버는 웹소켓이 무엇인지 몰라서 이상한 헤더가 붙은 요청이 들어왔다고 판단하고 연결을 끊어버림
- 반면 `wss://`는 TLS(전송 계층 보안(Transport Layer Security))이라는 보안 계층을 통과해 전달됨
- 송신자 측에서 데이터가 암호화되고 수신자 측에서 복호화가 이뤄짐
- 따라서 데이터가 담긴 패킷이 암호화된 상태로 프록시 서버를 통과하므로 프록시 서버는 패킷 내부를 볼 수 없게 됨

```javascript
let socket = new WebSocket(
  "wss://javascript.info/article/websocket/demo/hello"
);

socket.onopen = function (e) {
  alert("[open] 커넥션이 만들어졌습니다.");
  alert("데이터를 서버에 전송해봅시다.");
  socket.send("My name is John");
};

socket.onmessage = function (event) {
  alert(`[message] 서버로부터 전송받은 데이터: ${event.data}`);
};

socket.onclose = function (event) {
  if (event.wasClean) {
    alert(
      `[close] 커넥션이 정상적으로 종료되었습니다(code=${event.code} reason=${event.reason})`
    );
  } else {
    // 예시: 프로세스가 죽거나 네트워크에 장애가 있는 경우. event.code가 1006이 됨
    alert("[close] 커넥션이 죽었습니다.");
  }
};

socket.onerror = function (error) {
  alert(`[error]`);
};
```

### 웹소켓 핸드셰이크

- `new Websocket(url)`을 호출해 소켓을 생성하면 즉시 연결이 시작됨
- 커넥션이 유지되는 동안, 브라우저는 (헤더를 사용해) 서버에 '웹소켓을 지원하나요?'라고 물어봄
- 이에 서버가 '네'라는 응답을 하면 서버-브라우저간 통신은 HTTP가 아닌 웹소켓 프로토콜을 사용해 진행됨

```
브라우저                  서버
           HTTP 요청
    ----------------------->
      웹소켓을 지원하나요?

           HTTP 응답
    <-----------------------
              네

        웹소켓 프로토콜
    <---------------------->
```

```
GET /chat
Host: javascript.info
Origin: https://javascript.info
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: Iv8io/9s+lYFgZWcXczP8Q==
Sec-WebSocket-Version: 13
```

- `new WebSocket("wss://javascript.info/chat")`을 호출해 최초 요청을 전송했다고 가정했을 때 요청 헤더
- `Origin`: 클라이언트 오리진을 나타냄
  - 서버는 `Origin` 헤더를 보고 어떤 웹사이트와 소켓 통신을 할 지 결정하기 때문에 중요
  - 웹소켓 객체는 기본적으로 크로스 오리진 요청을 지원
  - 웹소켓 통신만을 위한 전용 헤더나 제약도 없음
  - 오래된 서버는 웹소켓 통신을 지원하지 못하기 때문에 웹소켓 통신은 호환성 문제도 없음
- `Connection: Upgrade`: 클라이언트 측에서 프로토콜을 바꾸고 싶다는 신호를 보냈다는 것을 의미
- `Upgrade: websocket`: 클라이언트 측에서 요청한 프로토콜은 websocket이라는 것을 의미
- `Sec-WebSocket-Key`: 보안을 위해 브라우저에서 생성한 키로, 서버가 웹소켓 프로토콜을 지원하는지 확인하는 데 사용
  - 프록시가 다음 통신을 캐싱하는 것을 방지하는 것은 무작위임
- `Sec-WebSocket-Version`: 웹소켓 프로토콜 버전이 명시됨

웹소켓 핸드셰이크는 모방이 불가능

## Server Sent Events

## 참고

> [네트워크 요청](https://ko.javascript.info/network)
