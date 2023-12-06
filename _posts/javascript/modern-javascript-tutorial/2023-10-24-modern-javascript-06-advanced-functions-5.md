---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 5
date: 2023-10-24 12:06:46 +0900
last_modified_at: 2023-12-06 20:31:01 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

call/apply와 데코레이터, 포워딩, 함수 바인딩, 화살표 함수 다시 살펴보기

## call/apply와 데코레이터, 포워딩

자바스크립트는 함수를 다룰 때 탁월한 유연성을 제공

- 이곳저곳 전달될 수 있음
- 객체로도 사용될 수 있음

함수 간에 호출을 어떻게 포워딩(forwarding) 하는지, 데코레이팅(decorating) 하는지 알아보기

### 코드 변경 없이 캐싱 기능 추가하기

CPU를 많이 잡아먹지만 결과는 안정적인 함수 `slow(x)`

- 결과가 안정적: `x`가 같으면 호출 결과도 같음
- `slow(x)`가 자주 호출되는 경우
  - 결과를 어딘가에 저장(캐싱)해 재연산에 걸리는 시간을 줄이는 것이 좋음
- 래퍼 함수를 만들어 캐싱 기능을 추가
  - `slow()` 안에 캐싱 관련 코드를 추가하지 않음

```javascript
function slow(x) {
  alert(`slow(${x})을/를 호출함`);
  return x;
}

function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) return cache.get(x);
    let result = func(x);
    cache.set(x, result);
    return result;
  };
}

slow = cachingDecorator(slow);
alert(slow(1)); // 캐시에 slow(1) 저장됨
alert("다시 호출: " + slow(1));
alert(slow(2)); // 캐시에 slow(2) 저장됨
alert("다시 호출: " + slow(2));
```

데코레이터(decorator)

- 인수로 받은 함수의 행동을 변경시켜주는 함수
- 함수에 추가된 기능 혹은 상(aspect)

`cachingDecorator(func)`

- 모든 함수를 대상으로 호출 가능
- 캐싱 래퍼(wrapper) `function(x)` 반환
- 함수에 `cachingDecorator`를 적용하기만 하면 캐싱이 가능한 함수를 원하는 만큼 구현 가능
  - 재사용성 증가
- 캐싱 관련 코드를 함수 코드와 분리
  - 함수의 코드가 간결해져 원 함수 자체의 복잡성이 증가하지 않음
- 바깥 코드에서 봤을 때 함수 `slow`는 래퍼로 감싼 이전이나 이후나 동일한 일을 수행
  - 행동 양식에 캐싱 기능이 추가된 것
- 여러 개의 데코레이터를 조합해서 사용할 수도 있음
  - 추가 데코레이터는 `cachingDecorator` 뒤를 따름

### func.call을 사용해 컨텍스트 지정하기

위에서 구현한 캐싱 데코레이터는 객체 메서드에 사용하기에는 적합하지 않음

- 객체 메서드 `worker.slow()`는 데코레이터 적용 후 제대로 동작하지 않음

```javascript
let worker = {
  someMethod() {
    return 1;
  },
  slow(x) {
    alert(`slow(${x})을/를 호출`);
    return x * this.someMethod(); // (*)
  }
};

function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) return cache.get(x);
    let result = func(x); // (**)
    cache.set(x, result);
    return result;
  };
}

alert(worker.slow(1)); // slow(1)을/를 호출, 1
worker.slow = cachingDecorator(worker.slow);
alert(worker.slow(2)); // TypeError: Cannot read properties of undefined (reading 'someMethod')

let func = worker.slow;
func(2); // 비슷한 증상 발생
```

- `(*)` 줄에서 `this.someMethod` 접근에 실패했기 때문에 에러 발생
- `(**)` 줄에서 래퍼가 기존 함수 `func(x)`를 호출하면 `this`가 `undefined`가 되기 때문
- 래퍼가 기존 메서드 호출 결과를 전달하려 했지만 `this`의 컨텍스트가 사라졌기 때문에 에러 발생

`func.call(context, ...args)`

- `this`를 명시적으로 고정해 함수를 호출할 수 있게 해주는 특별한 내장 함수 메서드
- 첫 번째 인수가 `this`
- 이어지는 인수가 `func`의 인수가 됨

```javascript
func.call(context, arg1, arg2, ...)
```

```javascript
func(1, 2, 3);
func.call(obj, 1, 2, 3); // this가 obj로 고정되는 것이 위 코드와의 차이
```

