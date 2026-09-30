# LangChain 핵심 구성 요소

> 좋은 LLM 앱은 한 번의 호출보다 입력·모델·출력의 경계를 명확히 나누는 데서 시작한다.

`PromptTemplate` · `Model` · `Chain` · `OutputParser` · `invoke`

## 핵심요약

- PromptTemplate은 변수 값으로 일관된 입력을 만든다.
- 모델은 생성 작업을 수행하고, Chain은 각 단계를 순서대로 연결한다.
- OutputParser는 자유 형식 응답을 앱이 쓰기 쉬운 형태로 정리한다.

## 1. 템플릿에서 구조화된 결과까지

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = PromptTemplate.from_template("{topic}을 한 문장으로 설명해줘.")
chain = prompt | model | StrOutputParser()
answer = chain.invoke({"topic": "벡터 검색"})
```

코드는 예시일 뿐이며, 실제 서비스에서는 모델 제공자 설정과 오류 처리가 추가로 필요하다.

| 구성 요소 | 역할 | 점검할 것 |
| --- | --- | --- |
| PromptTemplate | 입력 형식 재사용 | 필요한 변수 누락 여부 |
| Model | 생성 수행 | 비용, 속도, 품질 |
| Chain | 단계 연결 | 입력·출력 타입 순서 |
| OutputParser | 결과 가공 | 파싱 실패 처리 |

## 직접 해보기

1. 템플릿 변수에 값을 넣는 이유를 설명해 보세요.
2. Chain에서 순서가 중요한 이유를 적어 보세요.
3. JSON 형식 결과가 필요할 때 어떤 요소를 고려할지 답해 보세요.

<details>
<summary>정답 보기</summary>

1. 요청마다 바뀌는 값을 넣으면서도 프롬프트 구조를 일관되게 유지한다.
2. 앞 단계 출력이 다음 단계 입력이므로 타입과 의미가 맞아야 한다.
3. OutputParser와 파싱 실패 시 재시도·검증 전략을 고려한다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
| --- | --- |
| PromptTemplate vs OutputParser | 모델 입력 구성 vs 모델 출력 해석 |
| Chain vs 단일 호출 | 여러 처리 단계를 연결 vs 한 번의 생성 요청 |

## 연결되는 개념

- 이전 글: [LangChain의 역할](01-langchain-foundations.md)
- 다음 글: [Agent와 상태 관리](03-agents-tools-and-memory.md)

### 복습 질문 및 답변

**Q1. 파서가 없으면 응답을 사용할 수 없나요?**

<details>
<summary>답</summary>

텍스트 그대로는 사용할 수 있지만, API 응답이나 데이터 처리에 필요한 구조를 안정적으로 맞추기 어려울 수 있다.

</details>

**Q2. 모델을 바꾸면 Chain도 항상 바꿔야 하나요?**

<details>
<summary>답</summary>

입출력 계약이 유지되면 일부만 교체할 수 있다. 다만 모델별 호출 방식과 출력 특성은 검증해야 한다.

</details>

**Q3. 프롬프트 템플릿에 비밀값을 넣어도 되나요?**

<details>
<summary>답</summary>

아니다. 비밀값은 로그와 추적 정보에 남을 수 있으므로 별도 안전한 설정 관리가 필요하다.

</details>
