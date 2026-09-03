# 가중치 포맷과 샤딩

가중치 파일은 수많은 이름 있는 텐서를 저장합니다. 포맷은 로딩 안전성과 호환성에 영향을 주고, 샤딩은 큰 체크포인트를 여러 파일로 나눠 저장·전송하게 합니다.

**핵심 키워드:** checkpoint, safetensors, pickle, ONNX, shard, index

## 포맷 비교

| 포맷 | 특징 | 주의점 |
|---|---|---|
| PyTorch checkpoint | PyTorch 생태계와 유연하게 연동 | 신뢰하지 않는 pickle 기반 파일 실행 주의 |
| safetensors | 텐서 전용 구조, 빠른 부분 읽기와 안전한 역직렬화 | 지원 도구와 메타데이터 확인 |
| ONNX | 프레임워크 간 추론 그래프 교환 | 변환 후 연산 지원·수치 차이 검증 |
| HDF5 | 계층형 데이터 저장 | 사용하는 프레임워크·버전 호환성 확인 |

## 샤딩 구조

```text
model.safetensors.index.json
├─ parameter A → model-00001-of-00003.safetensors
├─ parameter B → model-00002-of-00003.safetensors
└─ parameter C → model-00003-of-00003.safetensors
```

인덱스는 파라미터 이름이 어느 shard에 있는지 알려줍니다. 샤딩은 모델을 자동으로 여러 GPU에 나누는 것과 같지 않습니다. 파일 분할은 저장 문제이고, 장치 배치는 실행 문제입니다.

```python
model.save_pretrained(
    "./exported-model",
    safe_serialization=True,
    max_shard_size="2GB",
)
```

## 안전한 선택

공개 저장소에서는 파일 출처와 revision을 확인하고 가능하면 텐서 전용 포맷을 선호합니다. 포맷 변환 뒤에는 같은 입력에 대한 출력 오차와 누락된 키를 검증합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| 샤딩 vs 모델 병렬 | 파일 분할 vs 계산을 여러 장치에 배치 |
| checkpoint vs model object | 저장된 상태 vs 메모리에 생성된 실행 객체 |
| 직렬화 안전성 vs 모델 안전성 | 파일 로딩 코드 위험 vs 모델 출력·편향 위험 |

## 직접 해보기

1. shard index의 역할을 설명하세요.
2. 포맷 변환 후 검증 항목을 작성하세요.
3. 낯선 checkpoint를 다룰 안전 절차를 세우세요.

<details>
<summary>정답 보기</summary>

1. 각 파라미터가 저장된 조각 파일을 매핑합니다.
2. 누락·예상 밖 키, dtype, 출력 오차와 성능을 확인합니다.
3. 출처·해시·revision을 확인하고 격리 환경과 안전한 로더를 사용합니다.

</details>

## 연결되는 개념

- 이전: [모델 저장소 파일과 설정 읽기](01-model-repository-files.md)
- 다음: [Auto 클래스와 모델 로딩](03-auto-classes-and-loading.md)

## 셀프 체크

- [ ] 대표 포맷의 차이를 설명한다.
- [ ] shard와 index 관계를 이해한다.
- [ ] 파일 분할과 장치 배치를 구분한다.
- [ ] 안전하지 않은 역직렬화 위험을 안다.
- [ ] 변환 결과를 검증한다.

### 복습 질문 및 답변

**Q1. shard가 여러 개면 반드시 GPU도 여러 개 필요한가요?**

<details>
<summary>답</summary>

아닙니다. shard는 저장 단위이며 실행 장치 수와 직접 일치하지 않습니다.

</details>

**Q2. safetensors를 쓰면 모델 출력도 안전한가요?**

<details>
<summary>답</summary>

아닙니다. 로딩 공격 면적을 줄이는 포맷이며 출력 품질과 유해성은 별도 평가가 필요합니다.

</details>

**Q3. 포맷 변환에서 dtype을 확인하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

정밀도 변경이 메모리와 수치 결과에 영향을 줄 수 있기 때문입니다.

</details>

## 한 줄 정리

> 가중치 포맷은 안전한 저장·로딩을, 샤딩은 큰 체크포인트의 관리 가능성을 좌우합니다.
