# 로컬과 원격 저장소 워크플로우

Git 작업은 파일을 수정하는 공간과 이력을 저장하는 공간, 팀이 공유하는 원격 공간 사이를 이동하는 과정입니다.

**핵심 키워드:** 작업 트리, 스테이징, 로컬 저장소, 원격 저장소, 동기화

## 핵심 내용

- 작업 트리는 실제로 파일을 수정하는 현재 디렉터리입니다.
- 스테이징 영역은 다음 커밋에 포함할 변경을 고르는 중간 단계입니다.
- `commit`은 로컬 저장소에 기록하고 `push`는 원격에 전달합니다.
- `fetch`는 원격 이력만 내려받고 `pull`은 내려받은 뒤 현재 브랜치에 통합합니다.
- 기존 원격 저장소를 시작점으로 삼을 때는 `clone`이 가장 간단합니다.

## 네 단계로 이해하기

| 공간 | 이동 명령 | 의미 |
|---|---|---|
| 작업 트리 | 파일 편집 | 아직 기록되지 않은 변경 |
| 스테이징 | `git add` | 다음 커밋 후보 |
| 로컬 저장소 | `git commit` | 내 컴퓨터의 확정 이력 |
| 원격 저장소 | `git push` | 팀이 공유하는 이력 |

## 원격 저장소 복제

```bash
git clone <repository-url>
cd <repository-name>
git remote -v
git status
```

- **목적:** 원격 저장소의 파일과 이력을 로컬에 복제합니다.
- **흐름:** 복제 → 폴더 이동 → 원격 연결 확인 → 상태 확인입니다.
- **결과:** 보통 `origin`이라는 원격 이름과 추적 브랜치가 설정됩니다.
- **실무 포인트:** 새 작업을 시작하기 전 기본 브랜치를 최신 상태로 맞추고 작업 브랜치를 만듭니다.

## 기존 폴더를 원격에 연결

```bash
git init
git add README.md
git commit -m "docs: 첫 문서 추가"
git branch -M main
git remote add origin <repository-url>
git push -u origin main
```

`-u`는 로컬 `main`과 원격 `origin/main`의 추적 관계를 설정합니다. 이후에는 보통 `git push`만으로 같은 대상에 보낼 수 있습니다. 원격에 이미 커밋이 있다면 무작정 강제 푸시하지 말고 먼저 이력을 확인합니다.

## fetch와 pull

`git fetch origin`은 원격 상태를 안전하게 확인할 때 유용합니다. 작업 파일을 즉시 바꾸지 않습니다. `git pull --rebase origin main`은 원격 변경을 받은 뒤 내 로컬 커밋을 그 위에 다시 놓아 선형 이력을 만들 수 있지만, 공유된 커밋을 대상으로 무분별하게 사용하지 않습니다.

## 실습

1. 원격 저장소를 복제하고 `origin` 주소를 확인하세요.
2. 파일 하나를 수정한 뒤 작업 트리, 스테이징, 커밋 상태를 차례로 확인하세요.
3. `fetch` 후 로컬과 원격 브랜치 차이를 확인하세요.

<details>
<summary>답</summary>

```bash
git clone <repository-url>
git remote -v
git status
git add README.md
git status
git commit -m "docs: 안내 수정"
git fetch origin
git log --oneline --left-right HEAD...origin/main
```

</details>

## 더 알아보기

- [저장소 설정과 보안](02-repository-setup-and-security.md)
- [Pull Request 협업 흐름](06-pull-request-workflow.md)

## 체크리스트

- [ ] `add`, `commit`, `push`의 저장 위치를 구분한다.
- [ ] 원격 이름과 주소를 확인할 수 있다.
- [ ] `clone`과 `init`의 시작 상황을 구분한다.
- [ ] `fetch`와 `pull`의 차이를 설명할 수 있다.
- [ ] 원격 이력을 확인하지 않은 강제 푸시를 피한다.

## 복습 질문 및 답변

### Q1. 커밋하면 GitHub에도 바로 반영되나요?

<details>
<summary>답</summary>

아닙니다. 커밋은 로컬 저장소에 기록됩니다. 원격에 반영하려면 해당 커밋을 `push`해야 합니다.

</details>

### Q2. `fetch`가 작업 중인 파일을 바꾸나요?

<details>
<summary>답</summary>

일반적으로 바꾸지 않습니다. 원격 추적 정보를 갱신하며, 실제 통합은 merge나 rebase 같은 별도 작업으로 수행합니다.

</details>

### Q3. 새 프로젝트에서는 항상 `clone`을 사용하나요?

<details>
<summary>답</summary>

원격 저장소가 시작점이면 `clone`을 사용합니다. 로컬 폴더가 먼저 존재한다면 `init` 후 원격을 연결할 수 있습니다.

</details>

## 요약

변경은 작업 트리에서 시작해 스테이징과 로컬 커밋을 거쳐 원격으로 이동합니다. 각 명령이 어느 공간을 바꾸는지 알면 동기화 문제를 훨씬 쉽게 진단할 수 있습니다.
