---
name: workflow-gatekeeper
description: PRD부터 Execution Spec, Spec-to-Tasks까지의 워크플로우가 일관되게 진행되는지 감리/게이트를 수행한다. PRD/Spec 품질 검증, 정합성 점검, 다음 단계 진행 여부 판단이 필요할 때 사용한다.
---

# Workflow Gatekeeper Skill

PRD → Execution Spec → Tasks → Context Scouting → Database Design 흐름이 **일관되고 검증 가능하게** 진행되었는지 판단하는 감독관 역할이다. 자동 수정이 아니라 **검토/승인/반려**가 핵심이다.

## Inputs

- `workspaces/features/{prd-id}/prd.md`
- `workspaces/features/{prd-id}/execution-spec.md` (있을 경우)
- `workspaces/features/{prd-id}/tasks/` (있을 경우)
- `workspaces/features/{prd-id}/tasks/*-context.md` (있을 경우)
- `database/migrations/` (있을 경우)
- `docs/` 또는 `workspaces/` 내 스키마 문서 (있을 경우)

## Outputs

리포트는 항상 파일로 저장한다:

```
workspaces/features/{prd-id}/
└── reports/
    ├── report-001.md
    ├── report-002.md
    └── ...
```

### Report Naming

- 최초 리포트는 `report-001.md`부터 시작한다.
- 다음 리포트는 기존 파일의 최대 번호 + 1을 사용한다.

## Gate Decision (Required)

다음 상태 중 하나로 판단한다:

- `APPROVE`: 다음 단계로 진행 가능
- `HOLD`: 필수 수정 필요 (재검토 전까지 진행 금지)
- `REJECT`: 문서 자체가 기준 미달 (재작성 필요)

## Review Workflow

### 1) PRD 품질 점검

- Problem/Goal/Target User/Core Use Cases/Scope/Priority/Decisions가 존재하는가?
- Problem이 기능이 아닌 문제 상태로 서술되었는가?
- Out of Scope가 명시되어 있는가?
- 성공 기준이 측정 가능하게 정의되었는가?

### 2) PRD → Execution Spec 정합성

- `execution-spec.md`의 `status`가 `frozen`인가?
- PRD의 Core Use Cases/Scope가 FR/Scope에 반영되었는가?
- PRD의 Decisions/Notes가 Constraints/Assumptions에 반영되었는가?
- PRD의 모호성이 제거되었는가? (Open Question 없음)

### 3) Execution Spec → Tasks 정합성

- 모든 FR이 Task에 매핑되었는가?
- Out of Scope 항목이 Task에 포함되지 않았는가?
- Constraints가 각 Task의 Acceptance Criteria/Notes에 반영되었는가?
- Traceability가 Task frontmatter와 index에 모두 기록되었는가?

### 4) Gate 4: Tasks → Context Scouting 정합성

- 각 Task에 대응하는 `task-*-context.md`가 존재하는가?
- Context 파일이 PRD/Task를 함께 참조하고 있는가?
- Existing/To Create가 구분되어 있는가?

### 5) Gate 5: Database Design & Migration 정합성

- Tasks 정의가 완료된 이후에 수행되었는가?
- 스키마 변경이 Execution Spec/Tasks의 범위와 일치하는가?
- 기존 스키마 목록과 테이블 정의(`workspaces/schemas/index.md`, `workspaces/schemas/tables/`)를 확인했는가?
- 마이그레이션 파일이 존재하고 명명 규칙을 따르는가?
- 모델/스키마 문서가 갱신되었는가? (해당 프로젝트 규칙 기준)
- DB 변경 사항이 관련 `task-*-context.md`에 반영되었는가?

## Report Format (Required)

```
---
prd-id: {prd-id}
report-id: report-001
gate: APPROVE | HOLD | REJECT
created: {YYYY-MM-DD}
inputs:
  - prd.md
  - execution-spec.md
  - tasks/
---

# Workflow Gatekeeper Report – {PRD title}

## Gate

{APPROVE | HOLD | REJECT}

## Reason

{핵심 판단 근거 요약}

## Findings

- [P1] {필수 수정 사항}
- [P2] {중요 수정 사항}

## Required Actions

- {수정 지시}

## Next Step

- {진행 단계 안내}
```

## Failure Conditions

아래 중 하나라도 해당하면 `HOLD` 이상으로 처리한다:

- PRD 필수 섹션 누락
- Execution Spec이 `frozen`이 아님
- FR ↔ Task 매핑 누락
- Out of Scope 위반
- Task 대비 context 파일 누락 (컨텍스트 단계가 요구되는 경우)
- DB 변경이 있으나 마이그레이션/스키마 문서가 누락됨

## References

- PRD Skill: [prd-generator](../prd-generator/SKILL.md)
- Execution Spec Skill: [spec-generator](../spec-generator/SKILL.md)
- Spec to Tasks Skill: [spec-to-tasks](../spec-to-tasks/SKILL.md)
- Context Scouter Skill: [context-scouter](../context-scouter/SKILL.md)
- DB Design & Migration Skill: [db-design-migration](../db-design-migration/SKILL.md)
