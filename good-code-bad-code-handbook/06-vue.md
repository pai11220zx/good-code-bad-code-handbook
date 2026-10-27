# Vue — Code Review Handbook

> Review single-file components, props, emits, reactivity, composables, stores, and slots.

- **Category:** Frontend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/vue](https://good-code-bad-code.pages.dev/tracks/vue)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [React](05-react.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Single-file component boundaries](#01-single-file-component-boundaries)** — A Vue SFC should show what it receives, what it renders, and which events it can send back.
- [ ] **[Props and emits contracts](#02-props-and-emits-contracts)** — Props describe input from the parent; emits describe output back to the parent.
- [ ] **[Computed values versus watchers](#03-computed-values-versus-watchers)** — If a value can be derived from reactive inputs, prefer computed over a watcher that copies state.
- [ ] **[List rendering keys](#04-list-rendering-keys)** — A v-for key should identify the data item, not its current position in the array.
- [ ] **[Form v-model boundaries](#05-form-v-model-boundaries)** — A form component should own draft input state and send a committed value back through an event.
- [ ] **[Composables and state ownership](#06-composables-and-state-ownership)** — A composable should return the state and actions one caller needs, without sharing mutable module state by accident.
- [ ] **[Async state, loading, and errors](#07-async-state-loading-and-errors)** — Async Vue components need visible loading and error paths, not only a happy-path ref assignment.
- [ ] **[Pinia store actions](#08-pinia-store-actions)** — A Pinia store should expose state, derived getters, and named actions instead of letting components mutate shared state from any file.
- [ ] **[Provide and inject boundaries](#09-provide-and-inject-boundaries)** — Injected values should have a small contract, a typed key, and a provider close to the feature that owns the context.
- [ ] **[Slots for component composition](#10-slots-for-component-composition)** — Slots let the parent own variable content while the child owns shared structure and accessibility.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Single-file component boundaries](#01-single-file-component-boundaries) | `sfc` `components` `boundaries` |
| 02 | [Props and emits contracts](#02-props-and-emits-contracts) | `props` `emits` `contracts` |
| 03 | [Computed values versus watchers](#03-computed-values-versus-watchers) | `computed` `watch` `reactivity` |
| 04 | [List rendering keys](#04-list-rendering-keys) | `v-for` `keys` `lists` |
| 05 | [Form v-model boundaries](#05-form-v-model-boundaries) | `forms` `v-model` `state` |
| 06 | [Composables and state ownership](#06-composables-and-state-ownership) | `composables` `state` `reuse` |
| 07 | [Async state, loading, and errors](#07-async-state-loading-and-errors) | `async` `loading` `errors` |
| 08 | [Pinia store actions](#08-pinia-store-actions) | `pinia` `stores` `actions` |
| 09 | [Provide and inject boundaries](#09-provide-and-inject-boundaries) | `provide` `inject` `context` |
| 10 | [Slots for component composition](#10-slots-for-component-composition) | `slots` `composition` `components` |

---

## 01. Single-file component boundaries

> Keep Vue single-file components focused on one UI responsibility with inputs visible at the component boundary.

**Tags:** `sfc` `components` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/single-file-component-boundaries)

### ✅ Good Code — `ReviewCard.vue`

```vue
<script setup lang="ts">
type ReviewCardProps = {
  title: string;
  reviewerName: string;
  status: "approved" | "changes-requested";
};

// Props keep display data at the component boundary.
defineProps<ReviewCardProps>();
</script>

<template>
  <article class="review-card">
    <h2>{{ title }}</h2>
    <p>{{ reviewerName }} marked this review as {{ status }}.</p>
  </article>
</template>
```

### ❌ Bad Code — `ReviewDashboard.vue`

```vue
<script setup>
import { useRoute } from "vue-router";
import { useSession } from "@/stores/session";

// This SFC owns routing, session state, fetching, and display at once.
const route = useRoute();
const session = useSession();
const response = await fetch("/api/reviews/" + route.params.id);
const review = await response.json();
</script>

<template>
  <section>
    <h2>{{ session.project.name }}: {{ review.title }}</h2>
    <p>{{ review.reviewer.name }} marked this review as {{ review.status }}.</p>
  </section>
</template>
```

### 🔍 What to review

- **Good Code:** The good version makes the SFC boundary visible: the component receives the exact display fields it needs and renders one card.
- **Bad Code:** The bad version mixes routing, session state, fetching, and display. A reviewer has to understand several app concerns before knowing what the component is responsible for.

### 📝 Review notes

In Vue reviews, start by asking what the SFC owns. If a component reads route state, store state, remote data, and markup in one file, split the data-loading boundary from the presentational component.

### 💡 Takeaway

> **A Vue SFC should show what it receives, what it renders, and which events it can send back.**

---

## 02. Props and emits contracts

> Declare props and emitted events so parent and child components agree on their data contract.

**Tags:** `props` `emits` `contracts` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/props-emits-contracts)

### ✅ Good Code — `ReviewActions.vue`

```vue
<script setup lang="ts">
const props = defineProps<{
  reviewId: string;
  canApprove: boolean;
}>();

const emit = defineEmits<{
  approve: [reviewId: string];
}>();

// The child asks the parent to change state through a named event.
function approveReview() {
  emit("approve", props.reviewId);
}
</script>

<template>
  <button :disabled="!canApprove" @click="approveReview">
    Approve
  </button>
</template>
```

### ❌ Bad Code — `ReviewActions.vue`

```vue
<script setup>
const props = defineProps(["review"]);

// Mutating a prop hides the parent-child data flow.
function approveReview() {
  props.review.status = "approved";
}
</script>

<template>
  <button @click="approveReview">
    Approve
  </button>
</template>
```

### 🔍 What to review

- **Good Code:** The good version names the input fields and the event payload. The parent can see exactly which action the child requests.
- **Bad Code:** The bad version mutates a prop object directly. That makes ownership unclear and can leave the parent, store, or cache out of sync.

### 📝 Review notes

In Vue, props should be read as inputs. When the child needs a change, review for a named emit or a store action so the write happens at the owner boundary.

### 💡 Takeaway

> **Props describe input from the parent; emits describe output back to the parent.**

---

## 03. Computed values versus watchers

> Use computed values for derived display data and reserve watchers for side effects.

**Tags:** `computed` `watch` `reactivity` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/computed-vs-watch)

### ✅ Good Code — `ReviewList.vue`

```vue
<script setup lang="ts">
import { computed, ref } from "vue";

const query = ref("");
const reviews = ref([{ id: "r1", title: "Auth review" }]);

// Computed output stays derived from the current inputs.
const visibleReviews = computed(() =>
  reviews.value.filter((review) =>
    review.title.toLowerCase().includes(query.value.toLowerCase()),
  ),
);
</script>

<template>
  <input v-model="query" aria-label="Filter reviews">
  <article v-for="review in visibleReviews" :key="review.id">
    {{ review.title }}
  </article>
</template>
```

### ❌ Bad Code — `ReviewList.vue`

```vue
<script setup>
import { ref, watch } from "vue";

const query = ref("");
const reviews = ref([{ id: "r1", title: "Auth review" }]);
const visibleReviews = ref(reviews.value);

// A watcher copies derived state and can fall out of sync.
watch(query, () => {
  visibleReviews.value = reviews.value.filter((review) =>
    review.title.toLowerCase().includes(query.value.toLowerCase()),
  );
});
</script>
```

### 🔍 What to review

- **Good Code:** The good version keeps filtered reviews as a computed value. The list updates whenever either the query or reviews change.
- **Bad Code:** The bad version copies derived data into a separate ref. A reviewer must check every input path to confirm the copied value is refreshed.

### 📝 Review notes

Use watchers when the code needs an effect such as saving to storage, calling an API, or syncing with a non-Vue library. Use computed when the output is just a view of current reactive inputs.

### 💡 Takeaway

> **If a value can be derived from reactive inputs, prefer computed over a watcher that copies state.**

---

## 04. List rendering keys

> Use stable item identifiers for v-for keys so Vue can preserve component and form state across list changes.

**Tags:** `v-for` `keys` `lists` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/list-rendering-keys)

### ✅ Good Code — `ReviewQueue.vue`

```vue
<script setup lang="ts">
defineProps<{
  reviews: Array<{ id: string; title: string; reviewer: string }>;
}>();

// Each key follows the review record when the list is sorted.
</script>

<template>
  <article v-for="review in reviews" :key="review.id">
    <h2>{{ review.title }}</h2>
    <p>{{ review.reviewer }}</p>
  </article>
</template>
```

### ❌ Bad Code — `ReviewQueue.vue`

```vue
<script setup>
defineProps(["reviews"]);

// Index keys change meaning when rows move.
</script>

<template>
  <article v-for="(review, index) in reviews" :key="index">
    <h2>{{ review.title }}</h2>
    <input v-model="review.note">
  </article>
</template>
```

### 🔍 What to review

- **Good Code:** The good version keys each row by the review id. Vue can keep row state attached to the same review after sorting or inserting.
- **Bad Code:** The bad version keys rows by array index. Moving a row can attach input state or child component state to the wrong review.

### 📝 Review notes

When reviewing `v-for`, compare the key with the data identity. Index keys are only low-risk for static lists that never reorder, insert, delete, or keep per-row state.

### 💡 Takeaway

> **A v-for key should identify the data item, not its current position in the array.**

---

## 05. Form v-model boundaries

> Keep form drafts local and emit submitted values instead of writing through props while the user types.

**Tags:** `forms` `v-model` `state` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/form-v-model-boundaries)

### ✅ Good Code — `ReviewTitleForm.vue`

```vue
<script setup lang="ts">
import { ref } from "vue";

const props = defineProps<{ initialTitle: string }>();
const emit = defineEmits<{ save: [title: string] }>();

// The draft is local until the user submits the form.
const draftTitle = ref(props.initialTitle);
</script>

<template>
  <form @submit.prevent="emit('save', draftTitle.trim())">
    <label>
      Review title
      <input v-model="draftTitle">
    </label>
    <button type="submit">Save</button>
  </form>
</template>
```

### ❌ Bad Code — `ReviewTitleForm.vue`

```vue
<script setup>
const props = defineProps(["review"]);

// Typing writes through the prop before the user saves.
</script>

<template>
  <form>
    <input v-model="props.review.title">
    <button>Save</button>
  </form>
</template>
```

### 🔍 What to review

- **Good Code:** The good version separates draft input from the saved value. The parent receives a single save event with the trimmed title.
- **Bad Code:** The bad version writes to a prop object as the user types. Cancel, validation, and optimistic update behavior become harder to reason about.

### 📝 Review notes

For Vue forms, look for the owner of draft state. If the form edits parent data immediately, reviewers need to check how validation errors, cancel actions, and failed saves restore the previous value.

### 💡 Takeaway

> **A form component should own draft input state and send a committed value back through an event.**

---

## 06. Composables and state ownership

> Use composables to group reusable reactive behavior while keeping each caller's state isolated.

**Tags:** `composables` `state` `reuse` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/composables-state-ownership)

### ✅ Good Code — `useReviewFilters.ts`

```ts
import { computed, ref } from "vue";

export function useReviewFilters(reviews: Ref<Review[]>) {
  // Each caller receives its own filter refs.
  const query = ref("");
  const activeOnly = ref(true);

  const visibleReviews = computed(() =>
    reviews.value.filter((review) => {
      return (!activeOnly.value || review.active) &&
        review.title.includes(query.value);
    }),
  );

  return { query, activeOnly, visibleReviews };
}
```

### ❌ Bad Code — `reviewFilters.ts`

```ts
import { computed, ref } from "vue";

// Module state is shared by every component that imports this file.
export const query = ref("");
export const activeOnly = ref(true);

export function visibleReviews(reviews: Ref<Review[]>) {
  return computed(() =>
    reviews.value.filter((review) => review.title.includes(query.value)),
  );
}
```

### 🔍 What to review

- **Good Code:** The good version creates filter state inside the composable call. Two pages can use the same composable without sharing query text.
- **Bad Code:** The bad version exports mutable refs from module scope. Components that import the module can overwrite each other's filters.

### 📝 Review notes

Composables are functions, not just folders for shared refs. In review, check whether state is created per caller or shared intentionally through a store.

### 💡 Takeaway

> **A composable should return the state and actions one caller needs, without sharing mutable module state by accident.**

---

## 07. Async state, loading, and errors

> Model loading, data, and error state together so async views do not render stale or silent failures.

**Tags:** `async` `loading` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/async-state-loading-errors)

### ✅ Good Code — `ReviewFeed.vue`

```vue
<script setup lang="ts">
import { onMounted, ref } from "vue";

const reviews = ref<Review[]>([]);
const isLoading = ref(false);
const errorMessage = ref("");

// Loading and error refs describe every request state.
async function loadReviews() {
  isLoading.value = true;
  errorMessage.value = "";

  try {
    const response = await fetch("/api/reviews");
    if (!response.ok) throw new Error("Reviews failed to load");
    reviews.value = await response.json();
  } catch (error) {
    errorMessage.value = "Reviews are unavailable right now.";
  } finally {
    isLoading.value = false;
  }
}

onMounted(loadReviews);
</script>

<template>
  <p v-if="isLoading">Loading reviews...</p>
  <p v-else-if="errorMessage">{{ errorMessage }}</p>
  <ReviewList v-else :reviews="reviews" />
</template>
```

### ❌ Bad Code — `ReviewFeed.vue`

```vue
<script setup>
import { onMounted, ref } from "vue";

const reviews = ref([]);

// Failed requests disappear into the console.
onMounted(async () => {
  reviews.value = await fetch("/api/reviews").then((response) => response.json());
});
</script>

<template>
  <ReviewList :reviews="reviews" />
</template>
```

### 🔍 What to review

- **Good Code:** The good version models loading, success, and error paths in the same component state. The template shows a user-facing result for each path.
- **Bad Code:** The bad version assigns data only on success. Failed requests can leave the page empty with no message or retry path.

### 📝 Review notes

Review async Vue code by following every request path: before the response, after success, after failure, and after a retry. Silent failures are production bugs, not just missing UI polish.

### 💡 Takeaway

> **Async Vue components need visible loading and error paths, not only a happy-path ref assignment.**

---

## 08. Pinia store actions

> Keep shared Vue state behind Pinia stores with actions that name writes and async transitions.

**Tags:** `pinia` `stores` `actions` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/pinia-store-actions)

### ✅ Good Code — `stores/reviews.ts`

```ts
import { defineStore } from "pinia";

export const useReviewsStore = defineStore("reviews", {
  state: () => ({
    reviews: [] as Review[],
    isLoading: false,
  }),
  getters: {
    pendingReviews: (state) =>
      state.reviews.filter((review) => review.status === "pending"),
  },
  actions: {
    // Actions name the write path and own the async transition.
    async loadReviews() {
      this.isLoading = true;
      try {
        this.reviews = await fetch("/api/reviews").then((response) =>
          response.json(),
        );
      } finally {
        this.isLoading = false;
      }
    },
  },
});
```

### ❌ Bad Code — `reviewState.ts`

```ts
import { reactive } from "vue";

// Any importer can rewrite shared state without a named action.
export const reviewState = reactive({
  reviews: [] as Review[],
  isLoading: false,
});

export async function loadReviews() {
  reviewState.reviews = await fetch("/api/reviews").then((response) =>
    response.json(),
  );
}
```

### 🔍 What to review

- **Good Code:** The good version gives shared state a store name, derived getter, and action. Components can call `loadReviews` without knowing every field that changes.
- **Bad Code:** The bad version exports a mutable object and a free function. Any importer can write to state directly, bypassing one reviewable update path.

### 📝 Review notes

Pinia stores make shared state easier to audit when writes go through actions. In review, look for state changes that happen outside the store or duplicate fetch logic in components.

### 💡 Takeaway

> **A Pinia store should expose state, derived getters, and named actions instead of letting components mutate shared state from any file.**

---

## 09. Provide and inject boundaries

> Use provide and inject for narrow context values with typed keys, not as a hidden app-wide dependency channel.

**Tags:** `provide` `inject` `context` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/provide-inject-boundaries)

### ✅ Good Code — `reviewThemeContext.ts`

```ts
import type { InjectionKey, Ref } from "vue";
import { inject, provide, readonly, ref } from "vue";

type ReviewTheme = "light" | "dark";
const reviewThemeKey: InjectionKey<Readonly<Ref<ReviewTheme>>> =
  Symbol("review-theme");

export function provideReviewTheme() {
  // Consumers can read the theme but cannot rewrite it directly.
  const theme = ref<ReviewTheme>("light");
  provide(reviewThemeKey, readonly(theme));
  return { theme };
}

export function useReviewTheme() {
  const theme = inject(reviewThemeKey);
  if (!theme) throw new Error("Review theme provider is missing");
  return theme;
}
```

### ❌ Bad Code — `appContext.ts`

```ts
import { inject, provide, reactive } from "vue";

const appContextKey = Symbol("app");

// One mutable object becomes a hidden dependency for many components.
export function provideAppContext() {
  const app = reactive({ user: null, reviews: [], settings: {}, theme: "light" });
  provide(appContextKey, app);
}

export function useAppContext() {
  return inject(appContextKey) as any;
}
```

### 🔍 What to review

- **Good Code:** The good version provides a narrow, typed context value and keeps mutation at the provider boundary.
- **Bad Code:** The bad version injects a broad mutable object. Components can become coupled to unrelated user, review, settings, and theme state.

### 📝 Review notes

Use provide/inject when props would pass the same context through many layers. Keep the injected contract small enough that a consumer can be tested without rebuilding the whole app.

### 💡 Takeaway

> **Injected values should have a small contract, a typed key, and a provider close to the feature that owns the context.**

---

## 10. Slots for component composition

> Use slots when a reusable Vue shell needs caller-owned content, actions, or layout regions.

**Tags:** `slots` `composition` `components` · [View on site](https://good-code-bad-code.pages.dev/tracks/vue/slots-component-composition)

### ✅ Good Code — `PanelShell.vue`

```vue
<script setup lang="ts">
defineProps<{ labelledBy: string }>();

// Slots let callers supply content without adding mode flags.
</script>

<template>
  <section class="panel" :aria-labelledby="labelledBy">
    <header>
      <slot name="title" />
    </header>
    <div class="panel__body">
      <slot />
    </div>
    <footer>
      <slot name="actions" />
    </footer>
  </section>
</template>
```

### ❌ Bad Code — `PanelShell.vue`

```vue
<script setup>
defineProps(["title", "body", "showApprove", "showArchive", "showCancel"]);

// Boolean flags force the shell to know every caller action.
</script>

<template>
  <section class="panel">
    <h2>{{ title }}</h2>
    <p>{{ body }}</p>
    <button v-if="showApprove">Approve</button>
    <button v-if="showArchive">Archive</button>
    <button v-if="showCancel">Cancel</button>
  </section>
</template>
```

### 🔍 What to review

- **Good Code:** The good version owns the panel structure and lets the caller provide title, body, and actions through named slots.
- **Bad Code:** The bad version adds one prop for every possible action. Each new caller can force the shell component to learn another special case.

### 📝 Review notes

When a component keeps gaining boolean props for visual variants or actions, review whether slots would keep the shell smaller. The shell should own structure; the caller should own feature-specific content.

### 💡 Takeaway

> **Slots let the parent own variable content while the child owns shared structure and accessibility.**

---

[← Back to index](README.md)
