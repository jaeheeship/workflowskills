# Skill: db-design-migration

Tasks가 모두 정의된 이후에만 DB 설계와 마이그레이션을 수행한다. 이 스킬은 스키마 설계, 마이그레이션 작성, 스키마 문서 갱신까지 포함한다.

## When to Use This Skill

- `tasks/index.md`와 `task-*.md`가 모두 생성된 이후
- DB 변경이 필요한 `data`/`api`/`feature` 타입 Task가 포함된 경우

## Inputs

- `workspaces/features/{prd-id}/execution-spec.md`
- `workspaces/features/{prd-id}/tasks/`
- `workspaces/features/{prd-id}/tasks/*-context.md`
- `workspaces/schemas/index.md`
- `workspaces/schemas/tables/`

## Preconditions

- Tasks 정의가 완료되어 있어야 한다
- 필요한 컨텍스트 파일이 존재해야 한다
- 기존 스키마 목록과 테이블 정의를 먼저 확인한다

## Outputs

- 마이그레이션 파일 (`database/migrations/`)
- 스키마/모델 문서 업데이트 (프로젝트 규칙에 따름)
- DB 변경 요약 메모 (선택)

## Rules

- DB 설계는 Tasks 정의 이후에만 수행한다
- Out of Scope는 스키마 변경에 포함하지 않는다
- 기존 테이블/관계/명명 규칙을 우선 준수한다
- 변경 이유는 Task 또는 Execution Spec에 근거해 기록한다

## Suggested Workflow

1. 관련 Task의 요구사항과 컨텍스트를 확인한다
2. `workspaces/schemas/index.md`와 `workspaces/schemas/tables/`를 먼저 확인한다
3. 기존 스키마/모델/마이그레이션을 조사한다
3. 변경 사항을 설계한다 (테이블/컬럼/관계)
4. 마이그레이션 파일을 작성한다
5. 스키마 문서를 갱신한다
6. 변경된 DB 설계를 관련 `task-*-context.md`에 반영한다

## Output Notes

- 마이그레이션은 되돌릴 수 있어야 한다
- 스키마 변경은 Gate 5 승인 전까지 병합하지 않는다
