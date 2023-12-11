---
title: 모던 JavaScript 튜토리얼 13 - 모듈
date: 2023-12-04 16:42:46 +0900
last_modified_at: 2023-12-11 04:52:14 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

모듈 소개, 모듈 내보내고 가져오기, 동적으로 모듈 가져오기

## 모듈 소개

모듈(module)

- 분리된 파일 각각
- 클래스 하나 혹은 특정한 목적을 가진 복수의 함수로 구성된 라이브러리 하나로 구성
- 자바스크립트가 만들어진 지 얼마 안 되었을 때
- 자바스크립트로 만든 스크립트의 크기도 작고 기능도 단순
- 긴 세월 동안 모듈 관련 표준 문법 없이 성장
- 점점 스크립트의 크기가 점차 커지고 기능도 복잡해짐
- 특별한 라이브러리를 만들어 필요한 모듈은 언제든지 불러올 수 있게 해줌
- 코드를 모듈 단위로 구성해 주는 방법을 만듦

모듈 시스템

- AMD
  - 가장 오래된 모듈 시스템 중 하나
  - require.js 라이브러리를 통해 처음 개발됨
- CommonJS
  - Node.js 서버를 위해 만들어짐
- UMD
  - AMD와 CommonJS와 같은 다양한 모듈 시스템을 함께 사용하기 위해 만들어짐
- ... 이런 모듈 시스템은 오래된 스크립트에서 여전히 발견할 수 있음
- 모듈 시스템은 2015년에 표준으로 등재
- 이 이후로 관련 문법은 발전해 이제는 대부분의 주요 브라우저와 Node.js가 모듈 시스템을 지원

### 모듈이란

모듈

- 단지 파일 하나에 불과함
- 스크립트 하나는 모듈 하나임
- 모듈에 특수한 지시자 `export`와 `import`를 적용하는 경우
- 다른 모듈을 불러와 불러온 모듈에 있는 함수를 호출하는 것과 같은 기능 공유가 가능

`export` 지시자

- 변수나 함수 앞에 붙임
- 외부 모듈에서 해당 변수나 함수에 접근 가능

`import` 지시자

- 외부 모듈의 기능을 가져옴

```javascript
// sayHi.js
export function sayHi(user) {
  alert(`Hello, ${user}!`);
}

// main.js
import { sayHi } from "./sayHi.js"; // import가 상대 경로 기준으로 모듈 가져오고 sayHi를 상응하는 변수에 할당
alert(sayHi);
sayHi("John"); // Hello, John!
```

브라우저에서 모듈이 동작하는 방법

- 모듈은 특수한 키워드나 기능과 함께 사용됨
- `<script type="module">` 같은 속성을 설정
- 해당 스크립트가 모듈이라는 것을 브라우저가 알 수 있게 해줌
- 모듈은 로컬 파일에서 동작하지 않음
  - HTTP 또는 HTTPS 프로토콜을 통해서만 동작
- 로컬에서 `file://` 프로토콜을 사용해 웹페이지를 열면 `import`, `export` 지시자가 동작하지 않음
- 아래 예시를 실행하려면 로컬 웹 서버인 static-server나 코드 에디터의 라이브 서버 익스텐션을 사용

```html
<!DOCTYPE html>
<script type="module">
  import { sayHi } from "./sayHi.js";
  document.body.innerHTML = sayHi("John");
</script>
```

### 모듈의 핵심 기능

일반 스크립트와 모듈의 차이

- 모든 호스트 환경에 공통으로 적용되는 모듈의 핵심 기능
- 모듈은 항상 `엄격 모드(use strict)`로 실행됨
- 모듈 레벨 스코프
  - 모듈 내부에서 정의한 변수나 함수는 다른 스크립트에서 접근 불가
- 동일한 모듈이 여러 곳에서 사용되더라도 모듈은 최초 호출 시 단 한 번만 실행됨
  - 실행 후 결과는 이 모듈을 가져가져는 모든 모듈에 내보내짐
  - 실무에서는 최상위 레벨 모듈을 대개 초기화나 내부에서 쓰이는 데이터 구조를 만들고 이를 내보내 재사용하고 싶을 때 사용
