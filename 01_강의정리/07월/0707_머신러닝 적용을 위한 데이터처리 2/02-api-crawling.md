# API로 데이터 수집하기 — JSON·XML과 병렬 처리

> HTML을 통째로 긁는 크롤링과 달리, API는 "순수한 데이터만 주고받기로 약속한 전용 통로"입니다. 훨씬 빠르고 안정적입니다.

`API` · `JSON` · `XML` · `requests` · `멀티스레드`

## 핵심요약

- API 방식은 서버가 디자인을 배제하고 구조화된 데이터(JSON/XML)만 돌려주므로, HTML 파싱보다 빠르고 안정적이다.
- JSON은 파이썬 자료구조와 1:1로 대응한다. 중괄호 `{}`는 딕셔너리, 대괄호 `[]`는 리스트로 파싱된다.
- `requests.get(url).json()`으로 응답을 즉시 파이썬 딕셔너리/리스트로 변환할 수 있다.
- XML은 태그 기반 계층 구조로, 오래된 대형 기관 API에서 여전히 표준으로 쓰인다.
- 데이터가 수만 건이면 한 번에 못 받으므로, 페이지 단위로 나눠 멀티스레드로 병렬 수집하면 속도가 크게 오른다.

## 1. 웹 크롤링과 API 방식의 차이

### 1) 정의

API(Application Programming Interface)는 프로그램끼리 데이터를 주고받기 위해 미리 정해 둔 규칙과 통로입니다. 데이터 수집 관점에서는, 웹 화면(HTML) 대신 서버가 제공하는 전용 데이터 창구에 요청을 보내 구조화된 결과를 받는 방식을 뜻합니다.

### 2) 왜 필요한가

- HTML 크롤링은 디자인·메뉴가 바뀌면 코드가 쉽게 깨지지만, API는 데이터 구조가 유지되어 안정적입니다.
- 불필요한 이미지·레이아웃 없이 순수 데이터만 받으므로 훨씬 빠르고 가볍습니다.
- 공공데이터, 지도, 날씨, 금융 등 상당수 서비스가 공식 API를 제공합니다.

### 3) 핵심 흐름 재구성

두 방식의 성격 차이를 정리하면 이렇습니다.

- **HTML 크롤링**: 사람이 보는 화면을 통째로 받아 복잡한 태그 사이에서 원하는 글자만 찾음. 화면 구조에 종속적이라 잘 깨짐.
- **API 방식**: 프로그램과 프로그램이 "순수 데이터만" 주고받기로 약속한 통로 이용. 화면이 바뀌어도 데이터 구조는 유지되어 안정적.

### 4) 쉬운 예시

식당에서 주방(서버)의 재료를 직접 뒤지는 대신, 잘 정리된 메뉴판을 보고 주문표(요청)를 넣으면 완성된 요리(구조화 데이터)가 나오는 것과 같습니다. 크롤링이 주방을 뒤지는 방식이라면, API는 메뉴판으로 주문하는 방식입니다.

### 5) 코드 예시

공개 API에서 데이터를 받아 표로 바꾸는 최소 패턴입니다. 인증키가 필요한 서비스라면 키는 코드에 직접 넣지 않고 환경 변수 등으로 분리하는 것이 안전합니다.

```python
import requests
import pandas as pd

# 인증키는 코드에 하드코딩하지 말고 별도 변수/환경변수로 관리
SERVICE_KEY = "YOUR_KEY_HERE"
url = f"https://example-open-api.org/{SERVICE_KEY}/json/dataset/1/1000"

req = requests.get(url)
content = req.json()               # JSON 텍스트 → 파이썬 딕셔너리로 변환

# 응답 구조가 {"dataset": {"row": [ {...}, {...} ]}} 형태라고 가정
rows = content["dataset"]["row"]   # 실제 데이터가 담긴 리스트 추출
df = pd.DataFrame(rows)            # 딕셔너리 리스트 → 표

print(df.shape)                    # 예: (1000, 21)
```

`content["dataset"]["row"]`처럼 대괄호로 접근할 수 있는 이유는, JSON이 파이썬 딕셔너리로 파싱되었기 때문입니다. JSON의 구조가 파이썬과 이렇게 대응합니다.

- JSON 객체 `{ }` → 파이썬 딕셔너리 (`Key: Value`)
- JSON 배열 `[ ]` → 파이썬 리스트 (순서 있는 목록)