```javascript
function sayHi() {
  alert(this.name);
}
let user = { name: "John" };
let admin = { name: "Admin" };
sayHi.call(user); // John (sayHi의 컨텍스트가 this = user)
sayHi.call(admin); // Admin (sayHi의 컨텍스트가 this = admin)
```

```javascript
function say(phrase) {
  alert(this.name + ": " + phrase);
}
let user = { name: "John" };
say.call(user, "Hello"); // John: Hello. call을 사용해 컨텍스트와 인수에 원하는 값 지정
```

```javascript
let worker = {
  someMethod() {
    return 1;
  },
  slow(x) {
    alert(`slow(${x})을/를 호출`);
    return x * this.someMethod();
  }
};

function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) return cache.get(x);
    let result = func.call(this, x);
    cache.set(x, result);
    return result;
  };
}

worker.slow = cachingDecorator(worker.slow);
alert(worker.slow(2)); // slow(2)을/를 호출, 2
alert(worker.slow(2)); // 2
```

1. 데코레이터를 적용한 후, `worker.slow`는 래퍼 `function (x) { ... }`가 됨
2. `worker.slow(2)`를 실행하면 래퍼는 `2`를 인수로 받고, `this=worker`가 됨(점 앞의 객체)
3. 결과가 캐시되지 않은 상황이라면 `func.call(this, x)`에서 현재 `this`(`=worker`)와 인수(`=2`)를 원본 메서드에 전달

### 여러 인수 전달하기

복수 인수를 가진 메서드, `worker.slow`를 캐싱하기

1. 복수 키를 지원하는 맵과 유사한 자료 구조 구현하기
   - 서드 파티 라이브러리 사용
2. 중첩 맵을 사용하기
   - `(max, result)` 쌍 저장은 `cache.set(min)` 사용
   - `result`는 `cache.get(min).get(max)`을 사용해 얻음
3. 두 값을 하나로 합치기
   - 맵의 키로 문자열 `"min,max"`를 사용
   - 여러 값을 하나로 합치는 코드는 해싱 함수(hashing function)에 구현해 유연성을 높임

```javascript
let worker = {
  slow(min, max) {
    alert(`slow(${min},${max})을/를 호출`);
    return min + max;
  }
};

function cachingDecorator(func, hash) {
  let cache = new Map();
  return function () {
    let key = hash(arguments); // (*)
    if (cache.has(key)) return cache.get(key);
    let result = func.call(this, ...arguments); // (**)
    cache.set(key, result);
    return result;
  };
}

function hash(args) {
  return args[0] + "," + args[1];
}

worker.slow = cachingDecorator(worker.slow, hash);
alert(worker.slow(3, 5)); // 8
alert("다시 호출: " + worker.slow(3, 5)); // 다시 호출: 8
```

- `(*)`: `hash`가 호출되면서 `arguments`를 사용한 단일 키가 생성됨
- `(**)`: `func.call(this, ...arguments)`로 컨텍스트(`this`)와 래퍼가 가진 인수 전부(`...arguments`)를 기존 함수에 전달함

### func.apply

`func.apply(this, arguments)`

- `func.call(this, ...arguments)` 대신 사용 가능
  - 받는 인수만 다르고 같은 역할을 함
- `func`의 `this`를 `context`로 고정
- 유사 배열 객체인 `args`를 인수로 사용할 수 있게 함
- `call`은 복수 인수를 따로 받음
- `apply`는 인수를 유사 배열 객체로 받음
  - 이터러블 사용 불가
  - 배열은 가능
- 대부분의 자바스크립트 엔진은 내부에서 `apply`를 최적화함
  - `apply`를 사용하는 것이 좀 더 빠름

```javascript
func.apply(context, args);
```

```javascript
func.call(context, ...args); // 전개 문법으로 인수가 담긴 배열 전달
func.apply(context, args); // 위 코드와 거의 같은 역할
```

```javascript
function func() {
  alert(arguments);
}
func(1, 2, 3);
func(...[1, 2, 3]);
func.call(null, 1, 2, 3);
func.apply(null, [1, 2, 3]);
```

콜 포워딩(call forwarding)

- 컨텍스트와 함께 인수 전체를 다른 함수에 전달하는 것
- 아래 코드처럼 외부에서 `wrapper`를 호출하는 경우, 기존 함수 `func`를 호출하는 것과 명확하게 구분할 수 없음

```javascript
// 가장 간단한 형태의 콜 포워딩
let wrapper = function () {
  return func.apply(this, arguments);
};
```