- 이런 특징을 이용해 모듈 설정(configuration)을 쉽게 할 수 있음
  - 최초로 실행되는 모듈의 객체 프로퍼티를 원하는 대로 설정하면 다른 모듈에서 이 설정을 그대로 사용

```javascript
// alert.js
alert("모듈이 평가되었습니다.");

// 동일한 모듈을 여러 모듈에서 가져오기
// 1.js
import "./alert.js"; // 모듈이 평가되었습니다.
// 2.js
import "./alert.js";
```

```javascript
// admin.js
export let admin = {};
export function sayHi() {
  alert(`Hello, ${admin.name}!`);
}
// init.js
import { admin } from "./admin.js";
admin.name = "John"; // 어느 한 모듈에서 수정하면 다른 모듈에서도 변경사항 확인 가능
// other.js
import { admin, sayHi } from "./admin.js";
alert(admin.name); // John
sayHi(); // Hello, John!
```

`import.meta` 객체

- 현재 모듈에 대한 정보 제공
- 호스트 환경에 따라 제공하는 정보의 내용은 다름
  - 브라우저 환경에서는 스크립트의 URL 정보를 얻을 수 있음
  - HTML 안에 있는 모듈이라면 현재 실행중인 웹페이지의 URL 정보를 얻을 수 있음

```html
<script type="module">
  alert(import.meta.url); // script URL (인라인 스크립트가 위치해 있는 html 페이지의 URL)
</script>
```

모듈 최상위 레벨의 `this`는 `undefined`

- 모듈이 아닌 일반 스크립트의 `this`는 전역 객체

```html
<script>
  alert(this); // window
</script>
<script type="module">
  alert(this); // undefined
</script>
```

### 브라우저 특정 기능

브라우저 환경에서 `type="module"` 스크립트 vs. 일반 스크립트

- 모듈 스크립트는 항상 지연 실행됨
  - 외부 스크립트, 인라인 스크립트와 관계없이 `defer` 속성을 붙인 것처럼 실행됨
  - 외부 모듈 스크립트 `<script type="module" src="...">`를 다운로드할 때 브라우저의 HTML 처리가 멈추지 않음
  - 브라우저는 외부 모듈 스크립트와 기타 리소스를 병렬적으로 불러옴
- 모듈 스크립트는 HTML 문서가 완전히 만들어진 이후 실행
  - 모듈의 크기가 아주 작아서 HTML보다 빨리 불러온 경우에도 동일
  - 페이지 내 특정 기능이 모듈 스크립트에 의존적인 경우
  - 모듈이 완전히 로딩되기 전에 페이지가 먼저 사용자에게 노출되면 사용자가 혼란을 느낄 수 있기 때문
  - 모듈 스크립트를 불러오는 동안에는 투명 오버레이나 로딩 인디케이터를 보여주어 사용자의 혼란을 예방
  - 어디에도 종속되지 않는 기능을 구현할 때 유용
  - 광고, 문서 레벨 이벤트 리스너, 카운터 등
- 스크립트의 상대적 순서가 유지됨
  - 문서상 위쪽의 스크립트부터 차례로 실행됨
- 그래서 모듈 스크립트는 항상 완전한 HTML 페이지를 '볼 수'있고 문서 내 요소에도 접근 가능

```html
<!-- 모듈 스크립트: 지연 실행되기 때문에 페이지가 모두 로드된 후 alert 함수 실행 -->
<script type="module">
  alert(typeof button); // (2) object. 아래 쪽의 button 요소를 정상적으로 출력
</script>

<!-- 일반 스크립트: 페이지가 완전히 구성되기 전이라도 바로 실행 -->
<script>
  alert(typeof button); // (1) undefined. 버튼 요소가 페이지에 생성되기 전에 접근
</script>

<button id="button">Button</button>
```

인라인 스크립트의 비동기 처리

