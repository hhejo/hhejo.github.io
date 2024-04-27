---
title: 모던 JavaScript 튜토리얼 24 - 브라우저에 데이터 저장하기 1
date: 2024-01-30 11:37:09 +0900
last_modified_at: 2024-04-27 13:48:09 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, cookie, localstorage, sessionstorage]
---

쿠키와 document.cookie, localStorage와 sessionStorage

## 쿠키와 document.cookie

쿠키

- 브라우저에 저장되는 작은 크기의 문자열
- RFC 6265 명세에서 정의한 HTTP 프로토콜의 일부

쿠키는 주로 웹 서버에 의해 만들어짐

1. 서버가 HTTP 응답 헤더의 `Set-Cookie`에 내용을 넣어 전달
2. 브라우저는 이 내용을 자체적으로 브라우저에 저장. 이것이 쿠키
3. 브라우저는 사용자가 쿠키를 생성하도록 한 동일 서버(사이트)에 접속할 때마다 쿠키의 내용을 `Cookie` 요청 헤더에 넣어 함께 전달

쿠키는 클라이언트 식별과 같은 인증에 가장 많이 쓰임

1. 사용자가 로그인하면 서버는 HTTP 응답 헤더의 `Set-Cookie`에 담긴 세션 식별자(session identifier) 정보를 사용해 쿠키를 설정
2. 사용자가 동일 도메인에 접속하려고 하면 브라우저는 HTTP `Cookie` 헤더에 인증 정보가 담긴 고윳값(세션 식별자)을 함께 실어 서버에 요청을 보냄
3. 서버는 브라우저가 보낸 요청 헤더의 세션 식별자를 읽어 사용자를 식별

`document.cookie` 프로퍼티

- 브라우저에서도 쿠키에 접근

### 쿠키 읽기

`document.cookie`

- `name=value` 쌍으로 구성
- 각 쌍은 `;`로 구분
- 쌍 하나는 하나의 독립된 쿠키
- `;`을 기준으로 `document.cookie`의 값을 분리해 원하는 쿠키를 찾음

```javascript
document.cookie.split("; ");
```

```
0: "pixelRatio=2"
1: "_ga_2LWB61WGYJ=GS1.1.1714193773.1.0.1714193773.0.0.0"
2: "_ga=GA1.1.1583472010.1714193774"
3: "_ym_uid=1714193775255873531"
4: "_ym_d=1714193775"
5: "_ym_isad=1"
6: "_ym_visorc=w"
```

### 쿠키 쓰기

- `document.cookie`에 직접 값 작성 가능

`cookie`

- 데이터 프로퍼티가 아닌 접근자 프로퍼티
- `document.cookie`에 값을 할당하면 브라우저는 이 값을 받아 해당 쿠키를 갱신
- 이때, 다른 쿠키의 값은 변경되지 않음
- 즉, `document.cookie=` 연산은 모든 쿠키를 덮어쓰지 않고 명시된 쿠키의 값만 갱신

`encodeURIComponent`

- 쿠키의 이름과 값에는 특별한 제약이 없어 모든 글자가 허용됨
- 하지만 형식의 유효성을 일관성 있게 유지하기 위해 반드시 내장 함수 `encodeURIComponent`를 사용해 이름과 값을 이스케이프 처리

```javascript
document.cookie = "user=John"; // 이름이 user인 쿠키의 값만 갱신
alert(document.cookie); // user=John; my%20name=John%20Smith; pixelRatio=2 (모든 쿠키가 보임)

let name = "my name"; // 공백 포함
let value = "John Smith"; // 공백 포함
document.cookie = encodeURIComponent(name) + "=" + encodeURIComponent(value);
alert(document.cookie); // ...; my%20name=John%20Smith (공백이 인코딩 처리됨)
```

쿠키의 한계

