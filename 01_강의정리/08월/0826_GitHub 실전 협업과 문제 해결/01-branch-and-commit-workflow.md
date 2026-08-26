# 브랜치와 커밋 실전 워크플로우

협업에서는 코드를 빨리 작성하는 것만큼 현재 위치와 변경 범위를 명확히 확인하는 습관이 중요합니다.

**핵심 키워드:** 브랜치, 작업 트리, diff, 스테이징, 커밋

## 핵심 내용

- 새 작업은 최신 기준 브랜치에서 목적별 브랜치를 만들어 시작합니다.
- `git status`는 현재 브랜치와 변경 파일의 상태를 함께 보여 줍니다.
- `git diff`로 실제 변경 줄을 검토한 뒤 필요한 파일만 스테이징합니다.
- 커밋은 한 가지 목적을 표현하는 작은 단위로 만듭니다.
- 첫 푸시에서 upstream을 설정하면 이후 동기화 명령이 간결해집니다.

## 작업 브랜치 만들기

```bash
git switch main
git pull --ff-only
git switch -c feature/readme-guide
git branch --show-current
```

- **목적:** 최신 기준에서 독립된 작업 공간을 만듭니다.
- **흐름:** 기준 브랜치 이동 → 최신화 → 생성과 전환 → 위치 확인입니다.
- **결과:** 새 커밋은 `feature/readme-guide`에 쌓입니다.
- **실무 포인트:** `git switch -c`는 브랜치 생성과 전환 의도가 명확합니다. 오래된 환경에서는 `git checkout -b`도 같은 목적에 사용됩니다.

## 변경을 검토하고 기록하기

```bash
git status
git diff
git add README.md
git diff --staged
git commit -m "docs: 사용 방법 보완"
```

| 명령 | 확인 대상 | 작업 파일 변경 여부 |
|---|---|---|
| `git status` | 파일 상태와 현재 브랜치 | 변경하지 않음 |
| `git diff` | 스테이징 전 변경 | 변경하지 않음 |
| `git diff --staged` | 다음 커밋에 포함될 변경 | 변경하지 않음 |
| `git add` | 스테이징 영역 | 작업 파일은 유지 |

`git add .`는 편리하지만 로그, 임시 파일, 관계없는 수정까지 포함할 수 있습니다. 커밋 직전에는 `git diff --staged`로 실제 포함 범위를 확인합니다.

## 원격에 작업 브랜치 게시하기

```bash
git remote -v
git push -u origin feature/readme-guide
```

`-u`는 로컬 브랜치와 원격 추적 브랜치를 연결합니다. 이후 같은 브랜치에서는 보통 `git push`와 `git pull`만 사용할 수 있습니다.

## 실습

1. 최신 `main`에서 `feature/readme-guide` 브랜치를 만드세요.
2. 파일을 수정하고 스테이징 전·후 차이를 각각 확인하세요.
3. 목적이 다른 두 변경을 별도 커밋으로 나누세요.

<details>
<summary>답</summary>

```bash
git switch main
git pull --ff-only
git switch -c feature/readme-guide
git diff
git add README.md
git diff --staged
git commit -m "docs: README 안내 추가"
```

</details>

## 더 알아보기

- [Fork와 upstream 협업](02-fork-and-upstream-collaboration.md)
- [Pull Request 작성과 리뷰](03-pull-request-review-etiquette.md)

## 체크리스트

- [ ] 작업 전 기준 브랜치를 최신화한다.
- [ ] 브랜치 이름에 작업 목적을 담는다.
- [ ] `status`와 `diff`로 변경을 검토한다.
- [ ] 스테이징된 차이를 커밋 전에 확인한다.
- [ ] 한 커밋에 한 가지 목적을 담는다.

## 복습 질문 및 답변

### Q1. `git diff`와 `git diff --staged`는 무엇이 다른가요?

<details>
<summary>답</summary>

기본 `git diff`는 아직 스테이징하지 않은 변경을, `--staged`는 다음 커밋에 포함될 변경을 보여 줍니다.

</details>

### Q2. 브랜치를 만든 직후 현재 위치를 확인해야 하는 이유는 무엇인가요?

<details>
<summary>답</summary>

잘못된 기준 브랜치나 보호 브랜치에서 커밋하는 실수를 예방하고 작업 이력을 의도한 곳에 남기기 위해서입니다.

</details>

### Q3. 첫 푸시의 `-u` 옵션은 무엇을 설정하나요?

<details>
<summary>답</summary>

현재 로컬 브랜치가 기본적으로 동기화할 원격 추적 브랜치를 설정합니다.

</details>

## 요약

실전 Git 작업은 위치 확인, 변경 검토, 선택적 스테이징, 목적별 커밋과 원격 게시의 반복입니다. 명령보다 각 단계에서 무엇을 확인하는지가 협업 품질을 결정합니다.
