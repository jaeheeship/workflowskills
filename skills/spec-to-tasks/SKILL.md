---
name: spec-to-tasks
description: Execution Spec 문서를 읽고 task-manager 포맷의 tasks/index.md 및 task-*.md를 생성한다. execution-spec.md의 Functional Requirements/Acceptance Criteria/Scope/Constraints를 Task로 분해하고 Traceability를 유지해야 할 때 사용한다.
---

# Spec to Tasks Skill

Execution Spec은 task 분해의 단일 입력 계약이다. 이 스킬은 `execution-spec.md`를 읽고 Task 목록과 개별 Task 파일을 생성한다.

## Input

- `workspaces/features/{prd-id}/execution-spec.md`

## Output

```
workspaces/
└── features/
    └── {prd-id}/
        ├── prd.md
        ├── execution-spec.md
        └── tasks/
            ├── index.md
            ├── task-001.md
            ├── task-002.md
            └── ...
```

## Preconditions

- `execution-spec.md`의 frontmatter `status`는 반드시 `frozen`이어야 한다.

## Traceability (Required)

사람과 에이전트가 함께 관리하기 쉽도록 **이중 기록**한다:

1. **개별 Task frontmatter**에 traceability를 기록한다.
2. **tasks/index.md**에 Traceability 매트릭스를 추가한다.

### Task frontmatter 규칙

```markdown
---
id: TASK-001
title: { Task 제목 }
type: feature | api | ui | data | test | infra
priority: must | should | could
status: todo | in-progress | done | blocked
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
traceability:
    functional_requirements: [FR-1, FR-2]
    acceptance_criteria: [PRD-AC-1]
---
```

### index.md Traceability 섹션

```markdown
## Traceability

| FR ID | Tasks              |
| ----- | ------------------ |
| FR-1  | TASK-001, TASK-003 |
| FR-2  | TASK-002           |
```

## Task Decomposition Rules

1. **Functional Requirements 우선**
    - 모든 FR은 최소 1개의 Task로 연결되어야 한다.
    - FR이 다중 행동이면 분리해 각각 Task로 매핑한다.

2. **Acceptance Criteria 반영**
    - PRD-AC는 관련 Task의 Acceptance Criteria로 분해한다.
    - PRD-AC가 광범위하면 여러 Task에 분배한다.

3. **Scope/Constraints 준수**
    - Out of Scope는 Task로 만들지 않는다.
    - Constraints는 각 Task의 Acceptance Criteria 또는 Technical Notes에 반영한다.

4. **Task 타입 분류**
    - `data`: 테이블/모델/마이그레이션
    - `api`: API 엔드포인트, 리소스
    - `ui`: 컴포넌트, 폼, 화면
    - `feature`: 도메인 흐름 통합
    - `test`: 통합/기능 테스트
    - `infra`: 설정/환경

5. **Task 크기 기준**
    - 1~4시간 내 완료 가능 규모로 분해한다.
    - 너무 큰 Task는 분할한다.

## Task Output Structure

Task는 Execution Spec이 위치한 폴더의 `tasks/` 하위에 생성한다:

```
workspaces/
└── features/
    └── {prd-id}/
        ├── prd.md
        ├── execution-spec.md
        └── tasks/
            ├── index.md
            ├── task-001.md
            ├── task-002.md
            └── ...
```

## Task Extraction Process

### Step 1: Execution Spec 분석

Execution Spec에서 다음 섹션을 분석한다:

1. **Functional Requirements**: Task의 주요 출처 (각 FR은 최소 1개 Task로 연결)
2. **Acceptance Criteria**: Task의 Acceptance Criteria로 분해
3. **Scope (In/Out)**: Out of Scope는 Task로 만들지 않음
4. **Constraints**: Task의 Acceptance Criteria 또는 Technical Notes에 반영

### Step 1.1: 기존 코드베이스 분석

Task 작성 전 프로젝트의 기존 구조를 파악한다:

1. **테이블 스키마**: 기존 DB 스키마를 확인하여 연관 테이블, 컬럼명 규칙, 관계 파악
2. **API 패턴**: 기존 API 구조와 네이밍 컨벤션 확인
3. **코드 컨벤션**: 프로젝트의 디렉토리 구조, 파일명 규칙 파악