### 6) 헷갈리는 점

- API가 항상 JSON을 주는 것은 아닙니다. XML을 주는 곳도 많아, 응답 형식을 먼저 확인해야 합니다.
- `.json()`은 응답이 유효한 JSON일 때만 동작합니다. 인증 실패나 오류 응답이 HTML/텍스트로 오면 오류가 납니다.

### 7) 한 줄 정리

> API 수집은 순수 데이터를 구조화된 형태로 받아 파싱 없이 바로 다루는, 크롤링보다 빠르고 안정적인 방식이다.

## 2. XML 데이터 구조

### 1) 정의

XML(Extensible Markup Language)은 HTML처럼 태그(`< >`)로 데이터의 구조와 의미를 표현하는 마크업 언어입니다. 사용자가 태그 이름을 직접 정의해 부모-자식 계층을 명확히 나타낼 수 있습니다.

### 2) 왜 필요한가

공공데이터포털이나 오래된 대형 기관의 API는 여전히 XML을 표준으로 제공하는 경우가 많습니다. JSON만 다룰 줄 알면 이런 데이터를 받지 못합니다.

### 3) JSON vs XML 핵심 차이

- **JSON**: 중괄호·대괄호 기반의 속성-값 포맷. 가볍고 파이썬 자료구조와 직결되어 다루기 쉬움.
- **XML**: 태그 기반 문서 형식. 계층 구조를 태그로 명확히 표현하지만, 상대적으로 무겁고 파싱 코드가 더 필요.

### 4) 코드 예시

XML 응답은 `xml.etree.ElementTree`로 태그를 따라 내려가며 값을 추출합니다.

```python
import requests
import xml.etree.ElementTree as ET

res = requests.get("https://example-open-api.org/xml/dataset")
root = ET.fromstring(res.text)     # XML 문자열 → 트리 구조

records = []
for row in root.iter("row"):       # <row> 태그를 반복 탐색
    records.append({child.tag: child.text for child in row})
```

## 코드로 보기 — 멀티스레드 병렬 수집

전체 데이터가 수만 건이면 한 번의 요청으로 다 받을 수 없습니다. 1,000건 단위로 페이지를 나눠 동시에 요청하면 훨씬 빠릅니다.

```python
import requests
import pandas as pd
from concurrent.futures import ThreadPoolExecutor, as_completed

BASE = "https://example-open-api.org/{key}/json/dataset/{start}/{end}"

# 1단계: 전체 건수 먼저 확인
meta = requests.get(BASE.format(key="KEY", start=1, end=1)).json()
total = meta["dataset"]["list_total_count"]     # 예: 134986건

# 2단계: 1000건 단위 요청 목록 생성
tasks = [(s, min(s + 999, total)) for s in range(1, total + 1, 1000)]

def fetch(start, end):
    url = BASE.format(key="KEY", start=start, end=end)
    return requests.get(url).json()["dataset"]["row"]

# 3단계: 멀티스레드로 동시 요청
all_rows = []
with ThreadPoolExecutor(max_workers=8) as ex:
    futures = [ex.submit(fetch, s, e) for s, e in tasks]
    for f in as_completed(futures):
        all_rows.extend(f.result())

df = pd.DataFrame(all_rows)
print(df.shape)
```

### 코드 목적

수십만 건 규모의 데이터를 페이지 단위로 쪼개 여러 요청을 동시에 보내, 순차 수집 대비 소요 시간을 크게 줄이는 것이 목적입니다.

### 코드 흐름

1. 전체 데이터 건수를 먼저 조회한다.
2. 1,000건 단위로 요청 구간(시작~끝)을 만든다.
3. `ThreadPoolExecutor`로 구간별 요청을 동시에 던진다.
4. 완료되는 대로 결과를 모아 하나의 표로 합친다.

### 실행 결과 해석

전체 건수가 예컨대 134,986건이면 약 135개 요청으로 나뉩니다. 순차로 하면 요청 하나당 대기 시간이 누적되지만, 병렬 처리는 대기 시간을 겹쳐 실행해 전체 소요를 단축합니다. 단, 서버가 허용하는 동시 요청 수를 넘기면 차단될 수 있어 `max_workers`를 적절히 제한합니다.

### 실무 연결

