---
name: laravel-tdd
description: Test-driven development workflow for Laravel using PHPUnit and Pest, covering factories, database strategies, fakes, authorization, HTTP mocking and coverage targets. Use when writing or refactoring Laravel tests, adding endpoints or models, or fixing a bug test-first.
license: MIT
metadata:
  category: backend
  origin: ECC
---

# Laravel TDD Workflow

Test-driven development for Laravel applications, targeting 80%+ combined unit and feature coverage.

## Description

A red-green-refactor workflow for Laravel covering test layer selection, database strategy, model factories, side-effect faking, authorization assertions, external HTTP isolation and coverage enforcement. Examples are given in both PHPUnit and Pest.

## When to use

- Building a new Laravel feature, endpoint, job, notification or policy
- Fixing a bug, when a regression test should be written before the fix
- Refactoring code that already has test coverage
- Testing Eloquent models, factories, events or queued work

## Do not use when

- The user only asks to read or explain existing tests without changing them
- The change is documentation, comments or formatting only
- The task is a pure frontend concern with no server-side behaviour to assert

## Workflow

Copy this checklist and mark each step as it completes.

1. **Reproduce** - Write a failing test that describes the expected behaviour. Run it and confirm it fails for the right reason.
2. **Implement** - Make the smallest change that turns the test green.
3. **Verify** - Re-run the affected tests, then the surrounding suite.
4. **Refactor** - Clean up while the tests stay green. Re-run after each change.
5. **Report** - State which tests were added or updated and the command used to run them.

## Instructions

### Test layers

| Layer           | Covers                                           | Prefer for                                        |
| --------------- | ------------------------------------------------ | ------------------------------------------------- |
| **Unit**        | Pure PHP classes, value objects, services        | Business logic with no framework or DB dependency |
| **Feature**     | HTTP endpoints, auth, validation, response shape | Anything reachable through a route                |
| **Integration** | Database + queue + external boundaries together  | Cross-cutting behaviour, jobs, webhooks           |

Default to Feature tests. Add Unit tests only where logic is genuinely isolated from the framework.

### Framework selection

- Use **Pest** for new tests when the project already uses Pest.
- Use **PHPUnit** when the project is standardised on PHPUnit, or when PHPUnit-specific tooling is required.
- Never mix both styles inside one test file. Follow the convention already present in `tests/`.

### Database strategy

| Trait                  | Behaviour                                                                                                                                 | Use when                                                        |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `RefreshDatabase`      | Migrates once per run, then wraps each test in a transaction. Re-migrates per test on `:memory:` SQLite or non-transactional connections. | Default for any test touching the database                      |
| `DatabaseTransactions` | Rolls back after each test, no migration                                                                                                  | Schema is already migrated and you only need per-test isolation |
| `DatabaseMigrations`   | Full `migrate:fresh` before every test                                                                                                    | You need a guaranteed clean schema and can afford the cost      |

Configure `phpunit.xml` for fast in-memory runs:

```xml
<php>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
</php>
```

Keep a separate testing environment so dev and production data are never touched.

### Factories and states

Use factories for all test data. Never insert rows by hand.

```php
// Named state, defined once on the factory and reused
// database/factories/UserFactory.php
public function admin(): static
{
    return $this->state(fn (array $attributes): array => ['role' => 'admin']);
}

$user = User::factory()->admin()->create();
```

For a one-off override, pass attributes directly. `Factory::state()` also accepts a raw array or closure when an ad-hoc state is needed without defining a named one.

```php
$user = User::factory()->create(['role' => 'admin']);
```

Define named states for edge cases that recur across tests, such as `archived`, `admin` or `trial`.

### Assertions

Prefer framework assertions over manual queries.

```php
$this->assertDatabaseHas('projects', ['name' => 'New Project']);
$this->assertDatabaseMissing('projects', ['name' => 'Deleted']);
$this->assertSoftDeleted('projects', ['id' => $project->id]);
```

### Faking side effects

| Fake                   | Intercepts                                                                               |
| ---------------------- | ---------------------------------------------------------------------------------------- |
| `Bus::fake()`          | Jobs dispatched through the bus (`dispatch()`, `Bus::dispatch()`), including queued jobs |
| `Queue::fake()`        | Raw queue pushes (`Queue::push()`, `Queue::later()`)                                     |
| `Event::fake()`        | Domain events and their listeners                                                        |
| `Mail::fake()`         | Mailables                                                                                |
| `Notification::fake()` | Notifications on every channel                                                           |
| `Storage::fake()`      | Filesystem disks                                                                         |
| `Http::fake()`         | Outbound HTTP client requests                                                            |

Use `Bus::fake()` for job classes. Reach for `Queue::fake()` only when payloads are pushed directly onto a queue.

```php
use Illuminate\Support\Facades\Queue;

Queue::fake();

dispatch(new SendOrderConfirmation($order->id));

Queue::assertPushed(SendOrderConfirmation::class);
```

```php
use Illuminate\Support\Facades\Notification;

Notification::fake();

$user->notify(new InvoiceReady($invoice));

Notification::assertSentTo($user, InvoiceReady::class);
```

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake([
    'api.provider.com/v1/prices*' => Http::response(['price' => 1250], 200),
]);

app(PriceGateway::class)->fetch('WHEAT');

