---
title: Express.js의 응답 res.send(), res.json(), res.end()의 차이점
date: 2023-08-04 00:00:00 +0900
last_modified_at: 2023-01-01 12:53:16 +0900
categories: [Express]
tags: [express, nodejs]
---

세 메서드는 각각 어떤 차이가 있을까요?

## Express.js의 응답

Express를 사용하여 응답을 보내는 방법에는 여러 가지가 있음

```javascript
app.get("/api/test", (req, res) => {
  // ... do something ...
});
```

`res` 객체

- Node.js에 있는 응답 객체의 향상된 버전

## res.send()

```javascript
res.send([body]);
```

- 전달된 인수를 기반으로 응답 헤더(response header)의 `Content-Type`을 자동으로 설정
- `body` 파라미터는 Buffer 객체, 문자열, 객체, 불리언, 배열이 될 수 있음
- JSON 데이터 전송을 위해 body에 객체를 넣으면, 응답 헤더의 `Content-Type`은 올바르게 `application/json`이 설정됨
- `res.set()`으로 따로 설정해 자동으로 설정하는 것을 방지 가능

```javascript
res.send({ hello: "world" }); // Content-Type: application/json; charset=utf-8
```

```javascript
res.set("Content-Type", "text/html");
res.send(Buffer.from("<p>some html</p>"));
```

## res.json()

```javascript
res.json([body]);
```

- JSON 응답을 전송하는 메서드

`res.send()`와 `res.json()`의 주요 차이점

- `res.json()`
  - 유효한 JSON이 아닌 비 객체(null, undefined 등)도 변환
  - json replacer와 json space 설정을 사용
- `res.send()`
  - 유효한 JSON이 아닌 비 객체를 변환하지 않음

## res.end()

```javascript
res.status(404).end();
```

- 데이터를 제공하지 않고 응답을 종료하는 경우 사용 가능
- 404 페이지에 유용
- 일부 응답 데이터를 보내고 싶다면 `res.send()`, `res.json()` 등을 사용
  - 둘은 응답을 종료하기도 하기 때문에 `res.end()`를 명시적으로 호출할 필요가 없음

## 결론

응답에 데이터를 보낼 때, `res.send()` 혹은 `res.json()`을 사용

`res.send()`

- 담긴 응답 데이터의 종류에 따라 응답 헤더의 `Content-Type`이 다르게 설정됨
- JSON 데이터도 문제 없이 전송 가능

`res.json()`

- JSON 데이터를 명시적으로 보내고 싶을 때 사용
- 응답 헤더의 `Content-Type`을 자동으로 `application/json`으로 변경

`res.end()`

- 보낼 데이터는 없지만 응답을 종료하고 싶은 경우 사용
- `res.send()`나 `res.json()`으로 응답을 전송하는 경우에는 사용할 필요가 없음

## 참고

> [Difference between res.json vs res.send vs res.end in Express.js](https://medium.com/gist-for-js/use-of-res-json-vs-res-send-vs-res-end-in-express-b50688c0cddf)

> [Express res.json과 res.send 비교](https://haeguri.github.io/2018/12/30/compare-response-json-send-func/)

> [Express res.send() vs res.json() vs res.end() 비교](https://yohanpro.com/posts/nodejs/express-response)

> [res.json() vs res.send() vs res.end() in Express](https://tpiros.dev/blog/res-json-vs-res-send-vs-res-end-in-express/)

> [res.send([body])](https://expressjs.com/en/4x/api.html#res.send)

> [res.json([body])](https://expressjs.com/en/4x/api.html#res.json)

> [res.end([data] [, encoding])](https://expressjs.com/en/4x/api.html#res.end)
