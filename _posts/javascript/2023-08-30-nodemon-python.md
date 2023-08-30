---
title: nodemon을 이용해서 Python 파일 저장시 자동 재실행하기
date: 2023-08-30 22:39:42 +0900
last_modified_at: 2023-08-30 22:39:42 +0900
categories: [JavaScript]
tags: [javascript, nodemon, python]
---

nodemon을 이용해 VSCode에서 Python으로 알고리즘을 풀이할 때, 파일 저장시 자동으로 재실행할 수 있게 합시다.

## 일일이 터미널에서 실행하는 번거로움

VSCode에서 Python으로 알고리즘을 풀이하고 있었습니다.

저는 프로젝트 폴더 내에 풀이용 `main.py`와 입력용인 `input.txt`를 두고, 풀이를 작성하면 터미널에 `python main.py`를 입력해서 출력 결과를 확인했었습니다.

그러다 문득, JavaScript로는 파일을 저장하면 자동으로 재실행할 수 있는 `nodemon`이 생각이 났습니다.

이것을 `main.py`에도 적용할 수 있나 궁금해서 시도해봤는데, 성공적이었습니다.

## 해결 과정

### 1. 프로젝트 폴더 내 package.json 파일 생성하기

```bash
npm init -y
```

### 2. nodemon 설치하기

```bash
npm install nodemon --save-dev
```

### 3. package.json에 스크립트 작성하기

```json
{
  "scripts": {
    "dev": "nodemon --exec python main.py"
  }
}
```

`scripts` 부분에 `dev` 속성을 추가합니다.

### 4. npm run dev로 실행하기

```bash
npm run dev
```

이제 `main.py`를 저장하면, 자동으로 재실행이 됩니다.

## 참고

> [How do i watch python source code files and restart when i save?](https://stackoverflow.com/questions/49355010/how-do-i-watch-python-source-code-files-and-restart-when-i-save)
