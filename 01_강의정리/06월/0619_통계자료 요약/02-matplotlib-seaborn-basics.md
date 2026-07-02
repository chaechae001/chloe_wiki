# Matplotlib & Seaborn 기본 문법 — fig/ax로 그래프 제어하기

> `plt.plot()` 한 줄로도 그래프는 그려집니다. 그런데 왜 굳이 `fig, ax`라는 두 글자를 더 쓸까요? 그 답을 알면 그래프를 "내 마음대로" 다룰 수 있게 됩니다.

`Matplotlib` `Seaborn` `figax` `subplot` `객체지향시각화` `선그래프` `막대그래프` `범례`

## 핵심요약

- Matplotlib은 **계층 구조**를 가진 시각화 라이브러리다. 빠르게 그리는 `plt` 방식과, 객체를 직접 다루는 `fig`/`ax` 방식이 있다.
- `fig`는 전체 그림(캔버스), `ax`는 실제 그래프가 그려지는 좌표축 영역이다.
- 그래프가 하나면 `plt`로 충분하지만, **여러 개를 나란히 그리거나 세밀하게 제어**하려면 `fig, ax` 방식이 필요하다.
- 제목·축 라벨·범례·눈금·마커·색·격자는 모두 옵션으로 조절한다.
- **Seaborn은 Matplotlib 기반**이라, 더 적은 코드로 완성도 높은 그래프를 그리되 세부 조정은 Matplotlib 문법을 그대로 쓴다.
- 그래서 "그래프는 Seaborn으로, 옵션 조정은 Matplotlib으로"가 실무 공식이다.

---

## 개념별 정리

### Matplotlib의 계층 구조 — Script Layer와 Artist Layer

**1. 정의**
Matplotlib은 두 층위로 그래프를 그립니다. `plt.plot()`, `plt.bar()`처럼 **빠르게 그리는 방식이 Script Layer**, 선·점·텍스트·축·색을 객체로 다루는 방식이 **Artist Layer**입니다.

**2. 왜 필요한가?**
간단한 그래프는 Script Layer로 충분합니다. 하지만 차트를 세밀하게 수정하거나 여러 그래프를 조합하려면 객체(`fig`, `ax`)를 직접 잡아야 합니다.

**3. 예시**
```python
import matplotlib.pyplot as plt

# 방식 1 — plt 직접 호출 (빠르고 간단)
plt.plot([1, 2, 3], [4, 5, 6])
plt.show()

# 방식 2 — fig, ax 명시적 방식 (제어가 자유로움)
fig, ax = plt.subplots(figsize=(6, 4))
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_title("title")
ax.set_xlabel("x-axis")
plt.show()
```

**4. 헷갈리기 쉬운 점**
`plt.title()`과 `ax.set_title()`은 비슷해 보이지만 대상이 다릅니다. `plt`는 "지금 활성화된 그래프"에 그리고, `ax`는 "내가 지목한 그 좌표축"에 그립니다. 그래프가 여러 개면 이 차이가 결정적입니다.

**5. 한 줄 정리**
빠르게 그리려면 `plt`, 정확히 제어하려면 `fig`/`ax`.

> 비유: `plt`는 자동 모드 카메라, `fig`/`ax`는 수동 모드 카메라. 한 장 막 찍을 땐 자동이 편하지만, 작품을 만들려면 수동이 필요하다.

---

### figure와 axes(subplot)

**1. 정의**
`fig`(figure)는 그래프가 담기는 **종이(캔버스)**, `ax`(axes)는 그 종이 위에 실제로 그래프가 그려지는 **칸**입니다. 한 종이에 여러 칸을 두는 것을 **서브플롯(subplot)**이라 합니다.

**2. 왜 필요한가?**
같은 주제의 그래프 여러 개를 한 화면에 나란히 비교하고 싶을 때, 종이 하나를 칸으로 나눠 각 칸에 다른 그래프를 그립니다.

**3. 예시**
```python
# 한 종이를 1행 2열로 나눠 두 칸(ax1, ax2)을 만든다
fig, (ax1, ax2) = plt.subplots(1, 2)

ax1.plot(year, python)      # 첫 번째 칸에 그리기
ax2.plot(year, java)        # 두 번째 칸에 그리기

ax1.set_title("ax1의 제목")
ax2.set_title("ax2의 제목")
plt.show()
```

**4. 헷갈리기 쉬운 점**
칸 간격이 좁으면 제목·축이 서로 겹칩니다. 이때 `plt.subplots(2, 1, constrained_layout=True)`처럼 `constrained_layout=True`를 주면 칸들의 위치를 자동으로 조정해 겹침을 막습니다(`fig.tight_layout()`도 비슷한 역할).