- 쿠키에는 몇 가지 제약 사항이 있음
- `encodeURIComponent`로 인코딩한 이후의 `name=value` 쌍은 `4KB`를 넘을 수 없음
- `4KB` 용량을 넘는 정보는 쿠키에 저장 불가
- 도메인 하나당 저장할 수 있는 쿠키의 개수는 20여 개 정도로 한정됨
- 개수는 브라우저에 따라 조금씩 다름

쿠키의 옵션

- 쿠키에는 몇 가지 옵션이 있음
- 몇몇 옵션은 아주 중요하기 때문에 꼭 지정해야 함
- 옵션은 `key=value` 뒤에 나열하고 `;`으로 구분

```javascript
document.cookie = "user=John; path=/; expires=Tue, 19 Jan 2038 03:14:07 GMT";
```

### path

`path=/mypath`

- URL path(경로)의 접두사로, 이 경로의 하위 경로에 있는 페이지만 쿠키에 접근 가능
- 절대 경로여야 하고 (미지정 시) 기본값은 현재 경로
- 특별한 경우가 아니라면 `path` 옵션을 `path=/` 같이 루트로 설정해 웹 사이트의 모든 페이지에서 쿠키에 접근할 수 있도록 함

예: `path=/admin` 옵션으로 설정한 쿠키

- `/admin`과 `/admin/something`에서는 볼 수 있지만 `/home`이나 `adminpage`에서는 볼 수 없음

### domain

`domain=site.com`

- 쿠키에 접근 가능한 domain(도메인)을 지정
- 몇 가지 제약이 있어서 아무 도메인이나 지정할 수 없음
- `domain` 옵션에 아무 값도 넣지 않았다면 쿠키를 설정한 도메인에서만 접근 가능

예: `site.com`에서 설정한 쿠키

- `other.com`에서 얻을 수 없음
- `site.com`에서 생성한 쿠키를 `other.com`에서 절대 전송받을 수 없음

서브 도메인(subdomain)

- 서브 도메인인 `forum.site.com`에서도 쿠키 정보를 얻을 수 없다는 제약 사항이 하나 더 있음
- 서브 도메인이나 다른 도메인에서 쿠키에 접속할 방법은 없음
- 이런 제약 사항은 안정성을 높이기 위해 만들어짐
- 민감한 데이터가 저장된 쿠키는 관련 페이지에서만 볼 수 있도록 하기 위함

사실 서브 도메인에서 쿠키 정보를 얻는 방법이 있음

- `site.com`에서 쿠키를 설정할 때 `domain` 옵션에 루트 도메인인 `domain=site.com`을 명시적으로 설정하면 됨
- 하위 호환성 유지를 위해 `domain=.site.com`도 `domain=site.com`과 동일하게 작동

```javascript
document.cookie = "user=John"; // site.com에서 쿠키를 설정
alert(document.cookie); // 찾을 수 없음 (site.com의 서브도메인 forum.site.com에서 user 쿠키에 접근하려 함)

document.cookie = "user=John; domain=site.com"; // site.com에서 서브 도메인(*.site.com) 어디서든 쿠키에 접속하게 설정할 수 있음
alert(document.cookie); // user=John 쿠키 확인 가능
```

### expires와 max-age

- `expires`(유효 일자)나 `max-age`(만료 기간) 옵션이 지정되어 있지 않으면 브라우저가 닫힐 때 쿠키도 함께 삭제됨
- 이런 쿠키를 세션 쿠키(session cookie)라고 부름
- `expires`나 `max-age` 옵션을 설정하면 브라우저를 닫아도 쿠키가 삭제되지 않음

`expires=Tue, 19 Jan 2038 03:14:07 GMT`

- 브라우저는 설정된 유효 일자까지 쿠키를 유지하다가 해당 일자가 도달하면 쿠키를 자동으로 삭제
- 쿠키의 유효 일자는 반드시 GMT 포맷으로 설정
- `date.toUTCString`을 사용하면 해당 포맷으로 쉽게 변경할 수 있음
- `expires` 옵션을 과거로 지정하면 쿠키는 삭제됨

```javascript
let date = new Date(Date.now() + 86400e3);
date = date.toUTCString();
document.cookie = "user=John; expires=" + date; // 유효기간이 하루인 쿠키 생성
```

