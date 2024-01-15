---
title: 모던 JavaScript 튜토리얼 20 - 기타 1
date: 2024-01-10 13:44:25 +0900
last_modified_at: 2024-01-14 08:45:16 +0900
categories: [JavaScript, Modern-JavaScript-Tutorial]
tags: [javascript]
---

Mutation observer, Selection and Range

## Mutation observer

`MutationObserver`

- DOM 요소를 관찰하고 변경 사항을 감지하면 콜백을 실행하는 내장 객체

### Syntax

```javascript
let observer = new MutationObserver(callback); // observer 생성
observer.observe(node, config); // DOM 노드에 연결
```

`config`

- 어떤 종류의 변경 사항에 반응할지에 대한 boolean 옵션이 있는 객체
- `childList`: `node`의 직계 children의 변경
- `subtree`: `node`의 모든 후손에서
- `attributes`: `node`의 속성들
- `attributeFilter`: 선택된 속성만 관찰하기 위한 속성 이름의 배열
- `characterData`: `node.data`(텍스트 내용) 관찰 여부
- `attributeOldValue`
  - `true`이면 속성의 이전과 새 값 모두 콜백에 전달
  - 그렇지 않으면 새 값만 전달(`attributes` 옵션 필요)
- `characterDataOldValue`
  - `true`이면 `node.data`의 이전과 새 값 모두 콜백에 전달
  - 그렇지 않으면 새 값만 전달(`characterData` 옵션 필요)

이후 어떠한 변경에도 `callback`이 실행됨

- 변경 사항은 첫 번째 인수에 MutationRecord 객체의 리스트로 전달됨
- observer 자체는 두 번째 인수로 전달됨

MutationRecord 객체의 프로퍼티

- `type`: 다음 중 하나의 mutation type
  - `"attributes"`: 수정된 속성
  - `"caracterData"`: 수정된 데이터로, 텍스트 노드에 사용됨
  - `"childList"`: 추가/제거된 자식 요소들
- `target`: 변경이 발생한 위치
  - `"attributes"`에 대한 요소
  - `"characterData"`에 대한 텍스트 노드
  - `"childList"` mutation에 대한 요소
- `addedNodes/removedNodes`: 추가/제거된 노드들
- `previousSibling/nextSibling`: 추가/제거된 이전 및 다음 형제 노드들
- `attributeName/attributeNamespace`: 변경된 속성의 이름/네임스페이스(XML 용)
- `oldValue`: 해당 옵션이 `attributeOldValue/characterDataOldValue` 설정된 경우 속성이나 텍스트 변경에 대해서만 이전 값

```html
<div contenteditable id="elem">Click and <b>edit</b>, please</div>
<script>
  let observer = new MutationObserver((mutationRecords) => {
    console.log(mutationRecords); // console.log(the changes)
  });
  // observe everything except attributes
  observer.observe(elem, {
    childList: true, // observe direct children
    subtree: true, // and lower descendants too
    characterDataOldValue: true // pass old data to callback
  });
</script>
```

- `contenteditable` 속성: 집중하고 편집 가능

```javascript
mutationRecords = [{
  type: "characterData",
  oldValue: "edit",
  target: <text node>,
  // other properties empty
}];
```

```javascript
mutationRecords = [{
  type: "childList",
  target: <div#elem>,
  removedNodes: [<b>],
  nextSibling: <text node>,
  previousSibling: <text node>
  // other properties empty
}, {
  type: "characterData"
  target: <text node>
  // ...mutation details depend on how the browser handles such removal
  // it may coalesce two adjacent text nodes "edit " and ", please" into one node
  // or it may leave them separate text nodes
}];
```

`MutationObserver`는 DOM 하위 트리 내 모든 변경 사항에 반응할 수 있음

### Usage for integration

도움되는 경우

- 유용한 기능을 포함하지만 원치 않는 작업도 수행하는 서드 파티 스크립트를 추가해야 하는 상황
  - 광고 표시 `<div class="ads">Unwanted ads</div>`
- 서드 파티 스크립트는 당연히 이를 제거하는 메커니즘을 제공하지 않음
- `MutationObserver`를 사용하면 원치 않는 요소가 DOM에 나타나는 시기를 감지하고 제거 가능
- 서드 파티 스크립트가 문서에 뭔가를 추가하는 경우
  - 그런 일이 발생하면 페이지를 조정하고, 무언가 크기를 동적으로 조정하는 등의 작업

### Usage for architecture

Dynamic highlight demo

### Additional methods

## Selection and Range

### Range

### Range methods

### Selection

### Selection properties

### Selection events

### Selection methods

### Selection in form controls

### Making unselectable

## 참고

> [기타](https://ko.javascript.info/ui-misc)
