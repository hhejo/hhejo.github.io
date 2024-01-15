---
title: 모던 JavaScript 튜토리얼 21 - 프레임과 윈도우
date: 2024-01-13 08:22:16 +0900
last_modified_at: 2024-01-15 08:22:16 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

Popups and window methods, Cross-window communication, The clickjacking attack

## Popups and window methods

팝업 창(popup window)

- 사용자에게 추가 문서를 표시하는 가장 오래된 방법 중 하나

```javascript
window.open("https://javascript.info/");
```

- 주어진 URL 이 포함된 새 창이 열림
- 대부분의 최신 브라우저는 별도의 창 대신 새 탭에서 URL을 열도록 구성됨
- 아주 옛날부터 존재했음
- 초기 아이디어는 메인 창을 닫지 않고 다른 컨텐츠를 표시하는 것이었음
- 현재는 이를 수행하는 다른 방법을 사용
  - `fetch`로 컨텐츠를 동적으로 로드하고 동적으로 생성된 `<div>`에 표시
- 여러 창을 동시에 표시하지 않는 모바일 장치에서는 팝업이 까다로움

팝업이 여전히 사용되는 작업

- 위의 단점에도 불구하고 다음과 같은 이유로 팝업이 여전히 사용되는 작업이 있음
  - OAuth 승인(Google/Facebook/...으로 로그인) 등
- 팝업은 자체적으로 독립적인 자바스크립트 환경을 갖는 별도의 창
  - 신뢰할 수 없는 제3자 사이트에서 팝업을 여는 것은 안전
- 팝업을 여는 것은 매우 쉬움
- 팝업은 탐색(URL 변경)하고 오프너(opener) 창으로 메시지 전송 가능

### Popup blocking

팝업을 차단하고 사용자를 보호

- 과거에는 악성 사이트에서 팝업을 많이 악용했음
- 이제 대부분의 브라우저는 팝업을 차단하고 사용자를 보호
- 대부분의 브라우저는 `onclick` 같은 사용자 트리거 이벤트 핸들러 외부에서 호출되는 경우 팝업을 차단

```javascript
window.open("https://javascript.info"); // 팝업 차단
button.onclick = () => {
  window.open("https://javascript.info"); // 팝업 허용
};
setTimeout(() => window.open("http://google.com"), 1000); // 1초 후 팝업 열림
setTimeout(() => window.open("http://google.com"), 3000); // Firefox에서는 열리지 않음
```

- 원치 않는 팝업으로부터 어느 정도 사용자가 보호되지만 완전히 막을 수는 없음
- Firefox에서는 2000ms 이후에는 신뢰를 제거해 팝업이 열리지 않음

### window.open

```javascript
window.open(url, name, params);
```

- `url`: 새 창에 로드할 URL
- `name`: 새 창의 이름
  - 각 창에는 `window.name`이 있음
  - 팝업에 사용할 창 지정
  - 해당 이름의 창이 있는 경우 해당 URL이 해당 창에서 열리고, 그렇지 않다면 새 창이 열림
- `params`: 새 창의 구성 문자열. 쉼표로 구분된 설정을 포함하고 공백이 없어야 함
  - `width=200,height=100`

`params`에 대한 설정

- `left/top`(숫자): 화면의 창 왼쪽 상단 모서리 좌표
  - 새 창은 화면 밖에 위치할 수 없음
- `width/height`(숫자): 새 창의 너비·높이
  - 최소 제한이 있어 보이지 않는 창을 만들 수 없음
- `menubar`(yes/no): 새 창에 브라우저 메뉴를 표시/숨김
- `toolbar`(yes/no): 새 창에서 브라우저 탐색 표시줄을 표시/숨김
- `location`(yes/no): 새 창에 URL 필드를 표시/숨김
- `status`(yes/no): 상태 표시줄을 표시/숨김(대부분의 브라우저는 강제로 표시)
- `resizable`(yes/no): 새 창의 크기 조정을 비활성화(권장되지 않음)
- `scrollbars`(yes/no): 새 창의 스크롤 막대 비활성화(권장되지 않음)

