# 용어집

React 입문과 최신 JavaScript 문법에서 반복되는 핵심 용어를 정리했습니다.

## React와 UI

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| React | 데이터에 따라 UI를 컴포넌트로 구성하는 JavaScript 라이브러리입니다. | [React의 역할과 학습 방향](01-react-overview.md) | 선언형 UI |
| 컴포넌트 | 재사용하고 조립할 수 있는 독립적인 UI 단위입니다. | [React의 역할과 학습 방향](01-react-overview.md) | Props, State |
| JSX | JavaScript 안에서 HTML과 유사한 형태로 UI를 표현하는 문법입니다. | [React의 역할과 학습 방향](01-react-overview.md) | 표현식 |
| 선언형 UI | 변경 절차보다 현재 데이터가 만들어야 할 화면 결과를 기술하는 방식입니다. | [명령형 DOM과 선언형 UI](02-imperative-vs-declarative-ui.md) | 렌더링 |
| Props | 부모 컴포넌트가 자식에게 전달하는 읽기 전용 입력입니다. | [컴포넌트와 State로 목록 설계](06-components-and-state.md) | 단방향 데이터 흐름 |
| State | 컴포넌트가 렌더링 사이에 기억하고 setter로 갱신하는 값입니다. | [컴포넌트와 State로 목록 설계](06-components-and-state.md) | 재렌더링 |
| key | 목록 항목의 정체성을 React가 비교하도록 돕는 안정적인 식별값입니다. | [컴포넌트와 State로 목록 설계](06-components-and-state.md) | 목록 렌더링 |

## 최신 JavaScript

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `const` | 식별자에 다른 값을 재할당하지 못하게 선언하는 블록 범위 변수입니다. | [변수 선언과 배열 메서드](03-variables-and-array-methods.md) | `let` |
| `map` | 배열의 각 항목을 변환해 새 배열을 만드는 메서드입니다. | [변수 선언과 배열 메서드](03-variables-and-array-methods.md) | JSX 목록 |
| `filter` | 조건을 만족하는 항목만 골라 새 배열을 만드는 메서드입니다. | [변수 선언과 배열 메서드](03-variables-and-array-methods.md) | 파생 데이터 |
| 화살표 함수 | 함수 표현식을 간결하게 쓰고 주변 환경의 `this`를 사용하는 문법입니다. | [화살표 함수와 구조 분해](04-arrow-functions-and-destructuring.md) | 콜백 |
| 구조 분해 | 객체는 이름, 배열은 위치를 기준으로 필요한 값을 꺼내는 문법입니다. | [화살표 함수와 구조 분해](04-arrow-functions-and-destructuring.md) | 기본값, 별칭 |
| 전개 구문 | 배열 요소나 객체 프로퍼티를 펼쳐 새 컨테이너를 조합하는 문법입니다. | [전개 구문과 안전한 값 접근](05-spread-template-and-optional-chaining.md) | 얕은 복사 |
| 템플릿 리터럴 | 백틱 안에서 표현식을 삽입하고 여러 줄 문자열을 만드는 문법입니다. | [전개 구문과 안전한 값 접근](05-spread-template-and-optional-chaining.md) | 문자열 보간 |
| 옵셔널 체이닝 | 중간 값이 nullish일 때 오류 대신 undefined를 반환하며 접근을 멈추는 문법입니다. | [전개 구문과 안전한 값 접근](05-spread-template-and-optional-chaining.md) | `?.` |

## 프로젝트 환경

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Node.js | 브라우저 밖에서 JavaScript와 개발 도구를 실행하는 런타임입니다. | [React 프로젝트 환경과 구조](07-react-project-setup.md) | 개발 서버 |
| npm | JavaScript 패키지 설치와 프로젝트 명령 실행을 관리하는 도구입니다. | [React 프로젝트 환경과 구조](07-react-project-setup.md) | package.json |
| 의존성 | 프로젝트가 실행되거나 개발될 때 필요로 하는 외부 패키지 관계입니다. | [React 프로젝트 환경과 구조](07-react-project-setup.md) | 버전, lock 파일 |

## 빠른 비교

| 비교 | 핵심 차이 |
|---|---|
| Props vs State | Props는 외부 입력, State는 컴포넌트가 기억하고 갱신하는 값입니다. |
| map vs filter | map은 변환, filter는 선택을 위한 새 배열을 만듭니다. |
| Node.js vs npm | Node.js는 실행 환경이고 npm은 패키지·명령 관리 도구입니다. |
| 명령형 vs 선언형 | 명령형은 변경 절차, 선언형은 상태에 따른 결과를 기술합니다. |
