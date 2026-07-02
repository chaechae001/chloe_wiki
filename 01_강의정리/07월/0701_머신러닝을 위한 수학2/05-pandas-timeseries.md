# pandas 시계열 처리 50제 — 날짜·시간 데이터 완전 정복

> 데이터 분석에서 날짜만큼 자주 등장하면서 자주 발목을 잡는 것도 없습니다. "문자열을 날짜로 바꾸고, 요일별로 집계하고, 월별 매출을 내고, 전월 대비 증감을 계산"하는 이 모든 작업이 pandas에서는 몇 줄로 끝납니다. 이 글은 시계열 처리의 핵심 패턴을 8개 주제로 나눠 실전 예제와 함께 정리했습니다.

`pandas` `시계열` `Timestamp` `datetime` `resample` `Timedelta` `DateOffset` `타임존` `rolling` `shift`

## 핵심요약

- **날짜 만들기**: `pd.to_datetime`, `pd.Timestamp`, `pd.date_range` 로 문자열·숫자를 날짜로 변환하고 날짜 시퀀스를 생성한다.
- **`.dt` 접근자**: datetime 컬럼에서 연·월·일·요일·분기를 손쉽게 뽑아낸다.
- **Timedelta**: 두 날짜의 차이를 계산하고 일수·시간으로 분해한다.
- **DateOffset**: 영업일·월말·분기말처럼 "비즈니스 규칙"에 맞는 날짜 이동을 한다.
- **인덱싱**: 날짜를 인덱스로 삼으면 `"2003-01"` 같은 문자열로 바로 필터링된다.
- **resample**: 일별 데이터를 주별·월별·분기별로 재집계하는 시계열의 핵심 도구.
- **shift·rolling**: 전월 대비 증감, 이동 평균, 누적 최댓값 같은 시간 기반 파생값을 만든다.
- **타임존**: `tz_localize` 로 시간대를 부여하고 `tz_convert` 로 변환한다.

> 예제 데이터는 주문(orders)·결제(payments)·고객(customers)·주문상세(orderdetails) 테이블로 구성된 **샘플 판매 데이터베이스**를 사용합니다. 실행 결과의 구체적 수치는 이 샘플 데이터 기준입니다.

## Part 1. Timestamp와 날짜 생성

**날짜로 변환하는 3가지 방법과 날짜 시퀀스 만들기.**

문자열이나 datetime 객체를 pandas의 Timestamp로 바꾸는 방법은 여러 가지지만 결과는 같습니다.

```python
import pandas as pd
import datetime

ts1 = pd.Timestamp("2003-01-06")                        # 방법 1
ts2 = pd.to_datetime("2003-01-06")                       # 방법 2 (가장 흔함)
ts3 = pd.Timestamp(datetime.datetime(2003, 1, 6))        # 방법 3
print("세 값이 같은가?", ts1 == ts2 == ts3)
```

```
세 값이 같은가? True
```

여러 컬럼(year, month, day)을 하나의 날짜로 조립하거나, 규칙적인 날짜 시퀀스를 만들 수도 있습니다.

```python
# year/month/day 컬럼을 한 번에 날짜로 조립
df = pd.DataFrame({"year":[2003,2003], "month":[1,2], "day":[6,14],
                   "hour":[9,13], "minute":[0,30]})
print(pd.to_datetime(df))

# freq 옵션으로 다양한 주기의 날짜 생성
print(pd.date_range("2003-01-01", periods=4, freq="ME").tolist())   # 월말
```

```
0   2003-01-06 09:00:00
1   2003-02-14 13:30:00
dtype: datetime64[ns]
[Timestamp('2003-01-31'), Timestamp('2003-02-28'), Timestamp('2003-03-31'), Timestamp('2003-04-30')]
```

핵심 도구: `pd.Timestamp`, `pd.to_datetime`, `pd.date_range`(일반), `pd.bdate_range`(영업일), `pd.period_range`(기간). 잘못된 날짜는 `errors="coerce"` 로 `NaT`(결측 날짜) 처리할 수 있고, Unix epoch 초는 `pd.to_datetime(x, unit="s")` 로 변환합니다.

