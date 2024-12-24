---
title: Call by Value, call by Reference
date: 2024-12-23 15:58:33 +0900
last_modified_at: 2024-12-23 15:58:33 +0900
categories: [CS]
tags: [cs, call-by-value, call-by-reference]
---

Call by Value와 call by Reference에 대해 알아보고 차이를 배우기

## 질문

- Call by Value의 특징, 장단점
- Call by Reference의 특징, 장단점
- C와 C++에서의 사용법은?

## Call by Value와 Call by Reference

값에 의한 호출, 참조에 의한 전달(Pass by Reference)

- 함수 호출 시 인수를 전달하는 방식
- 함수 내부에서 인수 값이 어떻게 처리되는지
- 함수 종료 후 원본 데이터가 어떻게 변하는지에 따라 차이가 있음

## Call by Value

값에 의한 호출

- 함수에 값을 복사해 전달하는 방식
- 함수 내부에서는 원본 값의 복사본만을 사용
- 함수 내부에서 해당 값이 변경되어도 원본 데이터에는 영향 없음
- 데이터 크기가 큰 경우, 복사본 생성으로 인해 메모리 사용량이 늘어날 수 있음

## Call by Reference

참조에 의한 호출

- 함수에 변수의 참조(메모리 주소)를 전달하는 방식
- 함수 내부에서 원본 데이터를 직접 참조
- 함수 내부에서 해당 값이 변경되면 원본 데이터에도 영향
- 원본 데이터를 직접 조작 가능
- 복사본을 생성하지 않으므로 효율적

## C에서의 Call by Value, Call by Reference

C는 C++와 다르게 참조자(Reference)가 없기 때문에 포인터를 사용하여 Call by Reference 방식을 구현

- 전달 인자에 주소값(포인터)을 전달해 참조에 의한 전달을 구현

## 출처

> [평가 전략 (컴퓨터 프로그래밍)](<https://ko.wikipedia.org/wiki/%ED%8F%89%EA%B0%80_%EC%A0%84%EB%9E%B5_(%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)>)

> [C++ 강좌 참조자(Reference)의 개념과 함수 활용](https://easycoding91.tistory.com/entry/C-%EA%B0%95%EC%A2%8C-%EC%B0%B8%EC%A1%B0%EC%9E%90Reference%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%ED%95%A8%EC%88%98-%ED%99%9C%EC%9A%A9)

> [씹어먹는 C++ - <2. C++ 참조자(레퍼런스)의 도입>](https://modoocode.com/141)

> [프로그래밍에서의 Call by Value와 Call by Reference 이해하기](https://f-lab.kr/insight/understanding-call-by-value-and-reference)

> [Call by Value 와 Call by Reference](https://velog.io/@kwontae1313/JS%EC%9D%98-Call-by-Value-%EC%99%80-Call-by-Reference)