### 메서드 빌리기

위에서 구현한 해싱 함수를 개선해보기

- `args`의 요소 개수에 상관 없이 요소를 합치게 개선
- 배열이 아닌 것에 `join`을 호출하면 에러 발생
- `args`는 진짜 배열이 아닌 이터러블 객체나 유사 배열 객체
- 아래 방법을 사용하면 배열 메서드 `arr.join(glue)` 사용 가능

```javascript
function hash(args) {
  // return args[0] + "," + args[1];
  alert(arguments.join()); // Error
  return args.join(); // Error. ...args로 받았다면 괜찮음
}
hash(1, 2); // TypeError: arguments.join is not a function
```

메서드 빌리기(method borrowing)

- 일반 배열에서 `join` 메서드를 빌려오고 (`[].join`)
- `[].join.call`을 사용해 `arguments`를 컨텍스트로 고정한 후 `join` 메서드를 호출

1. `glue`가 첫 번째 인수가 되도록 함. 인수가 없으면 `","`가 첫 번째 인수가 됨
2. `result`는 빈 문자열이 되도록 초기화함
3. `this[0]`을 `result`에 덧붙임
4. `glue`와 `this[1]`을 `result`에 덧붙임
5. `glue`와 `this[2]`를 `result`에 덧붙임
6. `this.length`개의 항목이 모두 추가될 때까지 이 일을 반복
7. `result`를 반환

- `this`를 받고 `this[0]`, `this[1]` 등이 합쳐짐
- 이렇게 내부 알고리즘이 구현되어 있기 때문에 어떤 유사 배열이든 `this`가 될 수 있음
  - 상당수의 메서드가 이런 관습을 따름
- `this`에 `arguments`가 할당되더라도 잘 동작하는 이유

```javascript
function hash() {
  return [].join.call(arguments);
}
hash(1, 2); // 1,2
```

### 데코레이터와 함수 프로퍼티

데코레이터와 함수 프로퍼티

- 함수 또는 메서드를 데코레이터로 감싸 대체하는 것은 대체적으로 안전
- 그러나 원본 함수에 `func.calledCount` 등의 프로퍼티가 있는 경우
  - 데코레이터를 적용한 함수에서는 프로퍼티를 사용할 수 없음
- 위 코드에서 함수 `slow`에 프로퍼티가 있었다면 `cachingDecorator(slow)` 호출 결과인 래퍼에는 프로퍼티가 없음
- 몇몇 데코레이터는 자신만의 프로퍼티를 갖기도 함
  - 함수 호출 횟수 정보, 호출 시 소모되는 시간 등
  - 래퍼의 프로퍼티에 저장 가능
- 함수 프로퍼티에 접근할 수 있게 해주는 데코레이터를 만드는 방법도 있음
  - `Proxy`라는 특별한 객체를 사용해 함수를 감싸야 함

### 예제

Spy decorator

```javascript
function work(a, b) {
  alert(a + b);
}

function spy(func) {
  function wrapper(...args) {
    wrapper.calls.push(args);
    return func.apply(this, args);
  }
  wrapper.calls = []; // 함수 프로퍼티
  return wrapper;
}

work = spy(work);
work(1, 2); // 3
work(4, 5); // 9
for (let arg of work.calls) {
  alert(`call: ${arg.join()}`);
}
```

Delaying decorator

```javascript
function f(x) {
  console.log(x);
}

function delay(f, ms) {
  return function () {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

let f1000 = delay(f, 1000);
let f1500 = delay(f, 1500);
f1000("test");
f1500("test");
```

Debounce decorator

- cooldown 기간 후에 함수를 한 번 처리
- 최종 결과를 처리하는 데 좋음
- 사용자 입력 등

```javascript
function debounce(func, ms) {
  let timeout;
  return function () {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, arguments), ms);
  };
}

let f = debounce(alert, 1000); // 마지막 함수가 호출되고 1초 후에 실행

f("a");
setTimeout(() => f("b"), 200);
setTimeout(() => f("c"), 500); // c
```

Throttle decorator

- 주어진 시간보다 더 자주 실행되지 않음
- 자주 발생해서는 안 되는 정기적인 업데이트에 적합