> 🍞 **비유**: `to_datetime`은 여러 나라 말로 적힌 날짜를 하나의 공용어(Timestamp)로 번역하는 통역사입니다. 번역만 되면 그다음 계산은 전부 통일된 규칙으로 처리됩니다.

## Part 2. `.dt` 접근자 — 날짜 분해

**datetime 컬럼에서 연·월·일·요일·분기를 뽑아내기.**

Series가 datetime 타입이면 `.dt` 를 통해 날짜의 구성 요소에 접근할 수 있습니다.

```python
# 요일 이름 추출과 요일별 집계
orders["weekday"] = orders["orderDate"].dt.day_name()
print(orders["weekday"].value_counts().head(3))
```

```
weekday
Monday      5
Friday      2
Thursday    1
```

자주 쓰는 `.dt` 속성: `.dt.year / .dt.month / .dt.day`(연월일), `.dt.hour / .dt.minute / .dt.second`(시분초), `.dt.day_name()`(요일 이름), `.dt.dayofweek`(요일 번호, 월=0~일=6), `.dt.quarter`(분기), `.dt.days_in_month`(월의 총 일수), `.dt.normalize()`(시간을 00:00:00으로), `.dt.strftime("%Y년 %m월")`(원하는 문자열 포맷), `.dt.to_period("M")`(월 단위 기간으로 변환).

이 접근자 덕분에 "월별 주문 수", "요일별 매출" 같은 집계의 기준 컬럼을 한 줄로 만들 수 있습니다.

## Part 3. 시간 차이와 Timedelta

**두 날짜의 차이를 계산하고 분해하기.**

날짜끼리 빼면 `Timedelta`(기간)가 나오고, `.dt.days` 로 일수만 정수로 뽑을 수 있습니다.

```python
# 납기(requiredDate)와 실제 배송일(shippedDate)의 차이로 지연 판별
orders["overdue_days"] = (orders["shippedDate"] - orders["requiredDate"]).dt.days
late = orders[orders["overdue_days"] > 0]
print(f"납기 초과 주문: {len(late)}건")
```

```
납기 초과 주문: 0건
```

이 샘플 데이터에서는 지연 주문이 없어 0건으로 나옵니다(결과가 비어 있는 것도 유효한 분석 결과입니다). 핵심 도구: `pd.Timedelta(days=3, hours=2)`(기간 객체 생성), `날짜 + Timedelta`(더하기/빼기), `pd.timedelta_range`(일정 간격 기간 시퀀스), `.dt.components`(일·시·분·초로 분해), `np.busday_count`(영업일 기준 경과일).

## Part 4. DateOffset과 비즈니스 날짜

**영업일·월말·분기말 같은 "업무 규칙" 날짜 이동.**

단순히 30일을 더하는 것과 "다음 달 말로 이동"하는 것은 다릅니다. `pd.offsets` 가 이런 비즈니스 규칙을 처리합니다.

```python
ts = pd.Timestamp("2003-01-06")
print("이번 달 말:", ts + pd.offsets.MonthEnd(0))
print("다음 달 말:", ts + pd.offsets.MonthEnd(1))
print("다음 달 초:", ts + pd.offsets.MonthBegin(1))
```

```
이번 달 말: 2003-01-31 00:00:00
다음 달 말: 2003-01-31 00:00:00
다음 달 초: 2003-02-01 00:00:00
```

핵심 도구: `pd.offsets.BusinessDay`(영업일 이동), `MonthEnd / MonthBegin`(월말/월초), `QuarterEnd`(분기말), `CustomBusinessDay(holidays=[...])`(공휴일 제외 영업일), `pd.DateOffset(months=1, days=5)`(복합 상대 이동). 급여일·마감일·정산일 계산에 그대로 쓰입니다.

> 🍞 **비유**: 단순 날짜 덧셈이 "달력을 30칸 넘기기"라면, DateOffset은 "다음 정산일까지 넘기기"처럼 업무 달력의 규칙을 아는 똑똑한 이동입니다.

