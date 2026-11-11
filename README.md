
# LineGate AOI

> Multi-View AI Quality Inspection for SMT First Boards

LineGate AOI는 **SMT 초품에 장착된 R0805 부품을 검사하는 AI 품질 검사 PoC**입니다. 상단 이미지(V1)로 부품의 실장 상태를, 좌우 경사 이미지(V2·V2.1)로 납땜 상태를 검사합니다. 세 시점의 결과를 종합해 RELEASE / HOLD / REVIEW를 권고하고, 작업자가 판정 근거를 확인해 최종 조치를 선택할 수 있도록 지원합니다.

본 프로젝트는 SolDef_AI의 Multi-view PCB 이미지 중 R0805 데이터를 활용합니다.

V1은 부품 전체의 위치 및 정렬 상태를 Bounding BOX 기반 Object Detection으로 검사하고, 
V2·V2.1의 양방향 경사 View에서는 납땜 접합부의 형상과 결함 유형을 Polygon 기반 Instance Segmentation 으로 검사하며, V3는 본 모델 학습 범위에서 제외했습니다.

## Demo
https://github-production-user-asset-6210df.s3.amazonaws.com/312392807/633385155-4c73f010-b160-40eb-bdb0-b33a755133c0.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20260809%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20260809T103050Z&X-Amz-Expires=300&X-Amz-Signature=3f72cf62f6d5a96417f4afda1645cf141a0fd85c1b1d3b6dad627106149ec062&X-Amz-SignedHeaders=host&response-content-type=video%2Fmp4

> V1 / V2 / V2.1 이미지를 입력한 뒤,
> AI 검사 → View별 판정 근거 확인 → RELEASE / HOLD / REVIEW 권고 →
> 작업자 최종 검토 및 승인 순으로 진행되는 전체 흐름을 시연합니다.
>

## Final Models

### V1 — RF-DETR Small Object Detection

V1은 초기에는 Polygon 기반 Instance Segmentation으로 학습했으나,
부품 외곽의 정밀 분할보다 **부품 단위의 정상/이상 판별**이 핵심 목적이라는 점을 반영해
최종적으로 Bounding Box 기반 Object Detection으로 전환했습니다.

최종 V1 학습 데이터는 다음과 같습니다.

| Class | Annotation |
|---|---:|
| `good_BBox` | 129 |
| `no_good_BBox` | 139 |
| **Total** | **268** |

최종 모델은 **RF-DETR Small**이며, 별도 Test Set 27장 기준 성능은 다음과 같습니다.

| Metric | Result |
|---|---:|
| Precision | 81.5% |
| Recall | 81.5% |
| F1-score | 81.5% |
| mAP | 0.658 |
| mAP@50 | 0.746 |
| Evaluation Confidence Threshold | 0.56 |

V1 최종 모델 클래스별 성능 지표는 다음과 같습니다.

| Class | Precision | Recall | F1-score | Test Instance |
|---|---:|---:|---:|---:|
| `no_good_BBBox` | 90.9% | 71.4% | 80.0% | 14 |
| `good_BBBox` | 75.0% | 92.3% | 82.8% | 13 |

> V1 모델의 학습과 보고서에서는 good_BBBox와 no_good_BBBox를 공식 클래스명으로 사용합니다.
> 서비스는 Superb AI Deployment 응답에서 클래스명 끝부분(접미사)이 달라져도,
> 판정 로직에 전달하기 전에 각각 good과 no_good으로 통일합니다.

### V2 / V2.1 — YOLO11s-seg Instance Segmentation

V2와 V2.1은 동일한 납땜 상태 클래스를 사용합니다. 두 각도의 이미지를 하나의 데이터셋으로 통합하고, 공통 모델인 **YOLO11s-seg**를 학습했습니다.

#### Dataset

이미지 264장 중 유효성 검토를 거쳐 **260장**을 사용했습니다. 전체 데이터셋에는 **627개의 Polygon 어노테이션**이 포함됩니다.

| Class | Polygon Objects |
|---|---:|
| `good` | 113 |
| `exc_solder` | 238 |
| `poor_solder` | 118 |
| `spike` | 158 |
| **Total** | **627** |

