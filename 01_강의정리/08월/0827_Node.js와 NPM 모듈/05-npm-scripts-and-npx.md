# NPM 스크립트와 NPX

팀원이 긴 명령과 도구 버전을 외우지 않도록 프로젝트의 표준 작업을 이름 있는 스크립트로 캡슐화할 수 있습니다.

**핵심 키워드:** scripts, npm run, NPX, 로컬 바이너리, 일회성 실행

## 핵심 내용

- `scripts`는 개발, 테스트, 빌드 같은 반복 명령의 공통 진입점입니다.
- `npm run`은 로컬 의존성의 실행 파일을 자동으로 찾습니다.
- `pre`와 `post` 접두 스크립트는 특정 스크립트 전후 작업을 연결할 수 있습니다.
- NPX는 패키지 실행 파일을 호출하며 로컬 설치본을 우선 활용할 수 있습니다.
- 원격 또는 처음 보는 패키지를 즉시 실행하기 전 출처, 버전과 실행 코드를 검토합니다.

## scripts 작성

```json
{
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js",
    "test": "node --test",
    "lint": "eslint ."
  }
}
```

```bash
npm run dev
npm test
npm run lint
```

- **목적:** 프로젝트 작업을 운영체제와 개인 환경에 덜 의존하는 이름으로 제공합니다.
- **흐름:** script 이름 조회 → 로컬 바이너리 경로 추가 → 명령 실행 → 종료 코드 반환입니다.
- **결과:** 개발자와 CI가 같은 명령을 사용합니다.
- **실무 포인트:** script 이름은 역할이 분명해야 하며 실패를 숨기는 명령 연결을 피합니다.

## 직접 명령과 npm script

| 방식 | 장점 | 주의점 |
|---|---|---|
| 직접 실행 | 간단한 일회 작업에 빠름 | 옵션과 도구 경로가 사람마다 다를 수 있음 |
| npm script | 팀 표준과 버전을 공유 | package.json 관리 필요 |

## NPX 사용

```bash
npx eslint .
npx --yes some-tool@1.2.3 --help
```

NPX는 프로젝트의 `node_modules/.bin`에 있는 도구를 편하게 실행할 수 있습니다. 로컬에 없는 패키지를 내려받아 실행할 수도 있으므로, 이름이 비슷한 악성 패키지와 임의 코드 실행 위험을 고려해야 합니다. 자동화에서는 버전을 고정하고 신뢰한 패키지만 사용합니다.

## 인자 전달

```bash
npm test -- --test-name-pattern="user"
```

`--` 뒤의 인자는 npm이 아니라 실제 script 명령에 전달됩니다. 도구마다 지원 옵션은 다르므로 해당 도구의 도움말을 확인합니다.

## 실습

1. start, test, lint 스크립트를 작성하고 실행하세요.
2. 로컬 설치된 린터를 NPX로 실행하세요.
3. 일회성 패키지 실행 전 확인할 보안 항목을 적으세요.

<details>
<summary>답</summary>

```json
{"scripts":{"start":"node src/server.js","test":"node --test","lint":"eslint ."}}
```

`npm run lint` 또는 `npx eslint .`로 로컬 도구를 실행할 수 있습니다. 패키지 이름, 게시자, 버전, 다운로드 출처와 실행할 코드를 확인합니다.

</details>

## 더 알아보기

- [NPM 프로젝트와 의존성 관리](04-npm-projects-and-dependencies.md)
- [ES Module과 모듈 선택](07-es-modules-and-module-selection.md)

## 체크리스트

- [ ] 반복 명령을 script로 표준화한다.
- [ ] script 이름이 목적을 드러낸다.
- [ ] 로컬 의존성 버전을 우선 사용한다.
- [ ] 전달 인자의 위치를 이해한다.
- [ ] 미확인 패키지를 즉시 실행하지 않는다.

## 복습 질문 및 답변

### Q1. npm script에서 로컬 바이너리를 경로 없이 실행할 수 있는 이유는 무엇인가요?

<details>
<summary>답</summary>

실행 시 npm이 프로젝트의 로컬 바이너리 디렉터리를 PATH에 추가해 해당 패키지의 실행 파일을 찾기 때문입니다.

</details>

### Q2. NPX는 항상 패키지를 새로 다운로드하나요?

<details>
<summary>답</summary>

항상 그렇지는 않습니다. 로컬에 적합한 실행 파일이 있으면 사용할 수 있고, 없으면 설정과 명령에 따라 패키지를 받아 실행할 수 있습니다.

</details>

### Q3. CI에서 NPX 패키지 버전을 고정해야 하는 이유는 무엇인가요?

<details>
<summary>답</summary>

시간에 따라 다른 코드가 실행되는 것을 방지해 빌드 재현성과 공급망 안전성을 높이기 위해서입니다.

</details>

## 요약

NPM scripts는 팀의 표준 작업을 선언하고 NPX는 로컬 또는 일회성 도구 실행을 단순화합니다. 편의성과 함께 버전 재현성과 실행 코드의 신뢰성을 관리해야 합니다.
