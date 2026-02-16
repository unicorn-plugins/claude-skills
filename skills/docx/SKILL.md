---
name: docx
description: "Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx files). Triggers include: any mention of 'Word doc', 'word document', '.docx', or requests to produce professional documents with formatting like tables of contents, headings, page numbers, or letterheads. Also use when extracting or reorganizing content from .docx files, inserting or replacing images in documents, performing find-and-replace in Word files, working with tracked changes or comments, or converting content into a polished Word document. If the user asks for a 'report', 'memo', 'letter', 'template', or similar deliverable as a Word or .docx file, use this skill. Do NOT use for PDFs, spreadsheets, Google Docs, or general coding tasks unrelated to document generation."
type: orchestrator
---

# Word 문서(DOCX) 처리 스킬

[DOCX 스킬 활성화]

## 목표

Word 문서(.docx) 생성, 읽기, 편집, 분석을 전문 에이전트에 위임하여 사용자 요청을 완수함.

## 활성화 조건

사용자가 `.docx`, `Word`, `워드`, `문서` 키워드를 언급하거나 보고서, 메모, 편지, 템플릿 등의 산출물을 Word 파일로 요청할 때 활성화.

## 워크플로우

### Phase 1: 작업 유형 판별 (`ulw` 활용)

사용자 요청을 분석하여 읽기/생성/편집 작업 유형을 결정함.

| 작업 유형 | 판별 조건 |
|----------|----------|
| 읽기/분석 | 파일 내용 추출, 텍스트 변환, 구조 분석 |
| 새 문서 생성 | `.docx` 파일 미존재, "만들어줘", "작성해줘" |
| 기존 문서 편집 | `.docx` 파일 존재, "수정", "변경", "추가", "삭제" |

이 Phase는 `ulw` 매직 키워드를 활용하여 수행함.

### Phase 2-A: 새 문서 생성 → Agent: office-editor (`/oh-my-claudecode:ralph` 활용)

- **TASK**: docx-js npm 패키지를 사용하여 새 Word 문서 생성
- **EXPECTED OUTCOME**: 유효한 OOXML 구조를 가진 `.docx` 파일 생성. XSD 검증 통과
- **MUST DO**:
  - US Letter 페이지 크기 명시 설정 (A4 기본값 회피)
  - 리스트는 numbering config 사용 (유니코드 bullet 금지)
  - 테이블은 DXA 단위로 width 설정 (percentage 금지)
  - 생성 후 validate.py로 검증
- **MUST NOT DO**:
  - PageBreak를 Paragraph 밖에 배치하지 않음
  - ImageRun에서 type 파라미터 누락하지 않음
  - 스타일 정의 없이 커스텀 스타일 참조하지 않음
- **CONTEXT**:
  - PYTHONPATH=gateway/tools 환경변수 설정 필수
  - docx-js API 레퍼런스는 `agents/office-editor/references/docx-api-reference.md` 참조

이 Phase는 `/oh-my-claudecode:ralph`를 활용하여 수행함.

### Phase 2-B: 기존 문서 읽기 → Agent: office-editor (`ulw` 활용)

- **TASK**: pandoc 변환 또는 unpack → XML 분석으로 문서 내용 추출
- **EXPECTED OUTCOME**: 텍스트, 서식, 구조 정보가 포함된 분석 결과
- **MUST DO**:
  - pandoc 사용 시 --track-changes=all 옵션 포함
  - XML 분석 시 unpack.py로 ZIP 해제
- **MUST NOT DO**: 직접 ZIP 바이너리 수정 시도하지 않음
- **CONTEXT**: PYTHONPATH=gateway/tools 설정, ooxml-structure.md 참조

이 Phase는 `ulw` 매직 키워드를 활용하여 수행함.

### Phase 2-C: 기존 문서 편집 → Agent: office-editor (`/oh-my-claudecode:ralph` 활용)

- **TASK**: unpack → XML 편집 → validate → pack 워크플로우로 문서 수정
- **EXPECTED OUTCOME**: 수정된 유효 OOXML 문서. XSD 검증 통과. LibreOffice 열기 가능
- **MUST DO**:
  - 원본 서식, 스타일, 네임스페이스 보존
  - 변경 추적 시 WHAT만 마킹 (최소 편집)
  - 스마트 쿼트 XML 엔티티 사용 (`&#x2019;` 등)
  - pack.py 실행 전 validate.py 검증
- **MUST NOT DO**:
  - XSD 검증 없이 pack 실행 금지
  - `<w:commentRangeStart>`를 `<w:r>` 안에 배치하지 않음
- **CONTEXT**:
  - PYTHONPATH=gateway/tools 설정
  - XML 편집 시 Edit 도구 직접 사용 (Python 스크립트 작성 금지)
  - 변경 추적 author는 "Claude" 사용 (사용자 명시 없으면)

이 Phase는 `/oh-my-claudecode:ralph`를 활용하여 수행함.

### Phase 3: 결과 검증 (`ulw` 활용)

