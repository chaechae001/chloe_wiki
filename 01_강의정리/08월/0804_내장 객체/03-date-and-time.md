# Date와 시간 계산

> 날짜 화면은 문자열처럼 보이지만, 비교와 계산은 밀리초 숫자로 바꿀 때 단순해집니다.

`Date` · `timestamp` · `getMonth` · `relative time` · `setInterval`

## 핵심요약

- `Date`는 특정 시점을 나타내며 내부 계산에는 타임스탬프를 활용합니다.
- 월은 0~11, 일은 `getDate()`, 요일은 `getDay()`로 조회합니다.
- 잘못된 `Date`도 객체일 수 있으므로 타임스탬프가 `NaN`인지 확인합니다.
- 두 시점의 차이는 밀리초로 구한 뒤 초·분·시간·일 단위로 환산합니다.
- 반복 시계는 타이머 ID를 보관하고 더 이상 필요 없을 때 정리합니다.

## 1. Date 생성과 구성요소

### 1) 정의

`Date` 객체는 한 시점을 나타냅니다. 현재 시각은 `new Date()`, 타임스탬프는 `new Date(milliseconds)`, 명시적인 구성요소는 `new Date(year, monthIndex, day, ...)`로 만들 수 있습니다.

### 2) 왜 필요한가

현재 시간 표시, 예약일 계산, 상대시간, 데이터 정렬은 문자열 비교보다 시간 값 비교가 안전합니다.

### 3) 핵심 흐름 재구성

```javascript
const now = new Date();
const year = now.getFullYear();
const month = now.getMonth() + 1;
const dayOfMonth = now.getDate();
const dayOfWeek = now.getDay();
```

월의 0 기반 인덱스와 일·요일 메서드 이름을 반드시 구분합니다.

### 4) 쉬운 예시

`Date`는 달력의 한 칸이 아니라 정확한 시점을 가리키는 좌표입니다. 연·월·일은 그 좌표를 사람이 읽기 좋게 꺼낸 구성요소입니다.

### 5) 코드 예시

```javascript
function isValidDate(value) {
  return value instanceof Date && Number.isFinite(value.getTime());
}

function formatDate(date) {
  if (!isValidDate(date)) return "유효하지 않은 날짜";

  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  return `${year}-${month}-${day}`;
}
```

### 6) 헷갈리는 점

`new Date("2026-08-04")` 같은 문자열 파싱은 형식과 시간대 해석을 주의해야 합니다. 확실한 로컬 날짜가 필요하면 숫자 구성요소를 명시하거나 입력 규칙을 고정합니다.

### 7) 한 줄 정리

> 날짜 구성요소의 인덱스와 시간대 규칙을 확인한 뒤 표시 문자열을 만듭니다.

## 2. 시간 차이와 상대시간

### 1) 정의

두 `Date`의 `getTime()` 값을 빼면 밀리초 단위 시간 차이를 얻습니다. 이를 기준값으로 나눠 초·분·시간·일로 환산할 수 있습니다.

### 2) 왜 필요한가

‘몇 분 전’, 남은 시간, 처리 시간, 만료 여부는 절대 날짜보다 시간 차이가 핵심입니다.

### 3) 핵심 흐름 재구성

```javascript
const second = 1000;
const minute = 60 * second;
const hour = 60 * minute;
const day = 24 * hour;
```

큰 단위부터 조건을 확인하면 해당 구간의 정수 몫으로 상대시간을 만들 수 있습니다.

### 4) 쉬운 예시

현재 위치와 출발 위치의 거리 차이를 구하듯, 현재 타임스탬프와 과거 타임스탬프의 차이를 구한 뒤 적절한 단위로 읽습니다.

### 5) 코드 예시

```javascript
function relativeTime(past, now = new Date()) {
  if (![past, now].every(isValidDate)) return "시간을 계산할 수 없습니다.";

  const seconds = Math.floor((now.getTime() - past.getTime()) / 1000);
  if (seconds < 0) return "미래 시간";
  if (seconds < 60) return "방금 전";
  if (seconds < 3600) return `${Math.floor(seconds / 60)}분 전`;
  if (seconds < 86400) return `${Math.floor(seconds / 3600)}시간 전`;
  return `${Math.floor(seconds / 86400)}일 전`;
}
```

### 6) 헷갈리는 점

하루를 항상 24시간으로 환산하는 방식은 일광절약시간 전환 같은 달력 규칙을 완전히 반영하지 못할 수 있습니다. 단순 상대시간과 달력 날짜 차이는 요구사항이 다릅니다.

