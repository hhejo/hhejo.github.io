---
title: 모던 JavaScript 튜토리얼 22 - 이진 데이터와 파일
date: 2024-01-15 20:36:07 +0900
last_modified_at: 2024-01-28 09:24:32 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags:
  [
    javascript,
    bindary-data,
    arraybuffer,
    typedarray,
    dataview,
    textdecoder,
    textencoder,
    blob,
    url,
    base64,
    file,
    filereader
  ]
---

ArrayBuffer, binary arrays, 텍스트 디코더와 텍스트 인코더, Blob, File and FileReader

## ArrayBuffer, binary arrays

바이너리 데이터(binary data)

- 웹 개발에서 우리는 주로 파일을 처리(생성, 업로드, 다운로드)하는 동안 바이너리 데이터를 만남
- 또 다른 일반적인 사용 사례는 이미지 처리. 이는 모두 자바스크립트에서 가능하며 이진 연산은 고성능
- 다만 클래스가 많아서 헷갈리는 경우도 있음 (`ArrayBuffer`, `Uint8Array`, `DataView`, `Blob`, `File` 등)
- 자바스크립트의 바이너리 데이터는 다른 언어에 비해 비표준 방식으로 구현됨

`ArrayBuffer`

```javascript
let buffer = new ArrayBuffer(16); // 16바이트의 연속 메모리 영역을 할당하고 0으로 채움
buffer.byteLength; // 16
```

- 기본 바이너리 객체(basic binary object)로, 고정 길이 연속 메모리 영역에 대한 참조
- 메모리에서 정확히 그만큼의 공간을 차지하고 길이는 고정되어 늘이거나 줄일 수 없음
- `ArrayBuffer`는 `Array`와 공통점이 없음. `ArrayBuffer`는 무언가의 배열이 아님
- `ArrayBuffer`는 메모리 영역으로, 이 안에 무엇이 저장되어 있는지는 전혀 단서가 없음. 단지 원시 바이트 시퀀스
- 개별 바이트에 접근하고 조작하려면 `buffer[index]`가 아닌 뷰(view) 객체가 필요
- 뷰 객체는 자체적으로 아무것도 저장하지 않고 `ArrayBuffer`에 저장된 바이트의 해석을 제공하는 안경
  - `Uint8Array`: ArrayBuffer의 각 바이트를 0~255의 정수로 처리 (부호 없는 8비트 정수, 8-bit unsigned integer)
  - `Uint16Array`: 2바이트마다 0~65535의 정수로 처리 (부호 없는 16비트 정수)
  - `Uint32Array`: 4바이트마다 0~4294967295 정수로 처리 (부호 없는 32비트 정수)
  - `Float64Array`: 8바이트마다 5.0x10^324~1.8x10^308 부동 소수점 숫자로 처리
- `ArrayBuffer`는 코어 객체, 모든 것의 루트, 원시 바이너리 데이터임
- 그러나 기본적으로 거의 모든 작업에 대해 작성하거나 반복하려면 뷰를 사용해야 함

```
             new ArrayBuffer(16)
Uint8Array    00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
Uint16Array     00    01    02    03    04    05    06    07
Uint32Array        00          01          03          04
Float64Array             00                      01
```

```javascript
let buffer = new ArrayBuffer(16); // 길이 16의 버퍼 생성
let view = new Uint32Array(buffer); // 버퍼를 32비트 정수 시퀀스로 처리
alert(Uint32Array.BYTES_PER_ELEMENT); // 4 (정수마다 4바이트)
alert(view.length); // 4 (많은 정수를 저장)
alert(view.byteLength); // 16 (바이트들의 사이즈)
view[0] = 123456; // 값 쓰기
for (let num of view) alert(num); // 123456, 0, 0, 0 (값에 대해 반복. 총 4개의 값)
```

### TypedArray

- `TypedArray`는 이러한 뷰에 대한 일반적인 용어로, 이들은 동일한 메서드 및 프로퍼티 집합을 공유
- `TypedArray`라는 생성자는 없고 그저 뷰 중 하나를 나타내는 일반적인 umbrella 용어
- `new TypedArray` 같은 것은 `new Int8Array`, `new Uint8Array` 등을 의미
- typed array는 일반 배열처럼 동작. 인덱스가 있고, 반복 가능
- typed array 생성자는 인수 타입에 따라 다르게 동작. 인수에는 5가지 변형이 있음

