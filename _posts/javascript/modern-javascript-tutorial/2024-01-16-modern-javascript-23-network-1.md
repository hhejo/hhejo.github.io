---
title: 모던 JavaScript 튜토리얼 23 - 네트워크 요청 1
date: 2024-01-16 20:04:41 +0900
last_modified_at: 2024-04-19 08:29:47 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    fetch,
    ajax,
    response,
    request,
    get,
    post,
    formdata,
    download-progress,
    abort
  ]
---

fetch, FormData 객체, Fetch: Download progress, Fetch: Abort

## fetch

네트워크 요청

- 자바스크립트를 사용하면 필요할 때 서버에 네트워크 요청을 보내고 새로운 정보를 받아오는 일을 할 수 있음
- 네트워크 요청은 주문 전송, 사용자 정보 읽기, 서버에서 최신 변경분 가져오기 등 같은 경우에 이뤄짐
- 그런데 이 모든 것들은 페이지 새로고침 없이도 가능

AJAX

- `Asynchronous JavaScript And XML`: 비동기적 JavaScript와 XML
- 서버에서 추가 정보를 비동기적으로 가져올 수 있게 해주는 포괄적인 기술을 나타내는 용어
- 만들어진 지 오래 됨 (AJAX에 XML이 포함된 이유)
- AJAX 이외에 서버에 네트워크 요청을 보내고 정보를 받아올 수 있는 방법은 다양함. 그 중 하나가 fetch

`fetch`

```javascript
let promise = fetch(url, [options]);
```

- 모던하고 다재다능한 메서드. 대부분의 모던 브라우저가 지원
- `url`: 접근하고자 하는 URL
- `options`: 선택 매개변수. method나 header 등을 지정 가능
  - 아무것도 넘기지 않으면 요청은 `GET` 메서드로 진행되어 `url`로부터 컨텐츠가 다운로드 됨
- `fetch()`를 호출하면 브라우저는 네트워크 요청을 보내고 프라미스가 반환됨
- 반환되는 프라미스는 `fetch()`를 호출하는 코드에서 사용됨

응답 첫 번째 단계

- 응답은 대개 두 단계를 거쳐 진행됨
- 먼저, 서버에서 응답 헤더를 받자마자 `fetch` 호출 시 반환받은 `promise`가 내장 클래스 Response의 인스턴스와 함께 이행 상태가 됨
- 이 단계는 아직 본문(body)이 도착하기 전이지만, 개발자는 응답 헤더를 보고 요청이 성공적으로 처리되었는지 아닌지를 확인할 수 있음
- 네트워크 문제나 존재하지 않는 사이트에 접속하려는 경우 같이 HTTP 요청을 보낼 수 없는 상태에서는 프라미스는 거부 상태가 됨
- HTTP 상태는 응답 프로퍼티를 사용해 확인 가능
  - `status`: HTTP 상태 코드
  - `ok`: 불린 값. HTTP 상태 코드가 200과 299 사이인 경우 `true`

응답 두 번째 단계

- 두 번째 단계에서는 추가 메서드를 호출해 응답 본문을 받음
- `response`에는 프라미스를 기반으로 하는 다양한 메서드가 있어 이 메서드들로 다양한 형태의 응답 본문 처리 가능
- 본문을 읽을 때 사용되는 메서드는 딱 하나만 사용 가능
- `response.text()`를 사용해 응답을 얻었다면 본문의 컨텐츠는 모두 처리된 상태이기 때문에 `response.json()`은 동작하지 않음
- `response.text()`: 응답을 읽고 텍스트를 반환
- `response.json()`: 응답을 JSON 형태로 파싱
- `response.formData()`: 응답을 `formData` 객체 형태로 반환
- `response.blob()`: 응답을 Blob 형태로 반환
- `response.arrayBuffer()`: 응답을 ArrayBuffer 형태로 반환
- `response.body`: ReadableStream 객체. 응답 본문을 청크 단위로 읽을 수 있음

