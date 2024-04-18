---
title: package.json과 package-lock.json의 차이
date: 2024-04-18 20:40:50 +0900
last_modified_at: 2024-04-18 20:40:50 +0900
categories: [JavaScript]
tags: [javascript, nodejs, package]
---

둘은 어떤 차이점이 있을까요?

## package.json과 package-lock.json

`package.json`과 `package-lock.json` 파일은 모두 `Node.js` 프로젝트의 중요한 구성 요소이지만 서로 다른 용도로 사용됨

### 목적

`package.json`

- 주로 이름, 버전, 작성자(author), 종속성(dependencies), 스크립트(scripts) 및 기타 구성 세부 정보를 포함하여 프로젝트에 대한 메타데이터를 관리하고 문서화하는 데 사용됨
- 프로젝트의 manifest 역할

`package-lock.json`

- 패키지를 설치하거나 업데이트할 때 `npm`에 의해 자동으로 생성 및 업데이트됨
- 프로젝트에 설치된 종속성의 정확한 버전을 잠그는(lock) 데 사용됨
- 여러 환경에서 재현성과 일관된 설치를 보장

### 종속성 사양(dependency specification)

`package.json`

- 시맨틱 버전 관리 또는 특정 버전 번호를 사용하여 원하는 버전 범위와 함께 프로젝트에 필요한 종속성 목록이 포함됨

`package-lock.json`

- 모든 종속성의 해결된 특정 버전, 하위 종속성 및 정확한 설치 위치가 포함됨
- 일관된 설치를 보장하기 위한 종속성 트리의 스냅샷 역할

### 버전 관리(version control)

`package.json`

- 일반적으로 Git과 같은 버전 관리 시스템에서 추적되며 프로젝트 기여자 간에 공유 구성 파일로 사용됨

`package-lock.json`

- 버전 관리 시스템에서도 역시 추적되어 여러 개발 환경에서 여러 개발 환경에서 일관된 종속성 설치를 보장함

### 수동 편집(manual editing)

`package.json`

- 개발자는 이 파일을 수동으로 편집하여
- 종속성을 추가 또는 제거하고
- 스크립트를 수정하고
- 버전 범위를 업데이트하거나
- 기타 구성 변경을 수행함

`package-lock.json`

- 일반적으로 `npm`에 의해 자동으로 관리되므로 수동으로 편집할 필요가 없음
- 이 파일을 수동으로 변경하면 종속성 해결에 불일치(inconsistencies) 또는 충돌이 발생할 수 있음

### 결론

`package.json`

- 프로젝트 메타데이터와 원하는 종속성 버전을 지정하는 데 중점을 둠

`package-lock.json`

- 정확한 종속성 버전과 해당 종속성을 잠가 결정론적 설치를 보장함

두 파일 모두 종속성을 관리하는 데 필수적이지만 `Node.js` 개발 워크플로에서 다른 용도로 사용됨

## 참고

> [Difference between package.json and package-lock.json](https://medium.com/@umarfarooquekhan/difference-between-package-json-and-package-lock-json-6225f83f247c)

> [알아두면 쓸데있는 package.json과 package-lock.json의 차이](https://dev-ellachoi.tistory.com/65)

> [Package.json과 Package-lock.json의 차이를 아시나요?](https://velog.io/@songyouhyun/Package.json과-Package-lock.json의-차이)