```javascript
function f(a) {
  console.log(a);
}

function throttle(func, ms) {
  let isThrottled = false,
    savedArgs,
    savedThis;

  function wrapper() {
    // (2)
    if (isThrottled) {
      savedArgs = arguments;
      savedThis = this;
      return;
    }

    func.apply(this, arguments); // (1)
    isThrottled = true;

    setTimeout(function () {
      isThrottled = false; // (3)
      if (savedArgs) {
        wrapper.apply(savedThis, savedArgs);
        savedArgs = savedThis = null;
      }
    }, ms);
  }

  return wrapper;
}

let f1000 = throttle(f, 1000);

f1000(1);
f1000(2);
f1000(3);
f1000(4);
f1000(5);
// 1초 후, 5 출력. 2, 3, 4 무시
```

## 함수 바인딩

객체 메서드를 콜백으로 전달할 때 `this` 정보가 사라지는 문제

### 사라진 this

사라진 `this`

- 객체 메서드가 객체 내부가 아닌 다른 곳에 전달되어 호출되면 `this`가 사라짐

브라우저 환경에서 발생하는 일

- 인수로 전달받은 함수를 호출할 때, `this`에 `window`를 할당
- `this.firstName`은 `window.firstName`이 됨
- `window` 객체에는 `firstName`이 없어 `undefined` 출력
- 다른 유사한 사례들에서도 대부분 `this`는 `undefined`가 됨
- 메서드를 전달할 때, 컨텍스트도 제대로 유지하려면?

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`); // this=window
  }
};
setTimeout(user.sayHi, 1000); // Hello, undefined!. 객체에서 분리된 함수 user.sayHi가 전달됨
```

```javascript
let f = user.sayHi; // 아래 줄은 위 코드와 같음
setTimeout(f, 1000); // user 컨텍스트를 잃음
```

### 방법 1: 래퍼

래퍼 함수를 사용하는 것이 가장 간단

- 외부 렉시컬 환경에서 `user`를 받아서 보통 때처럼 메서드를 호출했기 때문에 제대로 동작
- `setTimeout`이 트리거되기 전(1초가 지나기 전)에 `user`가 변경되는 경우
  - 변경된 객체의 메서드가 호출됨
  - 약간의 취약성 발생

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(function () {
  user.sayHi(); // Hello, John!
}, 1000);

setTimeout(() => user.sayHi(), 1000); // Hello, John!
```

```javascript
setTimeout(() => user.sayHi(), 1000); // 또 다른 사용자!

user = {
  sayHi() {
    alert("또 다른 사용자!");
  }
};
```

### 방법 2: bind

`bind`

- 모든 함수는 `this`를 수정하게 해주는 내장 메서드 `bind`를 제공
- 함수처럼 호출 가능한 특수 객체(exotic object) 반환
- 이 객체를 호출하면 `this`가 `context`로 고정된 `func`가 반환됨
- 따라서 `boundFunc`를 호출하면 `this`가 고정된 `func`를 호출하는 것과 동일한 효과
- 더 긴 문법의 버전도 있음

```javascript
let boundFunc = func.bind(context);
```

```javascript
let user = { firstName: "John" };
function func(phrase) {
  alert(phrase + ", " + this.firstName);
}
let funcUser = func.bind(user); // this가 user로 고정된 func 할당
funcUser("Hello"); // Hello, John. 인수는 원본 함수 func에 그대로 전달
```

```javascript
let user = {
  firstName: "John",
  sayHi(phrase = "Hello") {
    alert(`${phrase}, ${this.firstName}!`);
  }
};
let sayHi = user.sayHi.bind(user);
sayHi(); // Hello, John!
sayHi("Bye"); // Bye, John!
setTimeout(sayHi, 1000); // Hello, John!. 아까 문제가 발생하지 않음
user = {
  sayHi() {
    alert("또 다른 사용자!");
  }
};
```

`bindAll`

- 메서드 전체 바인딩
- 객체에 복수의 메서드가 있고, 이 메서드 전체를 전달할 때
  - 반복문을 통해 메서드 바인딩 가능
- lodash 라이브러리의 `_.bindAll(object, methodNames)`

```javascript
for (let key in user) {
  if (typeof user[key] == "function") {
    user[key] = user[key].bind(user);
  }
}
```

### 부분 적용

부분 적용(partial application)

- `this` 뿐만 아니라 인수도 바인딩
  - 특정 인수가 고정되어 전달할 인수 감소
- 기존 함수의 매개변수를 고정하여 새로운 함수 생성
- 매우 포괄적인 함수를 기반으로 덜 포괄적인 변형 함수 생성
- 가독성이 좋은 이름의 독립 함수를 만들 수 있음
- 함수 `send(from, to, text)`가 있을 때
  - 객체 `user` 안에서 부분 적용을 활용해 전송 주체가 현재 사용자인 함수 `sendTo(to, text)` 구현

```javascript
let bound = func.bind(context, [arg1], [arg2], ...)
```

```javascript
function mul(a, b) {
  return a * b;
}
let double = mul.bind(null, 2); // 항상 컨텍스트를 넘겨야 하므로 null을 넘김
double(5); // = mul(2, 5) = 10
let triple = mul.bind(null, 3);
triple(5); // = mul(3, 5) = 15
```

### 컨텍스트 없는 부분 적용

컨텍스트 없는 부분 적용

- 인수 일부는 고정하고 컨텍스트 `this`는 고정하고 싶지 않은 경우
- 네이티브 `bind`만으로는 컨텍스트를 생략하고 인수로 바로 뛰어넘을 수 없음
- 대신 인수만 바인딩해주는 헬퍼 함수 `partial`을 직접 구현

```javascript
function partial(func, ...argsBound) {
  // (*)
  return function (...args) {
    return func.call(this, ...argsBound, ...args);
  };
}

