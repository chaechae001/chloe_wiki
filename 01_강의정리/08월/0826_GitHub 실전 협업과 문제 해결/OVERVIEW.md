# GitHub 실전 협업과 문제 해결

브랜치에서 변경을 만드는 일상 작업부터 Fork 기반 기여, PR 리뷰, 충돌 해결, 대용량 파일과 고급 복구 도구까지 실전 흐름으로 학습합니다.

## 학습 목표

- 변경 범위를 검토해 목적별 커밋을 만듭니다.
- origin과 upstream을 구분해 Fork 기반으로 기여합니다.
- 설명과 검증 근거가 있는 Pull Request와 리뷰를 수행합니다.
- 충돌과 미완성 작업 전환을 안전하게 처리합니다.
- LFS와 고급 Git 도구를 상황에 맞게 선택합니다.

## 추천 학습 순서

1. [브랜치와 커밋 실전 워크플로우](01-branch-and-commit-workflow.md)
2. [Fork와 upstream 협업](02-fork-and-upstream-collaboration.md)
3. [Pull Request 작성과 리뷰](03-pull-request-review-etiquette.md)
4. [Merge Conflict 해결](04-merge-conflict-resolution.md)
5. [작업 중 변경 임시 보관](05-stash-and-work-in-progress.md)
6. [Git LFS로 대용량 파일 관리](06-git-lfs-for-large-files.md)
7. [Git 고급 복구와 추적](07-advanced-git-recovery-and-debugging.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
원본 최신화 → 작업 브랜치 → diff 검토 → 작은 커밋
→ 원격 푸시 → PR 설명과 리뷰 → 충돌·검사 확인 → 병합
→ 릴리스 표시·문제 복구·원인 추적
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| status, diff, staged commit | 01 브랜치와 커밋 실전 워크플로우 |
| origin, upstream, Fork 동기화 | 02 Fork와 upstream 협업 |
| base/head, 리뷰 매너 | 03 Pull Request 작성과 리뷰 |
| 충돌 마커와 3-way merge | 04 Merge Conflict 해결 |
| stash apply와 pop | 05 작업 중 변경 임시 보관 |
| 포인터와 .gitattributes | 06 Git LFS로 대용량 파일 관리 |
| tag, cherry-pick, reflog, bisect | 07 Git 고급 복구와 추적 |

## 최종 점검

- [ ] 현재 브랜치와 커밋 범위를 확인한다.
- [ ] Fork와 원본의 동기화 방향을 이해한다.
- [ ] PR에 변경 이유와 테스트 결과를 남긴다.
- [ ] 충돌 해결과 stash 복원 후 다시 검증한다.
- [ ] 고급 명령 전에 대상 커밋과 영향 범위를 확인한다.
