# 용어집

이번 회차(비지도학습 — 차원 축소와 클러스터링)에 등장한 핵심 용어를 비전공자도 이해할 수 있는 쉬운 말로 정리했습니다. 주제별로 묶었고, 각 용어는 가장 자세히 다룬 글과 연결했습니다.

## 비지도학습 · 군집 기초

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 비지도학습(Unsupervised Learning) | 정답 레이블 없이 입력 데이터($X$)만으로 데이터 속 구조·그룹·이상치를 스스로 찾는 학습. 지도학습이 "정답을 맞히는" 것이라면 비지도학습은 "구조를 발견하는" 것 | [01](01-unsupervised-learning-basics.md) | 지도학습, 클러스터링, 차원 축소 |
| 클러스터링(Clustering) | 비슷한 데이터끼리 자동으로 묶어 여러 그룹(군집)으로 나누는 작업. 정답이 없어도 데이터 사이 거리·밀도만으로 그룹을 만든다 | [01](01-unsupervised-learning-basics.md) | K-Means, GMM, 고객 세분화 |
| Hard Clustering | 한 데이터가 반드시 하나의 군집에만 속하는 방식. "이 펭귄은 1번 그룹"처럼 딱 잘라 배정한다. K-Means가 대표 | [01](01-unsupervised-learning-basics.md) | K-Means, Soft Clustering |
| Soft Clustering | 한 데이터를 여러 군집에 확률로 나눠 소속시키는 방식. "70%는 0번, 30%는 1번" 식. GMM이 대표 | [06](06-gmm-soft-clustering.md) | GMM, responsibility |
| 정답 레이블(y_true) | 실습에서 일부러 숨겨 둔 실제 정답(펭귄 종). 학습에는 쓰지 않고, 맨 마지막에 결과 검증용으로만 꺼낸다 | [01](01-unsupervised-learning-basics.md) | Crosstab, ARI |

## 차원 축소

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 차원의 저주(Curse of Dimensionality) | 특성(열)이 많아질수록 데이터가 공간에 듬성듬성 흩어져 거리·밀도 개념이 흐려지고 계산·시각화가 어려워지는 현상 | [02](02-pca-and-scaling.md) | 차원 축소, PCA |
| 차원 축소(Dimensionality Reduction) | 정보를 최대한 지키면서 특성 개수를 줄여 데이터를 낮은 차원으로 압축하는 것. 시각화·전처리·노이즈 제거에 쓴다 | [02](02-pca-and-scaling.md) | PCA, t-SNE |
| PCA(주성분 분석) | 데이터의 분산이 가장 큰 방향을 새 축으로 잡아, 정보(분산) 손실을 최소화하며 차원을 줄이는 선형 기법 | [02](02-pca-and-scaling.md) | 주성분, 설명분산, StandardScaler |
| 주성분(Principal Component) | PCA가 찾아낸 새로운 축. PC1은 분산이 가장 큰 방향, PC2는 PC1에 수직이면서 그다음으로 분산이 큰 방향 | [02](02-pca-and-scaling.md) | PCA, 설명분산 |
| 설명분산 비율(Explained Variance Ratio) | 각 주성분이 원래 데이터의 전체 분산 중 몇 %를 담고 있는지를 나타내는 값. 클수록 그 축이 정보를 많이 품고 있다 | [02](02-pca-and-scaling.md) | 누적 설명분산, Scree plot |
| 누적 설명분산(Cumulative Explained Variance) | 주성분을 PC1부터 차례로 더했을 때 쌓이는 설명분산 합. 보통 90% 근처에서 유지할 주성분 개수를 정한다 | [02](02-pca-and-scaling.md) | Scree plot, 차원 선택 |
| StandardScaler | 각 특성을 평균 0, 표준편차 1로 바꿔 서로 다른 단위·크기를 같은 척도로 맞추는 전처리. PCA·거리 기반 알고리즘 앞에서 필수 | [02](02-pca-and-scaling.md) | 표준화, PCA, K-Means |
| t-SNE | 고차원에서의 이웃 유사도(가까운 정도)를 저차원에서도 최대한 보존하도록 배치하는 비선형 시각화 기법. 군집 덩어리를 선명하게 보여준다 | [03](03-tsne-visualization.md) | PCA, perplexity, KL divergence |
| perplexity | t-SNE에서 "각 점이 몇 개의 이웃을 고려할지"를 정하는 값. 이 값에 따라 그림 모양이 달라진다(보통 5~50) | [03](03-tsne-visualization.md) | t-SNE |