`max-age=3600`

- `max-age`는 `expires` 옵션의 대안으로, 쿠키 만료 기간을 설정할 수 있게 해줌
- 현재부터 설정하고자 하는 만료일시까지의 시간을 초로 환산한 값을 설정
- 0이나 음수값을 설정하면 쿠키는 바로 삭제됨

```javascript
document.cookie = "user=John; max-age=3600"; // 1시간 뒤에 쿠키가 삭제됨
document.cookie = "user=John; max-age=0"; // 만료 기간을 0으로 지정하여 쿠키를 바로 삭제함
```

### secure

`secure`

- `secure` 옵션을 설정하면 HTTPS로 통신하는 경우에만 쿠키가 전송됨
- `secure` 옵션이 없으면 기본 설정이 적용됨
- `http://site.com`에서 설정(생성)한 쿠키를 `https://site.com`에서 읽을 수 있고,
- `https://site.com`에서 설정(생성)한 쿠키도 `http://site.com`에서 읽을 수 있음
- 쿠키는 기본적으로 도메인만 확인하지 프로토콜을 따지지 않기 때문
- 하지만 `secure` 옵션이 설정된 경우, `https://site.com`에서 설정한 쿠키는 `http://site.com`에서 접근할 수 없음
- 쿠키에 민감한 내용이 저장되어 있어 암호화되지 않은 HTTP 연결을 통해 전송되는 것을 원치 않는다면 이 옵션을 사용하면 됨

```javascript
document.cookie = "user=John; secure"; // https://로 통신하고 있다고 가정. 설정한 쿠키는 HTTPS 통신 시에만 접근 가능
```

### samesite

`samesite`

- 또 다른 보안 속성인 `samesite` 옵션
- 크로스 사이트 요청 위조(cross-site request forgery, XSRF) 공격을 막기 위해 만들어진 옵션

XSRF 공격 시나리오

- 현재 `bank.com`에 로그인됐다고 가정
- 해당 사이트에서 사용되는 인증 쿠키가 브라우저에 저장됨
- 브라우저는 `bank.com`에 요청을 보낼 때마다 인증 쿠키를 함께 전송할 것
- 서버는 전송받은 쿠키를 이용해 사용자를 식별하고 보안이 필요한 재정 거래를 처리

이제 (로그아웃하지 않고) 다른 창을 띄워서 웹 서핑을 하던 도중에 뜻하지 않게 `evil.com`에 접속했다고 가정

- 이 사이트에는 해커에게 송금을 요청하는 폼 `<form action="https://bank.com/pay">`이 있음
- 이 폼은 자동으로 제출되도록 설정됨
- 폼이 `evil.com`에서 은행 사이트로 바로 전송될 때 인증 쿠키도 함께 전송됨
- `bank.com`에 요청을 보낼 때마다 `bank.com`에서 설정한 쿠키가 전송되기 때문
- 은행은 전송받은 쿠키를 읽어 (해커가 아닌) 계정 주인이 접속한 것이라 생각하고 해커에게 돈을 송금
- 이런 공격을 크로스 사이트 요청 위조라고 부름

```
              evil.com                                          bank.com
<form action="https://bank.com/pay">   -- POST /pay -->   got the cookie? okay!
  ...                                  cookie: user=John
</form>
```

실제 은행은 당연히 이 공격을 막을 수 있도록 시스템을 설계

- `bank.com`에서 사용되는 모든 폼에 XSRF 보호 토큰(protection token)이라는 특수 필드를 넣음
- 이 토큰은 악의적인 페이지에서 만들 수 없고, 원격 페이지에서도 훔쳐올 수 없도록 구현됨
- 따라서 악의적인 페이지에서 폼을 전송하더라도 보호 토큰이 없거나 서버에 저장된 값과 일치하지 않기 때문에 요청이 무용지물 됨
- 하지만 이런 절차는 구현에 시간이 걸림
- 모든 폼에 보호 토큰을 세팅해야 하고 요청 전부를 검수해야 함

