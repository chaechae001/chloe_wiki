# 이름·함수 설계와 타입 힌트

> 좋은 이름은 코드를 설명하고, 작은 함수는 흐름을 나누며, 타입 힌트는 입력과 결과의 약속을 눈에 보이게 합니다.

`이름 규칙` · `단일 책임` · `반환값` · `타입 힌트` · `Optional`

## 핵심요약

- 변수와 함수의 이름은 짧음보다 의미가 분명하고 구체적인지를 먼저 판단한다.
- 함수는 한 가지 역할에 집중하고, 인자와 반환값을 예측 가능하게 설계한다.
- 반복 로직을 작은 함수로 나누면 재사용과 테스트가 쉬워진다.
- 타입 힌트는 `int`, `str`, `list[T]`, `dict[K, V]`, `Optional[T]`처럼 값의 예상 타입을 표현한다.
- 타입 힌트는 코드를 읽고 도구의 도움을 받는 데 유용하지만, 함수의 올바른 동작을 대신 보장하지는 않는다.

## 1. 좋은 이름 짓기 — 값과 행동을 코드에 드러내기

### 1) 정의

이름 짓기는 변수, 함수, 클래스, 상수의 역할을 식별자에 담는 작업입니다. 좋은 이름은 “이 값이 무엇인지”, “이 함수가 무엇을 하는지”를 주변 설명 없이도 추측하게 합니다.

파이썬 코드에서는 역할에 따라 다음 형태를 구분할 수 있습니다.

| 대상 | 이름의 방향 | 예시 |
|---|---|---|
| 변수 | 값의 의미를 나타내는 구체적인 이름 | `customer_count`, `average_score` |
| 함수 | 수행할 행동이 드러나는 동사형 이름 | `load_records()`, `calculate_total()` |
| 클래스 | 대상을 나타내는 명사형 파스칼 케이스 | `OrderSummary`, `TextProcessor` |
| 상수 | 바뀌지 않을 설정을 대문자와 밑줄로 표현 | `MAX_RETRY_COUNT`, `PASS_SCORE` |
| 불리언 | 참·거짓 질문처럼 읽히는 이름 | `is_active`, `has_items`, `can_retry` |

### 2) 왜 필요한가

이름이 모호하면 코드를 읽을 때마다 값을 추적해야 합니다.

```python
d = 7
r = 3
x = d * r
```

`d`, `r`, `x`가 무엇을 뜻하는지 코드를 작성한 사람만 알 수 있습니다. 의미를 이름에 담으면 관계가 즉시 보입니다.

```python
days = 7
requests_per_day = 3
total_requests = days * requests_per_day
```

코드 길이는 늘었지만 해석에 필요한 추측은 줄었습니다.

### 3) 핵심 흐름 재구성

이름을 정할 때는 다음 질문을 차례로 확인합니다.

1. 이 값이나 대상의 역할이 드러나는가?
2. 프로젝트 안에서 같은 개념을 같은 단어로 표현하는가?
3. 함수라면 행동과 대상이 함께 읽히는가?
4. 참·거짓 값이라면 질문처럼 해석되는가?
5. 너무 넓은 이름으로 실제 의미를 숨기지 않았는가?

`data`, `result`, `temp` 같은 이름이 항상 나쁜 것은 아닙니다. 범위가 매우 짧고 의미가 분명할 때는 사용할 수 있습니다. 그러나 여러 단계에서 서로 다른 데이터를 다룬다면 `raw_records`, `filtered_records`, `summary_result`처럼 상태를 구분하는 편이 안전합니다.

### 4) 쉬운 예시

주방의 모든 통에 `재료`라고만 적혀 있으면 뚜껑을 열어 봐야 내용물을 알 수 있습니다. `설탕`, `소금`, `밀가루`라고 적으면 다음 행동을 바로 결정할 수 있습니다.

코드 이름도 같은 역할을 합니다. 좋은 이름은 실행 결과를 바꾸지 않지만, 값을 확인하기 위해 코드를 왕복하는 횟수를 줄여 줍니다.

### 5) 코드 예시

