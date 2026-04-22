# 전처리 파이프라인 상세

## 1. K3A 정사보정 (Orthorectification)

### 입력
- K3A L1R 영상 (`*_PS.tif`) + RPC 계수 파일 (`*_rpc.txt`)
- GLO-30 Copernicus DEM (자동 다운로드)

### 처리
GDAL `gdalwarp`의 RPC 옵션과 DEM을 사용한 지형 보정:

```
gdalwarp -rpc -to RPC_DEM=<GLO30_DEM> \
         -t_srs EPSG:5179 -tr 2.5 2.5 \
         -r cubic \
         <input_L1R>.tif <output_ortho>.tif
```

- 타겟 좌표계: **EPSG:5179** (Korea 2000 / Unified CS)
- 리샘플링: Cubic
- 출력 해상도: 2.5 m

### 출력
- `k3a_ortho/{scene_id}_ortho.tif`
- `ortho_result_map.json` (씬 ID → 파일 경로 매핑)

---

## 2. Sentinel-2 매칭

### CDSE API 기반 자동 탐색
- 1차: K3A 촬영일 ±10일
- 2차: 실패 시 ±30일
- 필터: cloudCover < 20%, Tile ID 일치

### Level-1C → Level-2A 우선순위
- L2A가 있으면 L2A 사용 (대기보정 완료)
- 없으면 L1C만 사용

### 밴드 스택
R(B04), G(B03), B(B02), NIR(B08) 4밴드를 하나의 GeoTIFF로 스택.

---

## 3. Phase Correlation 공동 정합

### 핵심 아이디어
K3A 정사보정 후에도 S2와의 기하 오차가 0.5~3 픽셀 수준으로 잔존.  
**픽셀값을 재보간하지 않고 transform origin만 이동**하여 정렬.

### 순서
```
1. K3A ortho R밴드 로드 (2.5 m)
2. S2 R밴드를 K3A 격자로 업샘플링 (10 m → 2.5 m)
3. skimage.registration.phase_cross_correlation 실행
4. 서브픽셀 단위 (dy, dx) 추정
5. S2 스택의 rasterio transform에서 origin만 이동
6. s2_shifted.tif 저장 (픽셀값은 원본 그대로)
```

### 장점
- 중첩 재보간으로 인한 화질 저하 방지
- K3A/S2가 **같은 origin**을 공유하므로 칩 샘플링 시 완벽한 정렬

---

## 4. 칩 생성

### 파라미터
| 항목 | 값 |
|---|---|
| 칩 크기 (K3A) | 256 × 256 px = 640 × 640 m |
| 칩 크기 (S2) | 64 × 64 px (다운샘플 후 재계산) |
| 스트라이드 | 128 px (50% 오버랩) |
| NoData 비율 한계 | 5% |

### 저장 구조
```
dataset/
  {scene_id}/
    chip_0001_K3A.tif
    chip_0001_S2.tif
    chip_0002_K3A.tif
    chip_0002_S2.tif
    ...
```

---

## 5. MTF 역필터 + IR-MAD (방사보정)

S2의 PSF를 고려한 **현실적인 저해상도 시뮬레이션**을 위한 전처리.

### MTF 역필터
- 가우시안 커널의 sigma를 씬별로 최적화
- `configs/optimal_mtf.json`에 저장된 씬별 MTF 값 사용
- 평균 MTF = 0.673

### IR-MAD (Iteratively Re-weighted Multivariate Alteration Detection)
- K3A와 S2의 반사율 분포 차이를 보정
- 변화가 없는 픽셀만으로 선형 회귀 → 분광 보정

> 관련 구현은 별도 노트북에서 다뤄지며, 본 저장소 노트북은 이미 보정된 값을 입력으로 받음.
