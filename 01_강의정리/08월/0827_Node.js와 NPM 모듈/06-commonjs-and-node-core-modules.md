# CommonJS와 Node.js 기본 모듈

Node.js 프로그램은 기능을 작은 모듈로 나누고, 런타임이 제공하는 파일·프로세스·HTTP 기능을 조합해 만듭니다.

**핵심 키워드:** 모듈, CommonJS, require, module.exports, 기본 모듈

## 핵심 내용

- 모듈은 관련 값과 함수를 한 파일 단위로 캡슐화하고 필요한 API만 공개합니다.
- CommonJS는 `require`로 불러오고 `module.exports`로 내보냅니다.
- 패키지는 배포 가능한 파일과 메타데이터의 묶음이며 여러 모듈을 포함할 수 있습니다.
- Node.js는 `fs`, `path`, `http`, `process` 등 기본 모듈과 전역 기능을 제공합니다.
- 로컬 모듈 경로에는 `./` 또는 `../`를 사용하고 패키지 이름과 구분합니다.

## 모듈 작성과 사용

```javascript
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };
```

```javascript
// app.js
const { add } = require("./math");
console.log(add(2, 3));
```

- **목적:** 계산 기능을 별도 파일에 감추고 공개 API만 사용합니다.
- **흐름:** 모듈 평가 → `module.exports` 구성 → require 결과 반환 → 함수 호출입니다.
- **결과:** 파일 간 책임이 나뉘고 테스트와 재사용이 쉬워집니다.
- **실무 포인트:** `exports` 변수에 새 객체를 직접 재할당하면 `module.exports`와 연결이 끊길 수 있습니다. 최종 내보내기는 `module.exports`를 기준으로 이해합니다.

## 불러오기 대상 구분

| 작성 방식 | 대상 |
|---|---|
| `require("node:fs")` | Node.js 기본 모듈 |
| `require("express")` | 설치한 패키지 |
| `require("./math")` | 현재 파일 기준 로컬 모듈 |
| `require("./data.json")` | 로컬 JSON 데이터 |

`node:` 접두사는 기본 모듈임을 명확히 보여 줍니다.

## 주요 기본 기능

```javascript
const fs = require("node:fs/promises");
const path = require("node:path");

async function readConfig() {
  const file = path.join(process.cwd(), "config.json");
  return JSON.parse(await fs.readFile(file, "utf8"));
}
```

- `console`: 표준 출력과 디버깅
- `process`: 인자, 환경 변수, 종료 상태 등 현재 프로세스 정보
- `fs`: 파일과 디렉터리 입출력
- `path`: 운영체제에 맞는 경로 조합
- `http`: HTTP 서버와 클라이언트 기반 기능

## 실습

1. 두 숫자를 곱하는 CommonJS 모듈을 만들고 불러오세요.
2. 기본 모듈, 외부 패키지, 로컬 모듈 import 표기를 비교하세요.
3. `exports = { add }`가 기대와 다를 수 있는 이유를 설명하세요.

<details>
<summary>답</summary>

```javascript
// multiply.js
module.exports = (a, b) => a * b;

// app.js
const multiply = require("./multiply");
console.log(multiply(3, 4));
```

`exports`는 처음에 `module.exports`를 참조하지만 새 객체를 재할당하면 연결이 끊어집니다.

</details>

## 더 알아보기

- [NPM 프로젝트와 의존성 관리](04-npm-projects-and-dependencies.md)
- [ES Module과 모듈 선택](07-es-modules-and-module-selection.md)

## 체크리스트

- [ ] 모듈과 패키지를 구분한다.
- [ ] require 대상의 경로 규칙을 이해한다.
- [ ] module.exports로 공개 API를 구성한다.
- [ ] 기본 모듈에 node 접두사를 사용할 수 있다.
- [ ] 동기 파일 API가 이벤트 루프에 미치는 영향을 고려한다.

## 복습 질문 및 답변

### Q1. 하나의 NPM 패키지는 하나의 모듈만 포함하나요?

<details>
<summary>답</summary>

아닙니다. 패키지는 배포 단위이며 여러 JavaScript 모듈, 데이터, 타입과 메타데이터를 포함할 수 있습니다.

</details>

### Q2. 로컬 파일을 불러올 때 `./`가 필요한 이유는 무엇인가요?

<details>
<summary>답</summary>

패키지 이름 탐색과 구분하고 현재 파일을 기준으로 한 상대 경로임을 모듈 로더에 알리기 위해서입니다.

</details>

### Q3. `node:fs`와 `fs`는 어떤 관계인가요?

<details>
<summary>답</summary>

둘 다 Node.js 파일 시스템 기본 모듈을 가리킬 수 있지만 `node:` 접두사는 기본 제공 모듈이라는 의도를 명확하게 합니다.

</details>

## 요약

CommonJS는 require와 module.exports로 모듈 경계를 만듭니다. 기본 모듈, 외부 패키지와 로컬 파일의 탐색 규칙을 구분하면 Node.js 프로그램을 책임별로 구성할 수 있습니다.