```javascript
let url =
  "https://api.github.com/repos/javascript-tutorial/ko.javascript.info/commits";
let response = await fetch(url);
if (response.ok) {
  let commits = await response.json();
  alert(commits[0].author.login);
}

fetch(url)
  .then((response) => response.json())
  .then((commits) => alert(commits[0].author.login));

let response2 = await fetch(url);
if (response2.ok) {
  let text = await response2.text();
  alert(`${text.slice(0, 80)}...`);
}
```

```javascript
// fetch 명세서 우측 상단에 있는 로고(바이너리 데이터) 가져오기
let response = await fetch("/article/fetch/logo-fetch.svg");
let blob = response.blob(); // 응답을 Blob 객체 형태로 다운로드
let img = document.createElement("img"); // 다운로드받은 Blob을 담을 <img>
img.style = "position:fixed;top:10px;left:10px;width:100px";
document.body.append(img);
img.src = URL.createObjectURL(blob); // 이미지를 화면에 보여줌
setTimeout(() => {
  img.remove();
  URL.revokeObjectURL(img.src);
}, 3000); // 3초 후 이미지 숨김
```

### 응답 헤더

- `response.headers`에 맵과 유사한 형태로 저장되고 맵과 유사한 메서드를 지원
- 이 메서드들로 헤더 일부만 추출하거나 헤더 전체 순회 가능

```javascript
let response = await fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
);
alert(response.headers.get("Content-Type")); // application/json; charset=utf-8 (헤더 일부를 추출)
for (let [key, value] of response.headers) alert(`${key} = ${value}`); // 헤더 전체를 순회
```

### 요청 헤더

- `headers` 옵션을 사용해 `fetch`에 요청 헤더를 설정할 수 있음
- `headers`에는 아래와 같이 다양한 헤더 정보가 담긴 객체를 넘기게 됨
- `headers`를 사용해 설정할 수 없는 헤더도 있음
- 이런 제약은 HTTP를 목적에 맞고 안전하게 사용할 수 있도록 하려고 만들어짐
- 금지 목록에 있는 헤더는 브라우저만 배타적으로 설정·관리할 수 있음

```javascript
let response = fetch(protectedUrl, {
  headers: { Authentication: "secret" }
});
```

`headers`를 사용해 설정할 수 없는 헤더

- `Accept-Charset`, `Accept-Encoding`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`
- `Connection`
- `Content-Length`
- `Cookie`, `Cookie2`
- `Date`
- `DNT`
- `Expect`
- `Host`
- `Keep-Alive`
- `Origin`
- `Referer`
- `TE`
- `Trailer`
- `Transfer-Encoding`
- `Upgrade`
- `Via`
- `Proxy-*`
- `Sec-*`

### POST 요청

- `GET` 이외의 요청을 보내려면 추가 옵션을 사용해야 함
- `method`: HTTP 메서드
- `body`: 요청 본문으로 다음 항목 중 하나여야 함
  - 문자열(JSON 문자열 등)
  - `FormData` 객체: `form/multipart` 형태로 데이터를 전송하기 위해 쓰임
  - `Blob`이나 `BufferSource`: 바이너리 데이터 전송을 위해 쓰임
  - URLSearchParams: 데이터를 `x-www-form-urlencoded` 형태로 보내기 위해 쓰임(요즘에는 잘 사용하지 않음)
- 대부분은 JSON을 요청 본문에 실어 보냄
- 본문이 문자열이면, `Content-Type` 헤더가 `text/plain;charset=UTF-8`로 기본 설정됨
- `Blob`이나 `BufferSource` 객체를 사용하면 `fetch`로 바이너리 데이터(이미지 등)를 전송할 수 있음

```javascript
let user = { name: "John", surname: "Smith" };
let response = await fetch("/article/fetch/post/user", {
  method: "POST",
  headers: { "Content-Type": "application/json;charset=utf-8" }, // JSON을 전송하기 때문
  body: JSON.stringify(user) // user 객체를 본문에 실어 보냄
});
let result = await response.json();
alert(result.message); // User saved.
```

```html
<body style="margin:0">
  <canvas
    id="canvasElem"
    width="100"
    height="80"
    style="border:1px solid"
  ></canvas>
  <input type="button" value="전송" onclick="submit()" />
  <script>
    canvasElem.onmousemove = function (e) {
      let ctx = canvasElem.getContext("2d");
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };
    async function submit() {
      let blob = await new Promise((resolve) =>
        canvasElem.toBlob(resolve, "image/png")
      );
      let response = await fetch("/article/fetch/post/image", {
        method: "POST",
        body: blob // toBlob에 의해 Content-Type이 자동으로 image/png로 설정됨
      });
      let result = await response.json(); // 전송이 잘 되었다는 응답이 옴
      alert(result.message); // 이미지 사이즈 출력
    }
  </script>