```python
PASS_SCORE = 70

student_score = 84
is_passing = student_score >= PASS_SCORE

print(f"점수: {student_score}")
print(f"통과 여부: {is_passing}")
```

```text
점수: 84
통과 여부: True
```

`PASS_SCORE`는 기준값, `student_score`는 현재 값, `is_passing`은 비교 결과임을 이름만으로 구분할 수 있습니다.

### 6) 헷갈리는 점

좋은 이름은 무조건 긴 이름이 아닙니다. 필요한 의미를 담되 같은 문맥을 반복하지 않아야 합니다.

예를 들어 `student_student_score_value`는 구체적이지만 중복이 많습니다. `student_score`만으로도 충분히 역할을 설명할 수 있습니다.

### 7) 한 줄 정리

> 좋은 이름은 값의 역할과 함수의 행동을 코드 안에 기록해 불필요한 추측을 줄인다.

## 2. 함수 설계 — 한 함수에 한 가지 책임 주기

### 1) 정의

함수는 입력을 받아 한 가지 목적의 작업을 수행하고 결과를 반환하는 코드 단위입니다. 읽기 쉬운 함수는 이름, 인자, 처리, 반환값이 하나의 책임을 향합니다.

### 2) 왜 필요한가

하나의 함수가 파일 읽기, 데이터 정리, 계산, 출력까지 모두 담당하면 작은 변경도 전체 흐름에 영향을 줍니다. 역할을 나누면 각 단계를 따로 이해하고 확인할 수 있습니다.

```text
입력 준비 → 값 정리 → 계산 → 결과 표현
```

모든 단계를 무조건 가장 작은 함수로 쪼개는 것이 목표는 아닙니다. 변경 이유가 다르거나 독립적으로 검증할 가치가 있는 역할을 경계로 나눕니다.

### 3) 핵심 흐름 재구성

함수를 설계할 때 확인할 기준은 다음과 같습니다.

- 한 함수가 한 가지 역할에 집중하는가?
- 인자 수가 많아 사용법을 기억하기 어렵지는 않은가?
- 같은 입력에 대해 반환 형태를 예측할 수 있는가?
- 반복되는 로직을 재사용할 수 있는가?
- 입력과 출력만으로 작은 단위 테스트를 만들 수 있는가?

다음 흐름은 역할이 분리되어 있습니다.

```python
def normalize_name(name: str) -> str:
    return name.strip().title()


def calculate_average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def format_summary(name: str, average: float) -> str:
    return f"{name}: {average:.1f}점"
```

각 함수는 이름 정리, 평균 계산, 문장 생성 중 한 역할만 담당합니다.

### 4) 쉬운 예시

한 사람이 주문 접수, 조리, 결제, 배달을 동시에 맡으면 한 단계의 변경이 전체 업무를 흔듭니다. 역할을 나누면 각 단계의 책임과 문제 위치가 분명해집니다.

함수도 입력과 결과가 분명한 작은 작업 단위로 나누면, 어느 단계에서 예상과 달라졌는지 찾기 쉽습니다.

### 5) 코드 예시

```python
def remove_negative_scores(scores: list[int]) -> list[int]:
    return [score for score in scores if score >= 0]


def calculate_average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


raw_scores = [80, -1, 95, 75]
valid_scores = remove_negative_scores(raw_scores)
average_score = calculate_average(valid_scores)

print(valid_scores)
print(f"평균: {average_score:.2f}")
```

```text
[80, 95, 75]
평균: 83.33
```

잘못된 값을 제외하는 책임과 평균을 계산하는 책임을 분리했습니다. 따라서 필터 기준이 바뀌어도 평균 함수는 그대로 사용할 수 있습니다.

### 6) 헷갈리는 점

- 함수가 짧다고 자동으로 단일 책임을 만족하는 것은 아닙니다. 서로 다른 작업을 한 줄씩 섞을 수도 있습니다.
- 인자 수를 줄이기 위해 필요한 값을 함수 밖의 전역 변수에서 몰래 가져오면 의존 관계가 더 불분명해질 수 있습니다.
- 중복을 없애겠다고 관련 없는 로직을 하나의 범용 함수에 합치면 오히려 이름과 사용법이 모호해질 수 있습니다.
- 반환값을 `print()` 출력과 혼동하면 다른 코드에서 결과를 재사용하기 어렵습니다.