Seed를 42로 고정하고 이미지를 **Train / Validation / Test = 80 / 10 / 10**으로 분할했습니다.

| Split | Images |
|---|---:|
| Train | 208 |
| Validation | 26 |
| Test | 26 |
| **Total** | **260** |

#### Test Results

**전체 Test Set 26장**을 기준으로 평가한 Mask 성능입니다.

| Metric | Result |
|---|---:|
| Mask Precision | 0.810 |
| Mask Recall | 0.820 |
| Mask mAP50 | 0.861 |
| Mask mAP50–95 | 0.689 |

기준 모델인 YOLO11n-seg와 비교해 Mask Precision은 소폭 낮아졌지만, **Mask Recall은 0.640에서 0.820으로 높아졌고** Mask mAP50–95도 개선되었습니다. 납땜 이상을 놓치는 경우를 줄이기 위해 재현율 개선을 우선해 YOLO11s-seg를 최종 모델로 선정했습니다.

#### Results by View

동일한 최종 모델을 사용해 Test Set을 **V2 11장**과 **V2.1 15장**으로 나누어 각각 평가했습니다.

| View | Images | Mask Precision | Mask Recall | Mask mAP50 | Mask mAP50–95 |
|---|---:|---:|---:|---:|---:|
| V2 | 11 | 93.22% | 83.81% | 90.61% | 70.65% |
| V2.1 | 15 | 91.63% | 86.24% | 95.25% | 79.53% |

## Main Features

### 1. Multi-View Inspection

| View | 검사 대상 | 모델 | 화면 표시 |
|---|---|---|---|
| **V1 · Top** | 부품 실장 상태 | RF-DETR Small · Object Detection | Bounding Box + Confidence |
| **V2 · Left / V2.1 · Right** | 좌우 납땜 접합부 | YOLO11s-seg · Instance Segmentation | Polygon Mask + Confidence |

- **V1** — 정상(`good_BBBox`), 실장 이상(`no_good_BBBox`)
- **V2 / V2.1** — 정상(`good`), 과납(`exc_solder`), 납땜 부족(`poor_solder`), 돌기(`spike`)

### 2. AI Decision Support

한 각도에서 결함이 검출돼도 세 각도의 검사를 모두 진행합니다.
결과를 종합해 **`RELEASE`(진행) / `HOLD`(보류) / `REVIEW`(추가 검토)** 중 하나를 권고하고,
각도별 판정 근거와 권장 후속 조치를 표시합니다.

### 3. Human Approval

작업자가 AI 권고와 검사 근거를 확인한 뒤 **권고 승인, 예외 진행 승인, 보류 유지,
재작업 후 재검사, 추가 검사 요청** 중 최종 조치를 선택합니다.
작업자 의견과 변경 사유를 기록하고, 승인 기록을 JSON으로 다운로드할 수 있습니다.


## Inspection Flow

1. 생산 라인 및 배치 번호 입력
2. 동일 부품의 V1 / V2 / V2.1 이미지 등록
3. **AI 검사 시작**
4. RELEASE / HOLD / REVIEW 최종 로트 판정 확인
5. 판정 사유 확인
6. V1 BBox / V2·V2.1 Polygon Mask Overlay 증거 확인
7. 권장 조치 확인
8. 작업자 최종 승인 및 의견 저장
9. 승인 기록 JSON 다운로드

## AI Classes

### V1 — 부품 실장 상태

| Official Class | Service Internal Label | Meaning |
|---|---|---|
| `good_BBBox` | `good` | 부품이 패드에 정상적으로 실장된 상태 |
| `no_good_BBBox` | `no_good` | 부품 실장 상태가 정상 기준에서 벗어난 상태 |

V1은 위치 이상 원인을 세부 클래스로 분리하지 않고,
부품 본체와 패드 사이의 상대적인 위치·정렬 상태를 기준으로 정상/이상을 판별합니다.

### V2 / V2.1 — 납땜 접합부 상태

