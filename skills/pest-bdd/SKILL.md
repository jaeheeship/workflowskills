---
name: pest-bdd
description: |
  PHP Pest로 BDD 스타일 테스트를 작성하거나 리팩터링할 때 사용한다. 특히 "Pest", "BDD", "테스트 코드", "테스트 파일" 요청이 있거나, 테스트 파일 하나당 테스트 하나(1 test per file) 규칙이 필요한 경우에 사용한다.
---

# Pest BDD

PHP Pest에서 BDD 스타일 테스트를 작성한다. 모든 테스트 파일은 단 하나의 테스트만 포함해야 한다. 이는 실패 원인을 빠르게 좁히고, 시나리오를 독립적으로 관리하며, 리뷰와 유지보수를 단순화하기 위한 규칙이다.

## Workflow

1. 테스트 컨텍스트 파악

- 프로젝트가 Laravel인지 일반 PHP인지 확인한다.
- 테스트 대상 기능/행위를 한 문장으로 요약한다.

2. 테스트 파일 위치 결정

- Laravel: 다음 기준으로 `tests/Feature` 또는 `tests/Unit`을 선택한다.
  - Feature: Laravel 부팅이 필요한 경우, HTTP 요청/응답, 라우팅, 미들웨어, DB 사용 등 여러 컴포넌트가 함께 동작하는 시나리오
  - Unit: 단일 클래스/함수의 동작처럼 외부 의존성을 최소화한 순수 로직 테스트
- 그 외: 기존 테스트 구조를 우선 따르며, 없으면 `tests/` 아래에 목적에 맞게 디렉터리를 만든다.

3. 파일 이름과 테스트 이름 정의

- 파일은 행위 중심으로 이름을 짓고, 파일 하나당 테스트 하나만 둔다.
- 테스트 이름은 BDD 문장 형태로 `given/when/then`을 담아 명확하게 쓴다.

4. Pest 코드 작성

- `describe()` 또는 `it()`을 사용해 BDD 스타일로 작성한다.
- 필요하면 `beforeEach()`로 공통 준비를 둔다.
- 한 파일에 하나의 테스트만 유지한다. 추가 시나리오는 새 파일로 분리한다.

5. 테스트 데이터 세트 구성

- 테스트가 재현 가능하도록 데이터 세트를 명확히 구성한다.
- 데이터는 테스트 내부에서 로컬로 정의하거나, 공통 헬퍼/팩토리를 통해 준비한다.
- 데이터 세트는 최소 단위로 분리하고, 케이스 수가 늘어나면 별도 파일로 분리한다.
- 랜덤 데이터 사용 시 시드를 고정하고, 입력/출력 기대값을 명시한다.
- 외부 의존(DB, API, 파일)은 테스트 더블/fixtures로 대체한다.
- 재사용이 필요한 경우 클래스 기반 fixture를 만든다. 예: `datasets/orders/CreateValidatedOrder.php`
- payload 데이터(요청 입력)와 DB 데이터(저장/조회)는 역할이 다르므로 fixture를 분리한다.
- 모든 테스트에 공통인 준비 데이터만 `beforeEach`에 둔다. 테스트별 차이가 있으면 개별 테스트/데이터셋/fixture로 분리한다.
- 데이터 세트는 `datasets/` 하위에 도메인별로 분리해 관리한다. 예: `datasets/orders/`, `datasets/payments/`

## Dataset Example

```php
<?php

dataset('prices', [
  'single item' => [100, 100],
  'two items' => [150, 150],
  'empty cart' => [0, 0],
]);

it('given items when calculating then returns summed total', function ($input, $expected) {
  $cart = new Cart();
  $cart->add(new Item(price: $input));

  expect($cart->total())->toBe($expected);
})->with('prices');
```

```php
<?php

beforeEach(function () {
  $this->items = [new Item(price: 100), new Item(price: 50)];
});

it('given items when calculating then returns summed total', function () {
  $cart = new Cart();
  foreach ($this->items as $item) {
    $cart->add($item);
  }

  expect($cart->total())->toBe(150);
});
```

```php
<?php

use Orders\\Dataset\\CreateValidatedOrder;

it('given validated order when creating then persists', function () {
  $payload = (new CreateValidatedOrder())->make();

  $response = $this->postJson('/orders', $payload);

  $response->assertCreated();
});
```

6. 최소 실행 확인

- 변경한 테스트 파일이 기존 규칙을 지키는지 빠르게 확인한다.

## Output Rules

- 테스트 파일 하나당 테스트 하나만 포함한다.
- 테스트 이름은 BDD 문장 형태로 작성한다.
- 중복 시나리오가 필요하면 파일을 추가한다.

## Example

```php
<?php

beforeEach(function () {
  $this->cart = new Cart();
});

it('given items when calculating then returns summed total', function () {
  $this->cart->add(new Item(price: 100));
  $this->cart->add(new Item(price: 50));

  expect($this->cart->total())->toBe(150);
});
```
