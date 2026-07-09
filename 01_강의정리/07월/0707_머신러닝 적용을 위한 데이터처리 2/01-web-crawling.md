# 웹사이트 크롤링 기초 — requests와 BeautifulSoup

> 웹 크롤링은 결국 "브라우저가 하는 일을 코드로 대신하는 것"입니다. 요청을 보내고, 받은 HTML에서 원하는 조각만 골라냅니다.

`웹크롤링` · `requests` · `BeautifulSoup` · `HTTP` · `정적데이터`

## 핵심요약

- 크롤링은 `requests`로 서버에 HTML을 요청하고, `BeautifulSoup`로 그 HTML을 탐색 가능한 구조로 바꿔 원하는 데이터를 추출하는 과정이다.
- 서버는 클라이언트의 요청에 상태 코드(200 성공, 404 없음, 403 접근 거부 등)로 응답하므로, 상태 코드 확인이 필수다.
- 봇 차단을 피하려면 `User-Agent` 헤더로 "실제 브라우저처럼" 요청을 위장한다.
- `requests`는 서버가 처음에 완성해 보내주는 정적 HTML만 가져올 수 있고, 자바스크립트로 나중에 채워지는 동적 데이터는 별도 방법이 필요하다.
- 여러 페이지를 순회할 때는 `time.sleep()`으로 요청 간격을 두는 것이 서버 부하를 줄이는 기본 매너다.

## 1. 웹 크롤링의 통신 구조

### 1) 정의

웹 크롤링(web crawling / scraping)은 웹 페이지의 HTML을 프로그램으로 가져와, 그 안에서 필요한 데이터만 자동으로 뽑아내는 작업입니다. 사람이 브라우저로 페이지를 열어 내용을 읽는 과정을, 코드가 대신 수행한다고 보면 됩니다.

### 2) 왜 필요한가

- 공개된 웹 데이터(상품 정보, 뉴스, 리뷰 등)를 대량으로 모아 분석 자료로 만들 수 있습니다.
- 수작업 복사·붙여넣기로는 불가능한 규모의 데이터를 반복 가능하게 수집합니다.
- 머신러닝 모델에 넣을 학습 데이터를 확보하는 첫 단계가 되기도 합니다.

### 3) 핵심 흐름 재구성

크롤링은 네 단계로 흐릅니다.

```
[1] HTTP 요청으로 HTML 가져오기      (requests)
        ▼
[2] HTML을 탐색 가능한 구조로 변환   (BeautifulSoup)
        ▼
[3] 태그·클래스·id 기준으로 데이터 추출
        ▼
[4] 딕셔너리·리스트로 모아 표(DataFrame)로 정리
```

중요한 점은 크롤링이 "웹 서버와의 통신"이라는 사실입니다. 크롤러는 클라이언트(요청자) 역할을 하고, 서버에 "이 주소의 HTML을 주세요"라고 요청합니다. 서버는 상태 코드와 함께 데이터를 돌려줍니다. 그래서 응답을 처리하기 전에 상태 코드가 200(성공)인지 확인하는 습관이 필요합니다.

### 4) 쉬운 예시

도서관에 전화해 "3층 605번 책의 저자와 출판사를 알려주세요"라고 묻는 상황을 떠올려 보세요. 전화(요청)를 걸면 사서(서버)가 "네, 있습니다"(200)라거나 "그 책은 없어요"(404)라고 답합니다. 크롤링의 상태 코드 확인은 이 "있는지 없는지 먼저 확인하기"와 같습니다.

### 5) 코드 예시

아래는 크롤링 연습용으로 널리 쓰이는 공개 실습 사이트(books.toscrape.com)에서 도서 상세 정보를 가져오는 최소 예제입니다. 실제 실행 대신 흐름을 이해하는 데 목적을 둡니다.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

URL = "https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html"

# 실제 브라우저처럼 보이게 하는 신분증 역할의 헤더
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0"}

response = requests.get(URL, headers=headers)