### 7) 한 줄 정리

> 함수는 변경 이유가 같은 하나의 책임을 담고, 입력과 반환값을 예측할 수 있게 설계한다.

## 3. 타입 힌트 — 입력과 반환의 약속 표시하기

### 1) 정의

타입 힌트는 변수나 함수 인자, 반환값에 기대하는 값의 종류를 표시하는 문법입니다.

```python
def calculate_tax(price: int, tax_rate: float) -> float:
    return price * tax_rate
```

이 함수는 `price`에 정수, `tax_rate`에 실수 값을 기대하고 실수 값을 반환한다는 의도를 보여줍니다.

### 2) 왜 필요한가

함수 이름만으로는 입력값의 형태를 모두 알기 어렵습니다. 타입 힌트가 있으면 다음 정보를 함수 정의에서 확인할 수 있습니다.

- 한 번에 한 값인지 여러 값을 담은 목록인지
- 사전의 키와 값이 어떤 타입인지
- 결과가 항상 존재하는지 `None`일 수도 있는지
- 반환값을 다음 코드에서 어떤 방식으로 사용할지

이 정보는 코드를 읽는 사람과 개발 도구가 함수의 사용법을 이해하는 데 도움을 줍니다.

### 3) 핵심 흐름 재구성

#### 기본 타입

```python
age: int = 29
temperature: float = 23.5
user_name: str = "Chloe"
is_active: bool = True
```

#### 목록과 사전

```python
scores: list[int] = [82, 91, 77]
profile: dict[str, int] = {"age": 29, "level": 3}
```

`list[int]`는 정수 목록, `dict[str, int]`는 문자열 키와 정수 값을 가진 사전을 뜻합니다.

#### 반환 타입

함수 정의 뒤의 `->`는 반환할 값의 타입을 나타냅니다.

```python
def is_valid_score(score: float) -> bool:
    return 0 <= score <= 100
```

#### 값이 없을 수 있는 경우

값을 찾지 못했을 때 `None`을 반환할 수 있다면 `Optional[T]`로 표현할 수 있습니다.

```python
from typing import Optional


def find_highest_score(scores: list[int]) -> Optional[int]:
    if not scores:
        return None

    return max(scores)
```

`Optional[int]`는 결과가 정수이거나 `None`일 수 있다는 뜻입니다. 호출하는 쪽에서는 두 경우를 모두 처리해야 합니다.

### 4) 쉬운 예시

택배 상자에 “유리 제품”이라고 표시하면 내용물을 열기 전에도 다루는 방법을 예상할 수 있습니다. 타입 힌트도 함수 내부를 모두 읽기 전에 입력과 결과를 어떻게 다뤄야 할지 알려주는 표지입니다.

표지가 내용물을 바꾸지는 않듯, 타입 힌트 자체가 잘못된 값을 자동으로 고치거나 함수의 로직을 검증하는 것은 아닙니다.

### 5) 코드 예시

```python
from typing import Optional


def find_first_passing_score(
    scores: list[int],
    pass_score: int,
) -> Optional[int]:
    for score in scores:
        if score >= pass_score:
            return score

    return None


result = find_first_passing_score([52, 68, 81, 90], 70)

if result is None:
    print("통과 점수가 없습니다.")
else:
    print(f"첫 통과 점수: {result}")
```

```text
첫 통과 점수: 81
```

함수의 결과가 `int` 또는 `None`일 수 있으므로, 호출하는 코드는 `None` 여부를 먼저 확인한 뒤 정수처럼 사용합니다.

### 6) 헷갈리는 점

