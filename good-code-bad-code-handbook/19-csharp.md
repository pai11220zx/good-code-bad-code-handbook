# C# — Code Review Handbook

> Review nullable references, records, async cancellation, dependency injection, disposal, exceptions, LINQ, pattern matching, options, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/csharp](https://good-code-bad-code.pages.dev/tracks/csharp)
- **Related tracks:** [C](17-c.md), [C++](18-cpp.md), [Rust](20-rust.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Nullable reference boundaries](#01-nullable-reference-boundaries)** — A C# method should make absence visible in its return type so callers choose the missing-data response at the boundary.
- [ ] **[Records and immutable DTOs](#02-records-and-immutable-dtos)** — DTOs are easier to review when construction and mutation are separate: map once, then pass a stable value.
- [ ] **[Async await and cancellation](#03-async-await-and-cancellation)** — Async C# code should carry `CancellationToken` through every operation that can wait on I/O.
- [ ] **[Dependency injection boundaries](#04-dependency-injection-boundaries)** — A C# service should reveal the collaborators it needs instead of constructing database, clock, or HTTP clients inside methods.
- [ ] **[Using declarations and disposal](#05-using-declarations-and-disposal)** — Disposable resources should have a scope-owned lifetime so exceptions and early returns still release them.
- [ ] **[Exceptions and result boundaries](#06-exceptions-and-result-boundaries)** — C# exception handling should preserve the failure reason until a boundary can map it to HTTP, UI, or logging.
- [ ] **[LINQ query intent](#07-linq-query-intent)** — LINQ works best when each step names a collection transformation, not when the query hides mutation or logging.
- [ ] **[Pattern matching switches](#08-pattern-matching-switches)** — A C# switch expression should show each known state and make unknown states visible instead of silently falling through.
- [ ] **[Options configuration](#09-options-configuration)** — Configuration should cross into services as typed values that were validated during startup.
- [ ] **[Unit tests with arrange act assert](#10-unit-tests-with-arrange-act-assert)** — A C# test should name the behavior, create only the data it needs, execute one action, and assert the observable result.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Nullable reference boundaries](#01-nullable-reference-boundaries) | `nullable` `references` `boundaries` |
| 02 | [Records and immutable DTOs](#02-records-and-immutable-dtos) | `records` `immutability` `dto` |
| 03 | [Async await and cancellation](#03-async-await-and-cancellation) | `async` `cancellation` `await` |
| 04 | [Dependency injection boundaries](#04-dependency-injection-boundaries) | `dependency-injection` `services` `testing` |
| 05 | [Using declarations and disposal](#05-using-declarations-and-disposal) | `using` `disposal` `resources` |
| 06 | [Exceptions and result boundaries](#06-exceptions-and-result-boundaries) | `exceptions` `results` `boundaries` |
| 07 | [LINQ query intent](#07-linq-query-intent) | `linq` `queries` `collections` |
| 08 | [Pattern matching switches](#08-pattern-matching-switches) | `pattern-matching` `switch` `states` |
| 09 | [Options configuration](#09-options-configuration) | `options` `configuration` `validation` |
| 10 | [Unit tests with arrange act assert](#10-unit-tests-with-arrange-act-assert) | `tests` `assertions` `aaa` |

---

## 01. Nullable reference boundaries

> Use nullable reference types to show when a lookup can miss, instead of returning a non-null type that may hide null.

**Tags:** `nullable` `references` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/nullable-reference-boundaries)

### ✅ Good Code — `src/ReviewService.cs`

```csharp
public sealed class ReviewService
{
    private readonly ReviewRepository reviews;

    public ReviewService(ReviewRepository reviews)
    {
        this.reviews = reviews;
    }

    public async Task<ReviewDto?> FindAsync(
        Guid id,
        CancellationToken cancellationToken)
    {
        // Nullable return type tells callers the review may be missing.
        Review? review = await reviews.FindAsync(id, cancellationToken);
        return review is null ? null : ReviewDto.From(review);
    }
}
```

### ❌ Bad Code — `ReviewService.cs`

```csharp
public sealed class ReviewService
{
    private readonly ReviewRepository reviews;

    public ReviewDto Find(string id)
    {
        // Non-null return type hides the missing-review path.
        var review = reviews.Find(id);
        return new ReviewDto(review.Title, review.Score);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version returns `ReviewDto?`, making the miss path part of the contract. The caller must decide whether to return 404, show empty state, or continue.
- **Bad Code:** The bad version promises a non-null DTO but depends on a repository result that may be null. The failure appears later as a `NullReferenceException`.

### 📝 Review notes

In C# reviews, nullable annotations are boundary documentation. If a value can be absent, the type should say so where the value crosses layers.

### 💡 Takeaway

> **A C# method should make absence visible in its return type so callers choose the missing-data response at the boundary.**

---

## 02. Records and immutable DTOs

> Use records or init-only properties for response shapes that should not change after mapping.

**Tags:** `records` `immutability` `dto` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/records-immutability)

### ✅ Good Code — `src/ReviewSummary.cs`

```csharp
public sealed record ReviewSummary(
    Guid Id,
    string Title,
    int Score
)
{
    // Record data stays stable after response mapping.
    public bool IsPassing => Score >= 70;
}
```

### ❌ Bad Code — `ReviewSummary.cs`

```csharp
public sealed class ReviewSummary
{
    public Guid Id { get; set; }
    public string Title { get; set; } = "";
    public int Score { get; set; }

    // Public setters let later code rewrite the response shape.
    public bool IsPassing => Score >= 70;
}
```

### 🔍 What to review

- **Good Code:** The good version uses a record so the response shape is constructed with all required values and then stays stable.
- **Bad Code:** The bad version exposes public setters for every field. Any later code can rewrite a response after validation, mapping, or logging.

### 📝 Review notes

For C# DTOs, ask whether the object is a value to pass around or a mutable workflow object. Response DTOs usually benefit from records or init-only properties.

### 💡 Takeaway

> **DTOs are easier to review when construction and mutation are separate: map once, then pass a stable value.**

---

## 03. Async await and cancellation

> Thread cancellation tokens through async calls so abandoned requests can stop database, HTTP, or queue work.

**Tags:** `async` `cancellation` `await` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/async-await-cancellation)

### ✅ Good Code — `src/ReviewController.cs`

```csharp
public sealed class ReviewController
{
    public async Task<IResult> GetAsync(
        Guid id,
        CancellationToken cancellationToken)
    {
        // Cancellation flows from the request into the service.
        ReviewDto? review = await reviews.FindAsync(id, cancellationToken);
        return review is null ? Results.NotFound() : Results.Ok(review);
    }
}
```

### ❌ Bad Code — `ReviewController.cs`

```csharp
public sealed class ReviewController
{
    public IResult Get(Guid id)
    {
        // Blocking on async work can tie up request threads.
        ReviewDto? review = reviews.FindAsync(id).Result;
        return review is null ? Results.NotFound() : Results.Ok(review);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version accepts a `CancellationToken` from the request and passes it into the service call. If the client disconnects, downstream work can stop.
- **Bad Code:** The bad version blocks on `.Result`, loses cancellation, and can exhaust request threads under load.

### 📝 Review notes

Review async code as a chain. `await` should stay async all the way down, and cancellation should move through database, HTTP, queue, and file calls that support it.

### 💡 Takeaway

> **Async C# code should carry `CancellationToken` through every operation that can wait on I/O.**

---

## 04. Dependency injection boundaries

> Receive collaborators through constructors so services declare their dependencies and tests can replace infrastructure.

**Tags:** `dependency-injection` `services` `testing` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/dependency-injection-boundaries)

### ✅ Good Code — `src/ReviewPublisher.cs`

```csharp
public sealed class ReviewPublisher
{
    private readonly ReviewRepository reviews;
    private readonly MessageBus bus;

    public ReviewPublisher(ReviewRepository reviews, MessageBus bus)
    {
        this.reviews = reviews;
        this.bus = bus;
    }

    public async Task PublishAsync(Guid id, CancellationToken cancellationToken)
    {
        // Constructor dependencies make infrastructure visible.
        Review review = await reviews.GetAsync(id, cancellationToken);
        await bus.PublishAsync(new ReviewPublished(id, review.Title), cancellationToken);
    }
}
```

### ❌ Bad Code — `ReviewPublisher.cs`

```csharp
public sealed class ReviewPublisher
{
    public async Task PublishAsync(Guid id)
    {
        // New infrastructure inside the method hides dependencies.
        var reviews = new ReviewRepository("prod");
        var bus = new MessageBus("prod");
        Review review = await reviews.GetAsync(id);
        await bus.PublishAsync(new ReviewPublished(id, review.Title));
    }
}
```

### 🔍 What to review

- **Good Code:** The good version receives repository and bus dependencies through the constructor. The service boundary says what infrastructure the workflow needs.
- **Bad Code:** The bad version creates production infrastructure inside the method. Tests cannot replace those collaborators without touching the implementation.

### 📝 Review notes

Dependency injection is not only a framework feature. It is a review signal that tells readers which parts are business flow and which parts are infrastructure.

### 💡 Takeaway

> **A C# service should reveal the collaborators it needs instead of constructing database, clock, or HTTP clients inside methods.**

---

## 05. Using declarations and disposal

> Use using or await using to bind disposable resources to scope instead of relying on manual cleanup.

**Tags:** `using` `disposal` `resources` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/using-disposal)

### ✅ Good Code — `src/ReviewExport.cs`

```csharp
public async Task ExportAsync(
    string path,
    IReadOnlyCollection<ReviewDto> reviews,
    CancellationToken cancellationToken)
{
    await using FileStream file = File.Create(path);
    await using var writer = new StreamWriter(file);

    // await using disposes the writer and stream on every exit path.
    foreach (ReviewDto review in reviews)
    {
        await writer.WriteLineAsync(review.Title.AsMemory(), cancellationToken);
    }
}
```

### ❌ Bad Code — `ReviewExport.cs`

```csharp
public async Task ExportAsync(string path, IReadOnlyCollection<ReviewDto> reviews)
{
    var file = File.Create(path);
    var writer = new StreamWriter(file);

    foreach (ReviewDto review in reviews)
    {
        // An exception skips both Dispose calls below.
        await writer.WriteLineAsync(review.Title);
    }

    writer.Dispose();
    file.Dispose();
}
```

### 🔍 What to review

- **Good Code:** The good version ties the file stream and writer to scope with `await using`. Disposal still runs if a write throws.
- **Bad Code:** The bad version calls `Dispose` manually after the loop. Any exception before those calls leaves resources open until finalization.

### 📝 Review notes

Review disposable lifetimes like transaction lifetimes. The code that acquires a resource should make the release path visible in the same scope.

### 💡 Takeaway

> **Disposable resources should have a scope-owned lifetime so exceptions and early returns still release them.**

---

## 06. Exceptions and result boundaries

> Translate domain exceptions at application boundaries instead of catching everything and returning vague results.

**Tags:** `exceptions` `results` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/exceptions-result-boundaries)

### ✅ Good Code — `src/ReviewEndpoint.cs`

```csharp
public async Task<IResult> GetAsync(Guid id, CancellationToken cancellationToken)
{
    try
    {
        ReviewDto review = await reviews.GetRequiredAsync(id, cancellationToken);
        return Results.Ok(review);
    }
    catch (ReviewNotFoundException)
    {
        // Domain exception maps to the HTTP boundary.
        return Results.NotFound();
    }
}
```

### ❌ Bad Code — `ReviewEndpoint.cs`

```csharp
public async Task<IResult> GetAsync(Guid id)
{
    try
    {
        return Results.Ok(await reviews.GetRequiredAsync(id));
    }
    catch (Exception)
    {
        // Catching every exception turns many failures into 404.
        return Results.NotFound();
    }
}
```

### 🔍 What to review

- **Good Code:** The good version maps a named domain exception to a 404. Other exceptions keep their meaning and can travel to logging or middleware.
- **Bad Code:** The bad version catches every exception and returns 404. A database outage, serialization bug, or programming error becomes indistinguishable from a missing review.

### 📝 Review notes

Catch exceptions where you can translate them honestly. If the code cannot name the exception type, it probably cannot choose the right user-facing result.

### 💡 Takeaway

> **C# exception handling should preserve the failure reason until a boundary can map it to HTTP, UI, or logging.**

---

## 07. LINQ query intent

> Use LINQ when the chain reads as a clear query, and avoid mixing filtering with side effects.

**Tags:** `linq` `queries` `collections` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/linq-query-intent)

### ✅ Good Code — `src/ReviewQueries.cs`

```csharp
public static IReadOnlyList<ReviewSummary> PassingReviews(
    IEnumerable<Review> reviews)
{
    // LINQ reads as filter, sort, then project.
    return reviews
        .Where(review => review.Score >= 70)
        .OrderByDescending(review => review.Score)
        .Select(review => ReviewSummary.From(review))
        .ToList();
}
```

### ❌ Bad Code — `ReviewQueries.cs`

```csharp
public static IReadOnlyList<ReviewSummary> PassingReviews(IEnumerable<Review> reviews)
{
    var audit = new List<string>();

    // Side effects inside LINQ make the query hard to reason about.
    return reviews
        .Where(review => { audit.Add(review.Id.ToString()); return review.Score >= 70; })
        .Select(review => ReviewSummary.From(review))
        .ToList();
}
```

### 🔍 What to review

- **Good Code:** The good version reads as a query: filter, sort, project, materialize. Each LINQ method has one visible purpose.
- **Bad Code:** The bad version hides audit mutation inside `Where`. The expression looks like a query but also writes to an external collection.

### 📝 Review notes

Use LINQ when it clarifies the data pipeline. If a step needs side effects, branching, or multiple named intermediate values, a loop can be easier to review.

### 💡 Takeaway

> **LINQ works best when each step names a collection transformation, not when the query hides mutation or logging.**

---

## 08. Pattern matching switches

> Use pattern matching to make state-specific branches explicit and keep default cases from hiding unhandled states.

**Tags:** `pattern-matching` `switch` `states` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/pattern-matching-switches)

### ✅ Good Code — `src/ReviewStatusText.cs`

```csharp
public static string LabelFor(ReviewStatus status)
{
    return status switch
    {
        ReviewStatus.Draft => "Draft",
        ReviewStatus.Published => "Published",
        ReviewStatus.Archived => "Archived",
        // Unknown enum values fail loudly during review and tests.
        _ => throw new ArgumentOutOfRangeException(nameof(status), status, null)
    };
}
```

### ❌ Bad Code — `ReviewStatusText.cs`

```csharp
public static string LabelFor(ReviewStatus status)
{
    switch (status)
    {
        case ReviewStatus.Published:
            return "Published";
        default:
            // Default hides Draft, Archived, and future states.
            return "Draft";
    }
}
```

### 🔍 What to review

- **Good Code:** The good version names every current status and throws for unknown values. New enum values become review-visible because tests or runtime fail fast.
- **Bad Code:** The bad version sends every non-published status to `Draft`. Future statuses can ship with the wrong label and no compiler or test signal.

### 📝 Review notes

Pattern matching is strongest when it makes state coverage obvious. Be careful with `default` branches that look convenient but hide future enum values.

### 💡 Takeaway

> **A C# switch expression should show each known state and make unknown states visible instead of silently falling through.**

---

## 09. Options configuration

> Bind configuration into validated options objects instead of reading string keys across business code.

**Tags:** `options` `configuration` `validation` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/options-configuration)

### ✅ Good Code — `src/ReviewOptions.cs`

```csharp
public sealed class ReviewOptions
{
    public const string SectionName = "Reviews";
    public required Uri ApiBaseUrl { get; init; }
    public int PageSize { get; init; } = 50;
}

builder.Services
    .AddOptions<ReviewOptions>()
    .BindConfiguration(ReviewOptions.SectionName)
    .Validate(options => options.PageSize is >= 1 and <= 100)
    .ValidateOnStart();

// Validated options keep string keys out of service code.
```

### ❌ Bad Code — `ReviewService.cs`

```csharp
public sealed class ReviewService
{
    public ReviewService(IConfiguration configuration)
    {
        // String keys in services delay config errors until runtime calls.
        apiBaseUrl = new Uri(configuration["Reviews:ApiUrl"]!);
        pageSize = int.Parse(configuration["Reviews:PageSize"]!);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version binds a named section to `ReviewOptions` and validates it at startup. Services can receive typed values instead of raw configuration.
- **Bad Code:** The bad version reads string keys inside a service constructor. Missing or malformed config fails only when that service is built, and typos are hard to spot.

### 📝 Review notes

Configuration review should start at application startup. Ask whether required settings are typed, named, validated, and injected into the classes that need them.

### 💡 Takeaway

> **Configuration should cross into services as typed values that were validated during startup.**

---

## 10. Unit tests with arrange act assert

> Structure tests so setup, behavior, and assertions are separate enough for reviewers to see the contract under test.

**Tags:** `tests` `assertions` `aaa` · [View on site](https://good-code-bad-code.pages.dev/tracks/csharp/unit-tests-arrange-act-assert)

### ✅ Good Code — `tests/ReviewServiceTests.cs`

```csharp
[Fact]
public async Task FindAsyncReturnsNullWhenReviewIsMissing()
{
    var repository = new FakeReviewRepository();
    var service = new ReviewService(repository);

    // One act step makes the behavior under test easy to see.
    ReviewDto? result = await service.FindAsync(Guid.NewGuid(), CancellationToken.None);

    Assert.Null(result);
}
```

### ❌ Bad Code — `tests/ReviewServiceTests.cs`

```csharp
[Fact]
public async Task TestReviewService()
{
    var service = new ReviewService(new FakeReviewRepository());
    await service.FindAsync(Guid.NewGuid(), CancellationToken.None);
    await service.FindAsync(Guid.NewGuid(), CancellationToken.None);

    // No assertion means the test only proves the calls did not throw.
}
```

### 🔍 What to review

- **Good Code:** The good version names the missing-review behavior, performs one action, and asserts the null result.
- **Bad Code:** The bad version has a vague test name, calls the method twice, and contains no assertion. It can pass while the method returns the wrong value.

### 📝 Review notes

For C# tests, prefer one behavior per test. If the test name needs “and”, split it; if there is no assertion, the contract is not pinned.

### 💡 Takeaway

> **A C# test should name the behavior, create only the data it needs, execute one action, and assert the observable result.**

---

[← Back to index](README.md)
