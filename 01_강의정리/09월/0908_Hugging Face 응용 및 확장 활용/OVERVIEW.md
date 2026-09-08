# Hugging Face 응용 및 확장 활용

모델 학습을 안정적으로 운영하고, 제한된 자원에 맞게 최적화한 뒤, 재사용 가능하고 안전한 Hub 자산으로 배포하는 흐름을 학습합니다.

## 학습 목표

- loss·gradient·학습률 로그로 학습 문제를 진단합니다.
- 체크포인트 복구와 메모리 최적화의 교환관계를 이해합니다.
- LoRA, 양자화, 증류의 목적과 평가 기준을 구분합니다.
- 모델·tokenizer·config·모델 카드를 재로드 가능한 패키지로 구성합니다.
- 최소 권한 토큰과 gated 모델 정책을 안전하게 운영합니다.

## 추천 학습 순서

1. [학습 안정성 진단과 디버깅](01-training-stability-debugging.md)
2. [체크포인트 복구와 메모리 최적화](02-checkpoints-and-memory.md)
3. [PEFT와 LoRA 어댑터](03-peft-and-lora.md)
4. [양자화 전략과 실행 환경](04-quantization-strategies.md)
5. [지식 증류와 경량화 평가](05-distillation-and-evaluation.md)
6. [Hub 배포 패키지와 모델 카드](06-hub-packaging-and-model-cards.md)
7. [인증과 Gated 모델 운영](07-auth-and-gated-models.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
학습 로그 관측 → 안정화·복구 설계 → PEFT 적용
→ 양자화·증류 비교 → 품질·비용 벤치마크
→ 로컬 재로드 검증 → Hub 배포 → 권한·라이선스 운영
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| loss 급등과 gradient clipping | 01 학습 안정성 |
| optimizer까지 복구하는 checkpoint | 02 체크포인트·메모리 |
| 저랭크 adapter와 학습 파라미터 | 03 PEFT·LoRA |
| INT8·INT4와 실행 환경 | 04 양자화 |
| teacher·student와 soft target | 05 지식 증류 |
| `save_pretrained()`와 모델 카드 | 06 Hub 패키지 |
| 토큰·gated 승인·정책 검사 | 07 인증·Gated 모델 |

## 최종 점검

- [ ] 안정성과 성능 문제를 분리해 진단한다.
- [ ] 체크포인트 복구를 실제로 테스트한다.
- [ ] 경량화 전후 품질·메모리·latency를 함께 비교한다.
- [ ] 모델 패키지를 독립적으로 재로드한다.
- [ ] 토큰·승인·라이선스를 배포 전에 검토한다.