### 7) 한 줄 정리

> 시간 차이는 타임스탬프로 계산하고 목적에 맞는 단위와 경계값으로 표현합니다.

## 코드로 보기 — 실시간 시계 만들기

```javascript
const clock = document.querySelector("#clock");
let timerId = null;

function renderClock() {
  const now = new Date();
  const date = formatDate(now);
  const time = [now.getHours(), now.getMinutes(), now.getSeconds()]
    .map((value) => String(value).padStart(2, "0"))
    .join(":");

  clock.textContent = `${date} ${time}`;
}

function startClock() {
  if (timerId !== null) return;
  renderClock();
  timerId = setInterval(renderClock, 1000);
}

function stopClock() {
  clearInterval(timerId);
  timerId = null;
}
```

### 코드 목적

현재 시간을 `YYYY-MM-DD HH:mm:ss` 형식으로 매초 갱신하고 필요할 때 중단합니다.

### 코드 흐름

1. 현재 `Date`를 새로 만듭니다.
2. 날짜와 시간 구성요소를 두 자리 문자열로 만듭니다.
3. 초기 화면을 즉시 렌더링합니다.
4. 1초 간격 반복을 시작하고 ID를 보관합니다.
5. 종료 시 반복을 정리하고 ID를 초기화합니다.

### 실행 결과 해석

시작 직후 시간이 보이고 이후 약 1초마다 갱신됩니다. `startClock()`을 여러 번 호출해도 중복 타이머가 생기지 않습니다.

### 실무 연결

대시보드 기준 시각, 세션 남은 시간, 알림 상대시간, 예약 상태를 표시할 때 같은 원리를 사용합니다.

## 직접 해보기

1. 12월을 `getMonth()`로 읽으면 어떤 값이 나오는지 적어 보세요.
2. 두 날짜의 밀리초 차이를 시간 단위로 바꾸는 식을 작성해 보세요.
3. 잘못된 날짜 객체를 검증하는 코드를 작성해 보세요.

<details><summary>정답 보기</summary>

1. 0 기반이므로 `11`입니다.
2. `(later.getTime() - earlier.getTime()) / (60 * 60 * 1000)`입니다.
3. `date instanceof Date && Number.isFinite(date.getTime())`로 검사할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `getDate()` vs `getDay()` | 월의 몇 일인지와 주의 몇 번째 요일인지 다릅니다. |
| `getMonth()` 값 vs 표시 월 | 조회값은 0~11이고 사람용 표시는 보통 1을 더합니다. |
| 절대 날짜 vs 상대시간 | 특정 시점 표시와 현재로부터의 차이 표현입니다. |
| `Date` 객체 존재 vs 유효한 날짜 | 객체여도 내부 타임스탬프가 `NaN`일 수 있습니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [Number와 Math](02-number-and-math.md)
- 다음에 이어지는 개념: [String과 JSON](04-string-and-json.md)
- 함께 보면 좋은 키워드: `시간대`, `밀리초`, `타이머 수명주기`

## 셀프 체크

- [ ] 현재 날짜를 만들 수 있다.
- [ ] 월·일·요일 메서드를 구분할 수 있다.
- [ ] 날짜 유효성을 검사할 수 있다.
- [ ] 두 시점의 차이를 단위별로 환산할 수 있다.
- [ ] 반복 타이머를 시작하고 정리할 수 있다.

### 복습 질문 및 답변

**Q1. `Date`끼리 뺄셈하면 내부적으로 어떤 값이 사용되나요?**

<details><summary>답</summary>

각 시점을 나타내는 밀리초 타임스탬프 차이가 사용됩니다.

</details>

**Q2. 시계 렌더링에서 매번 새 `Date`를 만드는 이유는 무엇인가요?**

<details><summary>답</summary>

기존 `Date` 객체는 생성 당시 시점을 나타내므로 현재 시각을 다시 읽으려면 새 객체가 필요합니다.

</details>

**Q3. 타이머 ID를 `null`로 되돌리는 이유는 무엇인가요?**

<details><summary>답</summary>

코드가 현재 타이머 실행 여부를 정확히 판단하고 이후 다시 시작할 수 있게 하기 위해서입니다.

</details>

## 한 줄 정리

> 날짜·시간 코드는 유효한 시점, 타임스탬프 계산, 표시 형식, 타이머 정리를 분리해야 안정적입니다.

