# Skill: execution-spec-generator

Human-friendly PRD를 AI agent가 실행 가능한 Execution Spec으로 변환하는 스킬.  
이 스펙은 task 분해와 task loop의 단일 입력 계약(single source of truth)이다.

# Execution Spec Generator Skill

Execution Spec은 PRD를 **해석한 결과물**이며,
사람을 위한 문서가 아니라 **agent가 실행하기 위한 계약 문서**다.

PRD는 자유롭고 서술적일 수 있지만,
Execution Spec은 반드시 **정형적이고 판정 가능**해야 한다.

---

## When to Use This Skill

- Human-friendly PRD가 승인되었을 때
- PRD를 기반으로 task-decomposer를 실행하기 전
- PRD의 모호함을 제거하고 실행 가능한 계약으로 고정할 때

---

## Input

- `workspaces/features/{prd-id}/prd.md`

---

## Output

```
workspaces/features/{prd-id}/
├── prd.md
├── execution-spec.md   ← generated
└── tasks/
```

---

## Execution Spec Document Format (Required)

```markdown
Meta:
- prd_id: { prd-id }
- generated_from: prd.md
- generated_at: YYYY-MM-DD
- status: frozen

# Execution Spec – {PRD title}

## Functional Requirements

## Acceptance Criteria

## Scope

## Constraints

## Assumptions

## Traceability
```

Execution Spec은 생성 후 **수정 불가(immutable)** 이다.  
수정이 필요하면 PRD를 수정하고 **재생성**한다.

---

## Core Responsibilities

Execution Spec Generator는 다음을 수행한다:

1. PRD에서 실행 가능한 요구사항을 추출한다
2. 암묵적 조건을 명시적 계약으로 고정한다
3. task 분해가 가능한 최소 단위로 정규화한다
4. PRD ↔ task 간 추적 가능성을 만든다

---

## 1. Functional Requirements Normalization

### Goal

PRD에 흩어진 기능 설명을 **1행 = 1행동** 규칙으로 정규화한다.

### Rules

- 1 FR = 1 사용자 또는 시스템 행동
- AND / OR 조건 분리
- UI / API / DB / 기술 용어 제거
- 순수한 “What”만 남긴다

### Output Example

```text
Functional Requirements
- FR-1: Authenticated users can create text comments on a post
- FR-2: Users can delete only comments they created
- FR-3: Users can view comments for a post in chronological order
```

---

## 2. Acceptance Criteria Derivation (PRD-Level)

### Goal

PRD 성공 여부를 판단할 **최상위 결과 조건**을 명시한다.

### Rules

- 여러 task에 걸치는 조건만 허용
- Outcome 중심 (결과)
- HTTP / API / DB / 테스트 언급 금지
- 없으면 PRD 실패

### Output Example

```text
Acceptance Criteria
- PRD-AC-1: Authenticated users can successfully create comments
- PRD-AC-2: Users cannot delete comments created by others
- PRD-AC-3: Comments are consistently displayed in chronological order
```

---

## 3. Scope Fixation

### Goal

PRD에서 암묵적인 범위를 **명시적 In / Out**으로 고정한다.

### Rules

- Out-of-scope는 agent 확장을 차단하는 가드레일
- PRD의 Decisions / Notes를 적극 활용

### Output Example

```text
Scope
In:
- Comment creation
- Comment deletion by owner
- Comment listing

Out:
- Nested comments
- Mentions
- Real-time notifications
```

---

## 4. Constraints Extraction

### Goal

PRD에 흩어진 제약을 실행 규칙으로 고정한다.

### Rules

- Must / Must not 형태
- 위반 시 task 실패

### Output Example

```text
Constraints
- Only authenticated users can create comments
- Only text comments are supported
```

---

## 5. Assumption Resolution

### Goal

Open Question을 제거하고 loop가 멈추지 않게 한다.

### Rules

- 질문 형태 금지
- 명시적 고정값 제공
- 보수적(default-safe) 선택

### Output Example

```text
Assumptions
- Maximum comment length is 500 characters
- No profanity filtering is applied
```

---

## 6. Traceability Map Generation

### Goal

PRD → Execution Spec → Task의 연결 고리를 만든다.

### Output Example

```text
Traceability
- FR-1 → Core Use Case – Comment creation
- FR-2 → Scope – Comment deletion
```

이 맵은 이후 `prd-task-alignment-check`에서 사용된다.

---

## Quality Gate (Strict)

Execution Spec은 아래 조건을 모두 만족해야 한다:

- [ ] 모든 Functional Requirement가 단일 행동인가?
- [ ] Acceptance Criteria가 Outcome 수준인가?
- [ ] Open Question이 존재하지 않는가?
- [ ] task 분해가 가능해 보이는가?

조건을 만족하지 못하면 **생성 실패**로 간주한다.

---

## Execution Order in Workflow

```text
Human PRD
  ↓
execution-spec-generator
  ↓
execution-spec.md (frozen)
  ↓
task-decomposer
```

---

## Design Principles (Non-Negotiable)

- PRD는 설명 문서
- Execution Spec은 계약 문서
- Execution Spec은 절대 수정하지 않는다
- 모든 task는 Execution Spec에서만 생성된다

---

## Summary

> PRD는 생각을 담고  
> Execution Spec은 실행을 강제한다.

이 스킬은 **사람의 모호함을 기계의 확실함으로 번역**하는 단계다.
