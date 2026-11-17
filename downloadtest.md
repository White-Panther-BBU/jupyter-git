# 🎉 우리인재개발원 수강생 여러분!

오늘도 배움이라는 새로운 모험을 시작하신 여러분을 응원합니다! 🚀

때로는 코드가 에러를 내고, 모델이 학습을 거부하고,
컴퓨터가 내 마음처럼 움직이지 않아도 괜찮습니다.

**에러는 성장의 신호! 실패는 실력의 업데이트!** 😎

오늘 배운 하나가 내일의 실력이 되고,
오늘의 작은 도전이 멋진 미래를 만들어 갑니다.

서로 웃고, 함께 배우고, 즐겁게 도전하면서
우리 모두 원하는 목표까지 힘차게 달려가 봅시다!

**여러분의 오늘은 행복하게, 배움은 즐겁게,
미래는 누구보다 멋지게!** 🌈

화이팅입니다! 💪🔥

---


# GitHub 및 모델 저장소에서 Python으로 파일 다운로드하기

> **핵심 목표:** GitHub 또는 모델 저장소(Hugging Face 등)에 있는 Python
> 코드, 데이터, 학습 모델 파일을 Python으로 다운로드하여 내가 지정한
> **로컬 PC 경로에 저장**하고 확인한다.

이 문서는 Windows + Python 환경에서 직접 테스트할 수 있도록
작성했습니다.

------------------------------------------------------------------------

## 1. 다운로드 대상에 따른 방법 선택

  -----------------------------------------------------------------------
  다운로드 대상           권장 방법               설명
  ----------------------- ----------------------- -----------------------
  GitHub의 개별 코드·설정 Raw URL + `requests`    `.py`, `.json`, `.txt`
  파일                                            등

  GitHub 저장소 전체      ZIP 다운로드 또는       프로젝트 전체
                          `git clone`             

  GitHub Release 모델     Release 다운로드 URL +  배포 파일 또는 가중치
  파일                    `requests`              

  Hugging Face 모델 파일  `hf_hub_download()`     `.pt`, `.bin`,
  1개                                             `.safetensors`,
                                                  `.keras` 등

  Hugging Face 모델 전체  `snapshot_download()`   모델 저장소 파일 전체

  대용량 공개 파일의 직접 `requests` 스트리밍     메모리에 전부 올리지
  URL                                             않고 저장
  -----------------------------------------------------------------------

**중요:** 모델 파일은 GitHub 저장소에 일반 파일로 올라와 있을 수도
있지만, 대용량 파일은 Git LFS 또는 Hugging Face 같은 모델 저장소에
올라오는 경우가 많습니다.

------------------------------------------------------------------------

## 2. 사전 준비

### 기본 라이브러리 설치

``` bash
python -m pip install requests
```

Hugging Face 모델을 다운로드할 경우 추가 설치:

``` bash
python -m pip install -U huggingface_hub
```

가상환경을 사용한다면 다운로드 코드를 실행할 **동일한 Python
가상환경**에 설치해야 합니다.

------------------------------------------------------------------------

# Part A. GitHub 파일 다운로드

## 3. GitHub의 파일 하나 다운로드

GitHub 파일 페이지 주소와 실제 다운로드 주소(Raw URL)는 다릅니다.

파일 페이지 주소 예시:

``` text
https://github.com/사용자명/저장소명/blob/main/example.py
```

Raw URL 형태:

``` text
https://raw.githubusercontent.com/사용자명/저장소명/main/example.py
```

### Python 코드

``` python
from pathlib import Path
import requests
import os


# GitHub 파일의 Raw URL
raw_url = "https://raw.githubusercontent.com/사용자명/저장소명/main/example.py"

# 로컬 저장 경로
# os.makedirs("/lec_model", exist_ok=True)
save_path = Path(r"C:\IT\github_download\example.py")
save_path.parent.mkdir(parents=True, exist_ok=True)

response = requests.get(raw_url, timeout=30)
response.raise_for_status()

save_path.write_bytes(response.content)

print("다운로드 완료!")
print("저장 위치:", save_path)
print("파일 크기:", save_path.stat().st_size, "bytes")
```

**주의:** `blob` 주소를 그대로 다운로드에 사용하지 말고, 파일 하나를
받을 때는 Raw URL을 사용합니다.

## 4. GitHub 저장소 전체를 ZIP으로 다운로드