## Part 5. 인덱싱과 슬라이싱

**날짜를 인덱스로 삼아 문자열만으로 필터링하기.**

`DatetimeIndex` 를 인덱스로 설정하면, 부분 문자열로 특정 기간을 바로 선택할 수 있습니다.

```python
ts_orders = orders.set_index("orderDate").sort_index()

# 연도/월 문자열로 슬라이싱
jan = ts_orders.loc["2003-01"]                    # 2003년 1월 전체
q1  = ts_orders.loc["2003-01":"2003-03"]          # 1~3월 범위
```

핵심 도구: `.loc["2003"]`(연도), `.loc["2003-01"]`(월), `.loc["2003-01":"2003-03"]`(범위), boolean 마스크(`df[df.index > "2003-02-01"]`), `truncate(before=..., after=...)`(앞뒤 잘라내기). SQL의 `WHERE date BETWEEN ...` 를 훨씬 간결하게 대체합니다.

## Part 6. Resample과 집계

**일별 데이터를 주별·월별·분기별로 재집계하기.**

`resample` 은 시계열의 주기를 바꿔 집계하는 가장 강력한 도구입니다. `groupby`의 시간 버전이라고 생각하면 됩니다.

```python
ts_pay = payments.set_index("paymentDate").sort_index()

# 월별 결제 요약: 합계·평균·건수를 한 번에
monthly = (ts_pay["amount"]
           .resample("ME")
           .agg(["sum", "mean", "count"])
           .rename(columns={"sum":"total", "mean":"avg", "count":"txn_count"}))
print(monthly.head(6))
```

```
                total       avg  txn_count
paymentDate
2003-02-28   14191.12  14191.12          1
2003-03-31   20009.53  20009.53          1
2003-04-30       0.00       NaN          0
2003-05-31    6066.78   6066.78          1
2003-06-30   47213.42  23606.71          2
2003-07-31       0.00       NaN          0
```

결제가 없는 달(4월, 7월)도 0으로 채워져 시간 축이 끊기지 않는 것이 `resample`의 장점입니다. 핵심 옵션: `resample("W")`(주별), `resample("ME")`(월말), `resample("QE")`(분기말), `.ohlc()`(시가·고가·저가·종가 스타일), `resample("D").interpolate()`(결측 구간 보간), `groupby(pd.Grouper(freq="QE"))`(다른 기준과 함께 집계).

## Part 7. Shift와 Rolling

**전월 대비 증감, 이동 평균, 누적값 만들기.**

`shift`(한 칸 밀기)와 `rolling`(구간 이동 집계)으로 시간 기반 파생 지표를 만듭니다.

```python
monthly_total = ts_pay["amount"].resample("ME").sum().to_frame("amount")

# 전월 대비 증감 (shift)
monthly_total["prev_month"] = monthly_total["amount"].shift(1)
monthly_total["mom_change"] = monthly_total["amount"] - monthly_total["prev_month"]
monthly_total["mom_pct"] = (monthly_total["mom_change"] / monthly_total["prev_month"] * 100).round(1)
print(monthly_total.head(5))
```

```
              amount  prev_month  mom_change  mom_pct
paymentDate
2003-02-28  14191.12         NaN         NaN      NaN
2003-03-31  20009.53    14191.12     5818.41     41.0
2003-04-30      0.00    20009.53   -20009.53   -100.0
2003-05-31   6066.78        0.00     6066.78      inf
2003-06-30  47213.42     6066.78    41146.64    678.2
```

첫 달은 이전 달이 없어 `NaN`, 0에서 나눈 값은 `inf`가 되는 점을 유의하세요(실무에서는 이런 값을 별도 처리합니다). 핵심 도구: `shift(1)`(전 기간 값), `diff()`(연속 간격), `rolling(3).mean()`(3개월 이동 평균), `expanding().max()`(처음부터 현재까지 누적 최댓값). 전월 대비 성장률, 추세선, 신기록 추적이 모두 이 도구들로 만들어집니다.

