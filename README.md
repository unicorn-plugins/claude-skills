# claude-skills

[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blue)](https://claude.com)
[![DMAP](https://img.shields.io/badge/DMAP-v1.0-green)](https://github.com/unicorn-plugins/dmap)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Anthropic 공식 스킬 6종을 DMAP 플러그인으로 통합 제공하는 Claude Code 확장 플러그인

## 개요

Anthropic이 공식 제공하는 6가지 전문 스킬(DOCX, PPTX, XLSX, PDF, Frontend Design, Product Self-Knowledge)을 DMAP 플러그인 형식으로 통합하여 제공. 슬래시 명령과 자동 라우팅으로 Office 문서, PDF, UI 디자인, Anthropic 제품 지식을 활용 가능.

## 기능

### DOCX (Word 문서)
- **생성**: docx-js로 새 문서 작성
- **편집**: Office Open XML 직접 수정 (스타일, 표, 이미지)
- **읽기**: pandoc으로 마크다운 변환 및 내용 추출

### PPTX (PowerPoint)
- **생성**: PptxGenJS로 프레젠테이션 작성
- **편집**: Office Open XML 직접 수정 (슬라이드, 레이아웃, 도형)

### XLSX (Excel)
- **생성/편집**: openpyxl로 스프레드시트 조작
- **분석**: pandas로 데이터 분석 및 시각화
- **수식 재계산**: LibreOffice로 수식 업데이트

### PDF
- **읽기/추출**: pypdf, pdfplumber로 텍스트/이미지 추출
- **생성**: reportlab으로 새 PDF 작성
- **폼 작성**: pypdf로 인터랙티브 폼 편집
- **변환**: LibreOffice로 Office 문서 ↔ PDF 변환

### Frontend Design
- UI/UX 디자인 원칙 가이드
- 반응형 웹 디자인 패턴
- 접근성(a11y) 모범 사례
- 컴포넌트 디자인 시스템

### Product Self-Knowledge
- Claude API 문서 참조
- Claude Code 기능 설명
- Anthropic 제품 정보 라우팅

## 설치

```bash
# 1. 플러그인 클론
git clone https://github.com/unicorn-plugins/claude-skills.git

# 2. 환경 설정 (자동)
/claude-skills:setup
```

## 요구사항

### 시스템 도구
- **필수**: LibreOffice, pandoc
- **선택**: poppler-utils (PDF → 이미지), qpdf (PDF 병합/분할)

### Python 패키지
```bash
pip install lxml defusedxml pypdf pdfplumber reportlab pypdfium2 openpyxl pandas Pillow
```

### npm 패키지
```bash
npm install docx pptxgenjs react-icons sharp
```

## 사용법

### 슬래시 명령

| 명령 | 설명 |
|------|------|
| `/claude-skills:setup` | 환경 설정 (시스템 도구, Python, npm 설치) |
| `/claude-skills:help` | 사용 안내 |
| `/claude-skills:add-ext-skill` | 확장 스킬 추가 |
| `/claude-skills:remove-ext-skill` | 확장 스킬 제거 |
| `/claude-skills:docx` | Word 문서 생성, 편집, 읽기 |
| `/claude-skills:pptx` | PowerPoint 생성, 편집 |
| `/claude-skills:xlsx` | Excel 생성, 편집, 분석 |
| `/claude-skills:pdf` | PDF 읽기, 생성, 편집, 폼 작성 |
| `/claude-skills:frontend-design` | 프론트엔드 UI/UX 디자인 가이드 |
| `/claude-skills:product-self-knowledge` | Anthropic 제품 지식 (Claude API, Claude Code) |

### 자동 라우팅 예시

다음과 같은 요청은 자동으로 해당 스킬 실행:

- **DOCX**: "Word 문서 만들어줘", "이 DOCX 파일 편집해줘"
- **PPTX**: "프레젠테이션 만들어줘", "슬라이드 10개 추가해줘"
- **XLSX**: "Excel 파일 생성해줘", "데이터 분석해줘"
- **PDF**: "PDF 읽어줘", "이 문서를 PDF로 변환해줘"
- **Frontend Design**: "이 UI 디자인 리뷰해줘", "반응형 레이아웃 가이드"
- **Product**: "Claude API 사용법 알려줘", "Claude Code 기능은?"

## 에이전트 구성

### office-editor (MEDIUM)
- **역할**: DOCX/PPTX/XLSX Office XML 편집 전문가
- **도구**: lxml, defusedxml, openpyxl

### pdf-handler (MEDIUM)
- **역할**: PDF 처리 전문가
- **도구**: pypdf, pdfplumber, reportlab, pypdfium2

## 디렉토리 구조

```
claude-skills/
├── .claude-plugin/
│   ├── plugin.json              # 플러그인 매니페스트
│   └── marketplace.json         # 마켓플레이스 메타데이터
├── skills/                      # 스킬 정의 (11종)
│   ├── core/SKILL.md            # 의도 판별 + 라우팅
│   ├── setup/SKILL.md           # 환경 설정
│   ├── help/SKILL.md            # 사용 안내
│   ├── add-ext-skill/SKILL.md   # 확장 스킬 추가
│   ├── remove-ext-skill/SKILL.md # 확장 스킬 제거
│   ├── docx/SKILL.md            # Word 문서 처리
│   ├── pptx/SKILL.md            # PowerPoint 처리
│   ├── xlsx/SKILL.md            # Excel 처리
│   ├── pdf/SKILL.md             # PDF 처리
│   ├── frontend-design/SKILL.md # 프론트엔드 디자인
│   └── product-self-knowledge/SKILL.md # 제품 지식
├── agents/                      # 에이전트 정의 (2종)
│   ├── office-editor/           # DOCX/PPTX/XLSX 편집
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   ├── tools.yaml
│   │   └── references/ (7파일)
│   └── pdf-handler/             # PDF 처리
│       ├── AGENT.md
│       ├── agentcard.yaml
│       ├── tools.yaml
│       └── references/ (2파일)
├── gateway/                     # 런타임 설정
│   ├── install.yaml             # 설치 매니페스트
│   ├── runtime-mapping.yaml     # 추상→구체 매핑
│   └── tools/                   # 커스텀 도구
│       ├── office/              # 공유 Office 모듈
│       ├── docx/                # DOCX 전용 도구
│       ├── pptx/                # PPTX 전용 도구
│       ├── xlsx/                # XLSX 전용 도구
│       └── pdf/                 # PDF 전용 도구
├── commands/                    # 슬래시 명령 (10개)
└── README.md
```

## 라이선스

MIT License - [LICENSE](LICENSE) 파일 참조