``` python
from pathlib import Path
import requests
import zipfile
import io

owner = "사용자명"
repo = "저장소명"
branch = "main"

download_dir = Path(r"C:\IT\github_download")
download_dir.mkdir(parents=True, exist_ok=True)

zip_url = (
    f"https://github.com/{owner}/{repo}"
    f"/archive/refs/heads/{branch}.zip"
)

response = requests.get(zip_url, timeout=60)
response.raise_for_status()

zip_path = download_dir / f"{repo}.zip"
zip_path.write_bytes(response.content)

extract_dir = download_dir / repo
extract_dir.mkdir(parents=True, exist_ok=True)

with zipfile.ZipFile(io.BytesIO(response.content)) as zip_file:
    zip_file.extractall(extract_dir)

print("저장소 다운로드 및 압축 해제 완료!")
print("ZIP:", zip_path)
print("압축 해제:", extract_dir)
```

ZIP에는 저장소의 현재 파일이 들어가며 Git commit 이력은 포함되지
않습니다.

------------------------------------------------------------------------

# Part B. 학습 모델 파일 다운로드

## 5. GitHub에 올라온 모델 파일 다운로드

GitHub에 모델 파일이 일반 파일 또는 Release 자산으로 공개되어 있고, 직접
다운로드 URL을 제공한다면 `requests`로 받을 수 있습니다.

예를 들어 `.pt`, `.keras`, `.h5`, `.onnx` 같은 파일을 받을 수 있습니다.

### 대용량 파일에 권장하는 스트리밍 다운로드 코드

``` python
from pathlib import Path
import requests

# 실제 다운로드 가능한 파일 URL로 변경
model_url = "https://실제-모델-다운로드-주소/model.pt"

# 저장할 로컬 경로
save_path = Path(r"C:\IT\models\model.pt")
save_path.parent.mkdir(parents=True, exist_ok=True)

# stream=True: 파일 전체를 메모리에 한 번에 올리지 않음
with requests.get(model_url, stream=True, timeout=(15, 120)) as response:
    response.raise_for_status()

    with open(save_path, "wb") as file:
        for chunk in response.iter_content(chunk_size=1024 * 1024):
            if chunk:
                file.write(chunk)

print("모델 다운로드 완료!")
print("저장 경로:", save_path)
print("파일 크기:", f"{save_path.stat().st_size:,} bytes")
```

`model_url`에는 브라우저에서 파일 다운로드가 실제로 시작되는 직접 URL을
넣어야 합니다. GitHub의 파일 페이지 주소가 아니라 Raw 또는 Release
다운로드 주소인지 확인하세요.

### GitHub 대용량 파일 주의사항

-   GitHub 저장소가 Git LFS를 사용하면 Raw URL에서 실제 모델 대신 작은
    포인터 파일을 받을 수 있습니다.
-   대용량 모델은 GitHub Release 자산 또는 모델 전용 저장소를 이용하는
    편이 좋습니다.
-   저장소 ZIP은 Git LFS 모델 원본까지 항상 원하는 형태로 포함한다고
    가정하면 안 됩니다.
-   비공개 저장소는 인증이 필요합니다.

------------------------------------------------------------------------

## 6. Hugging Face에서 모델 파일 하나 다운로드

Hugging Face는 머신러닝 모델과 데이터셋을 공유하는 대표적인
저장소입니다.

모델 페이지 예시:

``` text
https://huggingface.co/조직명-또는-사용자명/모델명
```

모델 저장소에서 실제 파일명(예: `model.safetensors`,
`pytorch_model.bin`, `config.json`)을 확인합니다.

### 라이브러리 설치

``` bash
python -m pip install -U huggingface_hub
```

### Python 코드

``` python
from pathlib import Path
from huggingface_hub import hf_hub_download

# Hugging Face 저장소 ID
repo_id = "조직명-또는-사용자명/모델명"

# 저장할 로컬 폴더
local_dir = Path(r"C:\IT\models")
local_dir.mkdir(parents=True, exist_ok=True)

# 저장소 안에서 다운로드할 파일명
filename = "model.safetensors"

downloaded_path = hf_hub_download(
    repo_id=repo_id,
    filename=filename,
    local_dir=str(local_dir)
)

print("모델 파일 다운로드 완료!")
print("저장 위치:", downloaded_path)
```

`repo_id`와 `filename`은 실제 Hugging Face 모델 페이지의 값으로 바꿔야
합니다.

------------------------------------------------------------------------

## 7. Hugging Face 모델 저장소 전체 다운로드

모델이 여러 파일로 구성되어 있다면 저장소 전체를 다운로드할 수 있습니다.

``` python
from pathlib import Path
from huggingface_hub import snapshot_download

repo_id = "조직명-또는-사용자명/모델명"

local_dir = Path(r"C:\IT\models\my_model")
local_dir.mkdir(parents=True, exist_ok=True)

downloaded_dir = snapshot_download(
    repo_id=repo_id,
    local_dir=str(local_dir)
)

print("모델 저장소 다운로드 완료!")
print("저장 위치:", downloaded_dir)
```

예를 들어 모델 저장소에는 다음과 같은 파일이 함께 있을 수 있습니다.

