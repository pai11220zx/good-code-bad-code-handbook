# Rust — Code Review Handbook

> Review ownership, borrowing, lifetimes, Result and Option boundaries, traits, async safety, Cargo features, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/rust](https://good-code-bad-code.pages.dev/tracks/rust)
- **Related tracks:** [C](17-c.md), [C++](18-cpp.md), [C#](19-csharp.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Ownership and borrowing](#01-ownership-and-borrowing)** — A Rust function should take ownership only when it stores, transforms, or consumes the value.
- [ ] **[Lifetimes for returned data](#02-lifetimes-for-returned-data)** — A returned reference should point to data owned outside the function, with a lifetime that makes that relationship visible.
- [ ] **[Result error boundaries](#03-result-error-boundaries)** — Rust boundary code should return typed errors instead of panicking on expected failure paths.
- [ ] **[Option instead of unwrap](#04-option-instead-of-unwrap)** — A Rust lookup that can miss should keep the miss path visible instead of unwrapping inside helper code.
- [ ] **[Pattern matching exhaustiveness](#05-pattern-matching-exhaustiveness)** — Rust enum reviews should prefer exhaustive matching over string flags or default branches that hide new states.
- [ ] **[Traits and generic contracts](#06-traits-and-generic-contracts)** — A Rust generic should declare the smallest trait contract needed by the function body.
- [ ] **[Iterator ownership](#07-iterator-ownership)** — A Rust loop should make collection ownership visible through the iterator method it chooses.
- [ ] **[Async Send and Sync boundaries](#08-async-send-and-sync-boundaries)** — Rust async review should check which values cross await points and task boundaries.
- [ ] **[Cargo feature dependencies](#09-cargo-feature-dependencies)** — A Rust crate should make optional integrations explicit through Cargo features and guarded code paths.
- [ ] **[Tests, fixtures, and assertions](#10-tests-fixtures-and-assertions)** — A Rust test should name the input fixture and assert the observable result that protects the review rule.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Ownership and borrowing](#01-ownership-and-borrowing) | `ownership` `borrowing` `boundaries` |
| 02 | [Lifetimes for returned data](#02-lifetimes-for-returned-data) | `lifetimes` `borrowing` `returns` |
| 03 | [Result error boundaries](#03-result-error-boundaries) | `result` `errors` `boundaries` |
| 04 | [Option instead of unwrap](#04-option-instead-of-unwrap) | `option` `unwrap` `missing-data` |
| 05 | [Pattern matching exhaustiveness](#05-pattern-matching-exhaustiveness) | `match` `enums` `states` |
| 06 | [Traits and generic contracts](#06-traits-and-generic-contracts) | `traits` `generics` `contracts` |
| 07 | [Iterator ownership](#07-iterator-ownership) | `iterators` `ownership` `collections` |
| 08 | [Async Send and Sync boundaries](#08-async-send-and-sync-boundaries) | `async` `send` `sync` |
| 09 | [Cargo feature dependencies](#09-cargo-feature-dependencies) | `cargo` `features` `dependencies` |
| 10 | [Tests, fixtures, and assertions](#10-tests-fixtures-and-assertions) | `tests` `fixtures` `assertions` |

---

## 01. Ownership and borrowing

> Use borrowed inputs when a function only reads data, so callers keep ownership of their values.

**Tags:** `ownership` `borrowing` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/ownership-borrowing)

### ✅ Good Code — `src/review_title.rs`

```rust
pub fn normalize_title(title: &str) -> String {
    // Borrowing says this function reads the title without consuming it.
    title.trim().replace(char::is_whitespace, " ")
}
```

### ❌ Bad Code — `review_title.rs`

```rust
pub fn normalize_title(title: String) -> String {
    // Taking String forces callers to give up ownership for a read-only task.
    title.trim().replace(char::is_whitespace, " ")
}
```

### 🔍 What to review

- **Good Code:** The good version accepts `&str`, which works with `String`, string slices, and literals without moving the caller's value. The function creates a new normalized value only for the returned title.
- **Bad Code:** The bad version takes `String` even though it only reads the input. Callers lose ownership and may clone strings just to keep using the title after the call.

### 📝 Review notes

In Rust reviews, start at the function signature. If the body only reads a value, prefer a borrowed parameter and let the return type show where new ownership begins.

### 💡 Takeaway

> **A Rust function should take ownership only when it stores, transforms, or consumes the value.**

---

## 02. Lifetimes for returned data

> Return borrowed data only when the lifetime is tied to an input, instead of returning references to local values.

**Tags:** `lifetimes` `borrowing` `returns` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/lifetimes-returned-data)

### ✅ Good Code — `src/review_lookup.rs`

```rust
pub fn first_approved<'a>(reviews: &'a [Review]) -> Option<&'a Review> {
    // The returned reference can only live as long as the input slice.
    reviews.iter().find(|review| review.approved)
}
```

### ❌ Bad Code — `review_lookup.rs`

```rust
pub fn first_approved_title(reviews: &[Review]) -> &str {
    let title = reviews
        .iter()
        .find(|review| review.approved)
        .map(|review| review.title.clone())
        .unwrap_or_default();

    // Returning a reference to local storage cannot outlive this function.
    title.as_str()
}
```

### 🔍 What to review

- **Good Code:** The good version returns a reference to an element already owned by the input slice. The lifetime in the signature documents that relationship.
- **Bad Code:** The bad version builds a local `String` and tries to return a borrowed `&str` from it. That reference would point to storage dropped at the end of the function.

### 📝 Review notes

When a Rust function returns a reference, ask which input owns the referenced data. If the answer is "a local variable", the function should return an owned value instead.

### 💡 Takeaway

> **A returned reference should point to data owned outside the function, with a lifetime that makes that relationship visible.**

---

## 03. Result error boundaries

> Return Result from fallible boundaries so callers can choose retry, rejection, or user-facing error copy.

**Tags:** `result` `errors` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/result-error-boundaries)

### ✅ Good Code — `src/review_loader.rs`

```rust
use std::fs;
use std::path::Path;

pub fn load_review(path: &Path) -> Result<String, std::io::Error> {
    // File errors stay in the return type for the caller to classify.
    fs::read_to_string(path)
}
```

### ❌ Bad Code — `review_loader.rs`

```rust
use std::fs;
use std::path::Path;

pub fn load_review(path: &Path) -> String {
    // unwrap turns a normal missing-file path into a process panic.
    fs::read_to_string(path).unwrap()
}
```

### 🔍 What to review

- **Good Code:** The good version returns `Result`, leaving the caller in control of logging, retrying, returning 404, or surfacing validation copy.
- **Bad Code:** The bad version uses `unwrap` at an I/O boundary. A missing file, permission error, or invalid path becomes a panic instead of a recoverable branch.

### 📝 Review notes

In Rust reviews, treat `unwrap` and `expect` as sharp tools. They can be fine in tests or impossible states, but fallible production boundaries should return `Result`.

### 💡 Takeaway

> **Rust boundary code should return typed errors instead of panicking on expected failure paths.**

---

## 04. Option instead of unwrap

> Keep missing data in Option until the boundary decides the fallback or rejection path.

**Tags:** `option` `unwrap` `missing-data` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/option-unwrap-boundaries)

### ✅ Good Code — `src/review_score.rs`

```rust
pub fn reviewer_score(reviews: &[Review], reviewer_id: u64) -> Option<u8> {
    // The caller decides what a missing reviewer means for the request.
    reviews
        .iter()
        .find(|review| review.reviewer_id == reviewer_id)
        .map(|review| review.score)
}
```

### ❌ Bad Code — `review_score.rs`

```rust
pub fn reviewer_score(reviews: &[Review], reviewer_id: u64) -> u8 {
    // unwrap hides the missing-review branch inside a helper.
    reviews
        .iter()
        .find(|review| review.reviewer_id == reviewer_id)
        .unwrap()
        .score
}
```

### 🔍 What to review

- **Good Code:** The good version returns `Option<u8>`, so the caller can return a default, produce a validation error, or keep the field absent.
- **Bad Code:** The bad version assumes the reviewer exists. A normal miss path becomes a panic far away from the HTTP, CLI, or job boundary that should choose the response.

### 📝 Review notes

For Rust lookup helpers, make absence part of the return type. Reviewers should be able to see where `None` becomes a domain decision.

### 💡 Takeaway

> **A Rust lookup that can miss should keep the miss path visible instead of unwrapping inside helper code.**

---

## 05. Pattern matching exhaustiveness

> Match enum states directly so new variants force a review of every state-dependent branch.

**Tags:** `match` `enums` `states` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/pattern-matching-exhaustiveness)

### ✅ Good Code — `src/review_status.rs`

```rust
pub enum ReviewStatus {
    Draft,
    Approved,
    Rejected { reason: String },
}

pub fn status_label(status: &ReviewStatus) -> &'static str {
    // Exhaustive match makes every status visible during review.
    match status {
        ReviewStatus::Draft => "draft",
        ReviewStatus::Approved => "approved",
        ReviewStatus::Rejected { .. } => "rejected",
    }
}
```

### ❌ Bad Code — `review_status.rs`

```rust
pub fn status_label(status: &str) -> &'static str {
    // String states let unknown values slip into the default branch.
    match status {
        "draft" => "draft",
        "approved" => "approved",
        _ => "unknown",
    }
}
```

### 🔍 What to review

- **Good Code:** The good version models status as an enum and matches every variant. Adding a new status creates a compiler-guided review point.
- **Bad Code:** The bad version uses string states and a broad default branch. A misspelled or newly added status becomes `"unknown"` without forcing a design decision.

### 📝 Review notes

Prefer enums for closed state sets. In review, broad `_` branches deserve extra scrutiny when each state should have a deliberate response.

### 💡 Takeaway

> **Rust enum reviews should prefer exhaustive matching over string flags or default branches that hide new states.**

---

## 06. Traits and generic contracts

> Use trait bounds to name the behavior a generic function needs instead of tying code to one concrete type.

**Tags:** `traits` `generics` `contracts` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/traits-and-generics)

### ✅ Good Code — `src/review_export.rs`

```rust
use std::fmt::Write;

pub fn append_review_line<W: Write>(writer: &mut W, review: &Review) -> std::fmt::Result {
    // The function asks only for the ability to receive formatted text.
    writeln!(writer, "{}:{}", review.id, review.title)
}
```

### ❌ Bad Code — `review_export.rs`

```rust
pub fn append_review_line(buffer: &mut String, review: &Review) {
    // Concrete String forces callers to allocate even when another writer fits.
    buffer.push_str(&format!("{}:{}\n", review.id, review.title));
}
```

### 🔍 What to review

- **Good Code:** The good version depends on `Write`, so tests can pass a `String`, production code can pass another formatter, and errors stay visible.
- **Bad Code:** The bad version hard-codes `String` and uses `format!` before appending. Callers with an existing writer cannot reuse the function without extra allocation.

### 📝 Review notes

Rust trait bounds are API documentation. Ask whether the bound names the behavior the function uses, or whether a concrete type has leaked into a reusable boundary.

### 💡 Takeaway

> **A Rust generic should declare the smallest trait contract needed by the function body.**

---

## 07. Iterator ownership

> Choose iter, iter_mut, or into_iter based on whether the code reads, mutates, or consumes collection items.

**Tags:** `iterators` `ownership` `collections` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/iterator-ownership)

### ✅ Good Code — `src/review_counts.rs`

```rust
pub fn approved_count(reviews: &[Review]) -> usize {
    // iter reads each review without moving the collection.
    reviews.iter().filter(|review| review.approved).count()
}
```

### ❌ Bad Code — `review_counts.rs`

```rust
pub fn approved_count(reviews: Vec<Review>) -> usize {
    // into_iter consumes the vector even though the function only counts.
    reviews.into_iter().filter(|review| review.approved).count()
}
```

### 🔍 What to review

- **Good Code:** The good version borrows a slice and uses `iter`, so callers keep the collection after the count.
- **Bad Code:** The bad version takes `Vec<Review>` and consumes it with `into_iter`. A read-only operation now forces ownership transfer and may cause clones at the call site.

### 📝 Review notes

For collection code, connect the signature to the iterator method. `iter` reads, `iter_mut` mutates, and `into_iter` consumes.

### 💡 Takeaway

> **A Rust loop should make collection ownership visible through the iterator method it chooses.**

---

## 08. Async Send and Sync boundaries

> Use thread-safe shared state around async tasks instead of holding non-Send values across await points.

**Tags:** `async` `send` `sync` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/async-send-sync-boundaries)

### ✅ Good Code — `src/review_cache.rs`

```rust
use std::sync::Arc;
use tokio::sync::RwLock;

pub async fn remember_review(cache: Arc<RwLock<Vec<Review>>>, review: Review) {
    // Async-aware locking keeps shared state safe across spawned tasks.
    let mut reviews = cache.write().await;
    reviews.push(review);
}
```

### ❌ Bad Code — `review_cache.rs`

```rust
use std::cell::RefCell;
use std::rc::Rc;

pub async fn remember_review(cache: Rc<RefCell<Vec<Review>>>, review: Review) {
    // Rc and RefCell are not safe to share across multi-threaded async tasks.
    cache.borrow_mut().push(review);
    send_audit_event().await;
}
```

### 🔍 What to review

- **Good Code:** The good version uses `Arc` and an async-aware lock, making the shared cache suitable for task boundaries that may move between worker threads.
- **Bad Code:** The bad version uses `Rc<RefCell<_>>`, then awaits after mutating shared state. That design breaks down when the future must be `Send` or when borrowing spans grow.

### 📝 Review notes

In Rust async code, review both the data type and the await points. Shared state needs types that match the executor and the task movement model.

### 💡 Takeaway

> **Rust async review should check which values cross await points and task boundaries.**

---

## 09. Cargo feature dependencies

> Keep optional dependencies behind named features so builds include only the integrations they need.

**Tags:** `cargo` `features` `dependencies` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/cargo-feature-dependencies)

### ✅ Good Code — `src/lib.rs`

```rust
#[cfg(feature = "postgres")]
pub mod postgres_store;

pub fn storage_name() -> &'static str {
    // Feature-gated code keeps the default build small and intentional.
    #[cfg(feature = "postgres")]
    {
        return "postgres";
    }

    "memory"
}
```

### ❌ Bad Code — `lib.rs`

```rust
pub mod postgres_store;
pub mod redis_store;

pub fn storage_name() -> &'static str {
    // Every integration is compiled even when the app uses only memory storage.
    "postgres"
}
```

### 🔍 What to review

- **Good Code:** The good version gates the optional store behind a named feature. A default build avoids pulling in database code unless the package enables that integration.
- **Bad Code:** The bad version always compiles every store module. Optional infrastructure becomes a hidden dependency for simple builds, tests, and local tools.

### 📝 Review notes

For Rust libraries, review `Cargo.toml` and `#[cfg]` together. The feature name should match the integration and the guarded code path should be easy to find.

### 💡 Takeaway

> **A Rust crate should make optional integrations explicit through Cargo features and guarded code paths.**

---

## 10. Tests, fixtures, and assertions

> Build small fixtures and assert domain outcomes instead of only checking that code does not panic.

**Tags:** `tests` `fixtures` `assertions` · [View on site](https://good-code-bad-code.pages.dev/tracks/rust/tests-fixtures-assertions)

### ✅ Good Code — `tests/review_rules.rs`

```rust
#[test]
fn rejects_empty_review_title() {
    // The fixture names the invalid input the rule must reject.
    let review = ReviewDraft {
        title: String::from("   "),
        body: String::from("Useful notes"),
    };

    let error = validate_review(&review).unwrap_err();

    assert_eq!(error.field(), "title");
}
```

### ❌ Bad Code — `tests/review_rules.rs`

```rust
#[test]
fn validates_review() {
    let review = ReviewDraft::default();

    // This assertion only proves the function returned some success value.
    assert!(validate_review(&review).is_ok());
}
```

### 🔍 What to review

- **Good Code:** The good version builds the invalid input directly and asserts the field reported by the validator. A future regression in title validation will fail the test.
- **Bad Code:** The bad version uses a default fixture and checks only a success path. The test name and assertion do not protect a specific rule.

### 📝 Review notes

Rust tests are strongest when the fixture is small and the assertion names the domain behavior. Avoid tests that only prove a function returned without panic.

### 💡 Takeaway

> **A Rust test should name the input fixture and assert the observable result that protects the review rule.**

---

[← Back to index](README.md)
