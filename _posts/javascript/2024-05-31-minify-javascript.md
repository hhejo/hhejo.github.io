---
title: JavaScript 코드 최소화
date: 2024-05-31 07:22:51 +0900
last_modified_at: 2024-06-02 09:03:16 +0900
categories: [JavaScript]
tags:
  [javascript, minimization, uglification, encryption, obfuscation, compression]
---

JavaScript 최소화와 축소, 암호화, 난독화, 압축

## JavaScript 코드를 축소하는 이유

### JavaScript에서 최소화

최소화(minimization)

- JavaScript 소스 코드에서 기능을 변경하지 않고 불필요한 문자를 모두 제거하는 프로세스
- 공백, 주석, 세미콜론 제거
- 더 짧은 변수 이름, 함수 사용
- 코드를 최소화해 파일 크기 감소
- 웹페이지 로딩 속도를 높여 웹사이트 경험 개선

### 최소화와 축소, 암호화, 난독화, 압축과의 차이

축소(uglification)

- 본질적으로 최소화와 동일
- Uglify JS: JavaScript 파일을 최소화하기 위한 JavaScript 라이브러리
- Uglify로 파일을 축소해 가독성을 높이고 성능 개선

암호화(encryption)

- 일반 데이터라고 하는 데이터를 인코딩된 데이터로 변환
- 암호 텍스트: 인코딩된 데이터
- 암호를 해독하려면 비밀 키 필요
- 브라우저에서는 암호화된 코드 실행 불가
- 보안 기능이며 암호화로 반드시 파일 크기가 줄어들지는 않음

난독화(obfuscation)

- 비즈니스 로직을 숨기기 위해 사용되는 프로세스
- 코드를 수정해 사람이 읽을 수 없게 되어 리버스 엔지니어링이 어려움
- 컴퓨터는 여전히 코드를 이해하고 실행할 수 있어 암호화와 다름
- 변수, 함수, 멤버의 이름을 변경하는 방식으로 이루어짐
- 결과적으로 파일 크기가 줄어들면 성능도 향상되나 이것이 난독화의 주요 목표는 아님

압축(compression)

- 데이터 압축은 데이터를 표현하는 데 필요한 비트 수를 줄이는 프로세스
- 하드 드라이브의 공간 확보, 파일 전송 속도 향상, 네트워크 대역폭 비용 절감

## 참고

[Javascript 코드를 축소하는 이유는?](https://www.cloudflare.com/ko-kr/learning/performance/why-minify-javascript-code/)
