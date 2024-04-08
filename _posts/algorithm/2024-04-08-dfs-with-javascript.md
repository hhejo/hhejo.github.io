---
title: JavaScript로 DFS 구현하기
date: 2024-04-08 07:29:10 +0900
last_modified_at: 2024-04-08 07:29:10 +0900
categories: [Algorithm]
tags: [algorithm, javascript, dfs, recursive-dfs, iterative-dfs]
---

JavaScript로 DFS를 어떻게 구현할 수 있을까요? 재귀와 스택을 각각 사용해 구현해보겠습니다.

## JavaScript로 DFS 구현하기

### DFS

Depth First Search, 깊이 우선 탐색

### 재귀를 이용해 DFS 구현

Recursive DFS 방식

```javascript
function dfs(graph, v, visited) {
  // 현재 노드를 방문 처리
  visited[v] = true;
  console.log(v);

  // 현재 노드와 연결된 다른 노드를 재귀적으로 방문
  for (let node of graph[v]) {
    if (!visited[node]) {
      dfs(graph, node, visited);
    }
  }
}

const graph = [[1, 2, 4], [0, 5], [0, 5], [4], [0, 3], [1, 2]];
const visited = Array(7).fill(false);

dfs(graph, 0, visited);
// 0 1 5 2 4 3
```

```javascript
// 재귀를 이용한 DFS
function dfs(graph, x, y, visited) {
  // 1. 현재 위치 방문 처리
  visited[x][y] = true;
  // 2. 여기서 현재 위치에 대해 작업 수행..
  console.log(`x:${x}, y:${y}`);
  graph[x][y] = order++;
  // 3. 현재 위치에서 상하좌우로 이동
  for (let i = 0; i < 4; i++) {
    let [nx, ny] = [x + dx[i], y + dy[i]]; // 다음 위치
    if (nx < 0 || row <= nx || ny < 0 || col <= ny) continue; // 범위를 벗어나면 무시
    if (graph[nx][ny] === -1 || visited[nx][ny]) continue; // 이동할 수 없는 위치 혹은 방문했다면 무시
    dfs(graph, nx, ny, visited); // 다음 위치로 이동
  }
}

const [row, col] = [5, 5]; // 그래프 행, 열 크기
const [sx, sy] = [0, 0]; // 시작 위치
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

// DFS 수행
dfs(graph, 0, 0, visited); // 그래프, 시작 위치, 방문 여부 배열
// DFS 결과 확인
console.table(graph);
```

방문 결과

```
┌─────────┬────┬───┬────┬────┬────┐
│ (index) │ 0  │ 1 │ 2  │ 3  │ 4  │
├─────────┼────┼───┼────┼────┼────┤
│ 0       │ 1  │ 2 │ -1 │ 13 │ -1 │
│ 1       │ -1 │ 3 │ 11 │ 12 │ 14 │
│ 2       │ 10 │ 4 │ -1 │ -1 │ 15 │
│ 3       │ -1 │ 5 │ 7  │ 8  │ -1 │
│ 4       │ -1 │ 6 │ -1 │ 9  │ -1 │
└─────────┴────┴───┴────┴────┴────┘
```

### 스택을 이용해 DFS 구현

Iterative DFS 방식

```javascript
// 스택을 이용한 DFS
function dfs(graph, sx, sy, visited) {
  const stk = [[sx, sy]]; // 스택에 시작 위치 푸시
  // 스택이 빌 때까지 진행
  while (stk.length) {
    const [cx, cy] = stk.pop(); // 현재 위치 스택에서 팝
    visited[cx][cy] = true; // 현재 위치 방문 처리
    graph[cx][cy] = order++;
    // 다음 위치
    for (let i = 0; i < 4; i++) {
      let [nx, ny] = [cx + dx[i], cy + dy[i]]; // 다음 위치
      if (nx < 0 || row <= nx || ny < 0 || col <= ny) continue; // 범위를 벗어나면 무시
      if (graph[nx][ny] === -1 || visited[nx][ny]) continue; // 이동할 수 없는 위치 혹은 방문했다면 무시
      stk.push([nx, ny]); // 다음 위치 스택에 푸시
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

// DFS 수행
dfs(graph, 0, 0, visited); // 그래프, 시작 좌표, 방문 여부 배열
// DFS 결과 확인
console.table(graph);
```

방문 결과

```
┌─────────┬────┬────┬────┬────┬────┐
│ (index) │ 0  │ 1  │ 2  │ 3  │ 4  │
├─────────┼────┼────┼────┼────┼────┤
│ 0       │ 1  │ 2  │ -1 │ 8  │ -1 │
│ 1       │ -1 │ 3  │ 4  │ 5  │ 6  │
│ 2       │ 10 │ 9  │ -1 │ -1 │ 7  │
│ 3       │ -1 │ 11 │ 12 │ 13 │ -1 │
│ 4       │ -1 │ 15 │ -1 │ 14 │ -1 │
└─────────┴────┴────┴────┴────┴────┘
```

재귀를 이용한 방법과 방문 순서가 다른 것을 확인할 수 있습니다.

스택은 최근에 삽입된 노드부터 꺼내 반복하고 인접 노드를 한꺼번에 추가합니다.

따라서 마지막으로 스택에 담긴 노드부터 방문하기 때문입니다.

DFS 알고리즘으로 정상적으로 방문했기 때문에 잘못되지는 않았습니다.

### 스택을 사용한 DFS의 문제점

Python은 `sys.setrecursionlimit`으로 재귀의 최대 깊이를 설정해서 알고리즘 문제를 해결할 수 있지만, JavaScript는 그럴 수 없기 때문에 특정 알고리즘 문제는 스택을 이용해 풀어야 합니다.

하지만 스택을 사용했을 때 재귀 방식과 동일하게 동작하지 않는 부분이 있는데, 부모 노드로 되돌아가는 부분입니다.

재귀와 같이 동작하도록 개선하려면, 부모 노드의 정보를 스택에 같이 넣어 해결할 수 있습니다.

루트 노드의 경우 부모 노드가 없기 때문에 유효하지 않은 값을 넣어줍니다.

```javascript
function dfs(graph, sx, sy, visited) {
  const stk = [[sx, sy]]; // 스택에 시작 위치 푸시
  // 스택이 빌 때까지 진행
  while (stk.length) {
    const [cx, cy] = stk.pop(); // 현재 위치 스택에서 팝
    if (visited[cx][cy]) {
      // 여기에 필요한 로직 추가
      continue; // 방문했다면 무시 (*)
    }
    stk.push([cx, cy]); // 방문하지 않았다면 다시 푸시 (**)
    visited[cx][cy] = true; // 현재 위치 방문 처리
    graph[cx][cy] = order++;
    // 다음 위치
    for (let i = 0; i < 4; i++) {
      let [nx, ny] = [cx + dx[i], cy + dy[i]]; // 다음 위치
      if (nx < 0 || row <= nx || ny < 0 || col <= ny) continue; // 범위를 벗어나면 무시
      if (graph[nx][ny] === -1 || visited[nx][ny]) continue; // 이동할 수 없는 위치 혹은 방문했다면 무시
      stk.push([nx, ny]); // 다음 위치 스택에 푸시
    }
  }
}
```

## 참고

> [자료구조 Stack을 이용한 Iterative DFS 구현](https://velog.io/@longroadhome/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-Stack%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-Iterative-DFS-%EA%B5%AC%ED%98%84#iterative-dfs-%EB%B0%A9%EB%AC%B8-%EA%B5%AC%ED%98%84)

> [알고리즘 JavaScript로 구현하는 DFS](https://chamdom.blog/dfs-using-js/)
