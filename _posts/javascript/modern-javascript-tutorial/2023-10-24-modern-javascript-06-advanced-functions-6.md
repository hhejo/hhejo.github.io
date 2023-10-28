---
title: 모던 JavaScript 튜토리얼 06 - 함수 심화학습 6
date: 2023-10-24 12:06:46 +0900
last_modified_at: 2023-10-28 11:01:23 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

call/apply와 데코레이터, 포워딩

## call/apply와 데코레이터, 포워딩

자바스크립트는 함수를 다룰 때 탁월한 유연성을 제공

- 함수는 이곳저곳 전달될 수 있고, 객체로도 사용될 수 있음

함수 간에 호출을 어떻게 포워딩(forwarding) 하는지, 데코레이팅(decorating) 하는지 알아보기

### 코드 변경 없이 캐싱 기능 추가하기

CPU를 많이 잡아먹지만 결과는 안정적인 함수 `slow(x)`

- 결과가 안정적이라는 말은 `x`가 같으면 호출 결과도 같다는 것
- `slow(x)`가 자주 호출된다면 결과를 어딘가에 저장(캐싱)해 재연산에 걸리는 시간을 줄이는 것이 좋음
- `slow()` 안에 캐싱 관련 코드를 추가하는 대신, 래퍼 함수를 만들어 캐싱 기능을 추가할 것

```javascript
function slow(x) {
  // CPU 집약적인 작업이 여기 올 수 있음
  alert(`slow(${x})을/를 호출함`);
  return x;
}
function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
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

모든 함수를 대상으로 `cachingDecorator`를 호출할 수 있음

- 반환되는 것은 캐싱 래퍼
- 함수에 `cachingDecorator`를 적용하기만 하면 캐싱이 가능한 함수를 원하는 만큼 구현할 수 있기 때문에 데코레이터 함수는 유용하게 사용됨
- 캐싱 관련 코드를 함수 코드와 분리할 수 있기 때문에 함수의 코드가 간결해짐

`cachingDecorator(func)`를 호출하면 래퍼(wrapper), `function(x)`이 반환됨

- 래퍼 `function(x)`는 `func(x)`의 호출 결과를 캐싱 로직으로 감쌈(wrapping)

바깥 코드에서 봤을 때 함수 `slow`는 래퍼로 감싼 이전이나 이후나 동일한 일을 수행

- 행동 양식에 캐싱 기능이 추가된 것뿐

`slow` 본문을 수정하는 것보다 독립된 래퍼 함수 `cachingDecorator`를 사용할 때 생기는 이점

- `cachingDecorator`를 재사용 할 수 있음
  - 원하는 함수 어디에든 `cachingDecorator`를 적용할 수 있음
- 캐싱 로직이 분리되어 `slow` 자체의 복잡성이 증가하지 않음
- 필요하다면 여러 개의 데코레이터를 조합해서 사용할 수도 있음(추가 데코레이터는 `cachingDecorator` 뒤를 따름)

### func.call을 사용해 컨텍스트 지정하기

위에서 구현한 캐싱 데코레이터는 객체 메서드에 사용하기에는 적합하지 않음

- 객체 메서드 `worker.slow()`는 데코레이터 적용 후 제대로 동작하지 않음

```javascript
let worker = {
  someMethod() {
    return 1;
  },
  slow(x) {
    // CPU 집약적인 작업이라 가정
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
```

- `(*)` 줄에서 `this.someMethod` 접근에 실패했기 때문에 에러 발생
- `(**)` 줄에서 래퍼가 기존 함수 `func(x)`를 호출하면 `this`가 `undefined`가 되기 때문

```javascript
let func = worker.slow;
func(2); // 비슷한 증상 발생
```

- 래퍼가 기존 메서드 호출 결과를 전달하려 했지만 `this`의 컨텍스트가 사라졌기 때문에 에러 발생

`func.call(context, ...agrs)`

- `this`를 명시적으로 고정해 함수를 호출할 수 있게 해주는 특별한 내장 함수 메서드

```javascript
func.call(context, arg1, arg2, ...)
```

- 메서드를 호출하면 첫 번째 인수가 `this`, 이어지는 인수가 `func`의 인수가 된 후 `func`이 호출됨

```javascript
func(1, 2, 3);
func.call(obj, 1, 2, 3); // 윗 줄과 거의 동일한 일 발생
```

- `func.call`에선 `this`가 `obj`로 고정되는 것이 차이

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
say.call(user, "Hello");
```

...

## 참고

> [함수 심화학습](https://ko.javascript.info/advanced-functions)