if response.status_code == 200:                 # 성공 응답일 때만 진행
    soup = BeautifulSoup(response.text, "html.parser")

    # 제목: product_main 영역 안의 첫 h1 태그
    title = soup.find("div", class_="product_main").h1.text

    # 상세 테이블(th=항목명, td=값)을 딕셔너리로 수집
    info = {}
    for row in soup.select("table.table tr"):
        key = row.find("th").text
        value = row.find("td").text
        info[key] = value

    book = {"제목": title, **info}
    print(pd.Series(book))
else:
    print("요청 실패:", response.status_code)
```

핵심 문법 두 가지를 짚어 둡니다.

- `class_`처럼 언더바를 붙이는 이유: `class`는 파이썬 예약어라서 그대로 쓸 수 없습니다. BeautifulSoup는 `class_`로 받습니다.
- `dict.get("키")` 사용: 특정 항목이 없을 때 `dict["키"]`는 오류(KeyError)로 프로그램을 멈추게 하지만, `.get()`은 없으면 `None`을 돌려줘 안전합니다.

### 6) 헷갈리는 점

- `response.text`(문자열)와 `soup`(파싱된 객체)를 혼동하기 쉽습니다. `find()`, `select()` 같은 탐색 메서드는 `soup`에만 있습니다.
- `find()`는 조건에 맞는 **첫 번째** 요소 하나, `find_all()`은 **모든** 요소의 리스트를 돌려줍니다.

### 7) 한 줄 정리

> 웹 크롤링은 requests로 HTML을 받아 BeautifulSoup로 원하는 조각만 골라내는, "브라우저 흉내내기" 작업이다.

## 코드로 보기 — 여러 페이지 순회 크롤링

```python
import time
import requests
from bs4 import BeautifulSoup
import pandas as pd

BASE = "https://books.toscrape.com/"
headers = {"User-Agent": "Mozilla/5.0 ... Chrome/120.0.0.0"}

# 1단계: 목록 페이지에서 상세 페이지 링크 수집
res = requests.get(BASE + "index.html", headers=headers)
soup = BeautifulSoup(res.text, "html.parser")
cards = soup.find_all("article", class_="product_pod")
detail_urls = [BASE + "catalogue/" + c.h3.a["href"].replace("../", "") for c in cards]

# 2단계: 링크를 하나씩 방문하며 데이터 누적
records = []
for i, url in enumerate(detail_urls, 1):
    r = requests.get(url, headers=headers)
    s = BeautifulSoup(r.text, "html.parser")
    title = s.find("div", class_="product_main").h1.text
    records.append({"순번": i, "제목": title})
    time.sleep(0.5)     # 요청 간격 — 서버 부하를 줄이는 매너

