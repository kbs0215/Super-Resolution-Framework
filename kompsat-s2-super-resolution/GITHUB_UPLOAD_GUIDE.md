# 🚀 GitHub 업로드 가이드

이 가이드는 `kompsat-s2-super-resolution` 폴더를 GitHub에 올리는 두 가지 방법을 설명합니다.

---

## 방법 A. GitHub 웹 UI (가장 쉬움, 추천)

### 1. 레포지토리 생성
1. https://github.com/new 접속
2. 설정:
   - **Repository name**: `kompsat-s2-super-resolution`
   - **Description**: `KOMPSAT-3A × Sentinel-2 Super-Resolution with Prithvi-EO`
   - **Public** 선택
   - ⚠️ **"Add a README file" 체크 해제** (이미 있음)
   - ⚠️ **"Add .gitignore" None** (이미 있음)
   - ⚠️ **"Choose a license" None** (이미 있음)
3. **Create repository** 클릭

### 2. 파일 업로드
1. 방금 만든 레포 페이지에서 **"uploading an existing file"** 링크 클릭
2. zip 압축 풀고 `kompsat-s2-super-resolution` 폴더 **안의 내용물**을 전부 드래그 앤 드롭
   - ⚠️ 폴더 자체가 아니라 폴더 내용물(README.md, LICENSE, notebooks/ 등)을 올리세요
3. 맨 아래 커밋 메시지 입력: `Initial commit: 졸업작품 SR 파이프라인`
4. **Commit changes** 클릭

### 3. 확인
- README.md가 페이지 하단에 보이면 성공
- `notebooks/*.ipynb` 클릭하면 GitHub가 자동으로 렌더링해줍니다

---

## 방법 B. 커맨드라인 `git push` (향후 업데이트 편함)

### 사전 준비
- Git 설치: https://git-scm.com/downloads
- GitHub CLI 또는 Personal Access Token 필요

### 1. GitHub에서 빈 레포 생성
위와 동일하게 빈 레포 생성 (README/gitignore/license 모두 체크 해제)

### 2. 로컬에서 Git 초기화 및 푸시

```bash
# zip 압축 푼 후 해당 폴더로 이동
cd kompsat-s2-super-resolution

# Git 초기화
git init
git branch -M main

# 본인 정보 설정 (처음 한 번만)
git config user.name "멋쟁이"
git config user.email "본인이메일@example.com"

# 파일 추가 & 커밋
git add .
git commit -m "Initial commit: 졸업작품 SR 파이프라인"

# 원격 저장소 연결 (USERNAME을 본인 깃허브 아이디로)
git remote add origin https://github.com/USERNAME/kompsat-s2-super-resolution.git

# 푸시
git push -u origin main
```

### 첫 push 시 인증
- Username: GitHub 아이디
- Password: **GitHub 비밀번호가 아닌 Personal Access Token (PAT)**
- PAT 발급: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token
- 권한은 `repo` 체크

---

## 📝 업로드 후 해야 할 일

### 1. Topics 추가 (검색 노출용)
레포 페이지 우상단 ⚙️ 톱니바퀴 클릭 → Topics에 추가:
```
super-resolution, remote-sensing, kompsat, sentinel-2,
deep-learning, prithvi, pytorch, satellite-imagery
```

### 2. About 섹션 작성
레포 페이지 오른쪽 상단 About 편집:
- Description: `KOMPSAT-3A × Sentinel-2 Super-Resolution using Prithvi-EO foundation model`
- Website: 블로그나 논문 링크 (있으면)

### 3. README 배지 추가 (선택)
README.md 상단에 다음 배지 추가 고려:

```markdown
![License](https://img.shields.io/badge/license-MIT-blue)
![Python](https://img.shields.io/badge/python-3.10+-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-red)
```

### 4. 논문 업로드 (공개 가능하면)
학회 논문 PDF를 `docs/paper.pdf`에 올리고 README에 링크

### 5. 결과 이미지 추가
학습 결과 비교 이미지 (Baseline vs Ours vs GT)를 `docs/figures/`에 넣고 README에 삽입

```markdown
## 결과 예시
![Comparison](docs/figures/comparison.png)
```

---

## ⚠️ 추가 점검 사항

### 올리기 전 반드시 확인
- [ ] 노트북 output 비워져 있음 (이미 처리됨)
- [ ] `ortho_result_map.json` 실제 파일은 없고 `.example.json`만 있음 (이미 처리됨)
- [ ] 개인 드라이브 경로 제거됨 (이미 처리됨)
- [ ] `.gitignore`에 `data/`, `*.tif` 등 제외 규칙 있음 (이미 처리됨)

### 나중에 실수로 데이터 올라간 경우
```bash
# .tif 파일 실수로 커밋한 경우
git rm --cached path/to/file.tif
git commit -m "Remove accidentally committed data"
git push
```

만약 비밀번호나 토큰이 히스토리에 들어갔다면, **즉시 해당 토큰을 무효화**하고 `git filter-repo` 등으로 히스토리 정리 필요.

---

## 💡 다음 단계 추천

1. **GitHub Pages로 결과 공개**
   - Settings → Pages → Source: `main` branch `/docs` folder
   - `docs/index.md`에 프로젝트 데모 페이지 만들기

2. **모델 체크포인트 공개 (선택)**
   - 용량이 크면 GitHub Release에 첨부 (100MB 이하)
   - 더 크면 Hugging Face Hub에 업로드

3. **Citation 파일 추가**
   - `CITATION.cff` 파일을 만들면 GitHub가 "Cite this repository" 버튼을 자동 생성

4. **CI 추가 (선택)**
   - `.github/workflows/`에 노트북 문법 검증 워크플로 추가
