# 파인튜닝 운영과 재현성

파인튜닝은 `train()` 호출로 끝나지 않습니다. 기준선, 데이터 revision, 실험 설정, checkpoint, 최종 평가와 배포 자산을 연결해야 결과를 다시 만들고 비교할 수 있습니다.

**핵심 키워드:** fine-tuning, baseline, learning rate, mixed precision, reproducibility

## 전체 흐름

```text
목표·기준선 정의 → 데이터 품질·split 검증 → 전처리 고정
→ 작은 overfit 시험 → 본 학습·중간 평가 → best checkpoint 선택
→ test 1회 평가 → 모델·토크나이저·카드 저장 → 배포 검증
```

## 주요 설정

| 설정 | 영향 | 확인할 것 |
|---|---|---|
| learning rate | 수렴 속도·안정성 | 너무 크면 발산, 작으면 느린 학습 |
| batch·accumulation | 메모리·gradient 통계 | 유효 batch와 step 수 |
| scheduler·warmup | 초반·후반 갱신 크기 | 총 step과 비율 |
| mixed precision | 속도·메모리 | 장치 지원과 수치 안정성 |
| save limit | 디스크 사용 | best·last checkpoint 보존 |

```python
metrics = trainer.evaluate()
trainer.save_model("./artifacts/final-model")
tokenizer.save_pretrained("./artifacts/final-model")
print(metrics)
```

공개 전에 학습 데이터의 권리와 개인정보, 기반 모델 라이선스, 파생 모델 조건을 확인합니다. 실제 토큰과 비밀은 코드·로그·노트북 출력에서 제거합니다.

## 재현성 기록

모델·데이터 revision, 코드 commit, 패키지·CUDA 버전, random seed, 모든 TrainingArguments, 하드웨어와 최종 checkpoint를 기록합니다. GPU 연산은 seed를 같게 해도 완전 결정적이지 않을 수 있으므로 허용 오차를 둡니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| baseline vs best model | 개선 전 비교 기준 vs 검증으로 선택된 결과 |
| validation vs test | 설정 선택에 반복 사용 vs 최종 보고에 제한 사용 |
| reproducible vs identical | 조건을 추적·재현 가능 vs bit 단위 완전 동일 |

## 직접 해보기

1. 작은 overfit 시험의 목적을 설명하세요.
2. 실험 메타데이터 목록을 작성하세요.
3. 배포 전 평가 관문을 설계하세요.

<details>
<summary>정답 보기</summary>

1. 작은 batch를 거의 외우지 못한다면 데이터·loss·optimizer 연결 오류를 빠르게 찾을 수 있습니다.
2. revision, commit, 패키지, seed, 설정, 하드웨어, 지표와 checkpoint를 기록합니다.
3. 품질·안전·편향·지연·메모리·비용 기준을 통과한 모델만 승격합니다.

</details>

## 연결되는 개념

- 이전: [Trainer API 학습 흐름](06-trainer-api-workflow.md)
- 처음으로: [사전학습 백본에 커스텀 헤드 추가하기](01-custom-model-heads.md)

## 셀프 체크

- [ ] 학습 전 기준선을 만든다.
- [ ] 작은 overfit 시험을 수행한다.
- [ ] validation과 test를 분리한다.
- [ ] 실험 환경을 기록한다.
- [ ] 라이선스·개인정보를 검토한다.

### 복습 질문 및 답변

**Q1. test 점수가 가장 좋은 설정을 선택하면 왜 문제인가요?**

<details>
<summary>답</summary>

test 정보가 선택 과정에 누출되어 최종 일반화 성능이 과대평가됩니다.

</details>

**Q2. best checkpoint와 last checkpoint는 같은가요?**

<details>
<summary>답</summary>

마지막 학습 시점이 validation 지표 최적 시점과 다를 수 있으므로 같지 않을 수 있습니다.

</details>

**Q3. 모델만 저장하면 재현 가능한가요?**

<details>
<summary>답</summary>

아닙니다. 토크나이저, config, 데이터·코드 버전과 학습 설정도 필요합니다.

</details>

## 한 줄 정리

> 파인튜닝의 결과물은 가중치 하나가 아니라 데이터·설정·평가·환경을 추적할 수 있는 실험 기록입니다.
