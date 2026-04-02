---
name: action-class-generator
description: '액션 클래스 패턴으로 비즈니스 로직을 구현한다. 새로운 비즈니스 로직 작성, 컨트롤러나 Livewire 컴포넌트에서 로직 분리, 도메인별 기능 구현, 액션 클래스 생성·수정·리팩터링 시 활성화한다.'
---

# Action Class Pattern

액션 클래스는 이 프로젝트에서 비즈니스 로직을 캡슐화하는 핵심 패턴이다. 컨트롤러와 Livewire 컴포넌트는 HTTP 요청/응답 또는 UI 상호작용만 담당하고, 실제 로직은 액션 클래스에 위임한다.

## When to Apply

- 새로운 비즈니스 로직을 구현할 때
- 컨트롤러나 Livewire 컴포넌트에서 로직을 분리할 때
- 기존 액션 클래스를 수정·리팩터링할 때
- 여러 곳에서 재사용되는 로직을 작성할 때

## Directory Structure

```
//example
app/Actions/
├── AppContexts/          # 앱 컨텍스트 관련
│   └── SetUserAppContext.php
├── Auth/                 # 인증 관련
│   ├── Logout.php
│   └── SigninAndSyncUser.php
├── InboxItems/           # 알림 관련
│   └── FetchInboxItems.php
├── Products/             # 상품 관련
│   ├── FetchProduct.php
│   └── FetchProducts.php
├── Supports/             # 공통 유틸리티
│   └── ConvertJsonApiToArray.php
└── Users/                # 사용자 관련
    └── SyncUserAndTokenOnDevice.php
```

## Naming Conventions

### Class Name

동사형으로 작성한다. `{Verb}{Noun}` 패턴을 따른다.

| Prefix    | 용도                              | 예시                               |
| --------- | --------------------------------- | ---------------------------------- |
| `Get`     | 단일 리소스 조회 (캐시 포함 가능) | `GetBranch`                        |
| `Fetch`   | API에서 목록/데이터 조회          | `FetchProducts`, `FetchInboxItems` |
| `Create`  | 리소스 생성                       | `CreateNewUser`                    |
| `Delete`  | 리소스 삭제                       | `DeleteCartItem`                   |
| `Set`     | 상태/값 설정                      | `SetUserAppContext`                |
| `Switch`  | 컨텍스트 전환                     | `SwitchBranch`                     |
| `Sync`    | 외부 데이터 동기화                | `SyncUserAndTokenOnDevice`         |
| `Convert` | 데이터 변환                       | `ConvertJsonApiToArray`            |

### Namespace

`App\Actions\{Domain}\{ActionName}` — 도메인 디렉터리는 복수형 명사를 사용한다.

## Core Rules

1. **단일 public 메서드 `handle()`** — 모든 액션 클래스의 진입점이며 다른 public method는 갖을수 없다..
2. **동사형 클래스명** — 클래스가 무엇을 하는지 명확히 드러낸다.
3. **명시적 반환 타입** — `handle()` 및 모든 헬퍼 메서드에 반환 타입을 선언한다.
4. **PHPDoc 블록** — `handle()`과 헬퍼 메서드에 `@param`, `@return` 및 배열 shape 타입을 작성한다.
5. **헬퍼 메서드는 `private` 또는 `protected`** — 외부에서 직접 호출하지 않는 내부 로직을 분리한다.
6. **생성자는 의존성이 있을 때만** — 다른 액션이나 서비스를 주입할 때만 `__construct()`를 사용한다.
7. **무상태** — 인스턴스 변수에 상태를 보관하지 않는다 (재귀 추적 등 특수한 경우 제외).

## Examples

### Simple Action (의존성 없음)

```php
<?php

namespace App\Actions\CartItems;

use App\Enums\Api\V1\Endpoints\CartItems;

class DeleteCartItem
{
  public function handle($terminalId, $cartItemId): bool
  {
    $response = CartItems::Delete
      ->request()
      ->withToken(apiToken())
      ->segment(['terminal' => $terminalId, 'cartItem' => $cartItemId])
      ->send();

    if ($response->successful()) {
      return true;
    }

    return false;
  }
}
```

