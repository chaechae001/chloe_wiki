# Dockerfile로 이미지의 실행 규칙 정의하기

> Dockerfile은 사람이 읽는 배포 설명서이면서 Docker가 이미지를 만드는 재현 가능한 빌드 규칙이다.

`FROM` · `WORKDIR` · `COPY` · `RUN` · `CMD`

## 핵심요약

- Dockerfile은 베이스 이미지, 파일 복사, 의존성 설치, 기본 실행 명령을 순서대로 선언한다.
- `RUN`은 빌드 시 실행되고, `CMD`는 컨테이너 시작 시 기본으로 실행된다.
- 레이어 캐시를 고려해 자주 바뀌지 않는 단계와 자주 바뀌는 단계를 분리한다.

## 1. 핵심 지시어

| 지시어 | 역할 | 기억할 점 |
| --- | --- | --- |
| `FROM` | 베이스 이미지 선택 | 출처와 버전을 명시 |
| `WORKDIR` | 컨테이너 내부 작업 위치 | 이후 경로의 기준점 |
| `COPY` | 빌드 컨텍스트 파일 복사 | 불필요한 파일은 제외 |
| `RUN` | 빌드 중 명령 실행 | 결과가 이미지 레이어에 남음 |
| `EXPOSE` | 사용할 포트 의도 표현 | 실제 공개는 run의 포트 매핑 |
| `CMD` | 시작 기본 명령 | 실행 시 다른 명령으로 바꿀 수 있음 |

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

## 2. RUN, CMD, ENTRYPOINT

`RUN`은 이미지를 만드는 중에 실행된다. `CMD`는 컨테이너를 시작할 때 기본값으로 실행된다. `ENTRYPOINT`는 컨테이너의 주 실행 프로그램을 고정하고 싶을 때 주로 쓴다.

| 비교 | 실행 시점 | 용도 |
| --- | --- | --- |
| RUN | image build | 패키지 설치, 파일 생성 |
| CMD | container start | 기본 실행 명령 또는 인자 |
| ENTRYPOINT | container start | 기본 실행 프로그램 고정 |

## 3. 캐시를 활용하는 순서

Docker는 Dockerfile 단계별 결과를 레이어로 관리한다. 의존성 목록을 먼저 복사·설치하고 앱 소스를 나중에 복사하면, 소스만 바뀐 경우 의존성 설치 레이어를 재사용하기 쉽다.

## 직접 해보기

1. 베이스 이미지를 지정하는 지시어를 적어 보세요.
2. 패키지 설치처럼 빌드 중 실행할 명령에는 RUN과 CMD 중 무엇을 쓰는지 답해 보세요.
3. EXPOSE와 `docker run -p`의 역할 차이를 설명해 보세요.

<details>
<summary>정답 보기</summary>

1. `FROM`이다.
2. `RUN`이다. CMD는 컨테이너 시작 시의 기본 실행 명령이다.
3. EXPOSE는 이미지가 쓰는 포트를 문서화하고, `-p`는 호스트 포트를 컨테이너 포트에 실제 연결한다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
| --- | --- |
| COPY vs ADD | 일반 파일 복사는 COPY가 예측하기 쉬움 |
| RUN vs CMD | 빌드 과정 실행 vs 컨테이너 시작 시 기본 실행 |

## 연결되는 개념

- 이전 글: [이미지와 레지스트리](04-images-and-registries.md)
- 다음 글: [커스텀 이미지 빌드](06-build-and-run-custom-images.md)

## 셀프 체크

- [ ] Dockerfile의 주요 지시어를 역할별로 구분할 수 있다.
- [ ] RUN과 CMD의 실행 시점을 안다.
- [ ] 캐시 친화적인 순서를 설명할 수 있다.

### 복습 질문 및 답변

**Q1. WORKDIR를 지정하면 어떤 장점이 있나요?**

<details>
<summary>답</summary>

이후 COPY, RUN, CMD에서 경로를 일관되게 다룰 수 있어 Dockerfile이 더 읽기 쉬워진다.

</details>

**Q2. Dockerfile에 비밀값을 직접 쓰면 왜 위험한가요?**

<details>
<summary>답</summary>

이미지 레이어와 빌드 기록에 남아 공유 과정에서 노출될 수 있다. 비밀값은 안전한 런타임 주입 방식을 사용해야 한다.

</details>

**Q3. 소스보다 의존성 파일을 먼저 복사하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

소스가 바뀌어도 의존성 목록이 같다면 설치 레이어를 재사용할 가능성이 커져 빌드 시간을 줄일 수 있다.

</details>

## 한 줄 정리

> Dockerfile은 이미지의 재현 가능한 빌드와 실행 규칙을 선언하며, 지시어의 실행 시점을 구분하는 것이 핵심이다.
