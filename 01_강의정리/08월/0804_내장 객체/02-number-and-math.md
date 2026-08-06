# Number와 Math: 숫자 처리와 수학 계산

> 숫자를 계산하는 것과 사람이 읽을 문자열로 표시하는 것은 서로 다른 단계입니다.

`Number` · `NaN` · `toFixed` · `Math` · `random`

## 핵심요약

- 폼 입력값은 문자열이므로 계산 전에 명시적으로 숫자로 변환합니다.
- `Number.isFinite()`와 `Number.isNaN()`으로 계산 가능한 값인지 확인합니다.
- `toFixed()`는 반올림된 문자열을 반환하므로 표시 단계에 사용합니다.
- `Math`는 생성자 없이 정적 프로퍼티와 메서드로 사용합니다.
- 범위 난수 공식은 구간 길이와 시작값을 모두 반영해야 합니다.

## 1. 숫자 변환·검증·표시

### 1) 정의

`Number`는 숫자 타입과 관련된 변환·검증 기능을 제공합니다. 숫자 값의 메서드인 `toFixed()`와 `toLocaleString()`은 표시용 문자열을 만듭니다.

### 2) 왜 필요한가

HTML 입력값은 언제나 문자열입니다. `"10" + "5"`는 `"105"`가 되므로 계산 전에 숫자 변환과 유효성 검사가 필요합니다.

### 3) 핵심 흐름 재구성

```javascript
const value = Number(input.value);

if (!Number.isFinite(value)) {
  throw new Error("유효한 숫자가 아닙니다.");
}

const display = value.toFixed(2);
```

계산 중에는 숫자를 유지하고, 화면에 보여 줄 마지막 단계에서 문자열로 포매팅합니다.

### 4) 쉬운 예시

계산기는 숫자를 받아야 하지만 입력창은 글자를 전달합니다. 글자를 숫자로 통역하고, 계산이 끝난 뒤 다시 통화나 소수 형식의 글자로 번역하는 과정입니다.

### 5) 코드 예시

```javascript
function formatPrice(rawValue) {
  const price = Number(rawValue);
  if (!Number.isFinite(price) || price < 0) return "입력 오류";
  return `${price.toLocaleString("ko-KR")}원`;
}

console.log(formatPrice("12500")); // 12,500원
```

### 6) 헷갈리는 점

`NaN === NaN`은 `false`입니다. 값이 `NaN`인지 확인할 때는 `Number.isNaN(value)`를 사용합니다. 무한대까지 제외해야 하는 일반 계산 입력에는 `Number.isFinite(value)`가 더 직접적입니다.

### 7) 한 줄 정리

> 입력은 숫자로 검증하고, 계산은 숫자로 유지하며, 출력 직전에 문자열로 포매팅합니다.

## 2. Math 메서드와 범위 난수

### 1) 정의

`Math`는 제곱근, 거듭제곱, 반올림, 최댓값·최솟값, 난수 같은 수학 기능을 가진 내장 객체입니다. `new Math()`로 만들지 않고 `Math.sqrt()`처럼 바로 사용합니다.

### 2) 왜 필요한가

거리, 통계 요약, 금액 처리, 무작위 선택처럼 자주 쓰이는 계산을 검증된 표준 메서드로 표현하면 코드의 의도가 선명해집니다.

### 3) 핵심 흐름 재구성

| 목적 | 메서드 | 예시 결과 |
|---|---|---|
| 반올림 | `Math.round(3.6)` | `4` |
| 내림 | `Math.floor(3.9)` | `3` |
| 올림 | `Math.ceil(3.1)` | `4` |
| 제곱근 | `Math.sqrt(81)` | `9` |
| 거듭제곱 | `Math.pow(2, 3)` | `8` |
| 최댓값 | `Math.max(...values)` | 배열의 최대값 |

### 4) 쉬운 예시

`Math`는 공학용 계산기의 기능 버튼과 같습니다. 계산기 자체를 새로 만들지 않고 필요한 버튼을 눌러 결과를 얻습니다.

### 5) 코드 예시

```javascript
function randomInteger(min, max) {
  const lower = Math.ceil(min);
  const upper = Math.floor(max);
  return Math.floor(Math.random() * (upper - lower + 1)) + lower;
}

console.log(randomInteger(1, 6)); // 1~6 중 하나
```

### 6) 헷갈리는 점

