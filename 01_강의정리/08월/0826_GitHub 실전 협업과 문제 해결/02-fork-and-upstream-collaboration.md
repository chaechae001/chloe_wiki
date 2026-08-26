# Fork와 upstream 협업

원본 저장소에 직접 쓰기 권한이 없어도 Fork를 사용하면 자신의 저장소에서 안전하게 변경을 만들고 기여할 수 있습니다.

**핵심 키워드:** Fork, origin, upstream, 동기화, 기여

## 핵심 내용

- Fork는 다른 저장소를 자신의 계정 아래 독립된 원격 저장소로 복사합니다.
- 일반적으로 `origin`은 내 Fork, `upstream`은 원본 저장소를 가리킵니다.
- Fork 이후 두 저장소의 변경은 자동으로 동기화되지 않습니다.
- 작업 전 upstream의 최신 이력을 받아 내 기준 브랜치를 갱신합니다.
- 기여는 Fork의 작업 브랜치에서 원본 저장소로 Pull Request를 보내는 방식으로 진행합니다.

## 원본과 Fork의 관계

| 항목 | 원본 저장소 | Fork 저장소 |
|---|---|---|
| 소유자 | 프로젝트 관리자 | 기여자 |
| 기본 원격 이름 | `upstream`으로 등록 | `origin`으로 자동 등록 |
| 직접 푸시 | 권한 필요 | 소유자가 가능 |
| 변경 전달 | PR을 받음 | PR의 head 브랜치 제공 |

Fork는 일회성 파일 복사가 아니라 별도의 Git 저장소입니다. 원본에 새 커밋이 생겨도 Fork에 자동 반영되지 않으므로 주기적인 동기화가 필요합니다.

## 원격 구성과 동기화

```bash
git clone <fork-url>
cd <repository-name>
git remote add upstream <original-url>
git remote -v
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

- **목적:** 원본의 최신 커밋을 내 Fork의 기준 브랜치에 반영합니다.
- **흐름:** Fork 복제 → upstream 등록 → 원본 조회 → 로컬 기준 갱신 → Fork 푸시입니다.
- **결과:** `origin/main`이 원본의 최신 상태를 따라갑니다.
- **실무 포인트:** 개인 작업 커밋은 `main`이 아니라 별도 기능 브랜치에 두면 동기화가 단순해집니다.

## Fork 기반 기여 흐름

```text
원본 동기화 → 기능 브랜치 생성 → 수정과 테스트
→ origin에 푸시 → upstream을 대상으로 PR → 리뷰와 병합
```

PR을 만들 때 base 저장소·브랜치와 head 저장소·브랜치를 반드시 확인합니다. 방향을 반대로 선택하면 의도하지 않은 변경 요청이 만들어질 수 있습니다.

## 실습

1. `origin`과 `upstream`이 각각 무엇을 가리키는지 설명하세요.
2. upstream의 `main`을 내 Fork에 반영하는 명령 순서를 작성하세요.
3. Fork의 `main`에 직접 기능 커밋을 쌓을 때 생기는 문제를 적으세요.

<details>
<summary>답</summary>

`origin`은 일반적으로 내 Fork, `upstream`은 원본입니다. `fetch upstream` 후 로컬 `main`에서 `merge --ff-only upstream/main`을 실행하고 `origin/main`으로 푸시합니다. 기능 커밋을 별도 브랜치에 두면 원본 동기화와 PR 관리가 쉬워집니다.

</details>

## 더 알아보기

- [브랜치와 커밋 실전 워크플로우](01-branch-and-commit-workflow.md)
- [Pull Request 작성과 리뷰](03-pull-request-review-etiquette.md)

## 체크리스트

- [ ] Fork가 독립 저장소임을 이해한다.
- [ ] `origin`과 `upstream` 주소를 확인한다.
- [ ] 작업 전 원본의 최신 이력을 가져온다.
- [ ] 기능 작업을 별도 브랜치에 둔다.
- [ ] PR의 base와 head 방향을 확인한다.

## 복습 질문 및 답변

### Q1. Fork의 변경이 원본에 자동으로 반영되나요?

<details>
<summary>답</summary>

아닙니다. 두 저장소는 독립적이며, 변경을 원본에 반영하려면 일반적으로 Pull Request와 병합 과정이 필요합니다.

</details>

### Q2. upstream을 등록하는 이유는 무엇인가요?

<details>
<summary>답</summary>

원본 저장소의 새 커밋을 명시적으로 가져와 Fork와 작업 브랜치의 기준을 최신으로 유지하기 위해서입니다.

</details>

### Q3. 왜 Fork의 main을 깨끗하게 유지하나요?

<details>
<summary>답</summary>

원본 main과 빠르게 동기화할 수 있고, 기능별 PR의 변경 범위가 섞이지 않기 때문입니다.

</details>

## 요약

Fork 워크플로우는 권한을 분리하면서 외부 기여를 가능하게 합니다. origin과 upstream의 방향을 이해하고 원본 동기화 후 작업 브랜치에서 PR을 만드는 것이 핵심입니다.
