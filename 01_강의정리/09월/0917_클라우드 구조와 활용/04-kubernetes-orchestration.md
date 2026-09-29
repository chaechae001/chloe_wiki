# Kubernetes 오케스트레이션

> Kubernetes는 컨테이너를 직접 하나씩 실행하는 도구가 아니라, 선언한 상태를 클러스터가 계속 유지하도록 조정하는 오케스트레이션 시스템입니다.

`Kubernetes` · `desired state` · `control plane` · `Pod` · `reconciliation`

## 핵심요약

- 사용자는 원하는 상태를 선언하고 Kubernetes가 실제 상태를 맞춥니다.
- Control Plane은 요청·스케줄링·상태 저장·조정을 담당합니다.
- Worker Node는 Pod를 실행하고 네트워크 연결을 유지합니다.
- Pod는 배포 단위, Deployment는 복제와 교체, Service는 안정적인 접근점을 제공합니다.
- 장애 복구와 확장은 반복적인 조정 루프에서 이루어집니다.

## 1. 오케스트레이션이 필요한 이유

컨테이너 수가 늘면 실행만으로는 부족합니다. 어느 서버에 배치할지, 장애가 난 인스턴스를 어떻게 대체할지, 트래픽을 어느 복제본으로 보낼지, 새 버전을 어떻게 순차 교체할지 자동으로 결정해야 합니다.

<details>
<summary>답</summary>

Docker가 개별 컨테이너의 생성과 실행을 담당한다면 Kubernetes는 여러 노드에 걸친 배치, 복제, 복구, 네트워크 노출과 업데이트를 조정합니다.

</details>

## 2. 선언적 상태와 조정 루프

사용자는 매 순간의 명령 대신 “복제본 3개를 유지한다”처럼 목표를 선언합니다. 컨트롤러는 목표 상태와 실제 상태를 비교하고 차이가 있으면 새 Pod를 만들거나 불필요한 Pod를 종료합니다.

```text
Desired state → Compare → Reconcile → Actual state
       ↑                         │
       └──────── repeat ─────────┘
```

## 3. 클러스터 구성

| 영역 | 구성 요소 | 역할 |
|---|---|---|
| Control Plane | API Server | 모든 요청의 진입점과 검증 |
| Control Plane | Scheduler | Pod를 실행할 노드 선택 |
| Control Plane | Controller | 목표 상태와 실제 상태 조정 |
| Control Plane | etcd | 클러스터 상태 저장 |
| Worker Node | kubelet | 노드의 Pod 실행 상태 관리 |
| Worker Node | Container Runtime | 컨테이너 실행 |
| Worker Node | kube-proxy | 서비스 네트워크 규칙 관리 |

## 4. 핵심 객체

- **Pod**: 하나 이상의 밀접한 컨테이너가 네트워크와 저장소를 공유하는 최소 배포 단위입니다.
- **Deployment**: Pod 템플릿과 복제본 수를 관리하고 롤링 업데이트를 수행합니다.
- **Service**: 교체되는 Pod 집합 앞에 안정적인 이름과 접근점을 제공합니다.
- **ConfigMap·Secret**: 실행 이미지와 환경 설정을 분리합니다. 민감 정보는 저장·전달·권한 정책을 함께 설계해야 합니다.

## 5. 배포와 장애 복구 흐름

1. Deployment에 이미지와 복제본 수를 선언합니다.
2. API Server가 선언을 저장합니다.
3. Controller가 필요한 Pod를 계산합니다.
4. Scheduler가 각 Pod의 노드를 선택합니다.
5. kubelet이 컨테이너를 실행하고 상태를 보고합니다.
6. Pod가 사라지면 Controller가 대체 Pod를 요청합니다.

## 직접 해보기

1. 웹 애플리케이션 복제본 3개를 유지하는 데 필요한 객체를 고르세요.
2. Pod가 교체되어 주소가 바뀌어도 사용자가 안정적으로 접근하게 할 객체를 적으세요.
3. 특정 노드가 중단됐을 때 복구에 참여하는 구성 요소의 흐름을 설명하세요.

<details>
<summary>정답 보기</summary>

1. Deployment에 복제본 수 3을 선언합니다.
2. Service가 선택 조건에 맞는 Pod 집합에 안정적인 접근점을 제공합니다.
3. 노드와 Pod 상태가 갱신되면 Controller가 부족한 복제본을 감지하고 Scheduler가 새 노드를 선택하며 해당 노드의 kubelet이 Pod를 실행합니다.

</details>

## 복습 질문 및 답변

### Q1. 명령형 운영과 선언형 운영의 차이는 무엇인가요?

<details>
<summary>답</summary>

명령형 운영은 수행할 절차를 직접 지시하고, 선언형 운영은 원하는 결과를 기록합니다. Kubernetes는 현재 상태를 계속 관찰해 선언한 결과에 맞춥니다.

</details>

### Q2. Pod를 직접 여러 개 만드는 것보다 Deployment가 나은 이유는 무엇인가요?

<details>
<summary>답</summary>

Deployment는 복제본 수, 새 버전 교체, 실패한 Pod의 재생성을 지속적으로 관리하므로 개별 Pod 수명에 의존하지 않습니다.

</details>

### Q3. Control Plane과 Worker Node의 책임은 어떻게 나뉘나요?

<details>
<summary>답</summary>

Control Plane은 상태를 저장하고 배치와 조정을 결정합니다. Worker Node는 결정된 Pod를 실제로 실행하고 네트워크와 실행 상태를 보고합니다.

</details>

## 다음 학습

- [이전: 컨테이너와 Docker](./03-containers-and-docker.md)
- [다음: 마이크로서비스와 Serverless](./05-microservices-and-serverless.md)