### Action with DTO Return (컬렉션 반환)

```php
<?php

namespace App\Actions\Products;

use App\Dtos\ProductDto;
use App\Enums\Api\V1\Endpoints\Products;
use Illuminate\Support\Collection;

class FetchProducts
{
  /**
   * 터미널의 상품 목록을 조회합니다.
   *
   * @param  string  $terminalId  터미널 ID (필수)
   * @param  string|null  $categoryId  카테고리 ID (선택)
   * @param  array<int, string>  $tagCodes  태그 코드 목록 (선택)
   * @param  int  $perPage  페이지당 항목 수 (기본값: 15)
   * @return Collection<ProductDto> 상품 목록
   */
  public function handle(
    string $terminalId,
    ?string $categoryId = null,
    array $tagCodes = [],
    int $perPage = 15
  ): Collection {
    return $this->fetchProductsFromApi($terminalId, $categoryId, $tagCodes, $perPage);
  }

  private function fetchProductsFromApi(/* ... */): Collection
  {
    // API 호출 및 데이터 변환 로직
  }

  private function formatProductData(array $product): ProductDto
  {
    // DTO 변환 로직
  }
}
```

### Action with Constructor Injection (다른 액션 조합)

```php
<?php

namespace App\Actions\Branches;

use App\Actions\AppContexts\SetUserAppContext;
use App\Dtos\UserAppContextDto;
use App\Enums\FieldUpdate;
use App\Models\User;
use Illuminate\Support\Facades\Auth;

class SwitchBranch
{
  public function __construct(protected GetBranch $getBranch, protected SetUserAppContext $setUserAppContext)
  {
  }

  /**
   * 사용자의 현재 브랜치를 변경합니다.
   *
   * @param  string  $branchId  변경할 브랜치 ID
   * @return array|null 변경된 브랜치 정보
   */
  public function handle(string $branchId): ?array
  {
    $branchData = $this->getBranch->handle($branchId);

    if (!$branchData) {
      return null;
    }

    // 컨텍스트 업데이트 로직...

    return $branchData;
  }
}
```

### Action with Cache (캐시 활용)

```php
<?php

namespace App\Actions\Branches;

use App\Enums\Api\V1\Endpoints\Branches;
use Illuminate\Support\Facades\Cache;

class GetBranch
{
  /**
   * 단일 브랜치 정보를 조회합니다.
   *
   * @param  string  $branchId  브랜치 ID
   * @param  int  $cacheMinutes  캐시 저장 시간 (분, 기본값: 5분)
   * @return array|null 브랜치 정보 또는 null (실패시)
   */
  public function handle(string $branchId, int $cacheMinutes = 5): ?array
  {
    return Cache::remember($this->getCacheKey($branchId), now()->addMinutes($cacheMinutes), function () use (
      $branchId
    ) {
      return $this->fetchBranchFromApi($branchId);
    });
  }

  protected function fetchBranchFromApi(string $branchId): ?array
  {
    // API 호출 로직
  }

  protected function formatBranchData(array $branch): array
  {
    // 데이터 포맷팅 로직
  }

  protected function getCacheKey(string $branchId): string
  {
    return 'branch_' . $branchId;
  }
}
```

## Invocation Patterns

액션 클래스를 호출하는 방법은 다음 세 가지를 사용한다.

### 1. 직접 인스턴스화 (의존성 없는 경우 — 가장 일반적)

```php
// 단일 호출
new Logout()->handle();

// 인라인 호출
$result = (new ConvertJsonApiToArray)->handle($response->json());

// 변수에 할당 후 반복 호출
$action = new DeleteCartItem();
$action->handle($terminalId, $itemId);
```

### 2. 생성자 주입으로 조합 (다른 액션에 의존하는 경우)

```php
class SwitchBranch
{
  public function __construct(protected GetBranch $getBranch, protected SetUserAppContext $setUserAppContext)
  {
  }
}

// 테스트에서 수동 주입
$result = (new SwitchBranch($getBranch, app(SetUserAppContext::class)))->handle('branch-id');
```