**5. 한 줄 정리**
`fig`는 종이, `ax`는 칸, 서브플롯은 칸 나누기.

> 비유: `fig`는 도화지 한 장, 각 `ax`는 그 위에 그은 네모 칸. 칸마다 다른 그림을 그릴 수 있다.

---

### 선·막대 그래프와 꾸미기 옵션

**1. 정의**
- **선 그래프**: 시간·순서에 따른 변화·추이를 보여줄 때 적합합니다.
- **막대그래프**: 범주별 값의 크기를 비교할 때 적합합니다.

**2. 왜 필요한가?**
"무엇을 보여줄 것인가"에 따라 그래프 종류가 달라집니다. 추이는 선, 비교는 막대가 기본입니다.

**3. 예시**
```python
plt.plot(year, python, label="Python")   # 선 그래프
plt.title("언어 점유율", fontsize=20)      # 제목 + 글자 크기
plt.xlabel("연도", fontsize=15)            # x축 라벨
plt.ylabel("점유율", fontsize=15)          # y축 라벨
plt.xticks([2018, 2019, 2020, 2021, 2022, 2023])  # x축 눈금
plt.legend()                               # 범례 표시
plt.grid(True, axis="both")                # 격자
plt.show()
```
선의 생김새도 바꿀 수 있습니다.
```python
plt.plot(year, python, marker=".",        # 데이터 포인트 마커
                       linestyle="--",     # 선 모양(점선)
                       color="r")          # 색상
```

**4. 헷갈리기 쉬운 점**
`label="Python"`을 각 선에 미리 줬다면 `plt.legend()`만 호출하면 됩니다. 반대로 `plt.legend(['Python', 'C', 'JAVA'])`처럼 이름 리스트를 직접 넘기면, **그래프를 그린 순서대로** 이름이 매칭되므로 순서가 바뀌면 라벨도 어긋납니다.

**5. 한 줄 정리**
추이는 선, 비교는 막대. 제목·축·범례·색은 모두 옵션으로 붙인다.

---

### Seaborn — Matplotlib 위에 얹은 편의 도구

**1. 정의**
Seaborn은 Matplotlib을 기반으로, **더 적은 코드로 완성도 높은 그래프**를 그리게 해 주는 라이브러리입니다. 관용적으로 `sns`라는 이름으로 불러옵니다.

**2. 왜 필요한가?**
같은 선 그래프도 Seaborn은 데이터프레임과 컬럼 이름만 넘기면 그려집니다. 다만 세부 옵션(눈금 범위 등)은 여전히 Matplotlib 문법으로 조정합니다.

**3. 예시**
```python
import seaborn as sns

sns.set_style("darkgrid")                  # 내장 테마 적용
# data에 데이터프레임, x·y에 컬럼 이름만 전달
sns.lineplot(data=df, x="월", y="어른", errorbar=None)  # 선
sns.barplot(data=df, x="요일", y="청소년", errorbar=None) # 막대
sns.barplot(data=df, x="날씨", y="어린이", hue="공휴일")  # 그룹 나누기
```

**4. 헷갈리기 쉬운 점**
Seaborn의 `lineplot`·`barplot`은 기본적으로 **오차막대(95% 신뢰구간)**를 함께 그립니다. 단순 비교만 원하면 `errorbar=None`을 줘서 끕니다. 또 `hue="컬럼"`을 주면 그 컬럼 기준으로 색을 나눠 그룹을 비교할 수 있습니다.

**5. 한 줄 정리**
그래프는 Seaborn으로 빠르게, 옵션은 Matplotlib으로 정밀하게.

> 비유: Seaborn은 밀키트, Matplotlib은 재료 손질부터 하는 요리. 밀키트로 빨리 만들되 간(옵션)은 직접 본다.

---

## 코드로 보기 — `fig, ax`로만 가능한 일

`plt` 직접 호출은 "지금 활성화된 그래프"라는 전역 상태에 의존합니다. 그래서 여러 그래프를 함수로 재사용하거나 특정 칸을 지목해 그리는 일이 불가능합니다. 반면 `fig, ax`는 객체를 손에 쥐므로 자유롭습니다.

