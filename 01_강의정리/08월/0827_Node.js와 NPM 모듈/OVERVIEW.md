# Node.js와 NPM 모듈

Node.js의 실행 구조와 비동기 처리부터 NPM 의존성 관리, 프로젝트 스크립트, CommonJS와 ES Module까지 서버 JavaScript의 기반을 학습합니다.

## 학습 목표

- Node.js 런타임과 싱글 스레드 비동기 구조를 설명합니다.
- Callback, Promise와 async/await를 상황에 맞게 사용합니다.
- 이벤트 루프의 콜 스택과 작업 큐 관계를 이해합니다.
- NPM manifest와 lockfile로 재현 가능한 의존성을 관리합니다.
- CommonJS와 ES Module 중 프로젝트에 맞는 체계를 선택합니다.

## 추천 학습 순서

1. [Node.js 런타임과 실행 구조](01-nodejs-runtime-and-architecture.md)
2. [비동기 프로그래밍 패턴](02-asynchronous-programming-patterns.md)
3. [이벤트 루프와 작업 큐](03-event-loop-and-task-queues.md)
4. [NPM 프로젝트와 의존성 관리](04-npm-projects-and-dependencies.md)
5. [NPM 스크립트와 NPX](05-npm-scripts-and-npx.md)
6. [CommonJS와 Node.js 기본 모듈](06-commonjs-and-node-core-modules.md)
7. [ES Module과 모듈 선택](07-es-modules-and-module-selection.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
Node.js 런타임 이해 → 비동기 코드 작성 → 이벤트 루프 추론
→ NPM 프로젝트 초기화 → 의존성과 스크립트 관리
→ 기본·외부·로컬 모듈 조합 → 모듈 체계 선택
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| Node.js와 브라우저의 차이 | 01 Node.js 런타임과 실행 구조 |
| Callback, Promise, async/await | 02 비동기 프로그래밍 패턴 |
| 콜 스택, 마이크로태스크, 타이머 | 03 이벤트 루프와 작업 큐 |
| package.json, lockfile, npm ci | 04 NPM 프로젝트와 의존성 관리 |
| scripts, npm run, NPX | 05 NPM 스크립트와 NPX |
| require, module.exports, 기본 모듈 | 06 CommonJS와 Node.js 기본 모듈 |
| import, export, package type | 07 ES Module과 모듈 선택 |

## 최종 점검

- [ ] Node.js가 서버 측 JavaScript 런타임임을 설명한다.
- [ ] 비동기 흐름과 실패 경로를 명확하게 작성한다.
- [ ] lockfile과 로컬 의존성으로 환경을 재현한다.
- [ ] npm script에 팀의 표준 명령을 담는다.
- [ ] 모듈 체계와 실행 환경을 일관되게 설정한다.
