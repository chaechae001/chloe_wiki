# 딕셔너리와 반복문 활용

> "3번째 학생의 점수"가 아니라 "홍길동의 점수"로 바로 찾고 싶을 때가 있습니다. 번호가 아니라 이름표(Key)로 값을 꺼내는 자료형이 딕셔너리입니다.

`딕셔너리` · `Key` · `Value` · `items` · `zip` · `단어빈도`

## 핵심요약

- 딕셔너리는 **Key(이름표)–Value(값)** 짝으로 자료를 보관하며, 중괄호 `{}`와 콜론 `:`으로 만든다.
- 번호(인덱스)가 아니라 **Key로 값을 찾는다**: `person["name"]`.
- 값 추가·수정은 `딕셔너리[Key] = 값`, 삭제는 `del 딕셔너리[Key]`.
- `keys()`·`values()`·`items()`로 각각 Key·Value·(Key,Value)쌍을 꺼낸다.
- Key는 **변하지 않는 자료형**이어야 한다(튜플 가능, 리스트 불가).
- `items()`를 `for key, value in ...`으로 풀면 Key와 Value를 한 번에 받아 순회할 수 있다.
- `zip`은 여러 리스트의 같은 위치 원소를 짝지어 묶고, `dict(zip(...))`으로 딕셔너리를 만들 수 있다.
- 딕셔너리+반복문+조건문 조합으로 "단어 빈도 세기" 같은 집계를 할 수 있다.

## 개념 정리

### 딕셔너리의 기본 (생성·접근·추가·삭제)

**1. 정의**
사전(dictionary)에서 단어와 뜻이 짝을 이루듯, Key와 Value를 짝지어 저장하는 자료형. Key를 알면 그에 연결된 Value를 바로 꺼낼 수 있다.

**2. 왜 필요한가?**
"이름 → 전화번호", "상품코드 → 가격"처럼 의미 있는 이름표로 값을 찾아야 할 때, 몇 번째인지 외우는 리스트보다 훨씬 직관적이고 빠르다.

**3. 예시**
```python
person = {"name": "Rabbit", "age": 22}

print(person["name"])  # Rabbit   (Key로 값 꺼내기)
print(person["age"])   # 22

person["mail"] = "rabbit@elice.com"   # 추가
print(person)          # {'name': 'Rabbit', 'age': 22, 'mail': 'rabbit@elice.com'}

del person["age"]      # 삭제
print(person)          # {'name': 'Rabbit', 'mail': 'rabbit@elice.com'}
```

**4. 헷갈리기 쉬운 점**
리스트는 `[0]`처럼 번호로 접근하지만, 딕셔너리는 `["name"]`처럼 Key로 접근한다. 또 없는 Key에 값을 대입하면 추가, 있는 Key에 대입하면 덮어쓰기(수정)가 된다. 같은 동작이 상황에 따라 추가도 수정도 된다.

**5. 한 줄 정리**
딕셔너리는 "이름표로 값을 찾고 넣고 빼는" 자료형이다.

> 비유: 사물함이 번호로 찾는 리스트라면, 딕셔너리는 "이름이 붙은 우편함". 번호 몰라도 이름으로 바로 찾는다.

### keys / values / items

**1. 정의**
딕셔너리에서 Key들만(`keys()`), Value들만(`values()`), 또는 (Key, Value) 쌍 전체(`items()`)를 꺼내는 메서드다.

**2. 왜 필요한가?**
전체 항목을 훑으며 처리할 때(예: 모든 과목 점수 평균 내기) Key·Value를 반복문으로 꺼내야 하는데, 그 통로가 이 세 메서드다.

**3. 예시**
```python
person = {"name": "Rabbit", "age": 22, "mail": "rabbit@elice.com"}

print(person.keys())    # dict_keys(['name', 'age', 'mail'])
print(person.values())  # dict_values(['Rabbit', 22, 'rabbit@elice.com'])
print(person.items())   # dict_items([('name', 'Rabbit'), ('age', 22), ('mail', 'rabbit@elice.com')])
```