| Class | Meaning |
|---|---|
| `good` | 허용 가능한 정상 납땜 접합부 |
| `exc_solder` | 납땜량이 과도하게 형성된 상태 |
| `poor_solder` | 납땜량이 부족하거나 접합 상태가 불충분한 상태 |
| `spike` | 납땜부에 뾰족한 돌출이 형성된 상태 |

## Decision Policy

LineGate는 V1만 먼저 판정하고 검사를 중단하지 않습니다.
**V1, V2, V2.1 세 View 검사를 모두 수행한 뒤 결과를 종합**합니다.

| Case | V1 실장 상태 | V2·V2.1 납땜 상태 | Final Decision |
|---|---|---|---|
| 1 | 정상 | 모두 정상 | **RELEASE** |
| 2 | 이상 | 모두 정상 | **HOLD** |
| 3 | 정상 | 하나 이상 이상 | **HOLD** |
| 4 | 이상 | 하나 이상 이상 | **HOLD** |
| 5 | 저신뢰도 / 결과 불확실 | 어느 View든 해당 | **REVIEW** |
| 6 | 필수 View 누락 | 어느 View든 해당 | **REVIEW** |

- **RELEASE**: 전체 3각도 검사 결과가 정상
- **HOLD**: 실장 이상 또는 납땜 이상 검출
- **REVIEW**: 저신뢰도, 판정 불확실 또는 필수 View 누락

AI 결과는 최종 품질 결정을 자동 확정하는 것이 아니라,
작업자가 Bounding Box / Polygon Mask Overlay와 판정 근거를 검토할 수 있도록 지원합니다.

## Recommended Actions

- **V1 위치 이상**
  - 부품 실장 위치
  - 장착 프로그램
  - Setup 재확인

- **poor_solder**
  - 납땜 부족 여부 재검
  - 보강 가능성 검토

- **exc_solder**
  - 과납 및 인접부 접촉 가능성 점검

- **spike**
  - 돌출 형상 및 주변 간섭 가능성 점검


## System Architecture

```text
V1 · 상단 이미지
  └─ 부품 실장 상태 검사
     └─ Superb AI Deployment · RF-DETR Small (Object Detection)
        ├─ good_BBBox · 정상 실장
        └─ no_good_BBox · 실장 이상

V2 · 좌측 경사 이미지 / V2.1 · 우측 경사 이미지
  └─ 좌우 납땜 접합부 검사
     └─ FastAPI · YOLO11s-seg (Instance Segmentation)
        ├─ good · 정상
        ├─ exc_solder · 과납
        ├─ poor_solder · 납땜 부족
        └─ spike · 돌기

V1 / V2 / V2.1 검사 결과
  └─ LineGate 판정 로직으로 종합
     └─ AI 권고 · RELEASE / HOLD / REVIEW
        └─ 작업자가 판정 근거를 검토하고 최종 조치 선택
```

## Project Structure

```text
linegate-aoi/
├─ backend/
│  ├─ main.py
│  ├─ decision_engine.py
│  └─ __init__.py
├─ frontend/
│  ├─ app.py
│  └─ __init__.py
├─ models/
│  └─ best_yolo11s.pt
├─ demo_images/
   ├─ case1_v1-hold_v2-hold_v21-hold/
   └─ case2_v1-release_v2-hold_v21-hold/
├─ .gitignore
├─ requirements.txt
└─ README.md
```

