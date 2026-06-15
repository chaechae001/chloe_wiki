# 라이브러리 설치하기 — pip와 파일

> 라이브러리를 고르고 환경도 준비했습니다. 이제 진짜로 설치할 차례. 보통은 `pip install` 한 줄이면 되지만, PyPI에 없는 최신 버전이나 GitHub의 코드를 직접 설치해야 할 때도 있습니다. 설치의 두 갈래를 정리합니다.

`pip` `requirements.txt` `wheel` `tar.gz` `whl` `의존성 관리` `pip freeze`

## 핵심요약

- 라이브러리 설치는 크게 "PyPI에서 pip로" 또는 "파일로" 두 갈래다.
- `pip`는 PyPI 저장소에서 패키지를 검색·설치하고, 의존하는 패키지까지 자동으로 함께 설치한다.
- 패키지가 많으면 `requirements.txt`에 목록을 적어 한 번에 설치한다.
- `pip freeze`로 현재 환경의 설치 목록을 파일로 내보낼 수 있다.
- 파일 설치에는 빌드된 `.whl`(wheel)과 소스 코드 `.tar.gz` 두 형식이 있다.
- wheel은 빠르고 간단, tar.gz는 유연하지만 빌드가 필요해 느릴 수 있다.

## 개념별 정리

### pip로 설치하기

**1. 정의**
`pip`는 PyPI에 공유된 패키지를 검색하고 설치하는 도구입니다. 설치·업그레이드·제거·목록 관리를 할 수 있습니다.

**2. 왜 필요한가?**
외부 라이브러리는 PyPI에 있고, 그걸 내 환경으로 가져오는 표준 통로가 pip입니다. 명령어 한 줄로 설치가 끝납니다.

**3. 예시 — 주요 명령어**

```bash
pip install pandas                 # 설치
pip install pandas==2.2.1          # 특정 버전 설치
pip list                           # 설치된 전체 패키지 & 버전 확인
pip list | grep pandas             # 특정 패키지만 확인
pip install --upgrade pandas       # 버전 업그레이드
pip uninstall pandas               # 제거
```

**4. 헷갈리기 쉬운 점**
`pip install pandas` 하나를 실행했는데 numpy 같은 패키지가 줄줄이 함께 깔리는 걸 보고 당황하기 쉽습니다. 이는 오류가 아니라 **pip의 의존성 관리 기능**입니다. pandas가 동작하려면 필요한 패키지를 pip가 알아서 챙겨 주는 것입니다.

**5. 한 줄 정리**
pip는 "PyPI에서 골라 설치하고, 필요한 것까지 알아서 챙겨 주는" 설치 도구입니다.

### requirements.txt — 한 번에 설치

**1. 정의**
설치해야 할 패키지들과 버전 목록을 적어 둔 파일입니다. (이 이름은 관례적으로 정해진 이름입니다.)

**2. 왜 필요한가?**
설치할 패키지가 수십 개면 일일이 `pip install`을 치기 번거롭습니다. 목록 파일 하나로 한 번에 설치하면 빠르고, 협업자와 환경을 똑같이 맞출 수 있습니다.

**3. 예시**

```text
# requirements.txt
pandas==2.2.1
matplotlib==3.9.0
seaborn==0.13.2
```

```bash
# 목록 파일을 읽어 안에 적힌 패키지를 모두 설치
pip install -r requirements.txt
```

내 환경의 설치 목록을 거꾸로 파일로 뽑아낼 수도 있습니다.

```bash
# 현재 환경에 설치된 모든 패키지와 버전을 파일로 내보내기
pip freeze > requirements.txt
```

**4. 헷갈리기 쉬운 점**
`-r` 옵션은 "뒤에 오는 파일을 읽어 그 안의 목록을 설치하라"는 뜻입니다. `pip install requirements.txt`처럼 `-r`을 빠뜨리면 파일 이름을 패키지 이름으로 오해해 설치에 실패합니다.

**5. 한 줄 정리**
`requirements.txt`는 "장보기 목록"이고, `-r`은 그 목록대로 한 번에 담아 오는 명령입니다.

> 비유: `pip freeze`는 우리 집 냉장고에 든 재료를 목록으로 적는 일, `pip install -r`은 그 목록을 들고 장을 보는 일입니다.

### 파일로 설치하기 — wheel(.whl)과 .tar.gz

**1. 정의**
PyPI를 거치지 않고, 패키지 파일을 직접 받아 설치하는 방법입니다. 두 가지 형식이 있습니다.
- **wheel(`.whl`):** 빌드(컴파일)가 끝난 배포판
- **`.tar.gz`:** 소스 코드를 압축한 배포판

**2. 왜 필요한가?**
PyPI에 아직 등록되지 않았거나, PyPI에 없는 최신 버전을 써야 하거나, GitHub의 코드를 직접 설치해야 할 때 파일 설치가 필요합니다.

**3. 예시**

```bash
# wheel 파일로 설치 (빌드 과정 없이 바로 설치)
pip install wheel_filename.whl

# tar.gz는 압축을 풀고 설치
tar -zxvf 파일명.tar.gz
pip install .            # 현재 폴더(.)의 소스 코드를 설치

# GitHub 저장소에서 소스 코드를 직접 가져와 설치도 가능
git clone https://github.com/username/library_name.git
```