**4. 헷갈리기 쉬운 점**
출력 결과가 `dict_keys([...])`처럼 리스트처럼 보이지만 리스트가 아닌 "뷰(view)" 객체다. 리스트가 꼭 필요하면 `list(person.keys())`처럼 감싸 준다. 또 `items()`의 각 원소는 (Key, Value) 튜플이다.

**5. 한 줄 정리**
keys·values·items는 딕셔너리 속을 들여다보고 반복할 통로다.

### items()로 반복하기 (권장 방식)

**1. 정의**
`for key, value in 딕셔너리.items():`처럼 쓰면, 한 번의 반복마다 Key와 Value를 두 변수로 동시에 받아 처리할 수 있다.

**2. 왜 필요한가?**
"각 과목과 점수를 함께" 다뤄야 할 때 가장 간결하다. Key 따로, Value 따로 꺼내는 것보다 코드가 짧고 읽기 쉽다.

**3. 예시**
```python
my_dict = {"a": 11, "b": 13, "c": 17}

# 방법 1: zip 사용
for key, value in zip(my_dict.keys(), my_dict.values()):
    print(f"key: {key}, value: {value}")

# 방법 2: items() — 더 간결하고 Pythonic!
for key, value in my_dict.items():
    print(f"key: {key}, value: {value}")
# 두 방법 모두 결과 동일:
# key: a, value: 11
# key: b, value: 13
# key: c, value: 17
```

**4. 헷갈리기 쉬운 점**
`for x in my_dict.items():`처럼 변수 하나로만 받으면, `x`는 `('a', 11)` 같은 튜플 전체가 된다. 그러면 값만 쓰려 해도 `x[1]`처럼 꺼내야 해서 번거롭다. 그래서 보통 `for key, value in ...`처럼 두 변수로 받는다(언패킹).

**5. 한 줄 정리**
`items()`를 두 변수로 받으면 Key·Value를 한 번에 다룰 수 있다.

### zip — 여러 묶음을 짝짓기

**1. 정의**
여러 리스트(또는 튜플 등)의 같은 인덱스 원소끼리 하나의 튜플로 묶어 주는 명령어다.

**2. 왜 필요한가?**
"이름 리스트"와 "점수 리스트"가 따로 있을 때, 같은 순번끼리 짝지어 "이름–점수" 쌍으로 다루거나 곧장 딕셔너리로 만들 수 있다.

**3. 예시**
```python
list1 = ["A", "B", "C", "D"]
list2 = [11, 13, 17, 19]

print(list(zip(list1, list2)))   # [('A', 11), ('B', 13), ('C', 17), ('D', 19)]
print(dict(zip(list1, list2)))   # {'A': 11, 'B': 13, 'C': 17, 'D': 19}
```

**4. 헷갈리기 쉬운 점**
`print(zip(...))`만 하면 `<zip object ...>`처럼 내용이 보이지 않는다. 눈으로 확인하려면 `list()`나 `dict()`로 감싸야 한다. range와 비슷한 성질이다.

**5. 한 줄 정리**
zip은 "같은 자리끼리 짝지어 묶는" 도구다.

> 비유: 두 줄로 선 사람들의 손을 앞에서부터 차례로 맞잡게 하는 것.

## 코드로 보기 — 단어 빈도 세기 (초보 최대 난관 구간)

```python
sentence = "apple banana apple cherry banana apple"
words = sentence.split()        # 단어 리스트로 분리

freq = {}                       # 빈 딕셔너리에서 시작
for word in words:
    if word in freq:
        freq[word] = freq[word] + 1   # 이미 있으면 +1
    else:
        freq[word] = 1                 # 처음 나오면 1로 초기화

print("단어 빈도:", freq)
# 단어 빈도: {'apple': 3, 'banana': 2, 'cherry': 1}
```

**코드목적**
문장에서 각 단어가 몇 번 나오는지 세어 "단어 → 등장 횟수" 딕셔너리로 만든다.

