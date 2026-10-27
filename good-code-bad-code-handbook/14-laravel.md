# Laravel — Code Review Handbook

> Review routes, controllers, Form Requests, middleware, container bindings, Eloquent, migrations, resources, policies, queues, and feature tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/laravel](https://good-code-bad-code.pages.dev/tracks/laravel)
- **Related tracks:** [PHP](13-php.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Routing and controller boundaries](#01-routing-and-controller-boundaries)** — Routes should describe how HTTP enters the app while controllers and services own the work behind that route.
- [ ] **[Form request validation](#02-form-request-validation)** — A controller action should work with validated input instead of reading unchecked request payloads directly.
- [ ] **[Middleware and auth boundaries](#03-middleware-and-auth-boundaries)** — Access and request-wide rules should be visible where the route is declared so protected endpoints are easy to audit.
- [ ] **[Service container bindings](#04-service-container-bindings)** — Application code should depend on contracts or services that can be replaced in tests and configured at provider boundaries.
- [ ] **[Eloquent query scopes](#05-eloquent-query-scopes)** — Use database queries for filtering and eager loading so request handlers do not pull more rows than they need.
- [ ] **[Migrations and schema constraints](#06-migrations-and-schema-constraints)** — Important data rules should be enforced by the database, not only remembered by controllers or jobs.
- [ ] **[API resources and response shape](#07-api-resources-and-response-shape)** — Responses should expose a deliberate shape instead of leaking every model field and relationship.
- [ ] **[Policies and authorization](#08-policies-and-authorization)** — Authorization should read as a named rule at the controller boundary instead of scattered ownership checks.
- [ ] **[Queues and database commits](#09-queues-and-database-commits)** — A queued job should not observe data that the current database transaction has not committed yet.
- [ ] **[Feature tests with database assertions](#10-feature-tests-with-database-assertions)** — A useful feature test proves both the HTTP result and the persistent behavior users depend on.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Routing and controller boundaries](#01-routing-and-controller-boundaries) | `routing` `controllers` `boundaries` |
| 02 | [Form request validation](#02-form-request-validation) | `requests` `validation` `input` |
| 03 | [Middleware and auth boundaries](#03-middleware-and-auth-boundaries) | `middleware` `auth` `routes` |
| 04 | [Service container bindings](#04-service-container-bindings) | `container` `providers` `dependency-injection` |
| 05 | [Eloquent query scopes](#05-eloquent-query-scopes) | `eloquent` `queries` `scopes` |
| 06 | [Migrations and schema constraints](#06-migrations-and-schema-constraints) | `migrations` `schema` `constraints` |
| 07 | [API resources and response shape](#07-api-resources-and-response-shape) | `api-resources` `responses` `serialization` |
| 08 | [Policies and authorization](#08-policies-and-authorization) | `authorization` `policies` `security` |
| 09 | [Queues and database commits](#09-queues-and-database-commits) | `queues` `transactions` `jobs` |
| 10 | [Feature tests with database assertions](#10-feature-tests-with-database-assertions) | `testing` `feature-tests` `database` |

---

## 01. Routing and controller boundaries

> Keep Laravel route files focused on URL wiring and let controllers hand request work to application services.

**Tags:** `routing` `controllers` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/routing-controller-boundaries)

### ✅ Good Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreReviewRequest;
use App\Services\ReviewService;
use Illuminate\Http\JsonResponse;

final class ReviewController
{
    public function __construct(private ReviewService $reviews) {}

    public function store(StoreReviewRequest $request): JsonResponse
    {
        // Controller maps HTTP input to an application service call.
        $review = $this->reviews->create($request->user(), $request->validated());

        return response()->json($review, 201);
    }
}
```

### ❌ Bad Code — `routes/web.php`

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Route;

Route::post('/reviews', function (Request $request) {
    // The route closure owns validation, persistence, and response shape.
    $data = $request->validate(['title' => 'required', 'body' => 'required']);

    $id = DB::table('reviews')->insertGetId([
        ...$data,
        'user_id' => $request->user()->id,
    ]);

    return ['id' => $id, ...$data];
});
```

### 🔍 What to review

- **Good Code:** The good version makes the route and controller boundary easy to scan: HTTP data is validated elsewhere, the user and input are passed to a service, and the response status is explicit.
- **Bad Code:** The bad version puts too much behavior in the route file. A reviewer has to audit validation, database writes, ownership, and response shape in the same closure.

### 📝 Review notes

In Laravel reviews, start in `routes/*` and ask whether each route only wires a URL to a controller action. When the route file starts describing the business workflow, the application boundary is too blurry.

### 💡 Takeaway

> **Routes should describe how HTTP enters the app while controllers and services own the work behind that route.**

---

## 02. Form request validation

> Move request validation into Form Request classes so controllers receive named, reviewed input shapes.

**Tags:** `requests` `validation` `input` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/form-request-validation)

### ✅ Good Code — `app/Http/Requests/StoreReviewRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class StoreReviewRequest extends FormRequest
{
    public function rules(): array
    {
        // Validation rules name the accepted request shape before controller code runs.
        return [
            'title' => ['required', 'string', 'max:120'],
            'body' => ['required', 'string', 'min:20'],
            'rating' => ['required', 'integer', 'between:1,5'],
        ];
    }
}
```

### ❌ Bad Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

public function store(Request $request)
{
    // Raw request data reaches persistence before one reviewed validation boundary.
    if (! $request->title || strlen($request->body ?? '') < 20) {
        abort(422);
    }

    return Review::create($request->all());
}
```

### 🔍 What to review

- **Good Code:** The good version gives reviewers one class where the accepted fields and rules are visible before controller code runs.
- **Bad Code:** The bad version mixes manual checks with `$request->all()`. That makes mass assignment, missing fields, and inconsistent error responses harder to catch.

### 📝 Review notes

Form Requests are useful when the same endpoint needs clear validation, authorization, or named input. During review, look for controller actions that read raw request values repeatedly and ask whether the accepted shape should be declared once.

### 💡 Takeaway

> **A controller action should work with validated input instead of reading unchecked request payloads directly.**

---

## 03. Middleware and auth boundaries

> Attach authentication, rate limits, and request guards at the route boundary instead of hiding them inside actions.

**Tags:** `middleware` `auth` `routes` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/middleware-auth-boundaries)

### ✅ Good Code — `routes/api.php`

```php
<?php

use App\Http\Controllers\ReviewController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum', 'throttle:reviews'])
    ->prefix('reviews')
    ->group(function () {
        // Middleware makes authentication and rate limits visible at the route boundary.
        Route::post('/', [ReviewController::class, 'store']);
        Route::delete('/{review}', [ReviewController::class, 'destroy']);
    });
```

### ❌ Bad Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

public function destroy(Request $request, Review $review)
{
    // Auth parsing inside the action hides which routes are protected.
    if ($request->bearerToken() !== config('services.admin_token')) {
        abort(401);
    }

    $review->delete();

    return response()->noContent();
}
```

### 🔍 What to review

- **Good Code:** The good version makes the security story visible before reading controller code. Reviewers can see which routes require an authenticated user and which rate limiter applies.
- **Bad Code:** The bad version hides authentication details in the action body. That makes route audits and behavior reuse fragile.

### 📝 Review notes

Use middleware for request-wide concerns such as auth, session, throttling, localization, and tenant context. If every action repeats a guard clause, move that rule to a route or middleware boundary.

### 💡 Takeaway

> **Access and request-wide rules should be visible where the route is declared so protected endpoints are easy to audit.**

---

## 04. Service container bindings

> Use Laravel's service container to declare infrastructure choices once and inject contracts into application services.

**Tags:** `container` `providers` `dependency-injection` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/service-container-binding)

### ✅ Good Code — `app/Providers/AppServiceProvider.php`

```php
<?php

namespace App\Providers;

use App\Billing\PaymentGateway;
use App\Billing\StripePaymentGateway;
use Illuminate\Support\ServiceProvider;

final class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // The container owns the infrastructure choice for the payment contract.
        $this->app->bind(PaymentGateway::class, StripePaymentGateway::class);
    }
}
```

### ❌ Bad Code — `app/Services/CheckoutService.php`

```php
<?php

final class CheckoutService
{
    public function charge(Order $order): Receipt
    {
        // Creating infrastructure here couples the use case to env and SDK setup.
        $client = new Stripe\StripeClient(config('services.stripe.secret'));

        return $client->charges->create([
            'amount' => $order->total_cents,
            'currency' => 'usd',
        ]);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version declares the concrete payment gateway at a provider boundary. Services can request the contract without knowing which SDK or credential setup is used.
- **Bad Code:** The bad version creates the SDK client inside the use case. That makes tests depend on configuration and pushes infrastructure choices into business code.

### 📝 Review notes

The Laravel service container is the review point for dependency ownership. When a class calls `new` for infrastructure, ask whether that collaborator should be injected and bound in a provider.

### 💡 Takeaway

> **Application code should depend on contracts or services that can be replaced in tests and configured at provider boundaries.**

---

## 05. Eloquent query scopes

> Put repeated Eloquent filters into query scopes and keep collection filtering out of request paths.

**Tags:** `eloquent` `queries` `scopes` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/eloquent-query-scopes)

### ✅ Good Code — `app/Models/Review.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

final class Review extends Model
{
    public function scopePublished(Builder $query): Builder
    {
        // The database owns the repeated published-review filter.
        return $query->where('status', 'published')
            ->whereNotNull('published_at');
    }
}
```

### ❌ Bad Code — `app/Http/Controllers/ReviewFeedController.php`

```php
<?php

public function __invoke()
{
    // Pulling every row first makes filtering and lazy loading happen in PHP.
    return Review::all()
        ->filter(fn (Review $review) => $review->status === 'published')
        ->map(fn (Review $review) => [$review->title, $review->author->name]);
}
```

### 🔍 What to review

- **Good Code:** The good version keeps a repeated filter close to the model and lets callers compose it with eager loading, ordering, or pagination.
- **Bad Code:** The bad version fetches all rows and filters in memory. It also risks hidden N+1 queries through lazy-loaded relations.

### 📝 Review notes

In Laravel reviews, look for `all()->filter()` in request paths. If the database can filter, sort, or limit the rows, the Eloquent query should say so before data becomes a collection.

### 💡 Takeaway

> **Use database queries for filtering and eager loading so request handlers do not pull more rows than they need.**

---

## 06. Migrations and schema constraints

> Use migrations to encode database rules such as foreign keys, required fields, uniqueness, and timestamps.

**Tags:** `migrations` `schema` `constraints` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/migrations-schema-constraints)

### ✅ Good Code — `database/migrations/create_reviews_table.php`

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('reviews', function (Blueprint $table) {
    // Constraints protect core review data even when writes come from jobs or imports.
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->string('slug');
    $table->string('status')->default('draft');
    $table->text('body');
    $table->timestamps();

    $table->unique(['user_id', 'slug']);
});
```

### ❌ Bad Code — `database/migrations/create_reviews_table.php`

```php
<?php

Schema::create('reviews', function (Blueprint $table) {
    // Nullable strings leave core data rules outside the database.
    $table->id();
    $table->integer('user_id')->nullable();
    $table->string('slug')->nullable();
    $table->string('status')->nullable();
    $table->text('body')->nullable();
});
```

### 🔍 What to review

- **Good Code:** The good version gives the database enough information to reject orphaned reviews, duplicated slugs per user, and missing required fields.
- **Bad Code:** The bad version treats every field as optional and omits relationships. Any importer, queue job, or admin script can insert invalid rows.

### 📝 Review notes

Migrations are part of application behavior. When the app depends on a relationship or required field, check that the migration enforces it instead of trusting every write path to remember the rule.

### 💡 Takeaway

> **Important data rules should be enforced by the database, not only remembered by controllers or jobs.**

---

## 07. API resources and response shape

> Use API Resources to make public response fields explicit and keep model internals out of JSON responses.

**Tags:** `api-resources` `responses` `serialization` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/api-resources-response-shape)

### ✅ Good Code — `app/Http/Resources/ReviewResource.php`

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

final class ReviewResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        // The resource owns the public JSON contract for a review.
        return [
            'id' => $this->id,
            'title' => $this->title,
            'rating' => $this->rating,
            'author' => $this->author->only(['id', 'name']),
        ];
    }
}
```

### ❌ Bad Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

public function index()
{
    // Returning models directly exposes fields whenever the model changes.
    return Review::with('author')->latest()->get();
}
```

### 🔍 What to review

- **Good Code:** The good version names the response fields in a Resource class. The controller can return a stable API contract while the model stays free to change internally.
- **Bad Code:** The bad version returns Eloquent models directly. New columns, hidden fields, or relationships can change the API without a focused review.

### 📝 Review notes

Review API endpoints from the client's contract backward. If a response is public or shared across teams, prefer an API Resource so field names and nested shapes are deliberate.

### 💡 Takeaway

> **Responses should expose a deliberate shape instead of leaking every model field and relationship.**

---

## 08. Policies and authorization

> Use policies or gates for model-level authorization so access rules are named, reusable, and testable.

**Tags:** `authorization` `policies` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/policies-authorization)

### ✅ Good Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Review;
use Illuminate\Support\Facades\Gate;

final class ReviewController
{
    public function update(UpdateReviewRequest $request, Review $review)
    {
        // A named policy decides whether this user may update the review.
        Gate::authorize('update', $review);

        $review->update($request->validated());

        return $review->fresh();
    }
}
```

### ❌ Bad Code — `app/Http/Controllers/ReviewController.php`

```php
<?php

public function update(Request $request, Review $review)
{
    // Inline ownership checks drift when other actions need the same rule.
    if ($request->user()->id !== $review->user_id && ! $request->user()->is_admin) {
        abort(403);
    }

    $review->update($request->all());

    return $review;
}
```

### 🔍 What to review

- **Good Code:** The good version makes authorization visible as a named policy decision before mutation. That rule can be reused by controllers, jobs, or tests.
- **Bad Code:** The bad version repeats an ownership check inline and mixes authorization with unchecked input writes.

### 📝 Review notes

When reviewing protected model actions, search for policy or gate calls near the top of the action. Repeated `user_id` comparisons are a sign that the access rule needs a name.

### 💡 Takeaway

> **Authorization should read as a named rule at the controller boundary instead of scattered ownership checks.**

---

## 09. Queues and database commits

> Dispatch queued work after database commits when jobs depend on rows created inside a transaction.

**Tags:** `queues` `transactions` `jobs` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/queues-after-commit)

### ✅ Good Code — `app/Services/ReviewPublishingService.php`

```php
<?php

use App\Jobs\SendReviewPublishedMail;
use Illuminate\Support\Facades\DB;

final class ReviewPublishingService
{
    public function publish(Review $review): void
    {
        DB::transaction(function () use ($review) {
            // The job runs after the review update is committed.
            $review->update(['status' => 'published']);

            SendReviewPublishedMail::dispatch($review)->afterCommit();
        });
    }
}
```

### ❌ Bad Code — `app/Services/ReviewPublishingService.php`

```php
<?php

DB::transaction(function () use ($review) {
    // The job may read the review before this transaction commits.
    $review->update(['status' => 'published']);

    SendReviewPublishedMail::dispatch($review);
});
```

### 🔍 What to review

- **Good Code:** The good version uses `afterCommit()` so queued mail reads committed data.
- **Bad Code:** The bad version dispatches inside the transaction without saying when the worker may run. A fast worker can read stale data or fail to find the row.

### 📝 Review notes

Queue reviews need a transaction check. If a job depends on rows created or updated in the same transaction, make the commit boundary explicit before dispatching work.

### 💡 Takeaway

> **A queued job should not observe data that the current database transaction has not committed yet.**

---

## 10. Feature tests with database assertions

> Write Laravel feature tests that hit the HTTP boundary and assert the database state that matters.

**Tags:** `testing` `feature-tests` `database` · [View on site](https://good-code-bad-code.pages.dev/tracks/laravel/feature-tests-database)

### ✅ Good Code — `tests/Feature/CreateReviewTest.php`

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class CreateReviewTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_review(): void
    {
        // The test proves the HTTP endpoint writes the expected review row.
        $user = User::factory()->create();

        $this->actingAs($user)
            ->postJson('/reviews', ['title' => 'Practical Laravel', 'body' => 'Clear examples help reviewers.', 'rating' => 5])
            ->assertCreated();

        $this->assertDatabaseHas('reviews', ['user_id' => $user->id, 'title' => 'Practical Laravel']);
    }
}
```

### ❌ Bad Code — `tests/Feature/CreateReviewTest.php`

```php
<?php

public function test_create_review(): void
{
    // A status-only test misses whether the review was saved correctly.
    $this->post('/reviews', [
        'title' => 'Practical Laravel',
        'body' => 'Clear examples help reviewers.',
    ])->assertOk();
}
```

### 🔍 What to review

- **Good Code:** The good version exercises the HTTP endpoint, authenticates a user, and asserts the database row that should exist after the request.
- **Bad Code:** The bad version only checks a status code. It could pass while the endpoint silently drops fields or writes the wrong owner.

### 📝 Review notes

Feature tests should match the promise of the route. If the endpoint creates or updates data, include a database assertion for the fields that would break real users if they were wrong.

### 💡 Takeaway

> **A useful feature test proves both the HTTP result and the persistent behavior users depend on.**

---

[← Back to index](README.md)
