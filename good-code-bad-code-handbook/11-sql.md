# SQL — Code Review Handbook

> Review schemas, queries, joins, indexes, transactions, migrations, and data safety.

- **Category:** Data + Ops
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/sql](https://good-code-bad-code.pages.dev/tracks/sql)
- **Related tracks:** [Git](22-git.md), [Docker](26-docker.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Schema keys and constraints](#01-schema-keys-and-constraints)** — A schema should enforce the data rules the application depends on.
- [ ] **[Explicit SELECT columns](#02-explicit-select-columns)** — A query result is an API contract, so keep its shape intentional.
- [ ] **[Filtering and NULL handling](#03-filtering-and-null-handling)** — NULL is not equal to anything, so review nullable filters with extra care.
- [ ] **[Join cardinality](#04-join-cardinality)** — A join changes row count, so review the relationship before reviewing the selected columns.
- [ ] **[Aggregation and grouping](#05-aggregation-and-grouping)** — Every non-aggregated selected value should be part of the grouping intent.
- [ ] **[Index-friendly predicates](#06-index-friendly-predicates)** — A correct query can still be review-worthy if it prevents useful index access.
- [ ] **[Transactions for atomic workflows](#07-transactions-for-atomic-workflows)** — If one business action writes multiple rows, review where the transaction begins and ends.
- [ ] **[Migration backfills](#08-migration-backfills)** — A safe migration respects the data that already exists.
- [ ] **[Parameterized queries](#09-parameterized-queries)** — User input should become query data, not query text.
- [ ] **[Stable pagination](#10-stable-pagination)** — Pagination needs a stable order, not just a limit.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Schema keys and constraints](#01-schema-keys-and-constraints) | `schema` `constraints` `keys` |
| 02 | [Explicit SELECT columns](#02-explicit-select-columns) | `select` `projection` `api` |
| 03 | [Filtering and NULL handling](#03-filtering-and-null-handling) | `where` `null` `filters` |
| 04 | [Join cardinality](#04-join-cardinality) | `joins` `cardinality` `duplicates` |
| 05 | [Aggregation and grouping](#05-aggregation-and-grouping) | `aggregate` `group by` `reporting` |
| 06 | [Index-friendly predicates](#06-index-friendly-predicates) | `indexes` `where` `performance` |
| 07 | [Transactions for atomic workflows](#07-transactions-for-atomic-workflows) | `transactions` `atomicity` `writes` |
| 08 | [Migration backfills](#08-migration-backfills) | `migrations` `backfill` `constraints` |
| 09 | [Parameterized queries](#09-parameterized-queries) | `security` `injection` `parameters` |
| 10 | [Stable pagination](#10-stable-pagination) | `pagination` `ordering` `cursors` |

---

## 01. Schema keys and constraints

> Model table ownership with primary keys, foreign keys, uniqueness, and check constraints.

**Tags:** `schema` `constraints` `keys` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/schema-keys-constraints)

### ✅ Good Code — `schema.sql`

```sql
/* Keys and constraints let the database enforce core data rules. */
CREATE TABLE users (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE code_reviews (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  author_id BIGINT NOT NULL REFERENCES users(id),
  title TEXT NOT NULL,
  status TEXT NOT NULL CHECK (
    status IN ('draft', 'submitted', 'approved', 'rejected')
  ),
  submitted_at TIMESTAMPTZ
);
```

### ❌ Bad Code — `schema.sql`

```sql
/* Missing constraints lets invalid rows into the database. */
CREATE TABLE users (
  id BIGINT,
  email TEXT,
  name TEXT,
  created_at TEXT
);

CREATE TABLE code_reviews (
  id BIGINT,
  author_id BIGINT,
  title TEXT,
  status TEXT,
  submitted_at TEXT
);
```

### 🔍 What to review

- **Good Code:** The good version makes identity, uniqueness, relationships, required fields, and allowed review states explicit in the database.
- **Bad Code:** The bad version stores important rules only as hope. Duplicate users, orphaned reviews, invalid statuses, and inconsistent dates can all be inserted.

### 📝 Review notes

When reviewing SQL, start with the schema contract. If the application assumes a value is present or a relationship exists, the database should usually help enforce it.

### 💡 Takeaway

> **A schema should enforce the data rules the application depends on.**

---

## 02. Explicit SELECT columns

> Select the columns the caller needs instead of depending on table shape.

**Tags:** `select` `projection` `api` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/explicit-select-columns)

### ✅ Good Code — `submitted-reviews.sql`

```sql
/* Return only the fields this screen or API needs. */
SELECT
  reviews.id,
  reviews.title,
  reviews.status,
  users.name AS author_name,
  reviews.submitted_at
FROM code_reviews AS reviews
JOIN users ON users.id = reviews.author_id
WHERE reviews.status = 'submitted'
ORDER BY reviews.submitted_at DESC
LIMIT 20;
```

### ❌ Bad Code — `submitted-reviews.sql`

```sql
/* SELECT * leaks table shape and duplicate column names. */
SELECT *
FROM code_reviews
JOIN users ON users.id = code_reviews.author_id
WHERE status = 'submitted'
ORDER BY submitted_at DESC
LIMIT 20;
```

### 🔍 What to review

- **Good Code:** The good version names the result columns and aliases the author field so the consumer gets a stable shape.
- **Bad Code:** The bad version leaks every column from both tables, risks duplicate column names, and can break callers when schema changes add sensitive or unexpected fields.

### 📝 Review notes

`SELECT *` is tempting during exploration, but it is a poor production contract. In review, ask whether every returned column is needed by the caller.

### 💡 Takeaway

> **A query result is an API contract, so keep its shape intentional.**

---

## 03. Filtering and NULL handling

> Use SQL NULL operators and explicit boolean logic so filters match real data.

**Tags:** `where` `null` `filters` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/filtering-null-handling)

### ✅ Good Code — `review-filters.sql`

```sql
/* IS NULL and parentheses make nullable filters explicit. */
SELECT id, title, approved_at
FROM code_reviews
WHERE deleted_at IS NULL
  AND (
    approved_at IS NULL
    OR approved_at >= CURRENT_DATE - INTERVAL '30 days'
  )
ORDER BY submitted_at DESC;
```

### ❌ Bad Code — `review-filters.sql`

```sql
/* = NULL and loose OR logic do not match nullable data. */
SELECT id, title, approved_at
FROM code_reviews
WHERE deleted_at = NULL
  AND approved_at != NULL
   OR approved_at >= CURRENT_DATE - INTERVAL '30 days'
ORDER BY submitted_at DESC;
```

### 🔍 What to review

- **Good Code:** The good version uses `IS NULL` and groups the `OR` condition so deleted rows cannot leak into the result.
- **Bad Code:** The bad version compares to `NULL` with equality operators and relies on ambiguous operator precedence. It can return the wrong rows or no rows at all.

### 📝 Review notes

Nullable columns change how filters behave. In review, read every `AND` and `OR` with parentheses in mind, then check whether `NULL` should be included or excluded.

### 💡 Takeaway

> **NULL is not equal to anything, so review nullable filters with extra care.**

---

## 04. Join cardinality

> Review joins for one-to-one, one-to-many, and missing-row behavior before trusting the result.

**Tags:** `joins` `cardinality` `duplicates` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/join-cardinality)

### ✅ Good Code — `review-summary.sql`

```sql
/* Count comments first so each review stays one row. */
WITH comment_counts AS (
  SELECT
    review_id,
    COUNT(*) AS comment_count
  FROM review_comments
  GROUP BY review_id
)
SELECT
  reviews.id,
  reviews.title,
  COALESCE(comment_counts.comment_count, 0) AS comment_count
FROM code_reviews AS reviews
LEFT JOIN comment_counts
  ON comment_counts.review_id = reviews.id
WHERE reviews.author_id = $1;
```

### ❌ Bad Code — `review-summary.sql`

```sql
/* Joining comments directly creates one row per comment. */
SELECT
  reviews.id,
  reviews.title,
  review_comments.id AS comment_id
FROM code_reviews AS reviews
JOIN review_comments
  ON review_comments.review_id = reviews.id
WHERE reviews.author_id = $1;
```

### 🔍 What to review

- **Good Code:** The good version pre-aggregates comments and uses a left join so reviews without comments still appear once.
- **Bad Code:** The bad version turns each review into one row per comment and drops reviews that have no comments. That may silently corrupt pagination, counts, and UI summaries.

### 📝 Review notes

Before approving a join, ask what row count should come out. If the caller expects one row per review, joining directly to a many-side table is a review smell.

### 💡 Takeaway

> **A join changes row count, so review the relationship before reviewing the selected columns.**

---

## 05. Aggregation and grouping

> Group by the same dimensions you want to report, and make aggregate meaning clear.

**Tags:** `aggregate` `group by` `reporting` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/aggregation-grouping)

### ✅ Good Code — `review-metrics.sql`

```sql
/* Selected dimensions match the GROUP BY report question. */
SELECT
  DATE_TRUNC('day', submitted_at) AS review_day,
  status,
  COUNT(*) AS review_count
FROM code_reviews
WHERE submitted_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY review_day, status
ORDER BY review_day DESC, status;
```

### ❌ Bad Code — `review-metrics.sql`

```sql
/* submitted_at is selected but not part of the grouping intent. */
SELECT
  submitted_at,
  status,
  COUNT(*) AS review_count
FROM code_reviews
WHERE submitted_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY status
ORDER BY submitted_at DESC;
```

### 🔍 What to review

- **Good Code:** The good version reports counts by day and status, so both dimensions are selected, grouped, and ordered intentionally.
- **Bad Code:** The bad version mixes a raw timestamp with an aggregate grouped only by status. Depending on the database, it either fails or returns misleading values.

### 📝 Review notes

Aggregations answer a specific question. In review, write that question in plain language, then check that the `GROUP BY` columns match it exactly.

### 💡 Takeaway

> **Every non-aggregated selected value should be part of the grouping intent.**

---

## 06. Index-friendly predicates

> Keep indexed columns searchable by avoiding unnecessary functions on the column side.

**Tags:** `indexes` `where` `performance` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/index-friendly-predicates)

### ✅ Good Code — `monthly-reviews.sql`

```sql
/* Range predicates keep submitted_at searchable by index. */
SELECT id, title, submitted_at
FROM code_reviews
WHERE submitted_at >= TIMESTAMPTZ '2026-01-01'
  AND submitted_at < TIMESTAMPTZ '2026-02-01'
ORDER BY submitted_at DESC;
```

### ❌ Bad Code — `monthly-reviews.sql`

```sql
/* Formatting the column forces work on every row. */
SELECT id, title, submitted_at
FROM code_reviews
WHERE TO_CHAR(submitted_at, 'YYYY-MM') = '2026-01'
ORDER BY submitted_at DESC;
```

### 🔍 What to review

- **Good Code:** The good version expresses the month as a half-open range, which keeps an index on `submitted_at` useful.
- **Bad Code:** The bad version wraps the column in a formatting function for every row. It is readable, but it usually blocks normal index usage.

### 📝 Review notes

Performance review starts with shape, not guesswork. When a filter touches many rows, check whether the predicate lets the database seek through an index.

### 💡 Takeaway

> **A correct query can still be review-worthy if it prevents useful index access.**

---

## 07. Transactions for atomic workflows

> Wrap multi-step writes in a transaction so related changes succeed or fail together.

**Tags:** `transactions` `atomicity` `writes` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/transactions-atomic-workflows)

### ✅ Good Code — `approve-review.sql`

```sql
/* Both writes succeed or fail as one unit. */
BEGIN;

UPDATE code_reviews
SET status = 'approved',
    approved_at = now()
WHERE id = $1
  AND status = 'submitted';

INSERT INTO review_events (review_id, event_type, created_at)
VALUES ($1, 'approved', now());

COMMIT;
```

### ❌ Bad Code — `approve-review.sql`

```sql
/* Without a transaction, the audit event can fail after approval. */
UPDATE code_reviews
SET status = 'approved',
    approved_at = now()
WHERE id = $1;

INSERT INTO review_events (review_id, event_type, created_at)
VALUES ($1, 'approved', now());
```

### 🔍 What to review

- **Good Code:** The good version treats the status change and audit event as one unit of work.
- **Bad Code:** The bad version can approve a review without recording the event if the second statement fails. It also approves without checking the expected starting state.

### 📝 Review notes

Transactions are a product requirement hiding in database code. In review, ask what inconsistent state would exist if any statement in the middle failed.

### 💡 Takeaway

> **If one business action writes multiple rows, review where the transaction begins and ends.**

---

## 08. Migration backfills

> Split schema changes, backfills, and stricter constraints into safe migration steps.

**Tags:** `migrations` `backfill` `constraints` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/migration-backfills)

### ✅ Good Code — `20260525090000_add_display_name.sql`

```sql
/* Add the column, backfill old rows, then enforce NOT NULL. */
ALTER TABLE users
ADD COLUMN display_name TEXT;

UPDATE users
SET display_name = COALESCE(NULLIF(name, ''), email)
WHERE display_name IS NULL;

ALTER TABLE users
ALTER COLUMN display_name SET NOT NULL;
```

### ❌ Bad Code — `20260525090000_add_display_name.sql`

```sql
/* NOT NULL before backfill can fail on existing rows. */
ALTER TABLE users
ADD COLUMN display_name TEXT NOT NULL;

UPDATE users
SET display_name = name;
```

### 🔍 What to review

- **Good Code:** The good version adds the column, fills existing rows, then tightens the constraint after data is ready.
- **Bad Code:** The bad version adds a required column before existing rows have a value. It can fail immediately or force unsafe defaults.

### 📝 Review notes

Migration review is not just syntax review. Check the current data shape, table size, rollback story, and whether the application can run between migration steps.

### 💡 Takeaway

> **A safe migration respects the data that already exists.**

---

## 09. Parameterized queries

> Bind user input as parameters instead of building SQL strings from untrusted values.

**Tags:** `security` `injection` `parameters` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/parameterized-queries)

### ✅ Good Code — `users-repository.ts`

```ts
export async function findUserByEmail(email: string) {
  // Bound parameters keep user input out of the SQL text.
  const result = await db.query(
    "SELECT id, email, name FROM users WHERE email = $1",
    [email],
  );

  return result.rows[0] ?? null;
}
```

### ❌ Bad Code — `users-repository.ts`

```ts
export async function findUserByEmail(email: string) {
  // String concatenation turns user input into SQL text.
  const result = await db.query(
    "SELECT id, email, name FROM users WHERE email = '" + email + "'",
  );

  return result.rows[0] ?? null;
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the SQL text fixed and sends `email` as a bound parameter.
- **Bad Code:** The bad version inserts user input into the query string. A value that contains quotes or SQL syntax can change what the database executes.

### 📝 Review notes

In review, scan for string concatenation, template interpolation, and manual escaping around SQL. Parameter binding should be the default habit.

### 💡 Takeaway

> **User input should become query data, not query text.**

---

## 10. Stable pagination

> Use deterministic ordering and cursor predicates for pages that stay stable as data changes.

**Tags:** `pagination` `ordering` `cursors` · [View on site](https://good-code-bad-code.pages.dev/tracks/sql/stable-pagination)

### ✅ Good Code — `review-feed.sql`

```sql
/* Cursor uses timestamp plus id so the next page starts safely. */
SELECT id, title, submitted_at
FROM code_reviews
WHERE status = 'submitted'
  AND (submitted_at, id) < ($1, $2)
ORDER BY submitted_at DESC, id DESC
LIMIT 20;
```

### ❌ Bad Code — `review-feed.sql`

```sql
/* OFFSET can shift when new rows arrive between page loads. */
SELECT id, title, submitted_at
FROM code_reviews
WHERE status = 'submitted'
ORDER BY submitted_at DESC
LIMIT 20 OFFSET $1;
```

### 🔍 What to review

- **Good Code:** The good version uses a cursor made from `submitted_at` and `id`, giving the database a deterministic place to continue.
- **Bad Code:** The bad version uses offset pagination with a non-unique order. New rows can shift earlier pages, causing duplicates or missing items.

### 📝 Review notes

Offset pagination can fit low-change admin screens, but review feed-like queries for page drift. Stable ordering and a cursor prevent duplicates or missing items as rows arrive.

### 💡 Takeaway

> **Pagination needs a stable order, not just a limit.**

---

[← Back to index](README.md)
