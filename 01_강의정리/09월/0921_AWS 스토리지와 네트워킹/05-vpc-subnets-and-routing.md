# VPC, 서브넷과 라우팅

> VPC 설계는 주소를 나누는 작업이 아니라, 워크로드의 노출 범위와 통신 경로를 명확히 만드는 작업입니다.

`VPC` · `subnet` · `route table` · `Internet Gateway` · `NAT Gateway`

## 핵심요약

- VPC는 Region 범위의 논리적으로 격리된 네트워크입니다.
- 서브넷은 하나의 AZ에 속하며 용도와 노출 수준에 따라 나눕니다.
- 퍼블릭 여부는 이름이 아니라 인터넷 경로와 주소 조건으로 결정됩니다.
- Route Table은 목적지에 따른 다음 경로를 정의합니다.
- NAT Gateway는 프라이빗 자원의 외부 방향 통신을 중계하지만 외부의 임의 시작 연결을 허용하지 않습니다.

## 1. 주소 계획과 서브넷

VPC의 사설 주소 범위를 먼저 정하고 AZ별·계층별 서브넷으로 나눕니다. 미래 확장, 다른 네트워크와의 연결, 주소 중복을 고려해야 나중에 재구성 비용을 줄일 수 있습니다.

<details>
<summary>답</summary>

서브넷은 단순 폴더가 아닙니다. 하나의 AZ와 주소 범위를 가지며 연결된 Route Table과 NACL에 따라 통신 경로와 경계가 달라집니다.

</details>

## 2. Public과 Private

| 구분 | 인터넷 경로 | 대표 배치 |
|---|---|---|
| Public Subnet | Internet Gateway로 가는 경로와 공개 주소 조건 | 인터넷 진입 로드 밸런서, NAT Gateway |
| Private Subnet | 직접 인터넷 진입 경로 없음 | 애플리케이션, 데이터베이스 |

프라이빗 서브넷의 인스턴스가 업데이트를 내려받아야 하면 NAT 경로를 사용할 수 있지만 외부 사용자가 해당 인스턴스로 직접 접속하는 구조는 피합니다.

## 3. Route Table과 Internet Gateway

Route Table은 목적지 주소 범위와 다음 대상의 목록입니다. 더 구체적인 경로가 우선되며, VPC 내부 기본 경로는 같은 VPC 주소 간 통신을 가능하게 합니다. Internet Gateway는 VPC와 인터넷 경계에 연결됩니다.

## 4. NAT Gateway

NAT Gateway는 퍼블릭 서브넷에 두고 프라이빗 서브넷의 외부 목적지 경로가 이를 가리키게 합니다. 고가용성과 불필요한 AZ 간 전송을 줄이려면 AZ별 경로를 설계합니다.

## 5. 다중 AZ 기본 구조

```text
Internet → Internet Gateway → Public load balancer
                              ↓
                  Private app subnets across AZs
                              ↓
                      Private database subnets

Private outbound → NAT Gateway → Internet Gateway
```

## 직접 해보기

1. 인터넷 진입 로드 밸런서와 애플리케이션 인스턴스를 각각 어디에 둘지 정하세요.
2. 프라이빗 인스턴스가 외부 업데이트 서버에 연결할 경로를 설명하세요.
3. 두 VPC를 연결할 계획이 있을 때 주소 계획에서 먼저 확인할 항목을 적으세요.

<details>
<summary>정답 보기</summary>

1. 로드 밸런서는 퍼블릭 서브넷, 애플리케이션 인스턴스는 프라이빗 서브넷에 둡니다.
2. 프라이빗 Route Table에서 NAT Gateway로 보내고 NAT가 Internet Gateway를 통해 외부와 통신합니다.
3. 두 VPC의 사설 주소 범위가 겹치지 않는지 확인하고 향후 확장 공간을 남깁니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| VPC vs Subnet | Region 범위 네트워크와 AZ 범위 주소 구간입니다. |
| Internet Gateway vs NAT Gateway | 인터넷 경계 연결과 프라이빗 출발 통신의 주소 변환입니다. |
| Public IP vs Public Subnet | 주소 속성과 인터넷 경로를 가진 서브넷 구조입니다. |

## 연결되는 개념

- 이전: [Route 53과 DNS](04-route53-and-dns.md)
- 다음: [VPC 보안과 프라이빗 연결](06-vpc-security-and-private-connectivity.md)

## 셀프 체크

- [ ] VPC와 서브넷 범위를 구분한다.
- [ ] Public·Private 판정 기준을 설명한다.
- [ ] Route Table을 읽을 수 있다.
- [ ] IGW와 NAT 역할을 구분한다.
- [ ] 다중 AZ 네트워크를 그릴 수 있다.

### 복습 질문 및 답변

**Q1. 프라이빗 서브넷은 외부 통신을 전혀 못 하나요?**

<details>
<summary>답</summary>

직접 인터넷 진입 경로는 없지만 NAT나 프라이빗 엔드포인트처럼 통제된 경로로 필요한 외부·서비스 통신을 할 수 있습니다.

</details>

**Q2. 인스턴스에 공개 주소가 있으면 자동으로 인터넷에 연결되나요?**

<details>
<summary>답</summary>

주소 외에도 Internet Gateway 경로, 보안 그룹, NACL과 운영체제 방화벽 조건이 맞아야 합니다.

</details>

**Q3. 하나의 NAT Gateway를 모든 AZ가 공유해도 되나요?**

<details>
<summary>답</summary>

가능하지만 해당 AZ 장애와 AZ 간 전송 비용·지연의 영향을 받습니다. 가용성 요구에 따라 AZ별 배치와 경로를 검토합니다.

</details>

## 한 줄 정리

> VPC는 서브넷과 라우팅을 통해 인터넷 노출, 내부 통신과 외부 방향 경로를 계층별로 분리합니다.
