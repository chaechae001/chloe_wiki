# OpenStack 핵심 서비스

> VM 한 대가 만들어지려면 컴퓨트, 이미지, 인증, 네트워크, 블록·오브젝트 스토리지 서비스가 계약에 따라 협력해야 합니다.

`Nova` · `Neutron` · `Glance` · `Keystone` · `Cinder` · `Swift`

## 핵심요약

- Nova는 VM 생명주기와 컴퓨트 호스트를 조정합니다.
- Neutron은 가상 네트워크와 주소·라우팅을 제공합니다.
- Glance는 VM 이미지 카탈로그를 관리합니다.
- Keystone은 신원·토큰·역할·서비스 카탈로그를 담당합니다.
- Cinder는 블록 볼륨, Swift는 오브젝트 저장을 제공합니다.

## 1. 서비스 지도

| 서비스 | 담당 영역 | 대표 객체 |
|---|---|---|
| Nova | Compute | Instance, Flavor |
| Neutron | Network | Network, Subnet, Port |
| Glance | Image | VM Image |
| Keystone | Identity | User, Project, Role, Token |
| Cinder | Block Storage | Volume, Snapshot |
| Swift | Object Storage | Account, Container, Object |
| Horizon | Dashboard | Web UI |

<details>
<summary>답</summary>

서비스 이름보다 책임 경계를 먼저 이해해야 합니다. 특정 서비스 장애가 VM 생성의 어느 단계에 영향을 주는지 추적할 수 있기 때문입니다.

</details>

## 2. Nova와 Neutron

Nova API는 컴퓨팅 요청을 받고 Scheduler가 적합한 호스트를 선택하며 Compute 프로세스가 하이퍼바이저와 통신합니다. Neutron은 VM이 연결될 네트워크, 서브넷, 포트와 라우팅을 준비합니다.

```text
Nova: request → schedule → spawn instance
Neutron: network → subnet → port → routing
```

VM이 실행 중이어도 포트나 라우팅 구성이 잘못되면 외부와 통신하지 못합니다. 컴퓨팅 성공과 네트워크 성공을 별도로 확인해야 합니다.

## 3. Glance와 Keystone

Glance는 VM을 시작할 운영체제 이미지를 등록·조회·배포합니다. 이미지의 출처, 형식, 버전, 보안 패치를 관리해야 합니다.

Keystone은 사용자를 인증하고 토큰을 발급합니다. 각 서비스는 토큰을 검증하고 역할과 정책에 따라 작업을 허용합니다.

```text
Credentials → Token → Service request + Token → Policy check → Response
```

## 4. Cinder와 Swift

| 구분 | Cinder | Swift |
|---|---|---|
| 저장 모델 | Block | Object |
| 접근 방식 | VM에 볼륨 연결 | API로 객체 조회 |
| 대표 용도 | OS·DB 디스크 | 이미지·백업·정적 파일 |
| 주요 작업 | Create, Attach, Detach | Put, Get, Delete |

Cinder 볼륨은 VM과 분리해 보존하거나 다른 VM에 다시 연결할 수 있습니다. Swift 객체는 계정·컨테이너·객체의 논리 구조로 관리하며 고유 키와 API로 접근합니다.

## 5. 서비스 간 의존성

VM 생성은 한 서비스의 성공만으로 끝나지 않습니다.

1. Keystone에서 요청자를 인증합니다.
2. Glance에서 이미지를 찾습니다.
3. Nova가 호스트를 선택합니다.
4. Neutron이 포트를 준비합니다.
5. 필요하면 Cinder 볼륨을 연결합니다.
6. Nova가 인스턴스를 부팅합니다.

이 흐름을 알면 오류 메시지를 서비스 경계별로 좁힐 수 있습니다.

## 직접 해보기

1. VM은 생성됐지만 IP가 연결되지 않은 경우 담당 서비스를 고르세요.
2. 데이터베이스용 영구 디스크와 이미지 파일 저장소를 각각 선택하세요.
3. 토큰은 발급됐지만 볼륨 생성이 거부되는 원인을 두 가지 적으세요.

<details>
<summary>정답 보기</summary>

1. Neutron의 포트·서브넷·라우팅·할당량을 확인합니다.
2. 영구 디스크는 Cinder, 이미지 파일은 Swift 같은 오브젝트 스토리지가 적합합니다.
3. 역할 정책에 볼륨 생성 권한이 없거나 프로젝트 할당량을 초과했을 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Glance image vs Cinder volume | VM 생성 템플릿과 실행 중 데이터를 담는 블록 장치입니다. |
| Cinder vs Swift | VM에 연결하는 블록 저장과 API로 객체를 다루는 저장입니다. |
| Keystone authentication vs authorization | 신원 확인과 허용 행동 판단입니다. |

## 연결되는 개념

- 이전: [OpenStack과 프라이빗 클라우드 제어 구조](01-openstack-control-plane.md)
- 다음: [컨테이너와 Docker](03-containers-and-docker.md)

## 셀프 체크

- [ ] 핵심 서비스의 책임을 구분한다.
- [ ] VM 생성에 필요한 서비스 순서를 설명한다.
- [ ] 블록과 오브젝트 스토리지를 구분한다.
- [ ] 토큰과 역할 정책의 관계를 안다.
- [ ] 오류를 서비스 경계별로 추적한다.

### 복습 질문 및 답변

**Q1. Horizon이 없어도 OpenStack을 사용할 수 있나요?**

<details>
<summary>답</summary>

가능합니다. Horizon은 웹 대시보드이며 CLI나 SDK로 각 API를 직접 호출할 수 있습니다.

</details>

**Q2. Nova가 모든 네트워크를 직접 관리하나요?**

<details>
<summary>답</summary>

현대 OpenStack 구조에서는 네트워크 책임을 Neutron이 담당하고 Nova가 필요한 연결을 요청합니다.

</details>

**Q3. 이미지가 오래되면 어떤 문제가 생기나요?**

<details>
<summary>답</summary>

새 VM에도 오래된 패키지와 알려진 취약점이 반복 배포될 수 있으므로 검증·패치·폐기 정책이 필요합니다.

</details>

## 한 줄 정리

> OpenStack의 핵심 서비스는 각자 한 책임을 맡고 API와 토큰을 통해 VM의 전체 생명주기를 함께 완성합니다.