`samesite` 옵션

- 쿠키의 `samesite` 옵션을 이용하면 XSRF 보호 토큰 없이도 (이론상으로) 크로스 사이트 요청 위조를 막을 수 있음
- 두 가지 값 설정 가능

`samesite=strict`

- 값을 설정하지 않고 그냥 `samesite` 옵션만 써줘도 동일하게 동작
- 사용자가 사이트 외부에서 요청을 보낼 때, `samesite=strict` 옵션이 있는 쿠키는 절대 전송되지 않음
- 메일에 있는 링크를 따라 접속하거나 `evil.com` 같은 사이트에서 폼을 전송하는 경우 등과 같이 제3의 도메인에서 요청이 이뤄질 땐 쿠키가 전송되지 않음
- 인증 쿠키에 `samesite` 옵션이 있는 경우, XSRF 공격은 절대로 성공하지 못함
- `evil.com`에서 전송하는 요청에는 쿠키가 없을 것이고, `bank.com`은 미인식 사용자에게 지급을 허용하지 않을 것
- `bank.com`에서 수행하는 모든 작업은 `samesite` 쿠키를 함께 전송하기 때문에 꽤 믿을 만한 보호장치
- 하지만 약간의 불편함도 감수해야 함
- 만약 사용자가 메모장 등에 `bank.com`에 요청을 보낼 수 있는 링크를 기록해 놓았다가 이 링크를 클릭해 접속하면 `bank.com`이 사용자를 인식하지 못하는 상황이 발생하기 때문
- 실제로 이런 경우 `samesite=strict` 옵션이 설정된 쿠키는 전송되지 않음
- 이런 문제는 쿠키 두 개를 함께 사용해 해결 가능
- "Hello, John"과 같은 환영 메시지를 출력해주는 "일반 인증(general recognition)"용 쿠키, 데이터 교환 시 사용하는 `samesite=strict` 옵션이 있는 쿠키를 따로 둠
- 이렇게 하면 외부 사이트를 통해 접근한 사용자도 정상적으로 환영 메시지를 볼 수 있음
- 지급은 무조건 은행의 사이트를 통해서만 수행되도록 만들면 됨

`samesite=lax`

- 사용자 경험을 해치지 않으면서 XSRF 공격을 막을 수 있는 느슨한 접근법
- `strict`와 마찬가지로 `lax`도 사이트 외부에서 요청을 보낼 때 브라우저가 쿠키를 보내는 걸 막아줌
- 하지만 예외사항 존재

아래 두 조건을 동시에 만족할 때는 `samesite=lax` 옵션을 설정한 쿠키가 전송됨

1. "안전한" HTTP 메서드인 경우 (예: GET 방식. POST 방식은 해당하지 않음)
   - 안전한 메서드는 읽기 작업만 수행하고 쓰기나 데이터 교환 작업은 수행하지 않음
   - 링크를 따라가는 행위는 항상 GET 방식이기 때문에 안전한 메서드만 쓰임
2. 작업이 최상위 레벨 탐색에서 이루어질 때(브라우저 주소창에서 URL을 변경하는 경우)
   - 대다수의 작업은 이 조건을 충족
   - 하지만 `<iframe>` 안에서 탐색이 일어나는 경우는 최상위 레벨 탐색이 아니기 때문에 이 조건을 충족하지 못함
   - AJAX 요청 또한 탐색 행위가 아니므로 이 조건을 충족하지 못함

예

- 브라우저를 이용해 자주 하는 작업인 "특정 URL로 이동하기"를 실행하는 경우, `samesite=lax` 옵션이 설정되어 있으면 쿠키가 서버로 전송됨
- 노트에 저장된 링크를 여는 것도 특정 URL로 이동하는 행위이므로 위 조건들을 충족함
- 하지만 외부 사이트에서 AJAX 요청을 보내거나 폼을 전송하는 등의 복잡한 작업을 시도할 때는 쿠키가 전송되지 않음

