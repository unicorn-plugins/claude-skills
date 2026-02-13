---
name: help
description: 플러그인 사용 안내
user-invocable: true
---

# Help

[HELP 스킬 활성화]

## 목표

claude-skills 플러그인의 사용 가능한 명령, 자동 라우팅 규칙, 주요 기능을 안내함.
런타임 상주 파일(CLAUDE.md)에 라우팅 테이블을 등록하는 대신, 이 스킬이 호출 시에만 토큰을 사용하여 사용자 발견성을 제공함.

## 활성화 조건

사용자가 `/claude-skills:help` 호출 시 또는 "도움말", "뭘 할 수 있어", "사용법" 키워드 감지 시.

## 워크플로우

**중요: 추가적인 파일 탐색이나 에이전트 위임 없이, 아래 내용을 즉시 사용자에게 출력하세요.**

### 사용 가능한 명령

| 명령 | 설명 |
|------|------|
| `/claude-skills:setup` | 플러그인 초기 설정 (시스템 도구 + Python/npm 패키지 설치) |
| `/claude-skills:help` | 사용 안내 (이 메시지) |
| `/claude-skills:add-ext-skill` | 확장 스킬 추가 (외부 SKILL.md 동적 로드) |
| `/claude-skills:remove-ext-skill` | 확장 스킬 제거 |
| `/claude-skills:docx` | Word 문서 생성, 편집, 읽기 |
| `/claude-skills:pptx` | PowerPoint 프리젠테이션 생성, 편집 |
| `/claude-skills:xlsx` | Excel 스프레드시트 생성, 읽기, 수식 재계산 |
| `/claude-skills:pdf` | PDF 읽기, 생성, 편집, 폼 작성, 변환 |
| `/claude-skills:frontend-design` | 프론트엔드 디자인 가이드 (타이포그래피, 색상, 레이아웃 원칙) |
| `/claude-skills:product-self-knowledge` | Anthropic 제품 지식 (Claude API, Claude Code, 가격 안내) |

### 자동 라우팅

다음과 같은 요청은 자동으로 claude-skills 플러그인이 처리합니다:

| 키워드 | 라우팅 대상 |
|--------|-----------|
| `.docx`, `Word`, `워드`, `문서 작성`, `변경 추적` | docx 스킬 |
| `.pptx`, `PowerPoint`, `파워포인트`, `프리젠테이션`, `슬라이드` | pptx 스킬 |
| `.xlsx`, `Excel`, `엑셀`, `스프레드시트`, `수식` | xlsx 스킬 |
| `.pdf`, `PDF`, `피디에프`, `폼 작성` | pdf 스킬 |
| `UI`, `디자인`, `컴포넌트`, `프론트엔드`, `웹 페이지` | frontend-design 스킬 |
| `Claude API`, `Claude Code`, `Anthropic`, `Claude 가격` | product-self-knowledge 스킬 |

### 시작하기

1. **처음 설치 시**: `/claude-skills:setup`을 실행하여 필요한 도구와 패키지를 설치하세요.
2. **문서 작업**: 자연어 요청 (예: "이 Word 문서를 편집해줘") 또는 슬래시 명령 사용.
3. **확장**: 외부 SKILL.md를 `/claude-skills:add-ext-skill`로 추가하여 기능 확장 가능.

### 요구사항

**시스템 도구** (setup 스킬이 자동 설치 시도):
- LibreOffice (Office 문서 렌더링)
- pandoc (DOCX 읽기)
- poppler-utils (PDF → 이미지 변환)
- qpdf (PDF 병합/분할)

**Python 패키지**: lxml, defusedxml, pypdf, pdfplumber, reportlab, pypdfium2, openpyxl, pandas, Pillow

**npm 패키지**: docx, pptxgenjs, react-icons, sharp

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 추가 파일 탐색 없이 즉시 안내 출력 |
| 2 | 명령 목록 11개 모두 포함 |
| 3 | 자동 라우팅 키워드 테이블 포함 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 에이전트 위임하지 않음 (직결형 스킬) |
| 2 | 파일 시스템 탐색하지 않음 (정적 안내만) |
| 3 | 사용자에게 불필요한 질문하지 않음 |

## 검증 체크리스트

- [ ] frontmatter에 name, description, user-invocable: true 포함
- [ ] 슬래시 명령 11개 목록 포함
- [ ] 자동 라우팅 키워드 테이블 포함
- [ ] 시작하기 안내 포함
- [ ] 요구사항 섹션 포함
- [ ] MUST 규칙, MUST NOT 규칙 섹션 포함