### Example: a minimalistic window

```javascript
let params = `scrollbars=no,resizable=no,status=no,location=no,toolbar=no,menubar=no,
width=0,height=0,left=-1000,top=-1000`;
open("/", "test", params); // 크롬에서 새 탭으로 열림..
```

```javascript
let params = `scrollbars=no,resizable=no,status=no,location=no,toolbar=no,menubar=no,
width=600,height=300,left=100,top=100`;
open("/", "test", params);
```

생략된 설정에 대한 규칙

- `open` 호출에 3번째 인수가 없거나 비었으면 기본 창 매개변수가 사용됨
- 매개변수 문자열이 있으나 `yes/no`가 생략된 경우 `no`로 간주
- `left/top`이 없으면 브라우저는 마지막으로 열린 창 근처에서 새 창을 열음
- `width/height`가 없으면 새 창은 마지막으로 열었던 것과 동일한 크기가 됨

### Accessing popup from window

`open`

- 호출하면 새 창에 대한 참조를 반환
- 프로퍼티를 조작하고 위치를 변경하는 등의 작업에 사용

```javascript
let newWin = window.open("about:blank", "hello", "width=200,height=200");
newWin.document.write("Hello, world!"); // 자바스크립트로 팝업 컨텐츠 생성
```

```javascript
let newWindow = open("/", "example", "width=300,height=300");
newWindow.focus();
alert(newWindow.location.href); // (*) about:blank, loading hasn't started yet
newWindow.onload = function () {
  let html = `<div style="font-size:30px">Welcome!</div>`;
  newWindow.document.body.insertAdjacentHTML("afterbegin", html);
};
```

- `window.open` 직후에는 새 창이 아직 로드되지 않음
  - `(*)`에서 확인 가능
- `onload`가 그것을 수정하기를 기다림
- `DOMContentLoaded` 핸들러를 `newWinow.document`에 사용할 수도 있음

Same origin policy

- 창들은 동일한 출처(same origin)에서 온 경우에만 서로의 컨텐츠에 자유롭게 접근(access)
  - Same origin: same `protocol://domain:port`
- 메인 창이 `site.com`에 있고 팝업이 `gmail.com`에 있는 경우, 사용자 안전상의 이유로 불가

### Accessing window from popup

`window.opener`

- `window.opener` 참조를 사용해 `opener` 창에 접근 할 수 있음
- 팝업을 제외한 모든 창에서 `null`
- 창 사이의 연결은 양방향
  - 메인 창과 팝업은 서로 참조

```javascript
let newWin = window.open("about:blank", "hello", "width=200,height=200");
newWin.document.write(
  "<script>window.opener.document.body.innerHTML = 'Test'</script>"
); // opener(현재) 창의 내용이 "Test"로 대체됨
```

### Closing a popup

`win.close()`

- 창 닫기
- 기술적으로는 모든 `window`에서 사용 가능한 메서드
- `window`가 `window.open()`으로 생성되지 않았다면 대부분의 브라우저에서 `window.close()`는 무시됨
- 따라서 팝업에서만 작동

`win.closed`

- 창이 닫혔는지 확인
- 닫혔으면 `true`
- 팝업이나 메인 창이 열려있는지 아닌지 확인하는 데 유용
- 사용자는 언제든지 닫을 수 있기 때문에 이런 가능성을 고려해야 함

```javascript
let newWindow = open("/", "example", "width=300,height=300");
newWindow.onload = function () {
  newWindow.close(); // 창을 로드한 후 닫음
  alert(newWindow.closed); // true
};
```

### Scrolling and resizing

`win.moveBy(x, y)`

- 현재 위치를 기준으로 창을 오른쪽으로 `x`px, 아래 쪽으로 `y`px 이동
- 음수 허용

`win.moveTo(x, y)`

