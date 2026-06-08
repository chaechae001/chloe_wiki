# 정렬·람다·enumerate (그리고 map·filter)

> 이름 순도 아니고, 글자 수 순도 아니고, "연봉이 높은 순서로" 사람들을 줄 세우고 싶습니다. 게다가 같은 팀끼리 묶으면서요. 정렬 기준을 내 마음대로 정하는 방법이 있을까요?

`sort` `sorted` `key` `lambda` `enumerate` `map` `filter` `정렬 기준`

## 핵심요약

- `list.sort()`는 원본 리스트를 직접 바꾸고, `sorted(list)`는 원본을 그대로 두고 정렬된 새 리스트를 돌려준다.
- `reverse=True`를 주면 내림차순으로 정렬한다(기본은 오름차순).
- `key`에 함수를 주면 "무엇을 기준으로 정렬할지"를 직접 정할 수 있다.
- `lambda`는 이름 없이 그 자리에서 쓰고 버리는 일회용 함수이며, `key`와 짝꿍처럼 쓰인다.
- 튜플을 키로 쓰면 1차 기준·2차 기준 다중 정렬이 가능하다.
- `enumerate`는 순번과 값을 함께 꺼내 주고, `map`/`filter`는 람다와 함께 변환·필터를 함수로 처리한다.

## 개념별 정리

### sort vs sorted

**1. 정의**
`sort()`는 리스트 자체를 정렬해 원본을 바꾸는 메서드, `sorted()`는 정렬된 새 리스트를 반환하는 함수입니다.

**2. 왜 필요한가?**
원본을 보존해야 하는 상황(다른 곳에서 원래 순서를 다시 써야 할 때)과, 그냥 그 자리에서 정렬해 버려도 되는 상황이 다릅니다. 둘을 구분해 쓰면 의도치 않게 데이터가 바뀌는 사고를 막습니다.

**3. 예시**

```python
nums = [3, 1, 2]

result = sorted(nums)      # 새 리스트 반환, 원본 유지
print(result)              # [1, 2, 3]
print(nums)                # [3, 1, 2]  (그대로)

nums.sort(reverse=True)    # 원본을 직접 정렬
print(nums)                # [3, 2, 1]
```

**4. 헷갈리기 쉬운 점**
`sort()`는 원본을 바꾸고 **반환값이 없습니다**(`None`). 그래서 `new = nums.sort()`처럼 쓰면 `new`에는 `None`이 담깁니다. 결과를 변수에 받고 싶다면 `sorted()`를 쓰세요.

**5. 한 줄 정리**
원본을 지킬 땐 `sorted()`, 그 자리에서 바꿔도 되면 `sort()`.

> 비유: `sort()`는 내 책상을 직접 정리하는 것, `sorted()`는 책상은 그대로 두고 정리된 사진 한 장을 따로 받는 것입니다.

### key 매개변수 + lambda

**1. 정의**
`key`는 "각 항목에서 어떤 값을 기준으로 비교할지"를 정하는 함수입니다. `lambda`는 그 함수를 짧게 즉석에서 만드는 표현입니다.

**2. 왜 필요한가?**
숫자나 글자는 그냥 정렬되지만, 딕셔너리 목록은 "무엇을 기준으로?"를 알려 줘야 합니다. 연봉 기준인지, 이름 기준인지, 점수 합계 기준인지를 `key`로 지정합니다.

**3. 예시**

```python
members = [
    {"name": "Alice", "team": "A", "salary": 5200},
    {"name": "Bob",   "team": "B", "salary": 4800},
    {"name": "Carol", "team": "A", "salary": 6100},
    {"name": "Dave",  "team": "B", "salary": 5500},
]

# 연봉 기준 오름차순 정렬
by_salary = sorted(members, key=lambda m: m["salary"])
print([m["name"] for m in by_salary])
# ['Bob', 'Alice', 'Dave', 'Carol']
```

`key`에는 람다뿐 아니라 직접 만든 함수(`def`)나 내장 함수도 넣을 수 있습니다.

```python
words = ["banana", "kiwi", "apple"]
print(sorted(words, key=len))   # 길이 기준
# ['kiwi', 'apple', 'banana']
```

