# VTON ComfyUI API 🎨👔

AI 기반 가상 피팅(Virtual Try-On) 솔루션을 위한 Flask API 서버입니다. ComfyUI를 백엔드로 활용하여 IDM-VTON과 CatVTON 모델을 통해 고품질의 가상 착용 이미지를 생성합니다.

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-2.0+-green.svg)](https://flask.palletsprojects.com/)
[![ComfyUI](https://img.shields.io/badge/ComfyUI-0.2.2-orange.svg)](https://github.com/comfyanonymous/ComfyUI)

## 📋 목차

- [주요 기능](#-주요-기능)
- [시스템 아키텍처](#-시스템-아키텍처)
- [설치 방법](#-설치-방법)
- [사용 방법](#-사용-방법)
- [API 엔드포인트](#-api-엔드포인트)
- [워크플로우](#-워크플로우)
- [프로젝트 구조](#-프로젝트-구조)
- [스크린샷](#-스크린샷)
- [참고 자료](#-참고-자료)

## ✨ 주요 기능

### 1. 가상 피팅 (Virtual Try-On)

- **IDM-VTON**: 고품질 사람+옷 합성
- **CatVTON**: 빠르고 정확한 착용 이미지 생성
- 다중 이미지 동시 처리 지원

### 2. AI 드레스 생성

- 텍스트 프롬프트만으로 웨딩드레스 생성
- 사람 이미지 + 프롬프트로 맞춤형 드레스 착용 이미지 생성
- 6장 동시 생성으로 다양한 옵션 제공

### 3. 이미지 유사도 검색

- VGG16 기반 특징 추출
- 코사인 유사도 계산으로 유사 드레스 추천
- 실시간 배경 제거 기능

### 4. ComfyUI 자동 관리

- Flask 서버 시작 시 ComfyUI 자동 실행
- 워크플로우 큐 모니터링
- 메모리 자동 정리 (CUDA + ComfyUI 캐시)

## 🏗️ 시스템 아키텍처

```
┌─────────────┐      HTTP/WebSocket      ┌──────────────┐
│   Frontend  │ ◄──────────────────────► │  Flask API   │
│  (Web UI)   │                           │   Server     │
└─────────────┘                           └──────┬───────┘
                                                 │
                                                 │ WebSocket
                                                 ▼
                                          ┌──────────────┐
                                          │   ComfyUI    │
                                          │   Backend    │
                                          └──────┬───────┘
                                                 │
                        ┌────────────────────────┼────────────────────┐
                        │                        │                    │
                   ┌────▼─────┐          ┌──────▼──────┐      ┌──────▼──────┐
                   │ IDM-VTON │          │  CatVTON    │      │   VGG16     │
                   │  Model   │          │   Model     │      │   Model     │
                   └──────────┘          └─────────────┘      └─────────────┘
```

## 📦 설치 방법

### 사전 요구사항

- Python 3.8 이상
- NVIDIA GPU (CUDA 지원)
- ComfyUI 0.2.2 설치
- 10GB 이상의 여유 디스크 공간

### 1. ComfyUI 설치 및 커스텀 노드 설정

```bash
# ComfyUI 설치 (E:\ComfyUI_0.2.2\ComfyUI 경로 예시)
git clone https://github.com/comfyanonymous/ComfyUI.git

# 필수 커스텀 노드 설치
cd ComfyUI/custom_nodes

# IDM-VTON
git clone https://github.com/neuralninja22/IDM-VTON.git

# CatVTON Wrapper
git clone https://github.com/chflame163/ComfyUI_CatVTON_Wrapper.git

# 기타 필수 노드
git clone https://github.com/Kosinkadink/ComfyUI-Advanced-ControlNet.git comfyui_controlnet_aux
git clone https://github.com/storyicon/comfyui_segment_anything.git
git clone https://github.com/pythongosssss/ComfyUI-Custom-Scripts.git
```

### 2. API 서버 설치

```bash
git clone https://github.com/Me-in-U/VTON_ComfyUI_API.git
cd VTON_ComfyUI_API

# 필수 패키지 설치
pip install flask flask-cors pillow numpy scipy tensorflow numba psutil requests-toolbelt
```

### 3. 모델 다운로드

**Checkpoint:**

- [Beautiful Realistic Asians](https://civitai.com/models/25494?modelVersionId=63786) 다운로드
- `ComfyUI/models/checkpoints/` 폴더에 배치

**VAE:**

- [vae-ft-mse-840000-ema-pruned](https://huggingface.co/stabilityai/sd-vae-ft-mse-original/blob/main/vae-ft-mse-840000-ema-pruned.ckpt) 다운로드
- `ComfyUI/models/vae/` 폴더에 배치

### 4. 경로 설정

`__init__.py` 파일에서 경로를 시스템에 맞게 수정:

```python
COMFYUI_API_BASE_DIRECTORY = "E:\\Languages\\Apache24\\ComfyUI_API"
COMFYUI_BASE_DIRECTORY = "E:\\ComfyUI_0.2.2\\ComfyUI"
SERVER_ADDRESS = "127.0.0.1:8188"
```

## 🚀 사용 방법

### 서버 실행

```bash
python __init__.py
```

서버가 `http://0.0.0.0:1557`에서 실행되며, ComfyUI가 자동으로 시작됩니다.

### 웹 인터페이스 접속

브라우저에서 `http://localhost:1557`로 접속하면 테스트 UI가 표시됩니다.

## 📡 API 엔드포인트

### 1. 프롬프트로 드레스 생성

```http
POST /new_dress
Content-Type: multipart/form-data

positive_prompt: "elegant wedding dress, lace details, long train"
negative_prompt: "low quality, blurry"
```

**응답:**

```json
{
  "message": "Image processed successfully",
  "results": ["base64_image_1", "base64_image_2", ...],
  "imageMapKeys": [0, 1, 2, 3, 4, 5]
}
```

### 2. 사람 + 프롬프트로 드레스 착용

```http
POST /human_plus_dress
Content-Type: multipart/form-data

image1: [사람 이미지 파일]
positive_prompt: "white wedding dress"
negative_prompt: "dark colors"
```

### 3. 가상 피팅 (Single)

```http
POST /vton_dress
Content-Type: multipart/form-data

image1: [사람 이미지]
image2: [옷 이미지]
```

### 4. 가상 피팅 (Multi - CatVTON)

```http
POST /vton_dress_multi_cat
Content-Type: multipart/form-data

myPic: [사람 이미지]
ImageFileIndexArray: "0,1,2,3,4,5"  # 선택된 드레스 인덱스
```

### 5. 유사 드레스 검색

```http
POST /get_similar_dresses
Content-Type: multipart/form-data

imageData: "data:image/jpeg;base64,..."
```

### 6. 큐 상태 확인

```http
GET /queue_size
```

**응답:**

```json
{
	"queue_remaining": 2,
	"queue_pending": 1
}
```

### 7. ComfyUI 종료

```http
GET /terminate
```

## 🔄 워크플로우

### 1. human_plus_dress.json (사람 + 프롬프트 → 드레스 착용)

![스크린샷 2024-12-27 041251](https://github.com/user-attachments/assets/9f768ae1-884f-4c85-83d4-60d0809a1df1)

**특징:**

- 사람 이미지와 텍스트 프롬프트 입력
- KSampler를 통한 이미지 생성
- Beautiful Realistic Asians 체크포인트 활용

### 2. new_dress.json (프롬프트 → 드레스 입은 사람)

![스크린샷 2024-12-27 041740](https://github.com/user-attachments/assets/68befeeb-943d-4240-83f6-47368b8aa2e0)

**특징:**

- 순수 텍스트 프롬프트만으로 생성
- 랜덤 시드를 통한 다양성 확보
- 웨딩드레스에 최적화된 프롬프트 템플릿

### 3. SanAI_CatVTON.json (사람 + 옷 이미지 → 가상 피팅)

![스크린샷 2024-12-27 043612](https://github.com/user-attachments/assets/d55b0944-639f-41b8-a2c4-d34fa7c9fb65)

**특징:**

- CatVTON 모델 기반
- 빠른 처리 속도
- 옷의 디테일 보존

### 4. yisol IDM-VTON (고품질 가상 피팅)

![스크린샷 2024-12-27 043933](https://github.com/user-attachments/assets/61361e17-4449-4e6c-957d-b5d0318b54f2)

**특징:**

- IDM-VTON diffusers 모델
- 최고 품질의 착용 결과
- 신체 포즈 보존

## 📁 프로젝트 구조

```
VTON_ComfyUI_API/
├── __init__.py                 # Flask 메인 서버
├── clean_memory.py             # 메모리 정리 유틸리티
├── README.md
│
├── api/                        # ComfyUI API 통신
│   ├── api_helpers.py          # 이미지 생성 헬퍼 함수
│   ├── open_websocket.py       # WebSocket 연결
│   └── websocket_api.py        # ComfyUI API 래퍼
│
├── similarity/                 # 이미지 유사도 검색
│   ├── getSimilarity.py        # VGG16 기반 유사도 계산
│   └── removeBackground/       # 배경 제거 모듈
│       ├── removeBG.py
│       └── removeBG thread.py
│
├── static/                     # 프론트엔드 리소스
│   ├── css/
│   │   ├── faker.css
│   │   └── style.css
│   ├── js/
│   │   ├── faker.js
│   │   ├── fetchQueueSize.js   # 큐 상태 모니터링
│   │   ├── load_three_gltf.js
│   │   └── script-241104085800.js
│   └── models/
│
├── templates/                  # HTML 템플릿
│   ├── faker.html
│   ├── index.html
│   └── loading.html
│
├── utils/                      # 유틸리티 함수
│   ├── actions/
│   │   ├── getImagePath.py    # 이미지 경로 관리
│   │   ├── human_plus_dress.py # 사람+프롬프트 워크플로우
│   │   ├── load_workflow.py    # 워크플로우 로더
│   │   ├── new_dress.py        # 순수 생성 워크플로우
│   │   ├── vton_dress.py       # IDM-VTON 워크플로우
│   │   └── vton_dress_cat.py   # CatVTON 워크플로우
│   └── helpers/
│       └── randomize_seed.py   # 랜덤 시드 생성
│
└── workflows/                  # ComfyUI 워크플로우 JSON
    ├── cat_vton_api.json
    ├── human_plus_dress_api.json
    ├── new_dress_api.json
    ├── vton_api.json
    └── 일반 워크플로우/
        ├── asdaasdasd.json
        ├── human_plus_dress.json
        ├── multi.json
        ├── new_dress.json
        ├── SanAI_CatVTON.json
        └── vton-yisol.json
```

## 📸 스크린샷

### 웹 인터페이스

<img src="https://github.com/user-attachments/assets/043635ee-cad9-461c-8f99-a4387bbb3734" alt="Image" width="44%" />
<img src="https://github.com/user-attachments/assets/e54554a5-8b09-49d0-a714-9b279dd4a4e4" alt="Image" width="32%" />
<img src="https://github.com/user-attachments/assets/1879ad0a-6349-4d56-b0b4-5bbe717214ea" alt="Image" width="70%" />

## 🔧 기술 스택

### Backend

- **Flask**: 웹 서버 프레임워크
- **ComfyUI**: 이미지 생성 백엔드
- **NumPy/SciPy**: 수치 연산 및 유사도 계산
- **TensorFlow/Keras**: VGG16 모델 실행
- **Pillow**: 이미지 처리
- **psutil**: 프로세스 관리
- **CUDA (Numba)**: GPU 메모리 관리

### AI Models

- **IDM-VTON**: 가상 피팅 모델
- **CatVTON**: 빠른 가상 피팅 모델
- **VGG16**: 이미지 특징 추출
- **Beautiful Realistic Asians**: SD 체크포인트

### Frontend

- HTML5/CSS3/JavaScript
- Three.js (3D 모델 렌더링)
- WebSocket (실시간 통신)

## 🎯 주요 기능 설명

### 메모리 관리

```python
def clear_memory():
    clear_comfy_cache(SERVER_ADDRESS, unload_models=True, free_memory=True)
    device = cuda.get_current_device()
    device.reset()
```

각 이미지 생성 후 자동으로 CUDA 메모리와 ComfyUI 캐시를 정리하여 안정적인 운영을 보장합니다.

### 이미지 유사도 검색

VGG16의 fc2 레이어 출력을 특징 벡터로 사용하며, 코사인 유사도로 유사 이미지를 검색합니다:

```python
similarity = 1 - distance.cosine(target_features, features)
```

### WebSocket 기반 진행 상태 추적

ComfyUI와 WebSocket으로 통신하여 실시간으로 이미지 생성 진행 상태를 모니터링합니다.

## 🐛 트러블슈팅

### ComfyUI가 자동 시작되지 않는 경우

`__init__.py`에서 ComfyUI 경로 확인:

```python
COMFYUI_BASE_DIRECTORY = "E:\\ComfyUI_0.2.2\\ComfyUI"
```

### CUDA Out of Memory 오류

1. 배치 크기 줄이기
2. 이미지 해상도 낮추기
3. `clear_memory()` 함수가 정상 작동하는지 확인

### 워크플로우 실행 실패

1. 필수 커스텀 노드가 모두 설치되었는지 확인
2. 모델 파일이 올바른 경로에 있는지 확인
3. ComfyUI 로그 확인 (`comfyui.log`)

## 📚 참고 자료

### ComfyUI API

- [9elements/comfyui-api](https://github.com/9elements/comfyui-api)

### IDM-VTON CustomNode

- [neuralninja22/IDM-VTON](https://github.com/neuralninja22/IDM-VTON)

### CatVTON CustomNode

- [Zheng-Chong/CatVTON](https://github.com/Zheng-Chong/CatVTON)
- [chflame163/ComfyUI_CatVTON_Wrapper](https://github.com/chflame163/ComfyUI_CatVTON_Wrapper)

### Frontend

- [Roniebin/Armigo\_](https://github.com/Roniebin/Armigo_)

## 📄 라이선스

이 프로젝트는 개인 및 교육 목적으로 사용할 수 있습니다.

## 🔐 환경 설정

### ComfyUI 버전

- ComfyUI Revision: 2652 [0c7c98a9] | Released on '2024-09-05'

### 필수 커스텀 노드 로딩 시간

```
0.0 seconds: websocket_image_save.py
0.0 seconds: ComfyUI-Custom-Scripts
0.0 seconds: ComfyUI_essentials
0.0 seconds: rgthree-comfy
0.1 seconds: comfyui_controlnet_aux
0.1 seconds: comfyui_segment_anything
0.3 seconds: ComfyUI_CatVTON_Wrapper
0.3 seconds: ComfyUI-Manager
0.6 seconds: ComfyUI-tbox
1.1 seconds: ComfyUI-Impact-Pack
2.1 seconds: IDM-VTON
3.6 seconds: ComfyUI_LayerStyle
```
