# 비동기 반복과 작업 흐름 설계

여러 항목을 비동기로 처리할 때 반복문을 바꾸는 것만으로 실행 방식이 달라집니다. 순서를 지켜야 하는지, 모두 독립적인지, 일부 실패를 허용할지를 먼저 정해야 합니다.

## 핵심 키워드

`for...of` · `forEach` · `map` · `Promise.all` · 누적 상태 · 실패 정책

## 핵심 요약

- 순차 처리는 `for...of` 안에서 `await`하는 방식이 명확합니다.
- `forEach(async ...)`는 콜백 Promise를 기다리지 않습니다.
- 독립 항목은 `map`으로 Promise 배열을 만들고 `Promise.all`로 기다릴 수 있습니다.
- 반복 흐름은 순서, 속도, 실패 허용 범위를 함께 설계해야 합니다.

## 1. 순서가 필요한 반복

### 정의와 필요성

각 단계의 종료 값이 다음 단계의 시작 값이 되는 워크플로는 반드시 순서대로 처리해야 합니다.

```javascript
function runStage(total, duration) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(total + duration), duration);
  });
}

async function runPipeline(durations) {
  let total = 0;

  for (const duration of durations) {
    total = await runStage(total, duration);
  }

  return total;
}
```

`total`이 다음 반복의 입력이므로 각 Promise를 기다려야 합니다. `for...of`는 반복문 자체가 await 지점을 포함해 실행 순서를 그대로 보여 줍니다.

> 한 줄 정리: 앞 반복 결과가 다음 반복에 필요하면 for...of와 await로 순서를 보존합니다.

## 2. forEach와 async 콜백의 함정

```javascript
async function processWrong(items) {
  items.forEach(async (item) => {
    await saveItem(item);
  });

  console.log("저장 완료");
}
```

`forEach`는 콜백이 반환한 Promise를 수집하거나 기다리지 않습니다. 따라서 실제 저장이 끝나기 전에 `저장 완료`가 출력될 수 있고, 오류 처리도 바깥 try/catch와 분리될 수 있습니다.

### 순차 수정

```javascript
async function processSequentially(items) {
  for (const item of items) {
    await saveItem(item);
  }

  console.log("저장 완료");
}
```

### 병렬 수정

```javascript
async function processTogether(items) {
  await Promise.all(items.map((item) => saveItem(item)));
  console.log("저장 완료");
}
```

두 수정안은 의미가 다릅니다. 저장 순서가 중요하면 순차 방식, 항목이 독립적이고 동시 요청량이 안전하면 병렬 방식을 선택합니다.

## 3. 오류 범위 설계

```javascript
async function saveAll(items) {
  try {
    return await Promise.all(items.map((item) => saveItem(item)));
  } catch (error) {
    console.error("일괄 저장 실패:", error.message);
    throw error;
  }
}
```

하나의 실패로 전체를 실패시킬지, 실패한 항목만 기록하고 나머지를 계속할지는 제품 정책입니다. 일부 실패를 허용하려면 각 항목에서 오류를 결과 객체로 바꾸는 방법을 사용할 수 있습니다.

```javascript
async function saveSafely(item) {
  try {
    const value = await saveItem(item);
    return { ok: true, value };
  } catch (error) {
    return { ok: false, reason: error.message };
  }
}
```

### 자주 헷갈리는 점

- `await items.forEach(...)`도 forEach 내부 Promise를 기다리지 않습니다.
- 무제한 병렬 요청은 서버와 브라우저 자원을 압박할 수 있습니다.
- 순차 실행은 느릴 수 있지만 순서, 제한, 누적 상태를 보장하기 쉽습니다.

## 대표 코드: 단계별 문서 변환

### 목적

문서 목록을 순서대로 변환하며 앞 문서까지의 누적 크기를 다음 단계에 전달합니다.

```javascript
function convertDocument(name, offset) {
  return Promise.resolve({
    name,
    start: offset,
    size: name.length,
  });
}

async function buildManifest(names) {
  const manifest = [];
  let offset = 0;

  for (const name of names) {
    const document = await convertDocument(name, offset);
    manifest.push(document);
    offset += document.size;
  }

  return manifest;
}
```

### 흐름과 실무 활용

각 문서의 시작 위치가 앞 문서 크기에 의존하므로 순차 처리합니다. 업로드 순서, 마이그레이션 단계, 승인 워크플로처럼 누적 상태가 필요한 작업에도 같은 구조를 사용할 수 있습니다.

## 직접 해보기

독립적인 URL 배열을 `fetchPage(url)`로 모두 요청하고, 전부 완료된 뒤 결과 배열을 반환하는 함수를 작성하세요.

<details>
<summary>답</summary>

```javascript
async function fetchAllPages(urls) {
  const requests = urls.map((url) => fetchPage(url));
  return await Promise.all(requests);
}
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| for...of + await vs map + Promise.all | 순차 처리 vs 독립 작업을 함께 시작 |
| forEach(async) vs Promise.all | forEach는 반환 Promise를 기다리지 않고, Promise.all은 Promise 배열의 완료를 기다립니다. |
| 전체 실패 vs 부분 성공 | 하나의 실패로 전체를 거부할지, 항목별 결과로 변환할지 정책이 다릅니다. |

## 연결되는 개념

- [순차 실행과 병렬 실행](02-sequential-and-parallel-workflows.md)
- 반복 중 오류 정책은 [비동기 오류 처리와 전파](03-async-error-handling.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 순서가 필요한 반복을 for...of로 작성할 수 있다.
- [ ] forEach가 async 콜백을 기다리지 않는 이유를 안다.
- [ ] 전체 실패와 부분 성공 정책을 구분한다.

## 복습 질문 및 답변

### Q1. `await items.forEach(async item => ...)`가 모든 작업을 기다리지 않는 이유는?

<details>
<summary>답</summary>

forEach 자체가 콜백 Promise들을 모아 반환하지 않고 `undefined`를 반환하므로, 바깥 await가 기다릴 Promise가 없기 때문입니다.

</details>

### Q2. 항목을 반드시 입력 순서대로 저장해야 한다면 어떤 구조가 적합한가?

<details>
<summary>답</summary>

`for...of` 반복 안에서 각 저장 Promise를 await하는 순차 구조가 적합합니다.

</details>

### Q3. 병렬 작업 하나가 실패해도 나머지 결과를 보고 싶다면 어떻게 할 수 있는가?

<details>
<summary>답</summary>

각 작업 내부에서 오류를 `{ ok: false, reason }` 같은 결과로 변환해 Promise들이 모두 이행되게 하거나, 요구사항에 맞는 조합 메서드를 선택합니다.

</details>

> 최종 한 줄: 비동기 반복은 문법보다 순서·동시성·실패 정책을 먼저 설계해야 올바른 도구를 선택할 수 있습니다.