```javascript
new TypedArray(buffer, [byteOffset], [length]);
new TypedArray(object);
new TypedArray(typedArray);
new TypedArray(length);
new TypedArray();
```

1. `ArrayBuffer` 인수가 제공되면 그 위에 뷰가 생성됨
   - 뷰가 `buffer`의 `byteOffset`부터 `length`까지만 포함하도록 할 수 있음 (기본값은 0부터 버퍼 끝)
2. `Array`나 유사 배열 객체가 주어지면 동일한 길이의 형식화된(typed) 배열을 만들고 내용을 복사
   - 배열을 데이터로 미리 채울 수 있음
3. 다른 `TypedArray`가 제공되면 동일한 작업을 수행. 즉 동일한 길이의 형식화된(typed) 배열을 만들고 값을 복사
   - 필요한 경우 처리 도중에 값들이 새로운 타입으로 변환됨
4. 숫자 인수 `length`의 경우, 많은 요소를 포함하는 형식화된 배열을 생성
   - 해당 바이트 길이는 `length`와 단일 항목 `TypedArray.BYTES_PER_ELEMENT` 바이트들의 수를 곱한 값이 됨
5. 인수가 없으면 길이가 0인 typed array 생성

```javascript
// 1
let arrayBuffer = new ArrayBuffer(5);
let uint8Array = new Uint8Array(arrayBuffer); // Uint8Array(5) [ 0, 0, 0, 0, 0 ]
let uint16Array = new Uint16Array(arrayBuffer); // RangeError: byte length of Uint16Array should be a multiple of 2

// 2
let uint8Array = new Uint8Array([0, 1, 2, 3]);
alert(uint8Array.length); // 4 (같은 길이의 바이너리 배열 생성)
alert(uint8Array[1]); // 1 (주어진 값들을 4바이트로 채움(부호 없는 8비트 정수))

// 3
let uint16Array = new Uint16Array([1, 1000]);
let uint8Array = new Uint8Array(uint16Array);
alert(uint8Array[0]); // 1
alert(uint8Array[1]); // 232 (1000을 복사하려 했지만, 1000을 8비트에 맞출 수 없음)

// 4
let arr16 = new Uint16Array(4); // 4개 정수에 대한 형식화된 배열 생성 (각 값은 0)
alert(Uint16Array.BYTES_PER_ELEMENT); // 정수당 2바이트
alert(arr16.byteLength); // 8

// 5
let arr32 = new Uint32Array(); // Uint32Array(0) []
```

`ArrayBuffer`와 `TypedArray`

- `ArrayBuffer`를 언급하지 않고 `TypedArray`를 직접 만들 수 있음
- 그러나 기본 `ArrayBuffer` 없이는 뷰가 존재할 수 없기 때문에 첫번째 뷰(제공된 경우)를 제외한 모든 경우에 자동으로 생성됨
- `ArrayBuffer`에 접근하려면 다음 프로퍼티가 있어 항상 한 뷰에서 다른 뷰로 이동 가능
- `arr.buffer`: ArrayBuffer에 대한 참조
- `arr.byteLength`: ArrayBuffer의 길이

```javascript
let arr8 = new Uint8Array([0, 1, 2, 3]); // Uint8Array(4) [0, 1, 2, 3]
let arr16 = new Uint16Array(arr8.buffer); // Uint16Array(4) [0, 1, 2, 3] (동일한 데이터에 대한 다른 뷰)
let arr16x = new Uint16Array(arr8); // Uint16Array(2) [256, 770]
// [0, 1] -> [1, 0] -> (0b0000000100000000).toString() === "256"
// [2, 3] -> [3, 2] -> (0b0000001100000010).toString() === "770"
```

typed array 목록

- `Uin8Array`, `Uint16Array`, `Uint32Array`: 8, 16, 32비트 정수의 경우
- `Uint8ClampedArray`: 8비트 정수의 경우, 할당 시 'clamp'함
- `Int8Array`, `Int16Array`, `Int32Array`: 부호 있는 정수의 경우 (음수일 수 있음)
- `Float32Array`, `Float64Array`: 32, 64비트의 부호 있는 부동 소수점 숫자의 경우

