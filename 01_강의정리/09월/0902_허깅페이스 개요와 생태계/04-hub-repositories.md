# Hub 저장소와 버전 관리

Hugging Face Hub는 모델, 데이터셋과 데모를 저장소 단위로 공유하고 협업하는 공간입니다. 파일 다운로드 사이트가 아니라 버전, 설명 문서와 사용 조건을 함께 관리하는 플랫폼으로 이해해야 합니다.

**핵심 키워드:** Hub, repository, revision, model, dataset, Space

## 세 가지 저장소

| 유형 | 주요 내용 | 확인할 문서 |
|---|---|---|
| Model | 가중치, 설정, 토크나이저 | Model Card |
| Dataset | 데이터 파일과 로딩 정보 | Dataset Card |
| Space | 데모 애플리케이션 코드 | README·설정 |

각 저장소는 Git 기반 이력을 가지며 ML 대용량 파일을 다루는 저장 구조를 사용합니다. 재현성이 중요하면 움직일 수 있는 기본 브랜치 이름만 믿지 말고 검토한 revision을 기록합니다.

```python
from huggingface_hub import snapshot_download

local_dir = snapshot_download(
    repo_id="organization/reviewed-model",
    revision="reviewed-commit-id",
    allow_patterns=["*.json", "*.safetensors", "README.md"],
)
print(local_dir)
```

예시 식별자는 실제 검토한 저장소와 커밋으로 교체합니다. 토큰은 코드에 쓰지 않고 환경변수나 공식 로그인 저장소로 관리합니다.

## 공개와 신뢰

공개 저장소라는 이유만으로 파일이 안전하거나 목적에 맞는 것은 아닙니다. 작성자, 변경 이력, 라이선스, 실행 파일, 외부 코드 필요 여부와 모델 카드의 한계를 확인합니다. `trust_remote_code`가 필요한 모델은 코드를 검토하고 revision을 고정한 뒤 격리된 환경에서 시험합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| 저장소 ID vs revision | 자원의 위치 vs 특정 버전 |
| public vs gated | 누구나 접근 vs 조건 동의·승인이 필요 |
| 모델 저장소 vs Space | 모델 자산 관리 vs 실행 가능한 데모 코드 |

## 직접 해보기

1. 재현 가능한 다운로드에 기록할 정보를 적으세요.
2. 외부 코드를 실행하는 모델의 검토 절차를 세우세요.
3. 비공개 토큰 관리 방법을 설명하세요.

<details>
<summary>정답 보기</summary>

1. 저장소 ID, revision, 파일 범위와 라이브러리 버전을 기록합니다.
2. 코드와 이력을 검토하고 버전을 고정해 격리 환경에서 실행합니다.
3. 최소 권한 토큰을 환경변수·비밀 저장소로 주입하고 코드와 로그에서 제외합니다.

</details>

## 연결되는 개념

- 이전: [Hugging Face 핵심 라이브러리](03-hugging-face-libraries.md)
- 다음: [Model Hub와 Dataset Hub 탐색](05-model-dataset-discovery.md)

## 셀프 체크

- [ ] 세 저장소 유형을 구분한다.
- [ ] revision이 필요한 이유를 설명한다.
- [ ] 접근 권한과 라이선스를 확인한다.
- [ ] 원격 코드 실행 위험을 이해한다.
- [ ] 토큰을 안전하게 관리한다.

### 복습 질문 및 답변

**Q1. Hub 저장소가 Git 기반인 장점은 무엇인가요?**

<details>
<summary>답</summary>

변경 이력과 특정 버전을 추적하고 협업 흐름을 적용할 수 있습니다.

</details>

**Q2. 기본 브랜치만 사용하면 재현성이 약해지는 이유는 무엇인가요?**

<details>
<summary>답</summary>

같은 이름이 나중에 다른 커밋을 가리킬 수 있기 때문입니다.

</details>

**Q3. 공개 모델을 바로 운영에 배포해도 되나요?**

<details>
<summary>답</summary>

아닙니다. 파일·코드·라이선스·품질과 안전성을 조직 기준으로 검증해야 합니다.

</details>

## 한 줄 정리

> Hub는 AI 자산의 파일과 설명, 이력과 협업을 저장소 단위로 연결하는 플랫폼입니다.