- 창을 `(x, y)` 좌표로 이동

`win.resizeBy(width, height)`

- 현재 크기를 기준으로 `width/height`만큼 창 크기 조정
- 음수 허용

`win.resizeTo(width, height)`

- 창을 주어진 크기로 조정

`window.onresize` 이벤트

추가 탭이 없는 팝업에서만 안정적으로 작동

- 남용을 방지하기 위해 브라우저는 일반적으로 이러한 방법을 차단

이동/크기 조정 방법은 최대화/최소화된 창에서는 작동하지 않음

- 자바스크립트에는 창을 축소하거나 최대화할 수 있는 방법이 없음
- 이러한 OS 수준 기능은 FE 개발자에게 숨겨짐

### Scrolling a window

`win.scrollBy(x, y)`

`win.scrollTo(x, y)`

`elem.scrollIntoView(top=true)`

`window.onscroll`

### Focus/blur on a window

`window.focus()`, `window.blur()`

- 이론적으로 두 메서드를 사용해 창에 포커스하고 해제하는 것이 가능

`focus/blur` 이벤트

- 방문자가 창에 집중하고 다른 곳으로 전환하는 순간을 포착

```javascript
window.onblur = () => window.focus();
```

- 사용자가 창 밖으로 전환하려 하면 창에 다시 포커스됨

실제로는 과거에 사악한 페이지가 이를 남용했기 때문에 사용이 제한됨

- 모바일 브라우저는 일반적으로 `window.focus()`를 완전히 무시
- 새 창이 아닌 별도의 탭에서 팝업이 열리는 경우에도 포커스 작동하지 않음

유용할 수 있는 사례

- 팝업을 열 때 `newWindow.focus()`를 실행하는 것이 좋음
  - 일부 OS/브라우저 조합의 경우 사용자가 새 창에 있는지 확인
- 방문자가 실제로 웹앱을 사용하는 시점을 `window.onfocus/onblur`를 이용해 추적할 수 있음
  - 페이지 내 활동, 애니메이션 등을 일시중지하거나 재개할 수 있음

## Cross-window communication

Same Origin Policy(동일 출처 정책)

- 창과 프레임 간의 접근을 제한
- 정보 도용으로부터 사용자를 보호
  - 사용자가 두 개의 페이지(`john-smith.com`, `gmail.com`)를 열어 둔 경우
  - `john-smith.com`의 스크립트가 `gmail.com`의 메일을 읽는 것을 원하지 않을 것

### Same Origin

두 URL이 동일한 프로토콜, 도메인, 포트를 갖는 경우 동일한 출처(same origin)를 갖는다고 함

다음 URL은 모두 동일한 출처를 공유

- `http://site.com`
- `http://site.com/`
- `http://site.com/my/page.html`

다음은 그렇지 않음

- `http://www.site.com`: 다른 도메인(`www.`)
- `http://site.org`: 다른 도메인(`.org`)
- `https://site.com`: 다른 프로토콜(`https`)
- `http://site.com:8080`: 다른 포트(`8080`)

동일 출처 정책은 다음과 같이 명시함

- 다른 창에 대한 참조가 있고, 해당 창이 동일한 출처에서 온 경우 해당 창에 대한 전체 접근 권한이 있음
  - 다른 창: `window.open`으로 생성된 팝업, `<iframe>` 내부의 창
- 그렇지 않고 다른 출처에서 온 경우 해당 창의 내용(변수, 문서 등)에 접근 불가
- 유일한 예외는 `location`
  - 변경해서 사용자를 리디렉션함
- 그러나 `location`을 읽을 수 없고 사용자가 현재 어디에 있는지 알 수 없어 정보가 유출되지 않음

In action: iframe

`<iframe>`

- 별도의 `document`, `window` 객체를 포함하는 별도의 내장 창을 호스팅
- `iframe.contentWindow`: `<iframe>` 내부의 창 가져오기
- `iframe.contentDocument`: `<iframe>` 내부의 문서 가져오기
  - `iframe.contentWindow.document`로 축약
