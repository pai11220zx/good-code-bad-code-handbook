# CSS — Code Review Handbook

> Review layout, responsive styling, contrast, and maintainable selectors.

- **Category:** Foundations
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/css](https://good-code-bad-code.pages.dev/tracks/css)
- **Related tracks:** [HTML](01-html.md), [Tailwind CSS](27-tailwindcss.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Box model sizing](#01-box-model-sizing)** — Prefer border-box sizing so declared component width includes padding and borders.
- [ ] **[Cascade and specificity](#02-cascade-and-specificity)** — Prefer component classes and modifiers over deep selectors, IDs, and repeated !important fixes.
- [ ] **[Flex layout boundaries](#03-flex-layout-boundaries)** — Prefer local flex gaps over selectors that push margins onto every child.
- [ ] **[Grid layout tracks](#04-grid-layout-tracks)** — Let grid define track rules with minmax and gaps instead of distributing width through floats or nth-child margins.
- [ ] **[Responsive units](#05-responsive-units)** — Use bounded responsive values so scaling cannot make text too small or too large.
- [ ] **[Color contrast states](#06-color-contrast-states)** — Pair state changes with contrast and focus treatment that users can perceive.
- [ ] **[State selectors and focus-visible](#07-state-selectors-and-focus-visible)** — Use :focus-visible for keyboard focus and avoid removing outlines without a visible replacement.
- [ ] **[Logical properties](#08-logical-properties)** — Prefer inline and block logical properties when left/right/top/bottom are not the real intent.
- [ ] **[Custom properties as design tokens](#09-custom-properties-as-design-tokens)** — Name reusable color, spacing, radius, and component accents with custom properties.
- [ ] **[Motion and reduced motion](#10-motion-and-reduced-motion)** — Pair animated UI with a prefers-reduced-motion path that keeps the state change clear.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Box model sizing](#01-box-model-sizing) | `box model` `layout` `sizing` |
| 02 | [Cascade and specificity](#02-cascade-and-specificity) | `cascade` `specificity` `selectors` |
| 03 | [Flex layout boundaries](#03-flex-layout-boundaries) | `layout` `flexbox` `spacing` |
| 04 | [Grid layout tracks](#04-grid-layout-tracks) | `layout` `grid` `responsive` |
| 05 | [Responsive units](#05-responsive-units) | `responsive` `typography` `spacing` |
| 06 | [Color contrast states](#06-color-contrast-states) | `accessibility` `states` `contrast` |
| 07 | [State selectors and focus-visible](#07-state-selectors-and-focus-visible) | `accessibility` `states` `focus` |
| 08 | [Logical properties](#08-logical-properties) | `internationalization` `layout` `maintainability` |
| 09 | [Custom properties as design tokens](#09-custom-properties-as-design-tokens) | `design tokens` `custom properties` `maintainability` |
| 10 | [Motion and reduced motion](#10-motion-and-reduced-motion) | `accessibility` `motion` `preferences` |

---

## 01. Box model sizing

> Make element dimensions predictable when padding and borders are part of the component.

**Tags:** `box model` `layout` `sizing` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/box-model-sizing)

### ✅ Good Code — `pricing-card.css`

```css
*,
*::before,
*::after {
  /* Border-box keeps declared sizes honest when padding and borders grow. */
  box-sizing: border-box;
}

.pricing-card {
  width: min(100%, 24rem);
  padding: 1.5rem;
  border: 1px solid #d8dee8;
}

.pricing-card__media {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}
```

### ❌ Bad Code — `pricing-card.css`

```css
.pricing-card {
  /* Width ignores padding and border, so the real footprint is larger. */
  width: 24rem;
  padding: 24px;
  border: 1px solid #d8dee8;
}

.pricing-card__media {
  width: 24rem;
}
```

### 🔍 What to review

- **Good Code:** The good version makes the declared card width include its padding and border, so the component can shrink inside narrower containers without surprise overflow.
- **Bad Code:** The bad version looks simple, but its rendered width is wider than `24rem` after padding and border are added, and the media element repeats the fixed size instead of adapting to the card.

### 📝 Review notes

In review, treat sizing as a contract. When the code says a component is `24rem` wide, the surrounding layout should not need to mentally add padding and border to know the real footprint.

### 💡 Takeaway

> **Prefer border-box sizing so declared component width includes padding and borders.**

---

## 02. Cascade and specificity

> Keep selectors scoped and low-specificity so styles are easy to override intentionally.

**Tags:** `cascade` `specificity` `selectors` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/cascade-specificity)

### ✅ Good Code — `profile-card.css`

```css
.profile-card {
  /* Low-specificity component classes keep ownership easy to review. */
  color: #172033;
  background: #ffffff;
  border: 1px solid #d8dee8;
}

.profile-card--featured {
  border-color: #2563eb;
}

.profile-card__title {
  color: #0f172a;
}
```

### ❌ Bad Code — `profile-card.css`

```css
#app main .profile-card.featured div h2.title {
  /* This selector wins by force instead of describing component intent. */
  color: #0f172a !important;
}

#app main .profile-card div h2.title {
  color: #475569;
}
```

### 🔍 What to review

- **Good Code:** The good version uses explicit component classes and a modifier, so a reviewer can see which rule owns each state without tracing through the whole page.
- **Bad Code:** The bad version depends on page structure, ID specificity, and `!important`, which makes future overrides harder and ties a reusable card to one DOM shape.

### 📝 Review notes

In review, ask whether the selector communicates ownership or just wins the specificity fight. A selector can be technically correct while still making the next change unnecessarily brittle.

### 💡 Takeaway

> **Prefer component classes and modifiers over deep selectors, IDs, and repeated !important fixes.**

---

## 03. Flex layout boundaries

> Keep flex spacing and alignment local to the component that owns the layout.

**Tags:** `layout` `flexbox` `spacing` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/flex-layout-boundaries)

### ✅ Good Code — `review-toolbar.css`

```css
.review-toolbar {
  /* The container owns spacing; the status item owns its alignment exception. */
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 0.75rem;
}

.review-toolbar__status {
  margin-right: auto;
}
```

### ❌ Bad Code — `review-toolbar.css`

```css
.review-toolbar {
  /* Child selectors make every toolbar item inherit layout rules implicitly. */
  display: flex;
}

.review-toolbar > * {
  margin-left: 12px;
}

.review-toolbar > :first-child {
  margin-left: 0;
}
```

### 🔍 What to review

- **Good Code:** The good version makes spacing a property of the flex container and keeps the one alignment exception attached to the item that needs it.
- **Bad Code:** The bad version renders a row, but it spreads layout behavior across global child selectors that become fragile when children reorder, wrap, or include nested components.

### 📝 Review notes

In review, look for selectors that make every child pay for a layout decision. The bad version is worth calling out even when it looks fine because the next button, conditional badge, or wrapped line inherits spacing rules it did not explicitly ask for.

### 💡 Takeaway

> **Prefer local flex gaps over selectors that push margins onto every child.**

---

## 04. Grid layout tracks

> Use grid tracks to describe responsive columns instead of hand-balancing fixed widths.

**Tags:** `layout` `grid` `responsive` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/grid-layout-tracks)

### ✅ Good Code — `dashboard-grid.css`

```css
.dashboard-grid {
  /* Grid describes the responsive column rule in one place. */
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: 1rem;
}

.dashboard-card {
  min-width: 0;
}
```

### ❌ Bad Code — `dashboard-grid.css`

```css
.dashboard-card {
  /* Floats and nth-child margins encode fragile row math manually. */
  float: left;
  width: 33.333%;
  margin-right: 24px;
}

.dashboard-card:nth-child(3n) {
  margin-right: 0;
}
```

### 🔍 What to review

- **Good Code:** The good version describes the layout rule once: cards should create as many columns as fit, each with a sensible minimum width.
- **Bad Code:** The bad version tries to simulate columns with floats, percentages, and positional exceptions, so adding a gap or changing the column count becomes a fragile rewrite.

### 📝 Review notes

In review, look for layout code that encodes row math manually. Modern CSS can often express the rule directly, which leaves less hidden arithmetic for future reviewers to recalculate.

### 💡 Takeaway

> **Let grid define track rules with minmax and gaps instead of distributing width through floats or nth-child margins.**

---

## 05. Responsive units

> Use responsive ranges that respect readable text and container context.

**Tags:** `responsive` `typography` `spacing` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/responsive-units)

### ✅ Good Code — `lesson-card.css`

```css
.lesson-card {
  /* Bounded values scale with the component without losing readability. */
  container-type: inline-size;
  padding: clamp(1rem, 2cqi, 1.5rem);
  max-width: 42rem;
}

.lesson-card__title {
  font-size: clamp(1.25rem, 1rem + 1vw, 2rem);
  line-height: 1.2;
}

.lesson-card__body {
  font-size: 1rem;
  line-height: 1.6;
}
```

### ❌ Bad Code — `lesson-card.css`

```css
body {
  /* Viewport text size can become unreadable outside this exact screen width. */
  font-size: 1.2vw;
}

.lesson-card {
  padding: 4vw;
}

.lesson-card__title {
  font-size: 4vw;
}
```

### 🔍 What to review

- **Good Code:** The good version uses `clamp()` to set responsive bounds and keeps body text at a readable base size.
- **Bad Code:** The bad version renders responsively, but viewport-scaled body text can become tiny on narrow layouts or oversized on wide screens regardless of the content container.

### 📝 Review notes

In review, ask whether the unit responds to the container, viewport, or text measure that actually constrains the component. The bad version is review-worthy even when it renders because viewport units can make local content readability depend on a global screen dimension.

### 💡 Takeaway

> **Use bounded responsive values so scaling cannot make text too small or too large.**

---

## 06. Color contrast states

> Define interactive states that remain visible and readable.

**Tags:** `accessibility` `states` `contrast` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/color-contrast-states)

### ✅ Good Code — `primary-action.css`

```css
.primary-action {
  /* Base and state colors keep readable contrast. */
  color: #ffffff;
  background: #174ea6;
  border: 2px solid #174ea6;
}

.primary-action:hover {
  background: #0b3478;
}

.primary-action:focus-visible {
  outline: 3px solid #f9ab00;
  outline-offset: 2px;
}
```

### ❌ Bad Code — `primary-action.css`

```css
.primary-action {
  /* Low-contrast state changes are hard to perceive or read. */
  color: #7b8794;
  background: #eef2f7;
  border: 1px solid #d8dee8;
}

.primary-action:hover,
.primary-action:focus {
  color: #9aa4b2;
}
```

### 🔍 What to review

- **Good Code:** The good version specifies hover and keyboard focus states that preserve readable foreground and background contrast while making focus highly visible.
- **Bad Code:** The bad version renders a recognizable button, but the state change is low contrast and color-only, so it can be hard to detect or read.

### 📝 Review notes

In review, treat visual states as behavior, not decoration. The bad version is review-worthy even when it renders because a user may technically see a state change without having enough contrast or non-color signal to understand it.

### 💡 Takeaway

> **Pair state changes with contrast and focus treatment that users can perceive.**

---

## 07. State selectors and focus-visible

> Style hover, active, selected, and keyboard focus states as separate reviewable behavior.

**Tags:** `accessibility` `states` `focus` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/focus-visible-states)

### ✅ Good Code — `tabs.css`

```css
.tab-button {
  /* Hover, focus, and selected state each get a distinct visual signal. */
  color: #334155;
  border-bottom: 2px solid transparent;
}

.tab-button:hover {
  color: #0f172a;
}

.tab-button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 3px;
}

.tab-button[aria-selected="true"] {
  color: #1d4ed8;
  border-bottom-color: currentColor;
}
```

### ❌ Bad Code — `tabs.css`

```css
.tab-button {
  /* Removing outline without a replacement hides keyboard position. */
  outline: none;
}

.tab-button:hover,
.tab-button:focus {
  color: #1d4ed8;
}

.tab-button.active {
  border-bottom: 2px solid #1d4ed8;
}
```

### 🔍 What to review

- **Good Code:** The good version keeps mouse hover, keyboard focus, and selected state distinct, so each user interaction has a visible and meaningful style.
- **Bad Code:** The bad version removes the native outline and blends focus with hover, making keyboard navigation harder to detect and tying selected state to a visual-only class.

### 📝 Review notes

In review, do not accept `outline: none` unless the replacement focus style is obvious in the same change. Focus styling is interaction behavior, not optional polish.

### 💡 Takeaway

> **Use :focus-visible for keyboard focus and avoid removing outlines without a visible replacement.**

---

## 08. Logical properties

> Use flow-relative spacing and borders when styles should adapt to writing direction.

**Tags:** `internationalization` `layout` `maintainability` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/logical-properties)

### ✅ Good Code — `article-callout.css`

```css
.article-callout {
  /* Logical properties express flow-relative spacing and edges. */
  margin-block: 1.5rem;
  padding-block: 1rem;
  padding-inline: 1.25rem;
  border-inline-start: 4px solid #2563eb;
}

.article-callout__meta {
  text-align: start;
}
```

### ❌ Bad Code — `article-callout.css`

```css
.article-callout {
  /* Physical left/right values make writing direction harder to support. */
  margin-top: 24px;
  margin-bottom: 24px;
  padding: 16px 20px 16px 20px;
  border-left: 4px solid #2563eb;
}

.article-callout__meta {
  text-align: left;
}
```

### 🔍 What to review

- **Good Code:** The good version says the accent belongs at the start edge and the spacing belongs to the block or inline axis, so the component can adapt to different writing directions.
- **Bad Code:** The bad version hard-codes left and right even when the intent is start, end, inline, or block. That makes localization and layout reuse more expensive later.

### 📝 Review notes

In review, ask whether a physical direction is truly required. If the design means "start of the text" or "space above and below", logical properties usually describe the intent more directly.

### 💡 Takeaway

> **Prefer inline and block logical properties when left/right/top/bottom are not the real intent.**

---

## 09. Custom properties as design tokens

> Use custom properties to name shared design decisions instead of repeating magic values.

**Tags:** `design tokens` `custom properties` `maintainability` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/custom-properties-tokens)

### ✅ Good Code — `notice.css`

```css
:root {
  /* Named tokens make shared design decisions reviewable. */
  --color-surface: #ffffff;
  --color-border: #d8dee8;
  --space-4: 1rem;
  --radius-sm: 0.375rem;
}

.notice {
  --notice-accent: #2563eb;
  padding: var(--space-4);
  border: 1px solid var(--color-border);
  border-left: 4px solid var(--notice-accent);
  border-radius: var(--radius-sm);
  background: var(--color-surface);
}

.notice--warning {
  --notice-accent: #b45309;
}
```

### ❌ Bad Code — `notice.css`

```css
.notice {
  /* Repeated raw values hide which decisions are shared tokens. */
  padding: 16px;
  border: 1px solid #d8dee8;
  border-left: 4px solid #2563eb;
  border-radius: 6px;
  background: #ffffff;
}

.notice--warning {
  border-left-color: #b45309;
}
```

### 🔍 What to review

- **Good Code:** The good version gives shared design values names and lets the warning variant change only the accent token it owns.
- **Bad Code:** The bad version works visually, but every repeated value becomes a search-and-replace dependency when the design system changes.

### 📝 Review notes

In review, look for values that appear in multiple components or represent product decisions. Magic numbers are harder to discuss than named tokens because the intent is hidden in raw values.

### 💡 Takeaway

> **Name reusable color, spacing, radius, and component accents with custom properties.**

---

## 10. Motion and reduced motion

> Make motion purposeful and respect users who request less animation.

**Tags:** `accessibility` `motion` `preferences` · [View on site](https://good-code-bad-code.pages.dev/tracks/css/reduced-motion)

### ✅ Good Code — `drawer.css`

```css
.drawer {
  /* State-driven motion can be shortened for reduced-motion users. */
  transform: translateX(100%);
  transition: transform 180ms ease;
}

.drawer[data-open="true"] {
  transform: translateX(0);
}

@media (prefers-reduced-motion: reduce) {
  .drawer {
    transition-duration: 1ms;
  }
}
```

### ❌ Bad Code — `drawer.css`

```css
.drawer {
  /* The animation runs for everyone and does not expose a reduced path. */
  animation: slide-in 600ms cubic-bezier(.2, .8, .2, 1) both;
}

@keyframes slide-in {
  from {
    transform: translateX(100%);
  }

  to {
    transform: translateX(0);
  }
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the state change readable while shortening the transition for users who prefer reduced motion.
- **Bad Code:** The bad version forces the same animation on every user and hides the open state inside an animation that cannot easily respond to user preferences.

### 📝 Review notes

In review, ask what the motion communicates and what happens when motion is reduced. Useful animation should clarify state, not become the only way a state change makes sense.

### 💡 Takeaway

> **Pair animated UI with a prefers-reduced-motion path that keeps the state change clear.**

---

[← Back to index](README.md)