> 🍞 **비유**: `shift`는 "어제 값을 오늘 옆에 나란히 놓기", `rolling`은 "최근 3일 평균 온도처럼 창을 밀며 계산하기"입니다.

## Part 8. 타임존

**시간대를 부여하고 변환하기.**

타임존 정보가 없는(naive) 날짜에 `tz_localize` 로 시간대를 붙이고, `tz_convert` 로 다른 시간대로 바꿉니다.

```python
# UTC 09:00 기준을 서울/뉴욕 시각으로 변환
ts_utc = pd.to_datetime(orders["orderDate"].astype(str) + " 09:00:00").dt.tz_localize("UTC")
ts_seoul   = ts_utc.dt.tz_convert("Asia/Seoul")
ts_newyork = ts_utc.dt.tz_convert("America/New_York")
print(pd.DataFrame({"UTC": ts_utc.head(2), "Seoul": ts_seoul.head(2), "NewYork": ts_newyork.head(2)}))
```

```
                        UTC                     Seoul                   NewYork
0 2003-01-06 09:00:00+00:00 2003-01-06 18:00:00+09:00 2003-01-06 04:00:00-05:00
1 2003-01-09 09:00:00+00:00 2003-01-09 18:00:00+09:00 2003-01-09 04:00:00-05:00
```

UTC 09:00이 서울에서는 18:00(+9시간), 뉴욕에서는 04:00(-5시간)으로 나타납니다. 글로벌 서비스의 로그 분석에서 필수적인 처리입니다.

## 코드로 보기 — 월별 매출 요약 파이프라인

```python
import pandas as pd

# 1) 결제일을 인덱스로 설정하고 정렬
ts_pay = payments.set_index("paymentDate").sort_index()

# 2) 월별 합계로 재집계
monthly = ts_pay["amount"].resample("ME").sum().to_frame("amount")

# 3) 전월 대비 증감률 파생
monthly["mom_pct"] = monthly["amount"].pct_change().mul(100).round(1)

# 4) 3개월 이동 평균으로 추세 확인
monthly["rolling_3m"] = monthly["amount"].rolling(3).mean().round(1)

print(monthly.head(6))
```

**코드목적**
원시 결제 기록(거래 단위)을 월별 매출로 요약하고, 전월 대비 증감률과 3개월 이동 평균 추세까지 한 번에 만드는 실전 파이프라인입니다.

**해석**
`set_index → resample → pct_change → rolling` 이라는 네 단계가 시계열 분석의 전형적인 흐름입니다. 거래 단위의 흩어진 데이터가 "월별 매출 + 증감률 + 추세"라는 의사결정용 지표로 바뀝니다. `resample`이 시간 축을 균일하게 만들고, `pct_change`가 성장 여부를, `rolling`이 단기 변동을 걷어낸 추세를 보여 줍니다.

**실무 연결**
매출 대시보드, 사용자 활동 리텐션 분석, 재고 수요 예측의 전처리가 대부분 이 패턴을 따릅니다. 특히 `resample`로 시간 축을 규칙적으로 맞추는 단계는, 이후 예측 모델(ARIMA·Prophet 등)에 데이터를 넣기 전 반드시 거치는 표준 절차입니다.

## 직접 해보기

1. 문자열 `"2003/12/25"` 와 `"25-Dec-2003"` 처럼 형식이 다른 날짜들을 하나의 리스트에 담아 `pd.to_datetime`으로 한 번에 변환해 보세요.
2. 어떤 결제 데이터에서 **분기별** 결제 총액을 구하려면 `resample`에 어떤 freq 문자열을 넣어야 할까요?
3. 월별 매출 Series가 있을 때, 전월 대비 증감률을 구하는 두 가지 방법(`shift` 이용, `pct_change` 이용)을 각각 코드로 써 보세요.

<details>
<summary>정답 및 해설</summary>