| 경로                                               | 역할                                                                                                                                                                             |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `frontend/app.py`                                | **LineGate AOI 메인 웹 애플리케이션.** Streamlit 기반 UI를 제공하며 V1 Superb AI Deployment 호출, V2/V2.1 FastAPI 호출, 3-View 결과 통합, RELEASE/HOLD/REVIEW 표시, Overlay 시각화 및 Human Approval을 담당합니다. |
| `backend/main.py`                                | **V2/V2.1 납땜 검사용 FastAPI 서버.** 업로드 이미지를 YOLO11s-seg 모델에 전달하고 납땜 결함 추론 결과와 Polygon Mask Overlay를 반환합니다.                                                                         |
| `backend/decision_engine.py`                     | V2/V2.1 추론 결과를 기반으로 클래스 및 Confidence를 해석하고 View 단위 판정에 필요한 규칙을 처리합니다.                                                                                                          |
| `backend/__init__.py`                            | `backend` 디렉터리를 Python 패키지로 인식시키기 위한 파일입니다.                                                                                                                                    |
| `frontend/__init__.py`                           | `frontend` 디렉터리를 Python 패키지로 인식시키기 위한 파일입니다.                                                                                                                                   |
| `models/best_yolo11s.pt`                         | **V2/V2.1 최종 납땜 검사 모델 가중치.** 260장의 통합 V2/V2.1 데이터로 학습한 YOLO11s-seg 모델입니다.                                                                                                      |
| `demo_images/`                                   | LineGate AOI 시연에 사용하는 동일 부품의 V1/V2/V2.1 이미지 세트입니다.                                                                                                                             |
| `demo_images/case1_v1-hold_v2-hold_v21-hold/`    | V1 실장 상태와 V2/V2.1 납땜 검사에서 모두 HOLD 근거가 발생하는 시연 Case입니다.                                                                                                                         |
| `demo_images/case2_v1-release_v2-hold_v21-hold/` | V1은 정상이나 V2/V2.1에서 납땜 결함을 발견하여 최종 HOLD가 되는 Multi-View 검사의 필요성을 보여주는 Case입니다.                                                                                                   |
| `requirements.txt`                               | 프로젝트 실행에 필요한 Python 패키지 목록입니다. Python 3.12 환경을 기준으로 사용합니다.                                                                                                                     |
| `.gitignore`                                     | API Key, Python cache, 실행 로그 및 생성 결과 등 GitHub에 업로드하지 않아야 할 파일을 제외합니다.                                                                                                          |
| `README.md`                                      | 프로젝트 개요, 모델 구성, 성능, 판정 기준 및 실행 방법을 설명하는 문서입니다.                                                                                                                                 |


## Requirements

- Windows 10 / 11
- Anaconda 또는 Miniconda
- **Python 3.12**
- Superb AI Tenant API Key

## Initial Setup

프로젝트 폴더에서 Anaconda Prompt를 실행합니다.

```bat
conda create -n linegate312 python=3.12 -y
conda activate linegate312
python -m pip install -r requirements.txt
```

Python 버전을 확인합니다.

```bat
python --version
```

`Python 3.12.x`가 나오면 정상입니다.

## Superb AI API Key

API Key를 소스코드에 직접 저장하지 않습니다.

Streamlit을 실행할 Prompt에서 다음과 같이 환경변수를 설정합니다.

```bat
set SUPERB_AI_API_KEY=YOUR_API_KEY
```

> 실제 API Key를 GitHub 저장소, README, 스크린샷 등에 업로드하지 마세요.

## Run

현재 LineGate는 **FastAPI 서버와 Streamlit 웹 화면을 동시에 실행**합니다.
Anaconda Prompt 창 2개를 사용합니다.

### Prompt 1 — V2 / V2.1 FastAPI

```bat
cd /d "YOUR_PATH\linegate-aoi-v1.0"
conda activate linegate312
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000
```

이 창은 종료하지 않습니다.

### Prompt 2 — V1 Superb AI + Streamlit

```bat
cd /d "YOUR_PATH\linegate-aoi-v1.0"
conda activate linegate312
set SUPERB_AI_API_KEY=YOUR_API_KEY
python -m streamlit run frontend/app.py
```

브라우저에서 Streamlit 화면이 열리면 실행 완료입니다.


## Demo Data

`demo_images/`에는 로컬 시연에 사용할 수 있는 예시 이미지 세트가 포함되어 있습니다.

## Security

- `SUPERB_AI_API_KEY`는 환경변수로만 사용합니다.
- `.env`, Python cache, 실행 중 생성되는 Overlay 및 로그는 `.gitignore` 대상입니다.
- 실제 API Key를 GitHub 저장소에 커밋하지 마세요.

## Project

**Superb AI × BDAI Vision AI Hackathon — TRACK A 제조**

Team **코알라**