</body>
```

### 예제

fetch를 사용해 GitHub에서 사용자 정보 가져오기

- GitHub 사용자 이름이 담긴 배열을 인자로 받는 비동기 함수 `getUsers(names)`
- GitHub에서 fetch한 사용자 정보들이 담긴 배열을 반환하는 함수

```javascript
async function getUsers(names) {
  let jobs = [];
  for (let name of names) {
    let job = fetch(`https://api.github.com/users/${name}`).then(
      (successResponse) => {
        if (successResponse.status != 200) return null;
        else return successResponse.json();
      },
      (failResponse) => null;
    );
    jobs.push(job);
  }
  let results = await Promise.all(jobs); // (*)
  return results;
}
```

- `await Promise.all(names.map(name => fetch(...)))`의 반환값을 대상으로 `.json()`을 호출하면 모든 fetch 응답이 완료되기 전까지 기다려야 함
- 대신 각 `fetch`마다 `.json()`을 호출하면 다른 fetch 응답을 기다리지 않으면서 JSON 형식으로 데이터를 읽을 수 있음

## FormData 객체

`FormData`

```javascript
let formData = new FormData([form]);
```

- 파일 여부나 추가 필드 여부 등과 상관 없이 통용되는 HTML 폼 전송 방법
- 폼을 쉽게 보내도록 도와주는 객체. HTML 폼 데이터를 나타냄
- HTML에 `form` 요소가 있는 경우, 위 코드를 작성하면 해당 폼 요소의 필드 전체가 자동 반영됨
- `fetch` 등의 네트워크 메서드가 `FormData` 객체를 바디로 받는다는 것은 `FormData`의 특징
- 이때 브라우저가 보내는 HTTP 메시지는 인코딩되고 `Content-Type` 속성은 `multipart/form-data`로 지정된 후 전송됨
- 서버 관점에서는 `FormData`를 사용한 방식과 일반 폼 전송 방식에 차이가 없음

### 간단한 폼 전송하기

```html
<form id="formElem">
  <input type="text" name="name" value="John" />
  <input type="text" name="surname" value="Smith" />
  <input type="submit" />
</form>
<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();
    let response = await fetch("/article/formdata/post/user", {
      method: "POST",
      body: new FormData(formElem)
    });
    let result = await response.json();
    alert(result.message); // 사용자 저장을 성공하였습니다.
  };