- 포함된 창 내부에 접근하면 브라우저는 iframe 출처가 동일한지 확인
- 그렇지 않으면 접근 거부
- `location`에 쓰는 것은 예외이지만 여전히 허용됨

```html
<iframe src="https://example.com" id="iframe"></iframe>
<script>
  iframe.onload = function () {
    let iframeWindow = iframe.contentWindow; // OK (내부 창에 대한 참조를 얻을 수 있음)
    try {
      let doc = iframe.contentDocument; // ERROR (내부의 문서는 불가)
    } catch (e) {
      alert(e); // Security Error (다른 출처)
    }
    try {
      let href = iframe.contentWindow.location.href; // ERROR (Location 객체에서 URL을 읽을 수 없음)
    } catch (e) {
      alert(e); // Security Error (iframe의 페이지 URL을 읽을 수 없음)
    }
    iframe.contentWindow.location = "/"; // OK (location에 쓸 수 있고 iframe에 뭔가를 로드할 수 있음)
    iframe.onload = null; // clear the handler, not to run it after the location change
  };
</script>
```

- 다른 원본에서 읽고 쓰기 시도
- 다음을 제외한 모든 작업에 대한 에러를 표시
  - 내부 창에 대한 참조를 얻는 `iframe.contentWindow`
  - `location`에 쓰기

```html
<!-- iframe from the same site -->
<iframe src="/" id="iframe"></iframe>
<script>
  iframe.onload = function () {
    iframe.contentDocument.body.prepend("Hello, world!"); // 무엇이든 작업 가능
  };
</script>
```

- iframe이 동일한 출처를 가지고 있다면 그것을 가지고 무엇이든 할 수 있음

`iframe.onload`와 `iframe.contentWindow.onload`

- `iframe.onload`이벤트(`<iframe>` 태그의)는 `iframe.contentWindow.onload`(임베드된 window 객체)와 기본적으로 동일
- 임베드된 창이 모든 리소스와 함께 완전히 로드되면 트리거됨
- 그러나 다른 출처의 iframe에서 `iframe.contentWindow.onload`에 접근할 수 없으므로 `iframe.onload`를 사용

### Windows on subdomains: document.domain

정의에 따르면 도메인이 다른 두 URL의 출처는 다름

그러나 창이 동일한 두 번째 수준 도메인을 공유하는 경우

- 예를 들어 `john.ite.com`, `peter.site.com`, `site.com`
- 이들의 공통 두 번째 수준 도메인은 `site.com`

창 간 통신을 목적으로 브라우저가 해당 차이점을 무시해 '동일한 출처'에서 온 것으로 처리될 수 있음

작동하게 하려면 각 창에서 다음 코드를 실행

```javascript
document.domain = "site.com";
```

이제 제한 없이 상호작용 가능

다시 말하지만, 이는 동일한 두 번째 수준 도메인이 있는 페이지에만 가능

### Iframe: wrong document pitfall

iframe이 동일한 출처에서 왔으며 그것의 `document`에 접근할 수 있는 경우, 함정이 있음

- 교차 출처와 관련은 없지만 아는 것이 중요
- iframe이 생성되면 즉시 문서(document)가 생성됨
- 하지만 해당 문서는 로드되는 문서와 다름
- 따라서 문서에 대해 즉시 작업을 수행하면 해당 내용이 손실될 수 있음

```html
<iframe src="/" id="iframe"></iframe>
<script>
  let oldDoc = iframe.contentDocument;
  iframe.onload = function () {
    let newDoc = iframe.contentDocument;
    alert(oldDoc == newDoc); // false (로드된 문서는 처음과 같지 않음)
  };
</script>
```

아직 로드되지 않은 iframe의 문서로 작업하면 안 됨

- 그 문서는 잘못된 문서이기 때문
- 이벤트 핸들러를 설정하면 무시됨

문서가 있는 순간을 감지하는 방법

