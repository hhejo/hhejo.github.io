---
title: React에서 Firebase 사용하기
date: 2024-03-28 19:27:33 +0900
last_modified_at: 2024-04-06 14:55:29 +0900
categories: [React]
tags: [react, firebase, authentication, firestore]
---

React와 Firebase를 어떻게 연결해 사용할 수 있을까요? Authentication 방법과 Firestore 사용 방법까지 간단히 코드와 함께 알아보겠습니다.

## React에서 Firebase 사용하기

### 초기 설정 순서

1. Firebase에서 프로젝트를 생성
2. 프로젝트 이름을 작성
3. 구글 애널리틱스 사용 설정 (맘대로)
4. 구글 애널리틱스 계정 선택 (맘대로)
5. 프로젝트 만들기
6. 프로젝트가 준비되면 프로젝트 개요 페이지로 이동
7. 앱을 추가하여 시작하기 - 웹 선택
8. 앱 닉네임을 작성
9. 앱 등록하기
10. Firebase SDK 추가하기

```bash
npm install firebase
```

- 설치해줍니다.

```javascript
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "?????????????????????-?????????????????",
  authDomain: "test-test.firebaseapp.com",
  projectId: "test-test",
  storageBucket: "test-test.appspot.com",
  messagingSenderId: "????????????",
  appId: "?:????????????:web:??????????????????????",
  measurementId: "?-??????????"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
```

- 위와 비슷한 코드를 Firebase에서 알려줍니다.
- 저는 React 프로젝트 루트 폴더에 `firebase.js` 파일을 생성하고 거기에 코드를 작성했습니다.

### React에 연결하기

```javascript
// Firebase
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";
// import { getAnalytics } from "firebase/analytics";

// firebase 설정
const firebaseConfig = {
  apiKey: process.env.REACT_APP_apiKey,
  authDomain: process.env.REACT_APP_authDomain,
  projectId: process.env.REACT_APP_projectId,
  storageBucket: process.env.REACT_APP_storageBucket,
  messagingSenderId: process.env.REACT_APP_messagingSenderId,
  appId: process.env.REACT_APP_appId,
  measurementId: process.env.REACT_APP_measurementId
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app); // auth
export const firestore = getFirestore(app); // firestore
// const analytics = getAnalytics(app);
```

- 위 코드와 같이 작성했습니다.
- `.env` 파일에 환경변수를 작성하고, `.gitignore`에 명시해 업로드를 방지했습니다.
- React 프로젝트를 CRA를 사용해서 만들었기 때문에, 환경변수 이름은 `REACT_APP_`으로 시작합니다.

### React에서 사용하기

Firebase Authentication 사용 예시 코드입니다.

```javascript
import { signInWithEmailAndPassword } from "firebase/auth";
import { auth } from "../../firebase";
import { setCurrentUser } from "../../redux/user-slice";

const Login = () => {
  const navigate = useNavigate();

  const loginHandler = async (loginData) => {
    const { email, password: pw } = loginData;
    try {
      const { user } = await signInWithEmailAndPassword(auth, email, pw); // Firebase Authentication
      dispatch(setCurrentUser(user));
      navigate("/teams", { replace: true });
    } catch (error) {
      const { code, message } = error;
      if (code === "auth/invalid-credential")
        alertToast(toastType.warning, "유효하지 않은 이메일이나 비밀번호");
      else alertToast(toastType.error, "로그인 오류");
      console.error(message);
    }
  };

  return (
    <main>
      {/* 로그인 폼 */}
      <LoginForm login={loginHandler} />
      {/* 회원가입하기 버튼 */}
      <Link to="/signup">회원가입</Link>
    </main>
  );
};

export default Login;
```

Firestore 사용 예시 코드입니다.

```javascript
// Firebase
import { collection, doc, addDoc, updateDoc } from "firebase/firestore";
import { arrayUnion } from "firebase/firestore";
import { firestore } from "../../firebase";
import { setCurrentUser } from "../../redux/user-slice";

// 예시 코드 일부..

// 팀 생성 핸들러
const createTeamHandler = async (teamData) => {
  dispatch(startLoading());
  try {
    const extra = { leaderDocId: docId, teammates: [], projectDocId: null };
    const teamToAdd = { ...teamData, ...extra }; // 생성할 팀: teamName, projectType, teamGit, leaderDocId, teammates, projectDocId
    const teamsColRef = collection(firestore, "teams");
    const { id: teamDocId } = await addDoc(teamsColRef, teamToAdd); // 1. firestore의 teams 컬렉션에 생성된 팀 추가
    const updateTeamsField = { teams: arrayUnion(teamDocId) };
    await updateDoc(doc(firestore, "users", docId), updateTeamsField); // 2. firestore의 users 컬렉션에서 현재 로그인한 유저의 teams 필드에 생성된 팀 docId 추가
    const ProjectsColRef = collection(firestore, "projects");
    const projectToAdd = { teamDocId, directory: dummy, setting: {} }; // 생성할 프로젝트: teamDocId, directory, setting
    const { id: projectDocId } = await addDoc(ProjectsColRef, projectToAdd); // 3. firestore의 projects 컬렉션에 생성된 프로젝트 추가
    const updatePjtDocIdField = { projectDocId };
    await updateDoc(doc(firestore, "teams", teamDocId), updatePjtDocIdField); // 4. firestore의 teams 컬렉션에서 현재 생성된 팀의 projectDocId 필드를 생성된 프로젝트 projectDocId로 갱신
    alertToast(toastType.success, "팀 생성 완료");
    navigate("/teams", { replace: true });
  } catch (error) {
    console.error(error);
  } finally {
    dispatch(stopLoading());
  }
};
```

### Firebase Authentication

회원가입, 로그인 기능은 Firebase Authentication을 사용해 구현할 수 있습니다.

회원가입한 유저 정보는 Firestore에 `users` 컬렉션을 생성하고 유저의 도큐먼트를 추가해 구현했습니다.

각 컬렉션의 키는 `docId`로 표현되고, 값이 객체의 형태로 구성됩니다.

## 참고

> [JavaScript 프로젝트에 Firebase 추가](https://firebase.google.com/docs/web/setup?hl=ko&_gl=1*1p8kgst*_up*MQ..*_ga*MTkwMjQzMDE2MS4xNzEyMzgyODQy*_ga_CW55HF8NVT*MTcxMjM4Mjg0Mi4xLjAuMTcxMjM4Mjg0Mi4wLjAuMA..)