- 타입 힌트는 함수 설명을 돕지만 타입이 맞는다고 계산이 올바른 것은 아닙니다.
- `Optional[int]`는 인자가 선택 사항이라는 뜻으로만 한정되지 않습니다. 해당 위치의 값이 `int` 또는 `None`일 수 있다는 의미입니다.
- 빈 목록을 `max()`에 바로 전달하면 결과를 만들 수 없습니다. 타입 표기와 별개로 비어 있는 입력을 어떻게 처리할지 결정해야 합니다.
- 반환 타입을 적어도 실제 코드가 다른 타입을 반환하도록 작성될 수 있으므로 구현을 함께 검토해야 합니다.

### 7) 한 줄 정리

> 타입 힌트는 값의 예상 형태와 `None` 가능성을 코드에 표시해 함수 사용법을 더 명확하게 전달한다.

## 코드로 보기 — 역할과 타입이 분명한 점수 요약

```python
from typing import Optional


PASS_SCORE = 70


def normalize_name(name: str) -> str:
    return name.strip().title()


def find_best_score(scores: list[int]) -> Optional[int]:
    if not scores:
        return None

    return max(scores)


def is_passing(score: int) -> bool:
    return score >= PASS_SCORE


student_name = normalize_name("  chloe  ")
best_score = find_best_score([64, 78, 91])

print(f"이름: {student_name}")

if best_score is None:
    print("등록된 점수가 없습니다.")
else:
    print(f"최고 점수: {best_score}")
    print(f"통과 여부: {is_passing(best_score)}")
```

### 코드 목적

이름 규칙, 단일 책임 함수, 타입 힌트, `Optional` 결과 처리를 하나의 짧은 흐름에서 확인합니다.

### 코드 흐름

1. 통과 기준을 `PASS_SCORE` 상수로 선언한다.
2. `normalize_name`은 이름 문자열 정리만 담당한다.
3. `find_best_score`는 목록이 비었는지 확인하고 최고 점수 또는 `None`을 반환한다.
4. `is_passing`은 점수가 기준 이상인지 불리언으로 반환한다.
5. 호출부는 `best_score`가 `None`인지 먼저 확인한다.
6. 점수가 있으면 최고 점수와 통과 여부를 출력한다.

### 실행 결과

```text
이름: Chloe
최고 점수: 91
통과 여부: True
```

### 실행 결과 해석

입력 이름의 앞뒤 공백이 제거되고 단어 첫 글자가 대문자로 정리되어 `Chloe`가 됩니다. 점수 목록의 최댓값은 91이며, 상수로 정한 통과 기준 70 이상이므로 `True`가 출력됩니다.

점수 목록을 빈 목록으로 바꾸면 `find_best_score`는 `None`을 반환합니다. 호출부가 `None`을 먼저 확인하므로 숫자 연산을 시도하지 않고 “등록된 점수가 없습니다.”라는 분기로 이동합니다.

### 실무 연결

서비스 코드에서는 여러 함수가 서로의 반환값을 이어받습니다. 이름과 타입이 명확하면 다음 단계가 기대할 수 있는 값이 무엇인지 알기 쉽고, 값이 없을 수 있는 경로도 미리 드러납니다.

또한 한 역할만 수행하는 함수는 입력과 예상 결과를 짝지어 확인하기 쉽습니다. 예를 들어 `is_passing(69)`는 `False`, `is_passing(70)`은 `True`여야 한다는 경계 조건을 작은 단위로 검증할 수 있습니다.

## 직접 해보기

1. 주문 총액을 담는 변수와 재고가 있는지를 담는 불리언 변수에 적절한 이름을 각각 지어 보세요.
2. `find_best_score([])`의 반환값은 무엇이며, 반환 타입을 `Optional[int]`로 적는 이유는 무엇인가요?
3. 데이터 정리, 평균 계산, 결과 출력이 한 함수에 섞여 있다면 어떻게 역할을 나눌 수 있을지 함수 이름으로 표현해 보세요.

<details>
<summary>정답 보기</summary>

