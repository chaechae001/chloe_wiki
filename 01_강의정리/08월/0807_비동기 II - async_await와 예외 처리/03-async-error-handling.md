# 비동기 오류 처리와 전파

성공 흐름만 있는 비동기 코드는 실제 서비스에서 오래 버티지 못합니다. 네트워크 실패, 잘못된 입력, 파싱 오류를 어디서 잡고 어디까지 전달할지 설계해야 합니다.

## 핵심 키워드

`try` · `catch` · `finally` · `throw` · 오류 전파 · 복구

## 핵심 요약

- rejected Promise를 await하면 해당 지점에서 예외가 발생합니다.
- `try` 안의 동기 오류와 await 실패는 연결된 `catch`가 처리할 수 있습니다.
- 처리할 수 없는 오류는 다시 던져 호출자가 판단하게 합니다.
- `finally`는 성공·실패와 무관한 정리에 사용합니다.

## 1. await 실패는 예외가 된다

### 정의와 흐름

```javascript
function requestReport(allowed) {
  if (allowed) {
    return Promise.resolve({ rows: 12 });
  }

  return Promise.reject(new Error("접근 권한이 없습니다."));
}

async function showReport() {
  try {
    const report = await requestReport(false);
    console.log(report.rows);
  } catch (error) {
    console.error(error.message);
  }
}
```

Promise가 거부되면 `await` 다음 줄은 실행되지 않고 `catch`로 이동합니다. Promise 체인의 `.catch()`와 역할은 같지만 일반 제어문과 함께 읽기 쉽습니다.

> 한 줄 정리: rejected Promise는 await 지점에서 throw된 오류처럼 동작합니다.

## 2. 오류를 처리할지 다시 전달할지

### 복구 가능한 오류

```javascript
async function loadOptionalSetting() {
  try {
    return await Promise.reject(new Error("설정 없음"));
  } catch (error) {
    console.warn(error.message);
    return { mode: "default" };
  }
}
```

기본값으로 계속 진행할 수 있다면 catch에서 복구 값을 반환할 수 있습니다. 이때 함수의 Promise는 fulfilled 상태로 바뀝니다.

### 호출자가 판단해야 하는 오류

```javascript
async function loadRequiredProfile() {
  try {
    return await Promise.reject(new Error("프로필 조회 실패"));
  } catch (error) {
    console.error("조회 기록:", error.message);
    throw error;
  }
}
```

현재 함수가 해결할 수 없다면 기록 후 다시 던집니다. 오류를 무조건 삼키면 호출자는 실패를 성공으로 오해할 수 있습니다.

## 3. finally의 올바른 역할

```javascript
let loading = false;

async function refresh() {
  loading = true;

  try {
    return await Promise.resolve("새 데이터");
  } catch (error) {
    throw error;
  } finally {
    loading = false;
  }
}
```

로딩 해제는 성공과 실패 모두에서 실행되어야 하므로 `finally`가 알맞습니다. `finally`에서 값을 반환하면 앞선 반환값이나 오류를 덮을 수 있으므로 일반적으로 정리만 수행합니다.

### 자주 헷갈리는 점

- await하지 않고 시작한 Promise는 현재 try/catch가 놓칠 수 있습니다.
- catch에서 아무 처리도 하지 않으면 오류 원인이 사라집니다.
- 사용자 메시지와 개발자용 로그는 목적이 다르므로 분리하는 편이 좋습니다.

## 대표 코드: 오류를 분류하는 서비스 함수

### 목적

입력 오류는 호출자에게 명확히 알리고, 일시적 조회 실패에는 기본값을 사용합니다.

```javascript
async function findDisplayName(userId) {
  if (!Number.isInteger(userId)) {
    throw new TypeError("userId는 정수여야 합니다.");
  }

  try {
    const user = await Promise.resolve({ id: userId, name: "Mina" });
    return user.name;
  } catch (error) {
    console.error("사용자 조회 실패:", error.message);
    return "알 수 없음";
  }
}
```

### 흐름과 실무 활용

호출 규칙 위반은 즉시 `TypeError`로 거부합니다. 외부 조회 실패는 서비스 정책상 기본 이름으로 복구합니다. 오류 종류와 제품 요구에 따라 “복구할지, 다시 던질지”를 선택하는 예입니다.

## 직접 해보기

`saveDraft()`를 await하고, 실패하면 오류 메시지를 기록한 뒤 다시 던지며, 성공·실패와 관계없이 `saving`을 false로 바꾸는 함수를 작성하세요.

<details>
<summary>답</summary>

```javascript
let saving = false;

async function submitDraft() {
  saving = true;

  try {
    return await saveDraft();
  } catch (error) {
    console.error(error.message);
    throw error;
  } finally {
    saving = false;
  }
}
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| catch에서 반환 vs 다시 throw | 반환하면 성공 흐름으로 복구하고, 다시 던지면 실패를 호출자에게 전달합니다. |
| catch vs finally | catch는 실패 대응, finally는 결과와 무관한 공통 정리입니다. |
| 오류 로그 vs 사용자 메시지 | 로그는 원인 조사용 상세 정보, 사용자 메시지는 안전하고 이해하기 쉬운 안내입니다. |

## 연결되는 개념

- [async 함수와 await의 기본](01-async-functions-and-await.md)
- 파싱 오류 사례는 [JSON 직렬화와 안전한 파싱](04-json-serialization-and-safe-parsing.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] rejected Promise가 await에서 어떻게 동작하는지 안다.
- [ ] 오류 복구와 재전파를 구분한다.
- [ ] finally에 적합한 작업을 선택할 수 있다.

## 복습 질문 및 답변

### Q1. catch에서 기본값을 반환하면 async 함수의 최종 상태는?

<details>
<summary>답</summary>

catch가 정상적으로 값을 반환했으므로 그 값으로 fulfilled된 Promise가 됩니다.

</details>

### Q2. 오류를 기록한 뒤 호출자도 실패를 알아야 한다면 어떻게 하는가?

<details>
<summary>답</summary>

catch에서 로그를 남긴 뒤 `throw error`로 다시 던집니다.

</details>

### Q3. finally에서 return을 피하는 이유는?

<details>
<summary>답</summary>

앞선 try의 반환값이나 catch되지 않은 오류를 덮어 실행 결과를 왜곡할 수 있기 때문입니다.

</details>

> 최종 한 줄: 좋은 예외 처리는 오류를 무조건 숨기지 않고, 복구 가능한 곳에서만 복구하며 정리는 반드시 수행합니다.
