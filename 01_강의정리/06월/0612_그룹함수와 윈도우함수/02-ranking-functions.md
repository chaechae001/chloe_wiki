# 순위 함수 3형제 — RANK · DENSE_RANK · ROW_NUMBER

> 매출 1등을 뽑는 건 쉽습니다. 그런데 **공동 2등이 두 명**일 때 그다음은 3등일까요, 4등일까요? 이 질문 하나로 세 가지 순위 함수가 갈립니다.

`RANK` `DENSE_RANK` `ROW_NUMBER` `순위 함수` `ORDER BY` `PARTITION BY` `동점 처리`

## 핵심요약

- 순위 함수는 `OVER (ORDER BY ...)`로 "무엇을 기준으로 줄 세울지"를 정한다.
- **RANK**: 동점은 같은 순위, 그 수만큼 **다음 순위를 건너뛴다** (1, 2, 3, 3, 5).
- **DENSE_RANK**: 동점은 같은 순위, 다음 순위를 **건너뛰지 않는다** (1, 2, 3, 3, 4).
- **ROW_NUMBER**: 동점이어도 **무조건 고유한 연속 번호**를 매긴다 (1, 2, 3, 4, 5).
- 세 함수의 차이는 오직 **"동점을 어떻게 다루느냐"** 하나로 결정된다.
- `PARTITION BY`를 더하면 "전체 순위"가 "그룹별 순위(예: 국가별 1등)"로 바뀐다.
- 실무에서는 "GROUP BY로 먼저 집계 → 그 결과에 순위 컬럼 추가" 순서가 자연스럽다.

## 개념별 정리

### RANK — 건너뛰는 순위

**1. 정의**
같은 값에는 같은 순위를 주되, 공동 순위가 차지한 자리만큼 **다음 순위를 건너뜁니다**.

**2. 왜 필요한가?**
올림픽 메달처럼 "공동 2등이 둘이면 다음은 4등"인 현실의 순위 매김과 일치합니다. 등수의 절대적 위치(몇 번째 줄에 있는가)를 보존하고 싶을 때 씁니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    RANK() OVER (ORDER BY SALARY DESC) AS RANK
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
"건너뛴다"는 게 버그처럼 보일 수 있지만 의도된 동작입니다. 6000원이 두 명이면 둘 다 3위, 그다음 사람은 5위가 됩니다(4위는 비어 있음).

**5. 한 줄 정리**
동점은 같은 등수, 그만큼 뒷 등수는 점프.

> 비유: 달리기에서 공동 2위가 두 명이면 다음 주자는 4위. 2위 자리를 둘이 나눠 가졌으니 3위는 없다.

### DENSE_RANK — 빽빽한 순위

**1. 정의**
같은 값에는 같은 순위를 주지만, 공동 순위를 **한 건으로 취급**해 다음 순위를 건너뛰지 않습니다.

**2. 왜 필요한가?**
"서로 다른 급여 구간이 몇 단계인가?"처럼 **등급(레벨)의 개수**가 중요할 때 적합합니다. 빈 번호 없이 1, 2, 3…으로 빽빽하게 채웁니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    DENSE_RANK() OVER (ORDER BY SALARY DESC) AS DENSE_RANK
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
RANK와 결과가 같아 보이다가 **동점이 등장한 직후부터** 달라집니다. 동점이 하나도 없으면 RANK·DENSE_RANK·ROW_NUMBER가 전부 같은 값을 냅니다.

**5. 한 줄 정리**
동점은 같은 등수, 뒷 등수는 점프 없이 이어짐.

> 비유: 등급표. "금·은·동" 중 은메달이 둘이어도 그다음은 여전히 '동(3등급)'이다.

### ROW_NUMBER — 무조건 고유 번호

**1. 정의**
값이 같든 다르든 상관없이 **1부터 끊김 없는 고유 번호**를 부여합니다.

**2. 왜 필요한가?**
동점이라도 딱 한 명만 뽑아야 할 때(예: 그룹별 대표 1행, 페이지네이션) 필수입니다. 순서가 강제로 정해지므로 결과가 항상 유일합니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    ROW_NUMBER() OVER (ORDER BY SALARY DESC) AS ROW_NUMBER
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
동점일 때 누가 먼저 번호를 받을지는 `ORDER BY` 기준만으로는 정해지지 않을 수 있습니다(타이브레이커가 없으면 임의). 안정적인 결과가 필요하면 `ORDER BY SALARY DESC, ID` 처럼 보조 정렬 기준을 더하세요.

**5. 한 줄 정리**
값이 같아도 번호는 다르다 — 무조건 1, 2, 3, 4….

> 비유: 대기표. 같은 시간에 도착해도 한 명당 한 장씩, 번호는 절대 겹치지 않는다.

## 코드로 보기 — 세 함수 한 번에 비교

```sql
SELECT
    ID, NAME, SALARY,
    RANK()       OVER (ORDER BY SALARY DESC) AS RANK,
    DENSE_RANK() OVER (ORDER BY SALARY DESC) AS DENSE_RANK,
    ROW_NUMBER() OVER (ORDER BY SALARY DESC) AS ROW_NUMBER
FROM EMPLOYEE;
```

결과:

| ID | NAME | SALARY | RANK | DENSE_RANK | ROW_NUMBER |
| -- | ---- | ------ | ---- | ---------- | ---------- |
| 1004 | ALICE | 10000 | 1 | 1 | 1 |
| 1000 | JAMES | 8000 | 2 | 2 | 2 |
| 1001 | JESSICA | 6000 | 3 | 3 | 3 |
| 1005 | JANNET | 6000 | 3 | 3 | 4 |
| 1002 | STEVE | 4000 | 5 | 4 | 5 |
| 1003 | ROBERT | 1500 | 6 | 5 | 6 |

