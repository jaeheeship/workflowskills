---
name: workflow-manager
description: PRD → Execution Spec → Tasks 흐름을 단계별로 안내하고, 각 단계의 게이트(Workflow Gatekeeper)를 호출해 다음 단계 진행 여부를 결정한다. 에이전트가 워크플로우 순서를 잊지 않도록 강제할 때 사용한다.
---

# Workflow Manager Skill

이 스킬은 에이전트가 PRD부터 작업을 시작할 때 **정해진 순서**로 진행하도록 안내하고, 각 단계의 **게이트(검토/승인)** 를 호출한다.

## Canonical Workflow

```
PRD 작성
  ↓
Workflow Gatekeeper (Gate 1)
  ↓
Execution Spec 작성
  ↓
Workflow Gatekeeper (Gate 2: PRD ↔ Spec)
  ↓
Tasks 생성
  ↓
Workflow Gatekeeper (Gate 3: Spec ↔ Tasks)
  ↓
Context Scouting (per task)
  ↓
Workflow Gatekeeper (Gate 4: Tasks ↔ Context)
  ↓
Database Design & Migration
  ↓
Workflow Gatekeeper (Gate 5: DB Artifacts)
```

## Responsibilities

1. 현재 단계 파악
2. 다음 단계에 필요한 입력/산출물 확인
3. 해당 단계용 스킬 호출 지시
4. Workflow Gatekeeper 게이트 통과 여부 확인
5. 통과 시 다음 단계로 진행, 미통과 시 수정 루프로 전환

## Stage Definitions

### Stage 1: PRD Creation

- **Use**: `prd-generator`
- **Output**: `workspaces/features/{prd-id}/prd.md`
- **Exit Condition**: `workflow-gatekeeper` Gate 1 승인

### Stage 2: Execution Spec Creation

- **Use**: `spec-generator`
- **Input**: `prd.md`
- **Output**: `execution-spec.md` (status: frozen)
- **Exit Condition**: `workflow-gatekeeper` Gate 2 승인

### Stage 3: Tasks Creation

- **Use**: `spec-to-tasks`
- **Input**: `execution-spec.md`
- **Output**: `tasks/index.md` + `task-*.md`
- **Exit Condition**: `workflow-gatekeeper` Gate 3 승인

### Stage 4: Context Scouting (Per Task)

- **Use**: `context-scouter`
- **Input**: `prd.md` + `task-*.md`
- **Output**: `task-*-context.md` (Task와 동일 폴더)
- **Exit Condition**: `workflow-gatekeeper` Gate 4 승인

### Stage 5: Database Design & Migration

- **Use**: `db-design-migration`
- **Input**: `execution-spec.md` + `tasks/` + `task-*-context.md`
- **Output**: 스키마 설계 문서, 마이그레이션 파일, 관련 모델/스키마 정리
- **Exit Condition**: `workflow-gatekeeper` Gate 5 승인

## Orchestration Rules

- PRD → Tasks 직접 이동 금지
- `execution-spec.md` 없으면 Tasks 생성 금지
- Gatekeeper 게이트 미통과 시, 반드시 이전 단계로 되돌려 수정한다
- Gate 승인 전에는 다음 단계 진행을 허용하지 않는다
- Task 수행 전에는 해당 Task의 context 파일이 존재해야 한다
- DB 설계/마이그레이션은 Tasks 정의 완료 이후에만 수행한다
- DB 변경은 Gate 5 승인 전까지 병합/후속 작업을 진행하지 않는다

## Output Format (Suggested)

```
Current Stage: {PRD | SPEC | TASKS}
Next Action: {스킬 호출 지시}
Required Inputs: {필요 파일}
Gate Check: {어떤 gate를 호출해야 하는지}
```

## References

- PRD Skill: [prd-generator](../prd-generator/SKILL.md)
- Execution Spec Skill: [spec-generator](../spec-generator/SKILL.md)
- Spec to Tasks Skill: [spec-to-tasks](../spec-to-tasks/SKILL.md)
- Context Scouter Skill: [context-scouter](../context-scouter/SKILL.md)
- DB Design & Migration Skill: [db-design-migration](../db-design-migration/SKILL.md)
- Workflow Gatekeeper Skill: [workflow-gatekeeper](../workflow-gatekeeper/SKILL.md)
