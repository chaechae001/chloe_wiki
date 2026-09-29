# 용어집

클라우드 컴퓨팅 기초에서 등장한 핵심 용어를 쉬운 말로 정리했습니다.

## 클라우드 모델

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Cloud Computing | 원격 데이터 센터의 컴퓨팅 자원을 요청에 따라 사용하는 방식 | [기본 개념](01-cloud-computing-fundamentals.md) | 온디맨드, 탄력성 |
| SaaS | 완성된 소프트웨어를 서비스로 사용하는 모델 | [서비스 모델](02-service-and-deployment-models.md) | 공동 책임 |
| PaaS | 애플리케이션 실행 환경까지 제공받는 모델 | [서비스 모델](02-service-and-deployment-models.md) | 런타임, 배포 |
| IaaS | 가상 서버·네트워크·스토리지를 제공받는 모델 | [서비스 모델](02-service-and-deployment-models.md) | VM, 이미지 |
| Hybrid Cloud | Public과 Private 환경을 연결해 사용하는 방식 | [배포 모델](02-service-and-deployment-models.md) | 데이터 위치 |
| Multi Cloud | 둘 이상의 클라우드 제공자를 함께 쓰는 전략 | [배포 모델](02-service-and-deployment-models.md) | 이식성, 복잡도 |

## 가상화와 관리

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Hypervisor | 여러 VM의 물리 자원 사용을 중재하는 계층 | [가상화](03-virtualization-and-hypervisors.md) | Type 1, Type 2 |
| Virtual Machine | 가상 하드웨어와 Guest OS를 가진 독립 실행 환경 | [가상화](03-virtualization-and-hypervisors.md) | 자원 격리 |
| Cloud Manager | 자원 요청과 할당·생성·회수를 조정하는 관리 계층 | [관리 아키텍처](04-cloud-management-architecture.md) | Control Plane |
| Image | 같은 환경을 반복 생성하기 위한 OS·기본 설정 템플릿 | [관리 아키텍처](04-cloud-management-architecture.md) | Snapshot |
| IAM | 사용자 신원과 역할·권한을 관리하는 체계 | [관리 아키텍처](04-cloud-management-architecture.md) | 최소 권한 |

## 네트워크·스토리지·운영

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Subnet | 네트워크 주소 공간과 통신 경계를 나눈 단위 | [네트워크와 스토리지](05-cloud-network-and-storage.md) | Routing |
| Object Storage | 객체와 메타데이터를 API로 저장·조회하는 방식 | [네트워크와 스토리지](05-cloud-network-and-storage.md) | 모델 파일, 백업 |
| Block Storage | VM이 디스크처럼 사용하는 블록 단위 저장 방식 | [네트워크와 스토리지](05-cloud-network-and-storage.md) | 파일시스템 |
| File Storage | 디렉터리와 파일 경로로 공유하는 저장 방식 | [네트워크와 스토리지](05-cloud-network-and-storage.md) | NFS, 공유 폴더 |
| Provisioning | 자원을 생성해 실제 사용할 수 있는 상태로 준비하는 과정 | [운영](06-provisioning-security-and-cost.md) | 구성 자동화 |
| Cost Governance | 소유자·예산·정책으로 비용을 지속 관리하는 체계 | [운영](06-provisioning-security-and-cost.md) | 태그, 예산 경보 |

## 연결

- [전체 개요](OVERVIEW.md)
- [클라우드 컴퓨팅의 기본 개념](01-cloud-computing-fundamentals.md)
- [프로비저닝·보안·비용 운영](06-provisioning-security-and-cost.md)
