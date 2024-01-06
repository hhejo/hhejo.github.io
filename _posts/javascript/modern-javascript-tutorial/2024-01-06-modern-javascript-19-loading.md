---
title: 모던 JavaScript 튜토리얼 19 - 문서와 리소스 로딩
date: 2024-01-06 08:15:11 +0900
last_modified_at: 2024-01-06 08:15:11 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

DOMContentLoaded, load, beforeunload, unload 이벤트, defer, async 스크립트, Resource loading: onload and onerror

## DOMContentLoaded, load, beforeunload, unload 이벤트

HTML 문서의 생명주기

- 3가지 주요 이벤트가 관여
- `DOMContentLoaded`
  - 브라우저가 HTML을 읽고 DOM 트리를 완성하는 즉시 발생
  - 이미지 파일(`<img>`), 스타일 시트 등의 기타 자원은 기다리지 않음
  - DOM이 준비된 것을 확인 후, 원하는 DOM 노드를 찾아 핸들러를 등록해 인터페이스를 초기화 할 때 사용
- `load`
  - HTML로 DOM 트리를 완성되었을 뿐만 아니라 이미지, 스타일 시트 같은 외부 자원도 모두 불러오는 것이 끝났을 때 발생
  - 외부 자원이 로드된 후이기 때문에 스타일이 적용된 상태이므로 화면에 뿌려진 요소의 실제 크기 확인 가능
  - 이미지 사이즈를 확인할 때 등에 사용
- `beforeunload/unload`
  - 사용자가 페이지를 떠날 때 발생
  - `beforeunload`: 사용자가 사이트를 떠나려 할 때, 변경되지 않은 사항들을 저장했는지 확인시켜줄 때 사용
  - `unload`: 사용자가 진짜 떠나기 전, 사용자 분석 정보를 담은 통계 자료를 전송하고자 할 때 사용

### DOMContentLoaded

`DOMContentLoaded` 이벤트

```javascript
document.addEventListener("DOMContentLoaded", ready); // document.onDOMContentLoaded는 동작하지 않음
```

- `document` 객체에서 발생
  - `addEventListener`로 다룸
- 문서가 로드되었을 때 실행되므로 모든 요소에 접근 가능
- DOM 트리가 완성되면 발생하는 이벤트 같지만, 몇 가지 특이사항 존재

```html
<script>
  function ready() {
    alert("DOM이 준비되었습니다!");
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`); // 이미지가 로드되지 않은 상태이기 때문에 사이즈는 0x0
  }
  document.addEventListener("DOMContentLoaded", ready); // 문서가 로드되었을 때 실행. <img> 포함 모든 요소에 접근 가능
</script>
<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0" />
```

DOMContentLoaded와 scripts

- 브라우저는 HTML 문서를 처리하는 도중에 `<script>` 태그를 만나는 경우,
- DOM 트리 구성을 멈추고 `<script>`를 실행
- 스크립트 실행이 끝난 후에야 나머지 HTML 문서를 처리
- `<script>`에 있는 스크립트가 DOM 조작 관련 로직을 담고 있을 수 있기 때문에 만들어진 방지책
- `DOMContentLoaded` 이벤트 역시 `<script>` 안의 스크립트가 처리된 후 발생

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

DOMContentLoaded를 막지 않는 스크립트

- 위와 같은 규칙에는 두 가지 예외사항 존재
- `async` 속성이 있는 스크립트
- `document.createElement('script')`로 동적으로 생성되고 웹 페이지에 추가된 스크립트

DOMContentLoaded와 styles

- 외부 스타일시트는 DOM에 영향을 주지 않기 때문에
- `DOMContentLoaded`는 외부 스타일시트가 로드되기를 기다리지 않음
- 한 가지 예외 존재
- 스타일시트를 불러오는 태그 바로 다음에 스크립트가 위치하는 경우,
- 이 스크립트는 스타일시트가 로드되기 전까지 실행되지 않음
- 스크립트에서 스타일에 영향을 받는 요소의 프로퍼티를 사용할 가능성이 있기 때문에 만들어진 예외

```html
<link type="text/css" rel="stylesheet" href="style.css" />
<script>
  alert(getComputedStyle(document.body).marginTop); // 이 스크립트는 위 스타일시트가 로드될 때까지 실행되지 않음
</script>
```

- 스크립트에서 요소 정보를 활용하고 있음
- 스타일이 로드되고 적용된 후에야 좌표 정보가 확정됨

브라우저 내장 자동완성

- 폼 자동완성(form autofill)은 `DOMContentLoaded`에서 발생
  - Firefox, Chrome, Opera 등
- 페이지에 아이디와 비밀번호를 적는 폼이 있고, 브라우저에 아이디와 비밀번호가 저장된 경우
- `DOMContentLoaded` 이벤트가 발생할 때 인증 정보가 자동으로 채워짐
  - 사용자가 자동완성을 허용했을 때
- 따라서 실행해야 할 스크립트가 길어서 `DOMContentLoaded` 이벤트가 지연되는 경우,
- 자동완성 역시 뒤늦게 처리됨

### window.onload

`window.onload`

```javascript
window.addEventListener("load", function (event) {});
window.onload = function (event) {};
```

- `window` 객체의 `load` 이벤트
- 이미지, 스타일 등의 리소스들이 모두 로드되었을 때 실행
- 모든 자원이 로드되는 것을 기다리기에는 시간이 오래 걸릴 수 있기 때문에 잘 사용되지 않는 이벤트

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
- 예외적으로 아래 `navigator.sendBeacon`을 사용해 네트워크 요청 가능

`navigator.sendBeacon(url, data)`

- 사용자가 웹 사이트에서 어떤 행동을 하는지에 대한 분석 정보를 모으는 경우
- `unload` 이벤트는 사용자가 페이지를 떠날 때 발생
- `unload` 이벤트에서 분석 정보를 서버로 보내는 것이 자연스러움
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
- 다른 페이지로 전환 중에 이를 취소하고 싶은 경우,
- `unload`에서는 페이지 전환을 취소할 수 없음
- `onbeforeunload` 사용 시 가능

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

문서가 완전히 로드된 후 `DOMContentLoaded` 핸들러를 설정하면?

- 아마도 절대로 실행되지 않음
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

### defer

### async

### 동적 스크립트

### 요약

## Resource loading: onload and onerror

### Loading a script

### Other resources

### Crossorigin policy

### 요약

## 참고

> [문서와 리소스 로딩](https://ko.javascript.info/loading)
