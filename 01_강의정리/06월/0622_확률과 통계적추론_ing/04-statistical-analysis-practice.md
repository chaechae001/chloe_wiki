# 통계분석 실습과 회귀·분류 모델
> 통계분석의 핵심은 코드를 실행하는 것이 아니라, 문제에 맞는 검정을 고르고 결과를 문장으로 해석하는 것이다.

`통계분석` · `독립표본 t-검정` · `대응표본 t-검정` · `ANOVA` · `카이제곱 검정` · `다중회귀분석` · `로지스틱 회귀분석` · `정확도` · `AUC`

## 핵심요약

- 통계분석은 먼저 문제를 가설로 바꾸고, 변수 유형과 비교 구조에 따라 검정 방법을 선택한다.
- 두 독립 집단의 평균 비교에는 독립표본 t-검정을 사용한다.
- 같은 대상의 전후 비교에는 대응표본 t-검정을 사용한다.
- 3개 이상 집단의 평균 비교에는 One-Way ANOVA를 사용하고, 필요하면 사후검정을 한다.
- 두 범주형 변수의 연관성은 카이제곱 독립성 검정으로 확인한다.
- 연속형 종속변수를 여러 변수로 예측할 때는 다중회귀분석을 사용한다.
- 0/1 범주형 종속변수의 발생 확률을 예측할 때는 로지스틱 회귀분석을 사용한다.
- 분석 결과는 검정통계량보다 p-value, 계수 방향, 설명력, 정확도, AUC를 함께 해석해야 한다.

## 분석 문제를 검정으로 바꾸는 방법

### 변수 유형 먼저 보기

**1. 정의**  
분석 방법은 종속변수가 연속형인지 범주형인지, 비교 집단이 몇 개인지, 같은 대상을 반복 측정했는지에 따라 달라진다.

**2. 왜 필요한가?**  
같은 “차이가 있는가?”라는 질문도 데이터 구조에 따라 검정이 달라진다. 두 그룹 평균 차이는 t-검정, 세 그룹 이상 평균 차이는 ANOVA, 범주형 변수 간 관계는 카이제곱 검정이다.

**3. 예시**

```text
월매출(연속형) + 채널 2개(온라인/오프라인) → 독립표본 t-검정
전환율(연속형) + 같은 광고 소재의 전후 → 대응표본 t-검정
토익 점수(연속형) + 직군 4개 → One-Way ANOVA
합격 여부(범주형) + 채용 전형(범주형) → 카이제곱 검정
```

**4. 헷갈리기 쉬운 점**  
숫자로 되어 있어도 범주형일 수 있다. 예를 들어 합격 여부 `0/1`은 숫자지만 연속형 점수가 아니라 범주형 결과다.

**5. 한 줄 정리**  
분석 방법은 “무엇을 비교하는가”와 “변수가 어떤 형태인가”로 결정된다.

## 평균 차이 검정 실습

### 독립표본 t-검정

**1. 정의**  
서로 다른 두 그룹의 평균이 같은지 비교하는 검정이다.

**2. 왜 필요한가?**  
온라인 채널과 오프라인 채널의 월매출처럼 서로 다른 집단의 평균 차이를 확인할 때 사용한다.

**3. 예시**

```python
import pandas as pd
from scipy import stats

online = df[df["channel"] == "온라인"]["monthly_sales"]
offline = df[df["channel"] == "오프라인"]["monthly_sales"]

mean_on = round(float(online.mean()), 3)
mean_off = round(float(offline.mean()), 3)

lev_stat, lev_p = stats.levene(online, offline)
equal_var = lev_p >= 0.05

t_stat, p_val = stats.ttest_ind(online, offline, equal_var=equal_var)

print(mean_on, mean_off)
print(round(float(lev_p), 3), equal_var)
print(round(float(t_stat), 3), round(float(p_val), 3))
```

실습 결과는 온라인 평균 `3213.374`, 오프라인 평균 `2849.041`, Levene 검정 p-value `0.05`, t-통계량 `4.786`, p-value `0.0`으로 정리된다. 유의수준 5%에서 귀무가설을 기각하므로 두 채널의 월매출 평균에는 통계적으로 유의한 차이가 있다고 해석한다.