- 모듈이 아닌 일반 스크립트에서 `async` 속성은 외부 스크립트를 불러올 때만 유효
- `async`는 로딩이 끝나면 다른 스크립트나 HTML 문서가 처리되길 기다리지 않고 바로 실행됨
- 모듈 스크립트에서는 `async` 속성을 인라인 스크립트에도 적용 가능

외부 모듈 스크립트(`type="module"`)

- `src` 속성값이 동일한 외부 스크립트는 한 번만 실행됨
- 외부 사이트같이 다른 오리진에서 모듈 스크립트를 불러오려면 CORS 헤더가 필요
- 모듈이 저장된 원격 서버가 `Access-Control-Allow-Origin: *` 헤더를 제공해야만 외부 모듈을 불러옴
- `*` 대신 fetch를 허용할 도메인을 명시할 수도 있음
  - 이 특징은 보안을 강화해줌
- 경로가 없는 모듈은 허용되지 않음
  - 브라우저 환경에서 `import`는 반드시 상대 혹은 절대 URL 앞에 와야 함
  - Node.js나 번들링 툴은 경로가 없는 모듈 사용 가능
  - 경로가 없어도 해당 모듈을 찾을 수 있는 방법을 알기 때문
- 호환을 위한 `nomodule`
  - 구식 브라우저는 `type="module"`을 해석하지 못함
  - 모듈 타입의 스크립트를 만나면 무시하고 넘어감
  - `nomodule` 속성을 사용하면 대비 가능

```html
<script async type="module">
  import { counter } from "./analytics.js"; // 필요한 모듈(./analytics.js) 가져오기 작업이 끝나면
  counter.count(); // HTML 파싱이 끝나지 않았거나 다른 스크립트가 대기 상태에 있어도 바로 모듈 실행
</script>

<!-- my.js는 한 번만 로드 및 실행됨 -->
<script type="module" src="my.js"></script>
<script type="module" src="my.js"></script>

<!-- another-site.com이 Access-Control-Allow-Origin을 지원해야만 외부 모듈을 불러올 수 있음 -->
<!-- 그렇지 않으면 스크립트는 실행되지 않음 -->
<script type="module" src="http://another-site.com/their.js"></script>
```

```javascript
// './sayHi.js'같이 경로 정보를 지정해줘야 함
import { sayHi } from "sayHi"; // 에러
```

```html
<script type="module">
  alert("모던 브라우저를 사용하고 계시군요.");
</script>

<script nomodule>
  alert(
    "type=module을 해석할 수 있는 브라우저는 nomodule 타입의 스크립트는 넘어갑니다. 따라서 이 alert 문은 실행되지 않습니다."
  );
  alert(
    "오래된 브라우저를 사용하고 있다면 type=module이 붙은 스크립트는 무시됩니다. 대신 이 alert 문이 실행됩니다."
  );
</script>
```

### 빌드 툴

빌드 툴

- 브라우저 환경에서 모듈을 단독으로 사용하는 경우는 많지 않음
- 대개 웹팩(Webpack)과 같은 특별한 툴을 사용
- 모듈을 한 데 묶어(번들링) 프로덕션 서버에 올림
- 번들러를 사용하면 모듈 분해를 통제 가능
- 경로가 없는 모듈이나 CSS, HTML 포맷의 모듈을 사용 가능

빌드 툴의 역할

1. HTML의 `<script type="module">`에 넣을 '주요'(main) 모듈('진입점' 역할을 하는 모듈)을 선택
2. '주요' 모듈에 의존하고 있는 모듈 분석을 시작으로 모듈 간의 의존 관계를 파악
3. 모듈 전체를 한데 모아 하나의 큰 파일 생성
   - 설정에 따라 여러 개의 파일을 만드는 것도 가능
   - 이 과정에서 `import`문이 번들러 내 함수로 대체되므로 기존 기능은 그대로 유지됨
4. 이런 과정 중에 변형이나 최적화도 함께 수행
   - 도달 가능하지 않은 코드는 삭제됨
   - 내보내진 모듈 중 쓰임처가 없는 모듈을 삭제(가지치기(tree-shaking))
   - `console`, `debugger` 같은 개발 관련 코드를 삭제
   - 최신 자바스크립트 문법이 사용된 경우 바벨(Babel)을 사용해 동일한 기능을 하는 낮은 버전의 스크립트로 변환
   - 공백 제거, 변수 이름 줄이기 등으로 산출물의 크기를 줄임

