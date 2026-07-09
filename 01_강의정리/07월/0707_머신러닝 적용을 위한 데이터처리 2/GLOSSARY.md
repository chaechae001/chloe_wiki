# 용어집

이번 회차에서 등장한 핵심 용어를 쉬운 말로 정리했습니다. 주제별로 묶어 검색하기 좋게 구성했습니다.

## 데이터 수집

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 웹 크롤링 | 웹 페이지의 HTML을 코드로 가져와 필요한 데이터만 뽑는 작업. 브라우저가 하는 일을 코드가 대신함 | [웹 크롤링](01-web-crawling.md) | requests, BeautifulSoup |
| requests | 파이썬에서 웹 서버에 요청을 보내고 응답을 받는 HTTP 통신 라이브러리 | [웹 크롤링](01-web-crawling.md) | HTTP, 상태 코드 |
| BeautifulSoup | 받은 HTML을 탐색 가능한 구조로 바꿔 태그·클래스로 데이터를 뽑게 해주는 파싱 도구 | [웹 크롤링](01-web-crawling.md) | DOM, find/find_all |
| 상태 코드 | 서버 응답의 결과 신호. 200은 성공, 404는 없음, 403은 접근 거부 | [웹 크롤링](01-web-crawling.md) | HTTP |
| User-Agent | 요청이 실제 브라우저에서 온 것처럼 보이게 하는 헤더 정보. 봇 차단 우회에 사용 | [웹 크롤링](01-web-crawling.md) | 헤더, 차단 |
| 정적/동적 데이터 | 서버가 처음부터 완성해 보내면 정적(requests로 가능), JS로 나중에 채우면 동적(별도 도구 필요) | [웹 크롤링](01-web-crawling.md) | 자바스크립트 |
| API | 프로그램끼리 데이터를 주고받기 위해 정한 규칙과 통로. 순수 데이터만 안정적으로 받음 | [API 수집](02-api-crawling.md) | JSON, XML |
| JSON | 중괄호·대괄호 기반의 경량 데이터 포맷. 파이썬 딕셔너리/리스트와 1:1 대응 | [API 수집](02-api-crawling.md) | 딕셔너리, 파싱 |
| XML | 태그 기반의 계층형 데이터 문서. 오래된 대형 기관 API에서 표준으로 쓰임 | [API 수집](02-api-crawling.md) | 파싱, ElementTree |
| 멀티스레드 수집 | 대량 데이터를 페이지 단위로 나눠 여러 요청을 동시에 보내 빠르게 모으는 방식 | [API 수집](02-api-crawling.md) | ThreadPoolExecutor |

## 불균형 데이터

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 데이터 불균형 | 예측 대상 클래스 비율이 심하게 치우친 상태. 소수 클래스가 드묾 | [불균형 데이터](03-imbalanced-data.md) | IR, 재현율 |
| 불균형 비율(IR) | 다수 클래스 수 ÷ 소수 클래스 수. 10:1 이상이면 심각한 불균형 | [불균형 데이터](03-imbalanced-data.md) | 다수 클래스 편향 |
| 언더샘플링 | 다수 클래스를 줄여 균형을 맞추는 방법. 정보 손실 위험 | [불균형 데이터](03-imbalanced-data.md) | Tomek Links, CNN |
| 오버샘플링 | 소수 클래스를 늘려 균형을 맞추는 방법. 과적합 위험 | [불균형 데이터](03-imbalanced-data.md) | SMOTE |
| SMOTE | 소수 클래스 사이를 선형 보간해 새 가상 데이터를 합성하는 오버샘플링 기법 | [불균형 데이터](03-imbalanced-data.md) | 보간, 과적합 완화 |
| Tomek Links | 서로 다른 클래스인 최근접 이웃 쌍에서 다수 데이터를 제거해 경계를 선명하게 함 | [불균형 데이터](03-imbalanced-data.md) | 경계면, 노이즈 |
| 재현율(Recall) | 실제 양성 중 모델이 잡아낸 비율. 불균형에서 가장 중요한 지표 | [불균형 데이터](03-imbalanced-data.md) | 정밀도, F1 |
| 정밀도(Precision) | 양성이라 예측한 것 중 실제 양성인 비율 | [불균형 데이터](03-imbalanced-data.md) | 재현율, F1 |
| F1-score | 정밀도와 재현율의 조화평균. 불균형에서 종합 성능을 봄 | [불균형 데이터](03-imbalanced-data.md) | Recall, Precision |