df = pd.DataFrame(records)
print(df.shape)   # 예: (20, 2)
```

### 코드 목적

한 페이지가 아니라 목록에 걸린 여러 상세 페이지를 자동으로 돌며 데이터를 쌓아, 한 장의 표로 만드는 것이 목적입니다.

### 코드 흐름

1. 목록 페이지에서 상세 페이지 링크를 모두 모은다.
2. 링크를 반복문으로 하나씩 방문한다.
3. 각 페이지에서 원하는 값을 추출해 리스트에 누적한다.
4. 리스트를 `DataFrame`으로 변환해 표로 정리한다.

### 실행 결과 해석

`df.shape`가 `(20, 2)`로 나오면 20개 도서의 정보를 2개 열로 정상 수집했다는 뜻입니다. 상대 경로(`../`)를 절대 경로로 바꾸지 않으면 요청이 실패하므로, 링크 조합이 크롤링 성공의 관건이 됩니다.

### 실무 연결

가격 비교 서비스, 뉴스 모니터링, 리뷰 감성 분석 데이터 확보 등 대량 웹 데이터가 필요한 모든 곳에 쓰입니다. 다만 대상 사이트의 이용약관과 `robots.txt`를 확인하고, 요청 간격을 지키는 책임 있는 수집이 전제입니다.

## 직접 해보기

1. 상태 코드 403이 반환되었다면, 코드에서 가장 먼저 점검할 부분은 무엇일까?
2. `find()`와 `find_all()`의 반환값 타입은 각각 무엇인가? 하나만 필요할 때는 어느 것을 쓰는가?
3. 20개 페이지를 순회하는데 `time.sleep()` 없이 요청을 몰아 보냈더니 중간부터 응답이 실패했다. 원인과 해결책을 설명하라.

<details>
<summary>정답 보기</summary>

1. `User-Agent` 헤더 설정을 점검한다. 403은 접근 거부로, 서버가 요청을 봇으로 판단해 차단한 경우가 많다. 브라우저처럼 보이는 헤더를 추가하면 우회되는 경우가 있다(정상 접근이 허용된 사이트에 한함).
2. `find()`는 조건에 맞는 첫 요소 하나(Tag 객체 또는 None), `find_all()`은 모든 요소의 리스트를 반환한다. 하나만 필요하면 `find()`를 쓴다.
3. 짧은 시간에 요청이 몰리면 서버가 과도한 트래픽으로 판단해 일시 차단할 수 있다. `time.sleep(0.5)`처럼 요청 사이에 간격을 두어 부하를 낮추면 완화된다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| requests vs BeautifulSoup | requests는 HTML을 "가져오는" 통신 도구, BeautifulSoup는 가져온 HTML을 "해석·탐색"하는 파싱 도구 |
| find vs find_all | find는 첫 요소 하나, find_all은 조건에 맞는 전체 리스트 |
| 정적 데이터 vs 동적 데이터 | 서버가 처음부터 완성해 보내면 정적(requests로 가능), JS로 나중에 채우면 동적(별도 도구 필요) |
| `dict["키"]` vs `dict.get("키")` | 키가 없으면 앞은 오류로 중단, 뒤는 None 반환으로 안전 |

## 연결되는 개념

- 다음에 이어지는 개념: [API로 데이터 수집하기](02-api-crawling.md)
- 함께 보면 좋은 키워드: `HTTP`, `DOM`, `User-Agent`

## 셀프 체크

- [ ] 크롤링이 클라이언트-서버 통신이라는 점을 설명할 수 있다.
- [ ] 상태 코드 200/403/404의 의미를 말할 수 있다.
- [ ] `find`와 `find_all`을 구분해 쓸 수 있다.
- [ ] 정적 데이터와 동적 데이터의 차이를 안다.
- [ ] 여러 페이지 순회 시 요청 간격이 왜 필요한지 안다.

### 복습 질문 및 답변

**Q1. `User-Agent` 헤더는 왜 넣는가?**

<details>
<summary>답</summary>

서버가 짧은 시간 안에 쏟아지는 자동 요청을 봇으로 판단해 차단하는 것을 막기 위해서다. 실제 브라우저가 보내는 정보처럼 위장해 정상 사용자로 인식되게 한다.

</details>

**Q2. requests로 가져올 수 없는 데이터가 있는 이유는?**

<details>
<summary>답</summary>

requests는 서버가 처음 응답으로 완성해 보내는 정적 HTML만 받는다. 자바스크립트가 나중에 실행되며 채우는 동적 데이터는 응답 시점에 아직 HTML에 없어 보이지 않는다. 이 경우 브라우저를 실제로 구동하는 도구나 데이터 API를 직접 찾는 방법이 필요하다.

</details>

**Q3. 상대 경로를 절대 경로로 바꾸지 않으면 어떤 문제가 생기는가?**

<details>
<summary>답</summary>

`href`로 얻은 링크가 `../catalogue/...`처럼 상대 경로면, 그대로 요청하면 올바른 주소가 아니어서 접속에 실패한다. 앞에 기준 도메인(BASE URL)을 붙여 완전한 절대 경로로 만들어야 정상 요청이 된다.

</details>

## 한 줄 정리

> 웹 크롤링은 requests로 HTML을 받고 BeautifulSoup로 원하는 부분을 골라내는 통신 작업이며, 상태 코드 확인·헤더 설정·요청 간격이 안정적 수집의 기본이다.