`int8` 혹은 유사한 단일 값 유형 없음

- `Int8Array` 같은 이름에도 불구하고 `int` 혹은 `int8` 같은 단일 값 유형은 자바스크립트에 없음
- `Int8Array`는 이들 개별 값들의 배열이 아니고 `ArrayBuffer`에 대한 뷰이므로 논리적임

범위를 벗어난 행동

- typed array에 범위를 벗어난 값을 쓰려고 하면 에러가 발생하지 않으나, 여분의(extra) 비트는 잘림
- 256을 `Uint8Array`에 넣는 경우, 256은 바이너리 형식으로 `100000000`(9비트)이지만
- `Uint8Array`는 값당 8비트를 제공하므로 사용 가능한 범위는 0~255
- 더 큰 숫자의 경우, 가장 오른쪽(rightmost, less significant) 8비트만 저장되고 나머지는 잘림. 따라서 0을 얻음
- 257의 경우, 바이너리 형식은 `100000001`(9비트)이고 가장 오른쪽 8비트가 저장되므로 1을 얻음
- 즉, 모듈러 2^8이 저장됨
- `Uint8ClampedArray`: 255보다 큰 숫자에 대해서는 255를 저장하고, 음수에 대해서는 0을 저장. 이미지 처리에 유용

```
  8-bit integer
1 [0 0 0 0 0 0 0 0] 256
1 [0 0 0 0 0 0 0 1] 257
```

```javascript
let uint8array = new Uint8Array(16);
let num = 256;
alert(num.toString(2)); // 100000000 (바이너리 표현)
uint8array[0] = 256;
uint8array[1] = 257;
alert(uint8array[0]); // 0
alert(uint8array[1]); // 1
```

### TypedArray methods

- `TypedArray`는 주목할만한 예외를 제외하고 일반적인 `Array` 메서드를 가짐
- iterate, `map`, `slice`, `find`, `reduce` 등을 할 수 있음
- 할 수 없는 일도 몇 가지 있음
- `splice` 불가: typed array는 버퍼에 대한 뷰이고, 고정된 연속 메모리 영역이기 때문에 값 삭제 불가
  - 우리가 할 수 있는 일은 0을 할당하는 것뿐
- `concat` 불가
- 2가지 추가 메서드가 있음
- `arr.set(fromArr, [offset])`: `offset`에서 시작해(기본값은 0) `fromArr`에서 `arr`로 모든 요소 복사
- `arr.subarray([begin, end])`: `begin`에서 `end`까지 같은 타입의 새로운 뷰 생성
  - `slice` 메서드와 비슷하지만 아무것도 복사하지 않고 단지 주어진 데이터에 대해 작업하기 위해 새 뷰를 생성
- 이러한 메서드를 사용하면 typed array를 복사하고, 혼합하고, 기존 배열에서 새 배열을 만드는 등의 작업 수행 가능

### DataView

```javascript
new DataView(buffer, [byteOffset], [byteLength]);
```

- DataView: `ArrayBuffer`에 대한 특별하고 매우 유연한 untyped 뷰
- 모든 형식의 모든 오프셋의 데이터에 접근 가능
- typed array의 경우, 생성자는 형식이 무엇인지 지정
  - 전체 배열은 균일해야 함. i번째 숫자는 `arr[i]`
- `DataView`로 `.getUint8(i)` 혹은 `.getUint16(i)` 같은 메서드를 이용해 데이터에 접근
  - 생성 시간 대신 메서드를 호출할 때 형식을 지정
- `buffer`: 기본 `ArrayBuffer`. typed array와 다르게 `DataView`는 자체적으로 버퍼를 생성하지 않음. 우리는 그것을 준비해야 함
- `byteOffset`: 뷰의 시작 바이트 위치(기본값은 0)
- `byteLength`: 뷰의 바이트 길이(기본적으로 `buffer`의 끝까지)
- `DataView`는 혼합형식 데이터를 동일한 버퍼에 저장할 때 유용
- 일련의 쌍(16비트 정수, 32비트 부동 소수점)을 저장할 때 쉽게 접근 가능

