---
title: React Router 시작하기
date: 2024-01-01 21:18:07 +0900
last_modified_at: 2024-01-03 14:05:29 +0900
categories: [React]
tags: [react, react-router]
---

React Router의 초기 설정과 BrowserRouter, RouterProvider, useRouteError, Outlet, Link, useParams, 그리고 Routing Layer

## React Router Dom

Single-Page React 앱에서, 라우팅은 전체 페이지를 다시 로드하지 않고 여러 페이지들 사이를 탐색하는 프로세스를 의미

React는 SPA(Single Page Application)

- 렌더링을 SSR과 달리 클라이언트(브라우저)에서 처리
- 모든 컴포넌트의 변화가 한 페이지 안에서 발
- 다른 URL로 이동하는 것은 페이지를 교체하는 것이 아니라, 한 페이지 내부의 컴포넌트만 변경하는 것
- URL은 고정됨
  - 페이지 즐겨찾기, 뒤로가기, 새로고침 등 불가
  - SEO 문제
- React Router를 이용해 해결 가능
  - 사용자 경험 향상

```bash
npm install react-router-dom
```

## Adding a Router

`createBrowserRouter`

- DOM History API를 사용해 URL을 업데이트하고 history 스택을 관리
- v6.4 데이터 API들을 활성화함
  - loaders, actions, fetchers

`RouterProvider`

- 모든 데이터 라우터 객체는 이 컴포넌트에 전달되어 앱을 렌더링하고
- 나머지 데이터 API를 활성화
- `fallbackElement`로 사용자에게 앱이 작동 중이라는 표시 제공

```javascript
// src/main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { useBrowserRouter, RouterProvider } from "react-router-dom";
import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";
import ErrorPage from "./error-page";

const router = createBrowserRouter([
  {
    path: "/", // 루트 경로. 루트 레이아웃 역할
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    children: [{ path: "team", element: <Team />, loader: teamLoader }] // 하위 경로
  }
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} fallbackElement={<SpinnerOfDoom />} />
);
```

`useRouteError`

- `errorElement` 내부에서, action, loader, rendering 중에 발생한 모든 것을 반환하는 훅
- 던져진 응답은 특별하게 처리됨

`errorElement`

- 지정된 URL과 일치하는 경로가 없을 때 렌더링됨

```javascript
// src/error-page.jsx
import { useRouteError } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();

  return (
    <div id="error-page">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}
export default ErrorPage;
```

```javascript
import { useRouteError } from "react-router-dom";
function ErrorBoundary() {
  const error = useRouteError();
  console.error(error);
  return <div id="error-page">{error.statusText || error.message}</div>;
}

<Route
  errorElement={<ErrorBoundary />}
  loader={() => {
    something.that.breaks(); // unexpected errors in loaders/actions
  }}
  action={() => {
    throw new Response("Bad Request", { status: 400 }); // stuff you throw on purpose in loaders/actions
  }}
  element={
    <div>{breaks.while.rendering}</div> // and errors thrown while rendering
  }
/>;
```

## Nested Routes

`Outlet`

- root route의 child로 만들어 중첩
- `Outlet`을 이용해 해당 위치에 렌더링

```javascript
import { Outlet } from "react-router-dom";
```

```javascript
<div id="detail">
  <Outlet />
</div>
```

## Client Side Routing

`Link`

- 앱이 서버에 다른 문서를 요청하지 않고 URL을 업데이트

```javascript
import { Link } from "react-router-dom";
```

```javascript
<Link to={`contacts/1`}>클릭해서 넘어가기</Link>
```

##

useParams

## Routing Layer 구현하기

Routing Layer

- 경로를 관리하고, 적절한 페이지 구성 요소를 렌더링하는 FE React 앱의 일부
- 사용자가 링크를 클릭하거나 주소 표시줄에 URL을 입력하면,
- 라우팅 계층이 요청을 가로채고
- 현재 경로를 기반으로 어떤 구성 요소나 보기를 렌더링해야 하는지 결정

PathConstants

- 모든 페이지 경로를 포함하는 객체

```javascript
const pathConstants = {
  TEAM: "/team",
  REPORT_ANALYSIS: "reports/:reportId/analysis"
  // ...
};
```

routes

- 경로 문자열과 페이지 구성 요소 간의 매핑을 포함하는 배열

```javascript
const routes = [
  { path: PathConstants.TEAM, element: <TeamPage /> },
  { path: PathConstants.REPORT_ANALYSIS, element: <ReportAnalysisPage /> }
  // ...
];
```

```javascript
<Link to={PathConstants.TEAM}>팀 페이지로 이동</Link>
```

## 참고

> [Tutorial](https://reactrouter.com/en/main/start/tutorial)

> [How to Build a Routing Layer in React and Why You Need It](https://semaphoreci.com/blog/routing-layer-react)
