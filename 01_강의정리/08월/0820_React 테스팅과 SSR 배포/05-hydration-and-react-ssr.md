# Hydration과 React SSR

> SSR 화면은 HTML을 받는 것으로 끝나지 않고, 같은 React 트리가 이벤트를 이어받아야 완성됩니다.

`ReactDOMServer` · `renderToString` · `Streaming` · `Hydration` · `Markup Mismatch`

## 핵심요약

- 서버는 React 트리를 초기 HTML로 변환해 응답할 수 있습니다.
- 스트리밍은 완성된 HTML 조각부터 점진적으로 전달합니다.
- Hydration은 기존 HTML에 React의 상태와 이벤트 처리를 연결합니다.
- 서버와 클라이언트의 첫 렌더 결과가 다르면 불일치가 발생합니다.
- 브라우저 전용 API와 `useEffect` 데이터 로딩은 서버에서 별도 처리가 필요합니다.

## 1. 서버 렌더링 흐름

```text
요청 → 서버에서 경로·데이터 준비 → React HTML 생성 → 브라우저 표시
     → JavaScript 로드 → Hydration → 상호작용 가능
```

초기 HTML에는 제목과 본문 같은 콘텐츠가 포함될 수 있지만 클릭 핸들러는 HTML 문자열만으로 실행되지 않습니다. 클라이언트 번들이 로드되어 같은 컴포넌트 트리를 Hydration해야 합니다.

## 2. 서버와 클라이언트 결과 맞추기

```jsx
function Greeting({ initialName }) {
  const [name, setName] = useState(initialName);
  return <button onClick={() => setName('Visitor')}>Hello, {name}</button>;
}
```

```javascript
// server
const html = renderToString(<Greeting initialName="Guest" />);

// client
hydrateRoot(document.getElementById('root'), <Greeting initialName="Guest" />);
```

### 코드 목적

서버가 만든 버튼 마크업과 클라이언트의 첫 렌더 입력을 동일하게 유지합니다.

### 코드 흐름과 결과 해석

서버가 `Guest`로 HTML을 만들고 클라이언트도 같은 값으로 시작합니다. Hydration 뒤 클릭 이벤트가 연결되어 state가 바뀝니다. 두 초기 값이 다르면 텍스트 불일치 경고와 재렌더 비용이 생길 수 있습니다.

### 실무 연결

현재 시간, 난수, 브라우저 저장소, 화면 크기처럼 서버와 브라우저에서 다른 값은 첫 렌더에 바로 사용하지 않거나 서버에서 전달한 초기값을 공유합니다.

## 3. 데이터와 브라우저 API

`useEffect`는 서버 렌더 단계에서 실행되지 않습니다. 초기 콘텐츠에 필요한 데이터는 서버 렌더 전에 준비해 props나 프레임워크의 데이터 계층으로 전달합니다. `window`, `document`, `localStorage`는 브라우저에서만 존재하므로 Effect 또는 환경 검사 뒤 접근합니다.

## 4. 문자열 렌더링과 스트리밍

문자열 렌더링은 전체 결과가 준비된 뒤 응답하기 쉽고, 스트리밍은 준비된 부분부터 보내 브라우저가 점진적으로 처리하게 합니다. 스트리밍의 이점은 데이터 경계, 오류 처리, 캐시와 함께 설계해야 합니다.

## 직접 해보기

1. 서버에서 `Date.now()`를 바로 렌더링할 때 생길 수 있는 문제를 설명하세요.
2. 첫 화면 데이터 요청을 useEffect에만 두면 SSR HTML에 어떤 영향이 있는지 말하세요.
3. 브라우저 저장소 값을 안전하게 읽는 시점을 제안하세요.

<details>
<summary>답</summary>

1. 클라이언트 첫 렌더 시각과 달라 Hydration 불일치가 생길 수 있습니다.
2. Effect가 서버에서 실행되지 않아 초기 HTML에 데이터 콘텐츠가 없을 수 있습니다.
3. Hydration 이후 Effect에서 읽거나 서버가 제공한 안전한 초기값으로 먼저 렌더링합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Rendering vs Hydration | HTML 생성 vs 기존 HTML에 React 동작 연결 |
| `renderToString` vs streaming | 전체 문자열 완료 후 전달 vs 준비된 조각부터 전달 |
| Server data vs Effect data | 초기 HTML 전에 준비 가능 vs 브라우저 렌더 후 실행 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [CSR과 SSR 성능](04-csr-ssr-performance.md)
- 다음에 이어지는 개념: [React 프로덕션 빌드](06-react-production-build.md)

## 셀프 체크

- [ ] 서버 렌더와 Hydration을 구분한다.
- [ ] 초기 마크업 일치 조건을 안다.
- [ ] 브라우저 전용 API를 분리한다.
- [ ] 초기 데이터 준비 위치를 판단한다.
- [ ] 스트리밍의 목적을 설명한다.

### 복습 질문 및 답변

**Q1. HTML이 보이면 Hydration도 끝난 것인가?**

<details>
<summary>답</summary>

아닙니다. 콘텐츠는 보이지만 JavaScript와 이벤트 핸들러가 아직 연결되지 않았을 수 있습니다.

</details>

**Q2. useEffect가 서버에서 실행되지 않는 이유는?**

<details>
<summary>답</summary>

Effect는 DOM 렌더 이후의 클라이언트 부수 효과를 위한 단계이며 서버에는 브라우저 DOM 커밋이 없기 때문입니다.

</details>

**Q3. Hydration 경고를 무시하면 안 되는 이유는?**

<details>
<summary>답</summary>

서버와 클라이언트 UI가 다르다는 신호로 잘못된 화면, 추가 렌더링 또는 이벤트 연결 문제를 숨길 수 있습니다.

</details>

## 한 줄 정리

> React SSR은 서버 HTML과 클라이언트의 첫 트리를 일치시키고 Hydration으로 상호작용을 연결하는 과정입니다.
