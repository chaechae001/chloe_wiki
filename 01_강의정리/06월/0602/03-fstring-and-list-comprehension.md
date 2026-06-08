# 심화 문법 ① f-string과 List Comprehension — 코드를 짧고 똑똑하게

> "현재 시간은 " + str(hour) + "시" 처럼 더하기로 문자열을 잇는 게 지겹지 않으셨나요? 그리고 `for`문 다섯 줄을 한 줄로 줄일 수 있다면요?

`f-string` `포맷팅` `List Comprehension` `리스트 컴프리헨션` `조건문` `반복문`

## 핵심요약

- **f-string**은 문자열 안에 변수를 깔끔하게 끼워 넣는 방법이다. (Python 3.6 이상)
- 문자열 앞에 `f`를 붙이고, 중괄호 `{}` 안에 변수명을 넣으면 그 값이 자동으로 들어간다.
- **List Comprehension**은 리스트를 만드는 `for`문(과 `if`문)을 대괄호 한 줄로 압축하는 문법이다.
- 동작 순서: `[표현식 for 변수 in 시퀀스]` → ① 하나씩 꺼내고 ② 표현식으로 변환해 ③ 새 리스트에 담는다.
- 뒤에 `if`를 붙이면 **조건에 맞는 것만** 걸러서 담는다.
- `if ~ else`를 함께 쓰려면 **표현식 앞쪽**에 적어야 하며(`표현식 if 조건 else 표현식2 for ...`), 단순 필터링(`if`만)과 어순이 다르다.

---

## 개념별 정리

### f-string 포맷팅

**1. 정의**
문자열 앞에 `f`를 붙이고 중괄호 안에 변수를 넣어, 변수 값을 문자열에 자연스럽게 삽입하는 방식이다.

**2. 왜 필요한가?**
`print("현재 시간은", word_1, hour, "시", ...)`처럼 콤마로 잇거나 `+`로 연결하면 띄어쓰기와 형변환이 번거롭다. f-string은 읽기 쉽고 실수가 줄어든다.

**3. 예시**

```python
hour = 9
minute = 15
word_1 = "오전"

# 기존 방식
print("현재 시간은", word_1, hour, "시", minute, "분 입니다.")

# f-string 방식
print(f"현재 시간은 {word_1} {hour}시 {minute}분 입니다.")
```

**4. 헷갈리기 쉬운 점**
`f`를 빼먹으면 중괄호가 그대로 글자로 출력된다(`{hour}`가 그냥 "{hour}"로 나온다). 또한 중괄호 안에는 변수뿐 아니라 `{price * 0.9}` 같은 식도 넣을 수 있다.

**5. 한 줄 정리**
f-string은 문자열에 변수를 "빈칸 채우듯" 끼워 넣는 깔끔한 출력 방식이다.

> 비유: 빈칸이 뚫린 안내문 양식에 값을 채워 넣는 것과 같다.

### List Comprehension — 기본 (반복문만)

**1. 정의**
리스트를 만들 때 쓰는 `for`문을 대괄호 안에 한 줄로 압축하는 문법이다.

**2. 왜 필요한가?**
"빈 리스트 만들기 → for로 돌면서 → append" 패턴이 너무 자주 나오기 때문이다. 한 줄로 줄이면 의도가 더 잘 보인다.

**3. 예시 — 같은 결과, 다른 길이**

```python
# 일반 for문
num_list = []
for num in range(100):
    num_list.append(num)
print(num_list)

# List Comprehension
num_list = [num for num in range(100)]
print(num_list)
```

실행 결과 (둘 다 동일):
```
[0, 1, 2, ..., 98, 99]
```

동작 순서는 `[<표현식> for <변수> in <시퀀스>]`:
① 시퀀스에서 변수를 하나씩 꺼내고 → ② 표현식으로 변환하고 → ③ 새 리스트에 담는다. 예를 들어 `[num*10 for num in range(10)]`은 `[0, 10, 20, ..., 90]`이 된다.

**4. 헷갈리기 쉬운 점**
"표현식"과 "변수"의 위치가 헷갈린다. 읽을 때는 **뒤(for ...)부터** 보면 쉽다: "range(10)에서 num을 꺼내서 → num*10으로 바꿔 → 담는다."

**5. 한 줄 정리**
List Comprehension은 "꺼내서 → 바꿔서 → 담는다"를 한 줄로 쓴 것이다.

### List Comprehension — 필터링 (if만)

**1. 정의**
`for` 뒤에 `if 조건`을 붙여 조건에 맞는 원소만 골라 담는다.

**2. 왜 필요한가?**
원하는 데이터만 추출할 때 쓴다. 실무 데이터 정제에서 매우 자주 등장한다.

**3. 예시**

```python
region_list = ["수원(경기)", "군산(전북)", "포항(경북)",
               "경주(경북)", "익산(전북)", "강릉(강원)"]

# 일반 for + if
gyeonggi_list = []
for region in region_list:
    if "경기" in region:
        gyeonggi_list.append(region)
print(gyeonggi_list)

# List Comprehension
gyeonggi_list = [region for region in region_list if "경기" in region]
print(gyeonggi_list)
```

실행 결과:
```
['수원(경기)']
```

동작 순서: ① 꺼내고 → ② 조건 확인 → ③ True인 것만 변환 → ④ 담는다.

**4. 헷갈리기 쉬운 점**
여기서 `if`는 **걸러내기(필터)** 역할이다. `else`가 없다.

