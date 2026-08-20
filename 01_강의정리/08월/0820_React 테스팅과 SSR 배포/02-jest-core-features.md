# Jest 핵심 기능

> Jest는 테스트 실행, 값 비교, 비동기 검증, Mock과 생명주기 관리를 한 환경에서 제공합니다.

`Matcher` · `Async Assertion` · `Mock Function` · `Lifecycle` · `Snapshot`

## 핵심요약

- Matcher는 결과의 값과 구조, 포함 여부, 예외를 표현합니다.
- 객체와 배열의 내용 비교에는 참조 비교가 아닌 깊은 비교가 필요합니다.
- 비동기 테스트는 Promise를 반환하거나 await해 종료 시점을 알려야 합니다.
- Mock 함수는 반환값과 호출 여부·인자를 검증할 수 있습니다.
- Snapshot은 의도한 UI 변경인지 리뷰할 때만 갱신합니다.

## 1. Matcher 선택

```javascript
test('상품 정보를 만든다', () => {
  const product = { id: 1, name: 'Keyboard', tags: ['device'] };
  expect(product).toEqual({ id: 1, name: 'Keyboard', tags: ['device'] });
  expect(product.tags).toContain('device');
  expect(product.name).toMatch(/key/i);
});
```

`toBe`는 원시값 또는 같은 참조 비교에, `toEqual`은 객체와 배열의 구조·값 비교에 적합합니다. 실패 메시지가 테스트 의도를 잘 설명하도록 가장 구체적인 Matcher를 고릅니다.

## 2. 비동기 테스트

```javascript
test('사용자 이름을 불러온다', async () => {
  await expect(fetchUserName(3)).resolves.toBe('Mina');
});

test('없는 사용자는 실패한다', async () => {
  await expect(fetchUserName(-1)).rejects.toThrow('not found');
});
```

Promise를 반환하거나 `await`하지 않으면 테스트 함수가 먼저 끝나 잘못 통과할 수 있습니다. Callback API라면 `done`을 호출하되, 가능한 경우 Promise 형태로 감싸는 편이 읽기 쉽습니다.

## 3. Mock과 생명주기

```javascript
const notify = jest.fn();

beforeEach(() => notify.mockClear());

test('저장 성공을 알린다', () => {
  saveDraft({ title: 'note' }, notify);
  expect(notify).toHaveBeenCalledWith('saved');
  expect(notify).toHaveBeenCalledTimes(1);
});
```

### 코드 목적

실제 알림 UI 대신 Mock을 전달해 호출 계약만 검증합니다.

### 코드 흐름과 결과 해석

각 테스트 전에 이전 호출 기록을 지우고 함수를 실행합니다. 특정 인자와 호출 횟수가 맞으면 기능 간 연결이 의도대로 동작한 것입니다.

### 실무 연결

API 모듈, 로그, 시간, 랜덤 값처럼 테스트 결과를 불안정하게 만드는 경계를 Mock으로 제어합니다. 모든 내부 함수를 Mock하면 실제 통합 동작을 잃으므로 외부 경계 중심으로 사용합니다.

## 4. Snapshot과 그룹화

`describe`는 관련 테스트를 논리적으로 묶고 `beforeAll`, `beforeEach`, `afterEach`, `afterAll`은 준비·정리를 담당합니다. Snapshot은 큰 마크업을 무심코 승인하기보다 중요한 출력의 의도된 변경을 리뷰하는 보조 수단으로 씁니다.

## 직접 해보기

1. 배열 `[1, 2]`의 값을 비교할 Matcher를 고르세요.
2. Promise 테스트에서 await를 빠뜨리면 생기는 문제를 설명하세요.
3. 네트워크 모듈을 Mock할 때 성공과 실패 두 응답을 설계하세요.

<details>
<summary>답</summary>

1. 새 배열과 내용까지 비교하려면 `toEqual([1, 2])`이 적합합니다.
2. Assertion 전에 테스트가 종료되어 거짓 양성이 생길 수 있습니다.
3. `mockResolvedValueOnce`와 `mockRejectedValueOnce` 등으로 두 경로를 각각 검증합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `toBe` vs `toEqual` | 동일성·참조 비교 vs 깊은 값 비교 |
| `beforeAll` vs `beforeEach` | 그룹당 한 번 vs 각 테스트마다 실행 |
| Mock reset vs clear | 구현과 상태까지 초기화 가능 vs 호출 기록 중심 제거 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [React 테스트 전략](01-testing-strategy.md)
- 다음에 이어지는 개념: [Testing Library와 사용자 이벤트](03-testing-library-user-events.md)

## 셀프 체크

- [ ] 값에 맞는 Matcher를 선택한다.
- [ ] 비동기 테스트 종료를 보장한다.
- [ ] Mock 호출 인자를 검증한다.
- [ ] 생명주기 함수의 범위를 구분한다.
- [ ] Snapshot 변경을 리뷰할 수 있다.

### 복습 질문 및 답변

**Q1. 객체 두 개를 `toBe`로 비교하면 실패할 수 있는 이유는?**

<details>
<summary>답</summary>

내용이 같아도 서로 다른 객체 참조이기 때문입니다.

</details>

**Q2. 테스트 간 Mock 호출 기록을 지우는 이유는?**

<details>
<summary>답</summary>

이전 테스트의 호출이 다음 테스트의 횟수와 인자 검증에 섞이는 일을 막기 위해서입니다.

</details>

**Q3. Snapshot이 실패했다고 바로 갱신하면 안 되는 이유는?**

<details>
<summary>답</summary>

실패가 의도된 UI 변경이 아니라 회귀 버그일 수 있으므로 변경 내용을 먼저 검토해야 합니다.

</details>

## 한 줄 정리

> Jest의 핵심은 테스트가 언제 끝나고 무엇을 기대하며 어떤 의존성을 통제하는지 명시하는 것입니다.