let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

user.sayNow = partial(
  user.say,
  new Date().getHours() + ":" + new Date().getMinutes()
);

user.sayNow("Hello"); // [14:59] John: Hello!
```

- `partial(func[, arg1, arg2, ...])`을 호출하면 래퍼(`(*)`)가 반환됨
- 래퍼를 호출하면 `func`이 다음과 같이 동작
  - 동일한 `this`를 받음 (`user.sayNow`는 `user`를 대상으로 호출됨)
  - `partial`을 호출할 때 받은 인수(`"14:59"`)sms `...argsBound`에 전달됨
  - 래퍼에 전달된 인수(`"Hello"`)는 `...args`가 됨

### 예제

bind를 적용한 함수를 메서드에 정의하기

```javascript
"use strict";
function f() {
  alert(this);
}
let user = { g: f.bind(null) };
user.g(); // null
```

- bind를 적용한 함수의 컨텍스트는 완전히 고정되고 한번 고정되면 변경 불가

bind 두 번 적용하기

```javascript
function f() {
  alert(this.name);
}
f = f.bind({ name: "John" }).bind({ name: "Ahn" });
f(); // John
```

- `f.bind(...)`가 반환한 특수 객체인 묶인 함수(bound function)는 함수 생성 시점의 컨텍스트만 기억
- 인수가 제공되었다면 그 인수 또한 기억
- 한번 bind를 적용하면 bind를 사용해 컨텍스트를 다시 정의할 수 없음

bind를 적용한 함수의 프로퍼티

```javascript
function sayHi() {
  alert(this.name);
}
sayHi.test = 5; // 함수 프로퍼티에 값 할당
let bound = sayHi.bind({ name: "John" });
alert(bound.test); // undefined
```

- `bind`를 적용하면 또 다른 객체가 반환됨
- 새로운 객체에는 `test` 프로퍼티가 없으므로 `undefined`가 출력됨

this 값이 undefined인 함수 고치기

```javascript
function askPassword(ok, fail) {
  let password = prompt("비밀번호 입력", "");
  if (password == "rockstar") ok();
  else fail();
}

let user = {
  name: "John",
  loginOk() {
    alert(`${this.name}님이 로그인`);
  },
  loginFail() {
    alert(`${this.name}님이 로그인 실패`);
  }
};

askPassword(user.loginOk.bind(user), user.loginFail.bind(user));
```

- `askPassword`가 `loginOk`, `loginFail`을 객체 없이 가져오기 때문에 에러 발생
- `this=undefined`라고 가정함
- `bind` 함수로 고정시켜주면 해결
- 화살표 함수를 사용해 해결할 수도 있음
- ```javascript
  askPassword(
    () => user.loginOk(),
    () => user.loginFail()
  );
  ```
  - `askPassword`가 호출됐으나 프롬프트 대화상자에 값을 제출하고 `() => user.loginOk()`를 호출하기 전에 `user` 변수가 바뀌는 등의 복잡한 상황에서는 오작동할 수 있음

로그인에 부분 적용하기

```javascript
function askPassword(ok, fail) {
  let password = prompt("비밀번호 입력", "");
  if (password == "rockstar") ok();
  else fail();
}

let user = {
  name: "John",
  login(result) {
    alert(this.name + result ? " 로그인 성공" : " 로그인 실패");
  }
};

askPassword(
  () => user.login(true),
  () => user.login(false)
);

