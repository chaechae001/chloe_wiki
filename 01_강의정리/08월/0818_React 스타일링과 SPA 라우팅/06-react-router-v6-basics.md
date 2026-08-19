# React Router v6 기초

React Router v6에서는 `Routes`가 현재 URL과 가장 잘 맞는 `Route`의 `element`를 선택합니다.

**키워드:** BrowserRouter, Routes, Route, Link, useNavigate

## 핵심 포인트

- `BrowserRouter`가 History API 기반 라우팅 문맥을 제공합니다.
- `Routes` 안에 경로별 `Route`를 선언합니다.
- 렌더링 대상은 `element={<Page />}`로 전달합니다.
- 앱 내부 이동은 `Link`나 `NavLink`를 사용합니다.
- 코드에서 이동할 때는 `useNavigate`를 사용합니다.

## 기본 라우터 구성

```jsx
import {
  BrowserRouter, Routes, Route, Link, NavLink
} from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">홈</Link>
        <NavLink to="/projects">프로젝트</NavLink>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/projects" element={<Projects />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

목적은 URL마다 화면을 연결하고 존재하지 않는 경로를 처리하는 것입니다. `BrowserRouter`가 주소 변경을 감지하고 `Routes`가 경로를 비교해 한 element를 렌더링합니다. `Link`는 전체 새로고침 없이 이동하며 `NavLink`는 활성 상태 스타일에도 활용할 수 있습니다.

## 코드에서 이동하기

```jsx
import { useNavigate } from 'react-router-dom';

function LoginButton() {
  const navigate = useNavigate();
  const handleSuccess = () => navigate('/dashboard', { replace: true });
  return <button onClick={handleSuccess}>로그인 완료</button>;
}
```

`replace: true`는 현재 기록을 새 주소로 교체하므로 로그인 페이지로 뒤로 가는 일을 줄입니다. 일반 메뉴 이동은 선언적인 `Link`, 제출 성공처럼 로직 결과에 따른 이동은 `navigate`가 자연스럽습니다.

## v5에서 v6로 읽기

| v5 | v6 |
|---|---|
| `Switch` | `Routes` |
| `component={Page}` | `element={<Page />}` |
| `useHistory()` | `useNavigate()` |
| `history.push('/a')` | `navigate('/a')` |
| `history.replace('/a')` | `navigate('/a', { replace: true })` |
| 별도 Redirect 패턴 | `Navigate` 컴포넌트 |

## 연결해서 보기

- 라우팅 원리는 [SPA와 클라이언트 라우팅](05-spa-and-client-side-routing.md)을 참고합니다.
- 동적 URL과 보호 경로는 [동적 경로, 쿼리와 접근 제어](07-dynamic-routes-query-and-guards.md)에서 확장합니다.

## 직접 해보기

1. `/about` 경로와 링크를 추가하세요.
2. 모든 미등록 경로에 404 화면을 연결하세요.
3. 저장 성공 후 `/items`로 replace 이동하세요.

<details>
<summary>답</summary>

`<Route path="/about" element={<About />} />`, `<Link to="/about">소개</Link>`, `<Route path="*" element={<NotFound />} />`를 추가합니다. 저장 성공 콜백에서는 `navigate('/items', { replace: true })`를 호출합니다.

</details>

## 점검 목록

- [ ] 라우터 Provider가 앱을 감싼다.
- [ ] Route는 Routes 안에 있다.
- [ ] element에 JSX를 전달한다.
- [ ] 내부 이동에 일반 a 대신 Link를 쓴다.
- [ ] `*` 경로로 404를 처리한다.

## 복습 질문 및 답변

### 1. BrowserRouter의 역할은?

<details>
<summary>답</summary>

History API와 현재 위치 정보를 React 컴포넌트에 제공하고 주소 변경 시 다시 렌더링하게 합니다.

</details>

### 2. Link와 navigate는 언제 구분하는가?

<details>
<summary>답</summary>

사용자가 누르는 탐색 UI는 Link, 작업 결과에 따른 명령형 이동은 navigate를 사용합니다.

</details>

### 3. `path="*"`의 의미는?

<details>
<summary>답</summary>

앞의 경로와 매칭되지 않은 나머지 주소를 받아 404 같은 대체 화면을 보여 줍니다.

</details>

## 정리

v6 라우팅은 `BrowserRouter → Routes → Route element` 구조와 선언적 링크, `useNavigate`를 중심으로 이해하면 됩니다.