번들링 툴

- 스크립트들을 하나 혹은 여러 개의 파일로 번들링
- 번들링 전 스크립트에 있던 `import`, `export`는 특별한 번들러 함수로 대체됨
- 번들링 과정이 끝나면 기존 스크립트에서 `import`, `export`가 사라짐
  - `type="module"`이 필요 없어짐
- 따라서 아래와 같이 번들링 과정을 거친 스크립트는 일반 스크립트처럼 취급 가능
- 실제 애플리케이션을 출시할 때는 성능 개선 등의 이점 때문에 웹팩과 같은 번들러를 사용

```html
<!-- 웹팩과 같은 툴로 번들링 과정을 거친 스크립트인 bundle.js -->
<script src="bundle.js"></script>
```

## 모듈 내보내고 가져오기

### 선언부 앞에 export 붙이기

`export`

- 변수나 함수, 클래스를 선언할 때 `export`를 붙여 내보냄
- 클래스나 함수를 내보낼 때는 세미콜론을 붙이지 않기

```javascript
export let months = ["Jan", "Feb", "Mar", "Apr", "Sep", "Oct", "Nov", "Dec"]; // 배열 내보내기
export const MODULES_BECAME_STANDARD_YEAR = 2015; // 상수 내보내기
export class User {
  constructor(name) {
    this.name = name;
  }
} // 클래스 내보내기
```

### 선언부와 떨어진 곳에 export 붙이기

선언부와 `export`가 떨어져 있어도 내보내기 가능

- `export`문을 함수 선언부 위에 적어주는 것도 동일하게 동작

```javascript
// sayHi.js
function sayHi(user) {
  alert(`Hello, ${user}!`);
}
function sayBye(user) {
  alert(`Bye, ${user}!`);
}
export { sayHi, sayBye }; // 두 함수를 내보냄
```

### import

`import`

- 무언가를 가져오고 싶다면 이에 대한 목록을 만들어 `import {...}` 안에 작성
- `import * as <obj>`처럼 객체 형태로 가져올 수도 있음
  - 가져올 것이 많을 때 유용
- 그러나 가져올 때는 그 대상을 구체적으로 명시하는 것이 좋음

```javascript
// main.js
import { sayHi, sayBye } from "./say.js";
sayHi("John"); // Hello, John!
sayBye("John"); // Bye, John!

// main.js
import * as say from "./say.js";
say.sayHi("John");
say.sayBye("John");
```

`import` vs. `import *`

1. 웹팩과 같은 모던 빌드 툴은 로딩 속도를 높이기 위해 모듈들을 한데 모으는 번들링과 최적화를 수행
   - 이 과정에서 사용하지 않는 리소스가 삭제되기도 함
   - 빌드 툴은 실제 사용되는 함수가 무엇인지 파악해
   - 그렇지 않은 함수는 최종 번들링 결과물에 포함하지 않음
   - 이 과정에서 불필요한 코드가 제거되어 빌드 결과물의 크기가 작아짐
   - 이런 최적화 과정은 가지치기(tree-shaking)라고 불림
2. 어떤 걸 가지고 올지 명시하면 이름을 간결하게 써줄 수 있음
3. 어디서 어떤 게 쓰이는지 명확해짐
4. 코드 구조를 파악하기 쉬워 리팩토링, 유지보수 향상

### import 'as'

`import .. as ..`

- `as`를 사용해 이름을 바꿔서 모듈을 가져옴

```javascript
// main.js
import { sayHi as hi, sayBye as bye } from "./say.js";
hi("John"); // Hello, John!
bye("John"); // Bye, John!
```

### export 'as'

`export .. as ..`

- `export`에도 `as` 사용 가능

