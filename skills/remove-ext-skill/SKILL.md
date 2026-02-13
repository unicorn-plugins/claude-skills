---
name: remove-ext-skill
description: 확장 스킬 제거
user-invocable: true
---

# Remove External Skill

[REMOVE-EXT-SKILL 스킬 활성화]

## 목표

`skills/` 디렉토리에 추가된 확장 스킬을 제거함.

## 활성화 조건

사용자가 `/claude-skills:remove-ext-skill` 호출 시 또는 "확장 스킬 제거", "스킬 삭제" 키워드 감지 시.

## 워크플로우

### Step 1: 확장 스킬 목록 표시

`skills/` 디렉토리를 탐색하여 확장 스킬 목록을 표시:

**내장 스킬 (제거 불가)**:
- core
- setup
- help
- add-ext-skill
- remove-ext-skill
- docx
- pptx
- xlsx
- pdf
- frontend-design
- product-self-knowledge

**확장 스킬** (제거 가능):
- 위 내장 스킬 목록에 없는 `skills/` 하위 디렉토리

### Step 2: 제거 대상 선택 질문

`AskUserQuestion` 도구로 사용자에게 질문:

**질문**: "제거할 확장 스킬을 선택하세요."

**선택지**: 확장 스킬 목록 (Step 1에서 추출)

**확장 스킬이 없는 경우**: "제거 가능한 확장 스킬이 없습니다." 메시지 출력 후 종료.

### Step 3: skills/{name}/ 디렉토리 삭제

선택된 스킬의 디렉토리를 삭제:

```bash
rm -rf skills/{name}/
```

### Step 4: commands/{name}.md 삭제

진입점 파일 삭제:

```bash
rm -f commands/{name}.md
```

### Step 5: Core 스킬 라우팅 테이블 제거 안내

사용자에게 안내 메시지 출력:

```
확장 스킬 '{name}'이 성공적으로 제거되었습니다.

자동 라우팅을 비활성화하려면 `skills/core/SKILL.md`의 라우팅 테이블에서 다음 항목을 제거하세요:

| ... | {name} |
```

### Step 6: 완료 보고

제거된 스킬 정보 요약:
- 스킬명: `{name}`
- 삭제된 디렉토리: `skills/{name}/`
- 삭제된 진입점: `commands/{name}.md`

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 내장 스킬(11개) 제거 금지 |
| 2 | 사용자 확인 후에만 삭제 수행 |
| 3 | skills/{name}/ 와 commands/{name}.md 모두 삭제 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 내장 스킬을 제거하지 않음 |
| 2 | 사용자 확인 없이 파일 삭제 금지 |
| 3 | Core 스킬 라우팅 테이블을 자동 수정하지 않음 (안내만) |

## 검증 체크리스트

- [ ] frontmatter에 name, description, user-invocable: true 포함
- [ ] 내장 스킬 목록 하드코딩 (11개)
- [ ] 확장 스킬 목록 추출 로직 포함
- [ ] AskUserQuestion 사용하여 제거 대상 선택
- [ ] skills/{name}/ 와 commands/{name}.md 모두 삭제
- [ ] Core 라우팅 테이블 제거 안내
- [ ] MUST 규칙, MUST NOT 규칙 섹션 포함
