# State와 불변 갱신

State는 컴포넌트가 렌더링 사이에 기억하는 데이터이며, 직접 수정하지 않고 setter에 새 값을 전달해야 화면 변화가 예측 가능합니다.

## 핵심 키워드

`State` · `useState` · `setter` · `functional update` · `immutability`

## 핵심 요약

- `useState`는 현재값과 갱신 함수를 반환합니다.
- State 변수에 직접 대입하거나 객체를 직접 수정하지 않습니다.
- 이전값에 의존하면 함수형 갱신을 사용합니다.
- 객체 State는 전개 구문 등으로 새 객체를 만들어 변경합니다.

## 1. State 생성과 변경

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

setter 호출은 다음 렌더링을 요청합니다. 현재 실행 중인 이벤트 처리기 안의 State 값은 그 렌더링의 스냅샷처럼 유지됩니다.

## 2. 함수형 갱신

이전값을 바탕으로 여러 번 갱신할 때는 updater 함수를 사용합니다.

```jsx
const addThree = () => {
  setCount((previous) => previous + 1);
  setCount((previous) => previous + 1);
  setCount((previous) => previous + 1);
};
```

각 함수가 최신 대기 상태를 받아 순서대로 계산합니다.

## 3. 객체 State와 불변성

```jsx
const [profile, setProfile] = useState({ name: "새싹", point: 0 });

const addPoint = () => {
  setProfile((previous) => ({
    ...previous,
    point: previous.point + 10,
  }));
};
```

기존 객체를 직접 바꾸지 않고 필요한 프로퍼티가 달라진 새 객체를 반환합니다.

## 대표 코드: 설정 객체 갱신

### 목적

여러 프로퍼티가 있는 객체 State에서 하나의 값만 안전하게 변경합니다.

```jsx
import { useState } from "react";

function Settings() {
  const [settings, setSettings] = useState({ theme: "light", alerts: true });

  const toggleAlerts = () => {
    setSettings((previous) => ({
      ...previous,
      alerts: !previous.alerts,
    }));
  };

  return <button onClick={toggleAlerts}>{settings.alerts ? "알림 켜짐" : "알림 꺼짐"}</button>;
}
```

### 코드 흐름과 결과

1. 객체를 초기 State로 둡니다.
2. 클릭 시 최신 이전 State를 받습니다.
3. 기존 프로퍼티를 펼치고 alerts만 반전합니다.
4. 새 객체 참조가 다음 렌더링에 전달됩니다.

### 실무 연결

폼 데이터, 필터 설정, 선택 상태처럼 UI가 기억해야 하는 값을 관리하는 기본 패턴입니다.

## 직접 해보기

1. 일반 변수와 State의 차이를 설명하세요.
2. 객체 State의 name만 바꾸는 코드를 작성하세요.
3. 연속 갱신에 함수형 업데이트가 필요한 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. State는 렌더링 사이에 보존되고 setter가 재렌더링을 요청하지만 지역 변수는 렌더링마다 다시 만들어집니다.
2. `setProfile((previous) => ({ ...previous, name: nextName }));`처럼 작성합니다.
3. 각 updater가 최신 대기값을 받아 여러 변경이 덮어써지지 않고 누적되기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| State 변경 요청 vs 즉시 변수 변경 | setter는 다음 렌더링을 예약하고 현재 State 스냅샷을 바꾸지 않습니다. |
| 직접 변경 vs 불변 갱신 | 직접 변경은 기존 참조를 수정하고 불변 갱신은 새 참조를 만듭니다. |
| 값 전달 vs 함수형 갱신 | 독립적 새 값은 직접 전달, 이전값 의존 계산은 updater 함수를 사용합니다. |

## 연결되는 개념

- 외부 입력은 [Props와 단방향 데이터 흐름](04-props-and-data-flow.md)에서 확인할 수 있습니다.
- 입력 속성은 [스타일과 React DOM 속성](02-styles-and-dom-attributes.md)에서 설명합니다.
- 배열 State와 key는 [목록 key와 입력 상태 설계](06-list-keys-and-form-state.md)에서 이어집니다.

## 셀프 체크

- [ ] useState의 반환값을 설명할 수 있다.
- [ ] 함수형 갱신을 사용할 상황을 안다.
- [ ] 객체 State를 불변하게 갱신할 수 있다.

## 복습 질문 및 답변

### Q1. State 변수에 직접 대입하면 화면이 갱신되는가?

<details>
<summary>답</summary>

React에 변경을 알리지 못하므로 setter를 사용해야 합니다.

</details>

### Q2. 객체 State 일부를 바꿀 때 나머지 프로퍼티는 자동 유지되는가?

<details>
<summary>답</summary>

useState setter는 객체를 자동 병합하지 않으므로 전개 구문 등으로 필요한 기존 값을 포함해야 합니다.

</details>

### Q3. 화면에서 쓰지 않는 모든 값을 State로 두어야 하는가?

<details>
<summary>답</summary>

아닙니다. 렌더링에 영향을 주지 않는 값이나 기존 값에서 계산 가능한 파생값은 State가 아닐 수 있습니다.

</details>

## 한 줄 정리

> State는 setter와 새 참조를 통해 갱신하여 React가 데이터 변화와 화면 렌더링을 연결하도록 합니다.
