---
title: Node.js란?
date: 2023-07-16 00:00:00 +0900
last_modified_at: 2024-07-15 09:45:29 +0900
categories: [JavaScript]
tags: [javascript, nodejs]
---

Node.js란 무엇일까요?

### Question

- Node.js란?
- Node.js의 작동 방식
- Node.js와 브라우저의 차이

## Node.js

Node.js

- 구글 크롬의 V8 JavaScript 엔진으로 빌드된 자바스크립트 런타임 환경(runtime environment)
- 브라우저 밖에서 자바스크립트를 사용할 수 있게 함
- 이를 통해 여러 자바스크립트 애플리케이션 실행 가능
  - 서버, 데스크탑 앱 등
- 이벤트 중심의 비차단(non-blocking) 입/출력 모델을 사용
- 단일 스레드(single thread) 이벤트 루프(event loop)에서 작동

특징

- Node.js 앱은 모든 요청에 대해 새로운 스레드를 생성하지 않고 단일 프로세스에서 실행
- 또한 표준 라이브러리에서 자바스크립트 코드가 차단되지 않도록(non-blocking) 하는 비동기(asynchronous) I/O primitive 세트를 제공
- Node.js가 네트워크에서 읽기, DB 또는 파일 시스템 접근과 같은 I/O 작업을 수행할 때, 스레드를 차단하고 대기중인 CPU 주기를 낭비하는 대신 응답이 돌아오면 작업을 재개
- 이를 통해 스레드 동시성 관리 부담 없이 단일 서버에서 수천 개의 동시 연결 처리 가능

## 사용하는 이유

1. JavaScript 기반
2. Scalability
3. Extensibility
4. Availability
5. Self-sufficiency
6. Universality
7. Simplicity
8. Automation
9. High performance, speed, and resource-efficiency
10. Community support

### 1. API

개발자가 자바스크립트로 애플리케이션 개발 범위를 제공하면서 실시간 애플리케이션을 작성할 수 있음

### 2. Streaming web applications

막대한 양의 데이터를 chunk 단위로 순차적으로 전송할 수 있는 빌트인 스트림 모듈이 있어 음악을 듣거나 동영상을 시청하는 등의 스트리밍 서비스에 적합

### 3. Real-time web applications

이벤트 루프 API 및 WebSockets 덕분에 채팅, 화상 회의실, 동시 문서 편집 협업 도구와 같은 실시간 웹 애플리케이션 구축 가능

### 4. Microservices

독립적으로 작동하는 소규모 서비스 모음 애플리케이션을 쉽게 작성 가능

## Node.js와 브라우저의 차이

브라우저와 Node.js는 둘 다 자바스크립트를 프로그래밍 언어로 사용

하지만 다른 점이 있음

- 브라우저에서는 DOM, 쿠키와 같은 웹 플랫폼 API와 상호 작용
- Node.js에는 `document`, `window` 객체가 없고 브라우저에서 제공하는 다른 객체도 없음
- 브라우저에는 파일 시스템 엑세스 기능과 같은 Node.js가 모듈을 통해 제공하는 API가 없음

## 참고

> [Node.js®에 대해서](https://nodejs.org/ko/about)

> [What is Node.js?](https://www.w3schools.com/nodejs/nodejs_intro.asp)

> [Introduction to Node.js](https://nodejs.dev/en/learn/)

> [Node.js is a great runtime environment - and here's why you should use it](https://www.freecodecamp.org/news/what-are-the-advantages-of-node-js/)

> [Why Choose Node.js?](https://medium.com/selleo/why-choose-node-js-b0091ad6c3fc)

> [Differences between Node.js and the Browser](https://nodejs.dev/en/learn/differences-between-nodejs-and-the-browser/)
