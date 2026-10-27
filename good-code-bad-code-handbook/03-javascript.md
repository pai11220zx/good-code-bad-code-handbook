# JavaScript — Code Review Handbook

> Review control flow, async behavior, naming, and side effects.

- **Category:** Foundations
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/javascript](https://good-code-bad-code.pages.dev/tracks/javascript)
- **Related tracks:** [TypeScript](04-typescript.md), [React](05-react.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Strict equality and nullish checks](#01-strict-equality-and-nullish-checks)** — Use strict equality for comparisons and nullish coalescing when only null or undefined should fall back.
- [ ] **[Guard clauses](#02-guard-clauses)** — Let the common path stay shallow by returning early for invalid inputs.
- [ ] **[Validating data boundaries](#03-validating-data-boundaries)** — Validate API, storage, and message payloads at the boundary before using nested fields.
- [ ] **[Naming side effects](#04-naming-side-effects)** — Make mutation visible in the function name or return a new value instead.
- [ ] **[Array transformations](#05-array-transformations)** — Prefer filter, map, and reduce when they describe the transformation without mutating input data.
- [ ] **[Async error handling](#06-async-error-handling)** — Check both network failures and response status before trusting fetched data.
- [ ] **[Promise concurrency](#07-promise-concurrency)** — Use Promise.all for independent required work and reserve sequential awaits for true dependencies.
- [ ] **[Event listener cleanup](#08-event-listener-cleanup)** — Keep a stable handler reference and return a cleanup function when attaching listeners.
- [ ] **[Module boundaries and globals](#09-module-boundaries-and-globals)** — Expose small functions from modules and pass dependencies in when code needs storage, network, or browser APIs.
- [ ] **[Dates and time zones](#10-dates-and-time-zones)** — Use Intl date formatting and pass the intended time zone when the displayed day matters.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Strict equality and nullish checks](#01-strict-equality-and-nullish-checks) | `equality` `nullish` `coercion` |
| 02 | [Guard clauses](#02-guard-clauses) | `control-flow` `readability` `validation` |
| 03 | [Validating data boundaries](#03-validating-data-boundaries) | `validation` `boundaries` `defensive-code` |
| 04 | [Naming side effects](#04-naming-side-effects) | `naming` `mutation` `side-effects` |
| 05 | [Array transformations](#05-array-transformations) | `arrays` `immutability` `readability` |
| 06 | [Async error handling](#06-async-error-handling) | `async` `fetch` `errors` |
| 07 | [Promise concurrency](#07-promise-concurrency) | `async` `promises` `performance` |
| 08 | [Event listener cleanup](#08-event-listener-cleanup) | `events` `cleanup` `browser` |
| 09 | [Module boundaries and globals](#09-module-boundaries-and-globals) | `modules` `globals` `architecture` |
| 10 | [Dates and time zones](#10-dates-and-time-zones) | `dates` `time-zones` `internationalization` |

---

## 01. Strict equality and nullish checks

> Compare values intentionally and preserve meaningful falsy values.

**Tags:** `equality` `nullish` `coercion` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/strict-equality-nullish-checks)

### ✅ Good Code — `review-options.js`

```js
function formatReviewState(state) {
  // Strict equality avoids coercion hiding unexpected values.
  if (state === "approved") {
    return "Approved";
  }

  if (state === "changes_requested") {
    return "Changes requested";
  }

  return "Pending";
}

function getRetryCount(options) {
  return options.retryCount ?? 3;
}
```

### ❌ Bad Code — `review-options.js`

```js
function formatReviewState(state) {
  // Loose equality and || blur missing data with valid falsy values.
  if (state == "approved") {
    return "Approved";
  }

  if (state == "changes_requested") {
    return "Changes requested";
  }

  return "Pending";
}

function getRetryCount(options) {
  return options.retryCount || 3;
}
```

### 🔍 What to review

- **Good Code:** The good version compares exact values and only falls back when `retryCount` is `null` or `undefined`, so `0` remains a valid explicit value.
- **Bad Code:** The bad version depends on coercion and uses `||`, which treats valid falsy values like `0` as missing configuration.

### 📝 Review notes

In review, look for loose equality and fallback logic together. They often hide bugs where the code cannot tell the difference between absent data and a deliberately chosen falsy value.

### 💡 Takeaway

> **Use strict equality for comparisons and nullish coalescing when only null or undefined should fall back.**

---

## 02. Guard clauses

> Use early returns to keep exceptional cases out of the main path.

**Tags:** `control-flow` `readability` `validation` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/guard-clauses)

### ✅ Good Code — `review-access.js`

```js
function canStartReview(user, pullRequest) {
  // Guard clauses keep rejected cases out of the main path.
  if (!user) {
    return false;
  }

  if (pullRequest.isDraft) {
    return false;
  }

  if (!pullRequest.reviewers.includes(user.id)) {
    return false;
  }

  return pullRequest.filesChanged.length > 0;
}
```

### ❌ Bad Code — `review-access.js`

```js
function canStartReview(user, pullRequest) {
  // Nested branches make reviewers carry every condition at once.
  if (user) {
    if (!pullRequest.isDraft) {
      if (pullRequest.reviewers.includes(user.id)) {
        if (pullRequest.filesChanged.length > 0) {
          return true;
        }
      }
    }
  }

  return false;
}
```

### 🔍 What to review

- **Good Code:** The good version returns from disqualifying cases first, which keeps the final return focused on the actual positive condition.
- **Bad Code:** The bad version reaches the same answer, but each nested branch makes reviewers hold more state in their heads before they can see the main rule.

### 📝 Review notes

In review, look for indentation that represents policy complexity rather than necessary sequencing. Deep branching is review-worthy because it makes future conditions harder to add without hiding the reason a case is accepted or rejected.

### 💡 Takeaway

> **Let the common path stay shallow by returning early for invalid inputs.**

---

## 03. Validating data boundaries

> Check data from outside your code before the rest of the function trusts it.

**Tags:** `validation` `boundaries` `defensive-code` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/data-boundary-validation)

### ✅ Good Code — `review-payload.js`

```js
function normalizeReviewPayload(payload) {
  // Validate the boundary before reading nested fields.
  if (!payload || typeof payload !== "object") {
    return { ok: false, error: "Invalid payload." };
  }

  const { title, reviewers } = payload;

  if (typeof title !== "string" || title.trim() === "") {
    return { ok: false, error: "Title is required." };
  }

  if (!Array.isArray(reviewers)) {
    return { ok: false, error: "Reviewers must be a list." };
  }

  return {
    ok: true,
    value: {
      title: title.trim(),
      reviewers: reviewers.filter((id) => typeof id === "string"),
    },
  };
}
```

### ❌ Bad Code — `review-payload.js`

```js
function normalizeReviewPayload(payload) {
  // This trusts external shape before proving it exists.
  return {
    ok: true,
    value: {
      title: payload.title.trim(),
      reviewers: payload.reviewers.map((id) => id.toLowerCase()),
    },
  };
}
```

### 🔍 What to review

- **Good Code:** The good version treats incoming data as untrusted and returns a clear failure before the rest of the code touches nested fields.
- **Bad Code:** The bad version is short, but it crashes when the payload shape changes or when a field is missing, which turns a data problem into an unrelated runtime error.

### 📝 Review notes

In review, find the first line where external data enters the app. That boundary is the best place to make uncertainty explicit instead of spreading defensive checks across every caller.

### 💡 Takeaway

> **Validate API, storage, and message payloads at the boundary before using nested fields.**

---

## 04. Naming side effects

> Name functions so callers can see when data will be mutated.

**Tags:** `naming` `mutation` `side-effects` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/naming-side-effects)

### ✅ Good Code — `reviewers.js`

```js
function addReviewerInPlace(reviewers, reviewer) {
  // The name warns callers that the array will mutate.
  reviewers.push(reviewer);
  return reviewers;
}

const reviewers = ["Ada"];
addReviewerInPlace(reviewers, "Lin");
```

### ❌ Bad Code — `reviewers.js`

```js
function withReviewer(reviewers, reviewer) {
  // The name sounds immutable while mutating the input array.
  reviewers.push(reviewer);
  return reviewers;
}

const reviewers = ["Ada"];
const nextReviewers = withReviewer(reviewers, "Lin");
```

### 🔍 What to review

- **Good Code:** The good version makes the in-place mutation explicit, so the caller knows the original array changes.
- **Bad Code:** The bad version sounds like it returns a new collection, but it mutates the array it receives and can surprise callers that keep using the original reference.

### 📝 Review notes

In review, compare the function name with its side effects. Harmless-sounding mutation is review-worthy because future callers may make reasonable assumptions from the name and accidentally share changed state across features.

### 💡 Takeaway

> **Make mutation visible in the function name or return a new value instead.**

---

## 05. Array transformations

> Use array methods to make collection changes readable and avoid accidental mutation.

**Tags:** `arrays` `immutability` `readability` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/array-transformations)

### ✅ Good Code — `changed-files.js`

```js
function summarizeChangedFiles(files) {
  // The pipeline returns new summary objects instead of changing input.
  return files
    .filter((file) => file.status !== "deleted")
    .map((file) => ({
      path: file.path,
      churn: file.additions + file.deletions,
    }))
    .sort((left, right) => right.churn - left.churn);
}
```

### ❌ Bad Code — `changed-files.js`

```js
function summarizeChangedFiles(files) {
  // Sorting and assigning churn mutate data other code may still use.
  files.sort((left, right) => right.additions - left.additions);

  const summary = [];

  files.forEach((file) => {
    if (file.status !== "deleted") {
      file.churn = file.additions + file.deletions;
      summary.push(file);
    }
  });

  return summary;
}
```

### 🔍 What to review

- **Good Code:** The good version reads like a pipeline: remove deleted files, shape each item, then sort the new summaries by churn.
- **Bad Code:** The bad version mutates the original array order and adds `churn` onto existing file objects, which can surprise other code that still holds those references.

### 📝 Review notes

In review, ask whether the transformation changes the input or returns a new view of the data. Hidden mutation in collection code is easy to miss because the loop can look harmless at first glance.

### 💡 Takeaway

> **Prefer filter, map, and reduce when they describe the transformation without mutating input data.**

---

## 06. Async error handling

> Check request failures and non-2xx response statuses before parsing data.

**Tags:** `async` `fetch` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/async-error-handling)

### ✅ Good Code — `load-profile.js`

```js
async function loadProfile(userId) {
  // Check HTTP status before parsing and catch transport failures.
  try {
    const response = await fetch("/api/users/" + userId);

    if (!response.ok) {
      return { ok: false, error: "Profile request failed." };
    }

    const profile = await response.json();
    return { ok: true, profile };
  } catch {
    return { ok: false, error: "Network error while loading profile." };
  }
}
```

### ❌ Bad Code — `load-profile.js`

```js
async function loadProfile(userId) {
  // Happy-path parsing turns failures into unhandled exceptions.
  const response = await fetch("/api/users/" + userId);
  const profile = await response.json();

  return { ok: true, profile };
}
```

### 🔍 What to review

- **Good Code:** The good version separates transport failures from unsuccessful HTTP responses and only parses JSON after the response is acceptable.
- **Bad Code:** The bad version is concise, but it assumes every request succeeds and every response body can be parsed as JSON.

### 📝 Review notes

In review, ask what the caller receives for offline users, server errors, and non-JSON error pages. The bad version is review-worthy because it works on the happy path while pushing important failure behavior into unhandled exceptions.

### 💡 Takeaway

> **Check both network failures and response status before trusting fetched data.**

---

## 07. Promise concurrency

> Run independent async work together instead of waiting one request at a time.

**Tags:** `async` `promises` `performance` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/promise-concurrency)

### ✅ Good Code — `review-sidebar.js`

```js
async function loadReviewSidebar(pullRequestId) {
  // Independent requests start together and fail as one group.
  const [checks, comments, reviewers] = await Promise.all([
    fetchChecks(pullRequestId),
    fetchComments(pullRequestId),
    fetchReviewers(pullRequestId),
  ]);

  return { checks, comments, reviewers };
}
```

### ❌ Bad Code — `review-sidebar.js`

```js
async function loadReviewSidebar(pullRequestId) {
  // Sequential awaits add latency without a data dependency.
  const checks = await fetchChecks(pullRequestId);
  const comments = await fetchComments(pullRequestId);
  const reviewers = await fetchReviewers(pullRequestId);

  return { checks, comments, reviewers };
}
```

### 🔍 What to review

- **Good Code:** The good version starts three independent requests together and waits for the shared result once.
- **Bad Code:** The bad version makes each request wait for the previous one even though none of them uses the previous result, adding avoidable latency to the page.

### 📝 Review notes

In review, do not treat every chain of `await` as necessary sequencing. Ask which operations truly depend on earlier results and which can be grouped as concurrent work.

### 💡 Takeaway

> **Use Promise.all for independent required work and reserve sequential awaits for true dependencies.**

---

## 08. Event listener cleanup

> Pair event listeners with cleanup so repeated setup does not duplicate behavior.

**Tags:** `events` `cleanup` `browser` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/event-listener-cleanup)

### ✅ Good Code — `keyboard-shortcut.js`

```js
function attachSaveShortcut(target, onSave) {
  // A stable handler reference lets callers remove the listener later.
  function handleKeydown(event) {
    if ((event.ctrlKey || event.metaKey) && event.key === "s") {
      event.preventDefault();
      onSave();
    }
  }

  target.addEventListener("keydown", handleKeydown);

  return () => {
    target.removeEventListener("keydown", handleKeydown);
  };
}
```

### ❌ Bad Code — `keyboard-shortcut.js`

```js
function attachSaveShortcut(target, onSave) {
  // Anonymous listeners cannot be removed by the caller.
  target.addEventListener("keydown", (event) => {
    if ((event.ctrlKey || event.metaKey) && event.key === "s") {
      event.preventDefault();
      onSave();
    }
  });
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the handler reference and returns a cleanup function, so the caller can detach the listener when the UI is removed.
- **Bad Code:** The bad version adds an anonymous listener and gives the caller no practical way to remove it, which can duplicate shortcuts after repeated setup.

### 📝 Review notes

In review, every listener, subscription, timer, or observer should raise the same question: where does it get cleaned up? If the answer is unclear, the behavior may grow over time.

### 💡 Takeaway

> **Keep a stable handler reference and return a cleanup function when attaching listeners.**

---

## 09. Module boundaries and globals

> Keep shared behavior behind module APIs instead of scattering mutable global state.

**Tags:** `modules` `globals` `architecture` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/module-boundaries-globals)

### ✅ Good Code — `review-filters.js`

```js
const FILTER_STORAGE_KEY = "review-filters";

export function readReviewFilters(storage) {
  // Storage is a dependency, not hidden global state.
  const rawValue = storage.getItem(FILTER_STORAGE_KEY);

  if (!rawValue) {
    return { status: "open" };
  }

  return JSON.parse(rawValue);
}

export function saveReviewFilters(storage, filters) {
  storage.setItem(FILTER_STORAGE_KEY, JSON.stringify(filters));
}
```

### ❌ Bad Code — `review-filters.js`

```js
window.reviewFilters = { status: "open" };

function readReviewFilters() {
  // Global reads and writes let callers bypass the module API.
  return JSON.parse(localStorage.reviewFilters || "{}");
}

function saveReviewFilters(filters) {
  localStorage.reviewFilters = JSON.stringify(filters);
  window.reviewFilters = filters;
}
```

### 🔍 What to review

- **Good Code:** The good version hides the storage key inside the module and accepts storage as a dependency, which makes the behavior easier to test and reuse.
- **Bad Code:** The bad version writes to `window` and `localStorage` directly from the module, so unrelated code can change state without going through a clear API.

### 📝 Review notes

In review, globals are not automatically wrong, but they should be deliberate. A module boundary should tell callers how to use shared behavior without requiring them to know where every value is stored.

### 💡 Takeaway

> **Expose small functions from modules and pass dependencies in when code needs storage, network, or browser APIs.**

---

## 10. Dates and time zones

> Format dates with explicit locale and time zone choices instead of manual date math.

**Tags:** `dates` `time-zones` `internationalization` · [View on site](https://good-code-bad-code.pages.dev/tracks/javascript/dates-time-zones)

### ✅ Good Code — `release-date.js`

```js
function formatReleaseDate(isoDate, locale, timeZone) {
  // Locale and time zone are explicit formatting inputs.
  const formatter = new Intl.DateTimeFormat(locale, {
    dateStyle: "medium",
    timeZone,
  });

  return formatter.format(new Date(isoDate));
}

formatReleaseDate("2026-05-24T00:00:00Z", "en-US", "UTC");
```

### ❌ Bad Code — `release-date.js`

```js
function formatReleaseDate(dateString) {
  // Manual formatting depends on the runtime's local time zone.
  const date = new Date(dateString);

  return (
    date.getMonth() + 1 + "/" +
    date.getDate() + "/" +
    date.getFullYear()
  );
}
```

### 🔍 What to review

- **Good Code:** The good version makes locale and time zone part of the formatting decision, so the same timestamp can be displayed consistently for the intended audience.
- **Bad Code:** The bad version formats by hand using the runtime's local time zone, which can shift the displayed day for users in different regions.

### 📝 Review notes

In review, dates deserve extra suspicion because they often work on one machine and fail at the edge of a day, month, or time zone. Prefer explicit formatting over hidden runtime defaults.

### 💡 Takeaway

> **Use Intl date formatting and pass the intended time zone when the displayed day matters.**

---

[← Back to index](README.md)
