# SPA와 클라이언트 라우팅

SPA는 페이지 전체를 다시 받는 대신 하나의 앱 안에서 URL과 화면 컴포넌트를 교체합니다.

**키워드:** SPA, MPA, History API, client-side routing, server fallback

## 핵심 포인트

- SPA는 초기 문서 이후 필요한 데이터와 코드로 화면을 갱신합니다.
- 라우터는 현재 URL을 읽고 대응하는 컴포넌트를 렌더링합니다.
- History API는 새 문서 요청 없이 주소와 방문 기록을 바꿉니다.
- 직접 URL 접근을 위해 서버의 SPA fallback 설정이 필요합니다.
- 초기 번들, SEO, 접근성, 로딩·오류 상태를 함께 설계해야 합니다.

## SPA의 동작 흐름

1. 브라우저가 HTML, CSS, JavaScript를 받습니다.
2. JavaScript 앱이 실행되어 현재 URL에 맞는 화면을 그립니다.
3. 링크 클릭 시 라우터가 기본 문서 이동을 가로챕니다.
4. 주소를 갱신하고 필요한 컴포넌트와 데이터를 로드합니다.
5. 뒤로 가기 이벤트에서도 URL에 맞춰 다시 렌더링합니다.

```js
history.pushState({}, '', '/products/42');
window.addEventListener('popstate', renderFromLocation);
```

이 코드는 라우터의 핵심 원리를 단순화한 예입니다. `pushState`는 새 요청 없이 주소를 바꾸고, `popstate`는 방문 기록 이동을 감지합니다. 실제 앱에서는 React Router가 경로 매칭과 렌더링을 담당합니다.

## 서버 fallback이 필요한 이유

앱 내부에서 `/products/42`로 이동할 때는 이미 JavaScript가 실행 중입니다. 그러나 같은 주소를 새로고침하면 서버가 그 경로의 파일을 찾습니다. 서버가 모든 앱 경로에 진입 HTML을 돌려주도록 설정해야 라우터가 이어서 화면을 그릴 수 있습니다.

## SPA의 장단점

| SPA | MPA |
|---|---|
| 앱 내부 이동이 부드럽고 UI 재사용이 쉬움 | 요청마다 서버가 새 문서를 반환 |
| 초기 JavaScript가 클 수 있음 | 페이지별 초기 자원이 비교적 분리됨 |
| 클라이언트 라우팅·상태 관리 필요 | 문서 단위 라우팅이 단순함 |
| 렌더링 전략에 따라 SEO 보완 필요 | 서버 HTML이 검색 엔진에 직접 노출 |

코드 분할, 캐싱, CDN, 서버 렌더링 또는 정적 생성을 프로젝트 요구에 맞게 조합할 수 있습니다.

## 연결해서 보기

- 실제 경로 선언은 [React Router v6 기초](06-react-router-v6-basics.md)에서 다룹니다.
- 파라미터와 권한은 [동적 경로, 쿼리와 접근 제어](07-dynamic-routes-query-and-guards.md)로 이어집니다.

## 직접 해보기

1. SPA에서 링크 클릭부터 화면 교체까지 흐름을 적으세요.
2. 새로고침 시 404가 나는 이유를 설명하세요.
3. 초기 로딩 비용을 줄이는 방법 두 가지를 제시하세요.

<details>
<summary>답</summary>

라우터가 클릭을 처리해 History API로 주소를 바꾸고 대응 컴포넌트를 렌더링합니다. 새로고침은 서버 요청이므로 fallback이 없으면 파일을 찾지 못합니다. 코드 분할, 압축, 캐싱, CDN 등을 적용할 수 있습니다.

</details>

## 점검 목록

- [ ] SPA와 MPA의 요청 차이를 설명한다.
- [ ] URL과 화면 상태가 동기화된다.
- [ ] 뒤로 가기 동작을 검증했다.
- [ ] 서버 fallback을 설정했다.
- [ ] 로딩·오류·404 화면을 준비했다.

## 복습 질문 및 답변

### 1. AJAX가 SPA에서 하는 역할은?

<details>
<summary>답</summary>

문서 전체를 다시 받지 않고 필요한 데이터를 비동기로 받아 화면 일부를 갱신하게 합니다.

</details>

### 2. History API를 쓰는 이유는?

<details>
<summary>답</summary>

페이지를 다시 로드하지 않고도 주소와 브라우저 방문 기록을 관리하기 위해서입니다.

</details>

### 3. SPA의 초기 로딩이 길어질 수 있는 이유는?

<details>
<summary>답</summary>

첫 화면 실행에 필요한 JavaScript 번들이 커질 수 있기 때문입니다.

</details>

## 정리

SPA 라우팅은 URL을 애플리케이션 상태의 일부로 다루며, 클라이언트와 서버 설정이 함께 맞아야 안정적으로 동작합니다.
