# AWS 글로벌 인프라

> Region, Availability Zone, Edge Location은 위치 이름이 아니라 지연 시간과 장애 범위를 결정하는 설계 단위입니다.

`Region` · `Availability Zone` · `Edge Location` · `latency` · `high availability`

## 핵심요약

- Region은 서로 분리된 지리적 서비스 제공 단위입니다.
- Availability Zone은 Region 안에서 장애 영역을 분리합니다.
- Edge Location은 사용자 가까이에서 콘텐츠와 네트워크 기능을 제공합니다.
- Region 선택은 지연, 규제, 서비스 지원과 비용을 함께 봅니다.
- 고가용성은 여러 AZ에 배치하고 상태를 지속적으로 확인해 구현합니다.

## 1. Region

Region은 여러 데이터 센터와 Availability Zone을 포함하는 독립적인 지리 영역입니다. 데이터가 머무를 위치, 사용자와의 거리, 제공 서비스, 요금이 Region마다 다를 수 있습니다.

<details>
<summary>답</summary>

가장 가까운 Region만 고르는 것이 항상 정답은 아닙니다. 지연 시간 외에 데이터 주권, 필요한 서비스의 지원 여부, 장애 복구 목표와 비용을 함께 비교해야 합니다.

</details>

## 2. Availability Zone

AZ는 하나의 Region 안에서 전력·네트워크 등 장애 영향을 분리하도록 설계된 위치입니다. 한 AZ에만 애플리케이션을 두면 해당 장애 영역 문제가 서비스 전체 중단으로 이어질 수 있습니다.

```text
Region
├─ AZ-A: Load-balanced application instances
├─ AZ-B: Load-balanced application instances
└─ Regional services: DNS, managed control planes, shared endpoints
```

## 3. Edge Location

Edge Location은 사용자 가까운 지점에서 캐시, DNS 응답, 보안과 네트워크 가속을 수행합니다. 원본 애플리케이션 전체를 옮기는 것이 아니라 반복 요청의 응답 경로를 줄이는 데 주로 활용합니다.

## 4. 위치 선택 기준

| 기준 | 질문 | 설계 영향 |
|---|---|---|
| 지연 시간 | 사용자와 얼마나 가까운가? | 응답 속도 |
| 규제 | 데이터를 어디에 저장해야 하는가? | Region 제한 |
| 서비스 | 필요한 기능이 제공되는가? | 대체 서비스 필요 |
| 비용 | 컴퓨팅·전송 요금은 어떤가? | 총비용 |
| 복구 | 다른 위치로 복구할 목표가 있는가? | 백업·복제 구조 |

## 5. 고가용성과 재해 복구

다중 AZ는 같은 Region 안의 장애를 견디는 데 초점이 있고, 다중 Region은 더 큰 지역 장애와 재해 복구를 고려합니다. 범위가 넓어질수록 데이터 복제, 일관성, 비용과 운영 복잡도가 커집니다.

## 직접 해보기

1. 한 AZ에만 배치된 웹 서버의 위험을 설명하세요.
2. 전 세계 정적 콘텐츠 응답을 빠르게 하는 데 적합한 위치 계층을 고르세요.
3. Region 선택 체크리스트를 지연 시간 외 세 가지로 작성하세요.

<details>
<summary>정답 보기</summary>

1. 해당 AZ 장애가 곧 전체 서비스 중단이 될 수 있습니다.
2. 사용자 가까운 Edge Location을 통한 캐시와 전송이 적합합니다.
3. 데이터 규제, 필요한 서비스 지원 여부, 비용과 복구 목표 등을 확인합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Region vs AZ | 독립 지리 영역과 Region 내부의 분리된 장애 영역입니다. |
| AZ vs 데이터 센터 | AZ는 하나 이상의 데이터 센터로 구성될 수 있는 논리적 장애 단위입니다. |
| Edge vs Region | 사용자 가까운 전송 거점과 애플리케이션 자원을 배치하는 지리 영역입니다. |

## 연결되는 개념

- 이전: [AWS 클라우드 기초](01-aws-cloud-foundations.md)
- 다음: [IAM과 AWS 접근 방식](03-iam-and-access-methods.md)

## 셀프 체크

- [ ] Region 선택 기준을 설명한다.
- [ ] AZ가 분리되는 이유를 안다.
- [ ] Edge Location의 역할을 구분한다.
- [ ] 다중 AZ와 다중 Region의 목적을 구분한다.
- [ ] 가용성과 비용의 관계를 설명한다.

### 복습 질문 및 답변

**Q1. 한 Region의 여러 AZ는 같은 장애 영역인가요?**

<details>
<summary>답</summary>

아닙니다. 서로 분리된 장애 영역으로 설계되지만 Region 수준의 공통 영향 가능성까지 제거하는 것은 아닙니다.

</details>

**Q2. Edge Location에 데이터베이스를 그대로 옮기는 개념인가요?**

<details>
<summary>답</summary>

일반적으로는 아닙니다. 캐시와 네트워크 서비스를 사용자 가까이 제공해 원본까지의 요청을 줄이는 역할이 중심입니다.

</details>

**Q3. 고가용성과 재해 복구는 같은 말인가요?**

<details>
<summary>답</summary>

고가용성은 일상적인 장애에도 서비스를 계속 제공하는 능력이고, 재해 복구는 큰 장애 뒤 정해진 시간과 데이터 손실 목표 안에서 복구하는 계획입니다.

</details>

## 한 줄 정리

> AWS의 위치 계층은 서비스 제공 범위와 장애 경계를 나누므로 워크로드 요구에 맞춰 선택해야 합니다.