**4. 헷갈리기 쉬운 점**  
Levene 검정은 평균 차이가 아니라 분산이 같은지 확인하는 절차다. t-검정의 `equal_var` 옵션을 정하기 위해 사용한다.

**5. 한 줄 정리**  
독립표본 t-검정은 서로 다른 두 그룹의 평균 차이를 확인한다.

### 대응표본 t-검정

**1. 정의**  
같은 대상에서 측정한 두 값의 평균 차이를 검정한다.

**2. 왜 필요한가?**  
캠페인 전후 전환율처럼 같은 광고 소재에서 전후 변화가 있었는지 확인할 때 사용한다.

**3. 예시**

```python
from scipy import stats

pre = df["pre_cvr"]
post = df["post_cvr"]
diff = post - pre

t_stat, p_val = stats.ttest_rel(post, pre, alternative="greater")

print(round(float(pre.mean()), 3))
print(round(float(post.mean()), 3))
print(round(float(diff.mean()), 3))
print(round(float(t_stat), 3), round(float(p_val), 3))
```

실습 결과는 캠페인 전 평균 `3.252`, 후 평균 `3.777`, 차이 평균 `0.526`, t-통계량 `13.235`, p-value `0.0`이다. 유의수준 5%에서 귀무가설을 기각하므로 캠페인은 전환율 향상에 유의미한 효과가 있다고 해석한다.

**4. 헷갈리기 쉬운 점**  
같은 대상의 전후 데이터를 독립표본처럼 취급하면 정보 구조를 잃는다. 전후 차이를 직접 보는 것이 핵심이다.

**5. 한 줄 정리**  
대응표본 t-검정은 같은 대상의 변화량이 유의한지 확인한다.

### One-Way ANOVA

**1. 정의**  
3개 이상 집단의 평균이 모두 같은지 검정한다.

**2. 왜 필요한가?**  
직군별 토익 점수처럼 비교 그룹이 3개 이상일 때 t-검정을 반복하면 오류 가능성이 커진다. ANOVA로 한 번에 전체 평균 차이를 검정한다.

**3. 예시**

```python
from scipy import stats
from statsmodels.stats.multicomp import pairwise_tukeyhsd

groups = df.groupby("job_type")["toeic_score"]
group_data = [g.values for _, g in groups]

f_stat, p_val = stats.f_oneway(*group_data)
print(round(float(f_stat), 3), round(float(p_val), 3))

if p_val < 0.05:
    tukey = pairwise_tukeyhsd(
        endog=df["toeic_score"],
        groups=df["job_type"],
        alpha=0.05
    )
    print(tukey.summary())
```

실습 결과는 F-통계량 `10.878`, p-value `0.0`이다. 귀무가설을 기각하므로 적어도 하나의 직군 평균은 다르다. 이후 어떤 직군끼리 다른지 확인하려면 Tukey HSD 같은 사후검정을 진행한다.

**4. 헷갈리기 쉬운 점**  
ANOVA 결과가 유의하다는 것은 모든 집단이 서로 다르다는 뜻이 아니다. 적어도 하나의 평균이 다르다는 뜻이다.

**5. 한 줄 정리**  
ANOVA는 여러 집단 평균이 모두 같은지 먼저 확인하는 검정이다.

## 범주형 변수 검정

### 카이제곱 독립성 검정

**1. 정의**  
두 범주형 변수가 서로 독립인지, 즉 연관성이 없는지 검정한다.

**2. 왜 필요한가?**  
채용 전형 유형과 최종 합격 여부처럼 두 범주형 변수 사이에 관계가 있는지 확인할 때 사용한다.

**3. 예시**

```python
import pandas as pd
from scipy.stats import chi2_contingency

ct = pd.crosstab(df["hire_type"], df["final_pass"])
chi2, p, dof, expected = chi2_contingency(ct)

print(ct)
print(round(float(chi2), 3), dof, round(float(p), 3))
print(pd.DataFrame(expected, index=ct.index, columns=ct.columns).round(2))
```

실습 결과는 chi2-통계량 `46.137`, 자유도 `2`, p-value `0.0`이다. 귀무가설을 기각하므로 채용 전형 유형과 최종 합격 여부는 통계적으로 유의미한 연관성이 있다고 해석한다.

**4. 헷갈리기 쉬운 점**  
카이제곱 검정은 연관성을 보여주지만 인과관계를 증명하지는 않는다.

