---
name: context-folder-loader
description: |
  개발을 위한 컨텍스트 주입을 위해 프로젝트 폴더 하위 파일을 탐색·요약해야 할 때 사용한다.
  예: "이 폴더를 기준으로 하위 파일들을 읽고 컨텍스트에 넣어줘", "폴더 컨텍스트 주입", "레포 구조 요약하고 개발 컨텍스트 만들어줘"
---

# Context Folder Loader

프로젝트 폴더를 탐색해 개발에 필요한 핵심 컨텍스트를 빠르게 요약한다. 파일 본문을 그대로 길게 복사하지 말고, 요약/구조/핵심 규칙 중심으로 정리한다.

## Workflow

1) 기준 폴더와 범위 확정

- 사용자가 명시하지 않으면 현재 작업 루트를 기준으로 한다.
- 대규모 레포일 수 있으므로 기본 제한을 적용한다.
  - 최대 파일 수: 60개
  - 최대 파일 크기: 200KB
- 제한을 넘을 것 같으면 먼저 범위를 좁히도록 요청한다.

2) 후보 파일 목록 수집

- 기본 제외 패턴(필요 시 사용자에게 알려 조정):
  - `.git`, `node_modules`, `vendor`, `dist`, `build`, `out`, `.next`, `.nuxt`, `.svelte-kit`, `coverage`, `tmp`, `log`, `logs`, `storage`, `public/build`, `generated`
- 기본 포함 대상:
  - 문서: `README*`, `AGENTS.md`, `CONTRIBUTING*`, `ARCHITECTURE*`, `ADR*`, `docs/`
  - 설정/메타: `package.json`, `composer.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `requirements.txt`, `.env.example`, `tsconfig*.json`, `vite.config.*`, `webpack*.js`, `docker*`, `Makefile`
  - 소스 디렉토리: `src/`, `app/`, `lib/`, `packages/`, `server/`, `client/`

3) 우선순위에 따라 읽기

- 1순위: `AGENTS.md`, `README*`, 아키텍처/ADR/설계 문서
- 2순위: 패키지/빌드/런타임 설정 파일
- 3순위: 엔트리 포인트(예: `src/main`, `app/`, `server/`)
- 4순위: 테스트/스크립트/CI 설정

4) 컨텍스트 요약 작성

- 아래 출력 형식을 따른다.
- 읽은 파일 목록을 명시한다.
- 불확실한 부분은 `추정`으로 표시한다.
- 민감 정보(.env 등)는 포함하지 않는다.

## Output Format

- 읽은 파일
- 프로젝트 개요
- 기술 스택/런타임
- 핵심 디렉토리/엔트리 포인트
- 빌드/실행/테스트 흐름
- 중요한 규칙/컨벤션
- 추가로 필요한 정보(질문)

## Notes

- 파일이 너무 많거나 크면 즉시 범위 축소를 요청한다.
- 전체 본문을 그대로 붙이지 말고, 개발에 필요한 요약과 구조만 제공한다.