```python
import matplotlib.pyplot as plt

# 2행 3열 = 6칸짜리 캔버스를 한 번에 만든다
fig, axes = plt.subplots(2, 3, figsize=(15, 8))
fig.suptitle("iris dataset — fig, ax pattern demo", fontsize=14)

# 각 칸을 좌표(행, 열)로 지목해서 따로 그린다
axes[0, 0].set_title("sepal length vs petal length")
axes[0, 0].set_xlabel("sepal length (cm)")
axes[0, 0].spines["top"].set_visible(False)    # 위쪽 테두리 삭제(원리: 삭제)
axes[0, 0].spines["right"].set_visible(False)  # 오른쪽 테두리 삭제

# 캔버스 전체 여백을 조정 (fig 객체가 있어야 가능)
fig.tight_layout(rect=[0, 0.02, 1, 0.98])
plt.show()
```

**코드목적**
여러 그래프를 한 캔버스에 배치하고, 칸마다 제목·축·테두리를 개별 제어하기 위한 코드입니다.

**해석**
`axes[0, 0]`처럼 좌표로 칸을 지목하면 그 칸만 정확히 손볼 수 있습니다. `spines[...].set_visible(False)`로 불필요한 테두리를 지우는 것은 1편에서 배운 "삭제" 원리의 실제 구현입니다. `fig.suptitle()`이나 `fig.tight_layout(rect=...)`처럼 캔버스 전체를 다루는 기능은 `fig` 객체 없이는 호출할 수 없습니다.

**실무 연결**
대시보드나 분석 리포트는 그래프 한 장으로 끝나지 않습니다. 여러 지표를 한 화면에 배치하고, 재사용 가능한 플로팅 함수로 만들려면 `fig, ax` 방식이 사실상 필수입니다. Seaborn 그래프를 세밀하게 고칠 때도 결국 이 구조를 이해해야 합니다.

---

## 직접 해보기

1. `plt.subplots(1, 2)`로 칸 두 개를 만들고, 왼쪽엔 선 그래프, 오른쪽엔 막대그래프를 그려 보세요.
2. 같은 선 그래프를 `marker`, `linestyle`, `color`를 바꿔 가며 세 가지 버전으로 그려 보세요.
3. Seaborn `barplot`에서 `errorbar=None`을 줬을 때와 안 줬을 때의 그래프 차이를 직접 비교해 보세요.

## 헷갈리기 쉬운 포인트

- **`plt.title()` vs `ax.set_title()`**: 전자는 활성 그래프, 후자는 지목한 칸. 서브플롯이 여러 개면 후자를 써야 한다.
- **`plt` 방식 vs `fig, ax` 방식**: 그래프 1개면 전자가 간편, 여러 개·재사용·정밀 제어면 후자가 정답.
- **`tight_layout` vs `constrained_layout`**: 둘 다 칸 겹침을 막는다. `constrained_layout=True`는 `subplots` 생성 시 인자로, `tight_layout()`은 그린 뒤 호출.

## 연결되는 개념

- 이전 글: [시각화는 왜 중요한가](01-why-visualization-matters.md) — 삭제·강조 원리를 코드(`spines` 삭제 등)로 옮기는 출발점.
- 다음 글: [그림으로 한 변수 요약하기](03-summarize-one-variable-graphs.md) — 여기서 배운 도구로 실제 분포를 그려 본다.
- 더 찾아볼 키워드: `plt.figure`, `ax.twinx`, `figsize`, `seaborn 테마(whitegrid/ticks)`, `hue`

## 셀프 체크

- [ ] `fig`와 `ax`가 각각 무엇인지 설명할 수 있다.
- [ ] `plt` 방식과 `fig, ax` 방식을 언제 쓰는지 구분할 수 있다.
- [ ] 제목·축 라벨·범례·눈금을 코드로 붙일 수 있다.
- [ ] Seaborn에서 `errorbar=None`과 `hue`의 역할을 안다.
- [ ] 서브플롯이 겹칠 때 해결 방법을 안다.

**복습 질문 및 답변**

- (기본) `fig`와 `ax`의 차이는? → `fig`는 전체 캔버스(종이), `ax`는 실제 그래프가 그려지는 좌표축 영역(칸).
- (이해확인) 그래프 여러 개를 한 화면에 그릴 때 `plt` 방식의 한계는? → 전역 상태(활성 그래프)에 의존하므로 특정 칸을 지목하거나 함수로 재사용하기 어렵다.
- (응용) Seaborn으로 막대그래프를 그리되 오차막대를 없애고 날씨별·공휴일별로 그룹을 나누려면? → `sns.barplot(data=df, x="날씨", y="어린이", errorbar=None, hue="공휴일")`.

## 한 줄 정리

> 그래프는 Seaborn으로 빠르게 그리고, 제목·축·배치 같은 세부는 `fig`/`ax` 객체로 정밀하게 제어한다.
