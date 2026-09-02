# ES Module과 모듈 선택

Node.js는 CommonJS와 표준 ES Module을 모두 지원하므로 프로젝트 시작 시 한 가지 체계를 명확히 정하는 것이 중요합니다.

**핵심 키워드:** ESM, import, export, CommonJS, package type

## 핵심 내용

- ES Module은 `import`와 `export`를 사용하는 JavaScript 표준 모듈 체계입니다.
- Node.js에서는 파일 확장자와 `package.json`의 `type` 설정이 모듈 해석에 영향을 줍니다.
- ESM의 import는 정적 구조를 제공하며 필요할 때 동적 `import()`도 사용할 수 있습니다.
- CommonJS와 ESM의 기본 내보내기·경로·전역 값은 동일하지 않습니다.
- 새 프로젝트는 도구와 의존성 호환성을 확인해 한 체계를 기본으로 정하고 혼용 경계를 최소화합니다.

## ESM 설정과 사용

```json
{
  "type": "module"
}
```

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// app.js
import { add } from "./math.js";
console.log(add(2, 3));
```

- **목적:** 표준 모듈 문법으로 API를 내보내고 가져옵니다.
- **흐름:** 모듈 그래프 분석 → 의존 모듈 로드 → 평가 → 함수 사용입니다.
- **결과:** 정적 import 관계가 명시됩니다.
- **실무 포인트:** Node.js ESM의 상대 경로에는 파일 확장자를 명시하는 습관이 안전합니다.

## CommonJS와 ESM 비교

| 구분 | CommonJS | ES Module |
|---|---|---|
| 가져오기 | `require()` | `import` |
| 내보내기 | `module.exports` | `export` |
| 기본 파일 모드 | 프로젝트 설정에 따라 결정 | `type: module` 또는 `.mjs` |
| 동적 로드 | `require()` 호출 | `import()` Promise |
| 표준 위치 | Node.js 생태계에서 시작 | JavaScript 표준 |

## 경로와 전역 값 차이

CommonJS에서 익숙한 `__filename`, `__dirname`은 ESM에 그대로 제공되지 않습니다. 모듈 URL을 기반으로 필요한 경로를 계산합니다.

```javascript
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";

const filename = fileURLToPath(import.meta.url);
const currentDirectory = dirname(filename);
```

## 모듈 체계 선택

새 프로젝트라면 표준 문법과 생태계 방향을 고려해 ESM을 우선 검토할 수 있습니다. 기존 CommonJS 프로젝트는 마이그레이션 비용과 의존성 호환성을 평가합니다. 라이브러리는 사용자의 모듈 환경을 고려한 배포 구성이 필요하며, 무분별한 이중 패키지는 서로 다른 모듈 인스턴스 문제를 만들 수 있습니다.

## 실습

1. CommonJS 계산 모듈을 ESM 문법으로 바꾸세요.
2. ESM에서 현재 파일 디렉터리를 계산하세요.
3. 기존 프로젝트의 모듈 체계를 바꾸기 전 확인할 항목을 적으세요.

<details>
<summary>답</summary>

```javascript
export const multiply = (a, b) => a * b;
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";
const currentDirectory = dirname(fileURLToPath(import.meta.url));
```

Node.js 버전, package type, 파일 확장자, 테스트·빌드 도구와 모든 의존성의 ESM 호환성을 확인합니다.

</details>

## 더 알아보기

- [NPM 스크립트와 NPX](05-npm-scripts-and-npx.md)
- [CommonJS와 Node.js 기본 모듈](06-commonjs-and-node-core-modules.md)

## 체크리스트

- [ ] 프로젝트의 기본 모듈 체계를 확인한다.
- [ ] import와 export 문법을 사용할 수 있다.
- [ ] 상대 경로와 확장자 규칙을 확인한다.
- [ ] CommonJS 전역 값을 그대로 가정하지 않는다.
- [ ] 마이그레이션 전 도구와 의존성을 검증한다.

## 복습 질문 및 답변

### Q1. 같은 `.js` 파일은 항상 같은 모듈 형식으로 해석되나요?

<details>
<summary>답</summary>

아닙니다. Node.js에서는 가까운 package.json의 `type` 설정과 파일 확장자 등에 따라 해석이 달라질 수 있습니다.

</details>

### Q2. ESM에서도 동적으로 모듈을 불러올 수 있나요?

<details>
<summary>답</summary>

가능합니다. `import()`를 사용하면 Promise로 모듈 네임스페이스를 받아 조건부 또는 지연 로딩을 구현할 수 있습니다.

</details>

### Q3. 기존 CommonJS 프로젝트를 즉시 ESM으로 바꾸는 것이 항상 좋은가요?

<details>
<summary>답</summary>

아닙니다. 얻는 이점과 함께 의존성, 테스트, 빌드, 경로 처리와 배포 환경의 호환성 비용을 평가해야 합니다.

</details>

## 요약

ES Module은 JavaScript 표준 모듈 체계이며 Node.js에서 명시적인 설정과 경로 규칙이 필요합니다. 프로젝트 전체의 호환성과 유지보수성을 기준으로 한 체계를 일관되게 선택하세요.
