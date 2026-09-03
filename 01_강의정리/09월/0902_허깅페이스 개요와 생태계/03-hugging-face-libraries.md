# Hugging Face 핵심 라이브러리

Hugging Face는 하나의 라이브러리가 아니라 모델, 데이터, 토큰화, 학습과 평가를 연결하는 도구 생태계입니다. 각 도구의 책임을 알면 필요한 부분만 조합할 수 있습니다.

**핵심 키워드:** Transformers, Datasets, Tokenizers, Accelerate, Diffusers, PEFT

## 라이브러리 지도

| 도구 | 주된 역할 |
|---|---|
| Transformers | 사전학습 모델과 추론·학습 인터페이스 |
| Datasets | 데이터 로드, 변환, 스트리밍 |
| Tokenizers | 빠른 텍스트 토큰화 |
| Accelerate | 다양한 장치·분산 환경의 학습 실행 보조 |
| Diffusers | 확산 기반 생성 모델 파이프라인 |
| Evaluate | 평가 지표 적용과 결과 비교 보조 |
| PEFT | 일부 파라미터만 조정하는 효율적 파인튜닝 |

## 가장 작은 추론 흐름

```python
from transformers import pipeline

classifier = pipeline("text-classification", model="your-reviewed-model-id")
result = classifier("The delivery was fast and careful.")
print(result[0]["label"], result[0]["score"])
```

모델 ID는 목적·라이선스·카드를 검토한 값으로 바꿉니다. 첫 실행에는 모델 파일 다운로드가 발생할 수 있으며, 출력 점수는 사실의 확률이 아니라 해당 분류기의 예측 신뢰도입니다.

## 조합 흐름

```text
Datasets로 데이터 준비 → Tokenizers로 입력 변환
→ Transformers 모델 → Accelerate로 실행 환경 조정
→ Evaluate 또는 과업별 코드로 평가 → Hub에 기록
```

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| pipeline vs 개별 클래스 | 빠른 고수준 추론 vs 토크나이저·모델 제어 |
| Transformers vs Diffusers | 언어·비전 등 Transformer 생태계 vs 확산 생성 파이프라인 |
| 전체 파인튜닝 vs PEFT | 대부분 가중치 갱신 vs 작은 추가 파라미터 중심 학습 |

## 직접 해보기

1. 텍스트 분류 흐름에 필요한 라이브러리를 고르세요.
2. 큰 데이터셋에서 스트리밍이 필요한 이유를 설명하세요.
3. PEFT가 유리한 자원 조건을 적으세요.

<details>
<summary>정답 보기</summary>

1. 기본은 Transformers이며 데이터 처리에는 Datasets를 함께 쓸 수 있습니다.
2. 전체 데이터를 한 번에 저장·메모리에 적재하는 부담을 줄입니다.
3. GPU 메모리와 학습 시간이 제한되고 여러 과업 어댑터가 필요할 때 유리할 수 있습니다.

</details>

## 연결되는 개념

- 이전: [오픈 LLM과 자원 계획](02-open-llm-resource-planning.md)
- 다음: [Hub 저장소와 버전 관리](04-hub-repositories.md)

## 셀프 체크

- [ ] 각 라이브러리의 책임을 구분한다.
- [ ] 최소 추론 흐름을 설명한다.
- [ ] 토큰화가 필요한 이유를 안다.
- [ ] 고수준 API와 세부 제어를 구분한다.
- [ ] 평가 결과를 모델 선택과 연결한다.

### 복습 질문 및 답변

**Q1. `pipeline`은 언제 유용한가요?**

<details>
<summary>답</summary>

지원 과업의 모델을 빠르게 시험하고 입출력 형태를 확인할 때 유용합니다.

</details>

**Q2. Tokenizers가 모델과 별도로 중요한 이유는 무엇인가요?**

<details>
<summary>답</summary>

문자열을 모델이 학습한 동일한 토큰 ID 규칙으로 바꿔야 올바른 입력이 되기 때문입니다.

</details>

**Q3. 여러 라이브러리를 모두 설치해야 하나요?**

<details>
<summary>답</summary>

아닙니다. 과업과 실행 흐름에 필요한 도구만 선택하고 버전 호환성을 관리합니다.

</details>

## 한 줄 정리

> Hugging Face 라이브러리는 데이터부터 모델 실행·학습·평가까지 역할별로 조합하는 도구 상자입니다.