</script>
```

### FormData 메서드

- `FormData`에 속하는 필드는 아래와 같은 메서드로 수정 가능
- `formData.append(name, value)`: `name`과 `value`를 가진 폼 필드를 추가
- `formData.append(name, blob, fileName)`: `<input type="file">` 형태의 필드를 추가
  - `fileName`은 필드 이름이 아니고 사용자가 해당 이름을 가진 파일을 폼에 추가한 것처럼 설정해줌
- `formData.delete(name)`: `name`에 해당하는 필드를 삭제
- `formData.get(name)`: `name`에 해당하는 필드의 값을 가져옴
- `formData.has(name)`: `name`에 해당하는 필드가 있으면 `true`, 없으면 `false` 반환
- 폼은 이름(`name`)이 같은 필드 여러 개를 허용하기 때문에 `append` 메서드를 여러 번 호출해 이름이 같은 필드를 계속 추가해도 문제가 없음
- `append` 메서드 이외에 필드 추가 시 사용할 수 있는 메서드로 `set`도 있음

`set`이 `append` 메서드와 다른 점

- `set`은 `name`과 동일한 이름을 가진 필드를 제거하고 새로운 필드를 추가
- `set` 메서드를 사용하면 `name`을 가진 필드가 단 한 개만 있게끔 보장할 수 있음
- 이외의 다른 기능은 `append`와 동일
- `formData.set(name, value)`
- `formData.set(name, blob, fileName)`

폼 데이터 필드에 반복 작업을 할 때는 `for..of` 루프를 사용할 수 있음

```javascript
let formData = new FormData();
formData.append("key1", "value1");
formData.append("key2", "value2");
for (let [name, value] of formData) alert(`${name} = ${value}`);
```

### 파일이 있는 폼 전송하기

- 폼을 전송할 때 HTTP 메시지의 `Content-Type` 속성은 항상 `multipart/form-data`이고 메시지는 인코딩되어 전송됨
- 파일이 있는 폼도 당연히 이 규칙을 따르기 때문에 `<input type="file">`로 지정한 필드 역시 일반 폼을 전송할 때와 유사하게 전송됨

```html
<form id="formElem">
  <input type="text" name="firstName" value="Bora" />
  Picture: <input type="file" name="picture" accept="image/*" />
  <input type="submit" />
</form>
<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();
    let response = await fetch("/article/formdata/post/user-avatar", {
      method: "POST",
      body: new FormData(formElem)
    });
    let result = await response.json();
    alert(result.message);
  };
</script>
```

### Blob 데이터가 있는 폼 전송하기

- 이미지 같은 동적으로 생성된 바이너리 파일은 `Blob` 객체를 통해 쉽게 전송
- `Blob` 객체는 `fetch` 메서드의 `body` 매개변수에 바로 넘길 수 있음
- 그런데 실제 코딩을 하다 보면 이미지를 별도로 넘겨주는 것보다,
- 폼에 필드를 추가하고 여기에 이미지 이름 등의 메타데이터를 실어 넘겨주는 게 좀 더 편리함
- 서버 입장에서도 원시 바이너리 데이터를 받는 것보다 multipart-encoded 폼을 받는 게 좀 더 적합

```html
<body style="margin:0">
  <canvas
    id="canvasElem"
    width="100"
    height="80"
    style="border:1px solid"
  ></canvas>
  <input type="button" value="이미지 전송" onclick="submit()" />
  <script>
    canvasElem.onmousemove = function (e) {
      let ctx = canvasElem.getContext("2d");
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };
    async function submit() {
      let imageBlob = await new Promise((resolve) =>
        canvasElem.toBlob(resolve, "image/png")
      );
      let formData = new FormData();
      formData.append("firstName", "Bora");
      formData.append("image", imageBlob, "image.png"); // (*)
      let response = await fetch("/article/formdata/post/image-form", {
        method: "POST",
        body: formData
      });
      let result = await response.json();
      alert(result.message);
    }
  </script>