생성/편집된 문서의 유효성 확인.

| 검증 항목 | 검증 방법 |
|----------|----------|
| OOXML 유효성 | validate.py XSD 스키마 검증 |
| 문서 열기 가능 | soffice.py로 LibreOffice 열기 테스트 |
| 내용 반영 확인 | pandoc 변환 또는 markitdown으로 텍스트 추출 비교 |

이 Phase는 `ulw` 매직 키워드를 활용하여 수행함.

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN | 티어 |
|----------|-----|------|
| office-editor | `claude-skills:office-editor:office-editor` | MEDIUM |

### 프롬프트 조립 절차

1. `agents/office-editor/` 에서 3파일 로드 (AGENT.md + agentcard.yaml + tools.yaml)
2. `gateway/runtime-mapping.yaml` 참조하여 구체화:
   - **모델 구체화**: agentcard.yaml의 `tier: MEDIUM` → `tier_mapping.default.MEDIUM` → `claude-sonnet-4-5`
   - **툴 구체화**: tools.yaml의 추상 도구 → `tool_mapping`에서 실제 도구 결정
   - **금지액션 구체화**: `forbidden_actions: [user_interact, agent_delegate]` → `action_mapping`에서 `[AskUserQuestion, Task]` 제외
   - **최종 도구** = (구체화된 도구) - (제외 도구)
3. 프롬프트 조립: 3파일을 합쳐 하나의 프롬프트로 구성
   - **구성 순서**: 공통 정적(runtime-mapping) → 에이전트별 정적(3파일) → 동적(작업 지시)
4. `Task(subagent_type="claude-skills:office-editor:office-editor", model="claude-sonnet-4-5", prompt=조립된 프롬프트)` 호출

### 오케스트레이션 스킬 활용

| 워크플로우 Phase | 추천 스킬 | 적용 |
|-----------------|----------|:----:|
| Phase 1 (작업 유형 판별) | `ulw` 매직 키워드 | **필수** |
| Phase 2 (생성/편집 실행) | `/oh-my-claudecode:ralph` | **필수** |
| Phase 3 (결과 검증) | `ulw` 매직 키워드 | **필수** |

### 스크립트 호출 패턴

모든 커스텀 도구 호출 시 PYTHONPATH 환경변수를 설정:

```bash
PYTHONPATH=gateway/tools python gateway/tools/office/unpack.py input.docx output_dir/
PYTHONPATH=gateway/tools python gateway/tools/office/pack.py input_dir/ output.docx
PYTHONPATH=gateway/tools python gateway/tools/docx/accept_changes.py input.docx output.docx
```

## 완료 조건

| # | 조건 |
|---|------|
| 1 | 생성/편집된 문서가 유효한 OOXML 구조를 가짐 |
| 2 | XSD 스키마 검증 통과 (validate.py) |
| 3 | LibreOffice 또는 Word에서 열 수 있음 (soffice.py 검증) |
| 4 | 사용자 요청의 모든 항목이 반영됨 |

## 검증 프로토콜

1. pack.py 실행 시 validate.py가 자동으로 XSD 검증 수행
2. 생성된 문서를 soffice.py로 열기 가능 여부 확인
3. 모든 검증 통과 후에만 완료 선언

## 상태 정리

완료 시 임시 unpack 디렉토리 삭제. `.omc/state/` 에 상태 파일 미사용 (단건 작업).

## 취소

`cancelomc` 또는 `stopomc` 키워드로 즉시 중단. 임시 unpack 디렉토리가 존재하면 삭제.

## 재개

임시 unpack 디렉토리가 남아있으면 해당 상태에서 재개 가능. 원본 파일이 보존되어 있으므로 처음부터 재시작도 가능.

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 원본 문서 편집 시 반드시 unpack → XML 편집 → validate → pack 순서 준수 |
| 2 | XML 편집 시 기존 서식, 스타일, 네임스페이스를 보존 |
| 3 | 모든 Phase에 오케스트레이션 스킬 활용 필수 명시 |
| 4 | 에이전트 위임 시 5항목 (TASK, EXPECTED OUTCOME, MUST DO, MUST NOT DO, CONTEXT) 포함 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 직접 ZIP 파일을 바이너리 수정하지 않음 (반드시 unpack/pack 사용) |
| 2 | XSD 검증 없이 문서 전달 금지 |
| 3 | 에이전트의 내부 사고 방식이나 단계별 절차를 스킬에서 기술하지 않음 |

## 검증 체크리스트

- [ ] frontmatter에 name, description 포함
- [ ] 에이전트 호출 규칙 섹션 포함 (FQN, 프롬프트 조립 4단계, 오케스트레이션 활용)
- [ ] 모든 Phase에 스킬 부스팅 명시
- [ ] 완료 조건, 검증 프로토콜, 상태 정리, 취소/재개 섹션 포함
- [ ] MUST 규칙, MUST NOT 규칙 섹션 포함
- [ ] Agent 위임 단계에 5항목 포함
