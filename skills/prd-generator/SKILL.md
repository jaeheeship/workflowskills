---
name: prd-generator
description: PRD(Product Requirements Document) 작성 스킬. 일관되고 품질 높은 제품 요구사항 문서를 작성할 때 사용한다. 문제 정의, 목표 설정, 사용자 정의, 스코프 관리, 기능 설명, 우선순위 판단, 결정 기록 등 PRD 작성에 필요한 모든 기준과 가이드를 제공한다.
---

# PRD Writing Skill

PRD는 아이디어 문서가 아니라 **제품 결정을 기록하고 전달하는 도구**이며, Tasks가 올바르게 만들어지기 위한 **출발점**이다.

## When to Use This Skill

- 새로운 기능이나 제품에 대한 요구사항 문서를 작성할 때
- 기존 PRD를 검토하거나 개선할 때
- 문제 정의부터 스코프 결정까지 구조화된 문서가 필요할 때
 - 에이전트가 PRD를 생성/수정하기 위한 질문 흐름과 산출 규칙이 필요할 때

## What is a Good PRD

좋은 PRD는 다음 질문에 명확히 답한다:

1. 왜 이 문제를 지금 해결해야 하는가?
2. 누구의 문제인가?
3. 성공했다고 판단할 기준은 무엇인가?
4. 무엇을 만들고, 무엇을 만들지 않는가?

PRD는 **어떻게 만들 것인지(How)를 설명하지 않는다**.

## PRD Output Structure

PRD를 생성할 때는 작업 디렉토리를 기준으로 다음 폴더 구조를 따른다:

```
workspaces/
└── features/
    └── {prd-id}/
        ├── prd.md
        └── tasks/
```

### 규칙

- `workspaces/features/` 폴더가 없으면 생성한다
- `{prd-id}`는 PRD를 식별하는 고유 ID다 (예: `user-comment`, `order-history`)
- `prd.md`는 PRD 본문이다
- `prd.md` 문서 상단에 PRD ID를 메타데이터로 기록한다
- `tasks/` 폴더는 이후 Task 문서들이 위치할 공간이다
 - 기본 응답은 **마크다운 초안**으로 제공하며, 파일 생성/수정은 사용자가 명시적으로 요청할 때만 수행한다
 - `Core Use Cases`, `Scope`, `Priority`, `Decisions & Notes` 섹션의 각 항목에는 고유 ID를 부여한다
 - ID 형식: `Core Use Cases`는 `UC-001`, `Scope`는 `SCOPE-IN-001`/`SCOPE-OUT-001`, `Priority`는 `PRIORITY-001`, `Decisions & Notes`는 `DECISION-001`

### PRD 문서 형식

`prd.md` 파일은 다음과 같이 PRD ID를 문서 상단에 포함해야 한다:

```markdown
---
id: { prd-id }
title: { PRD 제목 }
status: draft | review | approved
created: { YYYY-MM-DD }
---

# {PRD 제목}

## Problem

...

## Goal

...

## Target User

...

## Core Use Cases

- UC-001 ...
- UC-002 ...

## Scope

### In Scope
- SCOPE-IN-001 ...
- SCOPE-IN-002 ...

### Out of Scope
- SCOPE-OUT-001 ...
- SCOPE-OUT-002 ...

## Priority

- PRIORITY-001 (Must) ...
- PRIORITY-002 (Should) ...

## Decisions & Notes

- DECISION-001 ...
- DECISION-002 ...
```

## Core Skills

### 1. Problem Definition Skill

PRD의 시작은 항상 **문제 정의**다. 기능이 아니라 **사용자 또는 비즈니스의 문제 상태**를 서술해야 한다.

**체크리스트:**

- 문제를 기능 없이 설명했는가?
- 증상(symptom)과 원인(problem)을 구분했는가?
- "없다 / 불편하다" 수준을 넘어서 있는가?

**Bad Example:** 댓글 기능이 없다

**Good Example:** 사용자가 피드백을 남길 수 있는 공식적인 수단이 없어, 재방문과 참여율이 낮다

### 2. Goal Setting Skill

Goal은 **의사결정의 기준점**이다. 모든 Spec 논의와 우선순위 판단은 Goal을 기준으로 이루어진다.

**체크리스트:**