wheel 파일 이름에는 규칙이 있습니다.

```text
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl

예) requests-2.31.0-py3-none-any.whl
   = requests 패키지 / 2.31.0 버전 / python3 호환 / 모든 플랫폼에서 호환
```

**4. 헷갈리기 쉬운 점**
wheel은 이미 빌드돼 있어 빠르고 간단하지만, 특정 플랫폼·파이썬 버전에 종속될 수 있습니다. tar.gz는 어디서나 빌드할 수 있어 유연하지만, 적절한 컴파일러·라이브러리가 필요하고 빌드에 시간이 걸릴 수 있습니다.

**5. 한 줄 정리**
빠르고 간단하면 wheel, 유연하지만 빌드가 필요하면 tar.gz.

## 코드로 보기 — wheel vs .tar.gz 한눈에 비교

| | `.tar.gz` 파일 | wheel(`.whl`) 파일 |
| --- | --- | --- |
| 형태 | 소스 코드 배포판 (여러 파일을 묶고 gzip 압축) | 빌드된 패키지의 배포 포맷 |
| 사용 사례 | 소스 코드를 직접 수정·검토하고 싶을 때 | 컴파일 과정 없이 바로 설치하고 싶을 때 |
| 장점 | 유연성이 높고 모든 시스템에서 빌드 가능 | 설치가 빠르고 간단함 |
| 단점 | 빌드가 복잡하고 시간이 오래 걸릴 수 있음 | 특정 플랫폼·파이썬 버전에 종속될 수 있음 |

**코드목적**
같은 "파일 설치"라도 두 형식의 성격이 다름을 한 표로 정리하는 것이 목적입니다.

**해석**
대부분의 일반 설치는 wheel로 충분합니다. 소스를 직접 들여다보거나 고쳐야 하는 특수한 경우에 tar.gz를 씁니다. PyPI의 다운로드 페이지에 가면 한 패키지에 대해 `.tar.gz`(Source Distribution)와 여러 `.whl`(Built Distributions)이 함께 올라와 있는 걸 볼 수 있습니다.

**실무 연결**
보통은 `pip install 이름`으로 끝나지만, 사내 비공개 라이브러리나 PyPI 미등록 도구를 받을 때 파일/Git 설치를 쓰게 됩니다. 설치가 끝난 뒤 로그를 보면 `whl(wheel)` 파일로 설치됐는지 확인할 수 있습니다.

## 직접 해보기

1. 가상환경을 켠 상태에서 `requests`를 설치하고, `pip list`로 함께 깔린 패키지가 있는지 살펴보세요.
2. `pip freeze > requirements.txt`로 현재 환경의 목록을 파일로 뽑아 보세요.
3. `requests-2.31.0-py3-none-any.whl`이라는 이름을 보고, 패키지명·버전·호환 파이썬·플랫폼을 각각 읽어 보세요.

## 헷갈리기 쉬운 포인트

- **`pip install 이름` vs `pip install -r 파일`:** 전자는 패키지 하나, 후자는 목록 파일대로 여러 개.
- **명시 안 한 패키지가 깔림:** 오류가 아니라 pip의 의존성 자동 설치입니다.
- **wheel vs tar.gz:** 빌드 완료(빠름) vs 소스 코드(유연하나 빌드 필요).

## 연결되는 개념

- 이전 글: [가상환경 — 프로젝트별 작업실 만들기](06-virtual-environment.md) — 설치는 활성화된 환경 안에서 합니다.
- 다음 글: [라이브러리 불러오기와 활용](08-import-and-using-libraries.md) — 설치한 라이브러리를 import해 씁니다.
- 함께 보면 좋은 글: [내 코드를 세상과 공유하기](09-open-source-and-packaging.md) — 내가 만든 패키지를 거꾸로 wheel/tar.gz로 만들어 배포합니다.
- 더 찾아볼 키워드: `site-packages`, ABI, 플랫폼 태그(manylinux), `pip show`

## 셀프 체크

- [ ] `pip install`로 패키지를 설치할 수 있다.
- [ ] 특정 버전 설치(`==`)와 업그레이드(`--upgrade`)를 구분한다.
- [ ] 명시하지 않은 패키지가 함께 깔리는 이유(의존성)를 안다.
- [ ] `requirements.txt`로 한 번에 설치할 수 있다.
- [ ] `pip freeze`로 목록을 내보낼 수 있다.
- [ ] wheel과 tar.gz의 차이를 설명할 수 있다.

**복습 질문 및 답변**

- (기본) pandas를 설치하는 명령은? → `pip install pandas`.
- (이해 확인) `pip install -r requirements.txt`에서 `-r`은 무엇을 하나요? → 뒤의 파일을 읽어 그 안에 적힌 패키지를 모두 설치합니다.
- (응용) PyPI에 없는 최신 버전을 써야 한다면 어떤 방법이 있나요? → `.whl`/`.tar.gz` 파일 설치나 GitHub에서 직접 가져와 설치합니다.

## 한 줄 정리

> 평소엔 `pip install`과 `requirements.txt`로 충분하고, PyPI에 없거나 소스를 직접 다뤄야 할 때 wheel·tar.gz 파일 설치를 씁니다.
