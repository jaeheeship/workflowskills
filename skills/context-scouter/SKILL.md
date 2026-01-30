---
name: context-scouter
description: |
  PRD 문서와 이미 작성된 태스크를 읽고, 태스크 수행에 필요한 컨텍스트를 사전에 주입하기 위해 `context.md`로 정리하는 작업에 사용합니다.

  사용 예시:
  - "태스크용 컨텍스트 주입"
  - "context.md 만들어줘"
  - "이 태스크 수행에 필요한 컨텍스트 정리"
---

# Context Scouter

PRD와 태스크를 함께 읽고, 단일 태스크 수행에 필요한 코드/문서 컨텍스트를 빠르게 찾아 `*-context.md`로 정리한다. 태스크 자체는 작성하지 않는다.

## Workflow

1. PRD/태스크 위치 확인

- PRD와 태스크 파일의 경로가 제공되면 모두 연다.
- 경로가 없으면 `specs/`, `docs/`에서 PRD를 찾고, `tasks/`에서 태스크를 찾는다.

2. 단일 태스크 컨텍스트 탐색

- 해당 태스크의 목표/입력/출력을 파악하고 필요한 클래스/파일을 나열한다.
- `rg`로 관련 클래스/파일을 찾고, 이미 존재하는 것과 없는 것을 나눈다.
- 일반적으로 확인할 위치:
  - 모델: `app/Models`
  - 액션/서비스: `app/Actions` / `app/Services`
  - 컨트롤러: `app/Http/Controllers`
  - 폼 요청: `app/Http/Requests`
  - 리소스: `app/Http/Resources`
  - Livewire/Volt: `app/Livewire` / `resources/views/livewire`
  - 라우트: `routes/`
  - 마이그레이션: `database/migrations`
  - 정책/게이트: `app/Policies` / `app/Providers`
  - 문서: `docs/` (특히 `docs/route.md`, `docs/schemas/`)
  - 프로젝트 관련문서 : `workspace/`

3. 컨텍스트 파일 작성

- 태스크 파일명에 `-context`를 붙여 저장한다. (예: `task-001-context.md`)
- 아래 템플릿을 사용해 간결하게 정리한다.
- 존재하는 것은 경로를 명시하고, 없는 것은 `To Create`에 명확히 적는다.

## context.md template

```markdown
# Context

## PRD Source

- file: <path>
- 핵심 요약: <1~3줄>

## Tasks

- source: <path>
- <Task>: <목표/입력/출력 요약>

## Domain Entities (Models)

- Existing:
  - <ModelName> — <path>
- To Create:
  - <ModelName> — 이유/근거(PRD 섹션)

## Actions / Services

- Existing:
  - <ActionName> — <path>
- To Create:
  - <ActionName> — 이유/근거

## Controllers / Livewire

- Existing:
  - <Controller/Component> — <path>
- To Create:
  - <Controller/Component> — 이유/근거

## Requests / Resources

- Existing:
  - <Request/Resource> — <path>
- To Create:
  - <Request/Resource> — 이유/근거

## Routes

- Existing:
  - <route name/path> — <path>
- To Create:
  - <route name/path> — 이유/근거

## Migrations / Schemas

- Existing:
  - <migration/schema> — <path>
- To Create:
  - <migration/schema> — 이유/근거

## Policies / Permissions

- Existing:
  - <Policy/Middleware> — <path>
- To Create:
  - <Policy/Middleware> — 이유/근거

## Notes / Risks

- <리스크 또는 의존성>
```

## Output Rules

- 컨텍스트 파일은 태스크와 같은 폴더에 `*-context.md` 규칙으로 생성한다.
- 파일 경로는 가능한 한 정확하게 기입한다.
- 태스크 번호/명과 PRD 섹션을 근거로 명시한다.
- 추정이 필요한 경우 `추정` 표시를 붙인다.
