# 심화 문법 ② sorted · enumerate · lambda — 정렬·순번·익명 함수

> 리스트를 정렬했더니 결과가 `None`이 나와 당황한 적 있나요? `for`문에서 "몇 번째인지"와 "값"을 동시에 쓰고 싶을 때는요? 이름 없는 짧은 함수가 필요할 때는요?

`sorted` `sort` `enumerate` `lambda` `익명함수` `map` `정렬`

## 핵심요약

- `sort()`는 **메서드**로 원본 리스트 자체를 정렬하고, 결과를 변수에 담으면 `None`이 들어간다.
- `sorted()`는 **함수**로 정렬된 **복사본**을 돌려주고, 원본은 그대로 둔다.
- 내림차순은 둘 다 `reverse=True` 옵션으로 지정한다.
- `enumerate()`는 순서가 있는 자료에서 **인덱스(순번)와 값을 함께** 꺼내 주는 함수로, `for`문과 짝꿍이다.
- `enumerate`의 시작 번호는 `start=`로 바꿀 수 있다(기본 0).
- `lambda`는 이름 없는 **한 줄짜리 익명 함수**로, 간단한 기능을 즉석에서 정의할 때 쓴다.
- `map()`은 시퀀스의 모든 원소에 함수를 한 번에 적용하며, 결과 확인엔 `list()` 형변환이 필요하다.

---

## 개념별 정리

### sort 메서드 vs sorted 함수

**1. 정의**
둘 다 리스트를 정렬하지만, `list.sort()`는 원본을 직접 바꾸는 메서드이고, `sorted(list)`는 정렬된 새 리스트를 돌려주는 함수다.

**2. 왜 필요한가?**
"원본을 바꿔도 되는가?"에 따라 골라 써야 한다. 원본을 보존해야 하면 `sorted`, 메모리를 아끼고 원본을 정렬해도 되면 `sort`.

**3. 예시**

```python
# sort 메서드: 원본을 정렬, 반환값은 None
my_list = [5, 2, 3, 1, 4]
sort_list = my_list.sort()
print(my_list)     # [1, 2, 3, 4, 5]  (원본이 바뀜)
print(sort_list)   # None             (반환값 없음)
```

```python
# sorted 함수: 복사본을 정렬, 원본 유지
my_list = [5, 2, 3, 1, 4]
sort_list = sorted(my_list)
print(my_list)     # [5, 2, 3, 1, 4]  (원본 그대로)
print(sort_list)   # [1, 2, 3, 4, 5]  (정렬된 복사본)
```

내림차순은 `my_list.sort(reverse=True)` 또는 `sorted(my_list, reverse=True)`.

**4. 헷갈리기 쉬운 점**
`sort_list = my_list.sort()`처럼 메서드 결과를 변수에 담는 실수. `sort()`는 정렬만 하고 `None`을 돌려주므로 `sort_list`는 `None`이 된다. ([함수와 메서드](01-functions-and-methods.md)에서 본 `return` 없는 함수와 같은 함정이다.)

**5. 한 줄 정리**
원본을 바꾸려면 `sort()`, 원본을 지키고 새 결과를 받으려면 `sorted()`.

> 비유: `sort()`는 책상 위 책을 직접 정리하는 것, `sorted()`는 정리된 사진을 한 장 따로 찍어 주는 것.

### enumerate

**1. 정의**
순서가 있는 자료(리스트, 문자열, 딕셔너리 등)에서 **인덱스와 값을 짝지어** 전달하는 함수다.

**2. 왜 필요한가?**
`for`문에서 "지금 몇 번째인지"와 "그 값"이 동시에 필요할 때, 따로 카운터 변수를 만들 필요 없이 깔끔하게 처리한다.

**3. 예시**

```python
my_list = ["Spring", "Summer", "Fall", "Winter"]

print(enumerate(my_list))          # <enumerate object ...> (그대로는 안 보임)
print(list(enumerate(my_list)))    # 리스트로 형변환하면 확인 가능
print(list(enumerate(my_list, start=2023)))  # 시작 번호 지정
```

