# Tailwind CSS — Code Review Handbook

> Review utility composition, responsive variants, states, theme tokens, dark mode, reuse boundaries, accessibility, and class conflicts.

- **Category:** Frontend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/tailwindcss](https://good-code-bad-code.pages.dev/tracks/tailwindcss)
- **Related tracks:** [HTML](01-html.md), [CSS](02-css.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Utility composition readability](#01-utility-composition-readability)** — A utility-first class list still needs structure; readable grouping makes design intent reviewable.
- [ ] **[Responsive breakpoints](#02-responsive-breakpoints)** — Responsive utilities should describe how a small-screen layout grows, not how a large layout is squeezed down.
- [ ] **[State variants: hover, focus, and disabled](#03-state-variants-hover-focus-and-disabled)** — Buttons and links should expose hover, focus-visible, and disabled states with the same care as their default style.
- [ ] **[Spacing and layout consistency](#04-spacing-and-layout-consistency)** — Repeated spacing utilities should reveal a rhythm; random values usually reveal local visual patching.
- [ ] **[Theme tokens and custom values](#05-theme-tokens-and-custom-values)** — A repeated color, spacing, or radius should usually become a token that reviewers can discuss by name.
- [ ] **[Dark mode variants](#06-dark-mode-variants)** — Dark mode should cover background, text, border, and state colors, not only the most visible foreground color.
- [ ] **[Class reuse and component boundaries](#07-class-reuse-and-component-boundaries)** — Repeated Tailwind strings are still duplication; extract them when they represent the same UI concept.
- [ ] **[Arbitrary values with restraint](#08-arbitrary-values-with-restraint)** — Arbitrary values are a precision tool; overuse makes the design system invisible to reviewers.
- [ ] **[Accessibility: contrast and focus](#09-accessibility-contrast-and-focus)** — Utility classes can make accessibility explicit when contrast, focus, and labels are part of the class list.
- [ ] **[Conditional classes and conflicts](#10-conditional-classes-and-conflicts)** — Conditional Tailwind should make variant precedence obvious; conflicting utilities turn state review into guesswork.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Utility composition readability](#01-utility-composition-readability) | `utilities` `readability` `composition` |
| 02 | [Responsive breakpoints](#02-responsive-breakpoints) | `responsive` `breakpoints` `layout` |
| 03 | [State variants: hover, focus, and disabled](#03-state-variants-hover-focus-and-disabled) | `states` `focus` `disabled` |
| 04 | [Spacing and layout consistency](#04-spacing-and-layout-consistency) | `spacing` `layout` `scale` |
| 05 | [Theme tokens and custom values](#05-theme-tokens-and-custom-values) | `theme` `tokens` `design-system` |
| 06 | [Dark mode variants](#06-dark-mode-variants) | `dark-mode` `variants` `color` |
| 07 | [Class reuse and component boundaries](#07-class-reuse-and-component-boundaries) | `reuse` `components` `duplication` |
| 08 | [Arbitrary values with restraint](#08-arbitrary-values-with-restraint) | `arbitrary-values` `scale` `maintainability` |
| 09 | [Accessibility: contrast and focus](#09-accessibility-contrast-and-focus) | `accessibility` `contrast` `focus` |
| 10 | [Conditional classes and conflicts](#10-conditional-classes-and-conflicts) | `conditional-classes` `conflicts` `state` |

---

## 01. Utility composition readability

> Group Tailwind utilities by purpose so reviewers can understand layout, color, state, and motion without decoding one long string.

**Tags:** `utilities` `readability` `composition` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/utility-composition-readability)

### ✅ Good Code — `PrimaryButton.tsx`

```tsx
type PrimaryButtonProps = {
  children: React.ReactNode;
  disabled?: boolean;
};

const baseClasses =
  // Layout and sizing live together.
  "inline-flex h-10 items-center justify-center gap-2 rounded-md px-4 text-sm font-medium";
const toneClasses =
  // Color, shadow, and hover tone live together.
  "bg-sky-600 text-white shadow-sm hover:bg-sky-500";
const stateClasses =
  // Focus and disabled styles live together.
  "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-sky-300 disabled:cursor-not-allowed disabled:opacity-50";

export function PrimaryButton({ children, disabled }: PrimaryButtonProps) {
  return (
    <button
      type="button"
      disabled={disabled}
      className={[baseClasses, toneClasses, stateClasses].join(" ")}
    >
      {children}
    </button>
  );
}
```

### ❌ Bad Code — `PrimaryButton.tsx`

```tsx
export function PrimaryButton({ children, disabled }) {
  // One long className mixes layout, color, state, and a duplicate radius.
  return (
    <button
      disabled={disabled}
      className="px-4 text-white rounded-md h-10 bg-sky-600 text-sm hover:bg-sky-500 inline-flex shadow-sm items-center disabled:opacity-50 justify-center font-medium focus-visible:ring-2 gap-2 rounded-lg disabled:cursor-not-allowed focus-visible:ring-sky-300"
    >
      {children}
    </button>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version separates base structure, visual tone, and interaction states. A reviewer can scan each concern and spot duplicate or conflicting utilities quickly.
- **Bad Code:** The bad version ships one long class list with mixed concerns and an accidental rounded value conflict. The UI may render, but the intent is harder to review.

### 📝 Review notes

Tailwind does not remove the need for naming. When a class list becomes part of a reusable UI decision, group it by concern or extract a small component-level constant.

### 💡 Takeaway

> **A utility-first class list still needs structure; readable grouping makes design intent reviewable.**

---

## 02. Responsive breakpoints

> Use Tailwind breakpoints as mobile-first overrides instead of locking layouts to a desktop width.

**Tags:** `responsive` `breakpoints` `layout` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/responsive-breakpoints)

### ✅ Good Code — `LessonGrid.tsx`

```tsx
export function LessonGrid({ lessons }: { lessons: Lesson[] }) {
  // Base classes describe mobile layout; sm/lg add columns as space grows.
  return (
    <section className="mx-auto w-full max-w-6xl px-4 py-8 sm:px-6 lg:px-8">
      <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {lessons.map((lesson) => (
          <a key={lesson.slug} href={lesson.href} className="rounded-md border border-slate-200 p-4 hover:border-sky-400">
            {lesson.title}
          </a>
        ))}
      </div>
    </section>
  );
}
```

### ❌ Bad Code — `LessonGrid.tsx`

```tsx
export function LessonGrid({ lessons }) {
  // Fixed desktop sizing can overflow or squeeze smaller screens.
  return (
    <section className="mx-auto w-[960px] px-[32px] py-[40px]">
      <div className="grid grid-cols-3 gap-[18px]">
        {lessons.map((lesson) => (
          <a key={lesson.slug} href={lesson.href} className="rounded-md border p-4">
            {lesson.title}
          </a>
        ))}
      </div>
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version starts with one column and adds columns at `sm` and `lg`, while the container uses flexible width with a maximum.
- **Bad Code:** The bad version fixes a desktop width and column count. On smaller screens, the layout is forced to overflow or shrink in ways the code does not explain.

### 📝 Review notes

Review responsive Tailwind by reading left to right: base utility first, breakpoint override after. If the base style is not a usable mobile layout, the breakpoint stack is probably backwards.

### 💡 Takeaway

> **Responsive utilities should describe how a small-screen layout grows, not how a large layout is squeezed down.**

---

## 03. State variants: hover, focus, and disabled

> Review interactive utilities as a full state set, not just the hover style that looks good with a mouse.

**Tags:** `states` `focus` `disabled` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/state-variants-hover-focus-disabled)

### ✅ Good Code — `SubmitButton.tsx`

```tsx
export function SubmitButton({ isSaving }: { isSaving: boolean }) {
  // The button covers hover, keyboard focus, and disabled states together.
  return (
    <button
      type="submit"
      disabled={isSaving}
      className="inline-flex h-10 items-center rounded-md bg-emerald-600 px-4 text-sm font-medium text-white transition hover:bg-emerald-500 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-emerald-300 disabled:cursor-not-allowed disabled:bg-slate-300 disabled:text-slate-600"
    >
      {isSaving ? "Saving" : "Save"}
    </button>
  );
}
```

### ❌ Bad Code — `SubmitButton.tsx`

```tsx
export function SubmitButton({ isSaving }) {
  // outline-none removes keyboard focus without a replacement.
  return (
    <button
      disabled={isSaving}
      className="rounded-md bg-emerald-600 px-4 py-2 text-white hover:bg-emerald-500 outline-none"
    >
      Save
    </button>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version includes a visible keyboard focus treatment and a disabled style that changes both pointer behavior and visual affordance.
- **Bad Code:** The bad version only styles hover and removes the outline without replacing it. Disabled users see the same call to action even when the button cannot be used.

### 📝 Review notes

When you see `hover:` in a review, also look for `focus-visible:` and `disabled:` if the element is interactive. Missing states are often UX bugs hiding in plain sight.

### 💡 Takeaway

> **Buttons and links should expose hover, focus-visible, and disabled states with the same care as their default style.**

---

## 04. Spacing and layout consistency

> Use a consistent spacing scale and layout rhythm instead of one-off nudges that make future changes fragile.

**Tags:** `spacing` `layout` `scale` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/spacing-layout-consistency)

### ✅ Good Code — `ReviewPanel.tsx`

```tsx
export function ReviewPanel({ title, items }: ReviewPanelProps) {
  // Shared spacing utilities make the panel rhythm easy to scan.
  return (
    <section className="rounded-md border border-slate-200 bg-white p-4">
      <div className="space-y-3">
        <h2 className="text-base font-semibold text-slate-950">{title}</h2>
        <ul className="space-y-2">
          {items.map((item) => (
            <li key={item.id} className="rounded-md bg-slate-50 p-3 text-sm text-slate-700">
              {item.label}
            </li>
          ))}
        </ul>
      </div>
    </section>
  );
}
```

### ❌ Bad Code — `ReviewPanel.tsx`

```tsx
export function ReviewPanel({ title, items }) {
  // Many one-off bracket values usually mean screenshot tuning.
  return (
    <section className="rounded-md border bg-white p-[19px]">
      <div className="mt-[3px]">
        <h2 className="mb-[14px] ml-[2px] text-[17px] font-semibold">{title}</h2>
        <ul>
          {items.map((item) => (
            <li key={item.id} className="mb-[7px] rounded-md bg-slate-50 px-[11px] py-[9px]">
              {item.label}
            </li>
          ))}
        </ul>
      </div>
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version uses container padding and `space-y` utilities to make vertical rhythm explicit.
- **Bad Code:** The bad version uses arbitrary pixel nudges for almost every gap. That usually means the code was tuned by eye for one screenshot, not designed as a reusable layout.

### 📝 Review notes

One arbitrary value can be reasonable. A cluster of arbitrary spacing values in the same component is a signal to ask whether the layout should return to the project spacing scale.

### 💡 Takeaway

> **Repeated spacing utilities should reveal a rhythm; random values usually reveal local visual patching.**

---

## 05. Theme tokens and custom values

> Prefer named theme tokens for shared decisions and reserve custom values for truly local exceptions.

**Tags:** `theme` `tokens` `design-system` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/theme-tokens-and-custom-values)

### ✅ Good Code — `PlanBadge.tsx`

```tsx
export function PlanBadge({ plan }: { plan: "team" | "pro" }) {
  // Named theme utilities keep brand values reviewable in one place.
  return (
    <span className="inline-flex items-center rounded-md bg-brand-600 px-3 py-1 text-xs font-semibold text-white shadow-sm ring-1 ring-brand-500">
      {plan === "team" ? "Team" : "Pro"}
    </span>
  );
}
```

### ❌ Bad Code — `PlanBadge.tsx`

```tsx
export function PlanBadge({ plan }) {
  // Hex and pixel values hide shared design choices in this component.
  return (
    <span className="inline-flex items-center rounded-[5px] bg-[#2563eb] px-[13px] py-[5px] text-[12px] font-semibold text-white shadow-[0_1px_2px_rgba(37,99,235,0.24)] ring-1 ring-[#3b82f6]">
      {plan === "team" ? "Team" : "Pro"}
    </span>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version uses named brand utilities, so repeated design decisions can be changed and reviewed through the theme.
- **Bad Code:** The bad version encodes the design system into one component with hex colors, custom shadows, and pixel values. The same decision will likely be copied differently elsewhere.

### 📝 Review notes

Tailwind custom values are useful, but repeated custom values are design tokens asking for a name. During review, ask whether the value belongs to the component or the theme.

### 💡 Takeaway

> **A repeated color, spacing, or radius should usually become a token that reviewers can discuss by name.**

---

## 06. Dark mode variants

> Keep dark mode styles paired with the base style so both themes are reviewable in the same component.

**Tags:** `dark-mode` `variants` `color` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/dark-mode-variants)

### ✅ Good Code — `CodeReviewCard.tsx`

```tsx
export function CodeReviewCard({ title, summary }: CodeReviewCardProps) {
  // Light and dark values are paired for surface, border, text, and link.
  return (
    <article className="rounded-md border border-slate-200 bg-white p-4 text-slate-800 shadow-sm dark:border-slate-800 dark:bg-slate-950 dark:text-slate-100">
      <h2 className="text-base font-semibold text-slate-950 dark:text-white">{title}</h2>
      <p className="mt-2 text-sm text-slate-600 dark:text-slate-300">{summary}</p>
      <a href="#" className="mt-4 inline-flex text-sm font-medium text-sky-700 hover:text-sky-600 dark:text-sky-300 dark:hover:text-sky-200">
        Review lesson
      </a>
    </article>
  );
}
```

### ❌ Bad Code — `CodeReviewCard.tsx`

```tsx
export function CodeReviewCard({ title, summary }) {
  // Only the heading changes; the card surface and body text stay light.
  return (
    <article className="rounded-md border border-slate-200 bg-white p-4 text-slate-800 shadow-sm">
      <h2 className="text-base font-semibold dark:text-white">{title}</h2>
      <p className="mt-2 text-sm text-slate-600">{summary}</p>
      <a href="#" className="mt-4 inline-flex text-sm font-medium text-sky-700">
        Review lesson
      </a>
    </article>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version pairs light and dark values for the surface, border, text hierarchy, and link state.
- **Bad Code:** The bad version changes only the heading in dark mode. The card can still have a bright surface and low-contrast body text when the rest of the page is dark.

### 📝 Review notes

Dark mode review is not a search for `dark:` alone. Check whether every layer that contributes to contrast has an intentional dark variant.

### 💡 Takeaway

> **Dark mode should cover background, text, border, and state colors, not only the most visible foreground color.**

---

## 07. Class reuse and component boundaries

> Extract reusable class decisions at component boundaries instead of duplicating long class lists across call sites.

**Tags:** `reuse` `components` `duplication` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/class-reuse-component-boundaries)

### ✅ Good Code — `ReviewStatusBadge.tsx`

```tsx
const statusClasses = {
  // One map owns the color variants for every status.
  passed: "bg-emerald-50 text-emerald-700 ring-emerald-200",
  failed: "bg-rose-50 text-rose-700 ring-rose-200",
  pending: "bg-amber-50 text-amber-700 ring-amber-200",
} as const;

export function ReviewStatusBadge({ status }: { status: keyof typeof statusClasses }) {
  // The badge shape lives in one component instead of every caller.
  return (
    <span className={["inline-flex rounded-md px-2 py-1 text-xs font-medium ring-1", statusClasses[status]].join(" ")}>
      {status}
    </span>
  );
}
```

### ❌ Bad Code — `ReviewList.tsx`

```tsx
export function ReviewList({ reviews }) {
  // Each branch repeats the same badge structure and can drift.
  return (
    <ul>
      {reviews.map((review) => (
        <li key={review.id}>
          {review.status === "passed" && <span className="inline-flex rounded-md bg-emerald-50 px-2 py-1 text-xs font-medium text-emerald-700 ring-1 ring-emerald-200">passed</span>}
          {review.status === "failed" && <span className="inline-flex rounded-md bg-rose-50 px-2 py-1 text-xs font-medium text-rose-700 ring-1 ring-rose-200">failed</span>}
          {review.status === "pending" && <span className="inline-flex rounded-md bg-amber-50 px-2 py-1 text-xs font-medium text-amber-700 ring-1 ring-amber-200">pending</span>}
        </li>
      ))}
    </ul>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version gives the badge a component boundary and keeps variant classes in one typed map.
- **Bad Code:** The bad version repeats the same structural utilities across branches. A small visual change now requires editing every branch and hoping the copies stay aligned.

### 📝 Review notes

Extract when duplication represents one UI concept. Do not extract every class list by reflex; extract when review comments would apply to all copies together.

### 💡 Takeaway

> **Repeated Tailwind strings are still duplication; extract them when they represent the same UI concept.**

---

## 08. Arbitrary values with restraint

> Use arbitrary values for specific layout constraints, not as a replacement for the shared design scale.

**Tags:** `arbitrary-values` `scale` `maintainability` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/arbitrary-values-restraint)

### ✅ Good Code — `CommandRow.tsx`

```tsx
export function CommandRow({ label, shortcut }: CommandRowProps) {
  // One arbitrary grid template captures the real layout constraint.
  return (
    <div className="grid grid-cols-[minmax(0,1fr)_auto] items-center gap-3 rounded-md px-3 py-2 text-sm hover:bg-slate-100">
      <span className="truncate text-slate-800">{label}</span>
      <kbd className="rounded border border-slate-200 bg-white px-1.5 py-0.5 text-xs text-slate-500">
        {shortcut}
      </kbd>
    </div>
  );
}
```

### ❌ Bad Code — `CommandRow.tsx`

```tsx
export function CommandRow({ label, shortcut }) {
  // Bracket values everywhere make the component drift from the shared scale.
  return (
    <div className="grid grid-cols-[417px_68px] items-center gap-[11px] rounded-[7px] px-[13px] py-[9px] text-[13.5px] hover:bg-[#f1f5f9]">
      <span className="max-w-[391px] truncate text-[#1f2937]">{label}</span>
      <kbd className="rounded-[3px] border border-[#e2e8f0] bg-[#ffffff] px-[5px] py-[2px] text-[11px] text-[#64748b]">
        {shortcut}
      </kbd>
    </div>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version uses one arbitrary grid template because the layout needs a flexible text column plus an auto shortcut column.
- **Bad Code:** The bad version uses arbitrary values for nearly every decision. That makes the component hard to align with the rest of the UI.

### 📝 Review notes

Ask why each arbitrary value exists. If the answer is "because it looked close", prefer the spacing, color, size, or radius scale.

### 💡 Takeaway

> **Arbitrary values are a precision tool; overuse makes the design system invisible to reviewers.**

---

## 09. Accessibility: contrast and focus

> Review Tailwind styling for visible focus, readable contrast, and accessible labels on icon-only controls.

**Tags:** `accessibility` `contrast` `focus` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/accessibility-contrast-focus)

### ✅ Good Code — `CopyLinkButton.tsx`

```tsx
export function CopyLinkButton() {
  // Contrast, focus ring, and sr-only text make an icon-only button usable.
  return (
    <button
      type="button"
      className="inline-flex h-9 w-9 items-center justify-center rounded-md border border-slate-300 bg-white text-slate-700 hover:bg-slate-50 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-sky-500 focus-visible:ring-offset-2"
    >
      <LinkIcon className="h-4 w-4" aria-hidden="true" />
      <span className="sr-only">Copy lesson link</span>
    </button>
  );
}
```

### ❌ Bad Code — `CopyLinkButton.tsx`

```tsx
export function CopyLinkButton() {
  // No label and outline-none make this icon-only button hard to use.
  return (
    <button className="h-9 w-9 rounded-md bg-slate-200 text-slate-300 outline-none hover:bg-slate-300">
      <LinkIcon className="mx-auto h-4 w-4" />
    </button>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version gives the icon-only button an accessible text label, adequate contrast, and a strong keyboard focus ring.
- **Bad Code:** The bad version removes the outline, uses weak foreground contrast, and leaves the icon as the only name for the control.

### 📝 Review notes

Tailwind can make accessibility easier to review because the focus ring, contrast choices, and `sr-only` label are visible at the call site.

### 💡 Takeaway

> **Utility classes can make accessibility explicit when contrast, focus, and labels are part of the class list.**

---

## 10. Conditional classes and conflicts

> Build conditional class lists so mutually exclusive states cannot accidentally emit conflicting utilities.

**Tags:** `conditional-classes` `conflicts` `state` · [View on site](https://good-code-bad-code.pages.dev/tracks/tailwindcss/conditional-classes-and-conflicts)

### ✅ Good Code — `SegmentedOption.tsx`

```tsx
import clsx from "clsx";

export function SegmentedOption({ selected, disabled, children }: SegmentedOptionProps) {
  // selected and unselected styles are separate paths, so backgrounds do not conflict.
  return (
    <button
      type="button"
      disabled={disabled}
      className={clsx(
        "rounded-md px-3 py-1.5 text-sm font-medium transition focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-sky-400",
        selected ? "bg-sky-600 text-white" : "bg-white text-slate-700 hover:bg-slate-50",
        disabled && "cursor-not-allowed opacity-50 hover:bg-white",
      )}
    >
      {children}
    </button>
  );
}
```

### ❌ Bad Code — `SegmentedOption.tsx`

```tsx
export function SegmentedOption({ selected, disabled, children }) {
  // Multiple conditions can emit conflicting bg, text, and opacity utilities.
  const className = [
    "rounded-md px-3 py-1.5 text-sm font-medium",
    "bg-white text-slate-700 hover:bg-slate-50",
    selected && "bg-sky-600 text-white",
    disabled && "opacity-50 bg-white",
    !disabled && "opacity-100",
  ].filter(Boolean).join(" ");

  return <button disabled={disabled} className={className}>{children}</button>;
}
```

### 🔍 What to review

- **Good Code:** The good version makes selected and unselected styles mutually exclusive, then layers disabled behavior deliberately.
- **Bad Code:** The bad version can emit conflicting background, text, and opacity utilities in the same string. The final style depends on ordering instead of clear state logic.

### 📝 Review notes

When reviewing conditional Tailwind, look for pairs that should not coexist, such as two backgrounds, two opacity values, or a disabled state that still keeps hover styling active.

### 💡 Takeaway

> **Conditional Tailwind should make variant precedence obvious; conflicting utilities turn state review into guesswork.**

---

[← Back to index](README.md)
