---
title: JavaScript의 깊은 복사, structuredClone
date: 2024-04-22 12:04:33 +0900
last_modified_at: 2024-04-22 12:04:33 +0900
categories: [JavaScript]
tags: [javascript, shallow-copy, deep-copy, structuredclone]
---

JavaScript에서 기존에 깊은 복사를 했던 방법을 살펴보고, structuredClone API를 사용하는 새로운 방법을 알아보겠습니다.

## 얕은 복사

JavaScript에서 값을 복사하는 것은 거의 항상 얕은 복사

- 깊이 중첩된 값에 대한 변경 사항은 복사본뿐만 아니라 원본에도 영향

spread 연산자 `...`를 사용해 얕은 복사 가능

- 기본형(primitive value, primitive data type)이 아닌 값은 참조로 처리됨
- primitive: `string`, `number`, `bigint`, `boolean`, `symbol`, `undefined`, `null`

```javascript
const myOriginal = {
  someProp: "with a string value",
  anotherProp: {
    withAnotherProp: 1,
    andAnotherProp: true
  }
};
const myShallowCopy = { ...myOriginal }; // ...를 사용한 얕은 복사
```

```javascript
// 복사본에만 영향
myShallowCopy.aNewProp = "a new value";
console.log(myOriginal.aNewProp); // undefined
// 복사본, 원본 모두 영향
myShallowCopy.anotherProp.aNewProp = "a new value";
console.log(myOriginal.anotherProp.aNewProp); // a new value
```

## 깊은 복사

다른 객체에 대한 참조를 찾으면, 자신을 재귀적으로 호출해 해당 객체의 복사본도 생성

JavaScript에서 깊은 복사본을 만드는 쉽고 좋은 방법은 없었음

- Lodash의 `cloneDeep()` 기능 같은 라이브러리에 의존
- `JSON` 기반 해결법도 가능

JSON을 사용한 해결법

```javascript
const myDeepCopy = JSON.parse(JSON.stringify(myOriginal));
```

몇 가지 단점 존재

- 재귀 데이터 구조
  - 재귀적인 데이터 구조로 `JSON.stringify()`를 사용하면 예외 에러(throw)가 발생
  - 연결 리스트나 트리 데이터 구조를 작업할 때 자주 발생
- 내장 함수 유형
  - `JSON.stringify()`는 JavaScript 내장 함수인 `Map`, `Set`, `Date`, `RegExp`, `ArrayBuffer`를 포함한 값이 있을 경우 예외 에러(throw)가 발생
- 함수
  - `JSON.stringify()`는 빠르게 함수를 폐기함

## Structured Clone

이미 여러 곳에서 JavaScript의 깊은 복사본을 생성하는 기능이 요구됨

- indexedDB에 JavaScript 값을 저장하려면 디스크에 저장하고 나중에 값을 복원하기 위해 역직렬화할 수 있도록 일종의 직렬화가 필요한 상황
- `postMessage()`를 통해 WebWorker에 메시지를 보내는 상황(JS 값을 한 JS 영역에서 다른 영역으로) 등

`structuredClone()`

```javascript
const myDeepCopy = structuredClone(myOriginal);
```

## 기능 및 제한사항

Structured Cloning은 `JSON.stringify()`의 많은 단점을 해결

- 순환되는 데이터 구조 처리
- 내장 데이터 타입 지원
- 일반적으로 더 강력하고 더 빠름

그러나 몇 가지 제한사항이 존재

- 프로토타입
  - `structuredClone`을 클래스 인스턴스와 함께 사용하는 경우, structured cloning이 객체의 프로토타입 체인을 폐기
  - 따라서 반환값은 일반 객체가 됨
- 함수
  - 객체에 함수가 포함되어 있으면 폐기됨
- 복제 불가
  - 일부 값, 특히 `Error`와 DOM 노드는 구조화된 복제가 불가능
  - 이것은 `structuredClone()` 예외 에러(throw)의 원인이 됨

## 참고

> [StructuredClone에 대해 꼭 알아두세요! JavaScript에서 깊은 복사가 가능해져요.](https://velog.io/@seonja/StructuredClone을-사용하여-JavaScript에서-깊은-복사)

> [structuredClone()으로 깊은 복사하기](https://velog.io/@sejinkim/structuredClone으로-깊은-복사하기)

> [(번역) StructuredClone API를 사용하여 객체를 깊은 복사하는 법](https://soobing.github.io/javascript/deep-copying-objects-with-the-structuredclone-api/)