**4. 헷갈리기 쉬운 점**
`key=lambda m: m["salary"]`에서 람다는 "각 항목 m을 받아 비교할 값을 돌려주는" 함수입니다. 람다를 **호출**하는 게 아니라 **함수 자체**를 `key`에 넘기는 것이라, 뒤에 `()`를 붙이지 않습니다.

**5. 한 줄 정리**
`key`는 정렬 기준을 정하는 자리, `lambda`는 그 기준을 즉석에서 적는 펜입니다.

### 다중 정렬 (튜플 키)

**1. 정의**
`key`가 튜플을 반환하게 하면, 앞 요소를 1차 기준, 뒤 요소를 2차 기준으로 정렬합니다.

**2. 왜 필요한가?**
"같은 팀끼리 묶되, 팀 안에서는 연봉 높은 순"처럼 기준이 여러 개일 때 한 번에 처리합니다.

**3. 예시**

```python
# 팀 오름차순, 같은 팀 안에서는 연봉 내림차순
result = sorted(members, key=lambda m: (m["team"], -m["salary"]))
for m in result:
    print(m["team"], m["name"], m["salary"])
# A Carol 6100
# A Alice 5200
# B Dave 5500
# B Bob 4800
```

**4. 헷갈리기 쉬운 점**
오름차순과 내림차순을 섞고 싶을 때, 숫자라면 앞에 마이너스를 붙여 `-m["salary"]`로 뒤집습니다. `reverse=True`는 튜플 전체에 적용되므로 일부만 뒤집을 때는 마이너스 트릭을 씁니다.

**5. 한 줄 정리**
튜플 키 `(1차, 2차)`로 "묶고 나서 그 안에서 다시 정렬"을 한 줄에 담습니다.

### enumerate

**1. 정의**
반복할 때 "몇 번째인지(인덱스)"와 "값"을 함께 꺼내 주는 함수입니다.

**2. 왜 필요한가?**
순번을 매겨 출력하거나, 몇 번째 항목인지 알아야 할 때 별도 카운터 변수를 만들 필요가 없어집니다.

**3. 예시**

```python
items = ["사과", "바나나", "포도"]
for idx, item in enumerate(items, start=1):
    print(idx, item)
# 1 사과
# 2 바나나
# 3 포도
```

**4. 헷갈리기 쉬운 점**
`enumerate`는 기본적으로 0부터 셉니다. 1부터 세려면 `start=1`을 주거나 `idx + 1`을 쓰세요. 사람에게 보여 줄 순번은 보통 1부터라 이 점을 자주 놓칩니다.

**5. 한 줄 정리**
`enumerate`는 "번호표를 자동으로 붙여 주는" 반복 도구입니다.

### map과 filter

**1. 정의**
`map`은 모든 항목에 같은 변환을 적용하고, `filter`는 조건을 통과한 항목만 남깁니다. 둘 다 보통 람다와 함께 씁니다.

**2. 왜 필요한가?**
컴프리헨션과 비슷한 일을 하지만, "함수를 통째로 적용한다"는 관점을 명확히 드러냅니다. 다른 사람의 코드나 함수형 스타일에서 자주 만나는 형태입니다.

**3. 예시**

```python
nums = [1, 2, 3, 4, 5]

squares = list(map(lambda x: x ** 2, nums))
print(squares)   # [1, 4, 9, 16, 25]

evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)     # [2, 4]
```

같은 일을 컴프리헨션으로 쓰면 `[x ** 2 for x in nums]`, `[x for x in nums if x % 2 == 0]`입니다.

**4. 헷갈리기 쉬운 점**
`map`과 `filter`는 결과를 곧바로 리스트로 주지 않고, 한 번 훑으면 사라지는 "지연 객체"를 돌려줍니다. 그래서 결과를 보거나 재사용하려면 `list(...)`로 감싸야 합니다.

**5. 한 줄 정리**
`map`은 전부 변환, `filter`는 골라내기 — 람다와 짝지어 쓰는 함수형 도구입니다.

> 비유: `map`은 모든 사진에 같은 필터를 입히는 일괄 보정, `filter`는 그중 마음에 드는 것만 추리는 선별입니다.

## 코드로 보기 — 총점 기준 성적 줄 세우기

