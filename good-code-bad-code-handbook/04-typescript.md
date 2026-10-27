# TypeScript — Code Review Handbook

> Review type narrowing, API boundaries, unions, and safer function shapes.

- **Category:** Foundations
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/typescript](https://good-code-bad-code.pages.dev/tracks/typescript)
- **Related tracks:** [JavaScript](03-javascript.md), [React](05-react.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Type inference and explicit boundaries](#01-type-inference-and-explicit-boundaries)** — Annotate function inputs, return contracts, and exported APIs while letting local implementation details infer naturally.
- [ ] **[Narrowing unknown](#02-narrowing-unknown)** — Narrow unknown input to make invalid states harder to express after validation.
- [ ] **[Avoiding unsafe assertions](#03-avoiding-unsafe-assertions)** — Treat `as` as a last resort at trusted boundaries, not a way to skip validation.
- [ ] **[Discriminated unions](#04-discriminated-unions)** — Use discriminated unions to make invalid states harder to express.
- [ ] **[Exhaustive checks with never](#05-exhaustive-checks-with-never)** — Use a `never` assignment in the final branch to make missing union cases a type error.
- [ ] **[Precise function types](#06-precise-function-types)** — Use precise function signatures to make invalid calls harder to express.
- [ ] **[Optional properties vs nullable values](#07-optional-properties-vs-nullable-values)** — Use `?` when a property may be omitted and `null` when the property is present but intentionally empty.
- [ ] **[Generic constraints](#08-generic-constraints)** — Use `extends` to keep generic functions flexible without losing the properties they rely on.
- [ ] **[Utility types at API boundaries](#09-utility-types-at-api-boundaries)** — Use Pick, Omit, Partial, and Readonly to express boundary-specific contracts from existing domain types.
- [ ] **[Type-only imports and module boundaries](#10-type-only-imports-and-module-boundaries)** — Use `import type` when a module only needs a type and keep runtime imports for values that execute.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Type inference and explicit boundaries](#01-type-inference-and-explicit-boundaries) | `inference` `boundaries` `api-design` |
| 02 | [Narrowing unknown](#02-narrowing-unknown) | `unknown` `narrowing` `runtime-data` |
| 03 | [Avoiding unsafe assertions](#03-avoiding-unsafe-assertions) | `assertions` `safety` `runtime-data` |
| 04 | [Discriminated unions](#04-discriminated-unions) | `unions` `state` `narrowing` |
| 05 | [Exhaustive checks with never](#05-exhaustive-checks-with-never) | `unions` `exhaustive` `never` |
| 06 | [Precise function types](#06-precise-function-types) | `functions` `callbacks` `api-design` |
| 07 | [Optional properties vs nullable values](#07-optional-properties-vs-nullable-values) | `optional` `nullable` `data-modeling` |
| 08 | [Generic constraints](#08-generic-constraints) | `generics` `constraints` `api-design` |
| 09 | [Utility types at API boundaries](#09-utility-types-at-api-boundaries) | `utility-types` `api-design` `boundaries` |
| 10 | [Type-only imports and module boundaries](#10-type-only-imports-and-module-boundaries) | `imports` `modules` `boundaries` |

---

## 01. Type inference and explicit boundaries

> Let TypeScript infer local values, but type the boundaries other code depends on.

**Tags:** `inference` `boundaries` `api-design` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/type-inference-boundaries)

### ✅ Good Code — `review-summary.ts`

```ts
type ReviewSummaryInput = {
  title: string;
  commentCount: number;
  state: "open" | "closed";
};

export function createReviewSummary(input: ReviewSummaryInput) {
  // Boundary type documents what callers must provide.
  const label = input.title + " (" + input.commentCount + ")";

  return {
    label,
    isClosed: input.state === "closed",
  };
}
```

### ❌ Bad Code — `review-summary.ts`

```ts
export function createReviewSummary(input: any) {
  // Local annotations do not protect an any input boundary.
  const label: string = input.title + " (" + input.comments.length + ")";
  const isClosed: boolean = input.closed;

  return {
    label,
    isClosed,
  };
}
```

### 🔍 What to review

- **Good Code:** The good version types the input boundary and lets the local `label` and returned object infer from real values.
- **Bad Code:** The bad version annotates local variables while leaving the public input as `any`, so reviewers still cannot trust the shape that matters most.

### 📝 Review notes

In review, look for annotations in the wrong place. Types are most valuable where code crosses a boundary: exported functions, API adapters, callback contracts, and data that comes from outside the module.

### 💡 Takeaway

> **Annotate function inputs, return contracts, and exported APIs while letting local implementation details infer naturally.**

---

## 02. Narrowing unknown

> Check unknown values before using them as trusted application data.

**Tags:** `unknown` `narrowing` `runtime-data` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/narrowing-unknown)

### ✅ Good Code — `parse-settings.ts`

```ts
type Settings = {
  theme: "light" | "dark";
};

function isSettings(value: unknown): value is Settings {
  return (
    typeof value === "object" &&
    value !== null &&
    "theme" in value &&
    (value.theme === "light" || value.theme === "dark")
  );
}

function readSettings(value: unknown): Settings {
  // The type guard earns trust before returning Settings.
  if (!isSettings(value)) {
    return { theme: "light" };
  }

  return value;
}
```

### ❌ Bad Code — `parse-settings.ts`

```ts
type Settings = {
  theme: "light" | "dark";
};

function readSettings(value: unknown): Settings {
  // Double casting skips the runtime proof unknown requires.
  return value as any as Settings;
}
```

### 🔍 What to review

- **Good Code:** The good version treats incoming data as unknown until a small runtime check proves it has the expected shape.
- **Bad Code:** The bad version tells TypeScript to trust the value without earning that trust, so invalid data can travel through the app as if it were safe.

### 📝 Review notes

In review, ask where the boundary between untrusted data and typed data is enforced. Casting through `any` is review-worthy because it removes the compiler's ability to make invalid states harder to express.

### 💡 Takeaway

> **Narrow unknown input to make invalid states harder to express after validation.**

---

## 03. Avoiding unsafe assertions

> Prefer checks that prove a value's shape over assertions that silence the compiler.

**Tags:** `assertions` `safety` `runtime-data` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/unsafe-assertions)

### ✅ Good Code — `session.ts`

```ts
type Session = {
  userId: string;
  expiresAt: string;
};

function isSession(value: unknown): value is Session {
  return (
    typeof value === "object" &&
    value !== null &&
    "userId" in value &&
    typeof value.userId === "string" &&
    "expiresAt" in value &&
    typeof value.expiresAt === "string"
  );
}

export function readSession(value: unknown): Session | undefined {
  // Runtime checks prove the value before exposing Session.
  if (!isSession(value)) {
    return undefined;
  }

  return value;
}
```

### ❌ Bad Code — `session.ts`

```ts
type Session = {
  userId: string;
  expiresAt: string;
};

export function readSession(value: unknown): Session {
  // Assertion turns unknown data into Session without evidence.
  return value as Session;
}
```

### 🔍 What to review

- **Good Code:** The good version earns trust by checking the runtime shape before returning typed application data.
- **Bad Code:** The bad version tells TypeScript to accept the value as a `Session` without proving anything, so invalid data can move through the app as if it were safe.

### 📝 Review notes

In review, assertions are worth a second look whenever the value came from JSON, storage, network, URL params, or third-party code. The closer the assertion is to untrusted data, the more evidence it needs.

### 💡 Takeaway

> **Treat `as` as a last resort at trusted boundaries, not a way to skip validation.**

---

## 04. Discriminated unions

> Model related states with a discriminant so callers can narrow safely.

**Tags:** `unions` `state` `narrowing` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/discriminated-unions)

### ✅ Good Code — `result.ts`

```ts
type SaveResult =
  | { status: "saved"; id: string }
  | { status: "failed"; message: string };

function renderSaveResult(result: SaveResult) {
  // The status field selects the only valid shape for each branch.
  if (result.status === "saved") {
    return "Saved draft " + result.id;
  }

  return "Could not save: " + result.message;
}
```

### ❌ Bad Code — `result.ts`

```ts
type SaveResult = {
  saved?: boolean;
  id?: string;
  message?: string;
};

function renderSaveResult(result: SaveResult) {
  // Optional fields allow impossible combinations to compile.
  if (result.saved) {
    return "Saved draft " + result.id;
  }

  return "Could not save: " + result.message;
}
```

### 🔍 What to review

- **Good Code:** The good version ties each shape to a clear `status`, so TypeScript can narrow the available fields in each branch.
- **Bad Code:** The bad version leaves optional fields to be guessed together, allowing states like `saved: true` without an `id` or a failure without a message.

### 📝 Review notes

In review, look for booleans and optional fields that are acting as a hidden protocol. The bad version is review-worthy because it asks every caller to remember combinations that the type system could make impossible.

### 💡 Takeaway

> **Use discriminated unions to make invalid states harder to express.**

---

## 05. Exhaustive checks with never

> Give every union case a branch so new states fail loudly during development.

**Tags:** `unions` `exhaustive` `never` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/exhaustive-never-checks)

### ✅ Good Code — `review-state.ts`

```ts
type ReviewState =
  | { status: "draft" }
  | { status: "open"; reviewerCount: number }
  | { status: "merged"; mergedBy: string };

function renderReviewState(state: ReviewState) {
  // Assigning to never makes missing union cases a compile error.
  switch (state.status) {
    case "draft":
      return "Draft";
    case "open":
      return "Open for " + state.reviewerCount + " reviewers";
    case "merged":
      return "Merged by " + state.mergedBy;
    default: {
      const neverState: never = state;
      return neverState;
    }
  }
}
```

### ❌ Bad Code — `review-state.ts`

```ts
type ReviewState =
  | { status: "draft" }
  | { status: "open"; reviewerCount: number }
  | { status: "merged"; mergedBy: string };

function renderReviewState(state: ReviewState) {
  // Generic fallback hides states without a branch.
  if (state.status === "draft") {
    return "Draft";
  }

  if (state.status === "open") {
    return "Open for " + state.reviewerCount + " reviewers";
  }

  return "Unknown state";
}
```

### 🔍 What to review

- **Good Code:** The good version makes the final branch impossible after every union case has its own branch, so adding a new state creates a compile-time reminder.
- **Bad Code:** The bad version hides missing cases behind a generic fallback, which can make new product states render as unknown instead of forcing a real branch for that state.

### 📝 Review notes

In review, default branches over discriminated unions can be too forgiving. If every case is known, exhaustive checking gives future changes a useful tripwire.

### 💡 Takeaway

> **Use a `never` assignment in the final branch to make missing union cases a type error.**

---

## 06. Precise function types

> Describe callbacks with the arguments and return value the caller actually supports.

**Tags:** `functions` `callbacks` `api-design` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/precise-function-types)

### ✅ Good Code — `review-actions.ts`

```ts
type ReviewAction = "approve" | "request-changes";
type ReviewHandler = (action: ReviewAction, comment: string) => void;

function submitReview(handleReview: ReviewHandler) {
  // The callback type limits actions and required arguments.
  handleReview("approve", "Ready to merge.");
}
```

### ❌ Bad Code — `review-actions.ts`

```ts
function submitReview(handleReview: Function) {
  // Function accepts any call shape, so invalid calls compile.
  handleReview("approve");
  handleReview("delete-branch", 42, true);
}
```

### 🔍 What to review

- **Good Code:** The good version documents exactly which actions and arguments the callback accepts.
- **Bad Code:** The bad version accepts any callable value, so TypeScript cannot help reviewers spot missing arguments, unsupported actions, or unexpected return contracts.

### 📝 Review notes

In review, treat `Function` as a sign that the API boundary is underspecified. The bad version is review-worthy because broad callback types make invalid calls easier to express and harder to catch before runtime.

### 💡 Takeaway

> **Use precise function signatures to make invalid calls harder to express.**

---

## 07. Optional properties vs nullable values

> Use optional and nullable fields to describe different kinds of absence.

**Tags:** `optional` `nullable` `data-modeling` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/optional-vs-nullable)

### ✅ Good Code — `profile.ts`

```ts
type ProfileDraft = {
  displayName?: string;
  avatarUrl: string | null;
};

function renderAvatar(profile: ProfileDraft) {
  // null means avatar is intentionally empty, not merely missing.
  if (profile.avatarUrl === null) {
    return "Use default avatar";
  }

  return profile.avatarUrl;
}
```

### ❌ Bad Code — `profile.ts`

```ts
type ProfileDraft = {
  displayName?: string | null;
  avatarUrl?: string | null;
};

function renderAvatar(profile: ProfileDraft) {
  // Mixing optional and null forces callers to guess absence.
  return profile.avatarUrl || "Use default avatar";
}
```

### 🔍 What to review

- **Good Code:** The good version separates an omitted draft field from a present field that explicitly has no avatar.
- **Bad Code:** The bad version allows both `undefined` and `null` everywhere, so callers cannot tell whether data was missing, intentionally cleared, or simply not loaded yet.

### 📝 Review notes

In review, absence is part of the domain model. Optional and nullable describe different forms of absence, and mixing them casually makes every consumer repeat the same guessing game.

### 💡 Takeaway

> **Use `?` when a property may be omitted and `null` when the property is present but intentionally empty.**

---

## 08. Generic constraints

> Constrain generics to the fields the function actually needs.

**Tags:** `generics` `constraints` `api-design` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/generic-constraints)

### ✅ Good Code — `sort-by-updated.ts`

```ts
type HasUpdatedAt = {
  updatedAt: string;
};

function sortByUpdatedAt<T extends HasUpdatedAt>(items: T[]): T[] {
  // The constraint keeps the required field in the contract.
  return [...items].sort((left, right) =>
    right.updatedAt.localeCompare(left.updatedAt),
  );
}
```

### ❌ Bad Code — `sort-by-updated.ts`

```ts
function sortByUpdatedAt<T>(items: T[]): T[] {
  // any hides the missing constraint until runtime.
  return [...items].sort((left, right) =>
    (right as any).updatedAt.localeCompare((left as any).updatedAt),
  );
}
```

### 🔍 What to review

- **Good Code:** The good version stays generic while making the required `updatedAt` property part of the type contract.
- **Bad Code:** The bad version accepts any item type and then escapes through `any` inside the implementation, so invalid calls compile and fail later.

### 📝 Review notes

In review, a generic that immediately casts to `any` is usually missing a constraint. The type parameter should describe what can vary and what the function still requires.

### 💡 Takeaway

> **Use `extends` to keep generic functions flexible without losing the properties they rely on.**

---

## 09. Utility types at API boundaries

> Derive focused request and view types from domain types without exposing everything.

**Tags:** `utility-types` `api-design` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/utility-types-api-boundaries)

### ✅ Good Code — `review-api.ts`

```ts
type PullRequest = {
  id: string;
  title: string;
  authorId: string;
  mergedAt: string | null;
  internalScore: number;
};

type PullRequestListItem = Pick<PullRequest, "id" | "title" | "authorId">;

function serializePullRequest(item: PullRequest): PullRequestListItem {
  // Pick exposes only the fields this boundary promises.
  return {
    id: item.id,
    title: item.title,
    authorId: item.authorId,
  };
}
```

### ❌ Bad Code — `review-api.ts`

```ts
type PullRequest = {
  id: string;
  title: string;
  authorId: string;
  mergedAt: string | null;
  internalScore: number;
};

function serializePullRequest(item: PullRequest): PullRequest {
  // Returning the domain object leaks internal fields.
  return item;
}
```

### 🔍 What to review

- **Good Code:** The good version exposes only the fields the list API needs while keeping the response type tied to the domain model.
- **Bad Code:** The bad version returns the full domain object, including internal fields that a boundary should not promise to external callers.

### 📝 Review notes

In review, utility types are useful when they shrink a boundary to the fields it promises. They should express a product contract, not just make a complex type shorter.

### 💡 Takeaway

> **Use Pick, Omit, Partial, and Readonly to express boundary-specific contracts from existing domain types.**

---

## 10. Type-only imports and module boundaries

> Separate type dependencies from runtime dependencies so modules stay lightweight.

**Tags:** `imports` `modules` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/typescript/type-only-imports-module-boundaries)

### ✅ Good Code — `review-card.ts`

```ts
import type { PullRequest } from "./pull-request";
import { formatAuthorName } from "./authors";

export function getReviewCardTitle(pullRequest: PullRequest) {
  // import type keeps PullRequest out of the runtime dependency graph.
  return pullRequest.title + " by " + formatAuthorName(pullRequest.authorId);
}
```

### ❌ Bad Code — `review-card.ts`

```ts
import { PullRequest } from "./pull-request";
import { formatAuthorName } from "./authors";

export function getReviewCardTitle(pullRequest: PullRequest) {
  // A value import blurs type-only and runtime dependencies.
  return pullRequest.title + " by " + formatAuthorName(pullRequest.authorId);
}
```

### 🔍 What to review

- **Good Code:** The good version makes it clear that `PullRequest` is only a type dependency while `formatAuthorName` is a runtime value dependency.
- **Bad Code:** The bad version imports a type like a runtime value, which can blur module boundaries and create unnecessary runtime coupling depending on compiler settings.

### 📝 Review notes

In review, imports tell a dependency story. `import type` helps separate compile-time shape from runtime behavior, which makes module boundaries easier to reason about.

### 💡 Takeaway

> **Use `import type` when a module only needs a type and keep runtime imports for values that execute.**

---

[← Back to index](README.md)
