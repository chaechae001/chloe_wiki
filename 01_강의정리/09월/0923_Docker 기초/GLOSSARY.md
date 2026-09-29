# Docker 용어집

Docker 학습에서 반복되는 핵심 용어를 짧고 정확하게 정리했습니다.

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| --- | --- | --- |
| 컨테이너 | 앱을 격리된 실행 환경에서 돌리는 단위 | [컨테이너와 격리](01-containers-and-isolation.md) | namespace, cgroup |
| 가상머신 | 가상 하드웨어 위에서 별도 OS를 실행하는 방식 | [컨테이너와 격리](01-containers-and-isolation.md) | hypervisor |
| namespace | 프로세스가 보는 PID·네트워크·파일 시스템 범위를 분리하는 기능 | [컨테이너와 격리](01-containers-and-isolation.md) | 격리 |
| cgroup | CPU·메모리 같은 자원 사용량을 제어하는 기능 | [컨테이너와 격리](01-containers-and-isolation.md) | 리소스 제한 |
| Docker daemon | 이미지·컨테이너 등 실제 Docker 작업을 수행하는 백그라운드 서비스 | [Docker 구성 요소](02-docker-architecture.md) | Docker CLI |
| image | 컨테이너 실행에 필요한 파일과 규칙을 담은 템플릿 | [Docker 구성 요소](02-docker-architecture.md) | layer, tag |
| container | 이미지에서 만들어진 실행 인스턴스 | [Docker 구성 요소](02-docker-architecture.md) | lifecycle |
| registry | 이미지를 저장·공유하는 서비스 | [이미지와 레지스트리](04-images-and-registries.md) | pull, push |
| tag | 이미지 버전을 식별하는 이름표 | [이미지와 레지스트리](04-images-and-registries.md) | 재현성 |
| Dockerfile | 이미지를 빌드하는 선언형 규칙 파일 | [Dockerfile 지시어](05-dockerfile-instructions.md) | build |
| layer | Dockerfile 단계 결과를 쌓아 둔 이미지 구성 단위 | [Dockerfile 지시어](05-dockerfile-instructions.md) | build cache |
| build context | 이미지 빌드 시 daemon에 전달하는 파일 범위 | [커스텀 이미지 빌드](06-build-and-run-custom-images.md) | .dockerignore |
| port mapping | 호스트 포트와 컨테이너 포트를 연결하는 설정 | [Docker 명령 흐름](03-docker-cli-workflow.md) | EXPOSE |
| EXPOSE | 컨테이너가 사용할 포트 의도를 이미지에 표현하는 지시어 | [Dockerfile 지시어](05-dockerfile-instructions.md) | `docker run -p` |
| Uvicorn | Python ASGI 애플리케이션을 실행하는 서버 | [FastAPI 컨테이너화](07-containerizing-fastapi.md) | FastAPI |