```python
students = [
    {"name": "Alice", "korean": 90, "math": 85, "english": 95},
    {"name": "Bob",   "korean": 80, "math": 95, "english": 70},
    {"name": "Carol", "korean": 88, "math": 88, "english": 88},
]

ranked = sorted(
    students,
    key=lambda s: s["korean"] + s["math"] + s["english"],
    reverse=True,
)

for rank, s in enumerate(ranked, start=1):
    total = s["korean"] + s["math"] + s["english"]
    print(f"{rank}등 {s['name']} (총점 {total})")
# 1등 Alice (총점 270)
# 2등 Carol (총점 264)
# 3등 Bob (총점 245)
```

**코드목적**
총점 항목이 따로 없는 데이터에서, 세 과목 점수를 더한 값을 기준으로 등수를 매기는 것이 목적입니다.

**해석**
`key` 안의 람다가 학생마다 세 과목을 더한 총점을 만들어 그 값으로 정렬합니다. `reverse=True`라 높은 점수가 위로 옵니다. 그다음 `enumerate(..., start=1)`로 1등부터 순번을 붙이고, f-string으로 사람이 읽을 문장으로 출력합니다. 정렬·순번 매기기·출력이 자연스럽게 이어집니다.

**실무 연결**
랭킹, 리더보드, "매출 상위 N개" 추출, 우선순위 큐 같은 곳에서 그대로 쓰입니다. 정렬 기준만 람다로 바꾸면 같은 틀로 다양한 보고서를 만들 수 있습니다.

> 자주 만나는 오류: `key`로 더할 항목 이름을 `s["Korean"]`처럼 대소문자나 철자를 틀리면 `KeyError`가 납니다. 에러가 나면 먼저 딕셔너리의 실제 키 이름부터 확인하세요.

## 직접 해보기

1. `nums = [5, 2, 8, 1]`을 내림차순으로 정렬해 보세요(원본은 보존).
2. 위 `members`를 이름(`name`) 알파벳 순으로 정렬해 보세요.
3. `["dog", "elephant", "cat"]`을 글자 수가 긴 순서로 정렬해 보세요.

## 헷갈리기 쉬운 포인트

- **`sort()` vs `sorted()`:** 전자는 원본 변경·반환값 없음, 후자는 원본 보존·새 리스트 반환.
- **`key=함수` vs `key=함수()`:** 함수 "자체"를 넘기므로 괄호를 붙이지 않습니다.
- **`enumerate` 시작값:** 기본 0, 사람이 볼 순번은 `start=1`.
- **`map`/`filter` 결과:** 바로 리스트가 아니라 `list()`로 감싸야 보입니다.

## 연결되는 개념

- 이전 글: [컴프리헨션과 데이터 전처리](02-comprehension-and-preprocessing.md) — `map`/`filter`는 컴프리헨션과 같은 일을 함수로 표현합니다.
- 함께 보면 좋은 글: [f-string으로 문자열 예쁘게 만들기](01-f-string-formatting.md) — 정렬 결과를 보기 좋게 출력할 때 씁니다.
- 더 찾아볼 키워드: `operator.itemgetter`, 안정 정렬(stable sort), `reduce`

## 셀프 체크

- [ ] `sort()`와 `sorted()`의 차이를 설명할 수 있다.
- [ ] `reverse=True`로 내림차순을 만들 수 있다.
- [ ] `key=lambda ...`로 정렬 기준을 직접 정할 수 있다.
- [ ] 튜플 키로 다중 정렬을 할 수 있다.
- [ ] `enumerate`로 순번과 값을 함께 꺼낼 수 있다.
- [ ] `map`/`filter` 결과를 `list()`로 감싸야 하는 이유를 안다.

**복습 질문 및 답변**

- (기본) 원본을 보존하며 정렬하려면? → `sorted()`를 씁니다.
- (이해 확인) `key=lambda m: m["salary"]`는 무엇을 정하나요? → 각 항목에서 정렬에 쓸 값(연봉)을 정합니다.
- (응용) "팀 오름차순, 연봉 내림차순"을 한 번에 정렬하려면? → `key=lambda m: (m["team"], -m["salary"])`처럼 튜플 키를 씁니다.

## 한 줄 정리

> `key`와 `lambda`로 정렬 기준을 자유롭게 정하고, `enumerate`로 순번을 붙이며, `map`/`filter`로 변환·선별을 함수로 처리하는 것이 파이썬의 데이터 다루기 기본기입니다.
