# OVERVIEW

> 260917 학습자료는 프라이빗 클라우드의 제어 구조에서 시작해 컨테이너 오케스트레이션과 분산 클라우드 활용으로 확장됩니다.

## 학습 목표

- OpenStack의 Control Plane과 핵심 서비스의 협력 관계를 설명합니다.
- 가상 머신과 컨테이너의 격리 방식을 비교합니다.
- Kubernetes의 선언적 상태와 조정 흐름을 이해합니다.
- 마이크로서비스와 Serverless의 이점과 운영 비용을 함께 판단합니다.
- NFV, Edge, Hybrid·Multi Cloud의 적용 기준을 구분합니다.

## 학습 순서

```mermaid
flowchart LR
    A[01 제어 구조] --> B[02 핵심 서비스]
    B --> C[03 Docker]
    C --> D[04 Kubernetes]
    D --> E[05 MSA와 Serverless]
    E --> F[06 NFV와 Edge 활용]
```

| 순서 | 문서 | 학습 초점 | 확인 질문 |
|---|---|---|---|
| 1 | [OpenStack과 프라이빗 클라우드 제어 구조](./01-openstack-control-plane.md) | API와 Control Plane | 요청은 어떤 단계를 거치는가? |
| 2 | [OpenStack 핵심 서비스](./02-openstack-core-services.md) | 컴퓨팅·네트워크·스토리지 협력 | 각 서비스의 책임은 무엇인가? |
| 3 | [컨테이너와 Docker](./03-containers-and-docker.md) | 커널 격리와 이미지 | VM과 컨테이너는 무엇을 공유하는가? |
| 4 | [Kubernetes 오케스트레이션](./04-kubernetes-orchestration.md) | 선언적 상태와 조정 | 장애 후 복제본은 어떻게 복구되는가? |
| 5 | [마이크로서비스와 Serverless](./05-microservices-and-serverless.md) | 서비스 경계와 이벤트 실행 | 분리가 만드는 운영 비용은 무엇인가? |
| 6 | [NFV·Edge와 클라우드 활용](./06-nfv-edge-and-cloud-applications.md) | 처리 위치와 운영 전략 | 어느 작업을 Edge에 둘 것인가? |

## 개념 연결

1. OpenStack은 물리 자원을 API로 추상화해 클라우드 자원으로 제공합니다.
2. 컨테이너는 애플리케이션 실행 환경을 가볍게 묶고 빠르게 복제합니다.
3. Kubernetes는 여러 노드의 컨테이너 상태를 선언적으로 유지합니다.
4. 마이크로서비스와 Serverless는 배포·확장 단위를 더 작게 나눕니다.
5. NFV와 Edge는 네트워크 기능과 연산을 서비스에 알맞은 위치로 이동시킵니다.

## 학습 점검

- [ ] VM 생성 흐름에서 인증, 이미지, 네트워크, 컴퓨팅의 역할을 연결할 수 있다.
- [ ] Namespace와 cgroups가 각각 무엇을 격리·제한하는지 설명할 수 있다.
- [ ] Pod, Deployment, Service의 책임을 구분할 수 있다.
- [ ] FaaS가 적합한 작업과 부적합한 작업을 비용·지연 기준으로 판단할 수 있다.
- [ ] 중앙 클라우드와 Edge의 역할을 지연·데이터·연결 기준으로 나눌 수 있다.

## 참고

- [핵심 용어집](./GLOSSARY.md)