``` text
C:\IT\models\my_model
│
├─ config.json
├─ model.safetensors
├─ tokenizer.json
├─ tokenizer_config.json
└─ README.md
```

실제 구성은 모델마다 다릅니다. 모든 모델에 tokenizer 파일이 있는 것은
아닙니다.

------------------------------------------------------------------------

## 8. 비공개 또는 접근 제한 모델 다운로드

Hugging Face에서 비공개 모델 또는 접근 승인이 필요한 모델은 인증이
필요할 수 있습니다.

1.  Hugging Face 계정으로 로그인
2.  해당 모델 페이지에서 접근 권한 및 사용 조건 확인
3.  필요한 경우 모델 접근 신청 또는 약관 동의
4.  본인 계정의 Access Token 발급
5.  토큰을 안전하게 사용하여 다운로드

### 토큰을 환경변수로 설정하는 예시 (PowerShell)

``` powershell
$env:HF_TOKEN = "본인의_Hugging_Face_Token"
```

Python 코드:

``` python
import os
from huggingface_hub import hf_hub_download

file_path = hf_hub_download(
    repo_id="조직명-또는-사용자명/모델명",
    filename="model.safetensors",
    token=os.environ["HF_TOKEN"],
    local_dir=r"C:\IT\models"
)

print(file_path)
```

**보안 주의:** 토큰을 Python 코드에 직접 입력하거나 GitHub에 업로드하지
마세요. 다른 사람과 공유하는 `.py`, `.ipynb`, `.md` 파일에도 토큰을 넣지
마세요. 토큰은 비밀번호처럼 취급해야 합니다.

------------------------------------------------------------------------

# Part C. 다운로드 결과 확인

## 9. 모델 파일이 실제로 저장됐는지 확인

``` python
from pathlib import Path

model_dir = Path(r"C:\IT\models")

if not model_dir.exists():
    print("모델 폴더가 없습니다.")
else:
    for file_path in model_dir.rglob("*"):
        if file_path.is_file():
            print(
                file_path,
                f"({file_path.stat().st_size:,} bytes)"
            )
```

### 파일 확장자별 예시

  확장자           용도 예시
  ---------------- ------------------------------------
  `.pt`            PyTorch / Ultralytics YOLO 모델 등
  `.pth`           PyTorch 체크포인트
  `.keras`         Keras 모델
  `.h5`            Keras/HDF5 모델 또는 데이터
  `.onnx`          프레임워크 간 모델 교환 형식
  `.safetensors`   안전한 텐서 저장 형식
  `.bin`           모델 가중치 등 바이너리 파일

확장자만으로 모델 종류와 실행 방법이 완전히 결정되는 것은 아닙니다.
모델의 README, 설정 파일, 프레임워크 버전도 확인해야 합니다.

## 10. 다운로드 후 모델 로드 테스트

### YOLO `.pt` 파일 예시

``` bash
python -m pip install ultralytics
```

``` python
from ultralytics import YOLO

model = YOLO(r"C:\IT\models\model.pt")

print("YOLO 모델 로드 성공!")
```

### Keras `.keras` 파일 예시

``` python
import tensorflow as tf

model = tf.keras.models.load_model(
    r"C:\IT\models\model.keras"
)

model.summary()
```

모델 파일을 로드하려면 해당 모델을 만든 프레임워크 및 필요한 라이브러리
버전이 호환되어야 합니다.

------------------------------------------------------------------------

## 11. 테스트 체크리스트

-   [ ] `requests` 설치 완료
-   [ ] 필요하면 `huggingface_hub` 설치 완료
-   [ ] GitHub 또는 Hugging Face 모델 페이지 확인
-   [ ] 저장소 ID와 파일명 확인
-   [ ] 다운로드 가능한 URL 또는 접근 권한 확인
-   [ ] 로컬 저장 경로 지정
-   [ ] Python 다운로드 코드 실행
-   [ ] 파일 크기와 저장 여부 확인
-   [ ] README 및 라이선스 확인
-   [ ] 필요한 프레임워크로 모델 로드 테스트

------------------------------------------------------------------------

## 핵심 정리

### GitHub 개별 파일

``` python
response = requests.get(raw_url)
response.raise_for_status()
save_path.write_bytes(response.content)
```

### 대용량 직접 다운로드

``` python
with requests.get(model_url, stream=True) as response:
    response.raise_for_status()
    with open(save_path, "wb") as f:
        for chunk in response.iter_content(1024 * 1024):
            if chunk:
                f.write(chunk)
```

### Hugging Face 모델 파일 하나

``` python
from huggingface_hub import hf_hub_download

hf_hub_download(
    repo_id="사용자명/모델명",
    filename="model.safetensors",
    local_dir=r"C:\IT\models"
)
```