// askPassword(user.login.bind(user, true), user.login.bind(user, false));
```

## 화살표 함수 다시 살펴보기

화살표 함수

- 단순히 함수를 짧게 쓰기 위한 용도로 사용되지 않음
- 독특하고 유용한 기능을 제공
- 자바스크립트 사용 중에 멀리 떨어진 곳에서 실행될 작은 함수를 작성해야 하는 상황이 많음
  - `arr.forEach(func)`
  - `setTimeout(func)` 등등
- 자바스크립트에서 함수를 생성하고 어딘가에 전달하는 것은 자연스러움
- 그런데 함수를 전달하게 되면 함수의 컨텍스트를 잃을 수 있음
- 이때 화살표 함수를 사용하면 현재 컨텍스트를 잃지 않아 편리함
- 화살표 함수는 컨텍스트가 있는 긴 코드보다는 자체 컨텍스트가 없는 짧은 코드를 담을 용도로 만들어짐

화살표 함수가 일반 함수와 다른 점

1. `this`를 가지지 않음
2. `new`와 함께 호출할 수 없음
3. `arguments`를 지원하지 않음
4. `super`가 없음

### 화살표 함수에는 this가 없습니다

화살표 함수에는 `this`가 없음

- 화살표 함수 본문에서 `this`에 접근하면 외부에서 값을 가져옴
- 객체의 메서드 안에서 동일 객체의 프로퍼티를 대상으로 순회하는 데 사용 가능

화살표 함수는 `new`와 함께 실행할 수 없음

- `this`가 없기 때문에 화살표 함수는 생성자 함수로 사용할 수 없음

화살표 함수 vs. bind

- `.bind(this)`는 함수의 한정된 버전(bound version)으로 만듦
- 화살표 함수는 어떤 것도 바인딩시키지 않음
  - 단지 `this`가 없을 뿐
- 화살표 함수에서 `this`를 사용하면 `this`의 값을 외부 렉시컬 환경에서 탐색
  - 일반 변수 서칭과 마찬가지

```javascript
let group = {
  name: "fruitGroup",
  fruits: ["Apple", "Banana", "Orange"],
  showList() {
    this.fruits.forEach((fruit) => alert(`${this.name}: ${fruit}`)); // this=group
    // console.log(this); // this=group
    // this.fruits.forEach(function (fruit) {
    //   console.log(this); // this=global
    //   console.log(`${this.name}: ${fruit}`); // undefined: ... (엄격 모드였으면 에러)
    // });
  }
};
group.showList();
```

```javascript
const f = () => console.log(this);
f(); // {}
```

헷갈리는 `this`

```javascript
const obj = {
  name: "testObject",
  func() {
    console.log("from func: ", this);
    function f() {
      console.log("from f: ", this);
    }
    f();
  }
};

obj.func();
/*
from func:  { name: 'testObject', func: [Function: func] }
from f:  <ref *1> Object [global] { ... }
*/

const obj = {
  name: "testObject",
  func() {
    console.log("from func: ", this);
    const f = () => console.log("from f: ", this);
    f();
  }
};
/*
from func:  { name: 'testObject', func: [Function: func] }
from f:  { name: 'testObject', func: [Function: func] }
*/

const obj = {
  name: "testObject",
  func() {
    console.log("from func: ", this);
    function outerF() {
      console.log("from outerF: ", this);
      const innerF = () => {
        console.log("from innerF: ", this);
      };
      innerF();
    }
    outerF();
  }
};

obj.func();
/*
from func:  { name: 'testObject', func: [Function: func] }
from outerF:  undefined
from innerF:  undefined
*/
```

### 화살표 함수엔 arguments가 없습니다

화살표 함수에는 `arguments`가 없음

- 일반 함수와는 다르게 `arguments`를 지원하지 않음
  - `arguments`: 모든 인수에 접근할 수 있게 해주는 유사 배열 객체
- 현재 `this` 값과 `arguments` 정보를 함께 실어 호출을 포워딩해주는 데코레이터를 만들 때 유용하게 사용되는 특징

```javascript
function defer(f, ms) {
  return function () {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(who) {
  alert("안녕, " + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred("철수"); // 2초 후 "안녕, 철수" 출력
```

```javascript
// 화살표 함수를 사용하지 않고 동일한 기능을 하는 데코레이터 함수
function defer(f, ms) {
  return function (...args) {
    let ctx = this; // 컨텍스트 따로 저장
    setTimeout(function () {
      return f.apply(ctx, args);
    }, ms);
  };
}
```

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