- 올바른 문서는 `iframe.onload`가 트리거 되면 확실히 제자리에 있음
- 그러나 모든 리소스가 포함된 전체 iframe이 로드될 때만 트리거됨
- `setInterval`로 검사해 그 순간을 더 빨리 포착할 수 있음

```html
<iframe src="/" id="iframe"></iframe>
<script>
  let oldDoc = iframe.contentDocument;
  let timer = setInterval(() => {
    let newDoc = iframe.contentDocument;
    if (newDoc == oldDoc) return;
    alert("New document is here!");
    clearInterval(timer); // 더이상 필요하지 않으므로 취소함
  }, 100); // 문서가 새것인지 100ms마다 체크
</script>
```

### Collection: window.frames

`<iframe>`의 window 객체를 가져오는 다른 방법

- `window.frames` 기명 컬렉션에서 가져오기
- `window.frames[0]`: document의 첫번쩨 프레임에 대한 window 객체
- `window.frames.iframeName`: `name="iframeName"`

```html
<iframe src="/" style="height:80px" name="win" id="iframe"></iframe>
<script>
  alert(iframe.contentWindow == frames[0]); // true
  alert(iframe.contentWindow == frames.win); // true
</script>
```

iframe 내부에는 다른 iframe이 있을 수 있음

- 해당 `window` 객체는 계층 구조를 형성

탐색 링크

- `window.frames`: children 창의 컬렉션(중첩된 프레임용)
- `window.parent`: parent(외부) 창에 대한 참조
- `window.top`: 최상위 창에 대한 참조
  - 현재 문서가 프레임 내부에 열려 있는지 확인 가능

```javascript
window.frames[0].parent === window; // true
```

```javascript
// current window == window.top?
if (window == top) {
  alert("The script is in the topmost window, not in a frame");
} else {
  alert("The script runs in a frame!");
}
```

### The "sandbox" iframe attribute

### Cross-window messaging

`postMessage` 인터페이스

- window의 출처에 관계 없이 서로 대화할 수 있음
- 동일 출처 정책을 우회하는 방법
- `john-smith.com`에서 `gmail.com`에 대화하고 정보 교환 가능
- 둘 다 동의하고 해당 자바스크립트 함수를 호출하는 경우에만 가능
- 그래야 사용자가 안전해짐
- 인터페이스는 두 부분으로 구성됨

postMessage

```javascript
postMessage(data, targetOrigin);
```

- 메시지를 보내려는 window는 수신하는 window의 `poseMessage`를 호출
- 즉, 메시지를 `win`에 보내려면 `win.postMessage(data, targetOrigin)`을 호출해야 함
- `data`: 전송할 데이터
  - 모든 객체가 될 수 있으며 데이터는 structured cloning algorithm을 사용하여 복제됨
- `targetOrigin`: 지정된 오리진의 window만 메시지를 받을 수 있도록 대상 window의 오리진을 지정
  - 안전 조치
  - 지정하면 window는 여전히 올바른 사이트에 있는 경우에만 데이터를 수신
  - 데이터가 민감한 경우 중요
  - `*`로 설정하면 확인하지 않음

```html
<iframe src="http://example.com" name="example"></iframe>
<script>
  let win = window.frames.example;
  win.postMessage("message", "http://example.com"); // win은 오리진 http://example.com의 문서가 있는 경우에만 메시지를 수신
  // win.postMessage("message", "*");
</script>
```

onmessage

- 메시지를 받으려면 대상 window의 `message`이벤트에 핸들러가 있어야 함
- `postMessage`가 호출되면 트리거됨(`targetOrigin` 확인도 성공적인 경우)
- 이벤트 객체의 특별한 프로퍼티
- `data`: `postMessage`로부터의 데이터
- `origin`: 발신자의 오리진
- `source`: 보낸 window에 대한 참조
  - `source.postMessage(...)`로 돌아갈 수 있음
- `addEventListener`를 사용해 핸들러를 할당
  - `window.onmessage`는 동작하지 않음

