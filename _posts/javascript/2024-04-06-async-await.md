---
title: JavaScript에서의 비동기 실행 순서 정복하기
date: 2024-04-06 14:34:35 +0900
last_modified_at: 2024-04-06 14:34:35 +0900
categories: [JavaScript]
tags:
  [
    javascript,
    asynchronous,
    promise,
    settimeout,
    call-stack,
    event-loop,
    task-queue,
    microtask-queue
  ]
---

JavaScript에서 헷갈리는 Promise, setTimeout 실행 순서를 알아보겠습니다!

## JavaScript에서의 비동기 실행 순서 정복하기

1. `Call Stack`이 비었다면 `Microtask Queue`에서 모든 작업을 꺼내 실행
2. `Task Queue`(`Macrotask Queue`)의 가장 오래된 작업을 하나 꺼내 실행
3. 다시 1번 동작을 실행

### 연습해보기

```javascript
console.log("stack [1]");

setTimeout(() => console.log("macro [2]"), 0);
setTimeout(() => console.log("macro [3]"), 1);

const p = Promise.resolve();
for (let i = 0; i < 3; i++)
  p.then(() => {
    setTimeout(() => {
      console.log("stack [4]");
      setTimeout(() => console.log("macro [5]"), 0);
      p.then(() => console.log("micro [6]"));
    }, 0);
    console.log("stack [7]");
  });

console.log("macro [8]");
```

위 코드가 실행되면, 어떻게 출력될까요?

정답은 아래에 있습니다.

```
stack [1]
macro [8]

stack [7], stack [7], stack [7]

macro [2]
macro [3]

stack [4]
micro [6]
stack [4]
micro [6]
stack [4]
micro [6]

macro [5], macro [5], macro [5]
```

## 참고

> [JavaScript Visualized: Event Loop, Web APIs, (Micro)task Queue](https://www.lydiahallie.com/blog/event-loop)

> [Difference between microtask and macrotask within an event loop context](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)

> [Javascript 비동기, Promise, async, await 확실하게 이해하기](https://springfall.cc/article/2022-11/easy-promise-async-await)