</body>
```

- `(*)`: 폼에 `<input type="file" name="image">` 태그가 있고,
- 사용자 기기의 파일 시스템에서 파일명이 `"image.png"`인 `imageBlob` 데이터를 추가한 것과 동일한 효과
- 요청을 받은 서버는 일반 폼과 동일하게 폼 데이터와 파일을 읽고 처리

## Fetch: Download progress

`fetch` 메서드

- 다운로드 진행 상황을 추적할 수 있게 해줌
- 현재 `fetch`에게 업로드 진행 상황을 추적할 수 있는 방법은 없음. 이를 위해서는 `XMLHttpRequest`를 사용

`response.body`

- 다운로드 진행 상황을 추적하기 위해 `response.body` 프로퍼티를 사용할 수 있음
- `response.body`는 `ReadableStream`
- body를 chunk 단위로 제공하는 특별한 객체
- `response.text()`, `response.json()` 등 기타 메서드들과 다르게,
- `response.body`는 읽기 과정을 완전히 통제하고, 우리는 어느 순간에 얼마나 소비되는지를 셀 수 있음

```javascript
const reader = response.body.getReader();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log(`Received ${value.length} bytes`);
}
```

- `await reader.read()` 호출의 결과는 두 가지 프로퍼티를 가진 객체
  - `done`: 읽기가 완료되면 `true`, 그렇지 않으면 `false`
  - `value`: 형식화된 바이트 배열(typed array of bytes) `Uint8Array`
- Streams API는 또한 `for await..of` 루프를 사용한 `ReadableStream`을 통한 비동기 반복을 설명
- 하지만 아직 널리 지원되지 않기 때문에 `while` 루프를 사용
- 우리는 로딩이 완료될 때까지 루프에서 응답 청크(chunk)를 받음 (`done`이 `true`가 될 때까지)
- 진행 상황을 기록하려면 받은 모든 조각(fragment) `value`의 길이를 counter에 추가하기만 하면 됨

응답을 받고 콘솔에 진행 상황을 기록하는 작업

```javascript
// 1단계 : fetch를 시작하고 reader 얻기
let response = await fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits?per_page=100"
);
const reader = response.body.getReader();

// 2단계: 전체 길이 얻기
const contentLength = +response.headers.get("Content-Length");

// 3단계: 데이터 읽기
let receivedLength = 0; // received that many bytes at the moment
let chunks = []; // array of received binary chunks (comprises the body)
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  chunks.push(value);
  receivedLength += value.length;
  console.log(`Received ${receivedLength} of ${contentLength}`);
}

// 4단계: 청크를 단일 Uint8Array로 연결
let chunksAll = new Uint8Array(receivedLength); // (4.1)
let position = 0;
for (let chunk of chunks) {
  chunksAll.set(chunk, position); // (4.2)
  position += chunk.length;
}

// 5단계: 문자열로 디코딩
let result = new TextDecoder("utf-8").decode(chunksAll);

// 끝
let commits = JSON.parse(result);
alert(commits[0].author.login);
```

1. 평소대로 `fetch`를 수행하지만 `response.json()`을 호출하는 대신 스트림 리더 `response.body.getReader()`를 얻음
2. 읽기 전에 `Content-Length` 헤더에서 전체 응답 길이를 파악할 수 있음
   - 교차 출처 요청(cross-origin request)에는 없을 수 있으며, 기술적으로 서버는 이를 설정할 필요가 없음. 하지만 대개 제자리에 있음
3. 완료될 때까지 `await reader.read()` 호출
   - 우리는 `chunks` 배열에 response chunks를 수집
   - 응답이 소비된 후에는 `response.json()`이나 다른 방법을 사용하여 '다시 읽을'(re-read) 수 없기 때문에 중요
4. 마지막에는 `Uint8Array` 바이트 chunks의 배열 `chunks`를 가짐
   - 하나의 결과로 결합해야 하는데 이들을 연결하는 메서드가 없으므로 이를 수행하는 몇 가지 코드가 있음
   - 결합된 길이를 가진 동일한 유형의 배열 `chunksAll = new Uint8Array(receivedLength)`를 생성
   - 그러고 나서 `.set(chunk, position)` 메서드를 사용해 `chunk`를 하나씩 복사
5. `chunksAll`에 결과를 얻음. 문자열이 아닌 바이트 배열
   - 문자열을 생성하려면 이러한 바이트를 해석해야 함

```javascript
let blob = new Blob(chunks); // 문자열 대신 바이너리 컨텐츠가 필요한 경우
```

## Fetch: Abort

진행중인 `fetch`를 취소할 수 있는 방법

- `fetch`는 프라미스를 반환
- 자바스크립트에는 일반적으로 프라미스를 중단(abort)한다는 개념이 없음
- 어떻게 진행 중인 `fetch`를 취소할 수 있을까
- 예를 들어 우리 사이트의 사용자가 `fetch`가 더 이상 필요하지 않다고 나타내는 경우

`AbortController`

- 이러한 목적을 위한 특별 내장 객체
- `fetch`뿐만 아니라 다른 비동기 작업도 중단하는 데 사용

### The AbortController object

```javascript
let controller = new AbortController();
```

- 컨트롤러는 매우 간단한 객체
- 단일 메서드 `abort()`를 가짐
- 이벤트 리스너를 설정할 수 있는 단일 프로퍼티 `signal`을 가짐

`abort()` 호출 시

- `controller.signal`은 `"abort"` 이벤트를 내보냄
- `controller.signal.aborted` 프로퍼티가 `true`가 됨
- 일반적으로 이 과정에는 두 당사자(parties)가 있음
  - 취소 가능한 작업을 수행하는 하나. `controller.signal`에 리스너를 설정
  - 취소하는 하나. 필요하면 `controller.abort()`를 호출

```javascript
let controller = new AbortController();
let signal = controller.signal;

