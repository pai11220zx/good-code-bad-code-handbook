# React — Code Review Handbook

> Review component boundaries, state, effects, and rendering decisions.

- **Category:** Frontend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/react](https://good-code-bad-code.pages.dev/tracks/react)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Props component boundaries](#01-props-component-boundaries)** — Prefer explicit props for presentational components over hidden app-state reach-ins.
- [ ] **[Stable keys in lists](#02-stable-keys-in-lists)** — Prefer durable IDs for list keys instead of array positions.
- [ ] **[Derived state](#03-derived-state)** — Prefer render-time derived values when state can be calculated from existing data.
- [ ] **[Updating state from previous state](#04-updating-state-from-previous-state)** — When state depends on prior state, pass an updater function to the setter.
- [ ] **[Controlled form inputs](#05-controlled-form-inputs)** — Controlled inputs make the displayed value and submitted value come from one source.
- [ ] **[Effect dependencies](#06-effect-dependencies)** — Complete dependency arrays make effects follow the data they read.
- [ ] **[Async effect cleanup](#07-async-effect-cleanup)** — Clean up async effects when inputs change or the component unmounts.
- [ ] **[Context boundaries](#08-context-boundaries)** — Keep context narrow so components do not subscribe to unrelated app state.
- [ ] **[Memoization when it helps](#09-memoization-when-it-helps)** — Memoization should have a clear cost to avoid and a complete dependency list.
- [ ] **[Composition over prop flags](#10-composition-over-prop-flags)** — Use children and named slots when a component has multiple content variants.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Props component boundaries](#01-props-component-boundaries) | `components` `props` `boundaries` |
| 02 | [Stable keys in lists](#02-stable-keys-in-lists) | `lists` `keys` `identity` |
| 03 | [Derived state](#03-derived-state) | `state` `rendering` `effects` |
| 04 | [Updating state from previous state](#04-updating-state-from-previous-state) | `state` `events` `updates` |
| 05 | [Controlled form inputs](#05-controlled-form-inputs) | `forms` `state` `events` |
| 06 | [Effect dependencies](#06-effect-dependencies) | `effects` `async` `state` |
| 07 | [Async effect cleanup](#07-async-effect-cleanup) | `effects` `async` `cleanup` |
| 08 | [Context boundaries](#08-context-boundaries) | `context` `components` `state` |
| 09 | [Memoization when it helps](#09-memoization-when-it-helps) | `memoization` `performance` `rendering` |
| 10 | [Composition over prop flags](#10-composition-over-prop-flags) | `composition` `components` `props` |

---

## 01. Props component boundaries

> Keep presentational components focused by passing the data they need explicitly.

**Tags:** `components` `props` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/props-component-boundaries)

### ✅ Good Code — `ReviewSummary.tsx`

```tsx
type ReviewSummaryProps = {
  title: string;
  reviewerName: string;
  status: "approved" | "changes-requested";
};

export function ReviewSummary({
  title,
  reviewerName,
  status,
}: ReviewSummaryProps) {
  // Explicit props make this component easy to move and test.
  return (
    <section>
      <h2>{title}</h2>
      <p>{reviewerName} marked this review as {status}.</p>
    </section>
  );
}
```

### ❌ Bad Code — `ReviewSummary.tsx`

```tsx
import { useAppState } from "./app-state";

export function ReviewSummary() {
  // Hidden app-state reads make the component depend on too much.
  const { currentProject, activeReview, viewer } = useAppState();

  return (
    <section>
      <h2>{currentProject.name}: {activeReview.title}</h2>
      <p>{viewer.name} marked this review as {activeReview.status}.</p>
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version makes the component boundary obvious: the component receives the display values it needs and renders them.
- **Bad Code:** The bad version reaches into broad application state, which couples a reusable summary to unrelated project, review, and viewer concerns.

### 📝 Review notes

In review, ask whether a component can be understood, tested, and reused from its props alone. The bad version is review-worthy because it renders correctly in one app context while hiding dependencies that make the component harder to move or reason about.

### 💡 Takeaway

> **Prefer explicit props for presentational components over hidden app-state reach-ins.**

---

## 02. Stable keys in lists

> Use stable item identities as keys so React can preserve the right state.

**Tags:** `lists` `keys` `identity` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/stable-keys-lists)

### ✅ Good Code — `CommentList.tsx`

```tsx
type Comment = {
  id: string;
  authorName: string;
  body: string;
};

export function CommentList({ comments }: { comments: Comment[] }) {
  // Stable IDs preserve the right item identity across list changes.
  return comments.map((comment) => (
    <article key={comment.id}>
      <h3>{comment.authorName}</h3>
      <p>{comment.body}</p>
    </article>
  ));
}
```

### ❌ Bad Code — `CommentList.tsx`

```tsx
type Comment = {
  id: string;
  authorName: string;
  body: string;
};

export function CommentList({ comments }: { comments: Comment[] }) {
  // Index keys describe position, not the comment's identity.
  return comments.map((comment, index) => (
    <article key={index}>
      <h3>{comment.authorName}</h3>
      <p>{comment.body}</p>
    </article>
  ));
}
```

### 🔍 What to review

- **Good Code:** The good version uses the comment ID as the key, so React can keep each rendered item attached to the same comment when the list changes.
- **Bad Code:** The bad version uses the array index, which only represents a position. Insertions, removals, and sorting can make React reuse the wrong DOM or component state for another comment.

### 📝 Review notes

In review, look for keys that survive reordering. The bad version is review-worthy because it often looks fine in static data while causing subtle bugs once users add, delete, filter, or sort items.

### 💡 Takeaway

> **Prefer durable IDs for list keys instead of array positions.**

---

## 03. Derived state

> Compute values from current props and state instead of syncing duplicates.

**Tags:** `state` `rendering` `effects` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/derived-state)

### ✅ Good Code — `ReviewerList.tsx`

```tsx
type Reviewer = {
  id: string;
  name: string;
  approved: boolean;
};

export function ReviewerList({ reviewers }: { reviewers: Reviewer[] }) {
  // Derived values can be computed from current props during render.
  const approvedCount = reviewers.filter((reviewer) => reviewer.approved).length;

  return <p>{approvedCount} of {reviewers.length} reviewers approved.</p>;
}
```

### ❌ Bad Code — `ReviewerList.tsx`

```tsx
import { useEffect, useState } from "react";

type Reviewer = {
  id: string;
  name: string;
  approved: boolean;
};

export function ReviewerList({ reviewers }: { reviewers: Reviewer[] }) {
  const [approvedCount, setApprovedCount] = useState(0);

  // Duplicated state needs an effect just to stay synchronized.
  useEffect(() => {
    setApprovedCount(reviewers.filter((reviewer) => reviewer.approved).length);
  }, [reviewers]);

  return <p>{approvedCount} of {reviewers.length} reviewers approved.</p>;
}
```

### 🔍 What to review

- **Good Code:** The good version derives the count from the current `reviewers` prop during render, so there is only one source of truth.
- **Bad Code:** The bad version stores a duplicate count and then uses an effect to keep it synchronized with props.

### 📝 Review notes

In review, ask whether the value needs to survive independently or can be recomputed from current inputs. The bad version is review-worthy because duplicated state renders fine but creates timing, synchronization, and missing-dependency risks for data React already has.

### 💡 Takeaway

> **Prefer render-time derived values when state can be calculated from existing data.**

---

## 04. Updating state from previous state

> Use functional state updates when the next value depends on the current value.

**Tags:** `state` `events` `updates` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/updating-state-from-previous-state)

### ✅ Good Code — `ReviewerPicker.tsx`

```tsx
import { useState } from "react";

type Reviewer = {
  id: string;
  name: string;
};

export function ReviewerPicker({ reviewers }: { reviewers: Reviewer[] }) {
  const [selectedIds, setSelectedIds] = useState<string[]>([]);

  function toggleReviewer(id: string) {
    // Functional updates read the latest state React has queued.
    setSelectedIds((currentIds) =>
      currentIds.includes(id)
        ? currentIds.filter((currentId) => currentId !== id)
        : [...currentIds, id],
    );
  }

  return reviewers.map((reviewer) => (
    <button
      key={reviewer.id}
      type="button"
      aria-pressed={selectedIds.includes(reviewer.id)}
      onClick={() => toggleReviewer(reviewer.id)}
    >
      {reviewer.name}
    </button>
  ));
}
```

### ❌ Bad Code — `ReviewerPicker.tsx`

```tsx
import { useState } from "react";

type Reviewer = {
  id: string;
  name: string;
};

export function ReviewerPicker({ reviewers }: { reviewers: Reviewer[] }) {
  const [selectedIds, setSelectedIds] = useState<string[]>([]);

  function toggleReviewer(id: string) {
    // This uses the selectedIds captured by the current render.
    if (selectedIds.includes(id)) {
      setSelectedIds(selectedIds.filter((currentId) => currentId !== id));
      return;
    }

    setSelectedIds([...selectedIds, id]);
  }

  return reviewers.map((reviewer) => (
    <button
      key={reviewer.id}
      type="button"
      aria-pressed={selectedIds.includes(reviewer.id)}
      onClick={() => toggleReviewer(reviewer.id)}
    >
      {reviewer.name}
    </button>
  ));
}
```

### 🔍 What to review

- **Good Code:** The good version asks React for the latest selected IDs before calculating the next state.
- **Bad Code:** The bad version calculates from the value captured by the current render. That can lose updates when multiple events or batched changes happen close together.

### 📝 Review notes

In review, scan setter calls for values derived from the current state variable. The bad version is review-worthy because it is readable and common, but it can fail under fast interaction, batching, or callbacks that run after the render that created them.

### 💡 Takeaway

> **When state depends on prior state, pass an updater function to the setter.**

---

## 05. Controlled form inputs

> Keep form values in React state when the UI needs validation, reset, or submit logic.

**Tags:** `forms` `state` `events` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/controlled-form-inputs)

### ✅ Good Code — `ReviewTitleForm.tsx`

```tsx
import { type FormEvent, useState } from "react";

type ReviewTitleFormProps = {
  onCreate: (title: string) => void;
};

export function ReviewTitleForm({ onCreate }: ReviewTitleFormProps) {
  const [title, setTitle] = useState("");

  function handleSubmit(event: FormEvent<HTMLFormElement>) {
    // React state is the single source for validation and reset.
    event.preventDefault();

    const trimmedTitle = title.trim();
    if (!trimmedTitle) {
      return;
    }

    onCreate(trimmedTitle);
    setTitle("");
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="review-title">Review title</label>
      <input
        id="review-title"
        value={title}
        onChange={(event) => setTitle(event.target.value)}
      />
      <button type="submit">Create review</button>
    </form>
  );
}
```

### ❌ Bad Code — `ReviewTitleForm.tsx`

```tsx
import type { FormEvent } from "react";

type ReviewTitleFormProps = {
  onCreate: (title: string) => void;
};

export function ReviewTitleForm({ onCreate }: ReviewTitleFormProps) {
  function handleSubmit(event: FormEvent<HTMLFormElement>) {
    // DOM queries bypass React's form state and testing surface.
    event.preventDefault();

    const input = document.querySelector<HTMLInputElement>("#review-title");
    const title = input?.value.trim();

    if (title) {
      onCreate(title);
      input.value = "";
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="review-title">Review title</label>
      <input id="review-title" defaultValue="" />
      <button type="submit">Create review</button>
    </form>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the input value in React state, so validation, submit behavior, and reset behavior all use the same value.
- **Bad Code:** The bad version reaches into the DOM to read and mutate the input. That bypasses React's data flow and makes the form harder to test or extend.

### 📝 Review notes

In review, ask whether the form value has behavior beyond native browser submission. The bad version is review-worthy because DOM queries hide the value from React, which becomes painful when adding validation, disabled states, autosave, or error messages.

### 💡 Takeaway

> **Controlled inputs make the displayed value and submitted value come from one source.**

---

## 06. Effect dependencies

> List every effect dependency and clean up async work when inputs change.

**Tags:** `effects` `async` `state` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/effect-dependencies)

### ✅ Good Code — `ReviewComments.tsx`

```tsx
import { useEffect, useState } from "react";

type Comment = {
  id: string;
  body: string;
};

export function ReviewComments({ reviewId }: { reviewId: string }) {
  const [comments, setComments] = useState<Comment[]>([]);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // The effect follows reviewId and cancels stale network work.
    const controller = new AbortController();

    async function loadComments() {
      setError(null);
      const response = await fetch("/api/reviews/" + reviewId + "/comments", {
        signal: controller.signal,
      });

      if (!response.ok) {
        setError("Could not load comments.");
        return;
      }

      setComments(await response.json());
    }

    loadComments().catch((error) => {
      if (error.name !== "AbortError") {
        setError("Could not load comments.");
      }
    });

    return () => controller.abort();
  }, [reviewId]);

  if (error) {
    return <p role="alert">{error}</p>;
  }

  return comments.map((comment) => <p key={comment.id}>{comment.body}</p>);
}
```

### ❌ Bad Code — `ReviewComments.tsx`

```tsx
import { useEffect, useState } from "react";

type Comment = {
  id: string;
  body: string;
};

export function ReviewComments({ reviewId }: { reviewId: string }) {
  const [comments, setComments] = useState<Comment[]>([]);

  useEffect(() => {
    // Empty dependencies freeze the first reviewId this render saw.
    fetch("/api/reviews/" + reviewId + "/comments")
      .then((response) => response.json())
      .then(setComments);
  }, []);

  return comments.map((comment) => <p key={comment.id}>{comment.body}</p>);
}
```

### 🔍 What to review

- **Good Code:** The good version lists `reviewId` as a dependency and aborts in-flight work when the component unmounts or the review changes.
- **Bad Code:** The bad version captures the initial `reviewId` forever, so it can display stale comments when the prop changes.

### 📝 Review notes

In review, compare every value read inside an effect with the dependency array. The bad version is review-worthy because it may pass casual testing on the first render while failing as soon as navigation reuses the component for another review.

### 💡 Takeaway

> **Complete dependency arrays make effects follow the data they read.**

---

## 07. Async effect cleanup

> Guard async effects so stale responses cannot update the current screen.

**Tags:** `effects` `async` `cleanup` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/async-effect-cleanup)

### ✅ Good Code — `ReviewDetails.tsx`

```tsx
import { useEffect, useState } from "react";

type Review = {
  id: string;
  title: string;
  status: string;
};

async function fetchReview(reviewId: string): Promise<Review> {
  const response = await fetch("/api/reviews/" + reviewId);

  if (!response.ok) {
    throw new Error("Could not load review.");
  }

  return response.json();
}

export function ReviewDetails({ reviewId }: { reviewId: string }) {
  const [review, setReview] = useState<Review | null>(null);
  const [status, setStatus] = useState<"idle" | "loading" | "ready" | "error">(
    "idle",
  );

  useEffect(() => {
    // Cleanup prevents stale responses from updating the current screen.
    let cancelled = false;

    setStatus("loading");
    fetchReview(reviewId)
      .then((nextReview) => {
        if (!cancelled) {
          setReview(nextReview);
          setStatus("ready");
        }
      })
      .catch(() => {
        if (!cancelled) {
          setStatus("error");
        }
      });

    return () => {
      cancelled = true;
    };
  }, [reviewId]);

  if (status === "loading") {
    return <p>Loading review...</p>;
  }

  if (status === "error") {
    return <p role="alert">Could not load review.</p>;
  }

  return review ? <h2>{review.title}</h2> : null;
}
```

### ❌ Bad Code — `ReviewDetails.tsx`

```tsx
import { useEffect, useState } from "react";

type Review = {
  id: string;
  title: string;
  status: string;
};

async function fetchReview(reviewId: string): Promise<Review> {
  const response = await fetch("/api/reviews/" + reviewId);
  return response.json();
}

export function ReviewDetails({ reviewId }: { reviewId: string }) {
  const [review, setReview] = useState<Review | null>(null);

  useEffect(() => {
    // Older requests can still win the race and overwrite state.
    fetchReview(reviewId).then(setReview);
  }, [reviewId]);

  return review ? <h2>{review.title}</h2> : <p>Loading review...</p>;
}
```

### 🔍 What to review

- **Good Code:** The good version marks the request as cancelled during cleanup, so an older response cannot update state after the component has moved on.
- **Bad Code:** The bad version starts a new request for every `reviewId` but allows older requests to finish later and overwrite the latest review.

### 📝 Review notes

In review, check what happens if the user navigates quickly or the component unmounts before a promise resolves. The bad version is review-worthy because it has the right dependency array but still misses the lifecycle cleanup that protects the current UI.

### 💡 Takeaway

> **Clean up async effects when inputs change or the component unmounts.**

---

## 08. Context boundaries

> Use context for shared ambient data, not as a shortcut around component boundaries.

**Tags:** `context` `components` `state` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/context-boundaries)

### ✅ Good Code — `ReviewCard.tsx`

```tsx
import { createContext, useContext } from "react";

type Viewer = {
  id: string;
  name: string;
  role: "viewer" | "maintainer";
};

const ViewerContext = createContext<Viewer | null>(null);

function useViewer() {
  const viewer = useContext(ViewerContext);

  if (!viewer) {
    throw new Error("ReviewCard must be rendered inside ViewerContext.");
  }

  return viewer;
}

type ReviewCardProps = {
  title: string;
  reviewerName: string;
  status: "approved" | "changes-requested" | "pending";
};

export function ReviewCard({
  title,
  reviewerName,
  status,
}: ReviewCardProps) {
  const viewer = useViewer();
  // Context stays narrow while review data remains explicit props.
  const canApprove = viewer.role === "maintainer";

  return (
    <article>
      <h2>{title}</h2>
      <p>{reviewerName} marked this review as {status}.</p>
      {canApprove ? <button type="button">Approve</button> : null}
    </article>
  );
}
```

### ❌ Bad Code — `ReviewCard.tsx`

```tsx
import { createContext, useContext } from "react";

type AppState = {
  viewer: { id: string; name: string; role: string };
  currentProject: { id: string; name: string };
  activeReview: { title: string; reviewerName: string; status: string };
  searchQuery: string;
  sidebarOpen: boolean;
};

const AppContext = createContext<AppState | null>(null);

export function ReviewCard() {
  // A broad app context subscribes this card to unrelated state.
  const appState = useContext(AppContext);

  if (!appState) {
    return null;
  }

  return (
    <article>
      <h2>{appState.currentProject.name}: {appState.activeReview.title}</h2>
      <p>
        {appState.activeReview.reviewerName} marked this review as{" "}
        {appState.activeReview.status}.
      </p>
      {appState.viewer.role === "maintainer" ? (
        <button type="button">Approve</button>
      ) : null}
    </article>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version uses context only for ambient viewer information and keeps review data explicit at the component boundary.
- **Bad Code:** The bad version treats a broad app context as a global bag. The component subscribes to project, review, search, sidebar, and viewer data even though it only needs a small subset.

### 📝 Review notes

In review, look at how much data a context consumer receives compared with what it renders. The bad version is review-worthy because broad context makes components harder to reuse and can cause rerenders from unrelated state changes.

### 💡 Takeaway

> **Keep context narrow so components do not subscribe to unrelated app state.**

---

## 09. Memoization when it helps

> Use memoization for repeated expensive work with correct dependencies, not as decoration.

**Tags:** `memoization` `performance` `rendering` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/memoization-when-it-helps)

### ✅ Good Code — `ReviewSearch.tsx`

```tsx
import { useMemo } from "react";

type Review = {
  id: string;
  title: string;
  authorName: string;
};

function matchesSearch(review: Review, search: string) {
  const query = search.toLowerCase();
  return (
    review.title.toLowerCase().includes(query) ||
    review.authorName.toLowerCase().includes(query)
  );
}

export function ReviewSearch({
  reviews,
  search,
}: {
  reviews: Review[];
  search: string;
}) {
  // Memoization is tied to the expensive list calculation inputs.
  const visibleReviews = useMemo(
    () => reviews.filter((review) => matchesSearch(review, search)),
    [reviews, search],
  );

  return (
    <ul>
      {visibleReviews.map((review) => (
        <li key={review.id}>{review.title}</li>
      ))}
    </ul>
  );
}
```

### ❌ Bad Code — `ReviewSearch.tsx`

```tsx
import { useMemo } from "react";

type Review = {
  id: string;
  title: string;
  authorName: string;
};

function matchesSearch(review: Review, search: string) {
  return review.title.toLowerCase().includes(search.toLowerCase());
}

export function ReviewSearch({
  reviews,
  search,
}: {
  reviews: Review[];
  search: string;
}) {
  // Empty dependencies freeze the list after the first render.
  const visibleReviews = useMemo(
    () => reviews.filter((review) => matchesSearch(review, search)),
    [],
  );
  const resultLabel = useMemo(
    () => "Showing " + visibleReviews.length + " reviews",
    [visibleReviews.length],
  );

  return (
    <>
      <p>{resultLabel}</p>
      <ul>
        {visibleReviews.map((review) => (
          <li key={review.id}>{review.title}</li>
        ))}
      </ul>
    </>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version memoizes a filtered list and lists the inputs that affect that list.
- **Bad Code:** The bad version memoizes the list with an empty dependency array, so the UI can ignore new reviews or search text. It also memoizes a cheap label that does not need caching.

### 📝 Review notes

In review, ask what cost the memoization avoids and whether the dependency array is complete. The bad version is review-worthy because extra `useMemo` calls can make code look optimized while freezing stale data.

### 💡 Takeaway

> **Memoization should have a clear cost to avoid and a complete dependency list.**

---

## 10. Composition over prop flags

> Prefer flexible component slots over growing boolean prop combinations.

**Tags:** `composition` `components` `props` · [View on site](https://good-code-bad-code.pages.dev/tracks/react/composition-over-prop-flags)

### ✅ Good Code — `Panel.tsx`

```tsx
import type { ReactNode } from "react";

type PanelProps = {
  title: string;
  tone?: "neutral" | "danger";
  actions?: ReactNode;
  children: ReactNode;
};

export function Panel({
  title,
  tone = "neutral",
  actions,
  children,
}: PanelProps) {
  // Slots let callers compose the variant-specific UI.
  return (
    <section data-tone={tone}>
      <header>
        <h2>{title}</h2>
        {actions}
      </header>
      <div>{children}</div>
    </section>
  );
}

export function DeleteReviewPanel({ onDelete }: { onDelete: () => void }) {
  return (
    <Panel
      title="Delete review"
      tone="danger"
      actions={<button type="button" onClick={onDelete}>Delete</button>}
    >
      <p>This review and its comments will be removed.</p>
    </Panel>
  );
}
```

### ❌ Bad Code — `Panel.tsx`

```tsx
import type { ReactNode } from "react";

type PanelProps = {
  title: string;
  mode: "details" | "delete" | "approval";
  showFooter: boolean;
  showDeleteButton: boolean;
  onDelete?: () => void;
  children: ReactNode;
};

export function Panel({
  title,
  mode,
  showFooter,
  showDeleteButton,
  onDelete,
  children,
}: PanelProps) {
  // Boolean flags create combinations the component must police.
  return (
    <section data-mode={mode}>
      <h2>{title}</h2>
      <div>{children}</div>
      {showFooter ? <footer>Review actions</footer> : null}
      {showDeleteButton ? (
        <button type="button" onClick={onDelete}>Delete</button>
      ) : null}
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version gives callers a clear slot for actions and content, so each variant composes the panel it needs.
- **Bad Code:** The bad version grows mode and boolean props for every new variation. Those combinations make the component harder to understand and easier to call incorrectly.

### 📝 Review notes

In review, watch for components where every new requirement adds another flag. The bad version is review-worthy because the API invites invalid combinations, while composition lets the caller provide the exact UI for that use case.

### 💡 Takeaway

> **Use children and named slots when a component has multiple content variants.**

---

[← Back to index](README.md)
