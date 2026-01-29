---
name: schema-lookup
description: app/Models 아래 생성된 model과 migrations만으로 정적 분석을 통해 테이블 스키마를 Markdown 테이블 형식으로 정리한다. 데이터베이스 스키마를 빠르게 파악하거나, 문서화할 때 사용한다.
---

# Schema Lookup Skill

마이그레이션 폴더와 model:show를 함께 활용해 데이터베이스 테이블 스키마를 Markdown 테이블 형식으로 정적 문서화한다.

## When to Use This Skill

- 특정 모델의 테이블 스키마를 빠르게 확인할 때
- 새로운 기능 개발 전 관련 테이블 구조를 파악할 때
- PRD → Task 분해 시 데이터 모델 설계를 위한 기초 자료가 필요할 때
- API 또는 Feature 개발 시 필드명, 타입, 관계를 확인할 때
- 스키마 문서를 생성하거나 업데이트할 때

## Commands

스키마 정보 조회를 위한 커맨드 (DB 연결 없이 사용):

### 전체 모델 목록 조회

```bash
ls app/Models/*.php | xargs -I {} basename {} .php | sort
```

모든 모델명을 알파벳 순으로 출력한다.

### 마이그레이션 파일 찾기

```bash
ls database/migrations | sort
```

마이그레이션 파일 목록을 시간순/이름순으로 확인한다.

예시:

```bash
ls database/migrations | sort
```

### 특정 테이블 마이그레이션 위치 찾기

```bash
rg -n "Schema::create\\('table_name'|Schema::table\\('table_name'" database/migrations
```

해당 테이블을 생성/수정(컬럼 추가/변경, 인덱스 추가/변경)하는 마이그레이션 파일을 찾는다.

### 특정 모델 상세 정보 (관계/속성 보완)

```bash
php artisan model:show {ModelName}
```

모델의 속성, 관계, 이벤트 등 상세 정보를 출력한다.

예시:

```bash
php artisan model:show User
php artisan model:show Organization
```

예시:

```bash
rg -n "Schema::create\\('users'|Schema::table\\('users'" database/migrations
```

## Output Structure

스키마 문서는 `workspaces/schemas/` 폴더에 저장한다:

```
workspaces/
└── schemas/
    ├── index.md           # 전체 테이블 목록
    └── tables # 테이블 스키마 정보를 모아두는 폴더
        ├── users.md           # users 테이블 스키마
        ├── organizations.md   # organizations 테이블 스키마
        └── ...
```

### 규칙

- `workspaces/schemas/` 폴더가 없으면 생성한다
- `workspaces/schemas/tables` 폴더가 없으면 생성한다
- `index.md`는 전체 테이블 목록과 간략한 설명을 포함한다
- 개별 스키마 파일은 `{table_name}.md` 형식으로 생성한다

## Output Format

### index.md (테이블 목록)

```markdown
# 테이블 스키마 목록

프로젝트에서 사용하는 데이터베이스 테이블 목록

## 테이블 목록

| 테이블명      | 모델         | 설명              |
| ------------- | ------------ | ----------------- |
| users         | User         | 사용자 정보       |
| organizations | Organization | 조직 정보         |
| workspaces    | Workspace    | 워크스페이스 정보 |
| ...           | ...          | ...               |
```

### {table_name}.md (개별 스키마)

```markdown
# {table_name}

{테이블에 대한 간략한 설명}

## 컬럼

| 컬럼명        | 타입     | Nullable | 기본값    | 설명             |
| ------------- | -------- | -------- | --------- | ---------------- |
| id            | uuid     | No       | -         | 고유 식별자 (PK) |
| {column_name} | {type}   | {Yes/No} | {default} | {description}    |
| created_at    | datetime | Yes      | -         | 생성 일시        |
| updated_at    | datetime | Yes      | -         | 수정 일시        |

## 인덱스

| 인덱스명     | 컬럼      | 유형           |
| ------------ | --------- | -------------- |
| PRIMARY      | id        | PRIMARY        |
| {index_name} | {columns} | {UNIQUE/INDEX} |

## 관계

| 타입      | 메소드       | 관련 모델    | 외래키          |
| --------- | ------------ | ------------ | --------------- |
| HasMany   | workspaces   | Workspace    | organization_id |
| BelongsTo | organization | Organization | organization_id |
```

