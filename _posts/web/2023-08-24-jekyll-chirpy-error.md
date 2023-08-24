---
title: GitHub 블로그 Jekyll 테마 Chirpy 에러 해결하기
date: 2023-08-24 13:59:27 +0900
last_modified_at: 2023-08-24 13:59:27 +0900
categories: [GitHub]
tags: [github, git, error]
img_path: /assets/img/posts/2023/08/230824
---

`'--- layout: home # Index page ---'`, `github-pages deployments failed`, `아예 아무것도 안 뜸`, ... 해결 과정

## 전개

Minimal Mistakes를 쓰다가, 블로그를 좀 더 정리하고 꾸미고 싶어져서 새로운 테마를 쓰면서 정리를 하려고 했었습니다.

Chirpy라는 테마가 마음에 들어서 어떻게 현재 블로그에 적용하고 바꿀지 고민했었습니다..

그러다가 생각했습니다!

Jekyll에 대해서 잘 모르지만, 어차피 테마만 바꾸니까 내용물(posts 등)을 쏙 바꿔 넣으면 되겠다는 좋은 생각이 떠올랐습니다.

Chirpy 테마를 좀 보니까 제가 기존에 사용하던 Minimal Mistakes와 뭔가 폴더나 파일, 구조들이 유사했습니다.

그래서 일단 기존의 블로그 파일은 따로 로컬에 저장해놓고, 해당 repository는 `hhejo-github-io-backup` 아카이브로 만들었습니다.

### 1차 시도

기존 파일을 모두 새로운 repository에 넣어 기존 블로그 repository와 동일하게 만든 후, Chirpy 테마의 파일들을 비교하며 넣고 덮어쓰기를 진행했습니다.

실패

### 2차 시도

빈 repository만들어 clone한 후, 미리 zip 파일로 다운 받았던 Chirpy 테마의 내용물을 넣었습니다.

실패

### 3차 시도

Chirpy Starter로 처음 세팅을 할 수 있다고 해서 그것으로 진행했습니다.

실패

### 4차 시도

안 되겠다, fork 따자!

실패

### 5차, 6차, 7차, ...

계속 계속 repository 지우고, 반복되는 clone과 삭제

자정을 넘어서 새벽임

### 대표 에러들

배포는 됐는데, 화면이 이렇게 밖에 나오지 않았습니다.

![Layout Error](01-layout-error.png)

혹은 아예 배포가 안 되어서 404 에러 페이지만 만났습니다..

## 해결

이슈 검색도 해보고, 구글 검색, 성공적으로 배포된 다른 사람들의 repository 비교, ...

어떻게 해결했는지 쓰겠습니다. 무엇이 원인을 고친 것인지는 저도 자세히는 모르겠습니다.

그대로 따라하셔도 안 될 수도 있습니다..

참고로 macOS 환경에서 진행했습니다.

### 1. 블로그 repository 생성, clone

빈 repository를 생성하고, 로컬로 clone 합니다.

메인 브랜치는 `master`가 아닌 `main`으로 해주세요.

### 2. Chirpy zip 파일을 로컬 repository로

[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)

zip 파일을 다운로드 하고, 로컬의 repository 폴더에 전부 압축을 풀어 넣습니다.

### 3. **bundle**, **npm** 으로 종속성 설치

압축 파일들이 들어간 프로젝트 폴더(블로그의 레포지토리 폴더)에서 작업을 진행합니다.

```bash
bundle
```

```bash
npm i && npm build
```

추가로 아래 명령어도 진행합니다. 에러 해결에 필요한 것 같습니다.

```bash
bundle lock --add-platform x86_64-linux
```

로컬에서 블로그가 정상적으로 동작하는지 확인하려면, 아래 명령어를 사용해 브라우저에서 `127.0.0.1:4000`로 접속해 확인합니다.

```bash
bundle exec jekyll serve
```

### 4. **.gitignore** 파일 수정하기

아래 코드를 찾아 `assets/js/dist` 부분을 주석 처리합니다.

```config
# Misc
# assets/js/dist
```

macOS 사용자라면 아래 코드도 추가하면 좋습니다.

```config
.DS_Store
```

### 5. **.github** 폴더 내용 수정하기

`workflows/` 폴더를 제외하고 모든 파일을 삭제합니다.

그리고 `workflows/` 폴더 안에 `commitlint.yml`, `pages-deploy-yml.hook` 파일을 제외하고 전부 삭제합니다.

### 6. commit, push하기

현재 브랜치는 `main`이어야 합니다.

commit 후, push까지 해줍니다.

```bash
git add .
git commit -m 'first commit'
git push origin main
```

### 7. 메인 브랜치 main으로 변경하기

![Branches](02-branches.png)

### 8. Build and deployment를 GitHub Actions로 변경하고 commit 변경하기

변경한 후, 아래 Jekyll의 Configure를 클릭합니다.

![Pages](03-pages.png)

아래 화면으로 와서 Commit changes... 버튼을 클릭하고, 아무것도 수정하거나 작성할 필요 없이 다시 Commit changes 버튼을 클릭해 완료합니다.

![Commit changes](04-commit-changes.png)

`jekyll.yml` 파일이 생성되었습니다.

### 9. git pull로 브랜치 최신화

```bash
git pull origin main
```

### 10. 완료

저는 위 방법을 진행해서 정상적으로 블로그를 배포했습니다.

혹시 도움 되지 않으신다면, 아래 제가 참고했던 자료들을 걸어둘테니 필요한 부분들을 확인해보시면 됩니다.

## 느낀점

하루 종일 붙잡고 새벽까지 해도 계속 배포 실패가 떠서 힘들었는데 결국 해결해서 기분이 좋습니다.

아주 신납니다.

테마가 마음에 들어서 꼭 에러를 잡고 싶었습니다.

😄

## 참고

> [Getting Started](https://chirpy.cotes.page/posts/getting-started/)

> [At deployment (via github pages) website just shows --- layout: home # Index page ---, whereas working fine on local machine #968](https://github.com/cotes2020/jekyll-theme-chirpy/issues/968)

> [📒Github Blog 만들기-2](https://velog.io/@hashnsalt/Github-Blog-%EB%A7%8C%EB%93%A4%EA%B8%B0-2)

> [Jekyll Chirpy(v6.0.1) 테마를 활용한 Github 블로그 만들기(2023.6 기준)](<https://jjikin.com/posts/Jekyll-Chirpy-%ED%85%8C%EB%A7%88%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0(2023-6%EC%9B%94-%EA%B8%B0%EC%A4%80)/>)

> [Jekyll Chirpy 테마 사용하여 블로그 만들기](https://www.irgroup.org/posts/jekyll-chirpy/)