이런 제약사항이 있어도 괜찮다면 `samesite=lax` 옵션은 사용자 경험을 해치지 않으면서 보안을 강화해주는 방법으로 활용할 수 있음

문제점

- 오래된 브라우저(2017년 이전 버전)에서는 `samesite` 옵션을 지원하지 않음
- 따라서 `samesite` 옵션으로만 보안 처리를 하게 되면 구식 브라우저에서 보안 문제가 발생할 수 있음

### httpOnly

`httpOnly`

- 웹 서버에서 `Set-Cookie` 헤더를 이용해 쿠키를 설정할 때 지정할 수 있음
- 자바스크립트 같은 클라이언트 측 스크립트가 쿠키를 사용할 수 없게 함
- `document.cookie`를 통해 쿠키를 볼 수도 없고 조작할 수도 없음
- 해커가 악의적인 자바스크립트 코드를 페이지에 삽입하고, 사용자가 그 페이지에 접속하기를 기다리는 방식의 공격을 예방할 때 이 옵션을 사용
- 우리가 만든 사이트에 해커가 악의적인 코드를 삽입하지 못하도록 예방해야 하지만, 버그가 있을 확률은 언제나 있기 때문에 해커가 코드를 삽입할 가능성이 있을 수 있음
- 이런 상황이 만에 하나 발생하면 사용자가 웹 페이지에 방문할 때 `document.cookie`를 볼 수도 있고 조작도 할 수 있는 해커의 코드도 함께 실행됨
- 물론 쿠키에는 인증 정보가 있어서 해커가 이 정보를 훔치거나 조작할 수 있음
- 하지만 `httpOnly` 옵션이 설정된 쿠키는 `document.cookie`로 쿠키 정보를 읽을 수 없기 때문에 쿠키를 보호할 수 있음

### 부록: 쿠키 함수

쿠키를 다룰 때 유용하게 사용할 수 있는 몇 가지 함수들

- `document.cookie`를 수동으로 조작하지 않고 이 함수들을 활용하면 좀 더 편리하게 쿠키를 다룰 수 있음
- 쿠키를 갱신하거나 삭제할 때는 쿠키를 설정할 때 지정했던 도메인이나 경로를 사용해야 함에 주의

getCookie(name)

```javascript
// 주어진 이름의 쿠키를 반환하는데,
// 조건에 맞는 쿠키가 없다면 undefined를 반환
function getCookie(name) {
  let matches = document.cookie.match(
    new RegExp(
      "(?:^|; )" +
        name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, "\\$1") +
        "=([^;]*)"
    )
  );
  return matches ? decodeURIComponent(matches[1]) : undefined;
}
```

setCookie(name, value, options)

```javascript
function setCookie(name, value, options = {}) {
  options = {
    path: "/",
    // 필요한 경우, 옵션 기본값을 설정할 수도 있음
    ...options
  };

  if (options.expires instanceof Date) {
    options.expires = options.expires.toUTCString();
  }

  let updatedCookie =
    encodeURIComponent(name) + "=" + encodeURIComponent(value);

  for (let optionKey in options) {
    updatedCookie += "; " + optionKey;
    let optionValue = options[optionKey];
    if (optionValue !== true) {
      updatedCookie += "=" + optionValue;
    }
  }

  document.cookie = updatedCookie;
}

// Example of use:
setCookie("user", "John", { secure: true, "max-age": 3600 });
```

deleteCookie(name)

```javascript
function deleteCookie(name) {
  setCookie(name, "", {
    "max-age": -1
  });
}
```

### 부록: 서드 파티 쿠키

서드 파티 쿠키(third-party cookie)

- 사용자가 방문 중인 도메인이 아닌 다른 도메인에서 설정한 쿠키

예