```javascript
// say.js
export { sayHi as hi, sayBye as bye };
// main.js
import * as say from "./say.js";
say.hi("John"); // Hello, John!
say.bye("John"); // Bye, John!
```

### export default

모듈은 크게 두 종류로 나뉨

1. 복수의 함수가 있는 라이브러리 형태의 모듈
2. 개체 하나만 선언되어있는 모듈
   - 대개 이 방식으로 모듈을 만드는 것을 선호
   - 함수, 클래스, 변수 등의 개체는 전용 모듈 안에 구현됨

`export default`

- 모듈이 지원하는 특별한 문법
- 해당 모듈에는 개체가 하나만 있다는 사실을 명확히 나타낼 수 있음
- `default`를 붙여서 모듈을 내보내면, 중괄호 `{}` 없이 모듈을 가져올 수 있음
- 파일당 최대 하나의 default export가 있을 수 있음
- 그래서 내보낼 객체에는 이름이 없어도 괜찮음

```javascript
// user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// main.js
import User from "./user.js";
new User("John");
```

named export vs. default export

- named export와 default export를 같은 모듈에 동시에 사용해도 문제는 없음
- 그런데 실무에서는 섞어 쓰는 사례가 흔치 않음
- 한 파일에는 named export나 default export 둘 중 하나만 사용
- `export default`는 이름이 없어도 괜찮음
  - `export default`는 파일당 하나만 있음
  - 이 개체를 가져오려는 모듈에서는 중괄호 없이도 어떤 개체를 가지고 올지 정확히 알 수 있음
  - `default`를 붙이지 않았다면 개체에 이름이 없는 경우 에러 발생

|        named export        |          default export           |
| :------------------------: | :-------------------------------: |
| `export class User {...}`  | `export default class User {...}` |
| `import { User } from ...` |      `import User from ...`       |

```javascript
export default class { // 클래스 이름이 없음
  constructor() { ... }
}
export default function(user) { // 함수 이름이 없음
  alert(`Hello, ${user}!`);
}
export default [1, 2, 3]; // 이름 없이 배열 형태의 값을 내보냄
```

### 'default' name

`default` 키워드

- 기본 내보내기를 참조하는 용도로 종종 사용됨
- 함수 선언부와 떨어진 곳에서 `default` 키워드를 사용해 해당 함수를 기본 내보내기 할 수 있음

```javascript
function sayHi(user) {
  alert(`Hello, ${user}!`);
}
export { sayHi as default }; // 함수 선언부 앞에 'export default'를 붙여준 것과 동일
```

`default export` 하나와 다수의 `named export`가 있을 때 동시에 가져오기

- `*`를 사용해 모든 것을 객체로 가져오는 방법도 있음
- 이 경우에 `default` 프로퍼티는 정확히 default export를 가리킴

```javascript
// user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}
export function sayHi(user) {
  alert(`Hello, ${user}!`);
}
// main.js
import { default as User, sayHi } from "./user.js"; // default export와 named export 동시에 가져오기
new User("John");
```

```javascript
// main.js
import * as user from "./user.js";
let User = user.default; // default export
new User("John");
```

named export의 이름에 관한 규칙

- 내보냈을 때 사용한 이름 그대로 가져와 관련 정보를 파악하기 쉬움
- 내보내기 할 때 쓴 이름과 가져오기할 때 쓸 이름이 동일해야 함

```javascript
import { User } from "./user.js"; // import { MyUser }은 사용할 수 없음. 반드시 { User }이어야 함
```

default export의 이름에 관한 규칙

- named export와 다르게 가져오기 할 때 원하는 대로 이름 지정 가능
- 자유롭게 이름을 짓다 보면 같은 걸 가져오는데도 이름이 달라 혼란의 여지가 생길 수 있음
- default export 한 것을 가져올 때는 파일 이름과 동일한 이름을 사용하도록 함
  - 팀원끼리 내부 규칙을 정할 수 있음
  - 이런 문제를 예방하고 코드의 일관성을 유지
- named export만 사용할 것을 강제하는 경우도 있음
  - 모듈 하나에서 단 하나의 개체만 내보내는 경우에도 `default` 없이 이름을 붙여 내보내 혼란을 방지
  - 이런 규칙은 모듈 다시 내보내기를 쉽게 해준다는 장점도 있음

