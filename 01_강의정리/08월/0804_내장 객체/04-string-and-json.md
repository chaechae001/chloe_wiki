# String과 JSON: 텍스트 처리와 데이터 변환

> 문자열을 다루는 메서드와 JSON 데이터 변환은 비슷해 보이지만 목적과 안전한 사용법이 다릅니다.

`String` · `immutable` · `split` · `JSON.stringify` · `JSON.parse`

## 핵심요약

- 문자열은 불변이므로 메서드 결과를 새 변수나 기존 변수에 다시 저장해야 합니다.
- 검색·분리·치환·공백 제거 메서드는 목적에 맞게 조합합니다.
- JSON 직렬화는 JavaScript 값을 데이터 교환용 문자열로 바꿉니다.
- 역직렬화는 실패할 수 있으므로 외부 문자열은 `try...catch`로 처리합니다.
- JSON을 수정할 때는 문자열 조작보다 파싱 → 객체 수정 → 재직렬화를 사용합니다.

## 1. 문자열 검색과 변환

### 1) 정의

문자열 객체의 메서드는 원본을 바꾸지 않고 결과 문자열이나 배열·불리언·인덱스를 반환합니다.

### 2) 왜 필요한가

사용자 입력 정리, 검색어 비교, CSV 형태 분리, URL·메시지 가공처럼 대부분의 웹 기능에는 텍스트 처리가 포함됩니다.

### 3) 핵심 흐름 재구성

| 목적 | 메서드 | 반환값 |
|---|---|---|
| 양끝 공백 제거 | `trim()` | 새 문자열 |
| 포함 여부 | `includes()` | 불리언 |
| 위치 검색 | `indexOf()` | 인덱스 또는 `-1` |
| 구분자로 나누기 | `split()` | 배열 |
| 첫 일치 치환 | `replace()` | 새 문자열 |
| 대소문자 통일 | `toLowerCase()` | 새 문자열 |

### 4) 쉬운 예시

문자열 메서드는 원본 종이를 지우는 도구가 아니라 복사본을 만들어 편집하는 복사기와 같습니다. 결과를 받지 않으면 원본은 그대로입니다.

### 5) 코드 예시

```javascript
function normalizeTags(rawTags) {
  return rawTags
    .split(",")
    .map((tag) => tag.trim().toLowerCase())
    .filter(Boolean);
}

console.log(normalizeTags(" JavaScript, DOM,  JSON "));
// ["javascript", "dom", "json"]
```

### 6) 헷갈리는 점

`replace()`는 문자열 패턴의 첫 일치만 바꿉니다. 모든 일치를 바꾸려면 `replaceAll()`이나 전역 정규표현식을 고려합니다.

### 7) 한 줄 정리

> 문자열 메서드는 원본이 아닌 반환값을 연결해 정규화 파이프라인을 만듭니다.

## 2. JSON 직렬화와 역직렬화

### 1) 정의

`JSON.stringify()`는 JavaScript 값을 JSON 문자열로 직렬화하고, `JSON.parse()`는 유효한 JSON 문자열을 JavaScript 값으로 역직렬화합니다.

### 2) 왜 필요한가

네트워크 요청 본문이나 브라우저 저장소에는 객체를 그대로 넣을 수 없는 경우가 많습니다. 서로 다른 시스템이 이해할 공통 텍스트 형식으로 변환해야 합니다.

### 3) 핵심 흐름 재구성

```javascript
const profile = { name: "Mina", age: 28 };
const payload = JSON.stringify(profile);
const restored = JSON.parse(payload);
```

`payload`는 문자열, `restored`는 다시 객체입니다. JSON의 객체 키와 문자열은 큰따옴표를 사용합니다.

### 4) 쉬운 예시

객체를 택배 상자에 포장하는 과정이 직렬화, 상자를 열어 물건을 다시 꺼내는 과정이 역직렬화입니다. 포장 문자열을 직접 잘라 붙이면 상자가 손상될 수 있습니다.

### 5) 코드 예시

```javascript
function safeParse(jsonText, fallback = null) {
  try {
    return JSON.parse(jsonText);
  } catch (error) {
    console.warn("JSON 파싱 실패", error.message);
    return fallback;
  }
}

const settings = safeParse('{"theme":"dark"}', {});
console.log(settings.theme); // dark
```

### 6) 헷갈리는 점

