# GLOSSARY

## 용어 정리

| 용어 | 설명 |
|---|---|
| Working Tree | 실제 파일을 수정하는 현재 작업 공간 |
| Staging Area | 다음 커밋에 포함할 변경을 선택해 두는 영역 |
| Upstream Branch | 로컬 브랜치가 기본적으로 추적하는 원격 브랜치 |
| Fork | 다른 원격 저장소를 자신의 계정 아래 독립 저장소로 복사하는 기능 |
| origin | 복제한 저장소가 기본으로 등록하는 원격 별칭; Fork 흐름에서는 보통 내 저장소 |
| upstream | 원본 프로젝트 저장소에 관례적으로 붙이는 원격 별칭 |
| Base Branch | Pull Request의 변경을 받아들이는 대상 브랜치 |
| Head Branch | Pull Request의 변경을 제공하는 브랜치 |
| Reviewer | Pull Request의 코드와 요구사항, 검증 결과를 검토하는 사람 |
| Merge Conflict | Git이 변경을 자동 통합하지 못해 사람의 결정이 필요한 상태 |
| HEAD | 현재 체크아웃한 커밋 또는 브랜치를 가리키는 참조 |
| Common Ancestor | 두 브랜치가 갈라지기 전 공유하는 기준 커밋 |
| 3-way Merge | 공통 조상, 현재 변경, 들어오는 변경을 비교해 통합하는 방식 |
| stash | 작업 트리의 미완성 변경을 임시로 보관하는 기능 |
| stash apply | stash를 적용하고 목록에는 유지하는 명령 |
| stash pop | stash를 적용한 뒤 성공 시 목록에서 제거하는 명령 |
| Git LFS | 대용량 파일을 별도 저장하고 Git에는 포인터를 기록하는 확장 도구 |
| Pointer File | 실제 LFS 파일의 식별 정보와 위치를 나타내는 작은 파일 |
| `.gitattributes` | 경로 패턴별 Git 처리 방식을 정의하는 저장소 설정 파일 |
| Annotated Tag | 작성자, 날짜와 메시지를 포함하는 주석 태그 |
| cherry-pick | 특정 커밋의 변경을 현재 브랜치에 새 커밋으로 적용하는 작업 |
| reflog | 로컬 HEAD와 브랜치 참조 이동을 기록한 로그 |
| bisect | 정상·문제 커밋 사이를 이진 탐색해 원인 커밋을 찾는 기능 |
| known-good | 문제가 없다고 확인된 기준 커밋 |

## 연결해서 기억하기

작업 브랜치에서 변경을 검토해 커밋하고, Fork 흐름에서는 origin에 푸시해 upstream으로 PR을 보냅니다. 충돌이나 긴급 전환에는 merge 도구와 stash를 사용하며, 큰 파일은 LFS로 분리합니다. 문제가 발생하면 reflog로 위치를 복구하고 bisect로 원인 시점을 좁힐 수 있습니다.

## 관련 학습

- [브랜치와 커밋 실전 워크플로우](01-branch-and-commit-workflow.md)
- [Fork와 upstream 협업](02-fork-and-upstream-collaboration.md)
- [Merge Conflict 해결](04-merge-conflict-resolution.md)
- [Git 고급 복구와 추적](07-advanced-git-recovery-and-debugging.md)
