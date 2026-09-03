# Spaces와 생태계 연결

Spaces는 모델을 사람들이 직접 시험할 수 있는 웹 데모로 연결합니다. 데모가 있다는 사실은 접근성을 높이지만 품질 보증이나 무제한 운영 용량을 뜻하지는 않습니다.

**핵심 키워드:** Spaces, Gradio, Docker, demo, reproducibility

## 동작 구조

```text
브라우저 입력 → Space 애플리케이션 → 모델 호출
→ 예측 결과 가공 → 브라우저 출력
```

Space 코드도 저장소에서 버전 관리됩니다. SDK와 하드웨어, 공개 범위를 선택하고 README 설정으로 실행 조건을 기술합니다. 모델·데이터셋 저장소를 카드 메타데이터와 README에 연결하면 사용자가 구성 요소의 출처를 추적하기 쉽습니다.

```python
import gradio as gr

def greet(name):
    return f"Hello, {name.strip() or 'visitor'}"

demo = gr.Interface(fn=greet, inputs="text", outputs="text")
demo.launch()
```

실제 AI 데모에서는 입력 크기 제한, 예외 처리, 속도 제한, 민감 정보 경고와 모델 출력 고지를 추가합니다. 비밀 키는 저장소 파일이 아니라 Space의 비밀 설정으로 주입합니다.

## 생태계 통합

Model 저장소는 가중치와 카드를, Dataset 저장소는 데이터와 카드를, Space는 실행 경험을 제공합니다. 서로 링크하면 “어떤 데이터와 모델이 어떤 코드로 결과를 냈는가”를 추적하는 학습·포트폴리오 흐름이 됩니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| 데모 vs 운영 API | 기능 검증·공유 중심 vs SLA·보안·확장성을 갖춘 서비스 |
| 공개 변수 vs secret | 사용자에게 보여도 되는 설정 vs 토큰 같은 비밀 값 |
| 모델 카드 vs Space README | 모델의 능력·한계 vs 앱의 실행·사용 방법 |

## 직접 해보기

1. 텍스트 분류 데모의 입력·출력을 설계하세요.
2. 공개 Space에 넣으면 안 되는 값을 나열하세요.
3. 모델·데이터·Space의 추적 링크를 구성하세요.

<details>
<summary>정답 보기</summary>

1. 길이가 제한된 텍스트를 받고 예측 라벨·점수·주의 문구를 표시합니다.
2. API 토큰, 개인정보, 내부 주소와 비공개 데이터가 포함됩니다.
3. Space README에서 모델과 데이터 카드를 연결하고 각 카드에도 관련 자원을 기록합니다.

</details>

## 연결되는 개념

- 이전: [Model Hub와 Dataset Hub 탐색](05-model-dataset-discovery.md)
- 다음: [모델 카드와 평가 지표 해석](07-model-cards-and-evaluation.md)

## 셀프 체크

- [ ] Space의 요청 흐름을 설명한다.
- [ ] 데모와 운영 서비스를 구분한다.
- [ ] 비밀 값을 설정으로 분리한다.
- [ ] 입력 제한과 오류 처리를 고려한다.
- [ ] 모델·데이터 출처를 연결한다.

### 복습 질문 및 답변

**Q1. Space가 모델 파일을 반드시 직접 포함해야 하나요?**

<details>
<summary>답</summary>

아닙니다. 별도 모델 저장소나 허용된 추론 서비스를 불러오는 구조도 가능합니다.

</details>

**Q2. 데모가 잘 동작하면 운영 준비가 끝난 것인가요?**

<details>
<summary>답</summary>

아닙니다. 부하, 보안, 모니터링, 장애 복구와 비용 검증이 추가로 필요합니다.

</details>

**Q3. 자원 간 링크가 왜 중요한가요?**

<details>
<summary>답</summary>

결과의 출처와 버전을 추적해 재현성과 책임 있는 사용을 돕기 때문입니다.

</details>

## 한 줄 정리

> Spaces는 모델과 데이터를 실행 가능한 경험으로 연결하되, 운영 수준의 안전성은 별도로 설계해야 합니다.
