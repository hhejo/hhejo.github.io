---
title: 모던 JavaScript 튜토리얼 19 - 문서와 리소스 로딩
date: 2024-01-07 08:15:11 +0900
last_modified_at: 2024-01-14 08:26:09 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

DOMContentLoaded, load, beforeunload, unload 이벤트, defer, async 스크립트, Resource loading: onload and onerror

## DOMContentLoaded, load, beforeunload, unload 이벤트

HTML 문서의 생명주기

- 3가지 주요 이벤트가 관여

`DOMContentLoaded`

- 브라우저가 HTML을 읽고 DOM 트리를 완성하는 즉시 발생
- 기타 자원은 기다리지 않음
  - 이미지 파일(`<img>`), 스타일시트 등
- DOM이 준비된 것을 확인 후, 원하는 DOM 노드를 찾아 핸들러를 등록해 인터페이스를 초기화 할 때 사용

`load`

- HTML로 만든 DOM 트리가 완성되었을 뿐만 아니라 외부 자원도 모두 불러오는 것이 끝났을 때 발생
  - 이미지, 스타일시트 같은 외부 자원
- 화면에 뿌려진 요소의 실제 크기 확인 가능
  - 외부 자원이 로드되고 스타일이 적용된 상태이기 때문
- 이미지 사이즈를 확인할 때 등에 사용

`beforeunload/unload`

- 사용자가 페이지를 떠날 때 발생
- `beforeunload`: 사용자가 사이트를 떠나려 할 때, 변경되지 않은 사항들을 저장했는지 확인시켜줄 때 사용
- `unload`: 사용자가 진짜 떠나기 전, 사용자 분석 정보를 담은 통계 자료를 전송하고자 할 때 사용

### DOMContentLoaded

`DOMContentLoaded` 이벤트

```javascript
document.addEventListener("DOMContentLoaded", ready);
```

- `document` 객체에서 발생
  - `document.onDOMContentLoaded`는 동작하지 않음
- 모든 요소에 접근 가능
  - 문서가 로드되었을 때 실행되기 때문
- DOM 트리가 완성되면 발생하는 이벤트라고 생각하지만, 몇 가지 특이사항 존재

```html
<script>
  function ready() {
    alert("DOM이 준비되었습니다!");
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`); // 이미지가 아직 로드되지 않아 사이즈는 0x0
  }
  document.addEventListener("DOMContentLoaded", ready); // 문서가 로드되었을 때 실행. <img> 포함 모든 요소에 접근 가능
</script>
<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0" />
```

DOMContentLoaded와 scripts

- 브라우저는 HTML 문서를 처리하는 도중에 `<script>` 태그를 만나면, DOM 트리 구성을 멈추고 `<script>`를 실행
- 스크립트 실행이 끝난 후에야 나머지 HTML 문서를 처리
- `<script>`에 있는 스크립트가 DOM 조작 관련 로직을 담고 있을 수 있기 때문에 만들어진 방지책
- `DOMContentLoaded` 이벤트 역시 `<script>` 안의 스크립트가 처리된 후 발생
- DOMContentLoaded를 막지 않는 두 가지 예외상황 스크립트
  1. `async` 속성이 있는 스크립트
  2. `document.createElement('script')`로 동적으로 생성되고 웹 페이지에 추가된 스크립트

```html
<script>
  document.addEventListener("DOMContentLoaded", () => {
    alert("DOM이 준비되었습니다!"); // (2). 스크립트가 모두 실행되고 난 후에야 DOMContentLoaded 이벤트 발생
  });
</script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>
<script>
  alert("라이브러리 로딩이 끝나고 인라인 스크립트가 실행되었습니다."); // (1)
</script>
```

DOMContentLoaded와 styles

- `DOMContentLoaded`는 외부 스타일시트가 로드되기를 기다리지 않음
  - 외부 스타일시트는 DOM에 영향을 주지 않기 때문
- 한 가지 예외 존재
  - 스타일시트를 불러오는 태그 바로 다음에 스크립트가 위치하면, 이 스크립트는 스타일시트가 로드되기 전까지 실행되지 않음
  - 스크립트에서 스타일에 영향을 받는 요소의 프로퍼티를 사용할 가능성이 있기 때문에 만들어진 예외

```html
<link type="text/css" rel="stylesheet" href="style.css" />
<script>
  alert(getComputedStyle(document.body).marginTop); // 이 스크립트는 위 스타일시트가 로드될 때까지 실행되지 않음
