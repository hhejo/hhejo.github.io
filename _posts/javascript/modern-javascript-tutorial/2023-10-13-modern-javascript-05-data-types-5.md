---
title: 모던 JavaScript 튜토리얼 05 - 자료구조와 자료형 5
date: 2023-10-13 08:20:13 +0900
last_modified_at: 2024-01-18 07:08:12 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript, date]
---

Date

## Date 객체와 날짜

`Date` 객체

- 생성 및 수정 시간을 저장
- 시간을 측정
- 현재 날짜 파악

### 객체 생성하기

- `new Date()`: 인수 없이 호출하면 현재 날짜와 시간이 저장된 `Date` 객체 반환
- `new Date(milliseconds)`
  - UTC 기준(UTC+0) 1970년 1월 1일 0시 0분 0초에서 `milliseconds` 밀리초 후의 시점이 저장된 `Date` 객체 반환
- 타임스탬프(timestamp): 1970년의 첫날을 기준으로 흘러간 밀리초를 나타내는 정수
  - 1970년 1월 1일 이전 날짜에 해당하는 타임스탬프 값은 음수
- `new Date(datestring)`: 인수가 하나인데 문자열이라면 자동으로 구문 분석(parsed)됨
  - `Date.parse`에서 사용하는 구문 분석 알고리즘과 동일
- `new Date(year, month, date, hours, minutes, seconds, ms)`
  - 주어진 인수를 조합해 만들 수 있는 날짜가 저장된 객체 반환(지역 시간대 기준)
  - `year`, `month`만 필수
  - `year`: 반드시 네 자리 숫자
  - `month`: `0`(1월) ~ `11`(12월) 사이의 숫자
  - `date`: 일을 나타내는데 값이 없으면 1일
  - `hours`, `minutes`, `seconds`, `ms`에 값이 없는 경우 `0`으로 처리
  - 최소 정밀도는 1밀리초

```javascript
let now = new Date();
alert(now); // Fri Oct 13 2023 08:23:56 GMT+0900 (GMT+09:00)

let Jan01_1970 = new Date(0); // Thu Jan 01 1970 09:00:00 GMT+0900 (GMT+09:00)
let Jan02_1970 = new Date(24 * 3600 * 1000); // Fri Jan 02 1970 09:00:00 GMT+0900 (GMT+09:00)
let Dec31_1969 = new Date(-24 * 3600 * 1000); // Wed Dec 31 1969 09:00:00 GMT+0900 (GMT+09:00)

let date = new Date("2017-01-26"); // Thu Jan 26 2017 09:00:00 GMT+0900 (GMT+09:00)
// 인수로 시간을 지정하지 않았기 때문에 GMT 자정이라고 가정하고
// 코드가 실행되는 시간대(timezone)에 따라 출력 문자열이 바뀜

new Date(2011, 0, 1, 0, 0, 0, 0); // Sat Jan 01 2011 00:00:00 GMT+0900 (GMT+09:00)
new Date(2011, 0, 1); // 위와 동일
let date = new Date(2011, 0, 1, 2, 3, 4, 567); // Sat Jan 01 2011 02:03:04 GMT+0900 (GMT+09:00)
```

### 날짜 구성요소 얻기

- `getFullYear()`: 연도(네 자릿수) 반환
  - `getYear()`는 더는 사용되지 않는(deprecated) 비표준 메서드
  - 자바스크립트 엔진이 구현하고 있으나 두 자릿수 연도를 반환하는 경우가 있기에 사용하지 말 것
- `getMonth()`: 월(0 이상 11 이하) 반환
- `getDate()`: 일(1 이상 31 이하) 반환
- `getHours()`, `getMinutes()`, `getSeconds()`, `getMilliseconds()`: 시, 분, 초, 밀리초 반환
- `getDay()`: 일요일(`0`) 부터 토요일(`6`)을 나타내는 숫자 중 하나 반환
- 위 메서드 모두 현지 시간 기준 날짜 구성 요소를 반환
  - get 다음 UTC를 붙여주면 표준시(UTC+0) 기준의 날짜 구성 요소를 반환해주는 메서드를 만들 수 있음
  - `getUTCFullYear()`, `getUTCMonth()`, `getUTCDate()`, `getUTCDay()`
  - `getUTCHours()`, `getUTCMinutes()`, `getSeconds()`, `getUTCMilliseconds()`
  - 현지 시간대가 UTC 시간대와 다르다면 다른 값 출력
- `getTime()`: 주어진 일시와 1970년 1월 1일 00시 00분 00초 사이의 간격(밀리초 단위)인 타임스태프 반환
  - 표준시(UTC+0) 기준의 날짜 구성 요소를 반환해주는 메서드 없음