1. `site.com`의 특정 페이지에서 이미지 배너(banner)를 불러옴. 배너는 다른 도메인 `<img src="https://ads.com/banner.png">`에서 가져옴
2. `ads.com`에 있는 원격 서버는 배너와 함께 `Set-Cookie` 헤더를 전송해 브라우저가 `id=1234`와 같은 쿠키를 설정하도록 함. 이 쿠키는 `ads.com` 도메인에서 설정한 것이기 때문에 `ads.com`에서만 볼 수 있음
3. 사용자가 `ads.com`에 다시 접속하면, 원격 서버는 요청과 함께 전송받은 쿠키의 `id`를 이용해 해당 유저를 인식
4. 사용자가 `site.com`을 떠나 `other.com`에 접속하고 이 사이트에도 배너가 있으면 `ads.com`은 또 쿠키를 전송받음. 이 쿠키는 `ads.com`에서 설정한 것이기 때문. 이를 이용해 `ads.com`은 사용자를 인식하고, 이 사용자가 어떤 사이트로 이동했는지를 추적

광고회사는 사용자의 이용 행태를 추적하고, 광고를 제공하기 위해 오래전부터 서드 파티 쿠키를 사용

- 서드 파티 쿠키는 쿠키를 설정한 도메인에 종속되기 때문에 `ads.com`은 사용자가 어떤 사이트를 방문했는지 추적 가능
- 그런데 사람들은 누군가 자신을 감시하는 걸 좋아하진 않음
- 브라우저에는 이런 쿠키를 비활성화할 수 있는 기능이 있어 이 기능을 사용하면 추적을 막을 수 있음

여기에 더하여 몇몇 모던 브라우저는 서드 파티 쿠키를 위한 특별한 정책을 도입하여 광고회사의 추적을 막을 수 있게 함

- Safari는 서드파티 쿠키를 전면적으로 허용하지 않음
- Firefox는 서드 파티 도메인 블랙 리스트를 만들어 리스트에 오른 도메인의 서드 파티 쿠키를 차단

주의

- `<script src="https://google-analytics.com/analytics.js">` 같은 태그로 서드 파티 도메인에서 스크립트를 읽어오고, 이 스크립트 안에 `document.cookie`로 쿠키를 설정하는 코드가 있다면, 이때 만들어진 쿠키는 서드파티 쿠키가 아님
- 스크립트에서 쿠키를 설정한 경우에 만들어지는 쿠키는 현재 페이지의 도메인에 속하게 됨
- 스크립트의 유래와 상관 없이

### 부록: GDPR

GDPR

- 사용자 개인 정보 보호를 강제하는 EU의 법령
- 쿠키를 추적하는 경우 사용자로부터 명시적인 허가를 얻어야 한다는 것이 이 법령의 중요 요건 중 하나
- 쿠키를 이용한 사용자 추적, 식별에 관한 내용을 담음
- 따라서 쿠키를 설정하고, 이 쿠키를 정보 저장의 용도로만 사용한다면 이 법령이 강제하는 사항을 지킬 필요가 없음
- 사용자를 추적하거나 식별하지 않는다면
- 하지만 인증 세션과 함께 쿠키를 설정하거나 id를 추적한다면 사용자의 동의를 반드시 얻어야 함

웹 사이트는 다음과 같은 방법으로 GDPR에 대응할 수 있음

1. 인증된 사용자에 대해서만 추적 쿠키를 설정하려는 경우
   - 가입 양식에 "개인 정보 취급 방침 동의" 같은 확인란을 만들고, 사용자가 이에 동의할 경우에만 추적 쿠키를 설정
2. 모든 사용자를 대상으로 추적 쿠키를 설정하려는 경우
   - 최초 방문자에게 쿠키 설정에 대한 동의를 요구하는 "작은 창"을 보여주고, 사용자가 이에 동의한 경우에만 콘텐츠를 표시하고, 추적 쿠키를 설정
   - 새로운 방문자는 이런 절차가 번거롭다고 생각할 수 있음
   - 콘텐츠를 가리면서 "무조건 클릭해야 하는 창"을 그 누구도 달가워하지 않으나 GDPR을 준수하려면 이 창이 반드시 있어야 함

## 참고

> [브라우저에 데이터 저장하기](https://ko.javascript.info/data-storage)
