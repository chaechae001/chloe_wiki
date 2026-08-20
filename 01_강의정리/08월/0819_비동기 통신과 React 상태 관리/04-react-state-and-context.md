# React 상태 관리와 Context

> 상태 관리 도구의 목표는 모든 값을 전역으로 옮기는 것이 아니라, 값을 사용하는 범위와 변경 책임을 분명히 하는 것입니다.

`useState` · `useRef` · `Context` · `Prop Drilling` · `캐싱`

## 핵심요약

- 상태는 시간에 따라 변하며 UI 출력에 영향을 주는 데이터입니다.
- 상태는 가능한 한 사용하는 컴포넌트 가까이에 둡니다.
- `useRef`는 값이 바뀌어도 렌더링이 필요 없는 데이터를 보관합니다.
- Context는 깊은 트리에서 공통 값을 직접 읽게 해 Prop Drilling을 줄입니다.
- 캐시와 전역 상태는 편리하지만 무효화와 리렌더링 범위를 설계해야 합니다.

## 1. 상태 범위 정하기

한 입력창만 사용하는 문자열은 로컬 `useState`가 자연스럽습니다. 여러 형제 컴포넌트가 같은 값을 사용하면 공통 부모로 끌어올립니다. 앱 전반의 테마나 인증 정보처럼 깊은 트리에서 공유하면 Context를 검토합니다.

```javascript
function SearchBox() {
  const [keyword, setKeyword] = useState('');
  const timerRef = useRef(null);

  function handleChange(event) {
    const value = event.target.value;
    setKeyword(value);
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => search(value), 400);
  }

  return <input value={keyword} onChange={handleChange} />;
}
```

`keyword`는 화면에 표시되므로 state이고, timer ID는 바뀌어도 UI를 다시 그릴 이유가 없으므로 ref입니다. 이 구분은 불필요한 렌더링을 줄입니다.

## 2. Context로 깊은 전달 줄이기

```javascript
const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = { theme, setTheme };
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>전환</button>;
}
```

### 코드 목적

중간 컴포넌트가 사용하지 않는 props를 계속 전달하지 않고 필요한 자식이 테마를 직접 읽게 합니다.

### 코드 흐름과 결과 해석

Provider가 공유 값을 정하고 소비자가 `useContext`로 읽습니다. value가 바뀌면 해당 Context를 구독하는 소비자가 다시 렌더링됩니다.

### 실무 연결

테마, 언어, 인증 세션처럼 넓게 읽히는 값에 유용합니다. 자주 바뀌는 거대한 객체 하나를 Context에 넣으면 많은 소비자가 함께 렌더링될 수 있으므로 책임별로 나누는 편이 좋습니다.

## 3. 캐싱과 서버 상태

SPA는 페이지 이동마다 같은 데이터를 다시 요청하지 않도록 캐싱할 수 있습니다. 다만 언제 오래된 것으로 판단할지, 저장 성공 후 어느 캐시를 갱신할지, 오류와 재시도를 어떻게 보여 줄지를 함께 설계해야 합니다.

## 직접 해보기

1. input 값과 timer ID를 각각 state와 ref 중 어디에 둘지 고르세요.
2. 네 단계 아래 컴포넌트만 쓰는 테마 값을 전달하는 방식을 제안하세요.
3. 게시글 캐시가 오래됐는지 판단할 기준을 설계하세요.

<details>
<summary>답</summary>

1. input 값은 state, timer ID는 ref가 적합합니다.
2. 해당 트리를 감싸는 Context Provider를 두고 소비자가 직접 읽게 할 수 있습니다.
3. 마지막 조회 시각, 사용자의 재진입, 저장 성공 이벤트 등을 무효화 기준으로 사용할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| state vs ref | 변경 시 렌더링 필요 vs 렌더링 없이 값 유지 |
| props vs Context | 명시적 부모-자식 전달 vs 트리 범위의 공통 값 접근 |
| 클라이언트 상태 vs 서버 상태 | UI 내부 값 vs 원본이 서버에 있고 동기화가 필요한 값 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [HTTP API와 CORS](03-http-api-openapi-and-cors.md)
- 다음에 이어지는 개념: [Flux와 useReducer](05-flux-and-usereducer.md)

## 셀프 체크

- [ ] 상태를 사용하는 최소 범위를 찾는다.
- [ ] state와 ref를 구분한다.
- [ ] Prop Drilling의 의미를 설명한다.
- [ ] Context 변경의 렌더링 범위를 고려한다.
- [ ] 캐시 무효화 시점을 설계할 수 있다.

### 복습 질문 및 답변

**Q1. 모든 상태를 Context로 옮기면 좋은가?**

<details>
<summary>답</summary>

아닙니다. 로컬 상태까지 넓게 공유하면 변경 범위와 의존성이 커질 수 있습니다.

</details>

**Q2. useRef 값 변경이 화면을 자동으로 갱신하는가?**

<details>
<summary>답</summary>

아닙니다. ref 변경은 리렌더링을 예약하지 않습니다.

</details>

**Q3. Context가 Prop Drilling을 줄이지만 비용이 생기는 이유는?**

<details>
<summary>답</summary>

공유 값 변경 시 이를 구독하는 소비자들이 함께 렌더링될 수 있고 데이터 출처가 props보다 덜 명시적으로 보일 수 있기 때문입니다.

</details>

## 한 줄 정리

> React 상태는 가장 가까운 곳에 두고, 공유 범위가 커질 때 Context와 캐시를 목적에 맞게 도입합니다.