</script>
```

브라우저 내장 자동완성

- 폼 자동완성(form autofill)은 `DOMContentLoaded`에서 발생
  - Firefox, Chrome, Opera 등
- 페이지에 아이디와 비밀번호를 적는 폼이 있고, 브라우저에 아이디와 비밀번호가 저장된 경우
- 사용자가 자동완성을 허용했으면 `DOMContentLoaded` 이벤트가 발생할 때 인증 정보가 자동으로 채워짐
- 따라서 실행해야 할 스크립트가 길어서 `DOMContentLoaded` 이벤트가 지연되면 자동완성 역시 뒤늦게 처리됨

### window.onload

`window.onload`

```javascript
window.addEventListener("load", function (event) {});
window.onload = function (event) {};
```

- `window` 객체의 `load` 이벤트
- 이미지, 스타일 등의 리소스들이 모두 로드되었을 때 실행
- 모든 자원이 로드되는 것을 기다리기에는 시간이 오래 걸릴 수 있어 잘 사용되지 않는 이벤트

```html
<script>
  window.onload = function () {
    alert("페이지 전체가 로드되었습니다.");
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`); // 이미지가 제대로 불러와 진 후에 얼럿창 실행
  };
</script>
<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0" />
```

### window.onunload

`window.onunload`

```javascript
window.addEventListener("unload", function (event) {});
window.onunload = function (event) {};
```

- `window` 객체의 `unload` 이벤트
- 사용자가 페이지를 떠날 때, 즉 문서를 완전히 닫을 때 실행
- 딜레이가 없는 작업 수행 가능
  - 팝업창 닫기 등
- 분석 정보를 보내는 것은 예외사항에 속함
- 지연을 유발하는 복잡한 작업이나 사용자와의 상호작용 불가
- 이 제약사항 때문에 `unload` 이벤트는 아주 드물게 사용됨
- 예외적으로 `navigator.sendBeacon`으로 네트워크 요청 가능

`navigator.sendBeacon(url, data)`

- 사용자가 웹 사이트에서 어떤 행동을 하는지에 대한 분석 정보를 모으는 경우
- `unload` 이벤트는 사용자가 페이지를 떠날 때 발생하므로 이 이벤트에서 분석 정보를 서버로 보내는 것이 자연스러움
- 이런 용도를 위해 만들어진 메서드
- 데이터를 백그라운드에서 전송
  - 다른 페이지로 전환 시 분석 정보는 제대로 서버에 전송되지만, 딜레이가 없는 이유
- 요청은 POST 메서드로 전송됨
- 요청 시 문자열뿐만 아니라 폼이나 fetch에서 설명하는 기타 포맷들도 보낼 수 있음
  - 대개는 문자열 형태의 객체 전송
- 전송 데이터는 64kb를 넘을 수 없음
- 서버 응답을 받을 수 있는 방법이 없음
  - `sendBeacon` 요청이 종료된 시점에는 브라우저가 다른 페이지로 전환을 마친 상태일 확률이 높음
  - 사용자 분석 정보에 관한 응답은 대개 빈 상태
- fetch 메서드는 '페이지를 떠난 후'에도 요청이 가능하도록 하게 해주는 플래그 `keepalive`를 지원
- 다른 페이지로 전환 중에 이를 취소하고 싶은 경우, `unload`에서 페이지 전환 취소 불가
  - `onbeforeunload`는 가능

```javascript
let analyticsData = {}; // 분석 정보가 담긴 객체
window.addEventListener("unload", function () {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

### window.onbeforeunload

`window.onbeforeunload`

```javascript
window.addEventListener("beforeunload", function (event) {});
window.onbeforeunload = function (event) {};
```

- `window` 객체의 `beforeunload` 이벤트
- 사용자가 현재 페이지를 떠나 다른 페이지로 이동하려 할 때, 창을 닫으려 할 때
- `beforeunload` 핸들러에서 추가 확인 요청 가능
- `beforeunload` 이벤트를 취소하려 하면, 브라우저는 사용자에게 확인을 요청

```javascript
window.onbeforeunload = function () {
  return false;
};
```

- `false` 말고도 비어있지 않은 문자열을 반환하면 이벤트를 취소한 것과 같은 효과
- 역사적인 이유 때문에 남은 기능
- 과거에는 문자열을 반환하면, 브라우저에서 이 문자열을 보여줬었음
- 근래의 명세서에서는 권장하지 않음

```javascript
window.onbeforeunload = function () {
  return "저장되지 않은 변경사항이 있습니다. 정말 페이지를 떠나실 건 가요?";
};
```

- 문자열을 반환하도록 해도 얼럿창에 문자열이 보이지 않게 된 이유
- 몇몇 사이트 관리자들이 오해가 생길 법하거나 성가신 메시지를 띄우면서 `beforeunload`를 남용했기 때문
- 모던 브라우저에서는 `beforeunload` 이벤트를 취소할 때 보이는 메시지를 커스터마이징 할 수 없음

### readyState

`document.readyState` 프로퍼티

- 문서가 완전히 로드된 후 `DOMContentLoaded` 핸들러를 설정한다면 아마도 절대로 실행되지 않음
- 그런데 가끔은 문서가 로드되었는지 아닌지를 판단할 수 없는 경우가 있음
- DOM이 완전히 구성된 후에 특정 함수를 실행해야 할 때는 DOM 트리 완성 여부를 알 수 없어 난감
- 이럴 때 현재 로딩 상태를 알려주는 `document.readyState` 프로퍼티 사용 가능

`document.readyState` 프로퍼티의 세 종류 값

- `"loading"`: 문서를 불러오는 중일 때
- `"interactive"`: 문서가 완전히 불러와졌을 때
- `"complete"`: 문서를 비롯한 이미지 등의 리소스들도 모두 불러와졌을 때
- `document.readyState`의 값을 확인하고 상황에 맞게 핸들러를 설정하거나 코드를 실행하면 됨

```javascript
function work() {}
if (document.readyState == "loading") {
  // 아직 로딩 중이므로 이벤트를 기다림
  document.addEventListener("DOMContentLoaded", work);
} else {
  // DOM이 완성되었습니다!
  work();
}
```

`readystatechange`

- 상태가 변경되었을 때 실행되는 이벤트
- 문서 로딩 상태 파악 가능
- 아주 오래 전부터 있었던 이벤트
- 요즘에는 잘 사용하지 않음

```javascript
console.log(document.readyState); // 현재 상태
document.addEventListener("readystatechange", () =>
  console.log(document.readyState)
); // 상태 변경 출력
```

### 이벤트 순서 정리

```html
<script>
  log(`initial readyState: ${document.readyState}`); // 1
  document.addEventListener("readystatechange", () =>
    log(`readyState: ${document.readyState}`)
  ); // 2, 4
  document.addEventListener("DOMContentLoaded", () => log("DOMContentLoaded")); // 2
  window.onload = () => log("window onload"); // 4
</script>

<!-- 3 -->
<iframe src="iframe.html" onload="log('iframe onload')"></iframe>

<img src="http://en.js.cx/clipart/train.gif" id="img" />
<script>
  img.onload = () => log("img onload"); // 4
</script>
```

1. (1) initial readyState: loading
2. (2) readyState: interactive
3. (2) DOMContentLoaded
4. (3) iframe onload
5. (4) img onload
6. (4) readyState: complete
7. (4) window onload

- `document.readyState`는 `DOMContentLoaded`가 실행되기 바로 직전에 `interactive`가 됨
- 따라서 `DOMContentLoaded`와 `interactive`는 같은 상태를 나타낸다고 볼 수 있음
- `document.readyState`는 `iframe`, `img`를 비롯한 리소스 전부가 로드되었을 때 `complete`가 됨
- 위 예시에서 `readyState`의 값이 `img.onload`와 `window.onload`가 실행된 시점과 거의 동일한 시점에 `complete`로 바뀐 것을 확인 가능
- `readyState`의 값이 `complete`로 바뀐다는 것은 `window.onload`가 실행된다는 것과 동일한 의미
- 이 둘의 차이점은 `window.onload`는 다른 `load` 핸들러가 전부 실행된 후에야 동작한다는 것에 있음

## defer, async 스크립트

모던 웹 브라우저의 스크립트

- 모던 웹 브라우저에서 돌아가는 스크립트들은 대부분 HTML보다 무거움
- 용량이 커서 다운로드받는 데 오랜 시간이 걸리고 처리하는 것 역시 마찬가지
- 브라우저는 HTML을 읽다가 `<script>...</script>` 태그를 만나면 스크립트를 먼저 실행해야 해서 DOM 생성 중단
- `src` 속성이 있는 외부 스크립트 `<script src="...">...</script>`도 마찬가지
- 외부에서 스크립트를 다운로드 받고 실행한 후에야 남은 페이지 처리 가능

이런 브라우저의 동작 방식은 2가지 중요한 이슈를 만듦

1. 스크립트에서는 스크립트 아래에 있는 DOM 요소에 접근 불가
   - DOM 요소에 핸들러를 추가하는 것과 같은 여러 행위 불가
2. 페이지 위쪽에 용량이 큰 스크립트가 있는 경우, 스크립트가 페이지를 '막아버림'
   - 페이지에 접속하는 사용자들은 스크립트를 다운받고 실행할 때까지 스크립트 아래쪽 페이지를 볼 수 없음

```html
<p>...스크립트 앞 콘텐츠...</p>
<script src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
<!-- 스크립트 다운로드 및 실행이 끝나기 전까지 아래 내용이 보이지 않음 -->
<p>...스크립트 뒤 콘텐츠...</p>
```

스크립트를 페이지 맨 아래 놓기

- `<body>` 태그 맨 아래, `</body>` 태그 바로 위
- 스크립트 위에 있는 요소에 접근 가능
- 페이지 컨텐츠 출력을 막지 않음
- HTML 문서 자체가 아주 큰 경우, 브라우저가 HTML 문서 전체를 다운로드한 다음 스크립트를 다운받게 하면 페이지가 정말 느려짐
- 스크립트를 다운받는 도중에 사용자가 버튼을 클릭하는 등 페이지와 상호작용을 시도하면 제대로 동작하지 않음

`async`, `defer` 속성 사용하기

### defer

`defer` 속성

- 브라우저는 `defer` 속성이 있는 스크립트를 '백그라운드'에서 다운로드
  - defer 스크립트 또는 지연 스크립트
- 지연 스크립트를 다운로드 하는 도중에도 HTML 파싱이 멈추지 않음
- `defer` 스크립트 실행은 페이지 구성이 끝날 때까지 지연됨
- 외부 스크립트에만 유효
  - `<script>`에 `src`가 없으면 `defer` 속성은 무시됨

지연 스크립트(defer 스크립트)

- 페이지 생성을 절대 막지 않음
- DOM이 준비된 후, `DOMContentLoaded` 이벤트 발생 전 실행
- 일반 스크립트와 마찬가지로 HTML에 추가된 순(상대순, 요소순)으로 실행
  - 길이가 긴 스크립트가 앞에, 길이가 짧은 스크립트가 뒤에 있어도 짧은 스크립트는 긴 스크립트 실행 전까지 대기
  - 작은 스크립트가 먼저 다운로드되어도 실행은 나중에 됨
- 브라우저는 성능을 위해 페이지에 어떤 스크립트들이 있는지 쭉 살펴본 후, 스크립트를 병렬적으로 다운로드
  - 명세서에서 스크립트를 문서에 추가한 순서대로 실행하라고 정의함

```html
<p>...스크립트 앞 콘텐츠...</p>
<script
  defer
  src="https://javascript.info/article/script-async-defer/long.js?speed=1"
></script>
<!-- 바로 볼 수 있음 -->
<p>...스크립트 뒤 콘텐츠...</p>
```

```html
<p>...스크립트 앞 콘텐츠...</p>
<script>
  document.addEventListener("DOMContentLoaded", () =>
    alert("`defer` 스크립트가 실행된 후, DOM이 준비되었습니다!")
  ); // (2). DOMContentLoaded 이벤트는 지연 스크립트 실행을 기다림
</script>
<script
  defer
  src="https://javascript.info/article/script-async-defer/long.js?speed=1"
></script>
<p>...스크립트 뒤 콘텐츠...</p>
```

`defer` 스크립트 유의점

- 스크립트 다운로드가 끝나지 않았어도 페이지는 동작해야 함
- `defer`는 스크립트가 실행되기 전에 페이지가 화면에 출력
- 사용자는 그래픽 관련 컴포넌트들이 준비되지 않은 상태에서 화면을 보게 될 수 있음
- 사용자에게 현재 어떤 것은 사용할 수 있고, 어떤 것은 사용할 수 없는지 알려주기
  - 지연 스크립트가 영향을 주는 영역에 로딩 인디케이터 배치, 관련 버튼 사용 불가(disabled) 처리

`<head>` 태그 마지막에 `<script defer src="...">` 넣기

### async

`async` 속성

- 페이지와 완전히 독립적으로 동작
- defer 스크립트와 마찬가지로 백그라운드에서 다운로드
- HTML 페이지는 async 스크립트 다운로드 완료를 기다리지 않고 페이지 내 컨텐츠를 처리·출력
- async 스크립트 실행 중에는 HTML 파싱 정지

비동기 스크립트(async 스크립트)

- `DOMContentLoaded` 이벤트와 async 스크립트는 서로를 기다리지 않음
- 페이지 구성이 끝난 후 async 스크립트 다운로드가 끝난 경우
  - `DOMContentLoaded` -> async 스크립트 실행
- async 스크립트가 짧아 페이지 구성이 끝나기 전에 다운로드 되거나, 스크립트가 캐싱 처리된 경우
  - async 스크립트 실행 -> `DOMContentLoaded`
- async 스크립트가 여러 개 있는 경우, 실행 순서는 다운로드가 끝난 순으로 진행
- 다른 스크립트들은 async 스크립트를 기다리지 않고, async 스크립트 역시 다른 스크립트들을 기다리지 않음
  - load-first order: 먼저 로드 된 스크립트가 먼저 실행
- 현재 개발 중인 스크립트에 각각 독립적인 역할을 하는 서드 파티 스크립트를 통합할 때 유용
  - 방문자 수 카운터, 광고 관련 스크립트 등
  - 비동기 스크립트는 개발 중인 스크립트에 의존하지 않고, 그 반대도 마찬가지이기 때문

```html
<p>...스크립트 앞 콘텐츠...</p>
<script>
  document.addEventListener("DOMContentLoaded", () =>
    alert("DOM이 준비 되었습니다!")
  );
</script>
<script
  async
  src="https://javascript.info/article/script-async-defer/long.js"
></script>
<script
  async
  src="https://javascript.info/article/script-async-defer/small.js"
></script>
<p>...스크립트 뒤 콘텐츠...</p>
```

```html
<!-- Google Analytics는 일반적으로 다음과 같이 삽입 -->
<script async src="https://google-analytics.com/analytics.js"></script>
```

실무에서 `defer`와 `async`

- `defer`: DOM 전체가 필요한 스크립트나 실행 순서가 중요한 경우 적용
- `async`: 방문자 수 카운터, 광고 관련 스크립트 같이 독립적인 스크립트에 혹은 실행 순서가 중요하지 않은 경우 적용

### 동적 스크립트

동적 스크립트(dynamic script)

- 자바스크립트를 사용해 문서에 동적으로 추가한 스크립트
- 기본적으로 async 스크립트처럼 행동
  - `script.async=false`가 있다면 문서에 추가된 순서대로 실행
  - 없다면 load-first order로 실행
- 동적 스크립트는 그 어떤 것도 기다리지 않고 그 어떤 것도 동적 스크립트를 기다리지 않음
- 먼저 다운로드된 스크립트가 먼저 실행됨(load-first order)

```javascript
let script = document.createElement("script");
script.src = "/article/script-async-defer/long.js";
document.body.append(script); // 관련 요소에 추가되자 마자 외부 스크립트 다운로드 시작
```

```javascript
function loadScript(src) {
  let script = document.createElement("script");
  script.src = src;
  script.async = false;
  document.body.append(script);
}
loadScript("/article/script-async-defer/long.js"); // async=false이기 때문에 long.js 먼저 실행
loadScript("/article/script-async-defer/small.js");
```

## Resource loading: onload and onerror

`onload`, `onerror`

- 브라우저를 사용하면 scripts, iframes, pictures 등 외부 리소스의 로딩 추적 가능
- `onload`: 성공적인 로드 시 발생하는 이벤트
- `onerror`: 에러가 생길 시 발생하는 이벤트

### Loading a script

서드 파티 스크립트

- 아래와 같이 동적으로 로드
- 해당 스크립트 내에 선언된 함수를 실행하려면 스크립트가 로드될 때까지 기다리고 그 후에 호출 가능

```javascript
let script = document.createElement("script");
script.src = "my.js";
document.head.append(script);
```

`script.onload`

- `load` 이벤트가 주요 도우미
- 스크립트가 로드되고 실행된 후에 트리거 됨
- `onload` 안에서 스크립트 변수를 사용하고 함수를 실행할 수 있음

```javascript
let script = document.createElement("script");
script.src = "https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js";
document.head.append(script);
script.onload = function () {
  alert(_.VERSION); // 스크립트는 변수 _를 생성. 라이브러리 버전 출력
};
```

`script.onerror`

- `error` 이벤트로 스크립트 로딩 중 발생하는 에러 추적 가능
- HTTP 오류 세부 정보를 얻을 수 없음
- 로딩이 실패했다는 것만 알 수 있음

```javascript
let script = document.createElement("script");
script.src = "https://example.com/404.js"; // 존재하지 않는 스크립트 요청
document.head.append(script);
script.onerror = function () {
  alert("Error loading " + this.src); // Error loading https://example.com/404.js
};
```

`onload`, `onerror`

- 로드 자체만 추적
- 스크립트 처리 및 실행 중에 발생할 수 있는 에러는 이벤트의 범위 밖
- 스크립트가 성공적으로 로드되면 프로그래밍 에러가 있어도 `onload` 트리거
- 스크립트 에러를 추적하려면 전역 핸들러 `window.onerror` 사용

### Other resources

`load`, `error` 이벤트

- 기본적으로 외부 `src`가 있는 모든 리소스에 대해 다른 리소스에서도 작동
- 대부분의 리소스는 문서에 추가되면 로드되기 시작
- `<img>`는 예외로, `src`를 받으면 로드 시작
- `<iframe>`의 경우 `iframe.onload` 이벤트는 성공적으로 로드되거나 에러 상황 둘 다에서 iframe 로드가 끝나면 트리거 됨
- 역사적인 이유 때문

```javascript
let img = document.createElement("img");
img.src = "https://js.cx/clipart/train.gif"; // src를 받아 로드 시작
img.onload = function () {
  alert(`Image loaded, size ${img.width}x${img.height}`);
};
img.onerror = function () {
  alert("Error occurred while loading image");
};
```

`readystatechange` 이벤트

- 리소스에도 동작하지만 `load/error` 이벤트가 더 간단하기 때문에 거의 사용되지 않음

### Crossorigin policy

크로스 오리진 정책(교차 출처 정책)

- 한 사이트의 스크립트는 다른 사이트의 컨텐츠에 접근할 수 없는 규칙 존재
  - `https://facebook.com`의 스크립트는 `https://gmail.com`에 있는 유저의 메일함을 읽을 수 없음
- 하나의 오리진(도메인/포트/프로토콜 triplet)이 다른 오리진의 컨텐츠에 접근 불가
  - 서브 도메인을 갖거나, 단지 다른 포트여도 이들은 서로 접근할 수 없는 다른 오리진
- 이 규칙은 다른 도메인의 리소스에도 영향을 미침
- 다른 도메인의 스크립트를 사용하고 있는데 그 안에 에러가 있는 경우, 에러 세부정보를 얻을 수 없음

```javascript
// 📁 error.js
noSuchFunction(); // 잘못된 단일 함수 호출로 구성된 스크립트 `error.js`
```

해당 사이트와 동일한 사이트에서 로드

```html
<script>
  window.onerror = function (message, url, line, col, errorObj) {
    alert(`${message}\n${url}, ${line}:${col}`);
  };
</script>
<script src="/article/onload-onerror/crossorigin/error.js"></script>
```

```
Uncaught ReferenceError: noSuchFunction is not defined
https://javascript.info/article/onload-onerror/crossorigin/error.js, 1:1
```

다른 도메인에서 동일한 스크립트 로드

```html
<script>
  window.onerror = function (message, url, line, col, errorObj) {
    alert(`${message}\n${url}, ${line}:${col}`);
  };
</script>
<script src="https://cors.javascript.info/article/onload-onerror/crossorigin/error.js"></script>
```

```
Script error.
, 0:0
```

다른 도메인에서의 스크립트 내부 에러

- 세부사항은 브라우저에 따라 다를 수 있지만 아이디어는 동일
- 에러 스택 traces를 포함해 스트립트 내부에 대한 모든 정보는 숨겨짐
  - 정확히는 다른 도메인에서 왔기 때문

에러 세부정보가 필요한 이유

- `window.onerror`를 사용해 전역 에러를 수신·저장·접근·분석할 수 있는 인터페이스를 제공하는 많은 서비스(직접 구축할 수도 있음)가 있음
- 사용자에 의해 발생한 실제 에러를 볼 수 있다는 점은 훌륭함
- 그러나 스크립트가 다른 오리진에서 온 경우, 에러에 대한 정보가 많지 않음
- 다른 유형의 리소스에도 유사한 cross-origin policy(CORS)가 적용됨

cross-origin access를 허용하기 위한 방법

- `<script>` 태그에 `crossorigin` 속성 추가
- 원격 서버는 특수 헤더를 제공해야 함

cross-origin access의 세 가지 level

1. `crossorigin` 속성 없음
   - 접근 금지됨
2. `crossorigin="anonymous"`
   - 서버가 헤더 `Access-Control-Allow-Origin`을 `*` 혹은 원본 오리진으로 응답하는 경우 엑세스 허용
   - 브라우저는 인증 정보(authorization information)와 쿠키를 원격 서버로 보내지 않음
3. `crossorigin="use-credentials"`
   - 서버가 헤더 `Access-Control-Allow-Origin`을 원본 오리진으로 하고
   - `Access-Control-Allow-Credentials: true`로 다시 보내는 경우 엑세스 허용
   - 브라우저는 인증 정보와 쿠키를 원격 서버로 보냄

```html
<script>
  window.onerror = function (message, url, line, col, errorObj) {
    alert(`${message}\n${url}, ${line}:${col}`);
  };
</script>
<script
  crossorigin="anonymous"
  src="https://cors.javascript.info/article/onload-onerror/crossorigin/error.js"
></script>
```

- `crossorigin` 속성이 없었기 때문에 크로스-오리진 엑세스가 금지됐었음
- `"anonymous`: 쿠키 전송 안 함, 서버 측 1개 헤더 필요
- `"use-credentials"`: 쿠키도 전송, 서버 측 헤더 2개 필요
- 서버가 `Access-Control-Allow-Origin` 헤더를 제공한다고 가정하면 `crossorigin="anonymous"`로 전체 에러 보고서를 받을 수 있음

### 예제

Load images with a callback

- 일반적으로 이미지는 생성될 때 로드됨
- 우리가 `<img>`를 페이지에 추가하면 사용자에게 사진이 즉시 표시되지 않음
- 브라우저가 이를 먼저 로드해야 함

```javascript
let img = document.createElement("img");
img.src = "my.jpg";
```

- 이미지를 즉시 표시하려면 위와 같이 미리 생성하면 됨
- 브라우저는 이미지 로드를 시작하고 이를 캐시에 기억
- 나중에 문서에 동일한 이미지가 나타나면(어떻게든) 즉시 나타남

`preloadImages(sources, callback)` 함수를 생성

- 배열 `sources`에서 모든 이미지를 로드하고 준비되면 `callback`을 실행

```javascript
function loaded() {
  alert("Images loaded");
}
preloadImages(["1.jpg", "2.jpg", "3.jpg"], loaded); // 이미지 로드 시 loaded 실행
```

- 에러가 발생한 경우에도 함수는 그림이 로드된 것으로 가정해야함
- 즉, 모든 이미지들이 로드되거나 에러가 발생하면 `callback` 실행됨
- 예를 들어 스크롤 가능한 이미지가 많은 갤러리를 표시할 계획이고, 모든 이미지가 로드되었는지 확인하고 싶은 경우 이 함수는 유용

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
  </head>
  <body>
    <script>
      function preloadImages(sources, callback) {
        let counter = 0;
        function onLoad() {
          counter++;
          if (counter == sources.length) callback();
        }
        for (let source of sources) {
          let img = document.createElement("img");
          img.onload = img.onerror = onLoad;
          img.src = source;
        }
      }

      // ---------- The test ----------

      let sources = [
        "https://en.js.cx/images-load/1.jpg",
        "https://en.js.cx/images-load/2.jpg",
        "https://en.js.cx/images-load/3.jpg"
      ];

      // add random characters to prevent browser caching
      for (let i = 0; i < sources.length; i++)
        sources[i] += "?" + Math.random();

      // for each image, let's create another img with the same src and check that we have its width
      function testLoaded() {
        let widthSum = 0;
        for (let i = 0; i < sources.length; i++) {
          let img = document.createElement("img");
          img.src = sources[i];
          widthSum += img.width;
        }
        alert(widthSum);
      }

      preloadImages(sources, testLoaded); // should output 300
    </script>
  </body>
</html>
```

## 참고

> [문서와 리소스 로딩](https://ko.javascript.info/loading)
