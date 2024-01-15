---
title: 모던 JavaScript 튜토리얼 22 - 이진 데이터와 파일
date: 2024-01-15 20:36:07 +0900
last_modified_at: 2024-01-15 20:36:07 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

ArrayBuffer, binary arrays, 텍스트 디코더와 텍스트 인코더, Blob, File and FileReader

## ArrayBuffer, binary arrays

바이너리 데이터(binary data)

- 웹 개발에서 우리는 주로 파일을 처리하는 동안(생성, 업로드, 다운로드) 바이너리 데이터를 만남
- 또 다른 일반적인 사용 사례는 이미지 처리
- 이는 모두 자바스크립트에서 가능하며 이진 연산은 고성능
- 다만 클래스가 많아서 헷갈리는 경우도 있음
  - `ArrayBuffer`, `Uint8Array`, `DataView`, `Blob`, `File` 등
- 자바스크립트의 바이너리 데이터는 다른 언어에 비해 비표준 방식으로 구현됨

`ArrayBuffer`

- 기본 바이너리 객체(basic binary object)
- 고정 길이 연속 메모리 영역에 대한 참조

```javascript
let buffer = new ArrayBuffer(16); // 길이 16의 버퍼 생성
alert(buffer.byteLength); // 16
```

- 16 바이트의 연속 메모리 영역을 할당하고 이를 0으로 미리 채움

## 텍스트 디코더와 텍스트 인코더

## Blob

## File and FileReader

## 참고

> [이진 데이터와 파일](https://ko.javascript.info/binary)
