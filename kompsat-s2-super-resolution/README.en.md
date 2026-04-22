# KOMPSAT-3A × Sentinel-2 Super-Resolution

> Deep learning-based **satellite image super-resolution** pipeline using KOMPSAT-3A (K3A) high-resolution imagery and Sentinel-2 (S2) mid-resolution imagery.

The model uses Prithvi-EO as a foundation backbone with LoRA fine-tuning, coupled with a UNet parallel edge encoder and a 4-term loss function (L1 + SAM + Edge + SSIM) to super-resolve S2 imagery to K3A-level spatial resolution.

Undergraduate capstone project at the University of Seoul, Department of Spatial Information Engineering — submitted to a domestic Korean conference.

---

## 📌 Overview

**Goal:** Super-resolve Sentinel-2 (10 m GSD) imagery to KOMPSAT-3A quality (≈2 m GSD).

**Key contributions:**
- End-to-end preprocessing pipeline for K3A–S2 pair construction (RPC orthorectification → Phase Correlation co-registration → chip generation)
- Realistic S2 simulation via MTF inverse filtering and IR-MAD radiometric calibration
- **LoRA fine-tuning of Prithvi-EO-2.0** with a UNet parallel edge encoder
- 4-term loss function balancing spatial, spectral, structural, and edge fidelity

---

## 🗂️ Repository Structure

```
.
├── notebooks/
│   ├── 01_preprocessing_k3a_ortho.ipynb    # K3A orthorectification + S2 download
│   ├── 02_chip_generation.ipynb            # Phase Correlation + chip extraction
│   └── 03_sr_training_and_eval.ipynb       # Prithvi + UNet training & eval
├── configs/
│   ├── optimal_mtf.json                    # Per-scene optimal MTF coefficients
│   └── ortho_result_map.example.json       # Example scene-to-ortho mapping
├── docs/
│   └── pipeline.md                         # Detailed pipeline documentation
├── requirements.txt
├── LICENSE
└── README.md
```

---

## 🔧 Pipeline

### Stage 1 — Preprocessing

```
K3A L1R (RPC + GLO-30 DEM) ──▶ Orthorectification
Sentinel-2 L1C (CDSE API)   ──▶ ±10d / ±30d search per K3A scene
                                 │
                                 ▼
                      ortho_result_map.json
```

### Stage 2 — Chip Generation

```
[Per scene]
Load K3A ortho R-band
Upsample S2 R-band to K3A grid (2.5 m)
Phase Correlation → (dy_px, dx_px) offset estimation
    │
    ▼
Shift S2 stack transform origin → s2_shifted.tif
(no pixel resampling — transform only)
    │
    ▼
[Per chip]
Sample chips from shifted S2 (shared origin with K3A)
```

### Stage 3 — Training & Evaluation

- **Backbone:** Prithvi-EO-2.0 (with LoRA fine-tuning)
- **Decoder:** UNet + parallel edge encoder (Sobel-based)
- **Loss:** `L_total = λ₁·L1 + λ₂·SAM + λ₃·Edge + λ₄·SSIM`
- **Metrics:** PSNR, SSIM, SAM, Edge-PSNR

---

## 🚀 Getting Started

### Environment

- Google Colab (A100 recommended) or local GPU (16GB+ VRAM)
- Python 3.10+, PyTorch 2.x, CUDA 11.8+

### Installation

```bash
pip install -r requirements.txt
```

### Data

**No raw satellite imagery is included in this repository.** Obtain data from the following sources:

| Data | Source | Notes |
|---|---|---|
| KOMPSAT-3A L1R | [AI Hub](https://www.aihub.or.kr/) or [KARI](https://www.kari.re.kr/) | RPC files required |
| Sentinel-2 L1C | [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/) | Auto-downloaded in notebook |
| GLO-30 DEM | Copernicus (auto-downloaded) | For orthorectification |

### Execution Order

1. Run `01_preprocessing_k3a_ortho.ipynb` → generates `ortho_result_map.json`
2. Run `02_chip_generation.ipynb` → produces `dataset/{scene_id}/{chip_id}_{K3A,S2}.tif`
3. Run `03_sr_training_and_eval.ipynb` → training and evaluation

---

## 📊 Key Parameters

| Parameter | Value |
|---|---|
| K3A resolution | 2.5 m (4× upsampling target) |
| S2 resolution | 10 m (original) |
| Chip size | 256 × 256 (at K3A grid) |
| Bands | R, G, B, NIR (4 bands) |
| Mean optimal MTF | 0.673 (see [configs/optimal_mtf.json](configs/optimal_mtf.json)) |

---

## 📝 License

Released under the MIT License. See [LICENSE](LICENSE).

> ⚠️ The **code** in this repository is MIT-licensed and freely usable. However, raw KOMPSAT-3A and Sentinel-2 imagery are subject to their respective providers' terms of use (KARI, ESA).

---

## 🔗 Related Resources

- [Prithvi-EO-2.0 (NASA × IBM)](https://huggingface.co/ibm-nasa-geospatial/Prithvi-EO-2.0-300M)
- [TerraTorch](https://github.com/IBM/terratorch)
- [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/)

---

**한국어 버전:** [README.md](README.md)
