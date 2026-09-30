# LangChain 용어집

| 용어 | 쉬운 설명 | 관련 글 |
| --- | --- | --- |
| LangChain | LLM 앱의 입력·모델·도구·출력 흐름을 조합하는 프레임워크 | [기본 흐름](01-langchain-foundations.md) |
| PromptTemplate | 변수를 넣어 일정한 형태의 프롬프트를 만드는 템플릿 | [구성 요소](02-langchain-components.md) |
| Chain | 여러 구성 요소를 순서대로 연결한 처리 흐름 | [구성 요소](02-langchain-components.md) |
| OutputParser | 모델 응답을 텍스트·구조화 데이터 등 원하는 형태로 가공하는 도구 | [구성 요소](02-langchain-components.md) |
| Agent | 목표에 맞게 도구 사용을 판단하는 실행 흐름 | [Agent와 Tool](03-agents-tools-and-memory.md) |
| Tool | 검색·계산·API 호출처럼 외부 기능을 수행하는 인터페이스 | [Agent와 Tool](03-agents-tools-and-memory.md) |
| Memory | 대화나 작업에 필요한 상태를 저장·활용하는 방식 | [Agent와 Tool](03-agents-tools-and-memory.md) |
| RAG | 검색한 문서 근거를 LLM 입력에 넣어 답하는 패턴 | [RAG](04-rag-and-use-cases.md) |
| Retriever | 질문과 관련된 문서 조각을 찾는 구성 요소 | [RAG](04-rag-and-use-cases.md) |
| Callback | 실행 단계의 이벤트를 받아 로그·추적에 쓰는 연결 지점 | [운영 검증](05-observability-and-limitations.md) |
| Tracing | 한 요청이 여러 단계를 거치는 흐름을 연결해 관찰하는 방식 | [운영 검증](05-observability-and-limitations.md) |
