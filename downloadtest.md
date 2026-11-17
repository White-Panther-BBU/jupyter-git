# GitHub 주소에서 Python으로 파일 다운로드하기 (테스트 가이드)

> **핵심 목표:** GitHub에 있는 파일 또는 프로젝트를 Python 코드로
> 다운로드하여, 내가 지정한 **로컬 PC 경로에 저장**하고 실제 파일이
> 생성됐는지 확인한다.

------------------------------------------------------------------------

## 1. 다운로드 방식 선택하기

GitHub에서 가져오려는 대상에 따라 방법이 다릅니다.

  -----------------------------------------------------------------------
  대상                    권장 방식               설명
  ----------------------- ----------------------- -----------------------
  저장소 전체             ZIP 다운로드 또는       프로젝트 폴더 전체를
                          `git clone`             가져옴

  특정 Python 파일        Raw URL + `requests`    `.py` 파일 하나만 저장

  특정 데이터 파일        Raw URL + `requests`    CSV, JSON, TXT 등 개별
                                                  파일 저장
  -----------------------------------------------------------------------

이 문서는 외부 라이브러리 설치가 간단한 `requests`를 이용해
테스트합니다.

## 2. 사전 준비

### requests 설치

터미널(CMD 또는 PowerShell)에서 실행합니다.

``` bash
python -m pip install requests
```

가상환경을 사용한다면 **다운로드를 실행할 Python 가상환경에 설치**해야
합니다.

## 3. 테스트 A --- GitHub의 파일 하나를 로컬 경로에 다운로드

GitHub 파일 페이지 주소와 실제 다운로드 주소(Raw URL)는 다릅니다.

예를 들어 GitHub 파일 페이지가 다음과 같다고 가정합니다.

``` text
https://github.com/사용자명/저장소명/blob/main/example.py
```

실제 파일 다운로드에 사용하는 Raw URL은 다음 형태입니다.

``` text
https://raw.githubusercontent.com/사용자명/저장소명/main/example.py
```

### Python 코드

아래 코드에서 `raw_url`을 실제 파일의 Raw URL로, `save_path`를 원하는
로컬 저장 경로로 변경합니다.

``` python
from pathlib import Path
import requests

# 1) GitHub 파일의 Raw URL
raw_url = "https://raw.githubusercontent.com/사용자명/저장소명/main/example.py"

# 2) 로컬 PC에 저장할 경로
save_path = Path(r"C:\IT\github_download\example.py")

# 3) 저장 폴더가 없으면 생성
save_path.parent.mkdir(parents=True, exist_ok=True)

# 4) GitHub 파일 다운로드
response = requests.get(raw_url, timeout=30)
response.raise_for_status()  # HTTP 오류가 있으면 예외 발생

# 5) 파일 저장
save_path.write_bytes(response.content)

print("다운로드 완료!")
print(f"저장 위치: {save_path}")
print(f"파일 크기: {save_path.stat().st_size:,} bytes")
```

**중요:** `Path(r"...")`처럼 경로 앞에 `r`을 붙이면 Windows 경로의
역슬래시(`\`)를 편하게 사용할 수 있습니다.

## 4. 테스트 B --- GitHub 저장소 전체를 ZIP으로 다운로드

특정 파일이 아니라 프로젝트 전체를 가져오려면 저장소의 ZIP 파일을
다운로드할 수 있습니다.

``` python
from pathlib import Path
import requests
import zipfile
import io

# GitHub 저장소 정보
owner = "사용자명"
repo = "저장소명"
branch = "main"

# 로컬 저장 폴더
download_dir = Path(r"C:\IT\github_download")
download_dir.mkdir(parents=True, exist_ok=True)

# GitHub 저장소 ZIP 주소
zip_url = f"https://github.com/{owner}/{repo}/archive/refs/heads/{branch}.zip"

# ZIP 다운로드
response = requests.get(zip_url, timeout=60)
response.raise_for_status()

# ZIP 파일을 로컬에 저장
zip_path = download_dir / f"{repo}.zip"
zip_path.write_bytes(response.content)

# 압축 해제
extract_dir = download_dir / repo
extract_dir.mkdir(parents=True, exist_ok=True)

with zipfile.ZipFile(io.BytesIO(response.content)) as zip_file:
    zip_file.extractall(extract_dir)

print("저장소 다운로드 및 압축 해제 완료!")
print(f"ZIP 파일: {zip_path}")
print(f"압축 해제 경로: {extract_dir}")
```

압축을 풀면 GitHub 저장소의 폴더 구조가 로컬에 생성됩니다. ZIP에는 보통
저장소 이름과 브랜치명이 포함된 최상위 폴더가 들어 있습니다.

## 5. 테스트 C --- 다운로드한 파일이 실제로 있는지 확인

``` python
from pathlib import Path

target = Path(r"C:\IT\github_download")

if target.exists():
    print("폴더가 존재합니다.")
    for path in target.rglob("*"):
        if path.is_file():
            print(path)
else:
    print("지정한 경로가 존재하지 않습니다.")
```

## 6. GitHub 주소에서 확인해야 할 항목

  항목        확인 내용                 예시
  ----------- ------------------------- ---------------------------------
  사용자명    저장소 소유자             `octocat`
  저장소명    프로젝트 이름             `Hello-World`
  브랜치명    다운로드할 브랜치         `main` 또는 `master`
  파일 경로   저장소 내부 파일 위치     `src/example.py`
  Raw URL     파일의 실제 콘텐츠 주소   `raw.githubusercontent.com/...`
  로컬 경로   내 PC의 저장 위치         `C:\IT\github_download`

### 주의사항

-   `blob`이 포함된 웹페이지 주소를 그대로 `requests.get()`에 넣지
    마세요. 파일 하나를 받을 때는 **Raw URL**을 사용해야 합니다.
-   저장소가 비공개(Private)라면 위의 공개 다운로드 예제만으로는 접근할
    수 없습니다. 인증이 필요합니다.
-   저장소의 기본 브랜치가 `main`이 아닐 수 있으므로 실제 브랜치명을
    확인하세요.
-   ZIP 다운로드는 저장소 전체를 가져오며, Git 이력(commit history)은
    포함하지 않습니다.
-   출처와 라이선스를 확인하고, 신뢰할 수 있는 코드인지 검토한 후
    실행하세요.
-   다운로드한 파일을 실행하기 전에 파일 내용과 출처를 확인하세요.

## 7. 테스트 체크리스트

-   [ ] `requests` 설치 완료
-   [ ] GitHub 저장소 및 브랜치 확인
-   [ ] 개별 파일의 Raw URL 확인
-   [ ] 로컬 저장 경로 지정
-   [ ] Python 다운로드 코드 실행
-   [ ] 저장 폴더에서 파일 생성 여부 확인
-   [ ] 다운로드한 파일 내용 확인

------------------------------------------------------------------------

## 핵심 정리

**GitHub URL → Python `requests.get()` → 응답 데이터 → 로컬 경로에
저장**

``` python
response = requests.get(raw_url)
response.raise_for_status()
save_path.write_bytes(response.content)
```

이 세 단계가 GitHub 파일을 Python으로 내려받아 로컬 PC에 저장하는 핵심
흐름입니다.
