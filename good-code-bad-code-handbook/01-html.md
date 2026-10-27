# HTML — Code Review Handbook

> Review structure, semantics, forms, and accessible document foundations.

- **Category:** Foundations
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/html](https://good-code-bad-code.pages.dev/tracks/html)
- **Related tracks:** [CSS](02-css.md), [Tailwind CSS](27-tailwindcss.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Document language and metadata](#01-document-language-and-metadata)** — Start every page with language, charset, viewport, and a useful title.
- [ ] **[Semantic document structure](#02-semantic-document-structure)** — Prefer meaningful landmarks over anonymous wrappers.
- [ ] **[Heading hierarchy](#03-heading-hierarchy)** — Heading levels should outline the content in order, not act as font-size shortcuts.
- [ ] **[Links and navigation](#04-links-and-navigation)** — If it takes the user somewhere, make it an anchor with a meaningful destination and label.
- [ ] **[Images and alternative text](#05-images-and-alternative-text)** — Alt text should explain the image purpose in context, not repeat the file name.
- [ ] **[Accessible form labels](#06-accessible-form-labels)** — Use a real label so the input keeps its name before, during, and after editing.
- [ ] **[Form help text and errors](#07-form-help-text-and-errors)** — Use aria-describedby and aria-invalid to keep field guidance tied to the input.
- [ ] **[Interactive elements](#08-interactive-elements)** — Reach for a native button when the user is triggering an action.
- [ ] **[Tables for tabular data](#09-tables-for-tabular-data)** — When content is rows and columns of data, prefer table, caption, th, and scope.
- [ ] **[ARIA restraint](#10-aria-restraint)** — Prefer native semantics; ARIA should clarify, not paper over the wrong element.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Document language and metadata](#01-document-language-and-metadata) | `metadata` `accessibility` `document` |
| 02 | [Semantic document structure](#02-semantic-document-structure) | `semantics` `accessibility` `structure` |
| 03 | [Heading hierarchy](#03-heading-hierarchy) | `headings` `semantics` `structure` |
| 04 | [Links and navigation](#04-links-and-navigation) | `links` `navigation` `accessibility` |
| 05 | [Images and alternative text](#05-images-and-alternative-text) | `images` `accessibility` `content` |
| 06 | [Accessible form labels](#06-accessible-form-labels) | `forms` `accessibility` `labels` |
| 07 | [Form help text and errors](#07-form-help-text-and-errors) | `forms` `accessibility` `errors` |
| 08 | [Interactive elements](#08-interactive-elements) | `semantics` `accessibility` `interaction` |
| 09 | [Tables for tabular data](#09-tables-for-tabular-data) | `tables` `semantics` `data` |
| 10 | [ARIA restraint](#10-aria-restraint) | `aria` `accessibility` `semantics` |

---

## 01. Document language and metadata

> Declare the document basics so browsers, search, and assistive technology start with the right context.

**Tags:** `metadata` `accessibility` `document` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/document-language-metadata)

### ✅ Good Code — `document.html`

```html
<!doctype html>
<!-- Document metadata gives browsers and assistive tech the right defaults. -->
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Code review checklist</title>
  </head>
  <body>
    <main>...</main>
  </body>
</html>
```

### ❌ Bad Code — `document.html`

```html
<html>
  <!-- Missing language and viewport leave important defaults implicit. -->
  <head>
    <title>Untitled</title>
  </head>
  <body>
    <div>Code review checklist</div>
  </body>
</html>
```

### 🔍 What to review

- **Good Code:** The good version gives the browser the page language, character encoding, viewport behavior, and a specific document title before rendering the body.
- **Bad Code:** The bad version leaves important document defaults implicit, which can cause wrong pronunciation, weak browser behavior, and unclear page identity.

### 📝 Review notes

When reviewing HTML, start at the top of the document. A page can have clean body markup and still be hard to use if the document shell does not describe language, encoding, viewport, and title clearly.

### 💡 Takeaway

> **Start every page with language, charset, viewport, and a useful title.**

---

## 02. Semantic document structure

> Use landmarks and headings to make the page structure understandable.

**Tags:** `semantics` `accessibility` `structure` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/semantic-document-structure)

### ✅ Good Code — `article.html`

```html
<!-- nav, main, and article expose page regions before CSS runs. -->
<nav aria-label="Primary">
  <a href="/reviews">Review guide</a>
</nav>
<main>
  <article aria-labelledby="article-title">
    <h1 id="article-title">Reviewing pull requests</h1>
    <p>Small reviews are easier to reason about.</p>
  </article>
</main>
```

### ❌ Bad Code — `article.html`

```html
<div>
  <!-- Generic divs hide navigation and content structure from the document outline. -->
  <div class="nav">
    <span>Review guide</span>
  </div>
  <div>
    <b>Reviewing pull requests</b>
    <div>Small reviews are easier to reason about.</div>
  </div>
</div>
```

### 🔍 What to review

- **Good Code:** The good version exposes navigation, main content, article content, and a real heading through native HTML.
- **Bad Code:** The bad version may look acceptable, but it hides navigation and content meaning behind generic containers.

### 📝 Review notes

In review, ask whether another developer or assistive technology can understand the page outline without reading CSS. Semantic HTML is not decoration; it is part of the interface contract.

### 💡 Takeaway

> **Prefer meaningful landmarks over anonymous wrappers.**

---

## 03. Heading hierarchy

> Use headings to describe document structure instead of choosing heading levels for visual size.

**Tags:** `headings` `semantics` `structure` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/heading-hierarchy)

### ✅ Good Code — `settings.html`

```html
<main>
  <!-- Heading levels follow the page outline in order. -->
  <h1>Account settings</h1>
  <section>
    <h2>Profile</h2>
    <h3>Public name</h3>
  </section>
</main>
```

### ❌ Bad Code — `settings.html`

```html
<main>
  <!-- Heading levels are being used for appearance instead of structure. -->
  <h3>Account settings</h3>
  <section>
    <h1>Profile</h1>
    <h4>Public name</h4>
  </section>
</main>
```

### 🔍 What to review

- **Good Code:** The good version creates a predictable outline: page title, section, then subsection.
- **Bad Code:** The bad version jumps between heading levels because the markup is trying to control appearance instead of structure.

### 📝 Review notes

Ask whether a reader can skim only headings and understand the page. If the answer is no, the heading hierarchy is probably carrying style decisions instead of information architecture.

### 💡 Takeaway

> **Heading levels should outline the content in order, not act as font-size shortcuts.**

---

## 04. Links and navigation

> Use real links with clear names for navigation instead of clickable generic elements.

**Tags:** `links` `navigation` `accessibility` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/links-and-navigation)

### ✅ Good Code — `nav.html`

```html
<nav aria-label="Primary">
  <!-- Real anchors preserve browser navigation behavior. -->
  <ul>
    <li><a href="/roadmaps/html">HTML roadmap</a></li>
    <li><a href="/review-examples">Review examples</a></li>
  </ul>
</nav>
```

### ❌ Bad Code — `nav.html`

```html
<div class="nav">
  <!-- Click handlers and vague labels break basic link expectations. -->
  <span onclick="go('/roadmaps/html')">Click here</span>
  <a href="#">More</a>
</div>
```

### 🔍 What to review

- **Good Code:** The good version uses anchors for navigation, gives each destination a clear name, and wraps the group in a named navigation landmark.
- **Bad Code:** The bad version hides navigation behind JavaScript and vague labels, making the links harder to open, copy, search, and understand.

### 📝 Review notes

During review, ask whether every navigation item still works with browser defaults: open in a new tab, copy link, keyboard focus, and readable link text. If those basics break, the markup is fighting the web platform.

### 💡 Takeaway

> **If it takes the user somewhere, make it an anchor with a meaningful destination and label.**

---

## 05. Images and alternative text

> Write alt text for meaningful images and hide decorative images from assistive technology.

**Tags:** `images` `accessibility` `content` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/images-alt-text)

### ✅ Good Code — `article-card.html`

```html
<article>
  <!-- Alt text describes meaning, while decorative images stay silent. -->
  <img src="/charts/review-time.png" alt="Review time dropped from 3 days to 1 day" />
  <img src="/decorations/divider.svg" alt="" />
  <h2>Faster pull request reviews</h2>
</article>
```

### ❌ Bad Code — `article-card.html`

```html
<article>
  <!-- File names and decorative descriptions add noise for screen readers. -->
  <img src="/charts/review-time.png" alt="review-time.png" />
  <img src="/decorations/divider.svg" alt="blue squiggle divider" />
  <h2>Faster pull request reviews</h2>
</article>
```

### 🔍 What to review

- **Good Code:** The good version describes the meaningful chart by its message and gives the decorative divider an empty alt attribute.
- **Bad Code:** The bad version exposes file names and decorative noise, so assistive technology announces details that do not help the reader.

### 📝 Review notes

Review the image purpose, not only whether an `alt` attribute exists. Useful alt text changes with context; the same image may need a different description depending on what the surrounding content already says.

### 💡 Takeaway

> **Alt text should explain the image purpose in context, not repeat the file name.**

---

## 06. Accessible form labels

> Connect form controls to durable labels instead of relying on placeholder hints.

**Tags:** `forms` `accessibility` `labels` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/accessible-form-labels)

### ✅ Good Code — `signup-form.html`

```html
<form>
  <!-- The visible label stays connected to the input by for and id. -->
  <label for="email">Email address</label>
  <input id="email" name="email" type="email" autocomplete="email" />
</form>
```

### ❌ Bad Code — `signup-form.html`

```html
<form>
  <!-- Placeholder text is not a durable label for the field. -->
  <input name="email" type="email" placeholder="Email address" />
</form>
```

### 🔍 What to review

- **Good Code:** The good version gives the email field a persistent accessible name by pairing the `label` and `input` with matching identifiers.
- **Bad Code:** The bad version depends on placeholder text, which disappears as soon as someone types and does not provide the same durable relationship as a label.

### 📝 Review notes

In review, check whether every control can be identified without relying on visual position or temporary hint text. Placeholder-only fields can render neatly, but they make the form harder to review, translate, autofill, and use with assistive technology.

### 💡 Takeaway

> **Use a real label so the input keeps its name before, during, and after editing.**

---

## 07. Form help text and errors

> Connect help and error text to the field so validation feedback is announced with the control.

**Tags:** `forms` `accessibility` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/form-help-errors)

### ✅ Good Code — `password-field.html`

```html
<label for="password">Password</label>
<!-- aria-describedby connects both guidance and error text to the field. -->
<input
  id="password"
  name="password"
  type="password"
  aria-describedby="password-help password-error"
  aria-invalid="true"
/>
<p id="password-help">Use at least 12 characters.</p>
<p id="password-error">Password is too short.</p>
```

### ❌ Bad Code — `password-field.html`

```html
<label>Password</label>
<!-- Nearby text is visual only unless it is programmatically connected. -->
<input name="password" type="password" />
<p class="help">Use at least 12 characters.</p>
<p class="error">Too short.</p>
```

### 🔍 What to review

- **Good Code:** The good version ties guidance and the current error to the input, and marks the field invalid when validation fails.
- **Bad Code:** The bad version places text near the field visually, but the input has no durable relationship to the help or error message.

### 📝 Review notes

In form review, proximity is not enough. Check whether the field, help text, and error state are connected in the accessibility tree, not just whether they appear close together on screen.

### 💡 Takeaway

> **Use aria-describedby and aria-invalid to keep field guidance tied to the input.**

---

## 08. Interactive elements

> Use native controls for actions so interaction behavior comes with the element.

**Tags:** `semantics` `accessibility` `interaction` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/interactive-elements)

### ✅ Good Code — `toolbar.html`

```html
<button type="button" aria-pressed="false">
  <!-- Native buttons bring keyboard and focus behavior with them. -->
  Save draft
</button>
```

### ❌ Bad Code — `toolbar.html`

```html
<div class="button" onclick="saveDraft()">
  <!-- A clickable div must recreate button behavior by hand. -->
  Save draft
</div>
```

### 🔍 What to review

- **Good Code:** The good version uses a native `button`, which already participates in keyboard interaction, focus order, and accessibility APIs.
- **Bad Code:** The bad version may be styled to look identical, but the generic `div` has to recreate expected button behavior by hand.

### 📝 Review notes

In review, ask whether the element's semantics match the user's action. A clickable `div` is review-worthy even when the click handler works because it quietly shifts keyboard, focus, disabled, and pressed-state responsibilities into custom code.

### 💡 Takeaway

> **Reach for a native button when the user is triggering an action.**

---

## 09. Tables for tabular data

> Use table markup for data grids so headers and cells keep their relationships.

**Tags:** `tables` `semantics` `data` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/tables-tabular-data)

### ✅ Good Code — `metrics-table.html`

```html
<table>
  <!-- Caption and scopes preserve header-cell relationships. -->
  <caption>Pull request review times</caption>
  <thead>
    <tr>
      <th scope="col">Team</th>
      <th scope="col">Median time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Frontend</th>
      <td>1.2 days</td>
    </tr>
  </tbody>
</table>
```

### ❌ Bad Code — `metrics-table.html`

```html
<div class="table">
  <!-- CSS can make divs look tabular but cannot create table semantics. -->
  <div class="row header">
    <div>Team</div>
    <div>Median time</div>
  </div>
  <div class="row">
    <div>Frontend</div>
    <div>1.2 days</div>
  </div>
</div>
```

### 🔍 What to review

- **Good Code:** The good version preserves the relationship between headers and values, and gives the table a caption that explains the data.
- **Bad Code:** The bad version may look like a table, but the data relationships only exist in CSS and visual position.

### 📝 Review notes

Ask whether users can understand a cell without seeing the layout. If the answer depends on visual columns alone, real table semantics are probably needed.

### 💡 Takeaway

> **When content is rows and columns of data, prefer table, caption, th, and scope.**

---

## 10. ARIA restraint

> Use native HTML first and add ARIA only when it describes behavior that HTML cannot express.

**Tags:** `aria` `accessibility` `semantics` · [View on site](https://good-code-bad-code.pages.dev/tracks/html/aria-restraint)

### ✅ Good Code — `dialog-actions.html`

```html
<button type="button">
  <!-- Native buttons provide the role before ARIA adds only the missing label. -->
  Save changes
</button>
<button type="button" aria-label="Close dialog">
  x
</button>
```

### ❌ Bad Code — `dialog-actions.html`

```html
<div role="button" aria-checked="true" aria-label="Submit">
  <!-- ARIA roles and states conflict with the element behavior here. -->
  Save changes
</div>
<span role="img" aria-label="Close dialog">x</span>
```

### 🔍 What to review

- **Good Code:** The good version starts with native controls and uses ARIA only where the visible icon needs an accessible name that screen readers can announce.
- **Bad Code:** The bad version adds roles and states that do not match the elements or behavior, which can make the interface more confusing than plain HTML.

### 📝 Review notes

ARIA is powerful, but it is not a shortcut around semantics. In review, first ask whether a native element already provides the role, keyboard behavior, and state you need.

### 💡 Takeaway

> **Prefer native semantics; ARIA should clarify, not paper over the wrong element.**

---

[← Back to index](README.md)