실행 결과:
```
<enumerate object at 0x...>
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
[(2023, 'Spring'), (2024, 'Summer'), (2025, 'Fall'), (2026, 'Winter')]
```

for문과 함께 쓰면 형변환 없이 바로 풀린다:

```python
grades = [85, 92, 78, 90, 88]
for index, grade in enumerate(grades, start=1):
    print(f"학생 {index}의 성적: {grade}점")
```

실행 결과:
```
학생 1의 성적: 85점
학생 2의 성적: 92점
학생 3의 성적: 78점
학생 4의 성적: 90점
학생 5의 성적: 88점
```

**4. 헷갈리기 쉬운 점**
`print(enumerate(my_list))`만 하면 내부 값이 안 보이고 객체 주소만 나온다. 확인하려면 `list()`로 형변환해야 한다. (단, `for`문에서는 형변환 없이 바로 쓸 수 있다.)

**5. 한 줄 정리**
`enumerate`는 "번호표 + 값"을 한 쌍으로 꺼내 주는 도구다.

### lambda (익명 함수)

**1. 정의**
이름 없이 한 줄로 정의하는 간단한 함수다. 형태는 `lambda 매개변수: 표현식`이며, 표현식의 결과가 곧 반환값이다.

**2. 왜 필요한가?**
아주 짧은 기능을 위해 `def`로 정식 함수를 만들기 번거로울 때, 즉석에서 간결하게 표현한다.

**3. 예시**

```python
square = lambda x: x ** 2
print(square(5))     # 25

# 정의와 동시에 바로 호출도 가능
print((lambda x: x ** 2)(5))   # 25
```

일반 함수와 비교:

```python
# 일반 함수
def calculate_total_price(item_price, quantity):
    tax_rate = 0.1
    total = item_price * quantity
    return total * (1 + tax_rate)

# lambda (간단한 경우)
calculate_total_price = lambda item_price, quantity: (item_price * quantity) * 1.1

print(calculate_total_price(100, 52))
```

**4. 헷갈리기 쉬운 점**
`lambda`의 매개변수 자리에는 **변수 이름**이 와야 한다. `lambda 3, 4: 3*4+2`처럼 값을 직접 넣을 수 없다(호출할 때 값을 넘긴다). 또한 복잡한 로직·여러 줄에는 적합하지 않다 — 그럴 땐 `def`를 쓴다.

**5. 한 줄 정리**
`lambda`는 "이름 붙일 가치도 없는 짧은 함수"를 즉석에서 만드는 문법이다.

### lambda 심화 — map과 함께

**1. 정의**
`map(함수, 시퀀스)`는 시퀀스의 모든 원소에 함수를 하나씩 적용한다. 함수 자리에 `lambda`를 자주 넣는다.

**2. 왜 필요한가?**
"리스트의 모든 값에 같은 가공을 적용"하는 작업을 간결하게 표현한다.

**3. 예시**

```python
names = ["elice", "rabbit", "clock", "queen"]
upper_names = map(str.upper, names)
print(upper_names)            # <map object ...> (그대로는 안 보임)
print(list(upper_names))      # ['ELICE', 'RABBIT', 'CLOCK', 'QUEEN']
```

```python
# lambda를 map에 넣기 (위치는 자유, 결과 확인엔 list 필요)
num_list = [10, 11, 12, 13, 14, 15]
result = list(map(lambda x: x ** 2, num_list))
print(result)   # [100, 121, 144, 169, 196, 225]
```

**4. 헷갈리기 쉬운 점**
`map`의 결과도 `enumerate`처럼 그대로 출력하면 객체 주소만 보인다. 값을 보려면 `list()`로 형변환한다. (map과 list는 거의 같이 다닌다.)

**5. 한 줄 정리**
`map`은 "리스트 전체에 같은 함수를 한 방에" 적용하고, 결과는 `list()`로 펼쳐 본다.

---

## 코드로 보기 — sort vs sorted 한눈에

