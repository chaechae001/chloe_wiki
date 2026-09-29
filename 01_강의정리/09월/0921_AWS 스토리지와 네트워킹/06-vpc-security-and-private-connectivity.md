# VPC 보안과 프라이빗 연결

> 안전한 네트워크는 하나의 방화벽에 의존하지 않고, 인스턴스·서브넷·서비스 경계마다 최소 경로를 겹겹이 적용합니다.

`Security Group` · `NACL` · `VPC Endpoint` · `VPC Peering` · `private access`

## 핵심요약

- Security Group은 자원 수준의 상태 저장형 허용 규칙입니다.
- NACL은 서브넷 경계의 상태 비저장형 허용·거부 규칙입니다.
- VPC Endpoint는 인터넷을 거치지 않고 지원 AWS 서비스에 접근합니다.
- VPC Peering은 두 VPC 사이의 사설 라우팅 연결입니다.
- 관리 접속은 직접 공개 주소보다 관리형 세션과 감사 가능한 경로를 우선합니다.

## 1. Security Group과 NACL

<details>
<summary>답</summary>

두 기능은 대체 관계가 아닙니다. Security Group은 워크로드별 최소 허용을 표현하고, NACL은 서브넷 경계의 추가 방어와 명시적 차단이 필요할 때 사용합니다.

</details>

| 관점 | Security Group | NACL |
|---|---|---|
| 적용 범위 | 네트워크 인터페이스·자원 | 서브넷 |
| 상태 | 상태 저장 | 상태 비저장 |
| 규칙 | 허용 중심 | 허용·거부 |
| 평가 | 모든 규칙의 합 | 번호 순서 |

상태 비저장 규칙은 요청뿐 아니라 반환 트래픽의 경로도 명시적으로 고려해야 합니다.

## 2. VPC Endpoint

Endpoint를 사용하면 프라이빗 서브넷의 워크로드가 지원 서비스에 접근할 때 인터넷 게이트웨이나 NAT를 통과하지 않아도 됩니다. 엔드포인트 정책과 서비스 권한을 함께 제한해야 합니다.

## 3. VPC Peering

Peering은 두 VPC의 사설 주소 간 통신을 가능하게 하지만 자동으로 모든 경로가 생기는 것은 아닙니다. 양쪽 Route Table과 Security Group을 구성하고 주소 중복이 없는지 확인합니다. 연결을 다른 VPC로 연쇄 전달하는 허브 방식은 별도 네트워크 서비스를 검토합니다.

## 4. 관리 접속

공개 관리 포트를 전체 인터넷에 열거나 장기 키를 배포하는 방식은 피합니다. 신원 기반 관리형 세션, 짧은 수명의 권한, 접속 로그와 승인 절차를 우선하고 필요한 경우 제한된 점프 경로를 사용합니다.

## 5. 계층형 방어

```text
Identity policy
  + Route boundary
  + NACL
  + Security Group
  + Host firewall
  + Application authorization
```

각 계층의 책임을 문서화하고 변경 로그·흐름 로그·경보를 연결하면 문제 원인을 더 빠르게 좁힐 수 있습니다.

## 직접 해보기

1. 프라이빗 애플리케이션이 S3에 접근할 때 인터넷 경로를 피하는 방법을 적으세요.
2. NACL에서 요청은 허용했지만 응답이 실패할 수 있는 이유를 설명하세요.
3. 두 VPC를 Peering했지만 통신되지 않을 때 확인 순서를 작성하세요.

<details>
<summary>정답 보기</summary>

1. 지원되는 VPC Endpoint를 만들고 Route Table·Endpoint 정책·IAM 권한을 제한합니다.
2. NACL은 상태 비저장이므로 반환 방향의 주소와 임시 포트 범위도 허용되어야 합니다.
3. 주소 중복, Peering 상태, 양쪽 Route Table, Security Group, NACL, 운영체제 방화벽과 이름 해석을 확인합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Security Group vs NACL | 자원 수준 상태 저장 허용과 서브넷 수준 상태 비저장 허용·거부입니다. |
| Endpoint vs NAT | AWS 서비스로 가는 사설 경로와 일반 외부 방향 주소 변환입니다. |
| Peering vs 인터넷 연결 | 두 VPC 사설 라우팅과 공개 네트워크 경로입니다. |

## 연결되는 개념

- 이전: [VPC와 서브넷](05-vpc-subnets-and-routing.md)
- 다음: [AWS 웹 아키텍처](07-resilient-web-architecture.md)

## 셀프 체크

- [ ] Security Group과 NACL을 구분한다.
- [ ] 상태 저장과 비저장을 설명한다.
- [ ] Endpoint의 보안 이점을 안다.
- [ ] Peering의 라우팅 제약을 이해한다.
- [ ] 안전한 관리 접속 원칙을 설명한다.

### 복습 질문 및 답변

**Q1. Security Group이 있으면 NACL은 반드시 복잡하게 설정해야 하나요?**

<details>
<summary>답</summary>

아닙니다. 기본 규칙을 단순하게 유지하고 조직의 방어 요구와 명시적 차단 필요가 있을 때 추가합니다. 복잡한 규칙은 운영 오류를 늘릴 수 있습니다.

</details>

**Q2. Endpoint를 만들면 IAM 권한 검사가 사라지나요?**

<details>
<summary>답</summary>

아닙니다. 네트워크 경로와 API 권한은 별도 계층입니다. 엔드포인트 정책, 자원 정책과 IAM 권한을 함께 통과해야 합니다.

</details>

**Q3. Peering된 VPC의 다른 연결까지 자동으로 이용할 수 있나요?**

<details>
<summary>답</summary>

일반적으로 Peering은 두 VPC 간 직접 연결이며 제3의 네트워크로 전이되는 허브 역할을 하지 않습니다. 요구 규모에 맞는 연결 서비스를 선택합니다.

</details>

## 한 줄 정리

> VPC 보안은 자원·서브넷·서비스 경계를 나누고 인터넷 대신 필요한 사설 경로만 허용하는 방식으로 강화합니다.
