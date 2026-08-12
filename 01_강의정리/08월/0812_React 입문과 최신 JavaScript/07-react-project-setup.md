# React 프로젝트 환경과 구조

React 개발 환경은 JSX 변환, 모듈 묶기, 개발 서버, 의존성 관리를 자동화해 개발자가 컴포넌트 작성에 집중하게 합니다.

## 핵심 키워드

`Node.js` · `npm` · `package.json` · `dependency` · `development server`

## 핵심 요약

- Node.js는 브라우저 밖에서 JavaScript 도구를 실행할 환경을 제공합니다.
- npm은 패키지 설치와 프로젝트 스크립트 실행을 담당합니다.
- `package.json`은 프로젝트 정보, 명령, 의존성의 기준 문서입니다.
- 프로젝트 생성 도구는 JSX 변환과 개발 서버 구성을 준비합니다.

## 1. 개발 도구의 역할

브라우저에 스크립트 링크를 직접 연결해 React를 맛볼 수 있지만 실제 프로젝트는 파일 분리, 모듈 import, CSS 처리, 배포 최적화가 필요합니다. 개발 도구가 이 과정을 묶어 제공합니다.

| 요소 | 역할 |
|---|---|
| Node.js | 빌드 도구와 개발 서버 실행 환경 |
| npm | 패키지 설치와 스크립트 실행 |
| package.json | 의존성·명령·프로젝트 메타데이터 기록 |
| 개발 서버 | 변경 감지와 빠른 브라우저 확인 |

## 2. 프로젝트의 핵심 구조

구성 도구에 따라 이름은 달라질 수 있지만 일반적으로 HTML 진입점, JavaScript 진입 파일, 루트 컴포넌트, 정적 파일, 패키지 정보가 있습니다.

```text
project/
├── package.json
├── public/
└── src/
    ├── main.jsx
    ├── App.jsx
    └── styles.css
```

`main.jsx`는 React 애플리케이션을 HTML의 루트 요소에 연결하고, `App.jsx`는 최상위 UI 구조를 담당합니다.

## 3. 의존성과 import

패키지를 설치하면 `package.json`의 의존성에 기록되고 코드에서는 모듈 문법으로 불러옵니다.

```javascript
import { useState } from "react";
import "./styles.css";
```

소스 코드만 복사하고 패키지 정보나 lock 파일을 무시하면 다른 환경에서 같은 버전을 재현하기 어렵습니다. 프로젝트 공유 시 소스와 의존성 선언을 함께 관리합니다.

## 대표 코드: 진입점 연결

### 목적

최상위 컴포넌트를 HTML의 루트 DOM에 렌더링합니다.

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const rootElement = document.getElementById("root");

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

### 코드 흐름과 결과

1. 필요한 라이브러리와 파일을 모듈로 불러옵니다.
2. HTML의 `root` 요소를 찾습니다.
3. React 루트를 생성합니다.
4. 최상위 `App` 컴포넌트를 렌더링합니다.

### 실무 연결

개발·테스트·배포 명령을 스크립트로 통일하면 팀과 다른 PC에서도 동일한 방식으로 프로젝트를 실행할 수 있습니다.

## 직접 해보기

1. Node.js와 npm의 역할을 구분하세요.
2. CSS 파일을 모듈 진입점에서 불러오는 코드를 작성하세요.
3. 의존성 버전을 기록해야 하는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. Node.js는 도구 실행 환경이고 npm은 패키지와 명령을 관리합니다.
2. `import "./styles.css";`처럼 작성할 수 있습니다.
3. 다른 PC와 배포 환경에서 호환되는 같은 패키지 조합을 재현하기 위해서입니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| Node.js vs npm | Node.js는 런타임, npm은 패키지 관리자와 명령 도구입니다. |
| dependency vs import | dependency는 설치 관계, import는 코드에서 모듈을 사용하는 문법입니다. |
| 개발 서버 vs 배포 파일 | 개발 서버는 빠른 개발용이고 배포 빌드는 최적화된 정적 결과를 만듭니다. |

## 연결되는 개념

- React의 전체 목적은 [React의 역할과 학습 방향](01-react-overview.md)에서 확인할 수 있습니다.
- 최신 문법은 [화살표 함수와 구조 분해](04-arrow-functions-and-destructuring.md)에서 다룹니다.
- 실행할 목록 예제는 [컴포넌트와 State로 목록 설계](06-components-and-state.md)에서 확인하세요.

## 셀프 체크

- [ ] Node.js와 npm을 구분할 수 있다.
- [ ] 프로젝트 진입점의 역할을 설명할 수 있다.
- [ ] package.json과 의존성의 관계를 안다.

## 복습 질문 및 답변

### Q1. JSX를 브라우저가 그대로 이해하는가?

<details>
<summary>답</summary>

일반적으로 빌드 도구가 JSX를 브라우저가 실행할 JavaScript로 변환합니다.

</details>

### Q2. package.json만 있으면 설치된 패키지 코드도 저장소에 모두 올려야 하는가?

<details>
<summary>답</summary>

보통은 의존성 선언과 lock 파일을 공유하고 패키지 폴더는 각 환경에서 다시 설치합니다.

</details>

### Q3. 프로젝트 생성 도구가 해 주는 핵심 일은?

<details>
<summary>답</summary>

모듈과 JSX 처리, 개발 서버, 빌드 명령 등 반복적인 개발 환경 구성을 준비합니다.

</details>

## 한 줄 정리

> React 프로젝트 환경은 Node.js와 npm 위에서 변환·개발 서버·의존성 관리를 표준화합니다.