**코드목적**
급여 내림차순으로 줄 세운 뒤, 세 가지 방식의 순위를 한 화면에서 비교하는 것이 목적입니다.

**해석**
6000원 동점인 JESSICA·JANNET을 보세요. RANK는 둘 다 3위 → 다음은 **5위**(4위 점프). DENSE_RANK는 둘 다 3위 → 다음은 **4위**(점프 없음). ROW_NUMBER는 **3위·4위**로 강제로 나눕니다. 정확히 이 동점 구간에서 세 함수의 성격이 드러납니다.

**실무 연결**
"국가별 매출 1등 고객만 추출", "부서별 연차 최상위 1명", "리스트 페이지네이션" 등에서 매일 쓰입니다. 특히 **그룹별 대표 1행만 뽑기**는 `ROW_NUMBER() ... PARTITION BY 그룹`으로 만든 뒤 바깥에서 `WHERE 번호 = 1`로 거르는 게 정석입니다.

## 그룹별 순위로 확장 — PARTITION BY

`ORDER BY`만 쓰면 **전체 기준** 순위, `PARTITION BY`를 더하면 **그룹별** 순위가 됩니다. 강의에서는 이를 "칸막이를 치고 그 안에서 순위를 다시 매긴다"고 표현했습니다.

음악 판매 DB로 "국가별 고객 총구매액 순위"를 만들어 봅니다.

```sql
SELECT
    c.Country,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    ROUND(SUM(i.Total), 2)           AS TotalPurchase,
    RANK() OVER (
        PARTITION BY c.Country
        ORDER BY SUM(i.Total) DESC
    ) AS CountryRank
FROM   customers c
JOIN   invoices  i ON c.CustomerId = i.CustomerId
GROUP BY c.Country, c.CustomerId, c.FirstName, c.LastName
ORDER BY c.Country, CountryRank;
```

이렇게 하면 각 나라 안에서 1등부터 다시 시작하는 순위가 나옵니다. 여기에 인라인 뷰로 감싸고 `WHERE CountryRank = 1`을 더하면 **나라별 1등 고객만** 뽑을 수 있습니다.

## 직접 해보기

1. 위 비교 쿼리를 `ORDER BY SALARY ASC`(오름차순)로 바꾸면 순위가 어떻게 뒤집히는지 확인해 보세요.
2. 동점이 전혀 없는 컬럼(예: 고유 ID)으로 세 함수를 돌리면 결과가 모두 같아질까요? 직접 확인해 보세요.
3. "국가별 매출 1등 고객만" 뽑는 쿼리를 인라인 뷰 + `WHERE` 조합으로 완성해 보세요.

## 헷갈리기 쉬운 포인트

- **RANK vs DENSE_RANK**: 동점 뒤 번호를 **건너뛰면 RANK**, **이어지면 DENSE_RANK**. "등수 위치 보존(RANK) vs 등급 개수 세기(DENSE_RANK)"로 기억.
- **RANK 계열 vs ROW_NUMBER**: 앞 둘은 동점을 같은 번호로 묶고, ROW_NUMBER는 무조건 쪼갠다. "딱 1행만 필요하면 ROW_NUMBER".
- **ORDER BY(윈도우) vs ORDER BY(맨 끝)**: `OVER(ORDER BY ...)`는 순위 계산 기준이고, 쿼리 맨 끝의 `ORDER BY`는 화면 출력 정렬이다. 둘은 별개다.

## 연결되는 개념

- 이전 글: [윈도우 함수 첫걸음](01-window-function-basics.md) — `OVER`와 `PARTITION BY`의 기본기.
- 다음 글: [집계 윈도우 함수와 누적합](03-aggregate-window-functions.md) — 순위 대신 합계를 누적해 봅니다.
- 함께 보면 좋은 글: [비율과 등급](05-ratio-ntile-functions.md) — 순위를 백분율(PERCENT_RANK)이나 등급(NTILE)으로 바꿔 봅니다.
- 더 찾아볼 키워드: `tie-breaking`, `top-N per group`, `pagination`, `inline view`

## 셀프 체크

- [ ] RANK·DENSE_RANK·ROW_NUMBER의 동점 처리 차이를 (1,2,3,3,?) 예시로 설명할 수 있다.
- [ ] 동점이 없으면 세 함수 결과가 같아진다는 점을 안다.
- [ ] `PARTITION BY`로 전체 순위를 그룹별 순위로 바꿀 수 있다.
- [ ] "그룹별 1등만 뽑기"를 ROW_NUMBER/RANK + 인라인 뷰로 구성할 수 있다.
- [ ] 윈도우의 `ORDER BY`와 출력용 `ORDER BY`를 구분한다.

**복습 질문 및 답변**

- (기본) 1, 2, 3, 3 다음에 5가 오면 어떤 함수인가?
  → RANK.
- (이해확인) ROW_NUMBER가 동점 상황에서 RANK·DENSE_RANK와 결정적으로 다른 점은?
  → 동점이어도 같은 번호를 주지 않고 무조건 고유한 연속 번호를 매긴다.
- (응용) "국가별 매출 1위 고객"을 뽑을 때 RANK와 ROW_NUMBER 중 무엇이 더 안전할까?
  → 1위가 공동일 수 있다면 ROW_NUMBER는 한 명만, RANK는 공동 1위를 모두 남긴다. "정확히 1행"이 필요하면 ROW_NUMBER, "공동 1위 모두"가 필요하면 RANK를 쓴다.

## 한 줄 정리

> 세 순위 함수의 차이는 오직 동점 처리 하나 — 건너뛰면 RANK, 이어지면 DENSE_RANK, 무조건 쪼개면 ROW_NUMBER.
