---
title: Node.js의 환경변수 관리
date: 2023-07-22 00:00:00 +0900
last_modified_at: 2024-07-18 21:28:39 +0900
categories: [JavaScript]
tags: [javascript, nodejs, process, env, dotenv]
---

Node.js에서 환경변수를 관리하기 위한 process, .env, dotenv에 대해 알아보겠습니다.

## Command Line Applications

- 명령줄에서 실행되는 애플리케이션
- CLI (Command Line Interface) App으로도 불림
- 사용자는 터미널 명령어로 클라이언트와 상호작용을 함
- 동일한 작업(배포, 테스트 등)을 반복 수행하는 경우, 스크립트를 써 해당 단계를 자동화해 시간을 절약
- Node.js로 CLI App 작성 가능

## 환경변수 (Environment Variable)

- 여러 환경에 애플리케이션을 배포할 때, 각 환경에 따라 다르게 설정되어야 하는 값들을 환경변수에 설정
  - 개발, 테스트, 운영 등의 여러 환경
- 또한 외부로부터 노출되지 않아야 하는 데이터를 환경 변수로 설정해 코드 외부에 위치시킴
  - API 키, 데이터베이스 비밀번호 등

`process` 객체

```javascript
const process = require("node:process");
```

- 현재 Node.js의 프로세스에 대한 정보와 제어를 제공

`process.env` 객체

```javascript
const { env } = require("node:process");
```

- 사용자 환경이 포함된 개체를 반환
- 런타임 중에 주입되는 전역 객체
- 환경변수를 설정하면 런타임 중에 `process.env`에 로드되고 이후에 접근 가능
- 환경변수는 항상 문자열의 값을 가짐
- 이 객체를 수정하는 것은 가능
- 그러한 수정 사항은 Node.js 프로세스 외부나 (명시적으로 요청하지 않는 한) 다른 Worker 스레드에 반영되지 않음

## Node.js 인터프리터로 값 확인하기

```bash
$ nvm use --lts
$ node
> process.env.PWD
'/Users/hejo/Documents/my-pjt'
> process.env.PASSWORD
undefined
```

- `process.env.PASSWORD`는 임의로 지정한 속성이기 때문에 `undefined`가 출력됨

환경변수를 설정하기

1. 환경변수의 값을 지정하고 `node`로 인터프리터를 실행
   - `process.env.PASSWORD`의 값이 잘 출력됨
   - 해당 환경변수는 해당 프로그램이 실행되는 동안에만 유효
   - CLI를 종료하고 `node`로만 실행하면 환경변수 값은 사라짐
2. 또는 인터프리터 실행 중에 값을 대입할 수도 있음
3. `delete`로 값 삭제 가능

```bash
❯ PASSWORD=abcd1234 node
Welcome to Node.js v18.16.1.
Type ".help" for more information.
> process.env.PASSWORD
'abcd1234'
> process.env.PASSWORD=1234
1234
> process.env.PASSWORD
'1234'
> delete process.env.PASSWORD
true
> process.env.PASSWORD
undefined
```

```bash
$ node
Welcome to Node.js v18.16.1.
Type ".help" for more information.
> process.env.PASSWORD
undefined
```

- 매번 명령줄에 작성하고 실행하는 것은 번거로움

`export` 명령어

- 프로세스가 종료된 후 다시 실행해도 지정했던 환경변수의 값을 얻을 수 있음
- 그러나 명령줄을 실행했던 터미널을 종료하면 해당 환경변수의 값은 사라짐

```bash
$ export PASSWORD=1234
$ node
Welcome to Node.js v18.16.1.
Type ".help" for more information.
> process.env.PASSWORD
'1234'
```

`.env` 파일과 `dotenv` 패키지를 사용해 환경변수를 관리 가능

## dotenv

- 환경변수 관리에 도움을 주는 라이브러리
- 기본값으로 현재 디렉터리에 위치한 `.env` 파일로부터 환경변수를 읽음

`.env` 파일

- 프로젝트의 소스 코드에서 접근할 수 있도록 Node.js의 환경변수를 설정 가능
- 그러나 Node.js는 기본적으로 `.env`의 파일을 로드하지 않음
- `dotenv` 패키지를 사용해 해당 파일을 로드하여 환경변수 값에 접근해야 함

### dotenv 패키지 설치

```bash
npm install dotenv
```

프로젝트 루트 폴더에 `.env` 파일을 생성하고 원하는 환경변수 작성

```
PASSWORD="1234"
```

### CJS

```javascript
require("dotenv").config();

const { PASSWORD } = process.env;

console.log(`PASSWORD: ${PASSWORD}`);
```

### ESM

```javascript
import dotenv from "dotenv";

dotenv.config(); // 환경변수에 접근하기 전에 호출

const { PASSWORD } = process.env;

console.log(`PASSWORD: ${PASSWORD}`);
```

```javascript
import "dotenv/config"; // 이렇게 작성할 수도 있음
```

- 해당 파일을 모듈로 작동시킬 때, `dotenv`의 `config`가 선행되도록 해야함

### 주의

`.env` 파일은 `.gitignore` 파일에 작성해 깃허브에 올라가지 않게 하는 것을 권장

```
# .gitignore
.env
```

### 예시

아래 예시처럼 `.env` 파일을 구성 가능

```
PORT = 8000
NODE_ENV = development
CLIENT_URL = "http://localhost:3000"
DATABASE_URL = "..."
```

## 참고

> [Node.js에서 환경 변수 다루기 (process.env)](https://www.daleseo.com/js-node-process-env/)

> [dotenv로 환경 변수를 .env 파일로 관리하기](https://www.daleseo.com/js-dotenv/)

> [The minimal Node.js with Babel Setup](https://www.robinwieruch.de/minimal-node-js-babel-setup/)

> [Reading Environment Variables From Node.js](https://www.geeksforgeeks.org/reading-environment-variables-from-node-js/)

> [Node Environment Variables: Process env Node](https://www.knowledgehut.com/blog/web-development/node-environment-variables)

> [Node.js process.env Property](https://www.geeksforgeeks.org/node-js-process-env-property/)

> [Dotenv - npm](https://www.npmjs.com/package/dotenv)

> [Process](https://nodejs.org/api/process.html)
