---
name: pdf
description: PDF 처리 스킬 — 읽기, 생성, 편집, 폼 작성, 변환
---

# PDF 처리

[PDF 처리 스킬 활성화]

## 목표

PDF 파일에 대한 모든 작업을 처리함. 텍스트/테이블 추출, PDF 병합/분할, 회전, 워터마크 추가, 신규 PDF 생성, 폼 작성, 암호화/복호화, 이미지 추출, OCR을 통한 스캔 PDF 검색 가능화 등을 포함.

## 활성화 조건

사용자가 `.pdf` 파일을 언급하거나 PDF 관련 작업 요청 시 활성화.

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN | 티어 |
|----------|-----|------|
| pdf-handler | `claude-skills:pdf-handler:pdf-handler` | MEDIUM |

### 프롬프트 조립 절차

1. `agents/pdf-handler/` 에서 3파일 로드 (AGENT.md + agentcard.yaml + tools.yaml)
2. `gateway/runtime-mapping.yaml` 참조하여 구체화:
   - **모델 구체화**: agentcard.yaml의 `tier: MEDIUM` → `tier_mapping.default.MEDIUM` → `claude-sonnet-4-5`
   - **툴 구체화**: tools.yaml의 추상 도구 → `tool_mapping`에서 실제 도구 결정
   - **금지액션 구체화**: `forbidden_actions: [user_interact, agent_delegate]` → `action_mapping`에서 `[AskUserQuestion, Task]` 제외
   - **최종 도구** = (구체화된 도구) - (제외 도구)
3. 프롬프트 조립: 공통 정적(runtime-mapping) → 에이전트별 정적(3파일) → 동적(작업 지시)
4. `Task(subagent_type="claude-skills:pdf-handler:pdf-handler", model="claude-sonnet-4-5", prompt=조립된 프롬프트)` 호출

### 오케스트레이션 스킬 활용

| 워크플로우 Phase | 추천 스킬 | 적용 |
|-----------------|----------|:----:|
| Phase 1 (작업 유형 판별) | `ulw` 매직 키워드 | **필수** |
| Phase 2-A (PDF 읽기/추출) | `ulw` 매직 키워드 | **필수** |
| Phase 2-B (새 PDF 생성) | `/oh-my-claudecode:ralph` | **필수** |
| Phase 2-C (병합/분할) | `ulw` 매직 키워드 | **필수** |
| Phase 2-D (폼 작성) | `/oh-my-claudecode:ralph` | **필수** |
| Phase 2-E (이미지 변환) | `ulw` 매직 키워드 | **필수** |
| Phase 3 (결과 검증) | `ulw` 매직 키워드 | **필수** |

## 워크플로우

### Phase 1: 작업 유형 판별 (`ulw` 활용)

사용자 요청을 분석하여 작업 유형을 읽기/생성/편집/폼/변환 중 하나로 분류.

### Phase 2-A: PDF 읽기/추출 → Agent: pdf-handler (`ulw` 활용)

- **TASK**: pypdf, pdfplumber를 활용하여 PDF에서 텍스트, 테이블, 메타데이터 추출
- **EXPECTED OUTCOME**: 추출된 텍스트/테이블 데이터, 메타데이터
- **MUST DO**: pdfplumber로 테이블 추출 시 레이아웃 보존, 스캔 PDF는 OCR 필요 여부 안내
- **MUST NOT DO**: 스캔 PDF를 일반 추출로 처리하지 않음 (OCR 필요)
- **CONTEXT**: 원본 PDF 경로, 추출 대상(텍스트/테이블/메타데이터), `PYTHONPATH=gateway/tools` 환경변수 설정 필수

### Phase 2-B: 새 PDF 생성 → Agent: pdf-handler (`/oh-my-claudecode:ralph` 활용)

- **TASK**: reportlab를 활용하여 신규 PDF 생성
- **EXPECTED OUTCOME**: 유효한 PDF 구조를 가진 신규 PDF 파일
- **MUST DO**: Paragraph 객체에서 `<sub>`, `<super>` 태그 사용 (Unicode 첨자 금지), 여러 페이지 생성 시 Platypus 프레임워크 활용
- **MUST NOT DO**: Unicode 첨자/위첨자 문자 (₀₁₂, ⁰¹²) 사용 금지 (검은 박스 렌더링됨)
- **CONTEXT**: 콘텐츠 요구사항, 페이지 레이아웃, `PYTHONPATH=gateway/tools` 환경변수 설정 필수

### Phase 2-C: 병합/분할 → Agent: pdf-handler (`ulw` 활용)

- **TASK**: pypdf + qpdf로 PDF 병합/분할
- **EXPECTED OUTCOME**: 병합된 PDF 또는 분할된 PDF 파일들
- **MUST DO**: qpdf 활용하여 구조 무결성 보존
- **MUST NOT DO**: 메타데이터 손실 금지
- **CONTEXT**: 원본 PDF 경로, 병합/분할 범위, `PYTHONPATH=gateway/tools` 환경변수 설정 필수

