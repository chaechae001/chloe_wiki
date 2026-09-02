# NPM 프로젝트와 의존성 관리

NPM은 패키지를 내려받는 도구를 넘어 프로젝트 메타데이터와 재현 가능한 의존성 설치를 관리합니다.

**핵심 키워드:** npm, package.json, dependencies, devDependencies, lockfile

## 핵심 내용

- NPM은 온라인 패키지 저장소와 명령줄 도구를 함께 가리킵니다.
- `package.json`은 프로젝트 정보, 의존성, 스크립트와 실행 규칙을 기록합니다.
- 실행에 필요한 패키지는 `dependencies`, 개발 과정에만 필요한 도구는 `devDependencies`로 구분합니다.
- `package-lock.json`은 실제 해결된 의존성 트리를 고정해 재현성을 높입니다.
- 애플리케이션에서는 lockfile과 `npm ci`를 활용해 CI 설치 결과를 일관되게 유지합니다.

## 프로젝트 초기화와 설치

```bash
npm init -y
npm install express
npm install --save-dev eslint
npm remove express
```

- **목적:** 프로젝트 메타데이터를 만들고 용도별 의존성을 관리합니다.
- **흐름:** 초기화 → 런타임 의존성 설치 → 개발 의존성 설치 → 불필요 패키지 제거입니다.
- **결과:** manifest와 lockfile, 로컬 패키지 폴더가 함께 갱신됩니다.
- **실무 포인트:** `node_modules`는 커밋하지 않고 `package.json`과 lockfile을 커밋합니다.

## 두 의존성 구분

| 구분 | 사용 시점 | 예시 |
|---|---|---|
| `dependencies` | 애플리케이션 실행 | 웹 프레임워크, 데이터베이스 클라이언트 |
| `devDependencies` | 개발·검사·빌드 | 테스트 도구, 린터, 포매터 |

빌드 결과만 배포하는지, 서버에서 직접 빌드하는지에 따라 필요한 패키지가 달라질 수 있으므로 배포 파이프라인과 함께 판단합니다.

## 버전 범위와 lockfile

`package.json`의 버전 범위는 허용 가능한 업데이트를 표현하고, lockfile은 특정 설치에서 선택된 정확한 버전과 하위 의존성을 기록합니다. 둘은 경쟁 관계가 아니라 선언과 재현을 담당하는 서로 다른 문서입니다.

```bash
npm install
npm ci
```

`npm install`은 의존성을 해석하며 manifest와 lockfile을 갱신할 수 있습니다. `npm ci`는 기존 lockfile을 기준으로 깨끗하게 설치하고 불일치 시 실패하므로 자동화 환경에 적합합니다.

## 로컬과 전역 설치

프로젝트 의존성은 기본적으로 로컬에 설치해 팀과 버전을 공유합니다. 전역 설치는 모든 프로젝트에서 쓰는 사용자 도구에 편리할 수 있지만 프로젝트별 버전 차이를 숨길 수 있습니다. 가능하면 로컬 설치와 npm script 또는 NPX를 우선 검토합니다.

## 실습

1. 프로젝트를 초기화하고 런타임·개발 의존성을 하나씩 설치하세요.
2. `package.json`과 lockfile의 역할 차이를 설명하세요.
3. CI에서 `npm ci`가 적합한 이유를 적으세요.

<details>
<summary>답</summary>

```bash
npm init -y
npm install express
npm install --save-dev eslint
npm ci
```

`package.json`은 허용 범위와 프로젝트 구성을 선언하고 lockfile은 실제 해석된 트리를 고정합니다. `npm ci`는 lockfile 불일치를 감지하고 재현 가능한 설치를 제공합니다.

</details>

## 더 알아보기

- [NPM 스크립트와 NPX](05-npm-scripts-and-npx.md)
- [CommonJS와 Node.js 기본 모듈](06-commonjs-and-node-core-modules.md)

## 체크리스트

- [ ] package.json의 역할을 설명할 수 있다.
- [ ] 실행·개발 의존성을 구분한다.
- [ ] lockfile을 저장소에 포함한다.
- [ ] node_modules를 추적하지 않는다.
- [ ] CI에서 설치 방식을 명시한다.

## 복습 질문 및 답변

### Q1. lockfile이 있으면 package.json은 필요 없나요?

<details>
<summary>답</summary>

둘 다 필요합니다. package.json은 직접 의존성과 허용 범위, 스크립트를 선언하고 lockfile은 해석된 전체 트리를 고정합니다.

</details>

### Q2. 모든 패키지를 전역으로 설치하면 편하지 않나요?

<details>
<summary>답</summary>

환경마다 버전이 달라 재현성이 떨어질 수 있습니다. 프로젝트 도구는 로컬 의존성으로 관리하고 script나 NPX로 실행하는 편이 일반적으로 안전합니다.

</details>

### Q3. devDependency는 운영에 절대 필요하지 않나요?

<details>
<summary>답</summary>

운영 런타임에는 보통 필요하지 않지만 운영 환경에서 빌드한다면 빌드 도구가 필요할 수 있습니다. 배포 단계에 맞춰 구분해야 합니다.

</details>

## 요약

NPM 의존성 관리는 설치 명령보다 선언과 재현성이 핵심입니다. manifest, lockfile, 로컬 설치와 CI 정책을 함께 관리해야 어디서든 같은 프로젝트를 실행할 수 있습니다.
