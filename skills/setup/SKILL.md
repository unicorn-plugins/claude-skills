---
name: setup
description: 플러그인 환경 설정 마법사
user-invocable: true
---

# Setup

[SETUP 스킬 활성화]

## 목표

claude-skills 플러그인 사용에 필요한 시스템 도구, Python 패키지, npm 패키지를 설치하고 플러그인을 활성화함.

## 활성화 조건

사용자가 `/claude-skills:setup` 호출 시 또는 "설정", "설치", "환경 구성" 키워드 감지 시.

## 워크플로우

### Step 1: 환경 탐지 → ulw

- OS 감지 (Windows, macOS, Linux)
- 기존 도구 설치 여부 확인 (`gateway/install.yaml` 기준)
- Python 버전, npm 버전 확인

이 단계는 `ulw` 매직 키워드를 활용하여 수행.

### Step 2: 시스템 도구 설치 → /oh-my-claudecode:ralph

`gateway/install.yaml`의 `system_tools` 섹션 기준으로 시스템 도구를 자동 설치 시도.

| 도구 | 설명 | Windows | macOS | Linux (apt) | Linux (dnf) |
|------|------|---------|-------|-------------|-------------|
| LibreOffice | Office 문서 렌더링, 변환, 매크로 | `choco install libreoffice-fresh -y` | `brew install --cask libreoffice` | `sudo apt install -y libreoffice` | `sudo dnf install -y libreoffice` |
| pandoc | DOCX 읽기/변환 | `choco install pandoc -y` | `brew install pandoc` | `sudo apt install -y pandoc` | `sudo dnf install -y pandoc` |
| poppler-utils | PDF → 이미지 변환 | `choco install poppler -y` | `brew install poppler` | `sudo apt install -y poppler-utils` | `sudo dnf install -y poppler-utils` |
| qpdf | PDF 병합/분할 | `choco install qpdf -y` | `brew install qpdf` | `sudo apt install -y qpdf` | `sudo dnf install -y qpdf` |

**OS별 패키지 매니저 감지 로직:**
1. Windows: choco 또는 winget 존재 확인
2. macOS: brew 존재 확인
3. Linux: apt 또는 dnf 존재 확인

**실패 시 폴백**: 수동 설치 URL 안내 (사용자 상호작용 섹션 참조)

이 단계는 `/oh-my-claudecode:ralph`를 활용하여 완료 보장.

### Step 3: Python 환경 설정 → ulw

`pip install` 명령으로 Python 패키지 설치:

| 패키지 그룹 | 패키지 목록 | 용도 |
|-----------|-----------|------|
| Office 공통 | lxml, defusedxml | XML 검증, 안전 파싱 |
| PDF | pypdf, pdfplumber, reportlab, pypdfium2, Pillow | PDF 처리 전반 |
| Excel | openpyxl, pandas | Excel 읽기/쓰기 |

```bash
pip install lxml defusedxml pypdf pdfplumber reportlab pypdfium2 openpyxl pandas Pillow
```

이 단계는 `ulw` 매직 키워드를 활용하여 수행.

### Step 4: npm 패키지 설치 → ulw

`npm install` 명령으로 npm 패키지 설치:

| 패키지 | 용도 |
|--------|------|
| docx | DOCX 신규 생성 |
| pptxgenjs | PPTX 신규 생성 |
| react-icons | 아이콘 SVG 추출 |
| sharp | 이미지 처리/리사이즈 |

```bash
npm install docx pptxgenjs react-icons sharp
```

이 단계는 `ulw` 매직 키워드를 활용하여 수행.

### Step 5: 검증 → ulw

각 도구의 `check` 명령 실행하여 설치 성공 여부 확인:

```bash
soffice --version
pandoc --version
pdftoppm -v
qpdf --version
python -c "import lxml; import defusedxml"
python -c "import pypdf; import pdfplumber; import reportlab"
python -c "import openpyxl; import pandas"
node -e "require('docx')"
node -e "require('pptxgenjs')"
```

실패한 항목은 사용자에게 수동 설치 안내.

이 단계는 `ulw` 매직 키워드를 활용하여 수행.

### Step 6: 플러그인 활성화 → ulw

사용자에게 플러그인 적용 범위를 질문 (사용자 상호작용 섹션 참조):

- **모든 프로젝트**: `~/.claude/CLAUDE.md`에 라우팅 테이블 추가
- **이 프로젝트만**: `./CLAUDE.md`에 라우팅 테이블 추가

**추가 내용 예시:**