**1.**
```python
import pandas as pd
dates = ["2003/12/25", "25-Dec-2003"]
print(pd.to_datetime(dates))
# DatetimeIndex(['2003-12-25', '2003-12-25'], dtype='datetime64[ns]', freq=None)
```
`to_datetime`은 흔한 형식을 자동 인식합니다. 형식이 아주 특이하면 `format=` 인자로 명시하는 편이 안전하고 빠릅니다.

**2. `"QE"`**
```python
ts_pay["amount"].resample("QE").sum()
```
분기말 기준으로 집계합니다. `"QS"`를 쓰면 분기 시작 기준이 됩니다.

**3.**
```python
# 방법 A: shift 이용
s.pct_change_manual = (s - s.shift(1)) / s.shift(1) * 100
# 방법 B: pct_change 이용 (더 간결)
s.pct_change().mul(100)
```
두 방법은 같은 값을 줍니다. `pct_change()`가 내부적으로 `shift` 계산을 대신 해 주는 것입니다.
</details>

## 헷갈리기 쉬운 포인트

- **`to_datetime` vs `Timestamp`**: 여러 값을 한 번에 변환하면 `to_datetime`, 단일 시각 객체는 `Timestamp`.
- **`.dt.dayofweek` vs `.dt.day_name()`**: 전자는 번호(월=0), 후자는 이름("Monday").
- **`resample` vs `groupby`**: `resample`은 시간 축 전용이라 빈 구간도 채움, `groupby`는 존재하는 값만 묶음.
- **`tz_localize` vs `tz_convert`**: 전자는 시간대를 처음 "부여", 후자는 이미 있는 시간대를 "변환".
- **`shift`의 방향**: `shift(1)`은 과거 값을 현재로 당김, `shift(-1)`은 미래 값을 당김.

## 연결되는 개념

- 함께 보면 좋은 개념: `groupby`, `pivot_table`, `merge`(시계열 데이터도 결국 테이블 결합이 필요합니다).
- 더 찾아볼 키워드: `PeriodIndex`, `asfreq`, `Prophet`, `ARIMA`, `이동 표준편차(rolling std)`, `계절성(seasonality)`

## 셀프 체크

- [ ] 문자열/숫자를 날짜로 변환하는 여러 방법을 안다.
- [ ] `.dt` 접근자로 연·월·일·요일을 추출할 수 있다.
- [ ] 두 날짜의 차이를 `Timedelta`로 계산할 수 있다.
- [ ] 날짜를 인덱스로 삼아 문자열로 슬라이싱할 수 있다.
- [ ] `resample`로 월별·분기별 집계를 만들 수 있다.
- [ ] `shift`·`rolling`로 증감률과 이동 평균을 계산할 수 있다.
- [ ] `tz_localize`와 `tz_convert`의 차이를 안다.

**복습 질문 및 답변**

*기본* — datetime 컬럼에서 요일 이름을 뽑으려면?
> `.dt.day_name()` 을 사용한다. 번호가 필요하면 `.dt.dayofweek`(월=0~일=6).

*이해확인* — `resample`이 `groupby`보다 시계열에 편리한 이유는?
> `resample`은 시간 축을 균일한 간격으로 재구성해, 데이터가 없는 구간도 빈 값(또는 0)으로 채워 시간 축이 끊기지 않는다. 월별 추이를 볼 때 누락된 달이 자동으로 표시된다.

*응용* — 원시 거래 데이터를 "월별 매출 + 전월 대비 증감률 + 추세"로 바꾸는 파이프라인의 단계를 순서대로 말해 보라.
> ① 날짜를 인덱스로 설정(`set_index`)하고 정렬, ② `resample("ME").sum()`으로 월별 합계, ③ `pct_change()`로 전월 대비 증감률, ④ `rolling(3).mean()`으로 이동 평균 추세. 이 흐름이 대부분의 시계열 대시보드 전처리의 표준이다.

## 한 줄 정리

> pandas 시계열 처리는 "날짜로 변환 → 구성 요소 추출 → 기간 재집계(resample) → 시간 기반 파생(shift·rolling)"의 흐름으로, 흩어진 거래 기록을 의사결정용 지표로 바꾸는 데이터 분석의 필수 전처리 기술이다.
