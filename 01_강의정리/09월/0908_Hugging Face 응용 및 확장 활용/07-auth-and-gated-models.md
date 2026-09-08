# 인증과 Gated 모델 운영

모델을 불러오는 기술적 성공과 사용할 권리가 있다는 판단은 다릅니다. 토큰 권한, gated 승인, 라이선스, 공개 범위를 배포 전 정책으로 점검해야 합니다.

**핵심 키워드:** access token, least privilege, gated model, license, policy as code

## 토큰 권한과 보관

| 권한 | 사용 예 | 운영 원칙 |
|---|---|---|
| read | private·gated 자산 다운로드 | 읽기 작업에 write 금지 |
| write | 저장소 생성·업로드 | 배포 작업에만 제한 |
| fine-grained | 특정 저장소·작업 접근 | 운영 환경의 기본 선택 |

토큰을 코드, 노트북 출력, Git 기록에 넣지 않습니다. 환경 변수나 secret manager에서 주입하고 로그에서는 마스킹합니다. 가능하면 앱·환경별로 토큰을 분리해 하나가 유출돼도 영향 범위를 줄입니다.

```python
import os

token = os.environ.get("HF_TOKEN")
if not token:
    raise RuntimeError("HF_TOKEN이 설정되지 않았습니다.")

model = AutoModel.from_pretrained(model_id, token=token)
```

공개 예제에는 실제 토큰이나 토큰 모양의 문자열도 넣지 않는 편이 안전합니다.

## Gated 모델의 의미

Gated 모델은 접근 요청이 활성화된 모델입니다. 사용자는 모델 제공자에게 계정 정보를 공유하고 조건에 동의해야 하며, 승인은 사용자 단위로 관리됩니다. 저장소가 public인지와 gated인지도 별개의 축입니다.

```text
모델 후보 등록 → 라이선스·사용 목적 확인 → 접근 요청·승인 확인
→ 최소 권한 토큰 선택 → 기술 smoke test → 사용 기록·재검토
```

## 정책을 코드로 검사하기

```python
def review_candidate(candidate, policy):
    reasons = []
    if candidate["gated"] and not candidate["approved"]:
        reasons.append("approval-required")
    if candidate["license"] not in policy["allowed_licenses"]:
        reasons.append("license-review")
    if candidate["required_scope"] not in policy["allowed_scopes"]:
        reasons.append("scope-review")
    return {"status": "ok" if not reasons else "review", "reasons": reasons}
```

자동화는 법적 판단을 대신하지 않습니다. 허용 목록, 승인 근거, 검토자, 정책 버전을 기록하고 애매한 라이선스는 담당 검토로 보냅니다.

## 운영 체크리스트

- 요청 목적과 모델 사용 조건이 일치하는가
- 계정 승인이 유효한가
- 최소 scope 토큰을 사용하는가
- private·public·gated 범위가 의도와 맞는가
- 모델 카드와 데이터 출처가 충분한가
- 토큰 회전·폐기와 접근 로그 정책이 있는가

CI/CD에서는 장기 토큰 대신 지원되는 경우 짧은 수명의 OIDC 기반 자격 증명도 검토합니다.

## 직접 해보기

1. 추론 전용 서비스에 write 토큰을 쓰면 안 되는 이유는 무엇인가요?
2. gated 승인과 라이선스 허용을 왜 따로 검사해야 하나요?
3. 모델 후보 자동 검토 리포트의 필드를 설계하세요.

<details>
<summary>정답 보기</summary>

1. 유출 시 저장소 수정·업로드까지 가능한 불필요한 권한이 노출되기 때문입니다.
2. 파일 접근 승인은 가능해도 프로젝트 목적에 대한 재사용·상업 조건이 허용되지 않을 수 있기 때문입니다.
3. 모델 식별자, 정책 버전, 승인 상태, 라이선스, 요구 scope, 공개 범위, 판정, 사유, 검토 시각을 포함합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| authentication vs authorization | 신원 확인 vs 허용된 행동 범위 |
| private vs gated | 지정 사용자만 접근 vs 접근 요청·동의가 필요한 모델 |
| 자동 검사 vs 최종 승인 | 명확한 규칙 판정 vs 맥락을 포함한 책임 있는 결정 |

## 연결되는 개념

- 이전: [Hub 배포 패키지와 모델 카드](06-hub-packaging-and-model-cards.md)
- 함께 볼 키워드: `secret manager`, `OIDC`, `license review`

## 셀프 체크

- [ ] read·write·fine-grained 권한을 구분한다.
- [ ] 토큰을 코드와 로그에서 제거한다.
- [ ] gated 승인과 라이선스를 따로 검사한다.
- [ ] 정책 판정 사유를 기록한다.
- [ ] 자격 증명 회전 전략을 세운다.

### 복습 질문 및 답변

**Q1. 승인된 계정의 토큰을 팀이 공유해도 되나요?**

<details>
<summary>답</summary>

개인 토큰 공유는 피하고 서비스별 최소 권한 자격 증명과 조직 정책을 사용해야 합니다.

</details>

**Q2. 403 오류는 항상 토큰이 틀렸다는 뜻인가요?**

<details>
<summary>답</summary>

아닙니다. 인증은 됐지만 모델 승인이 없거나 해당 저장소 권한이 부족한 경우도 있습니다.

</details>

**Q3. 정책 자동화의 가장 중요한 출력은 무엇인가요?**

<details>
<summary>답</summary>

단순 통과 여부뿐 아니라 어떤 조건 때문에 검토가 필요한지 보여 주는 구조화된 사유입니다.

</details>

## 한 줄 정리

> 안전한 Hub 운영은 최소 권한 인증과 모델별 승인·라이선스 판단을 기록 가능한 정책으로 연결합니다.