## 검증과 누수

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 데이터 누수 | 예측 시점에 알 수 없는 정보가 학습에 스며들어 점수가 부풀려지는 현상 | [데이터 누수](04-data-leakage.md) | Pipeline, 홀드아웃 |
| Pipeline | 전처리와 모델을 하나로 묶어 fit을 학습 데이터에만 강제하는 도구. 누수 차단 | [데이터 누수](04-data-leakage.md) | ColumnTransformer |
| 홀드아웃 | 튜닝·검증에 쓰지 않고 마지막 한 번만 보는 최종 점검용 데이터 | [데이터 누수](04-data-leakage.md) | test set |
| 층화 추출 | 데이터를 나눌 때 원본의 클래스 비율을 각 분할에도 유지하는 방식 | [분류 파이프라인](07-classification-pipeline.md) | stratify |
| StratifiedKFold | 클래스 비율을 유지하며 K번 교차검증하는 방식. 분류에 적합 | [분류 파이프라인](07-classification-pipeline.md) | 교차검증 |
| GridSearchCV | 하이퍼파라미터 조합을 교차검증으로 전수 탐색해 최적을 찾는 도구 | [분류 파이프라인](07-classification-pipeline.md) | 하이퍼파라미터 |
| ColumnTransformer | 수치형·범주형처럼 성격이 다른 열에 서로 다른 전처리를 적용하는 도구 | [분류 파이프라인](07-classification-pipeline.md) | Pipeline |
| ROC-AUC | 임계값 전반에서 두 클래스를 구분하는 능력을 하나의 값으로 요약한 지표 | [분류 파이프라인](07-classification-pipeline.md) | 정확도, 재현율 |

## 트리와 특성

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 의사결정나무 | "이 조건이 참인가?"를 반복해 데이터를 나누는 스무고개형 모델 | [의사결정나무](05-decision-tree-feature-importance.md) | Gini, Gain |
| 불순도 | 한 노드에 여러 클래스가 얼마나 섞였는지. 순수하면 0, 반반이면 최대 | [의사결정나무](05-decision-tree-feature-importance.md) | Gini, Entropy |
| Gini | 불순도 계산법의 하나. 1 − Σ(p²) | [의사결정나무](05-decision-tree-feature-importance.md) | Entropy |
| Entropy | 불순도 계산법의 하나. −Σ p·log₂(p) | [의사결정나무](05-decision-tree-feature-importance.md) | Gini |
| 정보 이득(Gain) | 부모 불순도에서 자식들의 가중 불순도를 뺀 값. 클수록 좋은 분할 | [의사결정나무](05-decision-tree-feature-importance.md) | 분할 조건 |
| max_depth | 나무가 질문을 몇 단계까지 이어갈지 정하는 값. 과소/과적합을 좌우 | [의사결정나무](05-decision-tree-feature-importance.md) | 가지치기 |
| 특성 중요도 | 각 특성이 예측에 기여한 정도. 모델마다 정의가 다름 | [의사결정나무](05-decision-tree-feature-importance.md) | 불순도 감소, 계수 |
| permutation importance | 특성 값을 뒤섞어 성능이 얼마나 떨어지는지로 중요도를 재는 model-agnostic 방법 | [의사결정나무](05-decision-tree-feature-importance.md) | KNN, 신경망 |
| 특성 엔지니어링 | 기존 컬럼으로 모델이 더 잘 배우는 새 특성을 만들거나 변환하는 작업 | [특성 엔지니어링](06-feature-engineering.md) | 스케일링, 로그변환 |
| 스케일링 | 특성의 값 범위·단위를 맞추는 전처리. 선형·거리 모델에 중요 | [특성 엔지니어링](06-feature-engineering.md) | 표준화, 정규화 |
| 로그변환 | 한쪽으로 치우친(왜도 큰) 값을 완만하게 만드는 변환 | [특성 엔지니어링](06-feature-engineering.md) | 왜도 |
| 상호작용항 | 두 특성을 곱해 결합 효과를 명시적으로 표현한 파생변수 | [특성 엔지니어링](06-feature-engineering.md) | 파생변수 |
| 원-핫 인코딩 | 순서 없는 범주를 여러 개의 0/1 열로 바꾸는 인코딩 | [특성 엔지니어링](06-feature-engineering.md) | 범주형, get_dummies |