| 구분 | sort 메서드 | sorted 함수 |
| --- | --- | --- |
| 결과 | 원본 리스트를 정렬 | 복사본 리스트를 정렬 |
| 사용법(오름차순) | `list.sort()` | `result = sorted(list)` |
| 내림차순 | `list.sort(reverse=True)` | `result = sorted(list, reverse=True)` |
| 반환값 | `None` | 정렬된 새 리스트 |
| 장점 | 계산 효율이 미세하게 좋음 | 원본 리스트를 보존 |

**코드목적**
같은 정렬이라도 "원본을 바꾸는지"와 "무엇을 돌려주는지"가 다르다는 점을 비교한다.

**해석**
`sort()`의 반환값을 변수에 담으면 `None`이라는 점이 핵심이다. 원본 보존이 필요한 데이터 분석에서는 `sorted()`가 안전하다.

**실무 연결**
분석 중인 데이터프레임/리스트의 원본을 실수로 망가뜨리면 디버깅이 어려워진다. "원본 보존이 필요한가?"를 먼저 묻고 `sort`/`sorted`를 고르는 습관이 사고를 줄인다.

---

## 직접 해보기

1. `[3, 1, 2]`를 `sorted()`로 내림차순 정렬한 결과와, `.sort(reverse=True)`로 정렬한 뒤 원본을 출력한 결과를 비교해 보자.
2. 과일 리스트 `["apple", "banana", "cherry"]`를 `enumerate`로 돌면서 "1. apple" 형태로 출력해 보자(시작 번호 1).
3. `lambda`와 `map`을 이용해 `[1, 2, 3, 4]`의 각 값을 3배로 만든 리스트를 만들어 보자.

---

## 헷갈리기 쉬운 포인트

- **sort vs sorted**: 원본 변경 + 반환 `None` vs 복사본 반환 + 원본 보존.
- **객체가 그대로 안 보임**: `enumerate(...)`, `map(...)`은 `list()`로 형변환해야 값이 보인다(단, `for`문에서는 바로 사용 가능).
- **lambda 매개변수**: 자리에는 값이 아니라 변수명이 와야 한다. 복잡하면 `def`.

---

## 연결되는 개념

- 이전 글: [f-string과 List Comprehension](03-fstring-and-list-comprehension.md) — `map`/`lambda`는 컴프리헨션과 자주 비교된다.
- 이전 글: [함수와 메서드](01-functions-and-methods.md) — `sort`(메서드) vs `sorted`(함수) 구분의 기반.
- 더 찾아볼 키워드: `filter`, `key=` 정렬 옵션, `reduce`, 정렬 안정성(stable sort)

---

## 셀프 체크

- [ ] `sort`와 `sorted`의 차이(원본 변경/반환값)를 설명할 수 있다.
- [ ] `sort()` 결과를 변수에 담으면 왜 `None`인지 안다.
- [ ] `enumerate`로 인덱스와 값을 동시에 꺼낼 수 있다.
- [ ] `lambda`의 기본 형태를 쓰고 호출할 수 있다.
- [ ] `map` 결과를 `list()`로 확인해야 하는 이유를 안다.

**복습 질문 및 답변**

- (기본) `sorted()`와 `sort()` 중 원본을 보존하는 것은?
  → `sorted()`. 정렬된 복사본을 돌려주고 원본은 그대로 둔다.
- (이해 확인) `for i, v in enumerate(["a","b"], start=1)`에서 첫 바퀴의 `i`, `v`는?
  → `i=1`, `v="a"`. 시작 번호를 1로 지정했기 때문이다.
- (응용) 리스트의 모든 문자열을 대문자로 바꾼 새 리스트가 필요하다. 두 가지 방법은?
  → `list(map(str.upper, names))` 또는 `[name.upper() for name in names]`.

---

## 한 줄 정리

> `sort`는 원본을 직접 정렬(반환 `None`)하고 `sorted`는 복사본을 돌려주며, `enumerate`는 번호와 값을 함께, `lambda`/`map`은 짧은 함수와 일괄 적용을 담당한다.