- `getTimezoneOffset()`: 현지 시간과 표준 시간의 차이(분)를 반환
  - 표준시(UTC+0) 기준의 날짜 구성 요소를 반환해주는 메서드 없음

```javascript
let date = new Date(); // 현재 일시
alert(date.getHours()); // 현지 시간 기준 시
alert(date.getUTCHours()); // 표준시간대(UTC+0, 일광 절약 시간제를 적용하지 않은 런던 시간) 기준 시

alert(new Date().getTimezoneOffset()); // -540
```

### 날짜 구성 요소 설정하기

- `setFullYear(year, [month], [date])`
- `setMonth(month, [date])`
- `setDate(date)`
- `setHours(hour, [min], [sec], [ms])`
- `setMinutes(min, [sec], [ms])`
- `setSeconds(sec, [ms])`
- `setMilliseconds(ms)`
- `setTime(milliseconds)`: 1970년 1월 1일 00:00:00 UTC부터 밀리초 이후를 나타내는 날짜를 설정
- `setTime()`을 제외한 모든 메서드는 `setUTCHours()` 같이 표준시에 따라 날짜 구성 요소를 설정해주는 메서드가 있음

```javascript
let today = new Date();
today.setHours(0); // 날짜는 변경되지 않고 시만 0으로 변경
today.setHours(0, 0, 0, 0); // 날짜는 변경되지 않고 시, 분, 초 모두 0으로 변경
```

### 자동 고침(autocorrection)

- 범위를 벗어나는 값을 설정하면 값이 자동으로 수정됨
- 입력받은 날짜 구성 요소가 범위를 벗어나면 초과분은 자동으로 다른 날짜 구성 요소에 배분됨
- 2016년 2월 28일의 이틀 뒤 날짜를 구하고 싶을 때
  - 답은 3월 2일 혹은 3월 1일(윤년)
  - 2016년이 윤년인지 아닌지 생각할 필요 없이 단순히 이틀을 더해주기만 하면 됨
- 자동 고침은 일정 시간이 지난 후의 날짜를 구하는 데도 사용
- 0이나 음수를 날짜 구성 요소에 설정할 수 있음

```javascript
let date = new Date(2023, 0, 32); // 2023년 1월 32일은 없음 -> 2월 1일로 자동 수정
alert(date); // Wed Feb 01 2023 00:00:00 GMT+0900 (GMT+09:00)

let date = new Date(2016, 1, 28);
date.setDate(date.getDate() + 2);
alert(date); // Tue Mar 01 2016 00:00:00 GMT+0900 (GMT+09:00)

let date = new Date();
date.setSeconds(date.getSeconds() + 70); // 09:21:33
alert(date); // 09:22:43

let date = new Date(2016, 0, 2); // 2016년 1월 2일
date.setDate(1); // 1일로 변경
alert(date); // 01 Jan 2016
date.setDate(0); // 일의 최솟값은 1이므로 0을 입력하면 전 달의 마지막 날을 설정한 것과 같은 효과
alert(date); // 31 Dec 2015
```

### Date 객체를 숫자로 변경해 시간차 측정하기

- `Date` 객체를 숫자형으로 변경
  - 타임스탬프(`date.getTime()`을 호출시 반환하는 값)가 됨
- 날짜에 마이너스 연산자를 적용해 밀리초 기준 시차 적용 가능

```javascript
let date = new Date();
alert(+date); // 1697416100633
alert(date.getTime()); // 1697416100633

let start = new Date(); // 측정 시작
for (let i = 0; i < 100000; i++)
  let doSomething = i * i * i; // 원하는 작업 수행
let end = new Date(); // 측정 종료
alert(`반복문을 모두 도는 데 ${end - start} 밀리초가 걸렸습니다.`);
```

### Date.now()

`Date.now()`

- 현재 타임스탬프를 반환하는 메서드
- `Date` 객체를 만들지 않고도 시차를 측정할 수 있음
- `Date.now()`는 `new Date().getTime()`과 의미론적으로 동일하나 중간에 `Date` 객체를 생성하지 않음
- `new Date().getTime()`을 사용하는 것보다 빠르고 가비지 컬렉터의 일을 덜어줌
- 성능이 중요한 경우 `Date.now()`를 자주 활용
  - 자바스크립트를 사용해 게임 구현, 전문 분야의 애플리케이션 구현
- 편의를 위해서도 활용

```javascript
// 위 예시보다 성능 더 좋아짐
let start = Date.now();
for (let i = 0; i < 100000; i++)
  let doSomething = i * i * i;
let end = Date.now();
alert(`반복문을 모두 도는 데 ${end - start} 밀리초가 걸렸습니다.`);
```

### 벤치마크 테스트

