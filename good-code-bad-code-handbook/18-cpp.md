# C++ — Code Review Handbook

> Review RAII, smart pointers, move semantics, const references, rule of zero, exception boundaries, templates, algorithms, locks, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/cpp](https://good-code-bad-code.pages.dev/tracks/cpp)
- **Related tracks:** [C](17-c.md), [C#](19-csharp.md), [Rust](20-rust.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[RAII resource ownership](#01-raii-resource-ownership)** — C++ resource code should tie acquisition to an object lifetime, so exits through return or exception still release the resource.
- [ ] **[Smart pointer ownership](#02-smart-pointer-ownership)** — A C++ API should show ownership in its type: `unique_ptr` for one owner, `shared_ptr` for shared lifetime, and raw pointers for observation.
- [ ] **[Move semantics and lifetimes](#03-move-semantics-and-lifetimes)** — Reference members are lifetime contracts; prefer owned values when the object needs data after the caller returns.
- [ ] **[Const references and string_view](#04-const-references-and-string_view)** — Read-only C++ parameters should say whether they borrow data, copy data, or mutate data.
- [ ] **[Rule of zero](#05-rule-of-zero)** — If a class does not own raw resources directly, it usually should not define a destructor, copy constructor, or move constructor.
- [ ] **[Exception boundaries](#06-exception-boundaries)** — Exceptions should travel with meaning until a boundary can translate them into a status, response, or log record.
- [ ] **[Template constraints](#07-template-constraints)** — Template code should name the operations it needs, because unconstrained templates fail far from the call site.
- [ ] **[Algorithm and iterator intent](#08-algorithm-and-iterator-intent)** — A standard algorithm can make a range operation read as a single reviewable action instead of a custom loop.
- [ ] **[Scoped locks for shared state](#09-scoped-locks-for-shared-state)** — Mutex code should make the lock lifetime visible in scope, not depend on matching manual lock and unlock calls.
- [ ] **[Unit tests for edge cases](#10-unit-tests-for-edge-cases)** — A C++ test should encode the boundary expectation in assertions so CI can fail without human log reading.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [RAII resource ownership](#01-raii-resource-ownership) | `raii` `resources` `ownership` |
| 02 | [Smart pointer ownership](#02-smart-pointer-ownership) | `unique_ptr` `ownership` `memory` |
| 03 | [Move semantics and lifetimes](#03-move-semantics-and-lifetimes) | `move` `lifetimes` `references` |
| 04 | [Const references and string_view](#04-const-references-and-string_view) | `const` `string_view` `parameters` |
| 05 | [Rule of zero](#05-rule-of-zero) | `rule-of-zero` `copy` `destructors` |
| 06 | [Exception boundaries](#06-exception-boundaries) | `exceptions` `boundaries` `errors` |
| 07 | [Template constraints](#07-template-constraints) | `templates` `concepts` `types` |
| 08 | [Algorithm and iterator intent](#08-algorithm-and-iterator-intent) | `algorithms` `iterators` `ranges` |
| 09 | [Scoped locks for shared state](#09-scoped-locks-for-shared-state) | `concurrency` `mutex` `locks` |
| 10 | [Unit tests for edge cases](#10-unit-tests-for-edge-cases) | `tests` `assertions` `edge-cases` |

---

## 01. RAII resource ownership

> Use RAII objects so files, locks, and other resources release through scope instead of manual cleanup paths.

**Tags:** `raii` `resources` `ownership` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/raii-resource-ownership)

### ✅ Good Code — `src/review_file.cpp`

```cpp
#include <filesystem>
#include <fstream>
#include <stdexcept>
#include <string_view>

void save_review(const std::filesystem::path& path, std::string_view body)
{
    std::ofstream file(path);
    if (!file) {
        throw std::runtime_error("cannot open review file");
    }

    // RAII closes the file when the stream leaves scope.
    file << body;
    if (!file) {
        throw std::runtime_error("cannot write review file");
    }
}
```

### ❌ Bad Code — `review_file.cpp`

```cpp
#include <cstdio>

bool save_review(const char* path, const char* body)
{
    FILE* file = std::fopen(path, "w");
    if (file == nullptr) {
        return false;
    }

    if (std::fputs(body, file) == EOF) {
        // Early return skips the manual fclose call.
        return false;
    }

    std::fclose(file);
    return true;
}
```

### 🔍 What to review

- **Good Code:** The good version uses an output stream whose destructor closes the file. Every exit path releases the resource, including exceptions raised after the write.
- **Bad Code:** The bad version uses manual `fclose` and returns early on write failure. That path leaves the file open and hides any close-time error.

### 📝 Review notes

In C++ reviews, prefer a resource-owning object over a cleanup checklist. The acquisition line and the release lifetime should be visible in the same scope.

### 💡 Takeaway

> **C++ resource code should tie acquisition to an object lifetime, so exits through return or exception still release the resource.**

---

## 02. Smart pointer ownership

> Return smart pointers when heap ownership crosses a boundary, and keep raw pointers for non-owning access.

**Tags:** `unique_ptr` `ownership` `memory` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/smart-pointer-ownership)

### ✅ Good Code — `src/review_factory.cpp`

```cpp
#include <memory>
#include <string>
#include <utility>

struct Review {
    explicit Review(std::string title) : title(std::move(title)) {}
    std::string title;
};

std::unique_ptr<Review> make_review(std::string title)
{
    // unique_ptr names the single owner of the Review.
    return std::make_unique<Review>(std::move(title));
}
```

### ❌ Bad Code — `review_factory.cpp`

```cpp
#include <string>
#include <utility>

struct Review {
    explicit Review(std::string title) : title(std::move(title)) {}
    std::string title;
};

Review* make_review(std::string title)
{
    // Raw new leaves callers to guess who must delete the Review.
    return new Review(std::move(title));
}
```

### 🔍 What to review

- **Good Code:** The good version returns `std::unique_ptr<Review>`, so the single-owner rule is part of the signature. Callers cannot forget that the object owns heap memory.
- **Bad Code:** The bad version returns a raw pointer from `new`. Some callers may delete it, some may leak it, and some may pass it around as a borrowed pointer.

### 📝 Review notes

When a C++ function returns a pointer, ask whether that pointer owns memory. If it owns memory, the type should say so before anyone reads the implementation.

### 💡 Takeaway

> **A C++ API should show ownership in its type: `unique_ptr` for one owner, `shared_ptr` for shared lifetime, and raw pointers for observation.**

---

## 03. Move semantics and lifetimes

> Move owned values into long-lived objects instead of storing references that can outlive the caller.

**Tags:** `move` `lifetimes` `references` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/move-semantics-lifetimes)

### ✅ Good Code — `src/review_batch.cpp`

```cpp
#include <string>
#include <utility>
#include <vector>

struct Review {
    std::string title;
};

class ReviewBatch {
public:
    explicit ReviewBatch(std::vector<Review> reviews)
        : reviews_(std::move(reviews))
    {
        // Moving owned input gives the batch a stable lifetime.
    }

private:
    std::vector<Review> reviews_;
};
```

### ❌ Bad Code — `review_batch.cpp`

```cpp
#include <string>
#include <vector>

struct Review {
    std::string title;
};

class ReviewBatch {
public:
    explicit ReviewBatch(const std::vector<Review>& reviews)
        : reviews_(reviews)
    {
        // Reference storage can outlive the caller's vector.
    }

private:
    const std::vector<Review>& reviews_;
};
```

### 🔍 What to review

- **Good Code:** The good version takes the vector by value and moves it into the batch. The class owns its data, so it can safely outlive the call site.
- **Bad Code:** The bad version stores a reference member. If the caller passes a temporary or a local vector, the batch later reads through a dangling reference.

### 📝 Review notes

Review reference members with extra suspicion. They can be correct, but only when the referenced object has a longer lifetime than the class that stores the reference.

### 💡 Takeaway

> **Reference members are lifetime contracts; prefer owned values when the object needs data after the caller returns.**

---

## 04. Const references and string_view

> Use const references or string_view for read-only inputs so call sites avoid copies and mutation stays visible.

**Tags:** `const` `string_view` `parameters` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/const-references-string-view)

### ✅ Good Code — `src/review_title.cpp`

```cpp
#include <string_view>

bool has_review_prefix(std::string_view title, std::string_view prefix)
{
    // string_view borrows read-only text without copying it.
    return title.starts_with(prefix);
}
```

### ❌ Bad Code — `review_title.cpp`

```cpp
#include <string>

bool has_review_prefix(std::string title, std::string prefix)
{
    // Copying both strings hides the read-only intent.
    title.shrink_to_fit();
    return title.rfind(prefix, 0) == 0;
}
```

### 🔍 What to review

- **Good Code:** The good version uses `std::string_view` for borrowed text. The signature communicates that the function reads the strings and does not retain them.
- **Bad Code:** The bad version copies both strings, then mutates a copy inside a predicate. That makes a read-only check look heavier and less direct than it is.

### 📝 Review notes

Parameter types are review documentation. Use `std::string_view` for borrowed text, `const T&` for larger read-only objects, and values when the function needs a copy or will move.

### 💡 Takeaway

> **Read-only C++ parameters should say whether they borrow data, copy data, or mutate data.**

---

## 05. Rule of zero

> Prefer standard-library members so copy, move, and destruction follow generated behavior instead of manual pointer code.

**Tags:** `rule-of-zero` `copy` `destructors` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/rule-of-zero)

### ✅ Good Code — `src/review_summary.cpp`

```cpp
#include <string>
#include <utility>
#include <vector>

class ReviewSummary {
public:
    ReviewSummary(std::string title, std::vector<int> scores)
        : title_(std::move(title)), scores_(std::move(scores)) {}

    // Standard members keep copy and move behavior generated by the type.
private:
    std::string title_;
    std::vector<int> scores_;
};
```

### ❌ Bad Code — `review_summary.cpp`

```cpp
#include <cstring>

class ReviewSummary {
public:
    explicit ReviewSummary(const char* title)
        : title_(new char[std::strlen(title) + 1])
    {
        std::strcpy(title_, title);
    }

    // Manual destructor without copy control makes copies share one pointer.
    ~ReviewSummary()
    {
        delete[] title_;
    }

private:
    char* title_;
};
```

### 🔍 What to review

- **Good Code:** The good version stores standard-library values, so generated copy, move, and destruction rules are enough.
- **Bad Code:** The bad version owns a raw character buffer and defines only a destructor. The compiler still generates copying, which can make two objects delete the same pointer.

### 📝 Review notes

If you see a destructor in modern C++, look for the rest of the ownership story. Often the best fix is not to add four special members, but to remove raw ownership from the class.

### 💡 Takeaway

> **If a class does not own raw resources directly, it usually should not define a destructor, copy constructor, or move constructor.**

---

## 06. Exception boundaries

> Catch exceptions at boundaries where code can translate them into a result the caller understands.

**Tags:** `exceptions` `boundaries` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/exception-translation-boundaries)

### ✅ Good Code — `src/review_loader.cpp`

```cpp
#include <expected>
#include <string>

std::expected<Review, std::string> load_review_result(const std::string& path)
{
    try {
        return parse_review_file(path);
    } catch (const ParseError& error) {
        // Boundary translation preserves the parser failure message.
        return std::unexpected(error.what());
    }
}
```

### ❌ Bad Code — `review_loader.cpp`

```cpp
#include <optional>
#include <string>

std::optional<Review> load_review_result(const std::string& path)
{
    try {
        return parse_review_file(path);
    } catch (...) {
        // Catching every exception erases the original failure.
        return std::nullopt;
    }
}
```

### 🔍 What to review

- **Good Code:** The good version catches a domain exception and translates it into an `expected` error string. The caller still receives the failure reason.
- **Bad Code:** The bad version catches every exception and returns an empty optional. It hides parse errors, file errors, and programming errors behind the same result.

### 📝 Review notes

Catch exceptions where the code has enough context to translate them. Catching too low erases meaning; catching too high can turn recoverable failures into crashes.

### 💡 Takeaway

> **Exceptions should travel with meaning until a boundary can translate them into a status, response, or log record.**

---

## 07. Template constraints

> Use concepts or focused type requirements so template errors point to the contract the caller missed.

**Tags:** `templates` `concepts` `types` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/template-constraints)

### ✅ Good Code — `src/review_score.cpp`

```cpp
#include <concepts>

template <typename T>
concept ScoredReview = requires(const T& value) {
    { value.score() } -> std::convertible_to<int>;
};

template <ScoredReview T>
bool passes_review_threshold(const T& review)
{
    // Concept names the API expected from T.
    return review.score() >= 70;
}
```

### ❌ Bad Code — `review_score.cpp`

```cpp
template <typename T>
bool passes_review_threshold(T review)
{
    // Unconstrained template errors appear far from the call site.
    return review.score() >= 70;
}
```

### 🔍 What to review

- **Good Code:** The good version defines a concept that says the type must expose `score()` convertible to `int`. Failed calls point to that named contract.
- **Bad Code:** The bad version accepts any type until the function body asks for `score()`. The compiler error can be long and detached from the API intent.

### 📝 Review notes

Template flexibility should come with a named constraint. If a template needs methods, operators, or value categories, make those requirements visible at the boundary.

### 💡 Takeaway

> **Template code should name the operations it needs, because unconstrained templates fail far from the call site.**

---

## 08. Algorithm and iterator intent

> Use standard algorithms when their names describe the pass over a range more directly than manual counters.

**Tags:** `algorithms` `iterators` `ranges` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/algorithm-iterator-intent)

### ✅ Good Code — `src/review_counts.cpp`

```cpp
#include <algorithm>
#include <vector>

int count_published_reviews(const std::vector<Review>& reviews)
{
    // Algorithm name says this pass counts matching reviews.
    return static_cast<int>(std::count_if(
        reviews.begin(),
        reviews.end(),
        [](const Review& review) { return review.is_published(); }
    ));
}
```

### ❌ Bad Code — `review_counts.cpp`

```cpp
#include <vector>

int count_published_reviews(const std::vector<Review>& reviews)
{
    int total = 0;
    for (std::size_t index = 0; index < reviews.size(); ++index) {
        // Manual counters make the range goal easy to miss.
        if (reviews[index].is_published()) {
            total++;
        }
    }

    return total;
}
```

### 🔍 What to review

- **Good Code:** The good version says the function counts reviews that match a predicate. The range and predicate are visible without index bookkeeping.
- **Bad Code:** The bad version works, but reviewers must read the loop body to learn that the goal is a count. Manual index code also adds room for off-by-one mistakes.

### 📝 Review notes

Use algorithms when the algorithm name matches the work: count, find, transform, any, all, none. Keep loops when the business rule needs named steps or multiple exits.

### 💡 Takeaway

> **A standard algorithm can make a range operation read as a single reviewable action instead of a custom loop.**

---

## 09. Scoped locks for shared state

> Use scoped lock objects so mutex ownership follows block lifetime and early returns cannot leave shared state locked.

**Tags:** `concurrency` `mutex` `locks` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/scoped-locks-shared-state)

### ✅ Good Code — `src/review_cache.cpp`

```cpp
#include <mutex>
#include <optional>
#include <string>
#include <unordered_map>

class ReviewCache {
public:
    std::optional<Review> find(const std::string& id) const
    {
        // scoped_lock ties the mutex lifetime to this read.
        std::scoped_lock lock(mutex_);
        auto found = reviews_.find(id);
        if (found == reviews_.end()) {
            return std::nullopt;
        }

        return found->second;
    }

private:
    mutable std::mutex mutex_;
    std::unordered_map<std::string, Review> reviews_;
};
```

### ❌ Bad Code — `review_cache.cpp`

```cpp
#include <mutex>
#include <optional>
#include <string>
#include <unordered_map>

class ReviewCache {
public:
    std::optional<Review> find(const std::string& id) const
    {
        mutex_.lock();
        auto found = reviews_.find(id);
        if (found == reviews_.end()) {
            // Early return leaves the mutex locked.
            return std::nullopt;
        }

        mutex_.unlock();
        return found->second;
    }

private:
    mutable std::mutex mutex_;
    std::unordered_map<std::string, Review> reviews_;
};
```

### 🔍 What to review

- **Good Code:** The good version creates a lock object at the top of the critical section. The mutex unlocks when the lock leaves scope.
- **Bad Code:** The bad version calls `lock()` and `unlock()` manually. The not-found branch returns before unlock, so later calls can deadlock.

### 📝 Review notes

Review lock lifetime the same way you review file lifetime. A scoped guard should make the protected region visible without matching distant calls by eye.

### 💡 Takeaway

> **Mutex code should make the lock lifetime visible in scope, not depend on matching manual lock and unlock calls.**

---

## 10. Unit tests for edge cases

> Write C++ tests that assert edge cases directly instead of printing values for reviewers to inspect.

**Tags:** `tests` `assertions` `edge-cases` · [View on site](https://good-code-bad-code.pages.dev/tracks/cpp/unit-tests-edge-cases)

### ✅ Good Code — `tests/review_validation_test.cpp`

```cpp
#include <cassert>

void rejects_empty_title()
{
    // Test name and assertion capture the boundary case.
    assert(validate_review("", 80) == ReviewStatus::empty_title);
}

int main()
{
    rejects_empty_title();
}
```

### ❌ Bad Code — `tests/review_validation_test.cpp`

```cpp
#include <iostream>

int main()
{
    auto status = validate_review("", 80);

    // Printing the status makes CI pass even when the value is wrong.
    std::cout << static_cast<int>(status) << "\n";
}
```

### 🔍 What to review

- **Good Code:** The good version names the edge case and asserts the exact status. CI fails when validation accepts an empty title or returns the wrong error.
- **Bad Code:** The bad version prints the status and exits successfully. The test only works if a person reads the log and notices the wrong number.

### 📝 Review notes

C++ tests should pin behavior at boundaries: empty values, moved-from values, missing keys, exceptions, and lock-protected paths. Prefer assertions over demonstration output.

### 💡 Takeaway

> **A C++ test should encode the boundary expectation in assertions so CI can fail without human log reading.**

---

[← Back to index](README.md)
