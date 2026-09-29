# 커스텀 이미지를 빌드하고 실행하기

> 이미지를 직접 만들면 애플리케이션 코드와 실행 규칙을 하나의 배포 단위로 검증할 수 있다.

`build context` · `Dockerfile` · `port mapping` · `logs` · `lifecycle`

## 핵심요약

- `docker build`는 Dockerfile과 빌드 컨텍스트를 이용해 이미지를 만든다.
- `docker run`으로 컨테이너를 만들 때 이름, 포트, 환경을 명시한다.
- 접속 실패는 앱 상태, 포트 매핑, 로그를 순서대로 확인하면 좁혀갈 수 있다.

## 1. 빌드 컨텍스트 이해하기

빌드 명령 끝의 `.`은 현재 폴더를 Docker daemon에 전달하는 **빌드 컨텍스트**로 뜻한다. 컨텍스트가 너무 크면 빌드가 느려지고 불필요한 파일이 이미지에 포함될 수 있으므로 `.dockerignore`로 제외 목록을 관리한다.

```dockerfile
FROM nginx:stable
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t custom-web:1.0 .
docker run -d --name custom-web -p 8080:80 custom-web:1.0
docker logs custom-web
```

## 2. 접속이 안 될 때의 진단 순서

1. `docker ps -a`로 컨테이너가 실행 상태인지 확인한다.
2. `docker logs 컨테이너이름`으로 앱 시작 오류를 확인한다.
3. 컨테이너가 앱 포트에서 수신하는지와 `-p` 매핑이 일치하는지 확인한다.
4. 필요한 경우 `docker exec`로 내부 파일과 프로세스를 점검한다.

| 증상 | 우선 확인 | 흔한 원인 |
| --- | --- | --- |
| 컨테이너가 바로 종료 | logs | 시작 명령 또는 의존성 오류 |
| 브라우저에서 접속 불가 | ps, 포트 매핑 | `-p` 누락, 앱 포트 불일치 |
| 변경 사항 미반영 | build context, tag | 이전 이미지·컨테이너를 재사용 |

## 직접 해보기

1. 빌드 컨텍스트에서 제외할 파일을 관리하는 파일 이름을 적어 보세요.
2. 컨테이너의 표준 출력 로그를 보는 명령을 적어 보세요.
3. 컨테이너가 종료됐는데도 이름이 충돌하면 어떤 순서로 정리할지 적어 보세요.

<details>
<summary>정답 보기</summary>

1. `.dockerignore`이다.
2. `docker logs 컨테이너이름`이다.
3. `docker ps -a`로 대상을 확인하고, 필요하면 `docker rm 컨테이너이름`으로 기존 컨테이너를 제거한 뒤 다시 실행한다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
| --- | --- |
| build context vs image | 빌드에 전달하는 파일 범위 vs 빌드 결과물 |
| logs vs exec | 앱이 출력한 기록 확인 vs 내부에서 추가 진단 명령 실행 |

## 연결되는 개념

- 이전 글: [Dockerfile 지시어](05-dockerfile-instructions.md)
- 다음 글: [FastAPI 컨테이너화](07-containerizing-fastapi.md)

## 셀프 체크

- [ ] 빌드 컨텍스트가 무엇인지 설명할 수 있다.
- [ ] 접속 실패 시 진단 순서를 말할 수 있다.
- [ ] 포트 매핑의 역할을 이해한다.

### 복습 질문 및 답변

**Q1. `.dockerignore`는 왜 필요한가요?**

<details>
<summary>답</summary>

빌드에 필요 없는 파일이 컨텍스트와 이미지에 들어가는 것을 줄여 빌드 속도, 이미지 크기, 노출 위험을 낮춘다.

</details>

**Q2. 컨테이너가 실행 중이라는 사실만으로 웹 서비스가 정상인가요?**

<details>
<summary>답</summary>

아니다. 애플리케이션이 기대한 포트에서 정상 응답하는지, 로그에 오류가 없는지까지 확인해야 한다.

</details>

**Q3. 새 이미지를 빌드한 뒤에도 예전 화면이 보이면 무엇을 점검하나요?**

<details>
<summary>답</summary>

빌드가 실제로 새 소스를 포함했는지, 새 태그를 실행했는지, 기존 컨테이너가 남아 있지 않은지 확인한다.

</details>

## 한 줄 정리

> 커스텀 이미지 실습은 빌드, 실행, 로그 확인, 정리까지의 짧은 배포 루프를 익히는 과정이다.
