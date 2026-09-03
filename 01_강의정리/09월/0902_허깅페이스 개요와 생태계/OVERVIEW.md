# 허깅페이스 개요와 생태계

클라우드 API와 로컬 LLM의 선택부터 Hugging Face 라이브러리, Hub의 모델·데이터셋·Spaces, 모델 카드와 평가 지표까지 오픈 ML 자원을 책임 있게 찾고 검증하는 흐름을 학습합니다.

## 학습 목표

- API와 로컬 추론의 데이터·비용·운영 책임을 비교합니다.
- 모델 규모와 실행 자원을 거칠게 계산하고 라이선스를 확인합니다.
- Hugging Face 핵심 라이브러리의 역할을 구분합니다.
- Hub 저장소에서 버전과 신뢰 정보를 확인합니다.
- 모델 카드와 성능 지표를 실제 사용 맥락에서 해석합니다.

## 추천 학습 순서

1. [LLM API와 로컬 추론 선택](01-api-and-local-llm.md)
2. [오픈 LLM과 자원 계획](02-open-llm-resource-planning.md)
3. [Hugging Face 핵심 라이브러리](03-hugging-face-libraries.md)
4. [Hub 저장소와 버전 관리](04-hub-repositories.md)
5. [Model Hub와 Dataset Hub 탐색](05-model-dataset-discovery.md)
6. [Spaces와 생태계 연결](06-spaces-and-ecosystem.md)
7. [모델 카드와 평가 지표 해석](07-model-cards-and-evaluation.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
사용 사례·데이터 경계 정의 → API/로컬 후보 결정 → 자원·라이선스 검토
→ Hub 검색 → 카드·버전 확인 → 작은 샘플 실행 → 자체 평가
→ Space 데모 또는 서비스 통합 → 결과와 한계 기록
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| 외부 API와 자체 실행 비교 | 01 API와 로컬 LLM |
| VRAM, 양자화, 오픈 모델 | 02 자원 계획 |
| Transformers와 관련 도구 | 03 핵심 라이브러리 |
| 저장소 유형, revision, 보안 | 04 Hub 저장소 |
| 모델·데이터 후보 탐색 | 05 Model/Dataset Hub |
| ML 데모와 자원 연결 | 06 Spaces |
| 카드, YAML, 지표와 벤치마크 | 07 모델 평가 |

## 최종 점검

- [ ] 사용 사례와 민감 데이터 경계를 먼저 정의한다.
- [ ] 모델의 자원 요구량과 총비용을 측정한다.
- [ ] 라이선스·카드·저장소 이력을 확인한다.
- [ ] revision을 고정하고 자체 데이터로 검증한다.
- [ ] 모델의 한계와 평가 조건을 함께 기록한다.