`Math.random()`은 0 이상 1 미만입니다. 상한을 포함하는 정수 구간에서는 구간 길이에 `+ 1`이 필요합니다. 보안 토큰이나 비밀번호 생성에는 암호학적으로 안전하지 않으므로 사용하지 않습니다.

### 7) 한 줄 정리

> `Math` 메서드는 계산 의도를 드러내되 입력 범위와 반올림 규칙을 명확히 해야 합니다.

## 코드로 보기 — 두 점 사이 거리 계산하기

```javascript
function distanceBetween(pointA, pointB) {
  const dx = Number(pointB.x) - Number(pointA.x);
  const dy = Number(pointB.y) - Number(pointA.y);

  if (![dx, dy].every(Number.isFinite)) return null;
  return Math.hypot(dx, dy);
}

const distance = distanceBetween({ x: 1, y: 2 }, { x: 4, y: 6 });
console.log(distance?.toFixed(2)); // "5.00"
```

### 코드 목적

좌표 입력을 숫자로 변환하고 두 점의 유클리드 거리를 구합니다.

### 코드 흐름

1. x축과 y축의 차이를 계산합니다.
2. 두 차이가 유한한 숫자인지 확인합니다.
3. `Math.hypot()`으로 제곱합의 제곱근을 구합니다.
4. 화면 표시 시에만 소수 둘째 자리 문자열로 바꿉니다.

### 실행 결과 해석

x 차이는 3, y 차이는 4이며 거리는 5입니다. 계산 결과는 숫자 `5`, 표시 결과는 문자열 `"5.00"`입니다.

### 실무 연결

지도 좌표의 단순 거리, 캔버스 드래그 거리, 게임 오브젝트 간 거리, 데이터 정규화 전 기본 계산에 같은 구조를 사용할 수 있습니다.

## 직접 해보기

1. 문자열 `"42.5"`를 숫자로 바꾸고 유효성을 검사해 보세요.
2. 배열 `[4, 9, -2]`의 최댓값과 최솟값 차이를 구해 보세요.
3. 10 이상 20 이하의 무작위 정수를 만드는 식을 작성해 보세요.

<details><summary>정답 보기</summary>

1. `const n = Number("42.5"); Number.isFinite(n);`입니다.
2. `Math.max(...values) - Math.min(...values)`로 `11`을 얻습니다.
3. `Math.floor(Math.random() * 11) + 10`입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `Number()` vs `parseFloat()` | 전체 값을 숫자로 변환하는지 앞부분의 숫자를 해석하는지 동작이 다릅니다. |
| `toFixed()` vs `Math.round()` | 소수 자릿수 문자열 포맷과 숫자 반올림의 차이입니다. |
| `Math.floor()` vs `Math.trunc()` | 음수에서 아래 정수로 내리는지 소수 부분만 버리는지 다릅니다. |
| `NaN` vs `Infinity` | 숫자 아님과 무한대이며 둘 다 일반 입력 계산에서는 제외할 수 있습니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [브라우저 내장 객체](01-browser-built-in-objects.md)
- 다음에 이어지는 개념: [Date와 시간 계산](03-date-and-time.md)
- 함께 보면 좋은 키워드: `형 변환`, `부동소수점`, `포매팅`

## 셀프 체크

- [ ] 입력 문자열을 숫자로 변환할 수 있다.
- [ ] 유한한 숫자인지 검증할 수 있다.
- [ ] 계산값과 표시 문자열을 분리할 수 있다.
- [ ] 주요 `Math` 메서드를 목적에 맞게 선택할 수 있다.
- [ ] 포함 범위 난수 공식을 설명할 수 있다.

### 복습 질문 및 답변

**Q1. `toFixed(2)`의 반환 타입은 무엇인가요?**

<details>
<summary>답</summary>

소수 둘째 자리로 반올림된 문자열입니다.

</details>

**Q2. `Math.floor(-1.2)`는 얼마인가요?**

<details>
<summary>답</summary>

주어진 값보다 크지 않은 가장 큰 정수인 `-2`입니다.

</details>

**Q3. 금액 계산 중간마다 `toFixed()`를 쓰면 어떤 문제가 생길 수 있나요?**

<details>
<summary>답</summary>

문자열로 바뀌고 중간 반올림이 누적되어 이후 계산의 정확성과 타입 일관성이 떨어질 수 있습니다.

</details>

## 한 줄 정리

> 숫자 처리는 변환·검증·계산·표시를 분리해야 타입 오류와 반올림 오류를 줄일 수 있습니다.