```javascript
// 동일한 버퍼에서 다양한 형식의 숫자 출력하기
let buffer = new Uint8Array([255, 255, 255, 255]).buffer; // 4바이트의 바이너리 배열, 모두 최댓값 255
let dataView = new DataView(buffer);
alert(dataView.getUint8(0)); // 255 (오프셋 0의 8비트 숫자)
alert(dataView.getUint16(0)); // 65535 (오프셋 0의 16비트 숫자, 2바이트로 구성) (부호 없는 16비트 정수 최댓값)
alert(dataView.getUint32(0)); // 4294967295 (오프셋 0의 32비트 숫자) (부호 없는 32비트 정수 최댓값)
dataView.setUint32(0, 0); // 4바이트 숫자를 0으로 설정. 따라서 모든 바이트를 0으로 설정
```

### Summary

바이너리 데이터에 대해 작동하는 메서드를 설명하는 데 사용되는 2가지 추가 용어

- `ArrayBufferView`: 이러한 모든 종류의 뷰를 총칭하는 umbrella 용어
- `BufferSource`: `ArrayBuffer` 혹은 `ArrayBufferView`의 포괄적인 용어
  - 모든 종류의 바이너리 데이터를 의미하는 가장 일반적인 용어 중 하나

```
BufferSource ->
|                                 new ArrayBuffer(16)
                    Uint8Array    00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
                    Int8Array
                    Uint8ClampedArray

                    Uint16Array     00    01    02    03    04    05    06    07
                    Int16Array
ArrayBufferView
                    Uint32Array        00          01          03          04
                    Int32Array
                    Float32Array

                    Float64Array             00                      01

                    DataView      get/setUint8(offset), get/setFloat32(offset), ...
```

### 예제

타입이 지정된 배열 연결하기

주어진 `Uint8Array`의 배열을 하나의 배열로 연결하여 반환하는 함수 `concat(array)`

```javascript
function concat(arrays) {
  let totalLength = arrays.reduce((acc, value) => acc + value.length, 0); // 개별 배열 길이의 합
  if (!arrays.length) return null;
  let result = new Uint8Array(totalLength);
  let length = 0;
  for (let array of arrays) {
    result.set(array, length);
    length += array.length;
  } // 각 배열에 대해 result 위에 복사. 다음 배열이 이전 배열 바로 뒤에 복사됨
  return result;
}
```

## 텍스트 디코더와 텍스트 인코더

`TextDecoder`

- 주어진 버퍼와 인코딩으로 값을 실제 자바스크립트 문자열로 읽을 수 있게 해주는 내장 객체
- 이진 데이터가 문자열인 경우, 텍스트 데이터가 있는 파일을 받은 경우

```javascript
let decoder = new TextDecoder([label], [options]);
```

- `label`: 기본 인코딩 방식은 `utf-8`. `big5`, `windows-1251` 및 다른 인코딩 방식도 지원
- `options`: 선택 항목
  - `fatal`: `true`인 경우, 잘못된 글자(디코딩 불가능한 글자)를 대상으로 예외를 던짐
    - `false`(기본값)인 경우, 글자를 `\uFFD`로 대체
  - `ignoreBOM`: `true`인 경우, 사용되지 않는 바이트 순서 표식(Byte Order Mark, BOM)을 무시

```javascript
let str = decoder.decode([input], [options]);
```

- `input`: 디코딩할 `BufferSource`를 입력
- `options`: 선택 항목
  - `stream`: 많은 양의 데이터를 받아들여 `decoder`를 반복적으로 호출할 때도 디코딩이 반복적으로 실행됨
    - 이런 경우 멀티 바이트 문자가 많은 데이터로 분할될 수 있음
    - 이 옵션은 데이터 분할을 방지하기 위해 `TextDecoder`에 'unfinished' 문자를 입력시키고 다음 데이터가 오면 디코딩하도록 지시
- 버퍼의 하위 배열 뷰를 생성해 버퍼의 일부를 디코딩 가능

