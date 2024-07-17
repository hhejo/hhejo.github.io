---
title: npm과 npx 그리고 package.json
date: 2023-07-20 00:00:00 +0900
last_modified_at: 2024-07-17 20:17:04 +0900
categories: [JavaScript]
tags: [javascript, nodejs, npm, npx, packagejson]
---

Node.js에서 사용되는 npm과 프로젝트 마다 항상 보게 되는 package.json에 대해 알아봅시다. 그리고 npx는 무엇일까요?

### Question

- npm이란?
- npm 명령어 예시

## npm (Node Package Manager)

- Node.js로 만든 패키지(모듈)를 설치하고 관리하는 표준 패키지 관리자
- 예전에는 따로 설치했었으나 현재는 Node.js에 내장되어 있어 `npm` 명령어로 npm 사용 가능
- npm을 이용해 몇 줄의 명령어로 기존에 공개된 모듈들을 설치하고 활용할 수 있음
- 패키지 업데이트와 버전 관리를 손쉽게 함
- 설치된 모듈은 CJS 방식이나 ESM 방식으로 가져와 사용하면 됨
- 로컬에 설치하면 현재 패키지의 루트 디렉터리의 `./node_modules`에 설치됨
- 전역 설치하면 `/usr/local` 또는 노드가 설치된 곳에 설치됨
- 다른 패키지 관리자로는 `yarn`, `pnpm`이 있음

## npm CLI

### 초기화

```bash
npm init -y
```

- Node.js가 설치된 상태에서, `npm init -y`를 터미널에 입력해 `package.json` 파일을 생성
- `-y` 옵션: package.json 파일에 기본 사항을 자동적으로 입력해 생성
  - 필수가 아니기 때문에 자유롭게 입력

### 패키지 설치

```bash
npm install
```

- `package.json` 파일에 작성된 의존성 패키지들을 `node_modules` 폴더에 설치
  - 하나의 프로젝트에 `package.json` 파일이 존재할 때
- 해당 폴더가 존재하지 않는다면 `node_modules` 폴더를 새로 생성한 후 진행함
- `npm i`로 줄여 입력할 수 있음

```bash
npm install <package-name>
```

- 원하는 패키지를 원격으로 설치
- npm 5 버전 이후부터는 패키지를 설치하면 자동적으로 `dependencies` 항목에 추가
  - `--save` 옵션을 추가할 필요 없음

```bash
npm install <package-name>@<version>
```

- 특정 버전의 패키지 설치
- `-D`, `-dev`, `--save-dev`: 개발할 때는 해당 패키지를 사용하나, 개발 이후에는 사용되지 않음
- `-g`: 패키지 전역 설치

### 지난 버전 확인

```bash
npm outdated
```

- 설치된 모든 종속성을 확인하고 현재 버전을 npm 레지스트리의 최신 버전과 비교
- 모든 패키지가 최신 버전이면, 아무것도 출력하지 않음

### 업데이트

```bash
npm update
```

- npm이 자동으로 패키지들의 최신 버전을 탐색
- 최신 버전이 있다면 업데이트함

### 스크립트 실행

npm 스크립트

- Node.js에서 npm 스크립트는 서버 시작, 프로젝트 빌드 및 테스트 실행을 위해 사용됨
- 웹 개발 및 배포에는 많은 반복 작업이 필요하기 때문에 이를 스크립트를 이용해 자동화할 수 있음

```bash
npm run <task-name>
```

```json
{
  "scripts": {
    "start-dev": "node lib/server-development",
    "start": "node lib/server-production",
    "watch": "webpack --watch --progress --colors --config webpack.conf.js",
    "dev": "webpack --progress --colors --config webpack.conf.js",
    "prod": "NODE_ENV=production webpack -p --config webpack.conf.js"
  }
}
```

- package.json에 실행할 작업을 명시해 명령어로 지정할 수 있음

```bash
npm run watch
npm run dev
npm run prod
```

```json
{
  "name": "example",
  "scripts": {
    "start": "npm run lite",
    "lite": "lite-server"
  }
}
```

- 다른 npm 스크립트 내에 npm 스크립트를 사용할 수도 있음
- 위 스크립트를 `npm run start`로 실행해 또다른 명령을 실행할 수 있음

### 기타 명령어

```bash
npm ls
```

- 의존성 그래프 분석

```bash
npm ls -g --depth=0
```

- 전역 설치 패키지 확인

```bash
npm -v
```

- npm 버전 확인

```bash
npm root -g
```

- 전역 패키지 설치 폴더 확인

```bash
npm view <package-name>
```

- 패키지 정보 참조

```bash
npm help
npm help <command>
npm <command> -h
```

- 도움말 관련 명령어

```bash
npm list depth <number>
```

- 특정 depth의 패키지의 리스트를 확인
- `depth 0`: 패키지 이름
- `depth 1`: dependency
- `depth 2`: 그 다음 dependency

## npx

- npm의 5.2.0부터 추가된 도구
- npm 패키지를 설치하지 않고(임시 설치) 실행할 수 있어 npm을 더 편리하게 사용
- Node.js로 빌드하고 npm registry를 통해 게시된 코드를 일회성으로 실행할 수 있음
- npx로 Node.js의 특정 버전을 사용해 실행 가능
  - 프로젝트의 Node.js 버전 별 실행 환경을 확인할 때 유용
  - 불필요한 패키지 설치를 방지하고 버전 관리, 종속성 문제에서 벗어날 수 있습니다

package.json에 스크립트를 작성하고 npm을 이용해 실행하는 번거로운 작업 대신 npx를 이용해 로컬에 설치된 패키지를 바로 실행해볼 수 있음