- 비교 대상을 두고 성능을 비교하여 시험하고 평가할 때
- CPU를 많이 잡아먹는 함수의 신뢰할만한 벤치마크(평가 기준)를 구하려면 주의가 필요

```javascript
function diffSubtract(date1, date2) {
  return date2 - date1;
}
function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}
function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();
  let start = Date.now();
  for (let i = 0; i < 100000; i++) f(date1, date2);
  return Date.now() - start;
}

alert(`diffSubtract를 십만번 호출하는데 걸린 시간: ${bench(diffSubtract)}ms`); // 16ms
alert(`diffGetTime을 십만번 호출하는데 걸린 시간: ${bench(diffGetTime)}ms`); // 3ms
```

- 형 변환이 없어서 엔진 최적화에 드는 자원이 줄어드므로 `getTime()`을 이용한 방법이 빠름
- 이 벤치마크는 좋지 않음
  - `bench(diffSubstract)`를 실행하고 있을 때 CPU가 어떤 작업을 병렬적으로 처리하고 있고, 여기에 CPU의 자원이 투입되고 있었다고 가정
  - `bench(diffGetTime)`을 실행할 땐 이 작업이 끝난 상태라고 가정
    - 멀티 프로세스를 지원하는 운영체제에서는 흔히 발생
  - 첫 번째 `benchmark`가 실행될 땐 사용할 수 있는 CPU 자원이 적었기 때문에 이 벤치마크는 좋지 않음
- 좀 더 신뢰할만한 벤치마크 테스트를 만들려면 `benchmark`를 번갈아 여러번 돌려야 함

```javascript
function diffSubtract(date1, date2) {
  return date2 - date1;
}
function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}
function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();
  let start = Date.now();
  for (let i = 0; i < 100000; i++) f(date1, date2);
  return Date.now() - start;
}

let time1 = 0;
let time2 = 0;
// 함수 bench를 각 함수(diffSubtract, diffGetTime)별로 10번씩 실행
for (let i = 0; i < 10; i++) {
  time1 += bench(diffSubtract);
  time2 += bench(diffGetTime);
}
alert(`diffSubtract에 소모된 시간: ${time1}`); // 125
alert(`diffGetTime에 소모된 시간: ${time2}`); // 4
```

- 모던 자바스크립트 엔진은 아주 많이 실행된 코드인 'hot code'를 대상으로 최적화 수행
  - 실행 횟수가 적은 코드는 최적화할 필요가 없음
- 위 예시에서 bench를 첫 번째 실행했을 때는 최적화가 잘 적용되지 않음
- 그래서 아래 코드처럼 메인 반복문을 실행하기 전에 예열용(heat-up)으로 bench를 실행할 수 있음

```javascript
// 메인 반복문 실행 전, "예열용"으로 추가한 코드
bench(diffSubtract);
bench(diffGetTime);

// 벤치마크 테스트 시작
for (let i = 0; i < 10; i++) {
  time1 += bench(diffSubtract);
  time2 += bench(diffGetTime);
}
```

- 모던 자바스크립트 엔진은 최적화를 많이 하기 때문에 만들어진 테스트가 실제 사례와 결과가 다를 수 있음
- 특히 연산자, 내장 함수와 같이 아주 작은 것일 수록 더 다를 수 있음

### Date.parse와 문자열

`Date.parse(str)`

- 문자열에서 날짜를 읽음

문자열의 형식

- `YYYY-MM-DDTHH:mm:ss.sssZ`
- `YYYY-MM-DD`: 날짜(연-월-일)
- `T`: 구분 기호
- `HH:mm:ss.sss`: 시:분:초.밀리초
- `Z`(옵션): `+-hh:mm` 형식의 시간대를 나타냄. `Z` 한 글자인 경우엔 UTC+0을 나타냄
- `YYYY-MM-DD`, `YYYY-MM`, `YYYY` 같이 더 짧은 문자열 형식도 가능
- 문자열과 대응하는 날짜의 타임스탬프 반환
- 전달한 문자열의 형식이 조건에 맞지 않은 경우 `NaN` 반환

```javascript
let ms = Date.parse("2012-01-26T13:51:50.417-07:00");
alert(ms); // 1327611110417 (타임스탬프)

// 타임스탬프만으로 새로운 Date 객체 만들기
let date = new Date(Date.parse("2012-01-26T13:51:50.417-07:00"));
alert(date);
```

### 기타

`Date` 객체는 마이크로초를 지원하지 않음

`performance.now()`

- 브라우저 환경의 메서드
- 페이지 로딩에 걸리는 밀리초 반환
- 소수점 아래 세 자리까지 지원

`microtime` 모듈

- Node.js 환경
- 마이크로초 사용 가능

## 참고

> [자료구조와 자료형](https://ko.javascript.info/data-types)
