# 컨테이너와 Docker

> 컨테이너는 운영체제 전체가 아니라 애플리케이션과 실행 의존성을 패키징해 빠르고 일관되게 실행합니다.

`container` · `Docker` · `image` · `namespace` · `cgroup`

## 핵심요약

- 컨테이너는 Host OS 커널을 공유하면서 프로세스 공간을 격리합니다.
- Namespace는 보이는 자원을, cgroup은 사용할 자원의 양을 제한합니다.
- Docker Image는 읽기 전용 실행 템플릿이고 Container는 실행 인스턴스입니다.
- Docker Client는 명령을 보내고 Daemon은 이미지와 컨테이너를 관리합니다.
- VM보다 가볍지만 커널 공유와 공급망 보안을 별도로 고려해야 합니다.

## 1. 컨테이너의 구조

컨테이너는 애플리케이션, 라이브러리, 설정을 하나의 실행 단위로 묶습니다. VM처럼 Guest OS 전체를 포함하지 않고 Host OS 커널을 공유합니다.

```text
Hardware
└─ Host OS kernel
   └─ Container runtime
      ├─ Container A: App + Libraries
      └─ Container B: App + Libraries
```

<details>
<summary>답</summary>

컨테이너는 완전히 독립된 운영체제가 아닙니다. 프로세스와 파일·네트워크 공간은 격리하지만 커널은 Host OS와 공유합니다.

</details>

## 2. Namespace와 cgroup

- **Namespace**는 프로세스 ID, 네트워크, 마운트, 사용자 등 컨테이너가 볼 수 있는 범위를 나눕니다.
- **cgroup**은 CPU, 메모리, I/O 같은 자원의 사용량을 제한하고 측정합니다.

Namespace만 있으면 서로 보이지 않게 할 수 있지만 한 컨테이너가 CPU와 메모리를 독점할 수 있습니다. cgroup까지 함께 사용해야 격리와 자원 통제가 결합됩니다.

## 3. Image와 Container

| 구분 | Image | Container |
|---|---|---|
| 의미 | 실행 템플릿 | 실행 중인 인스턴스 |
| 상태 | 읽기 전용 계층 중심 | 쓰기 가능한 실행 계층 추가 |
| 역할 | 배포와 재현 | 실제 프로세스 실행 |

Image를 버전으로 고정하면 같은 애플리케이션 구성을 여러 환경에 반복 배포할 수 있습니다. 단, 외부 데이터와 비밀은 이미지 안에 넣지 않고 볼륨과 비밀 관리 수단으로 분리합니다.

## 4. Docker Client와 Daemon

```text
docker CLI → Docker API / socket → Docker Daemon
                                  ├─ Images
                                  ├─ Containers
                                  ├─ Networks
                                  └─ Volumes
```

Client는 사용자 명령을 전달하고 Daemon은 실제 생성·실행·중지 작업을 수행합니다. Daemon 소켓에 접근할 수 있으면 강한 시스템 권한을 얻을 수 있으므로 접근 제어가 중요합니다.

## 5. VM과 비교

| 관점 | Container | VM |
|---|---|---|
| 커널 | Host와 공유 | Guest OS가 보유 |
| 시작 속도 | 빠름 | 상대적으로 느림 |
| 자원 오버헤드 | 낮음 | 상대적으로 큼 |
| OS 다양성 | Host 커널 제약 | 서로 다른 Guest OS 가능 |
| 격리 경계 | 프로세스 수준 | 하드웨어 가상화 수준 |

## 직접 해보기

1. Namespace와 cgroup의 역할을 각각 한 문장으로 설명하세요.
2. 컨테이너 이미지에 API 키를 넣으면 안 되는 이유를 적으세요.
3. 서로 다른 운영체제 커널이 필요한 두 애플리케이션의 실행 방식을 선택하세요.

<details>
<summary>정답 보기</summary>

1. Namespace는 보이는 자원을 분리하고 cgroup은 사용할 자원의 양을 제한합니다.
2. 이미지 레이어와 레지스트리 기록에 비밀이 남아 배포 대상 전체로 확산될 수 있습니다.
3. 서로 다른 커널이 필요하면 각 Guest OS를 가진 VM을 사용하고 그 안에서 컨테이너를 실행할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Image vs Container | 실행 템플릿과 그 템플릿에서 시작한 프로세스입니다. |
| Namespace vs cgroup | 가시 범위 격리와 자원 사용량 제어입니다. |
| Container vs VM | 커널을 공유하는 프로세스 격리와 Guest OS를 포함한 하드웨어 가상화입니다. |

## 연결되는 개념

- 이전: [OpenStack 핵심 서비스](02-openstack-core-services.md)
- 다음: [Kubernetes 오케스트레이션](04-kubernetes-orchestration.md)

## 셀프 체크

- [ ] 컨테이너의 커널 공유 구조를 설명한다.
- [ ] Namespace와 cgroup을 구분한다.
- [ ] Image와 Container를 구분한다.
- [ ] Docker Client와 Daemon의 역할을 안다.
- [ ] VM과 컨테이너를 상황에 맞게 고른다.

### 복습 질문 및 답변

**Q1. 컨테이너가 가벼운 이유는 무엇인가요?**

<details>
<summary>답</summary>

각 실행 단위가 Guest OS 전체를 포함하지 않고 Host OS 커널을 공유하기 때문입니다.

</details>

**Q2. 컨테이너를 삭제하면 데이터도 모두 안전하게 남나요?**

<details>
<summary>답</summary>

기본 쓰기 계층의 데이터는 함께 사라질 수 있습니다. 영구 데이터는 볼륨이나 외부 저장소에 분리해야 합니다.

</details>

**Q3. 컨테이너는 VM보다 항상 안전한가요?**

<details>
<summary>답</summary>

아닙니다. 커널을 공유하므로 이미지 취약점, 과도한 권한, 런타임 설정과 Host 보안을 함께 관리해야 합니다.

</details>

## 한 줄 정리

> 컨테이너는 커널을 공유하되 프로세스와 자원을 격리하여 애플리케이션을 빠르고 일관되게 실행하는 방식입니다.
