# async/await와 Promise 조합

> 여러 비동기 작업은 ‘무엇을 기다릴지’와 ‘실패를 어디까지 묶을지’를 먼저 정해야 읽기 쉬워집니다.

`async` · `await` · `try/catch` · `Promise.all` · `Promise.allSettled`

## 핵심요약

- `async` 함수는 항상 Promise를 반환합니다.
- `await`은 현재 async 함수의 다음 실행을 잠시 미룹니다.
- 서로 의존하는 요청은 순차 처리하고 독립적인 요청은 함께 시작할 수 있습니다.
- Promise 조합 메서드는 원하는 성공·실패 기준에 따라 선택합니다.
- 오류 범위를 작게 나누면 실패 단계와 복구 전략이 명확해집니다.

## 1. 순차 비동기 흐름

```javascript
async function loadOrder(orderId) {
  try {
    const orderResponse = await fetch(`/api/orders/${orderId}`);
    if (!orderResponse.ok) throw new Error('주문 조회 실패');
    const order = await orderResponse.json();

    const customerResponse = await fetch(`/api/customers/${order.customerId}`);
    if (!customerResponse.ok) throw new Error('고객 조회 실패');
    const customer = await customerResponse.json();

    return { order, customer };
  } catch (error) {
    console.error(error.message);
    throw error;
  }
}
```

두 번째 요청이 첫 번째 결과의 `customerId`에 의존하므로 순차 실행이 맞습니다. `catch`에서 기록한 뒤 다시 던지면 호출자도 실패를 처리할 수 있습니다.

## 2. 독립 요청을 동시에 시작하기

```javascript
async function loadDashboard() {
  const [profile, notices] = await Promise.all([
    fetch('/api/profile').then((r) => r.json()),
    fetch('/api/notices').then((r) => r.json()),
  ]);
  return { profile, notices };
}
```

### 코드 목적

서로 의존하지 않는 두 요청을 동시에 시작해 전체 대기 시간을 줄입니다.

### 코드 흐름과 결과 해석

두 Promise가 즉시 생성되고 `Promise.all`은 모두 fulfilled될 때 같은 순서의 배열을 반환합니다. 하나라도 rejected면 전체가 rejected됩니다.

### 실무 연결

대시보드의 여러 위젯, 초기 설정과 사용자 프로필처럼 함께 필요하지만 서로 독립적인 데이터를 로드할 때 적합합니다.

## 3. 조합 메서드 선택

| 메서드 | 완료 기준 | 대표 상황 |
|---|---|---|
| `Promise.all` | 모두 성공, 하나 실패 시 즉시 실패 | 화면 구성에 모든 결과가 필수 |
| `Promise.allSettled` | 모두 성공 또는 실패로 종료 | 일괄 저장 결과를 항목별 확인 |
| `Promise.race` | 가장 먼저 settled된 결과 | 요청과 제한 시간 경쟁 |
| `Promise.any` | 가장 먼저 성공한 결과 | 여러 미러 중 첫 성공 사용 |

`allSettled`의 결과에는 `status`와 함께 `value` 또는 `reason`이 들어 있으므로 실패 항목만 골라 재시도할 수 있습니다.

## 직접 해보기

1. 주문과 고객 조회를 `Promise.all`로 바로 묶기 어려운 이유를 설명하세요.
2. 세 파일 업로드 결과를 모두 확인하려면 어떤 메서드를 쓸지 고르세요.
3. 3초 제한 요청을 `Promise.race`로 설계해 보세요.

<details>
<summary>답</summary>

1. 고객 ID를 주문 응답에서 얻어야 하므로 두 요청이 의존 관계입니다.
2. 성공과 실패를 항목별로 확인하려면 `Promise.allSettled`가 적합합니다.
3. 실제 요청 Promise와 3초 뒤 reject하는 timeout Promise를 배열로 전달합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| 순차 await vs `Promise.all` | 앞 결과가 필요하면 순차, 독립적이면 동시 시작 가능 |
| `all` vs `allSettled` | 모두 성공이 조건 vs 모든 결과 상태 수집 |
| `race` vs `any` | 첫 종료를 선택 vs 첫 성공을 선택 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [이벤트 루프와 Promise](01-event-loop-and-promises.md)
- 다음에 이어지는 개념: [HTTP API와 CORS](03-http-api-openapi-and-cors.md)

## 셀프 체크

- [ ] async 함수의 반환값을 설명할 수 있다.
- [ ] 의존 요청과 독립 요청을 구분한다.
- [ ] 오류 처리 범위를 의도적으로 나눈다.
- [ ] 네 가지 Promise 조합 메서드를 비교한다.
- [ ] 동시 실행 결과의 순서를 해석할 수 있다.

### 복습 질문 및 답변

**Q1. await이 메인 스레드 전체를 멈추는가?**

<details>
<summary>답</summary>

아닙니다. 현재 async 함수의 이어지는 부분만 Promise가 끝날 때까지 미뤄집니다.

</details>

**Q2. `Promise.all` 결과 배열의 순서는 완료 순서인가?**

<details>
<summary>답</summary>

아닙니다. 입력 배열의 Promise 순서를 유지합니다.

</details>

**Q3. 일부 요청 실패를 허용해야 하는 화면에 `Promise.all`이 불편한 이유는?**

<details>
<summary>답</summary>

하나의 실패가 전체 Promise를 rejected로 만들기 때문에 성공한 항목까지 개별적으로 다루기 어렵습니다.

</details>

## 한 줄 정리

> async/await은 비동기 흐름을 읽기 쉽게 만들고, Promise 조합 메서드는 여러 작업의 완료 정책을 표현합니다.
