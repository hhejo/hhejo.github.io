---
title: React 프로젝트의 디렉터리 구조
date: 2024-01-02 09:09:28 +0900
last_modified_at: 2024-01-03 11:54:34 +0900
categories: [React]
tags: [react, structure]
---

React 프로젝트의 디렉터리는 어떻게 구성할까요

## 파일 타입에 따른 분류

```
/src
  /components
    /header
    /footer
    /button
    /nav
    /card
  /contexts
    todo-list.context.js
  /hooks
    use-todo-list.js
```

- 컴포넌트 개수가 많아지면 찾기 힘듦

## 페이지에 따른 분류

```
/src
  /components
    /todo-form
    /ui
  /contexts
    modal.context.js
    todo-list.context.js
  /pages
    /create-todo
    /home
      home-page.js
      /edit-todo-modal
        /todo-list
          todo-item.component.js
          todo-list.component.js
          todo-list.test.js
    /login
    /privacy
    /signup
    /terms
```

## 기능에 따른 분류

도메인 주도 설계의 핵심

- 해당 도메인의 핵심 엔티티가 중심이 되어야 함
- 특정 엔티티를 다루는 코드들은 엔티티 중심으로 colocate
- 데이터 의존성이 없는 presentation 컴포넌트의 경우, 기능이 지원하는 대상을 엔티티 명으로 함

```
/src
  /features
    /todos
      index.js
      /create-todo-form
      /edit-todo-modal
      /todo-form
      /todo-list
        index.js
        todo-item.component.js
        todo-list.component.js
        todo-list.context.js
        todo-list.text.js
        use-todo-list.js
    /projects
      index.js
      /create-project-form
      /project-list
    /ui
      index.js
      /button
      /card
      /checkbox
      /header
      /footer
      /modal
      /text-field
    /users
      index.js
      /login
      /signup
      use-auth.js
  /pages
    create-project.js
    create-todo.js
    index.js
    login.js
    privacy.js
    project.js
    signup.js
    terms.js
```

## 기능 중심 폴더 구조와 스크리밍 아키텍처

Feature-Driven Folder Structure and Screaming Architecture

- 아키텍처는 시스템에서 사용한 프레임워크를 설명하는 것이 아니라
- 해당 코드가 구현하는 시스템을 이야기해야 함
- 건강 관리 시스템을 구축하고 있다면,
- 새로운 프로그래머가 폴더 구조를 보자마자 건강 관리 시스템 코드임을 알아야 함

```
/src
  /components
  /contexts
  /hooks
```

- 위 구조를 보고 리액트 앱인지 알 수는 있으나, 프레임워크를 나타낼 뿐이지 시스템에 대해 설명하지는 않음

```
/src
  /features
    /todos
    /projects
    /ui
    /users
  /pages
    create-project.js
    create-todo.js
    index.js
    login.js
    privacy.js
    project.js
    signup.js
    terms.js
```

- features, pages 폴더 둘 다 프로젝트 관리 앱임을 파일명으로 나타냄
- 해당 폴더에서 해당 앱의 컴포넌트를 찾을 수 있음

## 자주 쓰는 폴더 이름

`components`

- 렌더링되는 페이지에서 사용되는 공통된 컴포넌트 관리
- `Header`, `Footer`, `Nav`, ...

`pages`

- 라우팅되는 실제 페이지
- `LoginPage`, `MainPage`, `DetailPage`, `SignUp`, ...

`assets`

- 이미지, 폰트, json 등

`helper`, `utils`

- 자주 사용하는 로직을 재활용할 수 있도록 만든 라이브러리
- 예전부터 프로젝트에서 사용하던 방식

`hooks`

- 리액트에서 자주 사용하는 로직을 재활용할 수 있도록 만든 라이브러리
- 커스텀 훅들의 모음
- `useFetch`, ...

`api`

- API 관련 로직을 모아둔 폴더

`services`

- API 호출이나 외부 서비스와 관련된 코드를 저장하는 폴더

`store`, `contexts`

- Redux, MobX 등의 상태 관리 라이브러리와 관련된 코드를 저장하는 폴더

## 참고

> [[React🍇] 폴더 구조, 자주 쓰는 폴더명](https://drmutterl.tistory.com/entry/React-%ED%8F%B4%EB%8D%94-%EA%B5%AC%EC%A1%B0-%EC%9E%90%EC%A3%BC-%EC%93%B0%EB%8A%94-%ED%8F%B4%EB%8D%94%EB%AA%85)

> [React 활용 - 7 : 폴더 구조 잡기](https://cookinghoil.tistory.com/127)

> [Refactoring React(리팩토링 리액트) : Folder Structure(폴더 구조)](https://itchallenger.tistory.com/900)

> [Delightful React File/Directory Structure](https://www.joshwcomeau.com/react/file-structure/)
