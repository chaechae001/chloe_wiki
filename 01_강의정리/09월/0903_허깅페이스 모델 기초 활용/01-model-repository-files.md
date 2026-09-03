# 모델 저장소 파일과 설정 읽기

사전학습 모델은 가중치 하나가 아니라 아키텍처 설정, 토크나이저 규칙, 특수 토큰과 생성 기본값이 함께 맞아야 동일하게 동작합니다.

**핵심 키워드:** config.json, tokenizer, special tokens, generation config, repository

## 핵심 파일

| 파일·구성 | 역할 |
|---|---|
| `config.json` | 모델 종류, hidden size, layer 수, vocabulary 크기 등 |
| 토크나이저 파일 | 문자열을 토큰 ID로 바꾸는 vocabulary와 분절 규칙 |
| 특수 토큰 설정 | BOS, EOS, PAD, UNK 등의 문자열·ID 연결 |
| generation config | temperature, top-k, 종료 토큰 같은 생성 기본값 |
| 가중치 파일 | 학습된 파라미터 텐서 |

```python
from transformers import AutoConfig

config = AutoConfig.from_pretrained("distilbert-base-uncased")
print(config.model_type)
print(config.vocab_size)
print(config.num_hidden_layers)
```

`AutoConfig`는 구조 정보만 확인할 때 유용하며 모델 가중치 전체를 메모리에 올릴 필요가 없습니다. 속성 이름은 모델 계열마다 다를 수 있으므로 `config.to_dict()`도 함께 확인합니다.

## 로딩 흐름

```text
저장소 식별자와 revision 확인 → config 읽기 → 올바른 모델 클래스 결정
→ 토크나이저 구성 → 가중치 파일 인덱스 확인 → 텐서 로드
```

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| config vs weights | 모델 구조·설정 vs 학습된 수치 텐서 |
| tokenizer config vs vocabulary | 동작 옵션 vs 토큰과 ID의 대응표 |
| model max length vs 생성 길이 | 전체 입력 허용 범위 vs 새로 생성할 토큰 수 |

## 직접 해보기

1. config만 먼저 읽는 장점을 설명하세요.
2. vocabulary 크기와 embedding 행 수의 관계를 예상하세요.
3. 재현 가능한 로딩에 기록할 값을 나열하세요.

<details>
<summary>정답 보기</summary>

1. 큰 가중치를 받기 전에 구조와 호환성을 빠르게 확인할 수 있습니다.
2. 보통 입력 embedding의 행 수는 vocabulary 크기와 대응합니다.
3. 저장소 ID, revision, 라이브러리 버전과 로딩 옵션을 기록합니다.

</details>

## 연결되는 개념

- 다음: [가중치 포맷과 샤딩](02-weight-formats-and-sharding.md)
- 함께 볼 키워드: `AutoConfig`, `vocabulary`, `revision`

## 셀프 체크

- [ ] 주요 모델 파일의 책임을 구분한다.
- [ ] config만 로드할 수 있다.
- [ ] 특수 토큰 설정의 필요성을 설명한다.
- [ ] 모델별 속성 차이를 확인한다.
- [ ] 버전 고정의 이유를 안다.

### 복습 질문 및 답변

**Q1. `config.json`만 있으면 추론할 수 있나요?**

<details>
<summary>답</summary>

아닙니다. 구조는 만들 수 있지만 학습된 동작에는 가중치와 맞는 토크나이저가 필요합니다.

</details>

**Q2. 다른 모델의 토크나이저를 섞으면 왜 문제가 되나요?**

<details>
<summary>답</summary>

같은 텍스트가 모델이 학습하지 않은 ID 배열로 변환되어 입력 의미가 깨질 수 있습니다.

</details>

**Q3. 생성 설정은 항상 고정인가요?**

<details>
<summary>답</summary>

저장된 값은 기본값이며 호출 시 목적에 맞게 재정의할 수 있지만 재현을 위해 실제 옵션을 기록해야 합니다.

</details>

## 한 줄 정리

> 모델 저장소는 구조·토큰화·가중치·생성 설정이 함께 움직이는 실행 가능한 묶음입니다.
