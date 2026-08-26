# GLOSSARY

## 용어 정리

| 용어 | 설명 |
|---|---|
| Git | 파일 변경 이력을 분산 방식으로 관리하는 버전 관리 시스템 |
| GitHub | Git 저장소 공유와 이슈·리뷰·자동화를 제공하는 협업 플랫폼 |
| Repository | 파일과 Git 이력이 함께 저장되는 저장소 |
| Working Tree | 사용자가 실제 파일을 편집하는 작업 공간 |
| Staging Area | 다음 커밋에 포함할 변경을 선택해 두는 영역 |
| Commit | 의미 있는 변경과 메타데이터를 기록한 이력 단위 |
| Branch | 특정 커밋을 가리키며 작업에 따라 이동하는 이름 |
| Remote | 로컬 저장소가 동기화할 외부 저장소의 별칭과 주소 |
| origin | 원격 저장소에 관례적으로 사용하는 기본 별칭 |
| clone | 원격 저장소의 파일과 이력을 로컬에 복제하는 작업 |
| push | 로컬 커밋을 원격 저장소에 전달하는 작업 |
| fetch | 작업 파일을 통합하지 않고 원격 이력만 갱신하는 작업 |
| pull | 원격 변경을 받아 현재 브랜치에 통합하는 작업 |
| Issue | 버그, 개선, 질문 등 추적할 작업 단위 |
| Pull Request | 한 브랜치의 변경을 다른 브랜치에 병합하기 위한 검토 요청 |
| Review | 변경의 요구사항, 설계, 코드와 검증 결과를 살피는 과정 |
| Merge | 서로 다른 브랜치의 이력을 통합하는 작업 |
| Merge Commit | 두 이력의 부모 관계를 보존하는 별도 병합 커밋 |
| Squash | 여러 커밋의 변경을 하나의 커밋으로 합치는 방식 |
| Rebase | 커밋을 새 기준 위에 다시 적용해 이력을 재구성하는 작업 |
| Conflict | Git이 변경을 자동으로 통합하지 못해 사람의 판단이 필요한 상태 |
| Git Flow | main, develop, feature, release, hotfix 역할을 구분하는 브랜치 모델 |
| Hotfix | 운영 환경의 긴급 문제를 수정하기 위한 작업 브랜치 |
| Tag | 특정 커밋에 버전처럼 고정된 이름을 붙이는 참조 |
| `.gitignore` | Git이 추적하지 않을 파일 패턴을 정의하는 파일 |

## 연결해서 기억하기

작업 트리에서 수정한 내용은 스테이징을 거쳐 커밋이 됩니다. 브랜치는 커밋 이력을 분리하고, 원격 저장소는 이 이력을 팀과 공유합니다. Issue가 작업의 이유를 설명한다면 Pull Request는 구현 결과와 검증 과정을 연결합니다.

## 관련 학습

- [Git과 GitHub 기초](01-git-and-github-basics.md)
- [로컬과 원격 저장소 워크플로우](03-local-remote-workflow.md)
- [브랜치 전략과 Git Flow](05-branching-and-git-flow.md)
- [병합 전략과 충돌 해결](07-merge-strategies-and-conflicts.md)
