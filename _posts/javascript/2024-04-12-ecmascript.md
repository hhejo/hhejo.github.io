---
title: ECMAScript란? ES 버전별 특징도 알아보기
date: 2024-04-12 19:45:13 +0900
last_modified_at: 2024-04-12 19:45:13 +0900
categories: [JavaScript]
tags: [javascript, ecmascript]
---

ECMAScript란 정확히 무엇일까요?

##

### Ecma International

- 정보 통신에 대한 표준을 제정하는 비영리 표준화 기구
- CD롬 볼륨과 파일 구조, C# 언어 규격, JSON 포맷 등 일부 정보 통신 기술에 대한 표준을 관리
- 각각의 표준은 고유한 이름과 번호를 가짐

### ECMA-262

- Ecma International에 의해 제정된 하나의 기술 규격의 이름
- 범용 목적의 스크립트 언어에 대한 명세를 담음
- 스크립트 언어에 대한 표준을 정의한 규칙

### Script Language

- 독립된 시스템에서 작동하도록 특별히 설계된 프로그래밍 언어
- 응용 프로그램과는 독립적이고, 사용자가 직접 프로그램을 의도에 따라 동작시킬 수 있음

### ECMAScript, ES

- Ecma International이 ECMA-262 기술 규격에 따라 정의하고 있는 표준화된 스크립트 프로그래밍 언어
- 자바스크립트를 표준화하기 위해 만들어짐
- 스크립트 언어가 준수해야 하는 규칙, 세부 사항 및 지침을 제공

### JavaScript

- ECMAScript 사양을 준수하는 범용 스크립트 프로그래밍 언어
- ECMAScript의 표준 사양을 가장 잘 구현한 언어로 인정받고 있음
- ES5까지는 대부분의 브라우저에서 기본적으로 지원되었으나, ES6 이후부터는 브라우저 호환성을 위해 트랜스파일러로 컴파일됨

### JavaScript Engine

- 자바스크립트 코드를 이해하고 실행하는 프로그램 또는 인터프리터
- 대체적으로 웹 브라우저에서 사용됨
- Google Chrome의 V8, Mozilla Firefox의 SpiderMonkey, Apple Safari의 WebKit 등

### ECMAScript 6, ES6

- ECMA-262 표준의 제 6판으로, ECMAScript 사양의 주요 변경 사항 및 개선 사항을 명세
- ECMAScript 2015, ES2015라고도 불림

### Babel

- ES6+ 코드를 이전 자바스크립트 엔진에서 실행할 수 있는 이전 버전과 호환되는 자바스크립트 코드로 변환하는 데 주로 사용되는 무료 오픈소스 자바스크립트 트랜스컴파일러(transcompiler)
- 개발자는 Babel을 사용하여 소스 코드를 웹 브라우저가 처리할 수 있는 자바스크립트 버전으로 변환함으로써 새로운 자바스크립트 언어 기능을 사용할 수 있음

### 버전 목록

ES1

- 97년 6월

ES2

- 98년 6월

ES3

- 99년 12월
- `try..catch` 추가

ES4

- 버려짐

ES5

- 09년 12월
- 배열 메서드 추가
- 객체 getter, setter 추가
- JSON 지원
- bind 메서드 지원
- strict mode 추가

ES5.1

- 11년 6월

ES2015 (ES6)

- 15년 6월
- 새로운 개념들이 대거 변경되고 추가된 메이저 업데이트
- 이 버전부터 Babel로 트랜스컴파일 필요
- 템플릿 리터럴
- 화살표 함수
- this
- let, const
- module (import, export)
- class
- 가변 파라미터

ES2016 (ES7)

- 16년 6월
- 제곱 연산자
- 배열 includes 메서드

ES2017 (ES8)

- 17년 6월
- `async/await` 지원
- Object 객체 심화 메서드 추가
- 문자열 단순 편의 기능 추가
- 매개변수 마지막에 콤마 붙이기 허용

ES2018 (ES9)

- 18년 6월
- Object rest, spread
- finally
- asynchronous iteration

ES2019 (ES10)

- 19년 6월

ES2020, ES2021, ES2022, ES2023, ...

- 현재는 ES 뒤에 연도를 붙이는 것을 기본 버전 이름으로 사용
- 매년 6월에 새 규격이 나옴

## 참고

> [JavaScript와 ECMAScript는 무슨 차이점이 있을까?](https://wormwlrm.github.io/2018/10/03/What-is-the-difference-between-javascript-and-ecmascript.html)

> [ECMA스크립트](https://ko.wikipedia.org/wiki/ECMA%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8)

> [스크립트 언어](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%96%B8%EC%96%B4)

> [자바스크립트](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8)

> [자바스크립트 엔진](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%97%94%EC%A7%84)

> [Babel (transcompiler)](<https://en.wikipedia.org/wiki/Babel_(transcompiler)>)

> [ECMAScript 정리](https://jijong.github.io/2022-01-13/ecmascript/)

> [자바스크립트 최신 문법 살펴보기 (ES7-ES13)](https://velog.io/@kyusung/after-es6)
