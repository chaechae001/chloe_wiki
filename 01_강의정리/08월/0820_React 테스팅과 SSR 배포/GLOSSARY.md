# 용어집

이번 학습에서 등장한 React 테스트, 렌더링, 빌드와 배포 용어를 쉬운 말로 정리했습니다.

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| Unit Test | 함수나 컴포넌트처럼 작은 단위를 분리해 검증하는 테스트 | [React 테스트 전략](01-testing-strategy.md) |
| Integration Test | 여러 컴포넌트와 상태·라우팅의 연결을 검증하는 테스트 | [React 테스트 전략](01-testing-strategy.md) |
| Mock | 실제 의존성을 대신해 결과와 호출을 제어·관찰하는 테스트 대역 | [Jest 핵심 기능](02-jest-core-features.md) |
| Matcher | 실제 결과가 기대 조건을 만족하는지 표현하는 검증 함수 | [Jest 핵심 기능](02-jest-core-features.md) |
| Snapshot | 직렬화한 이전 출력과 현재 출력을 비교하는 테스트 결과 | [Jest 핵심 기능](02-jest-core-features.md) |
| Accessible Name | 보조 기술이 요소를 식별할 때 사용하는 이름 | [Testing Library와 사용자 이벤트](03-testing-library-user-events.md) |
| CSR | 브라우저가 JavaScript와 데이터로 화면을 만드는 렌더링 방식 | [CSR과 SSR 성능](04-csr-ssr-performance.md) |
| SSR | 서버가 요청 시 초기 HTML을 만들어 보내는 렌더링 방식 | [CSR과 SSR 성능](04-csr-ssr-performance.md) |
| TTFB | 요청 후 첫 응답 바이트가 도착할 때까지의 시간 | [CSR과 SSR 성능](04-csr-ssr-performance.md) |
| FCP | 첫 콘텐츠가 화면에 그려질 때까지의 시간 | [CSR과 SSR 성능](04-csr-ssr-performance.md) |
| TTI | 주요 사용자 입력에 안정적으로 반응할 준비가 될 때까지의 시간 | [CSR과 SSR 성능](04-csr-ssr-performance.md) |
| Hydration | 서버 HTML에 React 상태와 이벤트 처리를 연결하는 과정 | [Hydration과 React SSR](05-hydration-and-react-ssr.md) |
| Production Build | 소스 코드를 최적화된 배포 산출물로 변환하는 과정 | [React 프로덕션 빌드](06-react-production-build.md) |
| Lockfile | 동일한 의존성 버전을 다시 설치하도록 고정한 파일 | [React 프로덕션 빌드](06-react-production-build.md) |
| SPA Fallback | 서버에 실제 파일이 없는 앱 경로를 진입 HTML로 보내는 설정 | [React 프로덕션 빌드](06-react-production-build.md) |
| VM | 클라우드나 호스트 위에서 독립 서버처럼 동작하는 가상 컴퓨터 | [웹서버와 VM 배포 운영](07-web-server-vm-deployment.md) |
| Reverse Proxy | 외부 요청을 받아 내부 앱 서버나 정적 파일로 전달하는 서버 역할 | [웹서버와 VM 배포 운영](07-web-server-vm-deployment.md) |