// 취소 가능한 작업을 수행하는 당사자
// signal 객체를 받음
// controller.abort()가 호출되면 트리거되는 리스너를 설정
signal.addEventListener("abort", () => alert("abort!"));

// 취소하는(나중에 언제든지) 다른 당사자
controller.abort(); // abort!

// 이벤트가 트리거되고 signal.aborted가 true가 됨
alert(signal.aborted); // true
```

- `AbortController`는 `abort()`가 호출될 때 `abort` 이벤트를 전달하는 수단일 뿐
- `AbortController` 객체 없이도 동일한 종류의 이벤트 리스닝을 자체적으로 구현할 수 있음
- 하지만 중요한 것은 `fetch`가 `AbortController` 객체를 가지고 작업하는 방법을 알고 그것이 대상과 통합되어 있다는 것

### Using with fetch

- `fetch`를 취소하려면 `AbortController`의 `signal` 프로퍼티를 `fetch`의 옵션으로 전달

```javascript
let controller = new AbortController();
fetch(url, { signal: controller.signal });
```

- `fetch` 메서드는 `AbortController`와 작업하는 방법을 알고 있음
- `signal`에서 `abort` 이벤트를 수신
- 중단하려면 `controller.abort()`를 호출
- `fetch`는 `signal`에서 이벤트를 가져오고 요청을 중단
- fetch가 중단되면 `AbortError` 에러와 함께 거부되므로 처리해야 함

```javascript
// 1초 후 중단
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);
try {
  let response = await fetch("/article/fetch-abort/demo/hang", {
    signal: controller.signal
  });
} catch (err) {
  if (err.name == "AbortError") alert("Aborted!");
  else throw err;
}
```

### AbortController is scalable

- `AbortController`는 확장 가능하므로 한번에 여러 fetch를 취소할 수 있음
- 아래 코드는 병렬로 많은 `urls`를 fetch하고 단일 컨트롤러로 모두 중단하는 예

```javascript
let urls = [...]; // a list of urls to fetch in parallel
let controller = new AbortController();
let fetchJobs = urls.map(url => fetch(url, { signal: controller.signal })); // fetch 프라미스의 배열
let results = await Promise.all(fetchJobs);
// controller.abort()가 어디에서든 호출되면 모든 fetch를 중단함
```

- `fetch`와 다른 자체 비동기 작업이 있는 경우, 이것들을 단일 `AbortController`로 중단하는 데 사용할 수 있음
- 우리 작업에서 해당 `abort` 이벤트를 듣기만 하면 됨

```javascript
let urls = [...];
let controller = new AbortController();

let ourJob = new Promise((resolve, reject) => { // our task
  ...
  controller.signal.addEventListener('abort', reject);
});

let fetchJobs = urls.map(url => fetch(url, { // fetches
  signal: controller.signal
}));

// Wait for fetches and our task in parallel
let results = await Promise.all([...fetchJobs, ourJob]);

// if controller.abort() is called from elsewhere,
// it aborts all fetches and ourJob
```

## 참고

> [네트워크 요청](https://ko.javascript.info/network)
