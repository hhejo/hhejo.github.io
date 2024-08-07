---
title: JavaScript의 모듈 시스템 - AMD, CommonJS, UMD, ESM
date: 2023-07-18 00:00:00 +0900
last_modified_at: 2024-07-16 19:31:06 +0900
categories: [JavaScript]
tags: [javascript, nodejs, commonjs, esmodules, amd, requrejs, umd]
---

JavaScript의 모듈 시스템에 대해 알아보겠습니다.

### Question

- 모듈은 무엇이며 장점은 무엇인가?
- 과거의 자바스크립트 모듈에 대해 설명하라
- AMD란 무엇인가? 그리고 사용 방법
- UMD란?
- CommonJS란? 작동 방식과 사용 방법은?
- ES Modules란? 작동 방식과 사용 방법, 그리고 특징

## Module

모듈

- 개발하는 애플리케이션의 크기가 커지면서 여러 개의 파일(코드)로 분리하게 되는데, 이 분리된 파일 각각을 모듈이라 함
- 하나의 클래스 혹은 함수로 구성되거나, 복수의 클래스 혹은 함수로 구성된 라이브러리로 만들어질 수도 있음

모듈의 장점

- 개발 중 크기가 큰 하나의 파일을 여러 작은 파일로 나누어 유지보수를 용이하게 함
- 각 모듈 간의 네임스페이스를 가질 수 있게 함
- 잘 나눠둔 모듈은 재사용성을 높여 업무 효율을 향상시킴

과거의 자바스크립트 모듈

- 자바스크립트는 탄생 초기에 짧은 코드, 작고 단순한 기능을 위해 사용되었기 때문에 모듈 관련 표준 문법이 존재하지 않았음
- 하지만 스크립트의 크기가 점차 커지고 기능도 복잡해지며, 사용하는 곳이 늘어나자 모듈 관련 시스템을 만들 필요성이 생김
- 이전 자바스크립트에서는 `<script src="..."></script>` 태그를 이용해 코드를 불러들이는 방식을 사용
- 그러나 이렇게 되면 네임스페이스가 중복되어 덮어씌워질 수 있음
- 코드가 길어질수록, 추가하는 파일이 많아질수록 관리가 어려워짐

## AMD (Asynchronous Module Definition)

- 자바스크립트의 사양으로, 코드 모듈과 해당 종속성(dependency)을 정의하는 API를 정의하고 원하는 경우 비동기식으로 로드
- 더 작은 자바스크립트 파일을 로드한 후, 필요할 때만 로드 가능
  - 웹사이트 성능을 개선할 수 있음
- 개발자는 AMD 구현을 통해 모듈이 실행되기 전에 로드해야 하는 종속성을 정의할 수 있음
  - 따라서 모듈은 아직 사용할 수 없는 외부 코드를 사용하려고 시도하지 않음
  - 따라서 페이지 오류가 적음

사용 방법

- 주로 `RequireJS`를 이용해 AMD를 구현
- `exports`, `require()`, `define()` 구문 사용

### 예시

```html
<script src="require.js"></script>
```

```javascript
define(["require", "dependency"], function (require, factory) {});
```

### UMD

- 모듈을 사용하는 방식이 나뉘어져 있기 때문에, 많이 사용하는 CommonJS, AMD 두 모듈 시스템을 통합해 호환시키려 하는 하나의 디자인 패턴

## CommonJS

- 자바스크립트를 웹 브라우저에서만이 아니라, 웹 브라우저 외부(웹 서버, 데스크탑 애플리케이션)에서 자바스크립트용 모듈 생태계를 표준화하는 프로젝트
- ESM 모듈 사양과 더불어 널리 쓰이는 모듈 시스템

CommonJS의 모듈 작동 방식

- Node.js를 사용하는 서버 측 자바스크립트에 사용됨
- 브라우저 측 자바스크립트에도 사용되지만, 브라우저가 CommonJS를 지원하지 않기 때문에 해당 코드를 트랜스파일러로 변환해야 함

사용 방법

- `require()`와 `module.exports` 구문 사용
- `.cjs` 파일 확장자로 명시적으로 CommonJS를 사용한다는 것을 나타냄
- Node.js에서는 기본적으로 CommonJS 방식을 사용하므로, `.js` 확장자로 모듈을 작성해도 됨

### 예시

`module` 출력하기

```javascript
console.log(module);
```

```javascript
Module {
  id: '.',
  path: '/Users/ejo/Documents/GitHub/example',
  exports: {},
  filename: '/Users/ejo/Documents/GitHub/example/index.js',
  loaded: false,
  children: [],
  paths: [
    '/Users/ejo/Documents/GitHub/example/node_modules',
    '/Users/ejo/Documents/GitHub/node_modules',
    '/Users/ejo/Documents/node_modules',
    '/Users/ejo/node_modules',
    '/Users/node_modules',
    '/node_modules'
  ]
}
```

