# KOMPSAT-3A × Sentinel-2 위성영상 초해상화

> KOMPSAT-3A(K3A) 고해상도 영상과 Sentinel-2(S2) 중해상도 영상을 활용한 **딥러닝 기반 위성영상 초해상화(Super-Resolution)** 파이프라인

Prithvi-EO 파운데이션 모델을 백본으로 사용하고, UNet 병렬 엣지 인코더와 4중 손실(L1 + SAM + Edge + SSIM) 구조로 S2 영상을 K3A 수준의 공간해상도로 복원합니다.

서울시립대학교 공간정보공학과 졸업작품 / 국내 학회 논문 투고작.

---

## 📌 프로젝트 개요

**목표:** Sentinel-2의 10 m 해상도 영상을 KOMPSAT-3A의 2 m급 해상도로 초해상화

**핵심 기여점:**
- K3A–S2 쌍 구성을 위한 end-to-end 전처리 파이프라인 (RPC 정사보정 → Phase Correlation 공동 정합 → 칩 생성)
- MTF 역필터 + IR-MAD 기반 방사보정으로 **현실적인 S2 시뮬레이션**
- Prithvi-EO-2.0 백본에 **LoRA 미세조정 + UNet 병렬 엣지 인코더**
- 공간·분광·구조·엣지를 동시에 보존하는 4중 손실 함수

---

## 🗂️ 저장소 구조

```
.
├── notebooks/
│   ├── 01_preprocessing_k3a_ortho.ipynb    # K3A 정사보정 + S2 다운로드
│   ├── 02_chip_generation.ipynb            # Phase Correlation + 칩 생성
│   └── 03_sr_training_and_eval.ipynb       # Prithvi+UNet 학습 + 평가
├── configs/
│   ├── optimal_mtf.json                    # 씬별 최적 MTF 계수
│   └── ortho_result_map.example.json       # 씬 ID → ortho 파일 매핑 예시
├── docs/
│   └── pipeline.md                         # 전처리 파이프라인 상세 문서
├── requirements.txt
├── LICENSE
├── README.md
└── README.en.md
```

---

## 🔧 파이프라인

### 1단계 — 전처리 ([01_preprocessing_k3a_ortho.ipynb](notebooks/01_preprocessing_k3a_ortho.ipynb))

```
K3A L1R (RPC + GLO-30 DEM) ──▶ 정사보정(ortho)
Sentinel-2 L1C (CDSE API)   ──▶ K3A 씬별 ±10일/±30일 탐색 후 다운로드
                                 │
                                 ▼
                     ortho_result_map.json 저장
```

### 2단계 — 칩 생성 ([02_chip_generation.ipynb](notebooks/02_chip_generation.ipynb))

```
[씬 단위]
K3A ortho R밴드 로드
S2 R밴드를 K3A 격자(2.5m)로 업샘플링
Phase Correlation → (dy_px, dx_px) 추정
    │
    ▼
S2 스택 전체 transform origin 이동 → s2_shifted.tif 저장
(픽셀값 재보간 없음 — transform만 수정)
    │
    ▼
[칩 단위]
shift된 S2로 칩 샘플링 (K3A/S2 동일 origin 사용)
```

### 3단계 — 학습 및 평가 ([03_sr_training_and_eval.ipynb](notebooks/03_sr_training_and_eval.ipynb))

- **백본:** Prithvi-EO-2.0 (LoRA 미세조정)
- **디코더:** UNet + 병렬 엣지 인코더(Sobel 기반)
- **손실 함수:** `L_total = λ₁·L1 + λ₂·SAM + λ₃·Edge + λ₄·SSIM`
- **평가 지표:** PSNR, SSIM, SAM, Edge-PSNR

---

## 🚀 실행 방법

### 환경

- Google Colab (A100 권장) 또는 로컬 GPU 환경 (VRAM 16GB+)
- Python 3.10+, PyTorch 2.x, CUDA 11.8+

### 설치

```bash
pip install -r requirements.txt
```

### 데이터 준비

본 저장소에는 **원본 위성영상이 포함되어 있지 않습니다.** 아래 경로에서 직접 수집하세요.

| 데이터 | 출처 | 비고 |
|---|---|---|
| KOMPSAT-3A L1R | [AI Hub — 위성영상 객체판독](https://www.aihub.or.kr/) 또는 [KARI 아리랑 정보센터](https://www.kari.re.kr/) | RPC 파일 포함 필요 |
| Sentinel-2 L1C | [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/) | 노트북 내 API 자동 다운로드 |
| GLO-30 DEM | Copernicus (노트북 내 자동 다운로드) | 정사보정용 |

### 실행 순서

1. `01_preprocessing_k3a_ortho.ipynb` → `ortho_result_map.json` 생성
2. `02_chip_generation.ipynb` → `dataset/{scene_id}/{chip_id}_K3A.tif`, `_S2.tif` 생성
3. `03_sr_training_and_eval.ipynb` → 학습 및 평가

---

## 📊 주요 설정값

| 항목 | 값 |
|---|---|
| K3A 해상도 | 2.5 m (4배 업샘플링 타깃) |
| S2 해상도 | 10 m (원본) |
| 칩 크기 | 256 × 256 (K3A 기준) |
| 밴드 | R, G, B, NIR (4 bands) |
| 최적 MTF 평균 | 0.673 ([configs/optimal_mtf.json](configs/optimal_mtf.json) 참조) |

---

## 📝 라이선스

MIT License. 자세한 내용은 [LICENSE](LICENSE) 참조.

> ⚠️ 본 저장소의 **코드**는 MIT 라이선스로 자유 사용 가능하지만, KOMPSAT-3A 및 Sentinel-2 원본 영상 데이터는 각 공급자(KARI, ESA)의 이용 약관을 따릅니다.

---

## 🔗 관련 자료

- [Prithvi-EO-2.0 (NASA × IBM)](https://huggingface.co/ibm-nasa-geospatial/Prithvi-EO-2.0-300M)
- [TerraTorch](https://github.com/IBM/terratorch)
- [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/)

---

## 🙏 감사의 말

GIST 수자원 AI 연구실에서의 원격탐사 연구 경험이 본 프로젝트의 기반이 되었습니다.

---

**English version:** [README.en.md](README.en.md)
