# 동적 경로, 쿼리와 접근 제어

실무 URL은 고정된 페이지 이름뿐 아니라 자원 식별자, 필터 조건, 로그인 상태도 표현합니다.

**키워드:** useParams, useLocation, URLSearchParams, Navigate, Outlet

## 핵심 포인트

- `:id`는 URL의 일부를 경로 파라미터로 받습니다.
- `useParams`는 동적 세그먼트를 문자열로 제공합니다.
- `useLocation`과 `URLSearchParams`로 쿼리 문자열을 읽습니다.
- 보호 경로는 인증 상태에 따라 `Outlet` 또는 `Navigate`를 반환합니다.
- URL 입력값은 검증하고 로딩·실패·빈 상태를 구분합니다.

## 동적 경로와 쿼리

```jsx
<Route path="/products/:productId" element={<ProductDetail />} />
```

```jsx
import { useLocation, useParams } from 'react-router-dom';

function ProductDetail() {
  const { productId } = useParams();
  const { search } = useLocation();
  const tab = new URLSearchParams(search).get('tab') ?? 'summary';
  return <h1>상품 {productId} · {tab}</h1>;
}
```

목적은 `/products/42?tab=review`에서 자원 ID와 보기 옵션을 분리하는 것입니다. 라우터가 `42`를 params에 담고 브라우저 쿼리는 search로 제공합니다. ID는 API 조회에, tab은 화면 상태에 사용합니다. 숫자로 필요하면 변환과 유효성 검사를 먼저 합니다.

## 보호 경로

```jsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';

function RequireAuth({ signedIn }) {
  const location = useLocation();
  if (!signedIn) {
    return <Navigate to="/login" replace state={{ from: location }} />;
  }
  return <Outlet />;
}

<Route element={<RequireAuth signedIn={signedIn} />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/settings" element={<Settings />} />
</Route>
```

인증되지 않으면 로그인으로 교체 이동하고 원래 위치를 state에 보관합니다. 인증되면 `Outlet` 위치에 자식 Route가 렌더링됩니다. 이는 UI 접근을 제어할 뿐이므로 서버 API도 별도로 권한을 검증해야 합니다.

| 경로 파라미터 | 쿼리 문자열 |
|---|---|
| 자원의 정체성을 표현 | 정렬·필터·탭 같은 선택 상태 표현 |
| `/products/:id` | `/products?sort=price` |
| 보통 화면을 구분 | 같은 화면의 표시 조건을 조정 |

## 연결해서 보기

- 기본 Route 구성은 [React Router v6 기초](06-react-router-v6-basics.md)를 먼저 봅니다.
- 서버 fallback을 포함한 원리는 [SPA와 클라이언트 라우팅](05-spa-and-client-side-routing.md)을 참고합니다.

## 직접 해보기

1. `/users/:userId`에서 userId를 출력하세요.
2. `?page=2`를 읽고 1 이상의 숫자로 검증하세요.
3. `/admin`을 로그인 사용자만 보게 만드세요.

<details>
<summary>답</summary>

`useParams()`에서 `userId`를 꺼냅니다. page는 `Number(new URLSearchParams(search).get('page') ?? 1)`로 변환한 뒤 `Number.isInteger`와 범위를 검사합니다. 보호 경로 부모가 미인증 시 `Navigate`, 인증 시 `Outlet`을 반환하게 합니다.

</details>

## 점검 목록

- [ ] params가 문자열임을 고려했다.
- [ ] 쿼리의 기본값과 유효성을 검사했다.
- [ ] 미존재 자원의 오류 화면이 있다.
- [ ] 로그인 후 원래 경로 복귀를 고려했다.
- [ ] 서버에서도 권한을 검증한다.

## 복습 질문 및 답변

### 1. params와 query의 설계 기준은?

<details>
<summary>답</summary>

화면이나 자원의 정체성은 params, 선택적인 필터와 보기 상태는 query에 두는 것이 일반적입니다.

</details>

### 2. Navigate의 replace를 쓰는 이유는?

<details>
<summary>답</summary>

접근할 수 없는 보호 URL을 방문 기록에 남겨 뒤로 가기 반복이 생기는 것을 줄입니다.

</details>

### 3. 프런트 보호 경로만으로 보안이 완성되는가?

<details>
<summary>답</summary>

아닙니다. 화면 제어일 뿐이며 데이터와 작업 권한은 서버가 반드시 다시 검증해야 합니다.

</details>

## 정리

동적 경로와 쿼리는 URL을 공유 가능한 상태로 만들고, 보호 경로는 인증 흐름을 선언적으로 구성합니다.