```javascript
window.addEventListener("message", function (event) {
  if (event.origin != "http://javascript.info") return; // 모르는 도메인은 무시
  alert("received: " + event.data);
  // can message back using event.source.postMessage(...)
});
```

## The clickjacking attack

클릭재킹 공격

- 악의적인 페이지가 방문자를 대신해 피해자 사이트를 클릭할 수 있음

### The idea

Facebook에서 클릭재킹이 수행된 방법

1. 방문자가 사악한 페이지로 유도됨
   - 방법은 중요하지 않음
2. 페이지에 무해해 보이는 링크가 있음
   - 지금 부자가 되세요, 여기를 클릭하세요 매우 재밌습니다
3. 해당 링크 위에는 사악한 페이지가 `src`가 facebook.com인 투명한 iframe을 배치해 좋아요 버튼이 해당 링크 바로 위에 위치하게 함
   - 일반적으로 `z-index`로 수행됨
4. 방문자는 링크를 클릭하려고 하면서 실제로 버튼을 클릭하게 됨

```html
<style>
  iframe {
    /* iframe from the victim site */
    width: 400px;
    height: 100px;
    position: absolute;
    top: 0;
    left: -20px;
    opacity: 0.5; /* 실제로는 opacity:0 */
    z-index: 1;
  }
</style>
<div>Click to get rich now:</div>
<!-- The url from the victim site -->
<iframe src="/clickjacking/facebook.html"></iframe>
<button>Click here!</button>
<div>...And you're cool (I'm a cool hacker actually)!</div>
```

- 버튼을 클릭하면 실제로 iframe이 클릭되지만 투명(현재는 반투명)하기 때문에 사용자에게는 표시되지 않음
- 결과적으로 방문자가 Facebook에서 승인을 받은 경우(대개 기억하기(remember me)가 켜져 있음) 좋아요가 추가됨
  - 트위터에서는 팔로우 버튼이 될 것

클릭재킹은 키보드가 아닌 클릭을 위한 것

- 공격은 마우스 동작(또는 모바일의 탭과 같은 유사한 동작)에만 영향을 미침
- 키보드 입력은 리디렉션하기 훨씬 어려움
- 기술적으로 해킹할 텍스트 필드가 있는 경우 텍스트 필드가 서로 겹치는 방식으로 iframe을 배치할 수 있음
- 따라서 방문자가 페이지에 표시되는 입력에 집중하려고 하면 실제로는 iframe 내부의 입력에 집중하게 됨
- 그런데 iframe이 표시되지 않기 때문에 방문자가 입력하는 모든 내용은 숨겨짐
- 사람들은 일반적으로 화면에 새 문자가 인쇄되는 것을 볼 수 없으면 입력을 중단

### Old-school defences (weak)

- 가장 오래된 방어 수단은 프레임에서 페이지를 여는 것을 금지하는(소위 framebusting) 약간의 자바스크립트

```javascript
if (top != window) {
  top.location = window.location;
}
```

- 즉, 창에서 자신이 맨 위에 있지 않다는 사실을 알게 되면 자동으로 맨 위가 됨
- 해킹할 수 있는 방법이 많기 때문에 신뢰할 수 있는 방어가 아님

Blocking top-navigation

- `beforeunload` 이벤트 핸들러에서 `top.location` 변경으로 인한 전환을 차단할 수 있음
- 최상위 페이지(해커가 속한 페이지)는 다음과 같이 방지 핸들러를 설정

```javascript
window.onbeforeunload = function () {
  return false;
};
```

- iframe이 `top.location` 변경을 시도하면, 방문자는 떠날 것인지 묻는 메시지를 받음
- 대부분의 경우 방문자는 iframe에 대해 모르기 때문에 부정적으로 대답
- 방문자가 볼 수 있는 것은 상단 페이지 뿐이므로 떠날 이유가 없음
- 그러니 `top.location`은 변하지 않을 것

