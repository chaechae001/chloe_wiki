# Testing Library와 사용자 이벤트

> 컴포넌트 테스트는 state 이름보다 사용자가 보고 누르고 입력하는 결과를 기준으로 작성할수록 오래 유지됩니다.

`React Testing Library` · `ByRole` · `findBy` · `queryBy` · `userEvent`

## 핵심요약

- Testing Library는 렌더링 결과인 DOM을 사용자 관점에서 검증합니다.
- 접근 가능한 role과 name을 이용하는 쿼리를 우선합니다.
- `getBy`, `findBy`, `queryBy`는 존재 시점과 실패 방식이 다릅니다.
- `userEvent`는 클릭과 입력을 실제 상호작용에 가깝게 재현합니다.
- 테스트하기 쉬운 마크업은 대체로 접근성도 좋습니다.

## 1. 쿼리 선택 기준

| 쿼리 | 찾지 못했을 때 | 대표 상황 |
|---|---|---|
| `getBy*` | 즉시 오류 | 지금 반드시 존재해야 함 |
| `findBy*` | 일정 시간 기다린 뒤 reject | 비동기로 나타남 |
| `queryBy*` | `null` 반환 | 존재하지 않음을 검증 |

우선순위는 `ByRole`과 accessible name, label·text, alt·title 순으로 생각합니다. `data-testid`는 사용자가 인식하는 정보가 없을 때 마지막 수단으로 사용합니다.

## 2. 사용자 흐름 테스트

```jsx
function NewsletterForm({ subscribe }) {
  const [email, setEmail] = useState('');
  const [done, setDone] = useState(false);

  async function handleSubmit(event) {
    event.preventDefault();
    await subscribe(email);
    setDone(true);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Subscribe</button>
      {done && <p>Subscribed</p>}
    </form>
  );
}
```

```javascript
test('이메일을 입력해 구독한다', async () => {
  const user = userEvent.setup();
  const subscribe = jest.fn().mockResolvedValue(undefined);
  render(<NewsletterForm subscribe={subscribe} />);

  await user.type(screen.getByRole('textbox', { name: 'Email' }), 'a@example.com');
  await user.click(screen.getByRole('button', { name: 'Subscribe' }));

  expect(subscribe).toHaveBeenCalledWith('a@example.com');
  expect(await screen.findByText('Subscribed')).toBeInTheDocument();
});
```

### 코드 목적

사용자가 입력하고 제출했을 때 외부 함수와 성공 UI가 연결되는지 검증합니다.

### 코드 흐름과 결과 해석

Role과 label로 요소를 찾고 실제 입력·클릭 이벤트를 발생시킵니다. 비동기 완료 후 나타나는 문구는 `findByText`로 기다립니다. 내부 state에는 직접 접근하지 않습니다.

### 실무 연결

폼, 모달, 검색, 필터처럼 사용자의 행동과 화면 피드백이 중요한 기능의 통합 테스트에 적합합니다.

## 직접 해보기

1. 즉시 존재하는 제출 버튼에 사용할 쿼리를 고르세요.
2. 로딩 뒤 나타나는 결과 제목을 어떻게 찾을지 적으세요.
3. 오류 문구가 초기에는 없음을 검증하세요.

<details>
<summary>답</summary>

1. `getByRole('button', { name: ... })`을 사용합니다.
2. `await screen.findByRole('heading', { name: ... })`처럼 기다립니다.
3. `expect(screen.queryByText(...)).not.toBeInTheDocument()`로 확인합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `getBy` vs `findBy` | 즉시 검색 vs 비동기 등장 대기 |
| `queryBy` vs `getBy` | 없음 검증 가능 vs 없으면 즉시 오류 |
| `fireEvent` vs `userEvent` | 단일 이벤트 발생 vs 사용자 상호작용에 가까운 연속 동작 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [Jest 핵심 기능](02-jest-core-features.md)
- 다음에 이어지는 개념: [CSR과 SSR 성능](04-csr-ssr-performance.md)

## 셀프 체크

- [ ] 쿼리 세 종류를 구분한다.
- [ ] ByRole을 우선해 요소를 찾는다.
- [ ] 비동기 DOM 변화를 기다린다.
- [ ] userEvent를 await해 사용한다.
- [ ] 구현이 아닌 사용자 결과를 검증한다.

### 복습 질문 및 답변

**Q1. ByRole 테스트가 접근성과 연결되는 이유는?**

<details>
<summary>답</summary>

접근성 트리의 역할과 이름을 기준으로 찾기 때문에 올바른 시맨틱 HTML과 label이 필요합니다.

</details>

**Q2. 요소가 없어야 하는 테스트에 getBy를 쓰면 어떤 일이 생기는가?**

<details>
<summary>답</summary>

Assertion까지 도달하기 전에 쿼리 자체가 오류를 던집니다.

</details>

**Q3. 컴포넌트의 내부 state를 직접 확인하지 않는 이유는?**

<details>
<summary>답</summary>

사용자에게 보이는 동작이 같다면 내부 구현을 바꿔도 테스트가 유지되도록 하기 위해서입니다.

</details>

## 한 줄 정리

> Testing Library는 접근 가능한 DOM과 사용자 이벤트를 중심으로 컴포넌트의 실제 사용 계약을 검증합니다.