```javascript
let uint8Array = new Uint8Array([72, 101, 108, 108, 111]);
alert(new TextDecoder().decode(uint8Array)); // Hello

let uint8Array2 = new Uint8Array([0, 72, 101, 108, 111, 0]); // 문자열을 나타내는 배열의 요소가 중간에 존재
let binaryString = uint8Array2.subarray(1, -1); // 배열의 복사 없이 문자열 출력
alert(new TextDecoder().decode(binaryString)); // Hello
```

### 텍스트 인코더

```javascript
let encoder = new TextEncoder();
```

- `TextEncoder`: 문자열을 바이트로 변환. 인코딩 시 `utf-8`만 지원
- 2가지 메서드 존재
- `encode(str)`: `Uint8Array`의 문자열을 반환
- `encodeInto(str, destination)`: `Uint8Array` 구조 형태로 문자열 `str`을 `destination`에 인코딩

```javascript
let encoder = new TextEncoder();
let uint8Array = encoder.encode("Hello");
alert(uint8Array); // 72,101,108,108,111
```

## Blob

Blob

- `ArrayBuffer`와 뷰는 자바스크립트의 일부인 ECMA 표준의 일부임
- 브라우저에는 File Api, 특히 `Blob`에 설명된 추가 상위 수준(higher-level) 객체가 있음

`Blob`의 구성 요소

```
           type                      blobParts
Blob = [ image/png ] + [ blob1 | blob2 | ... | str | buffer ]
```

- `type`: 일반적으로 MIME 타입인 선택적 문자열
- `blobParts`: 다른 `Blob` 객체들의 시퀀스, 문자열, `BufferSource`

Blob 생성자 문법

```javascript
let blob = new Blob(blobParts, options);
```

- `blobParts`: `Blob/BufferSource/String` 값들의 배열
- `options`: 선택적 객체
  - `type`: `Blob` 타입. 일반적으로 MIME 타입 (`image/png` 등)
  - `endings`: `Blob`이 현재 OS 개행문자(`\r\n`, `\n`)에 해당하도록 end-of-line을 변환할지 여부
    - 기본값은 `transparent`로, 아무것도 하지 않음. `native`는 변환한다는 뜻

```javascript
let blob = new Blob(["<html>…</html>"], { type: "text/html" }); // 문자열로부터 Blob 생성

let hello = new Uint8Array([72, 101, 108, 108, 111]); // "Hello" in binary form
let blob2 = new Blob([hello, " ", "world"], { type: "text/plain" }); // typed array와 문자열로 Blob 생성
```

`Blob` 슬라이스 추출하기

```javascript
blob.slice([byteStart], [byteEnd], [contentType]);
```

- `byteStart`: 시작 바이트. 기본값은 0
- `byteEnd`: 마지막 바이트 (exclusive, by default till the end)
- `contentType`: 새로운 blob의 `type`. 기본적으로 소스와 동일
- `array.slice`와 인수가 유사하며 음수도 허용

`Blob` 객체는 불변(immutable)

- `Blob`에서 직접적으로 데이터 변경 불가
- `Blob`의 일부를 분할하고 그것으로부터 새 `Blob` 객체를 만들고, 새 `Blob`에 혼합하는 등의 작업 가능
- 이 동작은 자바스크립트 문자열과 유사
- 문자열의 문자를 변경할 수는 없지만 수정된 새 문자열을 만들 수는 있음

### Blob as URL

- Blob은 해당 컨텐츠를 표시하기 위해 `<a>`, `<img>` 혹은 다른 태그의 URL로 쉽게 사용할 수 있음
- `type` 덕분에 `Blob` 객체를 다운로드/업로드할 수도 있음
- 그리고 `type`은 자연스럽게 네트워크 요청에서 `Content-Type`이 됨

```html
<!-- 다운로드 속성은 브라우저가 탐색하는 대신 다운로드하도록 강제함 -->
<a download="hello.txt" href="#" id="link">Download</a>
<script>
  let blob = new Blob(["Hello, world!"], { type: "text/plain" });
  link.href = URL.createObjectURL(blob); // 링크를 클릭해 동적으로 생성된 Blob을 파일로 다운로드
</script>
```

```javascript
let link = document.createElement("a");
link.download = "hello.txt";
let blob = new Blob(["Hello, world!"], { type: "text/plain" });
link.href = URL.createObjectURL(blob);
link.click();
URL.revokeObjectURL(link.href);
```