1. 예를 들어 주문 총액은 `order_total`, 재고 여부는 `has_stock` 또는 `is_in_stock`처럼 지을 수 있습니다. 이름은 프로젝트에서 사용하는 용어와 일관되어야 합니다.
2. 빈 목록에서는 `None`을 반환합니다. 결과가 항상 정수가 아니라 값이 없음을 뜻하는 `None`일 수도 있다는 사실을 호출부에 알리기 위해 `Optional[int]`로 표시합니다.
3. 예를 들어 `clean_scores()`, `calculate_average()`, `format_summary()`처럼 정리·계산·표현의 책임을 분리할 수 있습니다. 실제 경계는 데이터 흐름과 변경 이유에 맞춰 정합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 변수명 vs 함수명 | 변수명은 값이나 상태를 나타내고, 함수명은 수행하는 행동이 드러나야 한다. |
| 상수 vs 일반 변수 | 상수는 바뀌지 않을 기준을 대문자와 밑줄로 표현하고, 일반 변수는 실행 중 변할 수 있는 값을 담는다. |
| `return` vs `print` | `return`은 호출한 코드에 값을 돌려주고, `print`는 값을 화면에 표시한다. |
| 단일 책임 vs 한 줄 함수 | 단일 책임은 역할의 수에 관한 기준이며, 단순히 코드 줄 수가 적다는 뜻은 아니다. |
| 타입 힌트 vs 동작 검증 | 타입 힌트는 예상 타입을 설명하지만 함수의 로직과 실행 결과를 자동으로 보장하지 않는다. |
| `Optional[int]` vs `int` | 전자는 정수 또는 `None`, 후자는 정수 결과를 기대한다는 차이가 있다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [코드 컨벤션과 읽기 쉬운 파이썬](01-code-conventions-and-readable-python.md)
- 다음에 이어지는 개념: [문서화·예외 처리와 프로젝트 구조](03-documentation-errors-and-project-structure.md)
- 스타일을 자동 점검하는 방법: [포매터·린터와 코드 리뷰](04-formatters-linters-and-code-review.md)
- 함께 보면 좋은 키워드: `명명 규칙`, `단일 책임`, `반환값`, `정적 정보`

## 셀프 체크

- [ ] 변수, 함수, 클래스, 상수 이름의 역할 차이를 설명할 수 있다.
- [ ] 불리언 이름을 참·거짓 질문처럼 지을 수 있다.
- [ ] 한 함수에 한 가지 책임을 주어야 하는 이유를 말할 수 있다.
- [ ] `return`과 `print`의 차이를 설명할 수 있다.
- [ ] `list[int]`와 `dict[str, int]`의 의미를 해석할 수 있다.
- [ ] `Optional[int]` 결과를 안전하게 처리할 수 있다.
- [ ] 타입 힌트가 로직의 정확성을 보장하지 않는다는 점을 설명할 수 있다.

### 복습 질문 및 답변

**Q1. `is_active` 같은 이름이 불리언 변수에 적합한 이유는 무엇인가요?**

<details>
<summary>답</summary>

“활성 상태인가?”라는 참·거짓 질문처럼 읽히기 때문입니다. 조건문에서 `if is_active:`로 사용해도 의미가 자연스럽게 이어집니다.

</details>

**Q2. 함수의 인자와 반환값을 예측 가능하게 설계하면 어떤 점이 좋아지나요?**

<details>
<summary>답</summary>

함수를 호출하는 코드가 사용법을 쉽게 이해할 수 있고, 특정 입력에 대한 예상 결과를 작은 단위로 검증하기 쉬워집니다. 다른 코드에서 결과를 재사용하기도 편해집니다.

</details>

**Q3. `Optional[int]`로 표시된 반환값을 바로 숫자 계산에 사용하면 안 되는 이유는 무엇인가요?**

<details>
<summary>답</summary>

결과가 정수가 아니라 `None`일 수 있기 때문입니다. 먼저 `is None`으로 값의 존재 여부를 확인한 뒤, 정수인 분기에서 계산해야 합니다.

</details>

## 한 줄 정리

> 의미가 드러나는 이름, 한 역할에 집중한 함수, 입력과 반환을 설명하는 타입 힌트를 함께 사용하면 코드의 의도와 사용법이 함수 정의만으로도 선명해진다.
