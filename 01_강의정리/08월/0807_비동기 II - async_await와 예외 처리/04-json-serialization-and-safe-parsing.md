# JSON 직렬화와 안전한 파싱

외부에서 받은 JSON 문자열은 신뢰할 수 있는 객체가 아닙니다. 형식이 깨졌거나 예상한 필드가 없을 수 있으므로 변환 단계와 검증 단계를 분리해야 합니다.

## 핵심 키워드

직렬화 · 역직렬화 · `JSON.stringify` · `JSON.parse` · `SyntaxError` · 검증

## 핵심 요약

- 직렬화는 객체를 전송·저장 가능한 문자열로 바꾸는 과정입니다.
- 역직렬화는 JSON 문자열을 JavaScript 값으로 복원하는 과정입니다.
- `JSON.parse`는 문법이 잘못된 문자열에서 예외를 던집니다.
- 파싱 성공은 데이터 구조가 올바르다는 보장이 아니므로 별도 검증이 필요합니다.

## 1. 직렬화와 역직렬화

### 정의와 필요성

메모리 속 객체는 참조와 메서드를 포함한 실행 환경의 구조입니다. 네트워크나 파일에는 표준화된 문자열 형태가 필요하므로 객체를 JSON으로 직렬화하고, 받은 문자열을 다시 역직렬화합니다.

```javascript
const preferences = {
  language: "ko",
  notifications: true,
};

const payload = JSON.stringify(preferences);
console.log(payload);

const restored = JSON.parse(payload);
console.log(restored.language);
```

출력 문자열은 `{"language":"ko","notifications":true}` 형태이며, 복원한 값은 다시 객체 속성으로 접근할 수 있습니다.

> 한 줄 정리: stringify는 객체를 포장하고, parse는 문자열 포장을 풀어 JavaScript 값으로 복원합니다.

## 2. JSON.parse가 실패하는 이유

```javascript
const broken = '{"enabled": true';

try {
  const config = JSON.parse(broken);
  console.log(config.enabled);
} catch (error) {
  console.error("설정 형식이 올바르지 않습니다.");
}
```

닫는 중괄호가 없기 때문에 `SyntaxError`가 발생합니다. try/catch가 없다면 현재 실행 흐름이 중단될 수 있습니다.

### JSON 문법의 주요 특징

- 문자열과 객체 키는 큰따옴표를 사용합니다.
- 주석을 허용하지 않습니다.
- 마지막 항목 뒤의 불필요한 쉼표를 허용하지 않습니다.
- `undefined`, 함수, `Symbol`은 JSON 값으로 표현할 수 없습니다.

## 3. 파싱과 검증은 다른 단계다

```javascript
function isPreferences(value) {
  return (
    value !== null &&
    typeof value === "object" &&
    typeof value.language === "string" &&
    typeof value.notifications === "boolean"
  );
}
```

`{"language": 3}`은 JSON 문법상 올바르므로 파싱에는 성공하지만 앱이 기대한 구조는 아닙니다. 따라서 외부 데이터는 파싱 후 타입과 필수 필드를 검증해야 합니다.

### 자주 헷갈리는 점

- JSON은 JavaScript 객체 표기법과 비슷하지만 동일한 문법이 아닙니다.
- 파싱 성공과 비즈니스 규칙 통과는 별개입니다.
- 오류 객체 전체를 사용자 화면에 노출하면 내부 정보가 드러날 수 있습니다.

## 대표 코드: 안전한 설정 읽기

### 목적

JSON 문자열을 파싱하고 구조를 확인한 뒤, 실패하면 기본 설정을 반환합니다.

```javascript
function parsePreferences(text) {
  try {
    const value = JSON.parse(text);

    if (!isPreferences(value)) {
      throw new TypeError("설정 구조가 올바르지 않습니다.");
    }

    return value;
  } catch (error) {
    console.warn("기본 설정을 사용합니다:", error.message);

    return {
      language: "ko",
      notifications: false,
    };
  }
}
```

### 흐름과 결과

1. JSON 문법을 파싱합니다.
2. 필요한 속성과 타입을 검증합니다.
3. 어느 단계에서든 실패하면 catch로 이동합니다.
4. 앱이 계속 동작할 수 있는 기본값을 반환합니다.

### 실무 활용

로컬 스토리지 설정, 서버 응답, 메시지 큐 데이터처럼 프로그램 바깥에서 들어온 문자열을 처리할 때 사용합니다. 중요한 데이터라면 기본값보다 실패를 다시 전달하는 정책이 더 적절할 수도 있습니다.

## 직접 해보기

JSON 문자열을 파싱한 뒤 `count`가 숫자가 아니면 오류를 던지고, 실패 시 `{ count: 0 }`을 반환하는 함수를 작성하세요.

<details>
<summary>답</summary>

```javascript
function parseCounter(text) {
  try {
    const value = JSON.parse(text);

    if (typeof value.count !== "number") {
      throw new TypeError("count는 숫자여야 합니다.");
    }

    return value;
  } catch (error) {
    return { count: 0 };
  }
}
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| stringify vs parse | 객체를 문자열로 변환 vs 문자열을 JavaScript 값으로 복원 |
| 문법 검증 vs 구조 검증 | JSON으로 읽을 수 있는지 vs 앱이 기대한 필드와 타입인지 |
| 기본값 복구 vs 오류 재전파 | 계속 진행 가능한 선택 데이터 vs 실패를 숨기면 안 되는 필수 데이터 |

## 연결되는 개념

- [비동기 오류 처리와 전파](03-async-error-handling.md)
- 서버 응답을 기다리는 문법은 [async 함수와 await의 기본](01-async-functions-and-await.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 직렬화와 역직렬화를 구분한다.
- [ ] JSON.parse 실패를 try/catch로 처리할 수 있다.
- [ ] 파싱 후 구조 검증이 필요한 이유를 안다.

## 복습 질문 및 답변

### Q1. JSON.parse가 성공하면 데이터가 항상 안전한가?

<details>
<summary>답</summary>

아닙니다. JSON 문법이 올바르다는 뜻일 뿐, 필수 필드와 타입이 앱의 기대와 맞는지는 별도로 검증해야 합니다.

</details>

### Q2. JSON에 함수나 undefined를 그대로 저장할 수 있는가?

<details>
<summary>답</summary>

JSON 표준 값으로 표현할 수 없습니다. stringify 과정에서 생략되거나 배열에서는 null처럼 처리될 수 있으므로 데이터 모델을 명확히 해야 합니다.

</details>

### Q3. 파싱 실패 시 항상 기본값을 반환해야 하는가?

<details>
<summary>답</summary>

아닙니다. 선택 설정은 기본값으로 복구할 수 있지만 결제나 권한처럼 중요한 데이터는 오류를 호출자에게 전달하는 편이 안전합니다.

</details>

> 최종 한 줄: 외부 JSON은 파싱하고, 구조를 검증하고, 실패 정책까지 정해야 안전하게 사용할 수 있습니다.