부동산 실거래가, 대중교통, 기상 관측처럼 대량 공공데이터를 정기 수집해 대시보드나 분석 파이프라인에 넣을 때 병렬 수집이 핵심입니다. 수집 주기가 짧을수록 병렬화의 이득이 커집니다.

## 직접 해보기

1. `requests.get(url).json()`이 오류를 냈다. 서버는 200을 반환했는데도 그렇다면, 가능한 원인은?
2. 전체 데이터가 5만 건인데 API가 한 번에 1,000건만 준다. 몇 번의 요청이 필요한가? 병렬 처리를 쓰면 무엇이 좋아지는가?
3. JSON의 `{"user": {"name": "Kim", "tags": ["a", "b"]}}`에서 `"b"`에 접근하는 파이썬 코드를 작성하라.

<details>
<summary>정답 보기</summary>

1. 응답 본문이 유효한 JSON이 아닐 수 있다. 인증키 오류·요청 형식 오류 시 서버가 200과 함께 HTML/텍스트 안내를 주기도 한다. `req.text`를 먼저 출력해 실제 응답 형식을 확인한다.
2. 50번(50,000 ÷ 1,000)의 요청이 필요하다. 병렬 처리를 쓰면 각 요청의 대기 시간이 겹쳐 실행되어 전체 수집 시간이 크게 줄어든다.
3. `data["user"]["tags"][1]` — 딕셔너리 키로 내려간 뒤 리스트 인덱스 1로 접근한다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| HTML 크롤링 vs API 수집 | 화면(HTML)을 파싱 vs 구조화 데이터를 직접 수신. API가 더 빠르고 안정적 |
| JSON vs XML | 중괄호/대괄호 기반 경량 포맷 vs 태그 기반 계층 문서. 둘 다 API 응답 형식 |
| `.json()` vs `.text` | 앞은 JSON을 파싱해 딕셔너리로, 뒤는 원문 문자열 그대로 반환 |
| 순차 수집 vs 병렬 수집 | 요청을 하나씩 vs 여러 개 동시에. 대량 데이터에서 병렬이 훨씬 빠름 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [웹사이트 크롤링 기초](01-web-crawling.md)
- 다음에 이어지는 개념: [불균형 데이터 처리](03-imbalanced-data.md)
- 함께 보면 좋은 키워드: `ThreadPoolExecutor`, `파싱`, `공공데이터`

## 셀프 체크

- [ ] API 수집이 HTML 크롤링보다 안정적인 이유를 설명할 수 있다.
- [ ] JSON과 파이썬 자료구조의 대응 관계를 안다.
- [ ] `.json()`과 `.text`의 차이를 안다.
- [ ] JSON과 XML을 구분할 수 있다.
- [ ] 대량 데이터에서 병렬 수집이 필요한 이유를 안다.

### 복습 질문 및 답변

**Q1. API 방식이 크롤링보다 코드가 잘 안 깨지는 이유는?**

<details>
<summary>답</summary>

크롤링은 웹 화면의 태그·클래스 구조에 의존하므로 디자인이 바뀌면 추출 코드가 깨진다. API는 화면과 무관하게 데이터 구조를 유지하도록 설계되어, 프론트엔드가 바뀌어도 응답 형식이 유지된다.

</details>

**Q2. JSON 객체와 배열은 파이썬에서 각각 무엇이 되는가?**

<details>
<summary>답</summary>

JSON 객체 `{}`는 파이썬 딕셔너리, JSON 배열 `[]`는 파이썬 리스트로 파싱된다. 그래서 딕셔너리 키와 리스트 인덱스를 조합해 원하는 값에 접근할 수 있다.

</details>

**Q3. 멀티스레드로 무작정 동시 요청 수를 늘리면 안 되는 이유는?**

<details>
<summary>답</summary>

서버가 허용하는 동시 요청 한도를 넘기면 과도한 트래픽으로 판단해 차단하거나 오류를 반환할 수 있다. `max_workers`를 적절히 제한하고, 필요하면 요청 간격도 두어야 안정적으로 수집된다.

</details>

## 한 줄 정리

> API 수집은 구조화 데이터를 직접 받아 파싱 부담을 줄이는 안정적 방식이며, 대량 데이터는 페이지 분할 + 멀티스레드 병렬 처리로 빠르게 모은다.