- `module.exports`는 빈 객체를 참조
- `exports`는 `module.exports`를 참조

`require`로 `module.exports`를 받음

```javascript
// index.js
const A = require("./a.js");
A();
const B = require("./b.js");
console.log(B.bb);

// a.js
const a = () => {
  console.log("This is a");
};
module.exports = a;

// b.js
const b = 4;
exports.bb = b;
```

## ECMAScript Modules (ESM, ES Modules)

- ES2015에서 발표된 표준 모듈 시스템
- 동기식으로 작동하는 CommonJS와 달리 ESM은 비동기식으로 동작
- 이전까지 ES 모듈 시스템을 node.js에서 사용하려면 자바스크립트 트랜스파일러인 Babel을 사용했었음
- 하지만 최신 버전의 node.js에서는 `package.json`에 `"type": "module"`을 입력해 간단히 사용 가능
- `type="module"`을 사용하면 해당 파일에서는 `import`와 `export`를 사용 가능

사용 방법

- `import`와 `export` 구문을 사용
- 모듈 확장자는 `.mjs`로 표기하는 것을 권장

```json
{
  "name": "example",
  ...,
  "license": "ISC",
  "type": "module"
}
```

```html
<script type="module" src="myModule.mjs"></script>
```

모듈의 특징

- 모듈은 항상 엄격 모드(`'use strict'`)로 실행됨
- 모듈은 자신만의 스코프가 있어, 모듈 내부에서 정의한 변수나 함수는 다른 스크립트에서 접근할 수 없음
- 모듈은 단 한 번만 평가됨
- 모듈이 아닌 일반 스크립트의 `this`는 전역 객체이지만, 모듈 최상위 레벨의 `this`는 `undefined`
- 모듈 스크립트는 항상 지연 실행됨
  - 외부, 인라인 스크립트와 관계 없이 `defer` 속성을 붙인 것처럼 실행됨
  - 따라서 모듈 스크립트는 항상 완전한 HTML 페이지를 볼 수 있음

```html
<script type="module">
  const a = 10;
</script>
<script type="module">
  console.log(a); // 에러
</script>
```

### 예시

```javascript
// index.js
import a from "./a.js";
import { b } from "./b.js";
console.log(a, b);

// a.js
const a = 10;
export default a;

// b.js
export const b = 20;
```

스크립트로 로드할 때, 다음 `type="module"`을 사용

```html
<script type="module" src="my-module.js"></>
```

## global

브라우저에서 최상위 scope는 전통적으로 global scope

```javascript
var something;
```

- 위 코드는 브라우저에서 전역 변수를 나타내지만, ESM는 아님
- Node.js에서 CJS든 ESM이든 최상위 scope는 전역 scope가 아님
- 위 코드를 Node.js에서는 해당 모듈의 local 변수를 나타낼 것임

## 참고

> [모듈 소개](https://ko.javascript.info/modules-intro)

> [CommonJS](https://en.wikipedia.org/wiki/CommonJS)

> [JavaScript 표준을 위한 움직임: CommonJS와 AMD](https://d2.naver.com/helloworld/12864)

> [RequireJS - AMD의 이해와 개발](https://d2.naver.com/helloworld/591319)

> [[아티클 프로젝트 059] 모듈 시스템: CommonJS, AMD, UMD, ES6](https://okayoon.tistory.com/entry/%EC%95%84%ED%8B%B0%ED%81%B4-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-059-%EB%AA%A8%EB%93%88-%EC%8B%9C%EC%8A%A4%ED%85%9C-CommonJS-AMD-UMD-ES6)

> [AMD, CommonJS, UMD 모듈](https://www.zerocho.com/category/JavaScript/post/5b67e7847bbbd3001b43fd73)

> [모듈 시스템(CommonJS, AMD, UMD, ES6)](https://velog.io/@rlaclgns321/%EB%AA%A8%EB%93%88-%EC%8B%9C%EC%8A%A4%ED%85%9CCommonJS-AMD-UMD-ES6)

> [ES modules: 만화로 보는 심층 탐구](https://ui.toast.com/weekly-pick/ko_20180402)

> [자바스크립트 모듈 시스템: ESM, CommonJS, AMD, UMD](https://www.youdad.kr/js-module-system/)

> [CommonJS vs. ES modules in Node.js](https://blog.logrocket.com/commonjs-vs-es-modules-node-js/)

> [Using ES modules in Node.js](https://blog.logrocket.com/es-modules-in-node-today/)

> [Global objects Node.js v20.4.0 Documentation](https://nodejs.org/api/globals.html#global)