> 필수: `data` 타입 Task 작성 시 기존 테이블 스키마를 반드시 참고하여 일관성을 유지한다.

### Step 2: Task 분해 원칙

**Good Task의 특징:**

- 하나의 명확한 목표를 가진다
- 1-4시간 내 완료 가능한 크기
- 독립적으로 테스트 가능하다
- 완료 조건이 명확하다
- PR을 사람이 리뷰할 수 있는 수준으로 한다

**Task 분해 체크리스트:**

- [ ] 하나의 FR이 여러 Task로 분해되었는가?
- [ ] 각 Task는 독립적으로 완료 가능한가?
- [ ] 의존성이 명확히 정의되었는가?
- [ ] Priority가 Spec과 일치하는가?
- [ ] `api`, `feature` 타입 Task에 테스트가 포함되어 있는가?

**필수 규칙:**

- `api` 타입 Task는 항상 API 테스트를 포함한다
- `feature` 타입 Task는 항상 통합 테스트를 포함한다

### Step 3: Task 유형 분류

| 유형      | 설명                | 예시                  |
| --------- | ------------------- | --------------------- |
| `feature` | 사용자 기능 구현    | 댓글 작성 기능 구현   |
| `api`     | API 엔드포인트 구현 | 댓글 CRUD API 구현    |
| `ui`      | UI 컴포넌트 구현    | 댓글 입력 폼 컴포넌트 |
| `data`    | 데이터 모델/스키마  | 댓글 테이블 설계      |
| `test`    | 테스트 작성         | 댓글 기능 통합 테스트 |
| `infra`   | 인프라/설정         | 댓글 서비스 배포 설정 |

## index.md Format

`tasks/index.md`는 아래 포맷을 따른다:

```markdown
## Traceability

| FR ID | Tasks              |
| ----- | ------------------ |
| FR-1  | TASK-001, TASK-003 |
```

## Individual Task File Format

개별 Task 파일 (`task-001.md`)은 다음 형식을 따른다:

```markdown
---
id: TASK-001
title: { Task 제목 }
type: feature | api | ui | data | test | infra
priority: must | should | could
status: todo | in-progress | done | blocked
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
traceability:
    functional_requirements: [FR-1, FR-2]
    acceptance_criteria: [PRD-AC-1]
---

# TASK-001: {Task 제목}

## Description

{Task에 대한 상세 설명}

## Acceptance Criteria

- [ ] {완료 조건 1}
- [ ] {완료 조건 2}
- [ ] {완료 조건 3}

## Dependencies

- Depends on: {의존하는 Task ID 목록 또는 "None"}
- Blocks: {이 Task에 의존하는 Task ID 목록 또는 "None"}

## Technical Notes

{구현 시 참고할 기술적 사항}

## Related

- PRD: [{prd-id}](../prd.md)
- Related Tasks: {관련 Task 목록}
```

## Task Status Management

### Status Flow

```
todo → in-progress → done
         ↓
       blocked → in-progress → done
```

### Status Update Rules

- Task 시작 시: `todo` → `in-progress`
- Task 완료 시: `in-progress` → `done`, Acceptance Criteria 모두 체크
- 블로커 발생 시: `in-progress` → `blocked`, 블로커 사유 기록
- 블로커 해소 시: `blocked` → `in-progress`

## Quality Checklist

Task 목록 생성 후 다음을 확인한다:

- [ ] 모든 In Scope 항목이 Task로 분해되었는가?
- [ ] Out of Scope 항목이 Task에 포함되지 않았는가?
- [ ] Must 항목이 MVP Task로 분류되었는가?
- [ ] 각 Task의 Acceptance Criteria가 명확한가?
- [ ] Task 간 의존성이 순환하지 않는가?

## Failure Conditions

다음 조건을 만족하지 못하면 생성 실패로 간주한다:

- [ ] 모든 FR이 Task에 매핑되었는가?
- [ ] Out of Scope 항목이 포함되지 않았는가?
- [ ] `execution-spec.md`가 `frozen` 상태인가?
- [ ] Traceability가 Task와 index에 모두 기록되었는가?

## References

- Execution Spec Skill: [spec-generator](../spec-generator/SKILL.md)
