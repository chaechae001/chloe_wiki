# Hub 배포 패키지와 모델 카드

모델 배포는 가중치 파일을 올리는 일이 아닙니다. 모델, tokenizer, config, 모델 카드가 함께 재로드되어야 다른 환경에서도 같은 추론 흐름을 재현할 수 있습니다.

**핵심 키워드:** Hub, save_pretrained, model card, safetensors, reload test

## 재사용 가능한 패키지

```text
model-package/
├── config.json
├── model.safetensors
├── tokenizer.json 또는 vocabulary 파일
├── tokenizer_config.json
├── special_tokens_map.json
└── README.md
```

모델 종류에 따라 파일은 달라질 수 있으므로 고정된 목록만 믿지 않습니다. 저장 후 로컬 폴더에서 `from_pretrained()`로 모델과 tokenizer를 모두 다시 불러오는 것이 가장 직접적인 검사입니다.

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer

model.save_pretrained(output_dir, safe_serialization=True)
tokenizer.save_pretrained(output_dir)

reloaded_model = AutoModelForSequenceClassification.from_pretrained(output_dir)
reloaded_tokenizer = AutoTokenizer.from_pretrained(output_dir)
```

대표 입력을 원본과 재로드 모델에 넣어 출력 shape, label mapping, 허용 오차를 확인합니다.

## 모델 카드에 남길 내용

| 섹션 | 핵심 질문 |
|---|---|
| Model description | 무엇을 하는 모델인가 |
| Intended use | 누구에게 어떤 용도로 허용되는가 |
| Training data | 데이터 출처와 범위는 무엇인가 |
| Evaluation | 어떤 데이터·지표로 검증했는가 |
| Limitations | 실패하기 쉬운 조건은 무엇인가 |
| License | 재사용·배포 조건은 무엇인가 |

모델 카드 front matter의 task, language, license, dataset 태그는 검색과 도구 연동에 쓰이므로 실제 내용과 일치시킵니다. 측정하지 않은 성능이나 공개할 수 없는 데이터 출처를 꾸며 쓰지 않습니다.

## 업로드 전 검증 흐름

1. 임시 로컬 디렉터리에 저장합니다.
2. 새 프로세스 관점에서 재로드합니다.
3. smoke test와 모델 카드 필수 섹션 검사를 실행합니다.
4. 공개 범위와 라이선스를 검토합니다.
5. 필요한 최소 권한으로 저장소에 업로드합니다.
6. Hub에서 다시 다운로드해 최종 smoke test를 수행합니다.

대용량 파일은 Git LFS·Xet 같은 Hub 저장 방식을 따르고, 비밀값과 개인 데이터가 artifact에 섞이지 않았는지 별도 검사합니다.

## 직접 해보기

1. 모델은 재로드되지만 tokenizer가 실패하는 원인을 두 가지 쓰세요.
2. 모델 카드의 `Limitations`에 포함할 내용을 예로 드세요.
3. 업로드 전 CI 검사를 설계하세요.

<details>
<summary>정답 보기</summary>

1. tokenizer 설정 또는 vocabulary 파일이 누락됐거나 저장된 tokenizer 클래스 정보가 호환되지 않을 수 있습니다.
2. 학습 분포 밖 입력, 지원 언어, 알려진 편향, 긴 입력 처리 제한 등을 사실에 근거해 적습니다.
3. 필수 파일·비밀값 검사, 로컬 재로드, 대표 입력 추론, 모델 카드 섹션, 라이선스·공개 범위 검사를 순서대로 실행합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 저장 성공 vs 재로드 성공 | 파일 생성 여부 vs 독립 환경 재사용 가능성 |
| config vs 모델 카드 | 실행에 필요한 구조 설정 vs 사람이 읽는 사용 설명·제약 |
| private vs gated | 허가 사용자만 보는 저장소 vs 공개 메타데이터와 접근 요청 흐름 |

## 연결되는 개념

- 이전: [지식 증류와 경량화 평가](05-distillation-and-evaluation.md)
- 다음: [인증과 Gated 모델 운영](07-auth-and-gated-models.md)
- 함께 볼 키워드: `model card`, `safetensors`, `artifact`

## 셀프 체크

- [ ] 모델과 tokenizer를 함께 저장한다.
- [ ] 로컬 재로드 테스트를 수행한다.
- [ ] 모델 카드 필수 섹션을 설명한다.
- [ ] 공개 범위와 라이선스를 검토한다.
- [ ] artifact의 비밀값을 검사한다.

### 복습 질문 및 답변

**Q1. `config.json`이 왜 필요한가요?**

<details>
<summary>답</summary>

모델 구조, label 수, 매핑 등 가중치를 올바른 클래스와 shape로 재구성하는 설정을 담기 때문입니다.

</details>

**Q2. 모델 카드가 실행 코드가 아닌데 왜 중요한가요?**

<details>
<summary>답</summary>

재사용자가 성능의 근거, 허용 용도, 제한, 라이선스를 판단하게 하는 운영 계약이기 때문입니다.

</details>

**Q3. 로컬 재로드만 통과하면 배포가 끝났나요?**

<details>
<summary>답</summary>

아닙니다. 실제 Hub 권한·파일 전송·다운로드 경로까지 확인하는 원격 smoke test가 필요합니다.

</details>

## 한 줄 정리

> 배포 패키지는 저장된 파일 묶음이 아니라 다른 환경에서 설명 가능하고 다시 실행되는 모델 자산입니다.