`URL.createObjectURL`

- `Blob`을 가져와 그것을 위한 고유한 URL을 `blob:<origin>/<uuid>` 형식으로 생성
- `link.href`의 값은 `blob:https://javascript.info/1e67e00e-860d-40a5-89ae-6ab0cbee6273` 같이 보일 것

`URL.createObjectURL`로 생성된 각 URL

- 브라우저는 내부적으로 URL -> `Blob` 매핑을 저장하기에 이런 URL들은 짧지만 `Blob`에 접근할 수 있게 해줌
- 생성된 URL(및 그에 따른 링크)은 현재 문서가 열려있는 동안에만 유효
- 그것은 기본적으로 URL을 기대하는 다른 모든 객체의 `Blob`을 `<img>`,`<a>`에서 참조할 수 있음
- 그런데 부작용이 있음
- `Blob`에 대한 매핑이 있는 동안, `Blob` 자체는 메모리에 남아있고 브라우저는 그것을 해제할 수 없음
- 문서 unload 시 매핑은 자동적으로 지워지므로 `Blob` 객체는 해제됨
- 그러나 앱이 오래 지속된다면(long-living) 그런 일은 곧 일어나지 않음
- 따라서 URL을 생성하면 `Blob`은 더이상 필요하지 않더라도 메모리에 남아있게 됨

`URL.revokeObjectURL(url)`

- 내부 매핑에서 참조를 제거. revocation 후 매핑이 제거되므로 URL은 더이상 작동하지 않음
- `Blob`을 삭제하고(다른 참조가 없는 경우) 메모리를 해제할 수 있음
- 마지막 예에서는 `Blob`을 즉시 다운로드를 위해 한 번만 사용하려고 하므로 즉시 `URL.revokeObjectURL(link.href)`를 호출함
- 클릭 가능한 HTML 링크가 포함된 이전 예에서는 `URL.revokeObjectURL(link.href)`를 호출하지 않음
- `Blob` url이 유효하지 않게 되기 때문

### Blob to base64

- `URL.createObjectURL`의 대안은 `Blob`을 base64로 인코딩된 문자열로 변환하는 것임
- base64 인코딩은 바이너리 데이터를 0부터 64까지의 ASCII 코드를 사용하는 매우 안전한 readable(읽기 가능한) 문자열로 나타냄
- 더 중요한 것은 `data-urls`에서 이 인코딩을 사용할 수 있다는 것
- 브라우저는 문자열을 디코딩하고 이미지를 표시
- `FileReader`: `Blob`을 base64로 변환하기 위해 `FileReader` 내장 객체를 사용
- 이것은 다양한 형식의 Blob에서 데이터를 읽을 수 있음

```html
<img
  src="data:image/png;base64,R0lGODlhDAAMAKIFAF5LAP/zxAAAANyuAP/gaP///wAAAAAAACH5BAEAAAUALAAAAAAMAAwAAAMlWLPcGjDKFYi9lxKBOaGcF35DhWHamZUW0K4mAbiwWtuf0uxFAgA7"
/>
```

```javascript
let link = document.createElement("a");
link.download = "hello.txt";

let blob = new Blob(["Hello, world!"], { type: "text/plain" });
let reader = new FileReader();
reader.readAsDataURL(blob); // blob을 base64로 변환하고 onload 시 호출

reader.onload = function () {
  link.href = reader.result; // data url
  link.click();
};
```

Blob의 URL을 만드는 두 가지 방법

- `Blob`의 URL을 만드는 두 가지 방법 모두 사용할 수 있음
- 일반적으로 `URL.createObjectURL(blob)`이 더 간단하고 빠름
- URL.createObjectURL(blob)
  - 메모리에 관심(care)이 있는 경우 이를 취소(revoke)해야 함
  - blob에 직접 엑세스하고 인코딩/디코딩 없음
- Blob to data url
  - 아무것도 취소할 필요가 없음
  - 큰 `Blob` 객체의 인코딩을 위해 성능 및 메모리 손실

### Image to blob

이미지, 이미지 부분, 페이지 스크린샷에 대한 `Blob`을 만들 수 있음