Sandbox attribute

- `sandbox` 속성에 의해 제한되는 것 중 하나는 navigation
- 샌드박스가 적용된 iframe은 `top.location`을 변경하지 않을 것
- 따라서 iframe에 `sandbox="allow-scripts allow-forms"`를 추가할 수 있음
- 그러면 제한이 완회되어 스크립트와 폼이 허용됨
- 그러나 우리는 `allow-top-navigation`을 생략하기 때문에 `top.location`을 변경하는 것은 금지됨

```html
<iframe sandbox="allow-scripts allow-forms" src="facebook.html"></iframe>
```

### X-Frame-Options

`X-Frame-Options`

- 서버 측 `X-Frame-Options` 헤더는 프레임 내부의 페이지 표시를 허용하거나 금지할 수 있음
- 이는 정확히 HTTP 헤더로 전송되어야 함
- 브라우저는 HTML `<meta>` 태그에 있는 경우 이를 무시
- 그러니 `<meta http-equiv="X-Frame-Options"...>`는 아무것도 하지 않을 것임
- 헤더에는 3가지 값이 있을 수 있음
- `DENY`: 절대로 프레임 않에 페이지를 표시하지 마세요
- `SAMEORIGIN`: 상위 문서가 동일한 출처에서 나온 경우 프레임 내부를 허용
- `ALLOW-FROM domain`: 상위 문서가 지정된 도메인의 문서인 경우 프레임 내부를 허용
  - 예를 들어 트위터는 `X-Frame-Options: SAMEORIGIN`를 사용

```html
<iframe src="https://twitter.com"></iframe>
```

- 브라우저에 따라 위의 iframe은 비어있거나 해당 페이지를 이러한 방식으로 탐색하는 것을 허용하지 않는다는 경고 메시지를 표시

### Showing with disabled functionality

`X-Frame-Options`에는 부작용이 있음

- 다른 사이트에서는 합당한 이유가 있더라도 우리 페이지를 프레임에 표시할 수 없음
- 그래서 다른 해결책이 있음
- 예를 들어 페이지를 스타일 `height: 100%; width: 100%;` `<div>`를 사용해 덮어 모든 클릭을 차단할 수 있음
- `window == top`이거나 보호가 필요하지 않다고 판단되면 `<div>`를 제거해야 함

```html
<style>
  #protector {
    height: 100%;
    width: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 99999999;
  }
</style>
<div id="protector">
  <a href="/" target="_blank">Go to the site</a>
</div>
<script>
  // there will be an error if top window is from the different origin
  // but that's ok here
  if (top.document.domain == document.domain) {
    protector.remove();
  }
</script>
```

### Samesite cookie attribute

`samesite`

- `samesite` 쿠키 속성은 클릭재킹 공격도 방지 가능
- 이러한 속성을 가진 쿠키는 프레임 등을 통하지 않고 웹 사이트가 직접 열리는 경우에만 웹 사이트로 전송됨

```
Set-Cookie: authorization=secret; samesite
```

- Facebook과 같은 사이트는 인증 쿠키에 `samesite` 속성을 가짐
- 그러면 Facebook이 다른 사이트의 iframe에서 열려 있을 때 해당 쿠키가 전송되지 않음
- 그러면 공격은 실패함
- `samesite` 쿠키 속성은 쿠키가 사용되지 않으면 영향을 끼치지 않음
- 이를 통해 다른 웹 사이트에서 iframe에 인증되지 않은 공개 페이지를 쉽게 표시할 수 있음
- 그러나 이로 인해 몇 가지 제한된 경우에 클릭재킹 공격이 작동할 수 있음
- 예를 들어 IP 주소를 확인하여 중복 투표를 방지하는 익명 투표 웹 사이트
  - 쿠키를 사용해 사용자를 인증하지 않기 때문에 여전히 클릭재킹에 취약할 수 있음

## 참고

> [프레임과 윈도우](https://ko.javascript.info/frames-and-windows)
