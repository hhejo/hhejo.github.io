---
title: Disjoint Sets 자료구조와 Union Find 알고리즘
date: 2023-08-31 20:24:18 +0900
last_modified_at: 2024-04-08 16:00:37 +0900
categories: [Algorithm]
tags: [algorithm, union-find, disjoint-sets, python]
---

Union Find 알고리즘에 대해 알아보고, Python으로 코드를 작성해보겠습니다.

## Disjoint Sets (서로소 집합)

공통 원소가 없는 두 집합으로, 상호 배타 집합, 분리 집합이라고도 합니다.

## 서로소 집합 자료구조

서로소 부분 집합들로 나누어진 원소들의 데이터를 처리하기 위한 자료구조입니다.

연결성을 통해 집합의 형태를 확인할 수 있으며, 두 종류의 연산을 지원합니다.

- Union(합집합): 두 개의 원소가 포함된 집합을 하나의 집합으로 합치는 연산
- Find(찾기): 특정한 원소가 속한 집합이 어떤 집합인지 알려주는 연산

따라서 Union Find(합치기 찾기) 자료구조라고도 합니다.

1. Union 연산을 확인하여, 서로 연결된 두 노드 `A`, `B`를 확인
   1. `A`와 `B`의 루트 노드 `A'`, `B'`을 각각 찾음
   2. `A'`을 `B'`의 부모 노드로 설정
2. 모든 Union 연산을 처리할 때까지 1번의 과정을 반복

기본적인 형태의 서로소 집합에서는 루트 노드에 즉시 접근할 수 없어 루트 노드를 찾기 위해 부모 테이블을 계속해서 확인하며 거슬러 올라가야 합니다.

## 기본 구현

```python
# Find: 특정 원소가 속한 집합 탐색
def find(x):
    if p[x] != x:
        return find(p[x])
    return x


# Union: 두 원소가 속한 집합 병합
def union(a, b):
    a = find(a)
    b = find(b)
    if a < b:
        p[b] = a
    else:
        p[a] = b


V, E = map(int, input().split())  # 노드 개수, 간선 개수
p = [i for i in range(V + 1)]  # 자기 자신이 부모인 부모 테이블

# Union 연산 각각 수행
for i in range(E):
    A, B = map(int, input().split())
    union(A, B)

# 각 원소가 속한 집합 출력
for i in range(1, V + 1):
    print(find(i), end=' ')

print()

# 부모 테이블 내용 출력
for i in range(1, V + 1):
    print(p[i], end=' ')
```

Union 연산이 편향되게 이루어지는 경우, Find 함수가 비효율적으로 동작합니다.

최악의 경우, Find 함수가 모든 노드를 다 확인하게 되어 시간 복잡도가 `O(V)`이 됩니다.

- `{1, 2, 3, 4, 5}` : `Union(4, 5)`, `Union(3, 4)`, `Union(2, 3)`, `Union(1, 2)`

### 경로 압축 구현

Find 함수를 최적화하기 위한 방법으로 경로 압축(Path Compression)을 이용합니다.

Find 함수를 재귀적으로 호출한 뒤에 부모 테이블 값을 바로 갱신합니다.

### 변경 전

```python
  def find(x):
      if p[x] != x:
          return find(p[x])  # 그냥 부모 찾기
      return x  # 부모(루트) 리턴
```

### 변경 후

```python
  def find(x):
      if p[x] != x:
          p[x] = find(p[x])  # 부모 테이블 계속 갱신
      return p[x]  # 갱신된 부모 (루트) 리턴
```

경로 압축 기법을 적용하면 각 노드에 대하여 Find 함수를 호출한 이후, 해당 노드의 루트 노드가 바로 부모 노드가 됩니다.

동일한 예시에 대해, 모든 Union 함수를 처리한 후 각 원소에 대하여 Find 함수를 수행하면 부모 테이블이 갱신되어 시간 복잡도가 개선됩니다.

## 개선된 구현

```python
# Find: 특정 원소가 속한 집합을 찾기
def find(x):
    if p[x] != x:
        p[x] = find(p[x])
    return p[x]


# Union: 두 원소가 속한 집합을 합치기
def union(a, b):
    a = find(a)
    b = find(b)
    if a < b:
        p[b] = a
    else:
        p[a] = b


V, E = map(int, input().split())  # 노드 개수, 간선 개수
p = [i for i in range(V + 1)]  # 자기 자신이 부모인 부모 테이블

# Union 연산 각각 수행
for i in range(E):
    A, B = map(int, input().split())
    union(A, B)

# 각 원소가 속한 집합 출력
for i in range(1, V + 1):
    print(find(i), end=' ')

print()

# 부모 테이블 내용 출력
for i in range(1, V + 1):
    print(p[i], end=' ')
```

## 서로소 집합을 활용한 사이클 판별

서로소 집합을 이용해 무방향 그래프 내에서의 사이클을 판별할 수 있습니다.

방향 그래프에서의 사이클 여부는 DFS를 통해 판별합니다.

1. 각 간선을 하나씩 확인하며 두 노드의 루트 노드를 확인
   1. 루트 노드가 서로 다르다면 두 노드에 대해 Union 연산 수행
   2. 루트 노드가 서로 같다면 사이클이 발생한 것
2. 그래프에 포함된 모든 간선에 대해 1번 과정을 반복

```python
# 특정 원소가 속한 집합을 찾기
def find(x):
    if p[x] != x:
        p[x] = find(p[x])
    return p[x]


# 두 원소가 속한 집합을 합치기
def union(a, b):
    a = find(a)
    b = find(b)
    if a < b:
        p[b] = a
    else:
        p[a] = b


V, E = map(int, input().split())  # 노드 개수, 간선 개수
p = [i for i in range(V + 1)]  # 자기 자신이 부모인 부모 테이블

cycle = False  # 사이클 발생 여부

for _ in range(E):
    A, B = map(int, input().split())
    # 사이클이 발생한 경우 종료
    if find(A) == find(B):
        cycle = True
        break
    # 사이클이 발생하지 않았다면 합집합 연산 수행
    else:
        union(A, B)

if cycle:
    print('사이클 발생')
else:
    print('사이클이 발생하지 않음')
```

## 참고

> [(이코테 2021 강의 몰아보기) 8. 기타 그래프 이론](https://www.youtube.com/watch?v=aOhhNFTIeFI&list=PLRx0vPvlEmdAghTr5mXQxGpHjWqSz0dgC&index=8)
