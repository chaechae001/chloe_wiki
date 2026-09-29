# 용어집

이번 학습에서 등장한 AWS 입문과 EC2 운영 용어를 쉬운 말로 정리했습니다.

## 클라우드와 위치

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Cloud Computing | 인프라 자원을 네트워크와 API로 필요할 때 사용하는 운영 방식 | [클라우드 기초](01-aws-cloud-foundations.md) | 온디맨드, 탄력성 |
| IaaS | 운영체제부터 사용자가 관리하는 인프라 서비스 | [클라우드 기초](01-aws-cloud-foundations.md) | PaaS, SaaS |
| Region | AWS 서비스를 제공하는 독립적인 지리 영역 | [글로벌 인프라](02-aws-global-infrastructure.md) | AZ |
| Availability Zone | Region 안에서 장애 영향을 분리한 위치 단위 | [글로벌 인프라](02-aws-global-infrastructure.md) | 고가용성 |
| Edge Location | 사용자 가까이에서 캐시와 네트워크 기능을 제공하는 거점 | [글로벌 인프라](02-aws-global-infrastructure.md) | 지연 시간 |

## 신원과 컴퓨팅

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| IAM | AWS 신원과 API 권한을 관리하는 서비스 | [IAM](03-iam-and-access-methods.md) | 인증, 권한 부여 |
| Policy | 어떤 행동을 어떤 자원에 허용할지 표현한 규칙 | [IAM](03-iam-and-access-methods.md) | 최소 권한 |
| Role | 사용자나 서비스가 필요할 때 맡는 임시 권한 집합 | [IAM](03-iam-and-access-methods.md) | 임시 자격 증명 |
| MFA | 비밀번호 외 추가 요소로 인증을 강화하는 방식 | [IAM](03-iam-and-access-methods.md) | 강한 인증 |
| EC2 | 운영체제와 실행 환경을 직접 관리하는 가상 머신 서비스 | [EC2](04-ec2-and-security-groups.md) | IaaS |
| AMI | 같은 구성의 인스턴스를 만들기 위한 이미지 템플릿 | [EC2](04-ec2-and-security-groups.md) | 시작 템플릿 |
| Security Group | 인스턴스 수준에서 허용 트래픽을 정의하는 가상 방화벽 | [EC2](04-ec2-and-security-groups.md) | 인바운드, 아웃바운드 |

## 저장소와 고가용성

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| EBS | 인스턴스에 디스크처럼 연결하는 영구 블록 볼륨 | [스토리지](05-ebs-and-efs-storage.md) | 스냅샷, IOPS |
| EFS | 여러 인스턴스가 파일 경로로 공유하는 관리형 파일 시스템 | [스토리지](05-ebs-and-efs-storage.md) | 네트워크 파일 시스템 |
| Snapshot | 볼륨의 특정 시점 상태를 보존하는 백업 단위 | [스토리지](05-ebs-and-efs-storage.md) | 복구 |
| IOPS | 초당 처리 가능한 입출력 작업 수 | [스토리지](05-ebs-and-efs-storage.md) | 처리량, 지연 |
| ELB | 여러 컴퓨팅 대상에 트래픽을 분산하는 관리형 서비스 | [고가용성](06-load-balancing-and-auto-scaling.md) | ALB, NLB |
| Target Group | 로드 밸런서가 요청을 전달하고 상태를 확인할 대상 묶음 | [고가용성](06-load-balancing-and-auto-scaling.md) | Listener |
| Auto Scaling Group | 정책에 따라 인스턴스 수를 유지·확장·축소하는 그룹 | [고가용성](06-load-balancing-and-auto-scaling.md) | 탄력성 |

## 문서 바로가기

- [전체 학습 지도](OVERVIEW.md)
