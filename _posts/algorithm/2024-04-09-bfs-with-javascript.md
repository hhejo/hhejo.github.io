---
title: JavaScript로 BFS 구현하기
date: 2024-04-09 13:13:09 +0900
last_modified_at: 2024-04-09 13:13:09 +0900
categories: [Algorithm]
tags: [algorithm, javascript, bfs, queue]
---

JavaScript로 Queue를 구현해보고, 그것을 사용해 BFS를 구현하는 방법

## JavaScript로 BFS 구현하기

### BFS

Breadth First Search, 너비 우선 탐색

### Queue를 구현하기

```javascript
class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}

class Queue {
  constructor() {
    this.front = null;
    this.rear = null;
    this.length = 0;
  }
  isEmpty() {
    return this.length === 0;
  }
  push(data) {
    const newNode = new Node(data);
    if (this.isEmpty()) this.front = newNode;
    else this.rear.next = newNode;
    this.rear = newNode;
    this.length++;
    return this;
  }
  popleft() {
    if (this.isEmpty()) return;
    const { data, next } = this.front;
    this.front = next;
    this.length--;
    return data;
  }
  peek() {
    if (this.isEmpty()) return null;
    return this.front.data;
  }
  size() {
    return this.length;
  }
  [Symbol.iterator]() {
    return {
      current: this.front,
      next() {
        if (this.current === null) return { done: true };
        const { data: value, next } = this.current;
        this.current = next;
        return { done: false, value };
      }
    };
  }
}

let que = new Queue();
que.push(1).push(2).push(3).push(4);
console.log(que.size()); // 4
console.log(que.popleft()); // 1
console.log(que.peek()); // 2
console.log(...que); // 2 3 4
```

### 큐를 이용해 BFS 구현

```javascript
// 큐를 이용한 BFS
function bfs(graph, sx, sy, visited) {
  const que = new Queue();
  que.push([sx, sy]); // 큐에 시작 위치 푸시
  // 큐가 빌 때까지 진행
  while (!que.isEmpty()) {
    const [cx, cy] = que.popleft(); // 현재 위치 큐에서 팝
    visited[cx][cy] = true; // 현재 위치 방문 처리
    graph[cx][cy] = order++;
    // 다음 위치
    for (let i = 0; i < 4; i++) {
      let [nx, ny] = [cx + dx[i], cy + dy[i]]; // 다음 위치
      if (nx < 0 || row <= nx || ny < 0 || col <= ny) continue; // 범위를 벗어나면 무시
      if (graph[nx][ny] === -1 || visited[nx][ny]) continue; // 이동할 수 없는 위치 혹은 방문했다면 무시
      que.push([nx, ny]); // 다음 위치 큐에 푸시
    }
  }
}

const [row, col] = [5, 5]; // 그래프 행, 열 크기
const [sx, sy] = [0, 0]; // 시작 좌표
const [dx, dy] = [
  [-1, 1, 0, 0],
  [0, 0, -1, 1]
]; // 이동 범위 (상하좌우)
const visited = Array.from(Array(row), () => Array(col).fill(false)); // 방문 여부 배열
const graph = [
  [0, 0, -1, 0, -1],
  [-1, 0, 0, 0, 0],
  [0, 0, -1, -1, 0],
  [-1, 0, 0, 0, -1],
  [-1, 0, -1, 0, -1]
]; // 그래프. -1은 갈 수 없음을 의미
let order = 1; // 방문 순서

// BFS 수행
bfs(graph, 0, 0, visited); // 그래프, 시작 좌표, 방문 여부 배열
// BFS 결과 확인
console.table(graph);
```

방문 결과

```
┌─────────┬────┬───┬────┬────┬────┐
│ (index) │ 0  │ 1 │ 2  │ 3  │ 4  │
├─────────┼────┼───┼────┼────┼────┤
│ 0       │ 1  │ 2 │ -1 │ 11 │ -1 │
│ 1       │ -1 │ 3 │ 5  │ 8  │ 12 │
│ 2       │ 7  │ 4 │ -1 │ -1 │ 14 │
│ 3       │ -1 │ 6 │ 10 │ 13 │ -1 │
│ 4       │ -1 │ 9 │ -1 │ 15 │ -1 │
└─────────┴────┴───┴────┴────┴────┘
```

## 참고

> [자료구조 JS로 구현하는 큐 (Queue)](https://velog.io/@longroadhome/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-JS%EB%A1%9C-%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94-.%ED%81%90-Queue)

> [알고리즘 JavaScript로 구현하는 BFS ](https://chamdom.blog/bfs-using-js/)
