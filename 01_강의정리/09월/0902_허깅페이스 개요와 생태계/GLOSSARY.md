# 용어집

| 용어 | 설명 |
|---|---|
| LLM | 대규모 텍스트 데이터로 학습한 언어 모델 |
| API Inference | 외부 서비스의 엔드포인트를 호출해 모델 결과를 받는 방식 |
| Local Inference | 사용자가 관리하는 장비나 서버에서 모델을 실행하는 방식 |
| Parameter | 모델이 학습해 저장한 수치 값 |
| VRAM | GPU가 모델과 연산 데이터를 저장하는 메모리 |
| Quantization | 가중치·연산의 수치 정밀도를 낮춰 자원 사용을 줄이는 기법 |
| Tokenizer | 텍스트를 모델 입력 토큰 ID로 변환하는 구성 요소 |
| Transformers | 다양한 사전학습 모델을 다루는 Hugging Face 라이브러리 |
| Datasets | 데이터셋 로드·변환·스트리밍 라이브러리 |
| Accelerate | 여러 장치와 분산 환경의 실행을 단순화하는 라이브러리 |
| Diffusers | 확산 기반 생성 모델용 라이브러리 |
| PEFT | 일부 추가 파라미터를 중심으로 효율적으로 미세 조정하는 방법·라이브러리 |
| Hub | 모델·데이터셋·Space 저장소를 검색하고 공유하는 플랫폼 |
| Revision | 저장소의 특정 커밋, 태그 또는 브랜치를 가리키는 버전 식별자 |
| Model Card | 모델의 목적, 데이터, 평가, 제한과 라이선스를 설명하는 문서 |
| Dataset Card | 데이터의 출처, 구조, 용도, 편향과 라이선스를 설명하는 문서 |
| Space | ML 데모 애플리케이션을 공유·실행하는 Hub 저장소 |
| Benchmark | 정해진 데이터와 절차로 모델을 비교하는 평가 기준 |
| Accuracy | 전체 예측 중 맞힌 비율 |
| Precision | 양성으로 예측한 것 중 실제 양성의 비율 |
| Recall | 실제 양성 중 모델이 찾아낸 비율 |
| F1-score | Precision과 Recall의 조화 평균 |
| YAML front matter | Markdown 맨 위에 기록하는 구조화 메타데이터 영역 |
| Gated Model | 이용 조건 동의나 승인을 거쳐 접근하는 모델 |

## 함께 보기

- [Hugging Face 핵심 라이브러리](03-hugging-face-libraries.md)
- [Hub 저장소와 버전 관리](04-hub-repositories.md)
- [모델 카드와 평가 지표 해석](07-model-cards-and-evaluation.md)
