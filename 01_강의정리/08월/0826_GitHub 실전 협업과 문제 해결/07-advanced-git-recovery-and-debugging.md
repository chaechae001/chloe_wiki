# Git 고급 복구와 추적

문제가 생겼을 때 전체 이력을 되돌리기보다 필요한 커밋을 선택하고 이동 기록과 이진 탐색을 활용하면 더 안전하게 원인을 찾을 수 있습니다.

**핵심 키워드:** tag, cherry-pick, reflog, bisect, 복구

## 핵심 내용

- Tag는 릴리스처럼 중요한 커밋에 고정된 버전 이름을 붙입니다.
- Cherry-pick은 특정 커밋의 변경만 현재 브랜치에 새 커밋으로 적용합니다.
- Reflog는 로컬에서 HEAD와 브랜치 참조가 이동한 기록을 보여 줍니다.
- Bisect는 정상 커밋과 문제 커밋 사이를 이진 탐색해 최초 문제 지점을 찾습니다.
- 고급 명령 전에는 작업 트리를 정리하고 대상 커밋과 공유 범위를 확인합니다.

## 도구 선택

| 상황 | 도구 | 핵심 질문 |
|---|---|---|
| 릴리스 지점 표시 | `git tag` | 어떤 커밋이 배포 버전인가? |
| 수정 하나만 다른 브랜치에 적용 | `git cherry-pick` | 어떤 커밋의 변경이 필요한가? |
| 잘못 이동한 HEAD 복구 | `git reflog` | 이전에 어디를 가리켰는가? |
| 버그 유입 커밋 탐색 | `git bisect` | 정상에서 문제로 바뀐 최초 지점은? |

## Tag와 cherry-pick

```bash
git tag -a v1.2.0 -m "release v1.2.0"
git push origin v1.2.0

git switch release/1.2
git cherry-pick <commit-hash>
```

- **목적:** 배포 지점을 식별하거나 필요한 수정만 선택적으로 반영합니다.
- **흐름:** 정확한 대상 커밋 확인 → 명령 실행 → 결과와 테스트 확인입니다.
- **결과:** Tag는 커밋을 가리키고, cherry-pick은 현재 브랜치에 새 커밋을 만듭니다.
- **실무 포인트:** cherry-pick된 변경은 원본 커밋과 식별자가 다르며 이후 병합에서 중복 맥락을 고려해야 합니다.

## Reflog로 위치 찾기

```bash
git reflog
git show HEAD@{2}
```

Reflog는 브랜치 삭제나 reset 후 이전 커밋을 찾는 데 유용하지만 로컬 기록이며 영구 백업이 아닙니다. 찾은 커밋이 중요하다면 새 브랜치나 태그로 참조를 만들어 보존합니다.

## Bisect로 원인 찾기

```bash
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# 각 체크아웃 지점에서 테스트 후 good 또는 bad 표시
git bisect reset
```

Bisect는 후보 범위를 절반씩 줄입니다. 판정에 사용할 테스트가 재현 가능해야 정확한 결과를 얻을 수 있습니다. 작업이 끝나면 `git bisect reset`으로 원래 상태로 돌아갑니다.

## 실습

1. 릴리스 커밋에 주석 태그를 만들고 원격에 게시하세요.
2. 삭제된 브랜치의 커밋을 reflog에서 찾아 보존하는 방법을 적으세요.
3. known-good와 bad 커밋으로 bisect를 시작하고 종료하세요.

<details>
<summary>답</summary>

```bash
git tag -a v1.2.0 -m "release v1.2.0"
git push origin v1.2.0
git reflog
git branch recovered-work <found-commit>
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
git bisect reset
```

</details>

## 더 알아보기

- [작업 중 변경 임시 보관](05-stash-and-work-in-progress.md)
- [Git LFS로 대용량 파일 관리](06-git-lfs-for-large-files.md)

## 체크리스트

- [ ] 명령 전에 작업 트리 상태를 확인한다.
- [ ] 커밋 해시와 대상 브랜치를 재확인한다.
- [ ] cherry-pick 후 테스트한다.
- [ ] reflog로 찾은 중요 커밋에 참조를 만든다.
- [ ] bisect 판정 기준을 재현 가능하게 만든다.

## 복습 질문 및 답변

### Q1. Tag와 Branch의 차이는 무엇인가요?

<details>
<summary>답</summary>

브랜치는 새 커밋에 따라 이동하는 작업 참조이고, Tag는 일반적으로 릴리스 같은 특정 커밋을 고정해 가리킵니다.

</details>

### Q2. Reflog는 원격 팀원과 공유되는 복구 기록인가요?

<details>
<summary>답</summary>

아닙니다. 기본적으로 각 로컬 저장소의 참조 이동 기록이므로 다른 컴퓨터나 원격의 reflog와 동일하지 않습니다.

</details>

### Q3. Bisect를 효과적으로 사용하려면 무엇이 필요한가요?

<details>
<summary>답</summary>

확실한 정상 커밋과 문제 커밋, 그리고 각 중간 커밋을 일관되게 good 또는 bad로 판정할 재현 가능한 테스트가 필요합니다.

</details>

## 요약

Tag, cherry-pick, reflog와 bisect는 각각 버전 표시, 선택 적용, 위치 복구, 원인 추적을 담당합니다. 정확한 커밋과 작업 상태를 확인하고 검증 가능한 절차로 사용해야 합니다.
