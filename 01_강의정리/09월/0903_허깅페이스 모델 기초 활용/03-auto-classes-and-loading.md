# Auto 클래스와 모델 로딩

Auto 클래스는 저장소의 config를 읽어 알맞은 토크나이저와 모델 구현을 선택합니다. 과업에 맞는 모델 클래스를 고르는 것이 출력 head와 반환값을 결정합니다.

**핵심 키워드:** AutoTokenizer, AutoModel, from_pretrained, task head, cache

## 클래스 선택

| 클래스 | 사용 목적 |
|---|---|
| `AutoModel` | 과업 head 없는 기본 표현 모델 |
| `AutoModelForSequenceClassification` | 문장·문서 분류 |
| `AutoModelForTokenClassification` | 토큰 단위 분류 |
| `AutoModelForCausalLM` | 다음 토큰 생성 |

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer

model_id = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSequenceClassification.from_pretrained(model_id)

batch = tokenizer(["A clear explanation."], return_tensors="pt")
outputs = model(**batch)
print(outputs.logits.shape)
```

배치 1개와 라벨 수 2개인 모델이라면 shape은 일반적으로 `(1, 2)`입니다. 점수 의미는 config의 `id2label`로 확인합니다.

## 로딩 시 확인할 것

- 모델과 토크나이저에 같은 저장소·revision을 사용합니다.
- `dtype`과 `device_map`은 하드웨어와 모델 지원 범위에 맞춥니다.
- 원격 사용자 코드는 필요성과 내용을 검토한 뒤에만 허용합니다.
- 캐시는 다운로드를 줄이지만 디스크 용량과 오래된 revision을 관리해야 합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| `AutoModel` vs 과업 클래스 | hidden state 출력 vs 작업용 head가 포함된 출력 |
| download vs load | 파일을 로컬에 받기 vs 메모리·장치에 모델 구성 |
| cache vs revision | 로컬 재사용 위치 vs 원격 자원의 정확한 버전 |

## 직접 해보기

1. 감성 분류에 적합한 Auto 클래스를 고르세요.
2. logits의 마지막 차원이 뜻하는 바를 설명하세요.
3. 운영 로딩 옵션 체크리스트를 만드세요.

<details>
<summary>정답 보기</summary>

1. `AutoModelForSequenceClassification`을 사용합니다.
2. 분류 모델에서는 보통 라벨별 점수의 개수입니다.
3. revision, dtype, 장치 배치, 원격 코드, 메모리와 캐시 정책을 확인합니다.

</details>

## 연결되는 개념

- 이전: [가중치 포맷과 샤딩](02-weight-formats-and-sharding.md)
- 다음: [state_dict와 파라미터 분석](04-state-dict-parameter-analysis.md)

## 셀프 체크

- [ ] 과업별 Auto 클래스를 선택한다.
- [ ] config가 클래스 선택에 쓰임을 안다.
- [ ] logits shape를 해석한다.
- [ ] 모델과 토크나이저 버전을 맞춘다.
- [ ] 원격 코드 실행을 검토한다.

### 복습 질문 및 답변

**Q1. 분류 모델에 `AutoModel`만 쓰면 무엇이 부족할 수 있나요?**

<details>
<summary>답</summary>

라벨 점수를 만드는 분류 head가 없어 직접 head와 후처리를 구성해야 할 수 있습니다.

</details>

**Q2. 첫 실행이 느리고 다음 실행이 빠른 이유는 무엇인가요?**

<details>
<summary>답</summary>

첫 실행에서 파일을 다운로드하고 이후에는 로컬 캐시를 재사용하기 때문일 수 있습니다.

</details>

**Q3. 모델과 토크나이저 revision이 다르면 어떤 문제가 있나요?**

<details>
<summary>답</summary>

vocabulary나 특수 토큰 규칙이 어긋나 입력 ID와 embedding이 맞지 않을 수 있습니다.

</details>

## 한 줄 정리

> Auto 클래스는 config를 바탕으로 구현을 고르며, 과업 head와 버전·장치 옵션은 사용자가 검증해야 합니다.