- 정성적 목표 + 정량적 목표가 있는가?
- "완료"와 "성공"이 구분되어 있는가?

**Good Example:**

- 댓글 작성률 30% 증가
- 게시물당 평균 상호작용 수 증가

### 3. User Definition Skill

PRD의 사용자는 **가상의 모두**가 아니다. 명확한 사용자 범위를 정의해야 불필요한 기능 확장을 막을 수 있다.

**체크리스트:**

- 로그인 여부, 역할, 맥락이 정의되어 있는가?
- 예외 사용자를 명시했는가?

**Good Example:**

- 로그인한 일반 사용자
- 관리자 계정은 제외

### 4. Scope Control Skill (In / Out)

PRD에서 가장 중요한 스킬 중 하나는 **하지 않을 것을 명확히 적는 것**이다.

**체크리스트:**

- Out of Scope가 실제로 존재하는가?
- "나중에" 항목이 명시되어 있는가?

**Good Example:**

In Scope:

- 댓글 작성 / 삭제

Out of Scope:

- 대댓글
- 멘션
- 실시간 알림

### 5. Feature Description Skill (What, not How)

PRD의 기능 설명은 **사용자 관점의 행동 단위**여야 한다. 기술 용어, 구현 방식, UI 세부는 포함하지 않는다.

**체크리스트:**

- API, DB, UI 컴포넌트 언급이 없는가?
- 사용자 행동으로 설명되어 있는가?

**Bad Example:** POST /comments API 제공

**Good Example:** 사용자는 게시물에 텍스트 댓글을 남길 수 있다

### 6. Priority Thinking Skill

모든 기능은 **동일하게 중요하지 않다**. PRD는 우선순위에 대한 판단을 포함해야 한다.

**체크리스트:**

- Must / Should / Could 구분이 있는가?
- MVP 범위가 명확한가?

### 7. Decision Recording Skill

PRD는 토론 문서가 아니라 **결정의 기록**이다. 왜 이런 선택을 했는지 남겨야 이후 Spec/Tasks에서 흔들리지 않는다.

**체크리스트:**

- 중요한 결정에 이유가 기록되어 있는가?
- 대안이 있었음을 명시했는가?

## What PRD Must NOT Contain

PRD에 아래 내용이 들어가면 안 된다:

- API 명세
- DB 스키마
- 기술 스택
- 구현 로직
- 일정 / 담당자
- Task 단위 작업 목록

이 내용은 Spec 또는 Tasks의 책임이다.

## PRD Quality Checklist

PRD를 완료하기 전, 아래 질문에 모두 YES여야 한다:

- [ ] 이 PRD만 읽고 "무엇을 만들지" 이해할 수 있는가?
- [ ] 개발자가 "왜 이 기능이 필요한지" 물어보지 않아도 되는가?
- [ ] Spec 작성자가 임의의 판단을 하지 않아도 되는가?
- [ ] Scope creep를 막을 수 있는가?

## Agent Workflow (Recommended)

에이전트는 다음 순서로 PRD 생성을 진행한다:

1. **입력 수집**: 목표, 대상 사용자, 현재 문제를 3~5개의 질문으로 확인한다.
2. **ID 제안**: `{prd-id}` 후보를 1~3개 제안하고 사용자 선택을 받는다.
3. **초안 생성**: Problem/Goal/Target User/Core Use Cases/Scope/Priority/Decisions 순서로 작성한다.
4. **검토 요청**: 빠진 정보와 모호한 부분을 질문으로 요약한다.
5. **상태 업데이트**: 합의가 끝나면 `status`를 `review` 또는 `approved`로 갱신한다.

질문 예시:

- 어떤 지표가 개선되면 성공인가요? (정량/정성)
- 이 기능의 1차 사용자 범위는 어디까지인가요?
- 반드시 포함/제외할 항목이 있나요?

작성 규칙:

- 문서는 한국어로 작성한다
- 구현/기술/일정 내용은 포함하지 않는다
- 결정 사항은 이유와 함께 기록한다

## PRD → Spec Handoff Rule

- PRD가 승인되기 전에는 Spec을 작성하지 않는다
- PRD 변경 시, Spec은 반드시 재검토한다
- PRD는 Spec을 참조하지 않는다 (단방향)

## References

- [Good Example](reference/good-example.md) - 잘 작성된 PRD 예시