## 클러스터링 알고리즘

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| K-Means | K개의 중심점을 두고 "가장 가까운 중심에 배정 → 중심을 그룹 평균으로 이동"을 반복해 군집을 만드는 대표적 Hard Clustering 알고리즘 | [04](04-kmeans-and-elbow.md) | centroid, Inertia, Elbow |
| centroid(중심점) | 각 군집의 중심 좌표. K-Means는 매 반복마다 이 중심을 소속 데이터들의 평균으로 갱신한다 | [04](04-kmeans-and-elbow.md) | K-Means |
| Inertia | 각 데이터에서 자기가 속한 군집 중심까지 거리의 제곱을 모두 더한 값. 작을수록 군집이 응집돼 있다. K가 커지면 무조건 줄어든다 | [04](04-kmeans-and-elbow.md) | Elbow Method, K-Means |
| Elbow Method | Inertia를 K별로 그렸을 때 감소폭이 확 꺾이는 "팔꿈치" 지점을 적절한 K 후보로 고르는 방법 | [04](04-kmeans-and-elbow.md) | Inertia, K 선택 |
| GMM(가우시안 혼합 모델) | 데이터가 여러 개의 정규분포(가우시안)가 섞여 생성됐다고 보고, 각 점이 각 분포에 속할 확률을 추정하는 Soft Clustering | [06](06-gmm-soft-clustering.md) | EM 알고리즘, Soft Clustering |
| EM 알고리즘 | GMM을 학습시키는 반복 절차. 소속 확률을 계산하고(E-step) 그 확률로 분포의 평균·공분산·비율을 갱신하는(M-step) 과정을 반복한다 | [06](06-gmm-soft-clustering.md) | GMM, responsibility |
| responsibility(소속 확률) | GMM에서 한 데이터가 각 가우시안에 속할 확률. Soft Clustering의 핵심 출력 | [06](06-gmm-soft-clustering.md) | GMM, Soft Clustering |

## 타당성 · 검증 지표

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| Silhouette Score | 각 데이터가 "자기 군집엔 얼마나 가깝고, 옆 군집과는 얼마나 먼지"를 -1~1로 잰 점수. 1에 가까울수록 잘 나뉜 것. 정답 없이 품질을 잰다 | [05](05-clustering-validity.md) | 응집도, 분리도, Dunn Index |
| Dunn Index | (군집 사이 최소 거리) ÷ (군집 내 최대 지름)으로 계산. 값이 클수록 군집이 서로 멀고 각각은 빽빽하다는 뜻 | [05](05-clustering-validity.md) | Silhouette, 타당성 |
| Crosstab(교차표) | 행에 실제 정답, 열에 예측 군집을 놓고 개수를 세는 표. 어느 군집이 어느 실제 그룹에 대응하는지 한눈에 본다 | [07](07-full-pipeline-and-evaluation.md) | ARI, NMI |
| ARI(Adjusted Rand Index) | 예측 군집과 실제 정답이 얼마나 일치하는지를 우연 일치를 보정해 -1~1로 잰 지표. 1이면 완벽 일치 | [07](07-full-pipeline-and-evaluation.md) | NMI, Crosstab |
| NMI(정규화 상호정보량) | 정보 이론 기반으로 군집과 정답이 공유하는 정보량을 0~1로 잰 일치도 지표. 1에 가까울수록 잘 맞는다 | [07](07-full-pipeline-and-evaluation.md) | ARI, Crosstab |
