# 브랜치 전략과 Git Flow

브랜치 전략은 이름 규칙이 아니라 어떤 변경을 어디서 시작하고 어디로 합칠지에 대한 팀의 약속입니다.

**핵심 키워드:** main, develop, feature, release, hotfix

## 핵심 내용

- `main`은 배포 가능한 안정 상태를 유지합니다.
- `develop`은 다음 배포에 들어갈 기능을 통합하는 브랜치로 사용할 수 있습니다.
- `feature/*`는 개별 기능을 짧게 개발하고 통합 후 삭제합니다.
- `release/*`는 배포 준비와 최종 수정, `hotfix/*`는 운영 긴급 수정을 담당합니다.
- 팀 규모와 배포 빈도에 맞춰 전략을 단순화해야 합니다.

## Git Flow의 브랜치 역할

| 브랜치 | 시작 기준 | 병합 대상 | 목적 |
|---|---|---|---|
| `main` | 배포 이력 | - | 운영 가능한 기준 |
| `develop` | `main` | `main` | 다음 버전 통합 |
| `feature/*` | `develop` | `develop` | 기능 개발 |
| `release/*` | `develop` | `main`, `develop` | 배포 준비 |
| `hotfix/*` | `main` | `main`, `develop` | 긴급 수정 |

## 기능 브랜치 흐름

```bash
git switch develop
git pull --ff-only
git switch -c feature/45-login-api
# 작업 및 테스트
git add .
git commit -m "feat: 로그인 API 연결"
git push -u origin feature/45-login-api
```

- **목적:** 기능 작업을 통합 브랜치와 분리합니다.
- **흐름:** 기준 최신화 → 브랜치 생성 → 구현·테스트 → 커밋 → 원격 게시입니다.
- **결과:** 독립된 브랜치로 PR을 만들 수 있습니다.
- **실무 포인트:** 브랜치 이름에 유형과 작업 식별자, 짧은 설명을 포함하면 검색이 쉽습니다.

## release와 hotfix

릴리스 브랜치에서는 새 기능보다 버전 조정, 문서, 배포 전 결함 수정에 집중합니다. 배포 후에는 태그를 남기고 릴리스 수정도 `develop`에 반영합니다. 핫픽스는 `main`에서 시작해 긴급 문제만 고친 뒤 `main`과 다음 개발선 모두에 반영하여 수정이 다시 사라지는 일을 막습니다.

```bash
git switch main
git switch -c hotfix/login-crash
# 수정 및 검증 후 PR 생성
```

## 전략을 단순화할 때

지속적으로 배포하는 작은 팀이라면 `main`과 짧은 기능 브랜치만 사용하는 GitHub Flow가 더 적합할 수 있습니다. 중요한 것은 복잡한 이름을 따르는 것이 아니라 기준 브랜치 보호, 리뷰, 자동 검사, 배포 규칙이 일관된가입니다.

## 실습

1. 로그인 기능용 브랜치 이름을 규칙에 맞게 작성하세요.
2. 운영 장애 수정 브랜치의 시작점과 병합 대상을 정하세요.
3. 소규모 팀에서 Git Flow를 단순화할 기준을 적으세요.

<details>
<summary>답</summary>

예: `feature/45-login-api`. 핫픽스는 `main`에서 시작해 검증 후 `main`과 다음 개발 브랜치에 모두 반영합니다. 배포 주기가 짧고 병렬 릴리스가 없다면 `main`과 짧은 기능 브랜치 중심으로 단순화할 수 있습니다.

</details>

## 더 알아보기

- [Pull Request 협업 흐름](06-pull-request-workflow.md)
- [병합 전략과 충돌 해결](07-merge-strategies-and-conflicts.md)

## 체크리스트

- [ ] 각 브랜치의 시작점과 병합 대상을 안다.
- [ ] 기능 브랜치를 짧게 유지한다.
- [ ] 일관된 브랜치 이름 규칙을 쓴다.
- [ ] release·hotfix 변경을 다음 개발선에도 반영한다.
- [ ] 팀 상황보다 복잡한 전략을 강요하지 않는다.

## 복습 질문 및 답변

### Q1. `main`에서 직접 기능을 개발하면 왜 위험한가요?

<details>
<summary>답</summary>

미완성 변경이 안정 기준에 섞이고 독립적인 리뷰와 되돌리기가 어려워집니다. 보호된 기준 브랜치와 작업 브랜치를 분리하는 편이 안전합니다.

</details>

### Q2. 핫픽스를 `main`에만 병합하면 어떤 문제가 생기나요?

<details>
<summary>답</summary>

다음 개발 브랜치에 수정이 없어 이후 배포에서 같은 문제가 다시 나타날 수 있습니다.

</details>

### Q3. 모든 팀이 Git Flow를 그대로 써야 하나요?

<details>
<summary>답</summary>

아닙니다. 병렬 개발과 정기 릴리스에는 유용하지만, 작은 팀과 빈번한 배포에서는 더 단순한 전략이 적합할 수 있습니다.

</details>

## 요약

브랜치 전략은 안정성과 작업 속도의 균형을 만드는 규칙입니다. 브랜치 역할, 시작점, 병합 대상과 보호 정책을 팀 상황에 맞게 명확히 합의해야 합니다.