어딘가에 업로드하는 것이 편리

이미지 작업은 `<canvas>` 요소를 통해 수행됨

1. `canvas.drawImage`를 사용해 canvas에 이미지(또는 그 일부)를 그림
2. 완료되면 `Blob`을 생성하는 캔버스 메서드 `.toBlob(callback, format, quality)`를 호출하고 `callback`을 실행

```javascript
// take any image
let img = document.querySelector("img");

// make <canvas> of the same size
let canvas = document.createElement("canvas");
canvas.width = img.clientWidth;
canvas.height = img.clientHeight;

let context = canvas.getContext("2d");

// copy image to it (this method allows to cut image)
context.drawImage(img, 0, 0);
// we can context.rotate(), and do many other things on canvas

// toBlob is async opereation, callback is called when done
canvas.toBlob(function (blob) {
  // blob ready, download it
  let link = document.createElement("a");
  link.download = "example.png";

  link.href = URL.createObjectURL(blob);
  link.click();

  // delete the internal blob reference, to let the browser clear memory from it
  URL.revokeObjectURL(link.href);
}, "image/png");
```

- 이미지가 복사되었지만, blob을 만들기 전에 이미지에서 잘라내거나 캔버스에서 변환할 수 있음

```javascript
let blob = await new Promise((resolve) =>
  canvasElem.toBlob(resolve, "image/png")
);
```

- `async/await`를 콜백 대신 선호하는 경우

페이지 스크린샷을 찍기 위해 라이브러리를 사용할 수 있음

- 그것이 실제로 하는 일은 단지 페이지를 돌아다니면서 `<canvas>`에 그리는 것 뿐임
- 그러면 위와 같은 방법으로 `Blob`을 얻을 수 있음

### From Blob to ArrayBuffer

`Blob` 생성자를 사용하면 `BufferSource`를 포함한 거의 어떤 것이든 그로부터 blob을 생성할 수 있음
그러나 낮은 수준(low-level)의 처리를 수행해야 하는 경우, 그것으로부터 `FileReader`를 사용해 가장 낮은 수준의 `ArrayBuffer`를 얻을 수 있음

```javascript
// get arrayBuffer from blob
let fileReader = new FileReader();

fileReader.readAsArrayBuffer(blob);

fileReader.onload = function (event) {
  let arrayBuffer = fileReader.result;
};
```

### Summary

`ArrayBuffer`, `Uint8Array` 혹은 다른 `BufferSource`가 바이너리 데이터인 반면, Blob은 유형이 있는 바이너리 데이터(binary data with type)를 나타냄

따라서 Blob은 브라우저에서 매우 일반적인 업로드/다운로드 작업에 편리

`XMLHttpRequest`, `fetch` 등과 같은 웹 요청을 수행하는 메서드들은 기본적으로 Blob을 사용할 수 있고 물론 다른 바이너리 유형에도 동작

## File and FileReader

`File` 객체는 `Blob`에서 상속되며 파일 시스템 관련 기능으로 확장됨

얻는 방법에는 2가지가 있음

`Blob`과 유사한 생성자가 있음

```javascript
new File(fileParts, fileName, [options]);
```

- `fileParts`: Blob/BufferSource/String 값의 배열
- `fileName`: 파일 이름 문자열
- `options`: 선택적 객체
  - `lastModified`: 마지막 수정의 타임스탬프(정수 날짜)

`<input type="file">`이나 드래그 앤 드롭이나 다른 브라우저 인터페이스에서 파일을 얻는 경우가 더 많음

- 이 경우 파일은 OS에서 이 정보를 가져옴

`File`이 `Blob`에서 상속되므로 `File` 객체는 같은 프로퍼티를 갖고 추가로,

- `name`: 파일 이름
- `lastModified`: 마지막 수정의 타임스탬프

이것이 우리가 `<input type="file">`로부터 `File` 객체를 얻을 수 있는 방법임

```html
<input type="file" onchange="showFile(this)" />
<script>
  function showFile(input) {
    let file = input.files[0];
    alert(`File name: ${file.name}`); // e.g my.png
    alert(`Last modified: ${file.lastModified}`); // e.g 1552830408824
  }
</script>
```