**기억할 점:** GitHub는 코드와 프로젝트를 관리하는 데 편리하고, Hugging
Face는 학습된 모델과 관련 파일을 내려받는 데 편리합니다. 실제 다운로드
전에는 공개 여부, 접근 권한, 라이선스, 파일 크기 및 모델 사용 조건을
확인하세요.

------------------------------------------------------------------------

# Part D. 여러 Hugging Face 모델을 순서대로 시도하기 (Fallback)

## 12. 첫 번째 모델 실패 시 대체 모델 다운로드

질문에 나온 방식은 **Fallback(대체 경로)** 패턴입니다. 첫 번째
저장소에서 다운로드가 실패하면 두 번째 저장소를 시도합니다.

다음 예제는 번호판 탐지용 YOLO `.pt` 가중치를 다운로드하고, 결과를
`C:\IT\models\license_plate`에 저장합니다.

``` python
from pathlib import Path
from huggingface_hub import hf_hub_download

# 모델을 저장할 로컬 폴더
download_dir = Path(r"C:\IT\models\license_plate")
download_dir.mkdir(parents=True, exist_ok=True)

# 먼저 시도할 모델과 대체 모델
model_repositories = [
    "Koushim/yolov8-license-plate-detection",
    "love671/yolov8-license-plate-detection",
]

weights_path = None

for repo_id in model_repositories:
    try:
        print(f"\n모델 다운로드 시도: {repo_id}")

        weights_path = hf_hub_download(
            repo_id=repo_id,
            filename="best.pt",
            local_dir=str(download_dir)
        )

        print(f"모델 다운로드 완료: {weights_path}")
        break  # 성공했으므로 반복 종료

    except Exception as e:
        print(f"다운로드 실패: {repo_id}")
        print(f"실패 사유: {e}")

if weights_path is None:
    raise RuntimeError("모든 모델 저장소에서 다운로드에 실패했습니다.")

print("\n최종 모델 경로:", weights_path)
```

### 코드 흐름 이해하기

  코드                     역할
  ------------------------ ---------------------------------
  `model_repositories`     시도할 저장소 목록
  `for repo_id in ...`     저장소를 순서대로 확인
  `hf_hub_download()`      지정한 저장소에서 파일 다운로드
  `filename="best.pt"`     저장소 내부에서 받을 파일명
  `local_dir`              다운로드한 파일을 둘 로컬 폴더
  `except`                 해당 저장소 다운로드 실패 처리
  `break`                  다운로드 성공 후 반복 종료
  `weights_path is None`   모든 시도가 실패했는지 확인

## 13. 다운로드한 모델을 YOLO에 연결하기

다운로드가 성공하면 반환된 경로를 그대로 Ultralytics YOLO에 전달할 수
있습니다.

``` bash
python -m pip install -U ultralytics huggingface_hub
```

``` python
from ultralytics import YOLO

model = YOLO(weights_path)

print("YOLO 모델 로드 완료!")

# 이미지 테스트
results = model.predict(
    source=r"C:\IT\images\car.jpg",
    conf=0.25,
    save=True
)

print("번호판 탐지 테스트 완료!")
```

`weights_path`는 앞 단계에서 성공한 모델 파일의 경로입니다. 다운로드와
모델 로드는 서로 다른 단계이므로, 파일 다운로드가 성공해도 모델 구조나
라이브러리 버전이 맞지 않으면 로드 또는 추론에 실패할 수 있습니다.

## 14. 꼭 확인해야 할 사항

1.  `repo_id`는 Hugging Face의 실제 저장소 ID여야 합니다.
2.  `filename`은 저장소 파일 목록에 실제로 존재하는 이름이어야 합니다.
    `best.pt`가 없고 `weights/best.pt`에 있다면 그 경로를 지정해야
    합니다.
3.  공개 저장소라도 삭제되었거나 접근 제한이 생기면 다운로드가 실패할 수
    있습니다.
4.  대체 저장소의 모델은 첫 번째 모델과 클래스 수, 클래스 순서, 학습
    데이터 및 모델 구조가 다를 수 있습니다.
5.  따라서 **다운로드 성공만으로 두 모델이 동일하거나 서로 대체
    가능하다고 판단하면 안 됩니다.** 모델 카드와 클래스 정보를 확인하고
    실제 이미지로 검증하세요.
6.  비공개 또는 접근 승인 모델은 Hugging Face 인증 토큰이 필요할 수
    있습니다.

### 한 줄 요약

``` text
첫 번째 모델 다운로드
        ↓ 실패
두 번째 대체 모델 다운로드
        ↓ 성공
weights_path에 경로 저장
        ↓
YOLO(weights_path)로 모델 로드
```

이 패턴을 사용하면 모델 저장소 주소가 바뀌거나 첫 번째 저장소에 문제가
생겼을 때 다른 저장소를 자동으로 시도할 수 있습니다.