**5. 한 줄 정리**
`for` 뒤의 `if`는 "조건에 맞는 것만 통과시키는 체"다.

### List Comprehension — 변환 (if ~ else)

**1. 정의**
조건에 따라 값을 다르게 변환하고 싶을 때, `표현식 if 조건 else 표현식2`를 **앞쪽에** 적는다.

**2. 왜 필요한가?**
"조건에 맞으면 A로, 아니면 B로 바꾼다"는 가공이 필요할 때가 많다(예: 가격대별 할인 적용).

**3. 예시**

```python
original_prices = [800, 1200, 950, 1100]

# 일반 for + if/else
discounted_prices = []
for price in original_prices:
    if price >= 1000:
        discounted_prices.append(int(price * 0.9))
    else:
        discounted_prices.append(price)

# List Comprehension
discounted_prices = [
    int(price * 0.9) if price >= 1000 else price
    for price in original_prices
]
print(original_prices)
print(discounted_prices)
```

실행 결과:
```
[800, 1200, 950, 1100]
[800, 1080, 950, 990]
```

**4. 헷갈리기 쉬운 점**
**어순이 다르다.** 단순 필터(`if`만)는 `for` 뒤에 `if`가 오지만, `if ~ else`는 **`for` 앞**(표현식 자리)에 온다. 이걸 섞으면 문법 에러가 난다.

**5. 한 줄 정리**
"거르기"의 `if`는 뒤에, "값 바꾸기"의 `if ~ else`는 앞에 쓴다.

---

## 코드로 보기 — 가격대별 할인 적용

```python
original_prices = [800, 1200, 950, 1100]
discounted_prices = [
    int(price * 0.9) if price >= 1000 else price
    for price in original_prices
]
print(discounted_prices)
```

**코드목적**
1000원 이상인 상품에만 10% 할인을 적용하고, 그 미만은 원래 가격을 유지하는 새 가격 리스트를 한 줄로 만든다.

**해석**
`[800, 1080, 950, 990]`이 나온다. 1200 → 1080(10% 할인), 1100 → 990(할인), 800·950은 1000 미만이라 그대로다. 원본 리스트 `original_prices`는 손대지 않았다.

**실무 연결**
판매 데이터에서 "조건부 가공"은 일상이다. "1만 원 이상 무료배송 표시", "재고 0이면 '품절'로 치환" 같은 처리를 List Comprehension 한 줄로 표현하면 데이터 파이프라인이 간결해진다.

---

## 직접 해보기

1. f-string으로 `이름`과 `점수` 변수를 받아 "OO님의 점수는 OO점입니다." 형태로 출력해 보자.
2. `range(20)`에서 3의 배수만 담은 리스트를 List Comprehension으로 만들어 보자. (`if`만 사용)
3. 점수 리스트 `[55, 80, 42, 91]`에서 60점 이상이면 `"합격"`, 미만이면 `"불합격"`으로 바꾼 리스트를 만들어 보자. (`if ~ else` 사용)

---

## 헷갈리기 쉬운 포인트

- **`if`만 vs `if ~ else`**: 거르기는 `for` **뒤**(`... for x in seq if 조건`), 값 바꾸기는 `for` **앞**(`표현식 if 조건 else 표현식2 for ...`).
- **f 빠뜨림**: `f` 없이 중괄호를 쓰면 변수 대신 글자 `{...}`가 그대로 출력된다.
- **읽는 방향**: 컴프리헨션은 뒤쪽(`for ...`)부터 읽으면 이해가 빠르다.

---

## 연결되는 개념

- 이전 글: [함수와 메서드](01-functions-and-methods.md) — `int()` 같은 변환 함수가 표현식 안에서 자주 쓰인다.
- 다음 글: [sorted · enumerate · lambda](04-sorted-enumerate-lambda.md) — 데이터를 정렬·순회·변환하는 도구들.
- 더 찾아볼 키워드: dict comprehension, set comprehension, 제너레이터 표현식

---

## 셀프 체크

- [ ] f-string으로 변수를 문자열에 끼워 넣을 수 있다.
- [ ] 일반 for문 리스트 생성을 컴프리헨션으로 바꿀 수 있다.
- [ ] `if`만 쓰는 필터링과 `if ~ else` 변환의 어순 차이를 안다.
- [ ] 컴프리헨션을 뒤쪽부터 읽어 해석할 수 있다.
- [ ] 원본을 건드리지 않고 새 리스트를 만든다는 점을 안다.

**복습 질문 및 답변**

- (기본) f-string은 무엇이고 어떻게 쓰는가?
  → 문자열 앞에 `f`를 붙이고 `{}` 안에 변수를 넣어 값을 삽입하는 방식이다.
- (이해 확인) `[x for x in range(10) if x % 2 == 0]`의 결과는?
  → `[0, 2, 4, 6, 8]`. 짝수만 걸러서 담는다.
- (응용) "1000 이상이면 10% 할인, 아니면 그대로"를 컴프리헨션으로 쓰면 `if`는 어디에 위치하나?
  → `표현식 if 조건 else 표현식2`이므로 `for` **앞쪽**(표현식 자리)에 온다.

---

## 한 줄 정리

> f-string은 변수를 문자열에 깔끔하게 끼워 넣고, List Comprehension은 "꺼내서 바꿔 담는" 반복을 한 줄로 압축하되 거르기(`if`)와 값 바꾸기(`if ~ else`)의 어순이 다르다.
