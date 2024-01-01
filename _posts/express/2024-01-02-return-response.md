---
title: Express에서 response와 함께 return을 작성해야 하는 이유
date: 2024-01-02 08:36:15 +0900
last_modified_at: 2024-01-02 08:36:15 +0900
categories: [Express]
tags: [express, nodejs, return, response]
---

왜 return이 필요할까요?

## res 응답 메서드를 사용해도 끝나는 것이 아니다

```javascript
import express from "express";
import cors from "cors";

const app = express();
const port = 3000;

app.use(express.json());
app.use(cors({ origin: "http://localhost:5173" }));

app.get("/test", (req, res) => {
  res.json({ testKey: "testValue" });
  console.log("응답을 전송해도 끝나지 않고 이 문장은 출력됨"); // (*)
});

app.listen(port, () => console.log(`Listening on port ${port}`));
```

`/test`로 요청을 보내면 `res.json()`이 클라이언트로 응답을 전송합니다.

하지만 여기서 끝나지 않고 아랫줄 `(*)`를 더 실행하게 됩니다.

클라이언트는 응답을 잘 받겠지만, 서버에서는 응답을 전송한다고 끝나는 것이 아니기 때문입니다.

그렇기 때문에 응답을 전송한 후 종료해야 한다면 `return`을 작성해 종료될 수 있게 합니다.

```javascript
app.get("/test", (req, res) => {
  res.json({ testKey: "testValue" });
  return;
  console.log("이 문장은 출력되지 않음");
});
```

```javascript
app.get("/test", (req, res) => {
  return res.json({ testKey: "testValue" });
  console.log("이 문장은 출력되지 않음");
});
```

반환값이 중요한 것이 아니라면 `return`과 붙여 쓰는 것이 가독성이 좋아질 수 있습니다.

응답을 전송하고 무언가를 해야 하는 경우가 아니라면 `return`과 함께 함수를 종료하는 것이 좋습니다.

## 참고

> [Why you should ALWAYS use return before res.send in Express APIs and Applications](https://dev.to/adamkatora/why-you-should-always-use-return-before-ressend-in-express-apis-and-applications-k9k)
