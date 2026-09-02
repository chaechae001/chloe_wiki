# GLOSSARY

## 용어 정리

| 용어 | 설명 |
|---|---|
| Runtime | 특정 언어의 코드를 실행하고 시스템 기능을 제공하는 환경 |
| V8 | JavaScript를 실행하는 엔진으로 Node.js의 핵심 구성 요소 |
| Call Stack | 현재 실행 중인 JavaScript 함수 호출을 관리하는 구조 |
| Event Loop | 준비된 비동기 작업의 실행 시점을 조정하는 메커니즘 |
| Task Queue | 실행 가능한 콜백이 대기하는 큐의 일반적 표현 |
| Microtask | Promise 후속 작업처럼 높은 우선순위로 처리되는 작업 |
| Callback | 다른 작업이 끝난 뒤 실행하도록 전달하는 함수 |
| Promise | 비동기 작업의 대기·성공·실패 상태를 표현하는 객체 |
| async/await | Promise 기반 코드를 순차적인 형태로 작성하는 문법 |
| NPM | JavaScript 패키지 저장소와 패키지 관리 CLI 생태계 |
| package.json | 프로젝트 정보, 의존성, 스크립트와 설정을 선언하는 파일 |
| package-lock.json | 설치된 전체 의존성 트리의 정확한 버전을 기록하는 lockfile |
| dependencies | 애플리케이션 실행에 필요한 패키지 목록 |
| devDependencies | 개발, 테스트, 검사, 빌드 과정에 필요한 패키지 목록 |
| npm ci | lockfile을 기준으로 깨끗하고 재현 가능하게 설치하는 명령 |
| npm script | package.json에 이름을 붙여 저장한 프로젝트 명령 |
| NPX | 패키지가 제공하는 실행 파일을 호출하는 도구 |
| Module | 관련 구현을 캡슐화하고 공개 API를 제공하는 코드 단위 |
| Package | 배포 가능한 모듈과 메타데이터의 묶음 |
| CommonJS | require와 module.exports를 사용하는 Node.js 모듈 체계 |
| ES Module | import와 export를 사용하는 JavaScript 표준 모듈 체계 |
| Core Module | Node.js가 별도 설치 없이 제공하는 기본 모듈 |
| module.exports | CommonJS 모듈이 외부에 반환할 최종 값 |
| import.meta.url | 현재 ES Module의 위치를 URL로 나타내는 값 |
| LTS | 장기 지원을 제공해 운영 안정성을 중시하는 릴리스 계열 |

## 연결해서 기억하기

Node.js 런타임은 이벤트 루프로 비동기 작업을 조정하고, NPM은 프로젝트와 외부 패키지를 관리합니다. 코드는 CommonJS 또는 ES Module로 책임을 나누며, package.json은 의존성과 실행 스크립트뿐 아니라 모듈 해석 방식에도 영향을 줍니다.

## 관련 학습

- [Node.js 런타임과 실행 구조](01-nodejs-runtime-and-architecture.md)
- [이벤트 루프와 작업 큐](03-event-loop-and-task-queues.md)
- [NPM 프로젝트와 의존성 관리](04-npm-projects-and-dependencies.md)
- [ES Module과 모듈 선택](07-es-modules-and-module-selection.md)
