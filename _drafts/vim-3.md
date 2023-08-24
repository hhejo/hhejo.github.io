---
title: Vim 사용하기 3 -
date: 2023-08-10 00:00:00 +0900
last_modified_at: 2023-08-10 00:00:00 +0900
categories: [Vim]
tags: [vim]
---

ha ha ha ha

## 비주얼 모드

- `v`: 비주얼 모드로 진입
- `V`: 줄 단위 비주얼 모드로 진입

## 설명

- `i`: 안쪽
- `a`: 바깥쪽
- `f`: 포함
- `t`: 포함하지 않음

## 복사하기

명령을 다음과 같이 구성할 수 있습니다.

```
y <1> <2>
```

- `yy`: 한 줄 전체 복사
- `yw`: 커서부터 word 끝까지 복사
- `yW`: 커서부터 WORD 끝까지 복사
- `yiw`: 커서가 속한 word 전체 복사
- `yiW`: 커서가 속한 WORD 전체 복사
- `yaw`: 커서가 속한 word 전체 공백 포함해서 복사
- `yaW`: 커서가 속한 WORD 전체 공백 포함해서 복사

- `yf(ch)`: 커서부터 ch까지 포함해서 복사
- `yF(ch)`: 커서 앞부터 역방향으로 ch까지 포함해서 복사 (?)
- `yt(ch)`: 커서부터 ch 앞까지 복사
- `yT(ch)`: 커서 앞부터 역방향으로 ch 앞까지 복사 (?)

###

- `yi(`(`yi)`): 소괄호 내용 복사
- `yi{`(`yi}`): 중괄호 내용 복사
- `yi[`(`yi]`): 대괄호 내용 복사

- `ya(`(`ya)`): 소괄호 내용 복사. 소괄호 포함
- `ya{`(`ya}`): 중괄호 내용 복사. 중괄호 포함
- `ya[`(`ya]`): 대괄호 내용 복사. 대괄호 포함

- `yi'`: 작은 따옴표 내용 복사
- `yi"`: 큰 따옴표 내용 복사
- `yi백틱`: 백틱 내용 복사

### 비주얼 모드와 함께

- `vy`: 비주얼 모드로 하이라이트된 부분 복사

123 This yes varo Gt.

## 붙여녛기

## 삭제하기

## 수정하면서

- `ci`

찾기,

f, t

명령 조합

옵션 추가

dw daw dW daW

Ctrl-v + I(A)

```
                                                        bar
|                       To screen column [count] in the current line.
                        exclusive motion.  Ceci n'est pas une pipe.
```

That's, erm, no-1@there.com

## 참고

> [Vim documentation: help](https://vimdoc.sourceforge.net/htmldoc/help.html#reference_toc)
