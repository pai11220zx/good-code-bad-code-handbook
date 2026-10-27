# Kotlin — Code Review Handbook

> Review null safety, data classes, sealed states, scope functions, collections, coroutines, Flow, JVM interop, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/kotlin](https://good-code-bad-code.pages.dev/tracks/kotlin)
- **Related tracks:** [Java](15-java.md), [Go](25-go.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Null safety boundaries](#01-null-safety-boundaries)** — Kotlin review should make nullable contracts visible where request or Java data enters the code.
- [ ] **[Data class immutability](#02-data-class-immutability)** — Kotlin data classes should protect value state from caller mutation after construction.
- [ ] **[Sealed result states](#03-sealed-result-states)** — A sealed result lets reviewers see every branch the caller must render or log.
- [ ] **[Scope function intent](#04-scope-function-intent)** — Kotlin scope functions should reveal whether code transforms a value, configures an object, or records a side effect.
- [ ] **[Collection transformations](#05-collection-transformations)** — Kotlin collection review should connect `filter`, `map`, and `mapNotNull` to the domain rule being applied.
- [ ] **[Coroutine scope lifecycle](#06-coroutine-scope-lifecycle)** — Kotlin coroutine review should check which scope owns each child job and how cancellation reaches it.
- [ ] **[Flow state streams](#07-flow-state-streams)** — Kotlin Flow review should separate stream transformation from the lifecycle that collects it.
- [ ] **[Resource use boundaries](#08-resource-use-boundaries)** — Kotlin resource review should pair every opened stream with `use` or another explicit close path.
- [ ] **[JVM interop platform types](#09-jvm-interop-platform-types)** — Kotlin review should add a boundary check when Java APIs return values without nullability metadata.
- [ ] **[Kotest fixtures and assertions](#10-kotest-fixtures-and-assertions)** — Kotlin tests should assert the result shape that callers depend on, not only that a function returned a value.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Null safety boundaries](#01-null-safety-boundaries) | `null-safety` `boundaries` |
| 02 | [Data class immutability](#02-data-class-immutability) | `data-class` `immutability` |
| 03 | [Sealed result states](#03-sealed-result-states) | `sealed` `state` `results` |
| 04 | [Scope function intent](#04-scope-function-intent) | `scope-functions` `readability` |
| 05 | [Collection transformations](#05-collection-transformations) | `collections` `transformations` |
| 06 | [Coroutine scope lifecycle](#06-coroutine-scope-lifecycle) | `coroutines` `structured-concurrency` |
| 07 | [Flow state streams](#07-flow-state-streams) | `flow` `streams` `cancellation` |
| 08 | [Resource use boundaries](#08-resource-use-boundaries) | `resources` `use` `cleanup` |
| 09 | [JVM interop platform types](#09-jvm-interop-platform-types) | `jvm` `interop` `platform-types` |
| 10 | [Kotest fixtures and assertions](#10-kotest-fixtures-and-assertions) | `tests` `kotest` `assertions` |

---

## 01. Null safety boundaries

> Keep nullable values at input boundaries instead of spreading !! through service code.

**Tags:** `null-safety` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/null-safety-boundaries)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewService.kt`

```kotlin
data class Review(val title: String)

fun createReview(title: String?): Review? {
    // Nullable input is checked before service code builds the Review.
    val normalized = title?.trim()?.takeIf { it.isNotEmpty() } ?: return null
    return Review(normalized)
}
```

### ❌ Bad Code — `ReviewService.kt`

```kotlin
data class Review(val title: String)

fun createReview(title: String?): Review {
    // The crash moves request validation into a hidden runtime branch.
    return Review(title!!.trim())
}
```

### 🔍 What to review

- **Good Code:** The good version turns a nullable title into a domain decision before the Review object exists.
- **Bad Code:** The bad version uses `!!`, so a missing title becomes a runtime crash instead of a validation result.

### 📝 Review notes

In Kotlin review, ask where `String?` becomes `String`. That conversion should be near the input boundary or Java interop boundary, not scattered through service code.

### 💡 Takeaway

> **Kotlin review should make nullable contracts visible where request or Java data enters the code.**

---

## 02. Data class immutability

> Use immutable data classes and copy when a request creates a new state.

**Tags:** `data-class` `immutability` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/data-class-immutability)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewDraft.kt`

```kotlin
data class ReviewDraft(
    val title: String,
    val tags: List<String>,
)

fun rename(draft: ReviewDraft, title: String): ReviewDraft {
    // copy returns the next draft without mutating the caller's value.
    return draft.copy(title = title.trim())
}
```

### ❌ Bad Code — `ReviewDraft.kt`

```kotlin
class ReviewDraft(
    var title: String,
    val tags: MutableList<String>,
)

fun rename(draft: ReviewDraft, title: String) {
    // Shared mutable state changes every reference to this draft.
    draft.title = title.trim()
}
```

### 🔍 What to review

- **Good Code:** The good version models a review draft as a value and returns the next state through `copy`.
- **Bad Code:** The bad version exposes `var` and `MutableList`, so later code can change the draft without passing through a named operation.

### 📝 Review notes

When a Kotlin type crosses service, controller, or UI boundaries, prefer `val` fields and read-only collection types. Mutation should appear as a named function that returns the next value or writes through a repository.

### 💡 Takeaway

> **Kotlin data classes should protect value state from caller mutation after construction.**

---

## 03. Sealed result states

> Represent closed workflow outcomes with sealed types instead of status strings.

**Tags:** `sealed` `state` `results` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/sealed-result-states)

### ✅ Good Code — `src/main/kotlin/reviews/PublishResult.kt`

```kotlin
sealed interface PublishResult {
    data object Published : PublishResult
    data class Rejected(val reason: String) : PublishResult
}

fun publish(review: Review): PublishResult {
    // The return type names every branch the caller must render.
    if (review.title.isBlank()) return PublishResult.Rejected("title_blank")
    return PublishResult.Published
}
```

### ❌ Bad Code — `PublishResult.kt`

```kotlin
fun publish(review: Review): String {
    // String status hides the allowed workflow states from callers.
    if (review.title.isBlank()) return "bad"
    return "ok"
}
```

### 🔍 What to review

- **Good Code:** The good version keeps Published and Rejected in the type system, so a `when` expression can cover every branch.
- **Bad Code:** The bad version returns ad hoc strings. A typo or new status can reach the caller without a compiler signal.

### 📝 Review notes

Use sealed interfaces or sealed classes when a workflow has a closed set of states. Review string status only when the value is truly external data and gets parsed at the boundary.

### 💡 Takeaway

> **A sealed result lets reviewers see every branch the caller must render or log.**

---

## 04. Scope function intent

> Use scope functions when the receiver and return value match the reader's expectation.

**Tags:** `scope-functions` `readability` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/scope-function-intent)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewFactory.kt`

```kotlin
fun buildReview(input: ReviewInput): Review {
    val title = input.title.trim()
    val review = Review(title = title, score = input.score)

    // also records the created review without replacing the return value.
    return review.also { createdReview -> audit.created(createdReview.id) }
}
```

### ❌ Bad Code — `ReviewFactory.kt`

```kotlin
fun buildReview(input: ReviewInput) =
    input.let {
        Review(it.title.trim(), it.score).apply {
            // Nested receivers hide whether id belongs to input or Review.
            audit.created(id)
        }
    }
```

### 🔍 What to review

- **Good Code:** The good version names the intermediate Review and uses `also` only for the audit side effect.
- **Bad Code:** The bad version nests `let` and `apply`, so `it`, `this`, and `id` need extra mental tracking before a reviewer can see the returned value.

### 📝 Review notes

Scope functions are useful when their intent is narrow: `let` transforms, `also` observes, `apply` configures the receiver. Nested receivers in business logic often need named variables instead.

### 💡 Takeaway

> **Kotlin scope functions should reveal whether code transforms a value, configures an object, or records a side effect.**

---

## 05. Collection transformations

> Prefer collection operations that name the data rule instead of manual mutable accumulation.

**Tags:** `collections` `transformations` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/collection-transformations)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewQueries.kt`

```kotlin
fun approvedTitles(reviews: List<Review>): List<String> {
    // The pipeline names the approval rule before selecting titles.
    return reviews
        .filter { review -> review.score >= 4 }
        .map { review -> review.title }
}
```

### ❌ Bad Code — `ReviewQueries.kt`

```kotlin
fun approvedTitles(reviews: List<Review>): List<String> {
    val titles = mutableListOf<String>()
    for (review in reviews) {
        // Mutation hides the selection rule inside loop plumbing.
        titles.add(review.title)
    }
    return titles
}
```

### 🔍 What to review

- **Good Code:** The good version puts the approval rule next to the transformation, so the output list follows from the pipeline.
- **Bad Code:** The bad version builds a mutable list and forgets the score check, making the loop mechanics louder than the domain rule.

### 📝 Review notes

Review collection code by naming the rule first: which items stay, which fields leave, and what happens to invalid data. Kotlin pipelines should make those steps visible without hiding expensive work in callbacks.

### 💡 Takeaway

> **Kotlin collection review should connect `filter`, `map`, and `mapNotNull` to the domain rule being applied.**

---

## 06. Coroutine scope lifecycle

> Tie child coroutines to the request or screen that started the work.

**Tags:** `coroutines` `structured-concurrency` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/coroutine-scope-lifecycle)

### ✅ Good Code — `src/main/kotlin/reviews/DigestService.kt`

```kotlin
suspend fun buildDigest(userId: UserId): Digest = coroutineScope {
    // Child jobs cancel when the digest request is cancelled.
    val user = async { users.find(userId) }
    val reviews = async { reviewStore.recentFor(userId) }

    Digest(user.await(), reviews.await())
}
```

### ❌ Bad Code — `DigestService.kt`

```kotlin
fun buildDigest(userId: UserId) {
    // GlobalScope outlives the request that triggered the digest.
    GlobalScope.launch {
        val reviews = reviewStore.recentFor(userId)
        mailer.sendDigest(userId, reviews)
    }
}
```

### 🔍 What to review

- **Good Code:** The good version uses `coroutineScope`, so async work belongs to the suspending request and cancellation reaches both child jobs.
- **Bad Code:** The bad version launches work in `GlobalScope`, so a cancelled request can leave mail work running without the caller seeing failure.

### 📝 Review notes

For Kotlin coroutine code, trace ownership of the scope before reading the body. A job should belong to a request, screen, service lifetime, or worker lifetime that appears in the code.

### 💡 Takeaway

> **Kotlin coroutine review should check which scope owns each child job and how cancellation reaches it.**

---

## 07. Flow state streams

> Keep Flow pipelines cancellable and let callers choose where collection starts.

**Tags:** `flow` `streams` `cancellation` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/flow-state-streams)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewEvents.kt`

```kotlin
fun approvalMessages(events: Flow<ReviewEvent>): Flow<String> {
    // The Flow pipeline leaves collection and cancellation with the caller.
    return events
        .filterIsInstance<ReviewApproved>()
        .map { event -> event.title }
}
```

### ❌ Bad Code — `ReviewEvents.kt`

```kotlin
val messages = mutableListOf<String>()

fun start(events: Flow<ReviewEvent>) {
    GlobalScope.launch {
        // Background collection outlives the screen or request that started it.
        events.collect { event -> messages += event.toString() }
    }
}
```

### 🔍 What to review

- **Good Code:** The good version returns a transformed Flow, leaving collection, cancellation, and error reporting to the caller's lifecycle.
- **Bad Code:** The bad version starts collection inside a helper and stores messages in shared mutable state, so shutdown and failure paths are hard to trace.

### 📝 Review notes

A Flow-returning function should usually describe stream shape. A screen, request, or worker should own the collection point and cancellation scope.

### 💡 Takeaway

> **Kotlin Flow review should separate stream transformation from the lifecycle that collects it.**

---

## 08. Resource use boundaries

> Close files and streams at the same boundary that opens them.

**Tags:** `resources` `use` `cleanup` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/resource-use-boundaries)

### ✅ Good Code — `src/main/kotlin/reviews/ReviewImport.kt`

```kotlin
fun readReviewLines(path: Path): List<String> {
    // use closes the reader when parsing succeeds or fails.
    return path.toFile().bufferedReader().use { reader ->
        reader.readLines()
    }
}
```

### ❌ Bad Code — `ReviewImport.kt`

```kotlin
fun readReviewLines(path: Path): List<String> {
    // The reader can stay open when readLines throws.
    val reader = path.toFile().bufferedReader()
    return reader.readLines()
}
```

### 🔍 What to review

- **Good Code:** The good version opens the reader and closes it through `use` in the same expression.
- **Bad Code:** The bad version returns before closing the reader. An exception while reading can leak the file descriptor.

### 📝 Review notes

When Kotlin code opens a file, socket, stream, or database cursor, review the failure path as well as the success path. `use` keeps cleanup next to the resource owner.

### 💡 Takeaway

> **Kotlin resource review should pair every opened stream with `use` or another explicit close path.**

---

## 09. JVM interop platform types

> Treat Java platform types as nullable until Kotlin code proves the contract.

**Tags:** `jvm` `interop` `platform-types` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/jvm-interop-platform-types)

### ✅ Good Code — `src/main/kotlin/reviews/JavaReviewAdapter.kt`

```kotlin
fun reviewerName(javaReview: JavaReview?): String? {
    // Java values are checked before Kotlin treats the name as non-null.
    val name = javaReview?.reviewerName ?: return null
    return name.trim().takeIf { it.isNotEmpty() }
}
```

### ❌ Bad Code — `JavaReviewAdapter.kt`

```kotlin
fun reviewerName(javaReview: JavaReview): String {
    // Platform types can still carry null from Java callers.
    return javaReview.reviewerName.trim()
}
```

### 🔍 What to review

- **Good Code:** The good version checks the Java object and the Java-provided name before returning a Kotlin string.
- **Bad Code:** The bad version trusts a platform type. If Java returns null, Kotlin code fails away from the interop boundary.

### 📝 Review notes

At Java interop boundaries, read platform types as untrusted input. Convert them into Kotlin nullable or non-null contracts before they enter service logic.

### 💡 Takeaway

> **Kotlin review should add a boundary check when Java APIs return values without nullability metadata.**

---

## 10. Kotest fixtures and assertions

> Write Kotlin tests that name the behavior, input fixture, and exact output contract.

**Tags:** `tests` `kotest` `assertions` · [View on site](https://good-code-bad-code.pages.dev/tracks/kotlin/kotest-fixtures-assertions)

### ✅ Good Code — `src/test/kotlin/reviews/ReviewValidatorTest.kt`

```kotlin
class ReviewValidatorTest : StringSpec({
    "rejects a blank title" {
        // The fixture names the invalid input and expected error code.
        val result = validate(ReviewInput(title = " ", score = 5))

        result shouldBe ValidationError("title_blank")
    }
})
```

### ❌ Bad Code — `ReviewValidatorTest.kt`

```kotlin
class ReviewValidatorTest : StringSpec({
    "validates reviews" {
        // Non-null assertion misses the validation contract callers use.
        validate(ReviewInput(title = "", score = 5)) shouldNotBe null
    }
})
```

### 🔍 What to review

- **Good Code:** The good version gives the invalid input and expected error code, so a validation regression fails at the rule being protected.
- **Bad Code:** The bad version only checks for a non-null result. It can pass while the validator returns the wrong error.

### 📝 Review notes

Kotlin test review should connect fixture, action, and assertion. If the assertion does not name the returned state, error code, or persisted row, the test may miss the behavior callers rely on.

### 💡 Takeaway

> **Kotlin tests should assert the result shape that callers depend on, not only that a function returned a value.**

---

[← Back to index](README.md)