## Process Steps

### Step 1: 모델 목록 확인

```bash
ls app/Models/*.php | xargs -I {} basename {} .php | sort
```

### Step 2: 스키마 정보 수집 (정적 분석)

마이그레이션 파일을 직접 읽어 컬럼/인덱스 정보를 추출하고, 테이블 생성 외 추가/변경 마이그레이션도 반영한다. 필요 시 model:show로 관계/속성 정보를 보완:

```bash
rg -n "Schema::create\\('table_name'|Schema::table\\('table_name'" database/migrations
php artisan model:show {ModelName}
```

### Step 3: Markdown 문서 생성

커맨드 출력 결과를 기반으로 Output Format에 맞춰 Markdown 테이블로 정리한다.

### Step 4: 파일 저장

1. `workspaces/schemas/` 폴더가 없으면 생성
2. `workspaces/schemas/tables/{table_name}.md` 파일 저장
3. `workspaces/schemas/index.md` 파일 업데이트 (테이블 목록에 추가)

## Examples

### Example 1: 전체 모델 목록 조회

**요청**: "현재 프로젝트의 모든 모델 목록을 보여줘"

**실행**:

```bash
ls app/Models/*.php | xargs -I {} basename {} .php | sort
```

### Example 2: 특정 모델 스키마 확인 (DB 연결 없이)

**요청**: "User 모델의 스키마를 확인해줘"

**실행**:

```bash
php artisan model:show User
```

### Example 3: 스키마 문서 생성

**요청**: "User 모델의 스키마를 문서화해줘"

**실행**:

1. `rg -n "Schema::create\\('users'|Schema::table\\('users'" database/migrations` 실행
2. 해당 마이그레이션 파일을 열어 컬럼/인덱스 정보를 정리 (추가/변경 마이그레이션 포함)
3. `php artisan model:show User`로 관계/속성 정보를 확인
4. 결과를 `workspaces/schemas/users.md`로 정리
5. `workspaces/schemas/index.md`에 테이블 추가

## Model Naming Convention

| Model Class            | Table Name               | Schema File                 |
| ---------------------- | ------------------------ | --------------------------- |
| User                   | users                    | users.md                    |
| Organization           | organizations            | organizations.md            |
| Workspace              | workspaces               | workspaces.md               |
| BranchOperatingHour    | branch_operating_hours   | branch_operating_hours.md   |
| ProductCategoryProduct | product_category_product | product_category_product.md |

## Field Naming Convention (from AGENTS.md)

- `status` 필드: `{model}_status` 형태
- `name` 필드: `{model}_name` 형태
- 파일 경로 필드: `*_path` 접미사
- 외래키: `{related_table}_id` 형태
- 기본키: UUID v7

## Quality Checklist

스키마 문서 생성 후 다음을 확인한다:

- [ ] 모든 컬럼이 포함되었는가?
- [ ] 타입과 Nullable 정보가 정확한가?
- [ ] 인덱스 정보가 포함되었는가?
- [ ] 관계(Relationships)가 정확히 기술되었는가?
- [ ] 한국어 설명이 포함되었는가?
- [ ] `workspaces/schemas/index.md`에 테이블이 추가되었는가?

## Security & Consistency Rules

- DB 연결/조회 도구 사용 금지: `database-schema`, `database-query`, `db:table` 등은 사용하지 않는다.
- 스키마는 migrations와 `php artisan model:show` 결과만으로 정적 분석하여 작성한다.
- DB 드라이버별 스키마 차이를 피하기 위해 실제 DB에 접근하지 않는다.

## Related Skills

- [task-manager](../task-manager/SKILL.md): Task 분해 시 데이터 모델 설계 참고
- [prd-generator](../prd-generator/SKILL.md): PRD 작성 시 기존 스키마 참고

## Related Documentation

- `workspaces/schemas/`: Markdown 형식의 스키마 정의 파일
- `AGENTS.md`: 데이터베이스 네이밍 컨벤션 및 규칙