Http::assertSent(fn (Request $request): bool => $request->url() === 'https://api.provider.com/v1/prices/WHEAT'
    && $request->hasHeader('Accept', 'application/json'));
```

### Authentication

```php
// Session or token auth via the framework
$response = $this->actingAs($user)->getJson('/api/projects');

// Sanctum
use Laravel\Sanctum\Sanctum;
Sanctum::actingAs($user);

// Passport
use Laravel\Passport\Passport;
Passport::actingAs($user);
```

### Authorization

```php
use Illuminate\Support\Facades\Gate;

$this->assertTrue(Gate::forUser($owner)->allows('update', $project));
$this->assertFalse(Gate::forUser($stranger)->allows('update', $project));
```

Also assert the HTTP-level outcome, not only the policy. A 403 on the route is the behaviour users actually see.

```php
$this->actingAs($stranger)->patchJson("/api/projects/{$project->id}")->assertForbidden();
```

### Coverage

- Enforce 80%+ combined unit and feature coverage.
- Use `pcov` or `XDEBUG_MODE=coverage` in CI.

```bash
php artisan test --coverage --min=80
```

### Running tests

Run the narrowest scope while iterating, then widen before finishing.

```bash
php artisan test --filter=test_owner_can_create_project
php artisan test --filter=ProjectControllerTest
php artisan test
vendor/bin/phpunit
vendor/bin/pest
```

## Rules

- Always write the failing test first. Do not write implementation before a test exercises it.
- Every bug fix ships with a regression test that fails without the fix.
- Use `Model::query()` in assertions, never the `DB` facade.
- Keep tests isolated and deterministic. No shared mutable state between tests.
- Never let a test depend on wall-clock time. Use `$this->travelTo()` or `$this->freezeTime()`.
- Never commit real credentials or secrets into a test file. Use factories and fakes.
- Run only the affected tests during development, then offer to run the full suite.
- Never delete an existing test to make a suite pass.

## Examples

### PHPUnit feature test

```php
declare(strict_types=1);

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class ProjectControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_owner_can_create_project(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->postJson('/api/projects', [
            'name' => 'New Project',
        ]);

        $response->assertCreated();
        $this->assertDatabaseHas('projects', ['name' => 'New Project']);
    }

    public function test_name_is_required(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->postJson('/api/projects', []);

        $response->assertUnprocessable()->assertJsonValidationErrors(['name']);
        $this->assertDatabaseCount('projects', 0);
    }
}
```

### Pest feature test

```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

use function Pest\Laravel\actingAs;
use function Pest\Laravel\assertDatabaseHas;

uses(RefreshDatabase::class);

test('owner can create project', function (): void {
    $user = User::factory()->create();

    $response = actingAs($user)->postJson('/api/projects', [
        'name' => 'New Project',
    ]);

    $response->assertCreated();
    assertDatabaseHas('projects', ['name' => 'New Project']);
});
```

### Persistence test

```php
declare(strict_types=1);

namespace Tests\Feature;

use App\Models\Project;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class ProjectRepositoryTest extends TestCase
{
    use RefreshDatabase;

    public function test_project_can_be_retrieved_by_slug(): void
    {
        $project = Project::factory()->create(['slug' => 'alpha']);

        $found = Project::query()->where('slug', 'alpha')->firstOrFail();

        $this->assertSame($project->id, $found->id);
    }
}
```

### Paginated index test

```php
public function test_projects_index_returns_paginated_results(): void
{
    $user = User::factory()->create();
    Project::factory()->count(3)->for($user)->create();

    $response = $this->actingAs($user)->getJson('/api/projects');

    $response->assertOk();
    $response->assertJsonStructure(['data', 'links', 'meta']);
}
```

Adjust the asserted envelope to the project's actual API Resource shape.

### Inertia feature test

Prefer `assertInertia` over raw JSON assertions so tests track the Inertia response contract.

```php
declare(strict_types=1);

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Inertia\Testing\AssertableInertia;
use Tests\TestCase;

final class DashboardInertiaTest extends TestCase
{
    use RefreshDatabase;

    public function test_dashboard_inertia_props(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->get('/dashboard');

        $response->assertOk();
        $response->assertInertia(fn (AssertableInertia $page): AssertableInertia => $page
            ->component('Dashboard')
            ->where('user.id', $user->id)
            ->has('projects')
        );
    }
}
```

## Edge cases

- **Test fails with "table not found"** - The trait is missing or the migration is not in the default path. Confirm `RefreshDatabase` is on the test class and `php artisan migrate` runs clean.
- **Test passes alone but fails in the suite** - Shared state leaked between tests. Replace static properties and singletons with per-test setup, and add `RefreshDatabase` if it is absent.
- **Assertion on a faked job never fires** - `Bus::fake()` was called after the dispatch, or the job was dispatched through a path the fake does not intercept. Move the fake to the top of the test.
- **Coverage below threshold locally but not in CI** - The driver differs. Install `pcov` locally or run with `XDEBUG_MODE=coverage`.
- **Pest and PHPUnit tests coexist** - Check `phpunit.xml` `testsuites` includes both `tests/Unit` and `tests/Feature`, and that `pest.php` binds `TestCase` via `uses(Tests\TestCase::class)->in('Feature')`.

## References

- Laravel testing documentation: https://laravel.com/docs/testing
- PHPUnit documentation: https://docs.phpunit.de
- Pest Laravel plugin: https://pestphp.com/docs/plugins/laravel
