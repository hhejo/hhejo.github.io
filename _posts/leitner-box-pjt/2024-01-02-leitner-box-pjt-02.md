---
title: Leitner Box Project 01
date: 2024-01-02 21:37:11 +0900
last_modified_at: 2024-01-03 22:04:52 +0900
categories: [leitner-box-pjt]
tags: [leitner-box-pjt, react]
---

리액트 프로젝트 디렉터리 구조, React Router Dom

## 리액트 프로젝트 디렉터리 구조 설정

그냥 components 폴더, routes 폴더 등등으로 구분하는 것이 맞는지 궁금해져서 찾아봤다.

리액트 프로젝트든, 뷰 프로젝트든, 앵귤러 프로젝트든 어떤 프로젝트이든지 간에 프로젝트 디렉터리 구조는

시스템에서 사용한 프레임워크를 설명하는 것이 아닌, 해당 코드가 구현하는 시스템을 알려줘야 한다고 한다.

## React Router

URL 관리를 위해 도입했다.

기능이 많아 사용하면서 차근차근 정리하고 있다.

컴포넌트를 로드하면서 데이터를 불러들이는 기능도 있다. 나중에 사용해야겠다.

URL을 분리하면서 route 파일을 만들었고, URL 상수도 정의해서 따로 뺐다.

## 리액트 파일명

윈도우와 다르게 맥은 파일의 대소문자를 구분하지 않는다고 한다.

하지만 깃은 대소문자 구분을 한다.

그래서 호환성을 위해 파일 이름을 소문자와 `-`를 같이 사용하는 것이 좋다고 한다.

처음 배울 때는 `App.jsx`, `PostDetail.jsx`처럼 작성했었는데, 케밥케이스로 바꿔야겠다.

더 공부해서 포스트를 작성해야겠다.

## 현재까지 코드 저장소

> [leitner-box-frontend](https://github.com/hhejo/leitner-box-frontend/tree/0eff49fc413cc7ba4c2d719153296bc9533ca2c7)

> [leitner-box-backend](https://github.com/hhejo/leitner-box-backend/tree/12938585e3245510db250f042914185666368922)
