# 용어집

AWS 스토리지, 데이터베이스와 네트워킹에서 자주 사용하는 핵심 용어를 정리했습니다.

## 데이터와 전송

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| RDS | 관계형 데이터베이스의 설치·패치·백업을 관리형으로 제공하는 서비스 | [RDS](01-rds-managed-databases.md) | Multi-AZ |
| Read Replica | 읽기 요청을 분산하는 복제본 | [RDS](01-rds-managed-databases.md) | 복제 지연 |
| S3 | 객체를 키와 API로 저장하는 오브젝트 스토리지 | [S3](02-s3-object-storage.md) | 버킷, 객체 |
| Storage Class | 접근 빈도와 복원 요구에 맞춘 S3 비용·성능 등급 | [S3](02-s3-object-storage.md) | 수명주기 |
| CloudFront | 엣지에서 콘텐츠를 캐시·전송하는 CDN | [CloudFront](03-cloudfront-cdn.md) | Origin, TTL |
| Origin | CDN이 원본 콘텐츠를 가져오는 대상 | [CloudFront](03-cloudfront-cdn.md) | 캐시 미스 |

## DNS와 VPC

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Route 53 | DNS 레코드와 라우팅 정책을 관리하는 서비스 | [DNS](04-route53-and-dns.md) | 호스팅 영역 |
| TTL | DNS 또는 캐시 결과를 재사용하는 시간 | [DNS](04-route53-and-dns.md) | 변경 전파 |
| VPC | Region 범위의 논리적으로 격리된 네트워크 | [VPC](05-vpc-subnets-and-routing.md) | CIDR, 서브넷 |
| Subnet | 하나의 AZ에 속하는 VPC 주소 구간 | [VPC](05-vpc-subnets-and-routing.md) | Route Table |
| Internet Gateway | VPC와 인터넷 경계를 연결하는 구성 요소 | [VPC](05-vpc-subnets-and-routing.md) | Public Subnet |
| NAT Gateway | 프라이빗 자원의 외부 방향 통신을 중계하는 관리형 NAT | [VPC](05-vpc-subnets-and-routing.md) | Private Subnet |

## 보안과 아키텍처

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Security Group | 자원 수준의 상태 저장형 허용 방화벽 | [VPC 보안](06-vpc-security-and-private-connectivity.md) | 최소 허용 |
| NACL | 서브넷 수준의 상태 비저장형 허용·거부 방화벽 | [VPC 보안](06-vpc-security-and-private-connectivity.md) | 규칙 순서 |
| VPC Endpoint | 인터넷 없이 지원 AWS 서비스로 연결하는 사설 경로 | [VPC 보안](06-vpc-security-and-private-connectivity.md) | Endpoint 정책 |
| VPC Peering | 두 VPC의 사설 주소 사이를 잇는 직접 라우팅 연결 | [VPC 보안](06-vpc-security-and-private-connectivity.md) | 주소 중복 |
| Multi-AZ | 여러 장애 영역에 자원을 분산하는 고가용성 방식 | [웹 아키텍처](07-resilient-web-architecture.md) | 장애 조치 |
| Observability | 로그·지표·추적으로 시스템 상태를 설명하는 능력 | [웹 아키텍처](07-resilient-web-architecture.md) | 서비스 목표 |

## 문서 바로가기

- [전체 학습 지도](OVERVIEW.md)