- npm으로는 패키지를 설치한 후, 해당 폴더로 진입해 파일을 실행하지만, npx는 한번에 가능
- 기본적으로 실행되어야 할 패키지가 경로에 있는지 확인한 후, 경로에 존재한다면 실행
- 존재하지 않는다면 패키지가 설치되지 않은 것이므로, npx가 최신 버전의 패키지를 설치하고(실행 후 없어짐) 실행

짧고 간단한 코드, 메모 등을 기록 또는 공유하는 `Github gist`에 게시된 코드를 실행할 수도 있음

```bash
npx <github-gist-주소>
```

## package.json

```json
{
  "name": "PROJECT_DIRECTORY",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Node.js 프로젝트에서는 많은 패키지를 사용하게 되고, 각각의 버전도 빈번하게 업데이트 되기 때문에 현재 프로젝트가 의존하고 있는 패키지를 일괄적으로 관리할 필요가 있습니다.

`package.json` 파일을 통해 프로젝트 정보와 패키지의 의존성을 관리합니다.

npm에 패키지를 배포하고 npm registry에 올리기 위해 반드시 필요한 문서로, 개발자가 배포한 패키지에 대해 다른 사람들이 관리하고 쉽게 설치할 수 있습니다.

여기에는 자신의 프로젝트가 의존하는 패키지의 리스트와 프로젝트의 버전을 명시합니다.

npm이라는 오픈 소스 패키지 생태계를 사용하기 위한 명세이자 프로젝트의 의존성 관리를 위한 명세이며, 해당 생태계로의 배포를 위한 명세입니다.

### scripts

`npm run` 명령어를 통해 실행할 스크립트를 작성합니다. alias를 통해 디렉터리를 지정할 수 있습니다.

### dependencies, devDependencies

해당 프로젝트에서 설치할 모듈들입니다.

package.json이 있다면 `npm install` 명령어로 해당 프로그램에 사용한 모듈들을 간단히 내려받을 수 있습니다.

패키지의 이름에 해당 패키지의 버전 범위를 매핑한 형태의 객체로 지정합니다.

해당 패키지, 프로젝트가 어떤 외부 라이브러리에 의존성(dependency)을 가지고 있는지 프로젝트가 의존하고 있는 패키지를 일괄적으로 관리하기 위한 필드 -> 해당 프로젝트가 어떤 라이브러리를 갖고 있어야 제대로 구동될수 있는지에 대한 명세입니다.

`devDependencies`는 개발시에만 필요한 의존 패키지들을 의미합니다.

### name, version

반드시 있어야 하는 필드로, 해당 패키지의 이름과 버전을 나타냅니다.

### description

패키지에 대한 설명을 나타냅니다.

### main

패키지의 진입점(entry point)이 되는 모듈의 ID. 사용자가 foo라는 이름의 패키지를 설치하고 require("foo")를 통해 모듈을 import하면, "main"으로 지정한 모듈의 exports 객체를 반환합니다.

패키지 root의 상대경로로 지정해야 합니다.

지정하지 않은 경우 root 폴더의 index.js가 기본값이 됩니다.

### keywords

키워드를 문자열 배열로 설명합니다.

### author

배포자 한 사람을 위한 필드입니다. 다수의 사람을 표시하려면 `contributors`로 작성합니다.

### license

배포한 패키지에 대해 패키지 사용자가 패키지를 사용하는 데 어떤 권한과 제한 사항이 있는지 명시합니다.

## package-lock.json

version range 형태로 표시하는 `package.json`보다 모듈들의 버전이 정확하게 표시됩니다.

`package-lock.json`이 존재하면 `npm install`은 `package.json`가 아닌 `package-lock.json`를 사용하여 `node_modules`를 생성합니다.

## 버전 표현

npm은 Semantic Version에 따라 패키지를 관리합니다.

Major, Minor, Patch 세 가지 숫자를 조합합니다.

1. Major Version : 기존 버전과 호환되지 않게 변경
2. Minor Version : 기존 버전과 호환되면서 기능 추가, 개선
3. Patch Version : 기존 버전과 호환되면서 버그 수정
4. `~` : 마이너 버전이 명시되어 있으면 패치 버전을 변경
5. `^` : 정식 버전에서 마이너와 패치 버전을 변경

## cache

패키지 설치 후 삭제하고 재설치할 때 꼬이는 문제를 방지하기 위해 알아두면 좋습니다.

보통 `/npm-cache/_cache` 폴더에 저장됩니다.

```bash
npm cache clean --force
```

npm의 cache 전부 삭제합니다.

```bash
npm cache verify
```

cache에서 꼬인 부분을 체크 및 해결합니다. (추천)

## 참고

> [Node.js v20.4.0 Documentation](https://nodejs.org/api/globals.html#global)

> [An introduction to the NPM package manager](https://nodejs.dev/en/learn/an-introduction-to-the-npm-package-manager/)

> [How to Update NPM Dependencies](https://www.freecodecamp.org/news/how-to-update-npm-dependencies/)

> [Updating packages downloaded from the registry](https://docs.npmjs.com/updating-packages-downloaded-from-the-registry)

> [Running scripts](https://riptutorial.com/node-js/example/4592/running-scripts)

> [Introduction to NPM scripts](https://www.geeksforgeeks.org/introduction-to-npm-scripts/)

> [Introducing npx: an npm package runner](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

> [npm vs npx — What’s the Difference?](https://www.freecodecamp.org/news/npm-vs-npx-whats-the-difference/)
