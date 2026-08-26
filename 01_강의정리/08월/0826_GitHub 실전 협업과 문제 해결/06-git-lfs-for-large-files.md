# Git LFS로 대용량 파일 관리

이미지, 영상, 모델 파일처럼 변경 차이를 계산하기 어려운 바이너리는 일반 Git 이력을 빠르게 비대하게 만들 수 있습니다.

**핵심 키워드:** Git LFS, 바이너리, 포인터, .gitattributes, 저장 한도

## 핵심 내용

- 일반 Git은 파일 상태를 객체로 보존하므로 큰 바이너리의 반복 변경에 비효율적일 수 있습니다.
- Git LFS는 실제 파일을 LFS 저장소에 두고 Git에는 작은 포인터를 기록합니다.
- 추적 패턴은 `.gitattributes`에 저장되므로 팀과 함께 커밋해야 합니다.
- 모든 협업자와 CI 환경이 LFS를 설치하고 파일을 내려받을 수 있어야 합니다.
- 호스팅 서비스의 저장 용량, 대역폭, 파일 크기와 비용 제한을 먼저 확인합니다.

## 일반 Git과 Git LFS

| 항목 | 일반 Git | Git LFS |
|---|---|---|
| Git 객체 | 실제 파일 내용 | 포인터 파일 |
| 큰 바이너리 이력 | 저장소가 빠르게 커질 수 있음 | 별도 LFS 저장소에 보관 |
| 추가 설치 | 불필요 | 클라이언트와 CI 설정 필요 |
| 호스팅 제한 | Git 저장소 정책 적용 | LFS 용량·대역폭 정책도 적용 |

Git LFS가 “무제한 저장”을 의미하지는 않습니다. 기술적으로 큰 파일을 다루는 구조와 서비스가 제공하는 사용 한도는 별개의 문제입니다.

## 기본 설정

```bash
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add assets/design.psd
git commit -m "chore: 디자인 파일을 LFS로 추적"
git lfs ls-files
```

- **목적:** 특정 패턴의 실제 파일을 LFS에 저장하도록 설정합니다.
- **흐름:** 클라이언트 초기화 → 패턴 등록 → 속성 파일과 대상 파일 커밋 → 확인입니다.
- **결과:** Git 이력에는 포인터가, LFS 서버에는 실제 파일이 저장됩니다.
- **실무 포인트:** 이미 일반 Git 이력에 들어간 대용량 파일은 track만으로 과거 이력이 줄지 않습니다. 이력 이전은 영향 범위를 검토해 별도로 진행합니다.

## 협업과 CI

저장소를 복제하거나 pull할 때 LFS 파일이 함께 내려오도록 환경을 구성합니다. CI에서 실제 대용량 자산이 필요한 테스트나 배포를 수행한다면 checkout 단계의 LFS 지원과 인증, 대역폭을 확인합니다.

```bash
git lfs pull
git lfs fetch
git lfs checkout
git lfs prune
```

`prune`은 로컬의 오래된 LFS 객체를 정리할 수 있으므로 필요한 객체와 복구 가능성을 확인한 뒤 사용합니다.

## 실습

1. `.psd` 파일을 LFS로 추적하고 설정 파일을 확인하세요.
2. 팀 도입 전 확인할 비용·환경 항목을 작성하세요.
3. 이미 커밋된 대용량 파일에 `track`만 실행하면 충분한지 설명하세요.

<details>
<summary>답</summary>

```bash
git lfs track "*.psd"
git add .gitattributes
git lfs ls-files
```

협업자와 CI 설치 여부, 호스팅의 용량·대역폭·파일 크기·비용 정책을 확인합니다. 새 추적 규칙은 과거 Git 객체를 자동으로 제거하지 않습니다.

</details>

## 더 알아보기

- [브랜치와 커밋 실전 워크플로우](01-branch-and-commit-workflow.md)
- [Git 고급 복구와 추적](07-advanced-git-recovery-and-debugging.md)

## 체크리스트

- [ ] LFS가 필요한 파일 유형을 정의했다.
- [ ] `.gitattributes`를 저장소에 커밋했다.
- [ ] 협업자와 CI의 LFS 환경을 준비했다.
- [ ] 서비스의 용량·대역폭·비용을 확인했다.
- [ ] 과거 이력 이전의 영향을 검토했다.

## 복습 질문 및 답변

### Q1. Git LFS는 실제 대용량 파일을 Git 저장소에 넣나요?

<details>
<summary>답</summary>

Git 이력에는 실제 파일을 가리키는 포인터를 두고, 실제 내용은 별도의 LFS 저장소에 보관합니다.

</details>

### Q2. `.gitattributes`를 왜 커밋해야 하나요?

<details>
<summary>답</summary>

어떤 파일 패턴을 LFS로 처리할지 팀과 자동화 환경이 동일하게 알 수 있도록 저장소 규칙으로 공유하기 위해서입니다.

</details>

### Q3. LFS를 사용하면 저장 비용을 고려하지 않아도 되나요?

<details>
<summary>답</summary>

아닙니다. 호스팅 서비스마다 LFS 저장 용량과 다운로드 대역폭, 파일 크기와 과금 정책이 있습니다.

</details>

## 요약

Git LFS는 큰 바이너리를 포인터로 분리해 Git 이력을 가볍게 유지합니다. 도입 효과는 파일 유형뿐 아니라 팀 환경, CI, 호스팅 한도와 과거 이력까지 함께 설계할 때 얻을 수 있습니다.