```javascript
// 1
import User from "./user.js"; // 동작
import MyUser from "./user.js"; // 동작

// 2
import User from "./user.js";
import LoginForm from "./loginForm.js";
import func from "/path/to/func.js";
```

### 모듈 다시 내보내기

`export ... from ...`

- 가져온 개체를 즉시 다시 내보내기(re-export)함
- 이름을 바꿔서 다시 내보낼 수 있음

```javascript
export { sayHi } from "./say.js"; // sayHi를 다시 내보내기 함
export { default as User } from "./user.js"; // default export를 다시 내보내기 함
```

다시 내보내기가 실무에서 사용되는 경우

- NPM을 통해 외부에 공개할 패키지를 만들고 있다고 가정
  - 수많은 모듈로 구성된 패키지
  - 몇몇 모듈은 외부에 공개할 기능
  - 몇몇 모듈은 이러한 모듈을 도와주는 헬퍼 역할을 담당
- 진입점 역할을 하는 '주요 파일'인 `auth/index.js`를 통해 기능을 외부에 노출
  - 이 패키지를 사용하는 개발자들은 `(*)` 같은 코드로 해당 기능을 사용할 것
- 이때 우리가 만든 패키지를 사용하는 외부 개발자가 패키지 안의 파일들을 뒤져 내부 구조를 건드리게 하면 안 됨
- 공개할 것만 `auth/index.js`에 넣어 내보내기 하고 나머지는 숨기는 것이 좋음
- 이때 내보낼 기능을 패키지 전반에 분산하여 구성
  - `auth/index.js`에서 이 기능들을 가져오고 이를 다시 내보내면 원하는 바를 어느 정도 달성할 수 있음
- 이제 외부 개발자들은 `import { login } from 'auth/index.js'`로 우리가 만든 패키지를 이용 가능
- `export ... from ...`는 개체를 가지고 온 후 바로 내보낼 때 쓸 수 있는 문법

```
패키지 구조
auth/
    index.js
    user.js
    helpers.js
    tests/
        login.js
    providers/
        github.js
        facebook.js
        ...
```

```javascript
import { login, logout } from "auth/index.js"; // (*)
```

```javascript
// auth/index.js
// login과 logout을 가지고 온 후 바로 내보냄
import { login, logout } from './helpers.js';
export { login, logout };
// User를 가져온 후 바로 내보냄
import User from './user.js';
export { User };
...
```

- 아래 예시는 위 예시와 동일하게 동작

```javascript
// auth/index.js
// login과 logout을 가지고 온 후 바로 내보냄
export { login, logout } from './helpers.js';
// User 가져온 후 바로 내보냄
export { default as User } from './user.js';
...
```

default export 다시 내보내기

- 기본 내보내기를 다시 내보낼 때는 주의해야 할 점들이 있음
- `user.js` 내의 클래스 `User`를 다시 내보내기 한다고 가정

```javascript
// user.js
export default class User {
  // ...
}
```

1. `User`를 `export User from "./user.js"`로 다시 내보내기 할 때 문법 에러 발생
   - default export를 다시 내보내려면 위 예시처럼 `export { default as User }`를 사용해야 함
2. `export * from "./user.js"`를 사용해 모든 걸 한 번에 다시 내보내면 default export는 무시되고, named export만 다시 내보내짐
   - 두 가지를 동시에 다시 내보내고 싶다면 두 문을 동시에 사용해야 함

```javascript
export * from "./user.js"; // named export를 다시 내보내기
export { default } from "./user.js"; // default export를 다시 내보내기
```

블록 안에서의 import/export

- import/export문은 블록 `{...}` 안에서는 동작하지 않음

```javascript
if (something) {
  import { sayHi } from "./say.js"; // 에러: import 문은 최상위 레벨에 위치해야 함
}
```

## 동적으로 모듈 가져오기

## 참고

> [모듈](hhttps://ko.javascript.info/modules)