**5. 한 줄 정리**  
카이제곱 검정은 범주형 변수끼리 독립인지 확인한다.

## 회귀분석과 분류모델

### 다중회귀분석

**1. 정의**  
여러 독립변수로 연속형 종속변수를 예측하는 분석이다.

**2. 왜 필요한가?**  
월매출에 콜 횟수, 방문 횟수, 경력, 교육 점수가 얼마나 영향을 주는지처럼 여러 요인을 동시에 보고 싶을 때 사용한다.

**3. 예시**

```python
import statsmodels.formula.api as smf

formula = "monthly_sales ~ call_count + visit_count + work_years + training_score"
model = smf.ols(formula=formula, data=df).fit()

params = model.params.drop("Intercept", errors="ignore")
pvals = model.pvalues.drop("Intercept", errors="ignore")

print(params.round(4))
print(pvals.round(4))
print(round(float(model.rsquared), 3))
print(round(float(model.rsquared_adj), 3))
```

실습 결과에서 회귀계수는 `call_count 8.6052`, `visit_count 15.7719`, `work_years 29.1144`, `training_score 4.2019`로 나타났고, 네 변수 모두 p-value가 0.05보다 작았다. R²는 `0.94`, 수정 R²는 `0.939`였다.

**4. 헷갈리기 쉬운 점**  
회귀계수가 크다고 무조건 가장 중요한 변수라고 단정할 수는 없다. 변수 단위가 다르면 계수 크기도 달라진다. 단위와 맥락을 함께 봐야 한다.

**5. 한 줄 정리**  
다중회귀분석은 여러 요인이 연속형 결과에 미치는 방향과 크기를 추정한다.

### 로지스틱 회귀분석

**1. 정의**  
종속변수가 0/1 같은 범주형일 때, 사건이 발생할 확률을 예측하는 회귀모형이다.

**2. 왜 필요한가?**  
서류 합격 여부, 구매 여부, 이탈 여부처럼 결과가 0 또는 1인 문제에서는 일반 선형회귀보다 로지스틱 회귀가 적합하다.

**3. 예시**

```python
import statsmodels.formula.api as smf
from sklearn.metrics import accuracy_score

formula = "doc_pass ~ toeic_score + gpa + intern_count + cert_count"
model = smf.logit(formula=formula, data=df).fit(disp=False)

params = model.params.drop("Intercept", errors="ignore")
pvals = model.pvalues.drop("Intercept", errors="ignore")

prob_pred = model.predict(df)
y_pred = (prob_pred >= 0.5).astype(int)
acc = accuracy_score(df["doc_pass"], y_pred)

print(params.round(4))
print(pvals.round(4))
print(round(float(acc), 3))
```

실습 결과에서 회귀계수는 `toeic_score 0.0041`, `gpa 1.4085`, `intern_count 0.6128`, `cert_count 0.3361`로 나타났고 네 변수 모두 유의했다. 예측확률을 0.5 기준으로 분류했을 때 정확도는 `0.848`이었다.

**4. 헷갈리기 쉬운 점**  
로지스틱 회귀의 계수는 바로 확률 증가량이 아니라 log-odds 기준의 변화량이다. 확률로 해석하려면 로지스틱 함수를 거쳐야 한다.

**5. 한 줄 정리**  
로지스틱 회귀분석은 0/1 결과가 발생할 확률을 예측하고 분류한다.

### 오즈, 로짓, 로지스틱 함수

**1. 정의**  
오즈는 사건이 일어날 확률을 일어나지 않을 확률로 나눈 값이다. 로짓은 오즈에 로그를 취한 값이고, 로지스틱 함수는 어떤 실수 입력도 0과 1 사이 확률로 바꿔준다.

**2. 왜 필요한가?**  
선형회귀는 예측값이 음수나 1보다 큰 값이 될 수 있다. 로지스틱 회귀는 로짓과 로지스틱 함수를 이용해 예측값을 확률 범위로 제한한다.

**3. 예시**

```python
import numpy as np

p = 0.8
odds = p / (1 - p)
logit = np.log(odds)
prob = 1 / (1 + np.exp(-logit))

print(odds)
print(logit)
print(prob)
```