### 3. 컨트롤러에서 타입 힌트 주입

```php
class CreateOrganizationController extends Controller
{
  public function __invoke(OrganizationCreateRequest $request, CreateOrganization $createOrganization)
  {
    $organization = $createOrganization->handle($request->validated());
    // ...
  }
}
```

## Return Value Conventions

| 상황             | 반환 타입               | 예시                                     |
| ---------------- | ----------------------- | ---------------------------------------- |
| 단일 리소스 조회 | `?array` 또는 `?Model`  | `GetBranch → ?array`                     |
| 컬렉션 조회      | `Collection` (DTO 사용) | `FetchProducts → Collection<ProductDto>` |
| 생성/수정        | `Model`                 | `SyncUserAndTokenOnDevice → User`        |
| 삭제/부수효과    | `bool` 또는 `void`      | `DeleteCartItem → bool`                  |
| 데이터 변환      | `array`                 | `ConvertJsonApiToArray → array`          |

## Creating a New Action Class

### Checklist

```
- [ ] 도메인 디렉터리 확인: app/Actions/{Domain}/
- [ ] 동사형 클래스명 결정
- [ ] handle() 메서드에 명시적 파라미터 타입과 반환 타입 선언
- [ ] PHPDoc 블록 작성 (@param, @return)
- [ ] 복잡한 로직은 private/protected 헬퍼 메서드로 분리
- [ ] 복잡한 반환 데이터는 DTO 클래스 사용 검토
- [ ] 다른 액션 의존 시 생성자 주입 사용
- [ ] 테스트 작성
```

### Step-by-Step

1. **도메인 파악** — 기존 `app/Actions/` 하위 디렉터리에 적합한 도메인이 있는지 확인한다. 없으면 새 도메인 디렉터리를 생성한다.
2. **클래스 생성** — `php artisan make:class Actions/{Domain}/{ActionName} --no-interaction`으로 생성한다.
   - 예시: `php artisan make:class Actions/Orders/CreateNewOrder --no-interaction`
3. **`handle()` 작성** — 파라미터와 반환 타입을 명시하고 PHPDoc을 추가한다.
4. **헬퍼 분리** — API 호출, 데이터 포맷팅, 유효성 검사 등은 별도 메서드로 분리한다.
5. **DTO 검토** — 반환 데이터가 복잡하면 `app/Dtos/` 에 DTO를 생성한다.
6. **테스트 작성** — `php artisan make:test --pest {ActionName}Test`로 테스트를 생성한다.

## Anti-Patterns (하지 말 것)

- ❌ `handle()` 외에 추가 public 메서드를 만들지 않는다.
- ❌ 액션 클래스 안에서 HTTP 응답(redirect, response)을 반환하지 않는다. 이는 컨트롤러의 역할이다.
- ❌ 빈 생성자를 만들지 않는다. 주입할 의존성이 없으면 생성자를 생략한다.
- ❌ 액션 클래스에 상태를 저장하지 않는다.
- ❌ 인라인 주석으로 코드를 설명하지 않는다. PHPDoc 블록을 사용한다.

## Testing Action Classes

액션 클래스 테스트는 Pest BDD 스타일로 작성한다. 파일 하나에 테스트 하나 규칙을 따른다.

```php
<?php

use App\Actions\Branches\SwitchBranch;
use App\Actions\Branches\GetBranch;
use App\Actions\AppContexts\SetUserAppContext;

describe('SwitchBranch Action', function () {
  it('updates app context when branch is switched', function () {
    // Given: 사용자와 브랜치 데이터가 준비되었을 때
    $branchData = ['id' => 'branch-1', 'branchName' => '테스트 매장'];

    $getBranch = Mockery::mock(GetBranch::class);
    $getBranch->shouldReceive('handle')->with('branch-1')->andReturn($branchData);

    // When: SwitchBranch 액션을 실행하면
    $result = (new SwitchBranch($getBranch, app(SetUserAppContext::class)))->handle('branch-1');

    // Then: 브랜치 데이터가 반환되어야 한다
    expect($result)->toBe($branchData);
  });
});
```
