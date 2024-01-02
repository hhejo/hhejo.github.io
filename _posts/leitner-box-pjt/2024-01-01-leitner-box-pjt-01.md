---
title: Leitner Box Project 01
date: 2024-01-01 15:14:34 +0900
last_modified_at: 2024-01-02 08:03:12 +0900
categories: [leitner-box-pjt]
tags:
  [leitner-box-pjt, express, react, route, event, async, fetch, axios, objectid]
---

라이트너 박스 프로젝트를 하면서 배운 것들 01

## 서버 라우트 async

```javascript
app.get('/', () => {
  (async () => {
    await ...;
  })();
})
```

- 가독성이 좀 떨어져서 아예 async 함수를 밖으로 뺄 수 없나 고민

```javascript
app.get("/", async (_, res) => {
  const findResult = await db.collection("cards").find().toArray();
  return res.json(findResult);
});
```

- 잘 작동하나 에러가 생기면 문제가 발생한다고 함
- wrapper로 감싸고 에러를 잘 처리하는 방법과
- 따로 관련 라이브러리를 사용하는 방법이 찾아보니 있었음
- 더 자세히 공부하고 나중에 다른 포스트를 작성해야겠다

## React 이벤트 핸들러 async

```javascript
const deleteHandler = async (cardId) => {
  await axios.delete(`${URL}/cards/${cardId}`);
  getCards();
};
```

- 서버에서와 마찬가지로 즉시실행함수 형태를 async로 감싸 사용하는 것은 가독성이 좀 떨어졌다
- 그래서 아예 이벤트 핸들러를 async 함수로 적용해도 되나 궁금했다
- 일단은 문제 없이 작동하는 것 같음
- 관련 내용을 좀 더 공부해야겠다

## fetch, axios

fetch를 사용하다가 axios로 넘어갔다.

1. `fetch`로 응답을 받을 때, `res.json()`으로 한번 더 풀어줘야 하는 것이 번거로움
2. `fetch`로 요청을 전송할 때도 `axios`에 비해 좀 더 작성할 것이 많음

최대한 라이브러리 없이 프로젝트를 완성하려고 했지만 fetch를 활용해본 것으로 만족한다.

## ObjectId

`ObjectId`를 이용해서 mongodb를 사용해야 한다.

```javascript
import { ObjectId } from "mongodb";
```

```javascript
await db.collection("cards").findOneAndDelete({ _id: new ObjectId(cardId) });
```

- `find`로 특정 document를 찾기 위해 `new ObjectId()`를 사용

## 현재까지 코드 저장소

> [leitner-box-frontend](https://github.com/hhejo/leitner-box-frontend/tree/1e7a6007f246608c0f127a70e960fdd72a2cf52a)

> [leitner-box-backend](https://github.com/hhejo/leitner-box-backend/tree/12938585e3245510db250f042914185666368922)
