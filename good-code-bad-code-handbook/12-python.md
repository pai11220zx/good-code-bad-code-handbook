# Python — Code Review Handbook

> Review readable functions, data shapes, exceptions, typing, testability, and async boundaries.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/python](https://good-code-bad-code.pages.dev/tracks/python)
- **Related tracks:** [FastAPI](23-fastapi.md), [Django](24-django.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Naming and readability](#01-naming-and-readability)** — Readable Python starts with names that describe the domain, not the implementation shortcut.
- [ ] **[Truthy, falsy, and None checks](#02-truthy-falsy-and-none-checks)** — Use `is None` when absence is different from an empty value.
- [ ] **[Mutable default arguments](#03-mutable-default-arguments)** — Default arguments are evaluated once, so mutable defaults can leak state between calls.
- [ ] **[List and dict comprehensions](#04-list-and-dict-comprehensions)** — A comprehension should make simple filtering and mapping easier to read, not denser to parse.
- [ ] **[Exception boundaries](#05-exception-boundaries)** — Specific exceptions make failure modes reviewable; blanket catches hide the bug you need to see.
- [ ] **[Context managers for files](#06-context-managers-for-files)** — Resource lifetime should be visible in the code shape, not hidden in manual cleanup.
- [ ] **[Dataclasses for data shapes](#07-dataclasses-for-data-shapes)** — A named data shape makes fields, defaults, and equality behavior easier to review.
- [ ] **[Type hints at boundaries](#08-type-hints-at-boundaries)** — Type hints are most valuable at module boundaries, not as decoration on obvious local variables.
- [ ] **[Dependency injection for testability](#09-dependency-injection-for-testability)** — Injected dependencies make side effects explicit and keep tests focused on behavior.
- [ ] **[Async and await boundaries](#10-async-and-await-boundaries)** — Calling an async function creates work to await; it does not complete the work by itself.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Naming and readability](#01-naming-and-readability) | `naming` `readability` `functions` |
| 02 | [Truthy, falsy, and None checks](#02-truthy-falsy-and-none-checks) | `none` `conditionals` `truthiness` |
| 03 | [Mutable default arguments](#03-mutable-default-arguments) | `defaults` `mutation` `state` |
| 04 | [List and dict comprehensions](#04-list-and-dict-comprehensions) | `comprehensions` `collections` `readability` |
| 05 | [Exception boundaries](#05-exception-boundaries) | `exceptions` `errors` `boundaries` |
| 06 | [Context managers for files](#06-context-managers-for-files) | `files` `context managers` `resources` |
| 07 | [Dataclasses for data shapes](#07-dataclasses-for-data-shapes) | `dataclasses` `data` `models` |
| 08 | [Type hints at boundaries](#08-type-hints-at-boundaries) | `typing` `boundaries` `interfaces` |
| 09 | [Dependency injection for testability](#09-dependency-injection-for-testability) | `testing` `dependencies` `design` |
| 10 | [Async and await boundaries](#10-async-and-await-boundaries) | `async` `await` `concurrency` |

---

## 01. Naming and readability

> Use names and small functions that explain intent without forcing reviewers to decode abbreviations.

**Tags:** `naming` `readability` `functions` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/naming-and-readability)

### ✅ Good Code — `review_score.py`

```python
def calculate_review_score(review):
    # Names show which review signals affect the score.
    comment_score = len(review.comments) * 2
    approval_score = review.approvals * 5

    return comment_score + approval_score
```

### ❌ Bad Code — `review_score.py`

```python
def calc(x):
    # Short names force reviewers to decode the domain first.
    a = len(x.c) * 2
    b = x.a * 5

    return a + b
```

### 🔍 What to review

- **Good Code:** The good version names the function and intermediate values after the business idea, so the scoring rule can be read without reverse engineering attributes.
- **Bad Code:** The bad version hides the domain behind `calc`, `x`, `a`, `b`, and short attribute names. It may work, but the reviewer has to decode it before checking correctness.

### 📝 Review notes

Python code often reads like prose when naming is strong. In review, ask whether a new teammate could explain the function by reading the names alone.

### 💡 Takeaway

> **Readable Python starts with names that describe the domain, not the implementation shortcut.**

---

## 02. Truthy, falsy, and None checks

> Distinguish missing values from valid falsy values such as empty strings, zero, and empty collections.

**Tags:** `none` `conditionals` `truthiness` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/truthy-falsy-none-checks)

### ✅ Good Code — `display_name.py`

```python
def display_name(user):
    # None means the caller did not provide a nickname.
    if user.nickname is None:
        return user.full_name

    return user.nickname
```

### ❌ Bad Code — `display_name.py`

```python
def display_name(user):
    # This treats "", 0, False, and None as the same case.
    if user.nickname:
        return user.nickname

    return user.full_name
```

### 🔍 What to review

- **Good Code:** The good version treats `None` as the missing case and preserves an intentionally empty nickname.
- **Bad Code:** The bad version treats every falsy value as missing. Empty strings, zero, empty lists, and `False` can accidentally take the fallback path.

### 📝 Review notes

Truthy checks are fine when any empty value means "not usable." When the code specifically means "not provided," ask for `is None` or `is not None`.

### 💡 Takeaway

> **Use `is None` when absence is different from an empty value.**

---

## 03. Mutable default arguments

> Avoid shared mutable defaults by creating fresh lists, dicts, or sets inside the function.

**Tags:** `defaults` `mutation` `state` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/mutable-default-arguments)

### ✅ Good Code — `tags.py`

```python
def add_review_tag(tag, tags=None):
    # Create a fresh list only when the caller omitted tags.
    if tags is None:
        tags = []

    tags.append(tag)
    return tags
```

### ❌ Bad Code — `tags.py`

```python
def add_review_tag(tag, tags=[]):
    # This default list is reused across every call.
    tags.append(tag)
    return tags
```

### 🔍 What to review

- **Good Code:** The good version creates a new list only when the caller did not pass one.
- **Bad Code:** The bad version shares the same default list across calls. Tags from one request or test can appear in the next call.

### 📝 Review notes

Mutable defaults are a classic Python review catch. Scan function signatures for `[]`, `{}`, and `set()` defaults before reading the body.

### 💡 Takeaway

> **Default arguments are evaluated once, so mutable defaults can leak state between calls.**

---

## 04. List and dict comprehensions

> Use comprehensions for clear transformations, but switch to loops when branching becomes the main story.

**Tags:** `comprehensions` `collections` `readability` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/list-dict-comprehensions)

### ✅ Good Code — `reviewers.py`

```python
def active_reviewer_emails(users):
    # One filter plus one transformation fits a comprehension.
    return [
        user.email.lower()
        for user in users
        if user.is_active and user.email
    ]
```

### ❌ Bad Code — `reviewers.py`

```python
def active_reviewer_emails(users):
    # Nested branches hide the simple filter-and-map intent.
    emails = []
    for user in users:
        if user.is_active:
            if user.email:
                emails.append(user.email.lower())
    return emails
```

### 🔍 What to review

- **Good Code:** The good version expresses one filter and one transformation in a compact shape.
- **Bad Code:** The bad version spreads a simple collection transformation across nested conditionals, making the main intent harder to see.

### 📝 Review notes

Use a comprehension when the reader can say "filter these items and map this value." If the code needs side effects or multiple branches, use a named loop so the branching is visible.

### 💡 Takeaway

> **A comprehension should make simple filtering and mapping easier to read, not denser to parse.**

---

## 05. Exception boundaries

> Catch specific exceptions at the boundary and preserve the original failure with exception chaining.

**Tags:** `exceptions` `errors` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/exception-boundaries)

### ✅ Good Code — `settings.py`

```python
import json


class SettingsError(Exception):
    pass


def load_settings(path):
    try:
        # The boundary handles only the file and JSON errors it expects.
        with open(path, encoding="utf-8") as file:
            return json.load(file)
    except FileNotFoundError as error:
        raise SettingsError(f"Settings file not found: {path}") from error
    except json.JSONDecodeError as error:
        raise SettingsError(f"Settings file is invalid JSON: {path}") from error
```

### ❌ Bad Code — `settings.py`

```python
import json


def load_settings(path):
    try:
        # The file handle is not scoped, and the default encoding is hidden.
        return json.load(open(path))
    except Exception:
        # Catching everything turns real failures into silent defaults.
        return {}
```

### 🔍 What to review

- **Good Code:** The good version catches the expected file and JSON failures, then raises a domain error while preserving the original exception.
- **Bad Code:** The bad version catches everything and returns an empty config. Permission errors, programmer mistakes, and corrupt JSON all become silent defaults.

### 📝 Review notes

When reviewing `except Exception`, ask what failure is actually expected. If the caller needs a friendly error, wrap it with context instead of swallowing it.

### 💡 Takeaway

> **Specific exceptions make failure modes reviewable; blanket catches hide the bug you need to see.**

---

## 06. Context managers for files

> Use context managers so file handles and other resources close even when work fails.

**Tags:** `files` `context managers` `resources` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/context-managers-files)

### ✅ Good Code — `export_reviews.py`

```python
def write_review_export(path, rows):
    # with closes the file even if writing a row fails.
    with open(path, "w", encoding="utf-8") as file:
        for row in rows:
            file.write(f"{row.id},{row.status}\n")
```

### ❌ Bad Code — `export_reviews.py`

```python
def write_review_export(path, rows):
    # An exception before close can leave the file handle open.
    file = open(path, "w")
    for row in rows:
        file.write(f"{row.id},{row.status}\n")
    file.close()
```

### 🔍 What to review

- **Good Code:** The good version scopes the file lifetime with `with`, includes an encoding, and closes the file even if writing a row fails.
- **Bad Code:** The bad version relies on manual `close()`. An exception in the loop can leave the file open, and the default encoding depends on the environment.

### 📝 Review notes

Context managers are not just style. They make cleanup deterministic and give reviewers a visible boundary for resource ownership.

### 💡 Takeaway

> **Resource lifetime should be visible in the code shape, not hidden in manual cleanup.**

---

## 07. Dataclasses for data shapes

> Use dataclasses for stable internal data shapes instead of loose dictionaries with repeated string keys.

**Tags:** `dataclasses` `data` `models` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/dataclasses-data-shapes)

### ✅ Good Code — `review_summary.py`

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ReviewSummary:
    review_id: int
    title: str
    approved: bool


def summarize_review(review):
    # The named shape makes every returned field explicit.
    return ReviewSummary(
        review_id=review.id,
        title=review.title,
        approved=review.status == "approved",
    )
```

### ❌ Bad Code — `review_summary.py`

```python
def summarize_review(review):
    # Vague dict keys make the returned shape easy to misunderstand.
    return {
        "id": review.id,
        "name": review.title,
        "ok": review.status == "approved",
    }
```

### 🔍 What to review

- **Good Code:** The good version gives the shape a name and makes every field explicit.
- **Bad Code:** The bad version returns a loose dictionary with vague keys. Callers have to remember what `"name"` and `"ok"` mean, and typos will be found late.

### 📝 Review notes

Dictionaries are useful at boundaries like JSON. Inside the application, a small dataclass often makes the contract easier to review and test.

### 💡 Takeaway

> **A named data shape makes fields, defaults, and equality behavior easier to review.**

---

## 08. Type hints at boundaries

> Add type hints where data enters or leaves a module so assumptions are visible to tools and reviewers.

**Tags:** `typing` `boundaries` `interfaces` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/type-hints-boundaries)

### ✅ Good Code — `review_input.py`

```python
from dataclasses import dataclass
from typing import Mapping


@dataclass(frozen=True)
class ReviewInput:
    title: str
    author_id: int


def parse_review_input(payload: Mapping[str, object]) -> ReviewInput:
    # Boundary types show callers what comes in and what comes back.
    return ReviewInput(
        title=str(payload["title"]),
        author_id=int(payload["author_id"]),
    )
```

### ❌ Bad Code — `review_input.py`

```python
def parse_review_input(payload):
    # Untyped input and output make callers guess the contract.
    return {
        "title": payload["title"],
        "author_id": payload["author_id"],
    }
```

### 🔍 What to review

- **Good Code:** The good version tells callers what kind of payload is accepted and returns a named object with typed fields.
- **Bad Code:** The bad version accepts and returns anything. Reviewers cannot see whether `author_id` should be a string, number, or optional value.

### 📝 Review notes

You do not need to type every line to get value from typing. Review the boundary first: function parameters, return values, and shared data models.

### 💡 Takeaway

> **Type hints are most valuable at module boundaries, not as decoration on obvious local variables.**

---

## 09. Dependency injection for testability

> Pass collaborators into functions or classes so behavior can be tested without patching global modules.

**Tags:** `testing` `dependencies` `design` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/dependency-injection-testability)

### ✅ Good Code — `notifications.py`

```python
def notify_reviewer(review, mailer):
    # Tests can pass a fake mailer and inspect the message.
    message = f"Review {review.title} is ready"
    mailer.send(
        to=review.reviewer_email,
        subject="Review requested",
        body=message,
    )
```

### ❌ Bad Code — `notifications.py`

```python
from app.email import send_email


def notify_reviewer(review):
    # The global sender makes tests patch the import path.
    send_email(
        review.reviewer_email,
        "Review requested",
        f"Review {review.title} is ready",
    )
```

### 🔍 What to review

- **Good Code:** The good version makes the mailer dependency explicit, so tests can pass a fake mailer and assert behavior.
- **Bad Code:** The bad version imports a global side-effect function directly. Tests must patch the import path correctly, and production coupling stays hidden.

### 📝 Review notes

Dependency injection does not have to mean a framework. In Python, passing a collaborator as an argument is often enough to make code easier to test.

### 💡 Takeaway

> **Injected dependencies make side effects explicit and keep tests focused on behavior.**

---

## 10. Async and await boundaries

> Await asynchronous work and gather independent calls intentionally so coroutines do not leak out unfinished.

**Tags:** `async` `await` `concurrency` · [View on site](https://good-code-bad-code.pages.dev/tracks/python/async-await-boundaries)

### ✅ Good Code — `profiles.py`

```python
import asyncio


async def fetch_profiles(user_ids, api):
    # Gather independent async calls so the result is real data.
    tasks = [api.fetch_profile(user_id) for user_id in user_ids]
    return await asyncio.gather(*tasks)
```

### ❌ Bad Code — `profiles.py`

```python
async def fetch_profiles(user_ids, api):
    profiles = []
    for user_id in user_ids:
        # This appends coroutine objects, not fetched profiles.
        profiles.append(api.fetch_profile(user_id))
    return profiles
```

### 🔍 What to review

- **Good Code:** The good version collects independent async calls and awaits them together.
- **Bad Code:** The bad version returns coroutine objects instead of profile data. The caller may see confusing values, and the intended I/O may never complete.

### 📝 Review notes

When reviewing async Python, trace every coroutine. It should be awaited, returned intentionally for a caller to await, or scheduled with clear ownership.

### 💡 Takeaway

> **Calling an async function creates work to await; it does not complete the work by itself.**

---

[← Back to index](README.md)
