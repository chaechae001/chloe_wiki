# GLOSSARY

> 260917 클라우드 구조와 활용 학습자료에서 자주 사용하는 핵심 용어를 빠르게 확인합니다.

## 클라우드 인프라

| 용어 | 의미 |
|---|---|
| Private Cloud | 조직이 직접 소유하거나 통제하는 환경에서 클라우드 운영 모델로 자원을 제공하는 방식 |
| OpenStack | 컴퓨팅·네트워크·스토리지 자원을 API로 제어하는 오픈소스 클라우드 플랫폼 |
| Control Plane | 목표 상태를 결정하고 인증·스케줄링·상태 관리를 조정하는 영역 |
| Data Plane | 실제 사용자 트래픽과 워크로드가 처리되는 영역 |
| Hypervisor | 하나의 물리 서버에서 여러 가상 머신을 실행하도록 자원을 가상화하는 계층 |
| Nova | OpenStack의 컴퓨팅 자원 관리 서비스 |
| Neutron | OpenStack의 네트워크 관리 서비스 |
| Cinder | 블록 스토리지 볼륨 서비스 |
| Swift | 분산 객체 스토리지 서비스 |
| Glance | 가상 머신 이미지 카탈로그와 전달 서비스 |
| Keystone | 인증과 권한, 서비스 카탈로그를 담당하는 서비스 |
| Horizon | OpenStack 자원을 조작하는 웹 대시보드 |

## 컨테이너와 애플리케이션

| 용어 | 의미 |
|---|---|
| Namespace | 프로세스·네트워크·파일 시스템 등의 보이는 범위를 격리하는 커널 기능 |
| cgroups | 프로세스 그룹의 CPU·메모리 등 자원 사용량을 제한하고 측정하는 기능 |
| Container Image | 애플리케이션과 실행에 필요한 파일을 재현 가능하게 묶은 읽기 전용 템플릿 |
| Pod | Kubernetes에서 함께 배치되고 네트워크를 공유하는 최소 실행 단위 |
| Deployment | Pod의 복제본과 롤링 업데이트를 관리하는 Kubernetes 객체 |
| Service | 변하는 Pod 집합에 안정적인 네트워크 접근점을 제공하는 객체 |
| Desired State | 시스템이 유지해야 하는 목표 상태 |
| Reconciliation | 실제 상태를 목표 상태와 비교하고 차이를 줄이는 반복 과정 |
| Microservices | 업무 경계를 기준으로 독립 배포 가능한 작은 서비스들의 구조 |
| BaaS | 인증·데이터베이스 같은 완성된 백엔드 기능을 서비스로 사용하는 방식 |
| FaaS | 이벤트 발생 시 함수를 실행하고 실행량에 따라 사용하는 방식 |
| Idempotency | 같은 요청을 여러 번 처리해도 최종 결과가 달라지지 않는 성질 |

## 네트워크와 분산 클라우드

| 용어 | 의미 |
|---|---|
| NFV | 네트워크 기능을 전용 장비 대신 범용 인프라의 소프트웨어로 구현하는 방식 |
| SDN | 네트워크 제어와 전달을 분리해 경로와 정책을 프로그램 가능하게 만드는 방식 |
| Edge Cloud | 데이터 발생 지점 가까운 곳에 연산·저장 자원을 배치하는 클라우드 계층 |
| Edge AI | 가까운 장치나 거점에서 AI 추론을 수행하는 방식 |
| Hybrid Cloud | 사내 환경과 공개 클라우드를 연결해 함께 운영하는 구성 |
| Multi Cloud | 둘 이상의 클라우드 제공자를 함께 사용하는 구성 |
| Cold Start | 일정 시간 실행되지 않은 함수가 초기화되며 추가 지연이 발생하는 현상 |
| Vendor Lock-in | 특정 제공자의 API·서비스에 대한 의존 때문에 이전 비용이 커지는 상태 |

## 문서 바로가기

- [01 OpenStack과 프라이빗 클라우드 제어 구조](./01-openstack-control-plane.md)
- [02 OpenStack 핵심 서비스](./02-openstack-core-services.md)
- [03 컨테이너와 Docker](./03-containers-and-docker.md)
- [04 Kubernetes 오케스트레이션](./04-kubernetes-orchestration.md)
- [05 마이크로서비스와 Serverless](./05-microservices-and-serverless.md)
- [06 NFV·Edge와 클라우드 활용](./06-nfv-edge-and-cloud-applications.md)
- [전체 학습 지도](./OVERVIEW.md)