JSON은 함수, `undefined`, 순환 참조를 일반 객체처럼 완전히 보존하지 못합니다. `Date`는 보통 문자열로 직렬화되므로 복원 뒤 자동으로 `Date` 객체가 되지 않습니다.

### 7) 한 줄 정리

> JSON은 객체 자체가 아니라 제한된 데이터 구조를 표현하는 문자열 형식입니다.

## 코드로 보기 — 객체를 안전하게 수정해 다시 저장하기

```javascript
function updatePreference(jsonText, key, value) {
  const preferences = safeParse(jsonText, {});

  if (preferences === null || Array.isArray(preferences) || typeof preferences !== "object") {
    return JSON.stringify({ [key]: value });
  }

  preferences[key] = value;
  return JSON.stringify(preferences);
}

const before = '{"theme":"light"}';
const after = updatePreference(before, "fontSize", 18);
console.log(after); // {"theme":"light","fontSize":18}
```

### 코드 목적

JSON 문자열을 직접 이어 붙이지 않고 객체 상태에서 속성을 추가한 뒤 다시 직렬화합니다.

### 코드 흐름

1. JSON 문자열을 안전하게 파싱합니다.
2. 수정 가능한 일반 객체인지 확인합니다.
3. 대괄호 표기법으로 동적 키를 추가합니다.
4. 완성된 객체를 새 JSON 문자열로 바꿉니다.

### 실행 결과 해석

기존 `theme` 속성을 유지하면서 숫자 타입의 `fontSize`가 추가됩니다. 따옴표나 쉼표를 수동으로 조합하지 않습니다.

### 실무 연결

API 요청·응답, `localStorage`, 설정 파일, 메시지 큐처럼 데이터를 저장하거나 전달하는 경계에서 JSON을 사용합니다.

## 직접 해보기

1. 문자열 `"  Hello JS  "`의 공백을 제거하고 소문자로 바꿔 보세요.
2. 객체 `{ active: true }`를 JSON 문자열로 바꿔 보세요.
3. 외부 JSON 파싱에 `try...catch`가 필요한 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. `"  Hello JS  ".trim().toLowerCase()`입니다.
2. `JSON.stringify({ active: true })`입니다.
3. 문법이 잘못된 JSON이면 `JSON.parse()`가 예외를 던져 이후 코드가 중단될 수 있기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| JavaScript 객체 vs JSON 문자열 | 실행 중 데이터 구조와 텍스트 교환 형식입니다. |
| `replace()` vs `replaceAll()` | 첫 일치만 바꾸는지 모든 일치를 바꾸는지 다릅니다. |
| `split()` vs `join()` | 문자열을 배열로 나누는지 배열을 문자열로 합치는지 다릅니다. |
| 파싱 실패 vs 빈 객체 | 잘못된 입력과 유효하지만 속성이 없는 값은 의미가 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [Date와 시간 계산](03-date-and-time.md)
- 다음에 이어지는 개념: [계산기 설계 패턴](05-calculator-design-patterns.md)
- 함께 보면 좋은 키워드: `API`, `localStorage`, `예외 처리`

## 셀프 체크

- [ ] 문자열 메서드가 원본을 바꾸지 않음을 설명할 수 있다.
- [ ] 검색·분리·치환 메서드를 구분할 수 있다.
- [ ] 객체를 JSON 문자열로 직렬화할 수 있다.
- [ ] JSON 문자열을 안전하게 파싱할 수 있다.
- [ ] 객체 수정 후 재직렬화하는 흐름을 구현할 수 있다.

### 복습 질문 및 답변

**Q1. `trim()`을 호출한 원본 변수는 자동으로 바뀌나요?**

<details>
<summary>답</summary>

아닙니다. 새 문자열을 반환하므로 결과를 저장하거나 다음 메서드에 연결해야 합니다.

</details>

**Q2. JSON 문자열에 함수가 포함될 수 있나요?**

<details>
<summary>답</summary>

JSON 데이터 타입에는 함수가 없으며 일반적인 직렬화에서 함수 값은 보존되지 않습니다.

</details>

**Q3. `Date`를 직렬화한 뒤 파싱하면 무엇이 되나요?**

<details>
<summary>답</summary>

일반적으로 날짜 표현 문자열이 되며 필요하면 파싱 후 `new Date(value)`로 다시 만들어야 합니다.

</details>

## 한 줄 정리

> 문자열은 반환값을 받아 가공하고, JSON은 파싱한 객체를 수정한 뒤 다시 직렬화해야 안전합니다.