### Phase 2-D: 폼 작성 → Agent: pdf-handler (`/oh-my-claudecode:ralph` 활용)

- **TASK**: 전략 판별(필드 방식 vs 주석 오버레이) 후 PDF 폼 작성
- **EXPECTED OUTCOME**: 작성 완료된 PDF 폼, 바운딩 박스 검증 통과
- **MUST DO**: check_fillable_fields.py로 필드 확인, fill_fillable_fields.py(필드 방식) 또는 fill_pdf_form_with_annotations.py(주석 오버레이) 선택, check_bounding_boxes.py로 좌표 검증
- **MUST NOT DO**: 전략 판별 없이 폼 작성 시도 금지, 바운딩 박스 검증 없이 결과 전달 금지
- **CONTEXT**: 원본 PDF 경로, 폼 데이터, agents/pdf-handler/references/pdf-forms-guide.md 참조, `PYTHONPATH=gateway/tools` 환경변수 설정 필수

**스크립트 호출 패턴**:

```bash
python gateway/tools/pdf/check_fillable_fields.py input.pdf
python gateway/tools/pdf/fill_fillable_fields.py input.pdf output.pdf
PYTHONPATH=gateway/tools python gateway/tools/pdf/fill_pdf_form_with_annotations.py input.pdf output.pdf
```

### Phase 2-E: 이미지 변환 → Agent: pdf-handler (`ulw` 활용)

- **TASK**: PDF를 PNG 이미지로 변환
- **EXPECTED OUTCOME**: 각 페이지가 PNG 이미지로 변환됨
- **MUST DO**: convert_pdf_to_images.py 활용
- **MUST NOT DO**: 이미지 품질 저하 금지
- **CONTEXT**: 원본 PDF 경로, 출력 디렉토리, `PYTHONPATH=gateway/tools` 환경변수 설정 필수

**스크립트 호출 패턴**:

```bash
PYTHONPATH=gateway/tools python gateway/tools/pdf/convert_pdf_to_images.py input.pdf output_dir/
```

### Phase 3: 결과 검증 (`ulw` 활용)

- PDF 읽기 가능 여부 확인 (pypdf)
- 폼 작성인 경우 check_bounding_boxes.py로 좌표 검증
- create_validation_image.py로 시각적 검증 이미지 생성
- 모든 검증 통과 후에만 완료 선언

**스크립트 호출 패턴**:

```bash
python gateway/tools/pdf/check_bounding_boxes.py output.pdf
PYTHONPATH=gateway/tools python gateway/tools/pdf/create_validation_image.py output.pdf validation.png
```

## 완료 조건

- [ ] 생성/편집된 PDF가 유효한 PDF 구조를 가짐
- [ ] 폼 작성 시 check_bounding_boxes.py 검증 통과
- [ ] 시각적 검증 이미지(create_validation_image.py)로 확인 가능
- [ ] 사용자 요청의 모든 항목이 반영됨

## 검증 프로토콜

1. PDF 읽기 가능 여부 확인 (pypdf)
2. 폼 작성인 경우 check_bounding_boxes.py로 좌표 검증
3. create_validation_image.py로 시각적 검증 이미지 생성
4. 모든 검증 통과 후에만 완료 선언

## 상태 정리

완료 시 임시 이미지 파일 생성 시 사용자에게 안내. 상태 파일 미사용 (단건 작업).

## 취소

`cancelomc` 또는 `stopomc` 키워드로 즉시 중단.

## 재개

원본 PDF가 보존되어 있으므로 처음부터 재시작 가능.

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 폼 작성 시 반드시 전략 판별 (필드 방식 vs 주석 오버레이) 선행 |
| 2 | 시각적 검증 이미지를 생성하여 결과 확인 |
| 3 | 모든 Phase에 오케스트레이션 스킬 활용 필수 명시 |
| 4 | 에이전트 위임 시 5항목 (TASK, EXPECTED OUTCOME, MUST DO, MUST NOT DO, CONTEXT) 포함 |
| 5 | PDF 커스텀 도구 호출 시 `PYTHONPATH=gateway/tools` 환경변수 설정 필수 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 폼 전략 판별 없이 직접 폼 작성 시도 금지 |
| 2 | 바운딩 박스 검증 없이 폼 결과 전달 금지 |
| 3 | 에이전트의 내부 사고 방식이나 단계별 절차를 스킬에서 기술하지 않음 |
| 4 | reportlab 사용 시 Unicode 첨자/위첨자 문자 사용 금지 |

## 검증 체크리스트

- [ ] frontmatter에 name, description 포함
- [ ] 에이전트 호출 규칙 섹션 포함 (pdf-handler FQN, 프롬프트 조립 4단계, 오케스트레이션 활용)
- [ ] 모든 Phase에 스킬 부스팅 명시
- [ ] 완료 조건, 검증 프로토콜, 상태 정리, 취소/재개 섹션 포함
- [ ] MUST 규칙, MUST NOT 규칙 섹션 포함
- [ ] Agent 위임 단계에 5항목 포함
- [ ] 스크립트 호출 패턴에 PYTHONPATH 설정 명시