**4. 헷갈리기 쉬운 점**  
오즈는 확률이 아니다. 확률 0.8의 오즈는 `0.8 / 0.2 = 4`로, 실패할 가능성보다 성공할 가능성이 4배라는 뜻이다.

**5. 한 줄 정리**  
로지스틱 회귀는 선형식의 결과를 확률로 바꾸기 위해 오즈와 로짓을 사용한다.

## 코드로 보기 — 통계분석 선택 템플릿

```python
def interpret_pvalue(p_value, alpha=0.05):
    if p_value < alpha:
        return "귀무가설 기각: 통계적으로 유의한 근거가 있음"
    return "귀무가설을 기각하지 못함: 통계적으로 유의하다고 보기 어려움"

print(interpret_pvalue(0.003))
print(interpret_pvalue(0.073))
```

**코드목적**  
여러 통계검정 결과를 같은 기준으로 해석한다.

**해석**  
`0.003`은 0.05보다 작으므로 유의하고, `0.073`은 0.05보다 크므로 유의하다고 보기 어렵다. 실제 실습에서도 기업 유형별 토익 점수 ANOVA는 `p=0.003`으로 유의했고, 영업 등급별 월매출 ANOVA는 `p=0.073`으로 유의하지 않았다.

**실무 연결**  
보고서에서는 숫자만 제시하지 않고 “어떤 가설을 기각했는지”, “그래서 어떤 의사결정을 할 수 있는지”까지 연결해야 한다.

## 직접 해보기

1. 고객 등급 4개 그룹의 평균 구매액이 같은지 ANOVA로 검정하는 코드를 작성해보기.
2. 광고 전후 전환율 데이터에서 대응표본 t-검정을 수행하고 결론 문장을 써보기.
3. 구매 여부를 종속변수로 하는 로지스틱 회귀모형에서 예측확률 0.5 기준 분류 결과를 만들어보기.

## 헷갈리기 쉬운 포인트

| 구분 | 핵심 차이 | 실무 질문 |
| --- | --- | --- |
| t-검정 vs ANOVA | 2개 평균 비교 vs 3개 이상 평균 비교 | 비교 집단이 몇 개인가? |
| 독립표본 vs 대응표본 | 다른 대상 vs 같은 대상 | 전후 데이터인가? |
| 카이제곱 vs t-검정 | 범주형 관계 vs 평균 차이 | 종속변수가 범주형인가? |
| 회귀분석 vs 로지스틱 회귀 | 연속형 예측 vs 0/1 확률 예측 | 결과가 숫자인가, 사건 여부인가? |
| 정확도 vs AUC | 맞힌 비율 vs 분류 성능의 전반적 순위화 | 임계값 하나만 볼 것인가? |

## 연결되는 개념

- 이전 글: [통계적 추론과 가설검정](03-inference-hypothesis-testing.md)
- 함께 볼 키워드: Welch t-test, Tukey HSD, 기대도수, 결정계수, 수정 결정계수, Wald 검정, Hosmer-Lemeshow 검정

## 셀프 체크

- [ ] 분석 문제를 보고 적절한 검정 방법을 선택할 수 있다.
- [ ] 독립표본 t-검정에서 등분산 검정을 먼저 확인할 수 있다.
- [ ] ANOVA 결과가 유의할 때 사후검정이 필요한 이유를 설명할 수 있다.
- [ ] 카이제곱 검정 결과를 독립성 관점에서 해석할 수 있다.
- [ ] 다중회귀의 계수, p-value, R²를 함께 해석할 수 있다.
- [ ] 로지스틱 회귀의 예측확률과 분류 기준을 설명할 수 있다.

**복습 질문 및 답변**

Q. 온라인과 오프라인 채널의 평균 매출을 비교할 때 쓰는 검정은?  
A. 두 그룹이 독립이면 독립표본 t-검정이다.

Q. 채용 전형과 합격 여부의 연관성을 확인할 때 쓰는 검정은?  
A. 카이제곱 독립성 검정이다.

Q. 서류 합격 여부처럼 0/1 결과를 예측할 때 쓰는 모델은?  
A. 로지스틱 회귀분석이다.

## 한 줄 정리

> 통계분석은 문제를 변수 구조로 바꾸고, 적절한 검정·모델을 선택한 뒤, p-value와 계수·성능지표를 근거로 해석하는 과정이다.