- input은 여러 파일을 선택할 수 있으므로 `input.files`는 유사 배열 객체

### FileReader

`FileReader`

- `Blob` 객체(따라서 `File` 또한)에서 데이터를 읽는 유일한 목적을 가진 객체
- 디스크에서 읽는 데 시간이 걸릴 수 있으므로 이벤트를 사용하여 데이터를 전달

```javascript
let reader = new FileReader(); // 인수 없음
```

- 주요 메서드
- `readAsArrayBuffer(blob)`: 데이터를 바이너리 형식의 `ArrayBuffer`로 읽음
- `readAsText(blob, [encoding])`: 주어진 인코딩(기본값 `utf-8`)을 사용해 데이터를 텍스트 문자열로 읽음
- `readAsDataURL(blob)`: 바이너리 데이터를 읽고 이를 base64 데이터 URL로 인코딩
- `abort()`: 작업을 취소

`read*` 메서드의 선택은 어떤 형식을 선호하는지, 데이터를 어떻게 사용할지에 따라 달라짐

- `readAsArrayBuffer`: 바이너리 파일의 경우 낮은 수준의 바이너리 작업을 수행
  - 슬라이싱과 같은 상위 수준 작업인 경우 `File`은 `Blob`에서 상속되므로 읽지 않고도 직접 호출 가능
- `readAsText`: 텍스트 파일의 경우, 문자열을 받고 싶을 때
- `readAsDataURL`: `src`에 있는 이 데이터를 `img` 또는 다른 태그에 사용하고 싶을 때
  - 그것을 위해 파일을 읽는 대안이 있음(`URL.createObjectURL(file)`)

읽기가 진행됨에 따라 이벤트가 발생함

- `loadstart`: 로딩이 시작됨
- `progress`: 읽는 동안 발생
- `load`: 에러 없이 읽기 완료
- `abort`: `abort()` 호출됨
- `error`: 에러가 발생함
- `loadend`: 성공 또는 실패로 읽기가 끝남

읽기가 끝나면 다음과 같이 결과에 엑세스할 수 있음

- `reader.result`: 성공한 경우의 결과
- `reader.error`: 실패한 경우의 에러

가장 널리 사용되는 이벤트는 `load`와 `error`

```html
<input type="file" onchange="readFile(this)" />
<script>
  function readFile(input) {
    let file = input.files[0];
    let reader = new FileReader();
    reader.readAsText(file);
    reader.onload = function () {
      console.log(reader.result);
    };
    reader.onerror = function () {
      console.log(reader.error);
    };
  }
</script>
```

blob에 대한 `FileReader`

- `FileReader`는 파일 뿐만 아니라 모든 blob을 읽을 수 있음
- 이를 사용해 blob을 다른 형식으로 변환할 수 있음
- `readAsArrayBuffer(blob)`: `ArrayBuffer`로 변환
- `readAsText(blob, [encoding])`: 문자열로 변환(`TextDecoder`의 대안)
- `readAsDataURL(blob)`: base64 데이터 url로 변환

`FileReaderSync`는 웹 워커 내부에서 사용 가능

- 웹 워커의 경우, `FileReaderSync`라는 `FileReader`의 동기 변형도 있음
- 그것의 읽기 메서드 `read*`는 이벤트를 생성하지 않고 다른 일반 함수처럼 결과를 반환
- 하지만 이는 웹 워커 내부에만 해당됨
- 왜냐하면 웹 워커에서 파일을 읽는 동안 가능한 동기 호출의 지연은 덜 중요하기 때문
- 페이지에는 영향을 주지 않음

### Summary

하지만 많은 경우에는 파일 내용을 읽을 필요가 없음

blob에서와 마찬가지로 `URL.createObjectURL(file)`로 짧은 URL을 생성하고 `<a>`나 `<img>`에 할당할 수 있음

이런 방식으로 파일을 다운로드하거나 이미지, 캔버스 등의 일부로 표시할 수 있음

네트워크를 통해 `File`을 전송하려는 경우 다음과 같이 간단함

`XMLHttpRequest`나 `fetch` 같은 네트워크 API는 기본적으로 `File` 객체를 받아들임

## 참고

> [이진 데이터와 파일](https://ko.javascript.info/binary)
