# 허깅페이스 모델 기초 활용

모델 저장소의 파일 구조에서 시작해 가중치 포맷·샤딩, Auto 클래스 로딩, 파라미터 분석, 토큰화·패딩·offset과 채팅 템플릿까지 모델 입력이 추론 준비 상태가 되는 과정을 학습합니다.

## 학습 목표

- 설정·토크나이저·가중치 파일의 역할을 구분합니다.
- checkpoint 포맷과 샤딩 구조를 안전성·운영 관점에서 이해합니다.
- 과업별 Auto 클래스를 선택하고 출력 shape를 해석합니다.
- `state_dict`로 계층별·학습 가능 파라미터를 집계합니다.
- 토큰화, padding, truncation, offset과 chat template을 적용합니다.

## 추천 학습 순서

1. [모델 저장소 파일과 설정 읽기](01-model-repository-files.md)
2. [가중치 포맷과 샤딩](02-weight-formats-and-sharding.md)
3. [Auto 클래스와 모델 로딩](03-auto-classes-and-loading.md)
4. [state_dict와 파라미터 분석](04-state-dict-parameter-analysis.md)
5. [토큰화와 offset mapping](05-tokenization-and-offsets.md)
6. [배치, 패딩과 트런케이션](06-padding-truncation-attention-mask.md)
7. [채팅 템플릿과 대화 입력](07-chat-templates.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
저장소·revision 선택 → config로 아키텍처 확인 → 가중치 로드
→ state_dict로 구조·크기 분석 → 토크나이저로 입력 ID 생성
→ padding·truncation·mask 구성 → chat template 적용 → 모델 추론
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| config와 토크나이저 파일 | 01 저장소 구조 |
| safetensors와 shard index | 02 가중치 포맷 |
| from_pretrained와 과업 클래스 | 03 Auto 클래스 |
| shape, numel, trainable | 04 파라미터 분석 |
| subword, 특수 토큰, offset | 05 토큰화 |
| batch shape와 mask | 06 입력 길이 정규화 |
| roles와 apply_chat_template | 07 채팅 템플릿 |

## 최종 점검

- [ ] 모델과 토크나이저의 저장소·revision을 맞춘다.
- [ ] 파일 포맷과 원격 코드의 신뢰 경계를 확인한다.
- [ ] 파라미터 수와 학습 가능 비율을 해석한다.
- [ ] padding·truncation으로 생기는 변화를 검증한다.
- [ ] 모델별 채팅 템플릿과 특수 토큰을 사용한다.