```markdown
# claude-skills 플러그인

## 사용 가능한 명령

| 명령 | 설명 |
|------|------|
| `/claude-skills:setup` | 환경 설정 |
| `/claude-skills:help` | 사용 안내 |
| `/claude-skills:docx` | Word 문서 처리 |
| `/claude-skills:pptx` | PowerPoint 처리 |
| `/claude-skills:xlsx` | Excel 처리 |
| `/claude-skills:pdf` | PDF 처리 |
| `/claude-skills:frontend-design` | 프론트엔드 디자인 가이드 |
| `/claude-skills:product-self-knowledge` | Anthropic 제품 지식 |
| `/claude-skills:add-ext-skill` | 확장 스킬 추가 |
| `/claude-skills:remove-ext-skill` | 확장 스킬 제거 |

## 자동 라우팅

다음과 같은 요청은 자동으로 claude-skills 플러그인이 처리합니다:
- ".docx", "Word", "워드", "문서 작성" → /claude-skills:docx
- ".pptx", "PowerPoint", "프리젠테이션" → /claude-skills:pptx
- ".xlsx", "Excel", "스프레드시트" → /claude-skills:xlsx
- ".pdf", "PDF", "폼 작성" → /claude-skills:pdf
- "UI", "디자인", "컴포넌트" → /claude-skills:frontend-design
- "Claude API", "Anthropic 제품" → /claude-skills:product-self-knowledge
```

이 단계는 `ulw` 매직 키워드를 활용하여 수행.

### Step 7: 완료 보고

설치 결과 요약 출력:

- 설치 성공 항목 목록
- 실패 항목 및 수동 설치 안내
- 플러그인 활성화 위치 (`~/.claude/CLAUDE.md` 또는 `./CLAUDE.md`)

## 사용자 상호작용

Setup 스킬은 `AskUserQuestion` 도구를 사용하여 사용자에게 선택지를 제공함.

### Step 2 실패 시: 수동 설치 안내

**질문**: "일부 시스템 도구 자동 설치에 실패했습니다. 수동 설치 방법을 안내받으시겠습니까?"

**선택지**:
- "수동 설치 URL 안내"
- "건너뛰기"

**수동 설치 URL** (안내 시 제공):
- LibreOffice: https://www.libreoffice.org/download/
- pandoc: https://pandoc.org/installing.html
- poppler-utils: https://poppler.freedesktop.org/
- qpdf: https://qpdf.sourceforge.io/

### Step 6: 플러그인 적용 범위

**질문**: "플러그인을 어디에 활성화하시겠습니까?"

**선택지**:
- "모든 프로젝트 (권장)" → `~/.claude/CLAUDE.md`에 추가
- "이 프로젝트만" → `./CLAUDE.md`에 추가

## 상태 관리

상태 파일 미사용 (단건 작업). 각 Step 완료 여부는 검증 명령으로 확인.

## 문제 해결

| 문제 | 해결 방법 |
|------|----------|
| choco/brew 미설치 | 수동 설치 URL 안내 + 패키지 매니저 설치 가이드 링크 |
| pip 명령 실패 | Python 가상환경 활성화 여부 확인 + `python -m pip install` 시도 |
| npm 명령 실패 | Node.js 설치 여부 확인 + 최신 버전 설치 안내 |
| LibreOffice 실행 실패 | PATH 환경변수 확인 + 재부팅 권장 |

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | `gateway/install.yaml`을 데이터 소스로 사용하여 설치 항목 결정 |
| 2 | 시스템 도구 설치 실패 시 반드시 사용자에게 수동 안내 제공 |
| 3 | 모든 Step에 스킬 부스팅 명시 (ralph 또는 ulw) |
| 4 | 검증(Step 5) 없이 완료 보고하지 않음 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 사용자 확인 없이 시스템 명령 실행 금지 (sudo 등) |
| 2 | 검증 실패한 항목을 성공으로 보고하지 않음 |
| 3 | CLAUDE.md 기존 내용을 덮어쓰지 않고 추가만 수행 |

## 검증 체크리스트

- [ ] frontmatter에 name, description, user-invocable: true 포함
- [ ] `## 사용자 상호작용` 섹션 포함 (AskUserQuestion 기반 분기)
- [ ] 모든 Step에 스킬 부스팅 명시 (ralph 또는 ulw)
- [ ] install.yaml 참조 로직 포함
- [ ] OS별 패키지 매니저 전략 명시
- [ ] 검증(Step 5) 포함
- [ ] MUST 규칙, MUST NOT 규칙 섹션 포함