**해석**
빈 딕셔너리에서 출발해 단어를 하나씩 본다. 핵심은 "이 단어가 이미 딕셔너리에 있나?"를 검사하는 부분이다. 있으면 기존 횟수에 1을 더하고, 처음 보는 단어면 1로 새로 등록한다. 그래서 apple은 3, banana는 2, cherry는 1이 된다. 반복문·조건문·딕셔너리가 한꺼번에 맞물리는 구간이라 처음엔 막히는 게 자연스럽다.

**실무 연결**
리뷰에서 가장 많이 나온 단어 찾기, 로그에서 에러 종류별 발생 횟수 집계, 설문 응답 분류 집계 등 "무엇이 몇 번 나왔는가"를 세는 거의 모든 분석이 이 패턴 위에서 동작한다.

## 직접 해보기

1. 이름·나이·전공을 담은 프로필 딕셔너리를 만들고, `"취미"` 키를 추가한 뒤 `"나이"`를 한 살 올려 출력해 보세요.
2. 성적 딕셔너리 `{"수학": 95, "영어": 88, "과학": 92, "국어": 79, "역사": 85}`의 전체 평균을 `items()`로 순회해 구하고, 90점 이상인 과목만 따로 출력해 보세요.
3. `list1 = ["A","B","C","D"]`와 `list2 = [11,13,17,19]`를 `zip`으로 묶어 딕셔너리로 만들어 보세요.

## 헷갈리기 쉬운 포인트

- **리스트 접근 vs 딕셔너리 접근**: 리스트는 `[번호]`, 딕셔너리는 `[Key]`. 딕셔너리에는 순번 개념이 핵심이 아니다.
- **items() 변수 1개 vs 2개**: 1개로 받으면 `(key, value)` 튜플 통째, 2개로 받으면 각각 풀려서 들어온다. 보통 2개 권장.
- **Key로 쓸 수 있는 것 vs 없는 것**: 변하지 않는 자료형만 Key로 가능 → 튜플 OK, 리스트는 `TypeError: unhashable type: 'list'`로 불가.
- **dict_keys/뷰 vs 리스트**: `keys()`·`items()` 결과는 리스트처럼 보여도 뷰 객체다. 필요하면 `list()`로 변환.

## 연결되는 개념

- 먼저 알면 좋은 것: [반복문](01-loops.md)(for로 순회), [리스트와 시퀀스](02-sequence-list.md)(split으로 단어 분리), [튜플](03-tuple.md)(Key로 튜플이 가능한 이유).
- 다음으로 이어지는 것: [함수와 메서드](05-functions.md). 딕셔너리를 반환하거나 집계하는 함수를 만든다.
- 더 찾아볼 키워드: `get()`(없는 Key 안전 접근), `setdefault`, `collections.Counter`, 딕셔너리 컴프리헨션.

## 셀프 체크

- [ ] 딕셔너리를 만들고 Key로 값을 꺼낼 수 있다.
- [ ] 값 추가·수정·삭제 방법을 각각 안다.
- [ ] `keys()`·`values()`·`items()`가 무엇을 돌려주는지 구분한다.
- [ ] `for key, value in d.items():`가 어떻게 동작하는지 설명할 수 있다.
- [ ] zip으로 두 리스트를 딕셔너리로 만들 수 있다.
- [ ] 단어 빈도 세기 로직의 "있으면 +1, 없으면 1" 흐름을 따라갈 수 있다.

**복습 질문 및 답변**

- (기본) `person = {"name": "Rabbit"}`에서 이름을 꺼내려면? → `person["name"]`.
- (이해 확인) `for x in d.items():`에서 x는 무엇인가요? → `(key, value)` 형태의 튜플. 값만 쓰려면 `x[1]`로 꺼내야 한다.
- (응용) 단어 빈도 세기에서 `if word in freq:` 분기가 없다면 어떤 문제가 생길까요? → 처음 보는 단어를 `freq[word] + 1` 하려다 그 Key가 없어 에러가 나거나, 누적이 제대로 되지 않는다. 그래서 "처음이면 1, 있으면 +1"로 나눈다.

## 한 줄 정리

> 딕셔너리는 이름표(Key)로 값을 찾는 자료형이고, `items()` 순회와 `zip`·조건문을 결합하면 단어 빈도 같은 집계를 자연스럽게 해낼 수 있다.
