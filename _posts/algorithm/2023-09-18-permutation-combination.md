---
title: 순열과 조합 Python으로 알아보기
date: 2023-09-18 22:51:24 +0900
last_modified_at: 2023-09-18 22:51:24 +0900
categories: [Algotirhm]
tags: [algorithm, permutation, combination, python]
---

순열과 조합, 중복순열과 중복조합에 대해 알아보고, Python으로 코드를 작성하겠습니다.

## 순열 (Permutation)

`nPr = n! / (n-r)!`

### 구현 1

```python
import sys

sys.stdin = open('src/input.txt')
input = sys.stdin.readline


# nPr
def permutation(n, r, k):
    if r == k:
        print(*res)
        return
    for i in range(n):
        if not visited[i]:
            visited[i] = True
            res[k] = arr[i]
            permutation(n, r, k + 1)
            visited[i] = False
    return


# 3P2
arr = [1, 2, 3]
n = len(arr)
r = 2
res = [0] * r
visited = [False] * n

permutation(n, r, 0)  # 배열의 크기, 뽑는 개수, 깊이
```

```
[1, 2], [1, 3], [2, 1], [2, 3], [3, 1], [3, 2]
```

### 구현 2

```python
def nPr(n, k):
    if k == n:
        print(*arr)
        return
    for i in range(k, n):
        arr[k], arr[i] = arr[i], arr[k]
        nPr(n, k + 1)
        arr[k], arr[i] = arr[i], arr[k]
    return


arr = [1, 2, 3]
n = len(arr)
r = 2

nPr(n, 0)
```

```
[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 2, 1], [3, 1, 2]
```

## 조합 (Combination)

`nCr = nPr / r!`

### 구현 1

```python
# nCr
def combination(n, r, k, s):
    if r == k:
        print(*res)
        return
    for i in range(s, n - r + k + 1):
        res[k] = arr[i]
        combination(n, r, k + 1, i + 1)
    return


# 3C2
arr = [1, 2, 3]
n = len(arr)
r = 2
res = [0] * r

combination(n, r, 0, 0)  # 배열의 크기, 뽑는 개수, 깊이, 시작 인덱스
```

```
[1, 2], [1, 3], [2, 3]
```

### 구현 2

```python
# nCr
def combination(n, r, s):
    if r == 0:
        print(*res)
        return
    for i in range(s, n - r + 1):
        res[r - 1] = arr[i]  # res[2 - r] = arr[i]
        combination(n, r - 1, i + 1)
    return


# 3C2
arr = [1, 2, 3]
n = len(arr)
r = 2
res = [0] * r

combination(n, r, 0)  # 배열의 크기, 뽑는 개수, 깊이
```

```
[2, 1], [3, 1], [3, 2]
```

### 구현 3

```python
def nCr(n, r):
    if r == 0:
        print(*res)
    elif n < r:
        return
    else:
        res[r - 1] = arr[n - 1]
        nCr(n - 1, r - 1)
        nCr(n - 1, r)


arr = [1, 2, 3, 4]
n = len(arr)
r = 3
res = [0] * r
nCr(n, r)
```

```
[2, 3, 4], [1, 3, 4], [1, 2, 4], [1, 2, 3]
```

### 구현 4

```python
def nCr(i, j, k):
    print(i, j, k)
    return


n = 4
r = 2

for i in range(n - 2):
    for j in range(i + 1, n - 1):
        for k in range(j + 1, n):
            nCr(i, j, k)
```

```
[0, 1, 2], [0, 1, 3], [0, 2, 3], [1, 2, 3]
```

### `nCr`의 개수

```python
def num_of_combination(n, r):
    if r == 0:
        return 1
    elif n < r:
        return 0
    else:
        return num_of_combination(n - 1, r - 1) + num_of_combination(n - 1, r)


arr = [1, 2, 3, 4]
n = len(arr)
r = 3
res = [0] * r
print(num_of_combination(n, r))
```

```
4
```

## 중복순열

`nπr` = `n^r`

## 중복조합

`nHr` = `n+r-1Cr`
