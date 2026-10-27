# Next.js — Code Review Handbook

> Review App Router structure, server boundaries, data fetching, and route APIs.

- **Category:** Frontend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/nextjs](https://good-code-bad-code.pages.dev/tracks/nextjs)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [React](05-react.md), [Vue](06-vue.md), [Node.js](08-nodejs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[App Router file conventions](#01-app-router-file-conventions)** — Routes should be expressed with folders and special files, not invented naming schemes.
- [ ] **[Layout and page boundaries](#02-layout-and-page-boundaries)** — Layouts should hold persistent UI while pages own the leaf route content.
- [ ] **[Server vs Client Components](#03-server-vs-client-components)** — Do not move a whole route to the client just because one child needs state.
- [ ] **[Small client component islands](#04-small-client-component-islands)** — A tiny client island keeps static UI and server data out of the browser bundle.
- [ ] **[Server-side data fetching](#05-server-side-data-fetching)** — Server Components can fetch data close to the source without exposing secrets to the client.
- [ ] **[Parallel data fetching](#06-parallel-data-fetching)** — Independent server requests should be initiated together, then awaited together.
- [ ] **[Loading and error boundaries](#07-loading-and-error-boundaries)** — Use loading and error files to keep navigation responsive and failures contained.
- [ ] **[Caching and revalidation intent](#08-caching-and-revalidation-intent)** — Cache data with an explicit lifetime or tag when shared content can be reused.
- [ ] **[Metadata per route](#09-metadata-per-route)** — Route metadata should live in metadata exports or generateMetadata.
- [ ] **[Route handlers as API boundaries](#10-route-handlers-as-api-boundaries)** — Route handlers should validate requests and return explicit HTTP responses.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [App Router file conventions](#01-app-router-file-conventions) | `routing` `app-router` `files` |
| 02 | [Layout and page boundaries](#02-layout-and-page-boundaries) | `layouts` `pages` `routing` |
| 03 | [Server vs Client Components](#03-server-vs-client-components) | `server-components` `client-components` `boundaries` |
| 04 | [Small client component islands](#04-small-client-component-islands) | `client-components` `bundle-size` `interactivity` |
| 05 | [Server-side data fetching](#05-server-side-data-fetching) | `data-fetching` `server-components` `not-found` |
| 06 | [Parallel data fetching](#06-parallel-data-fetching) | `data-fetching` `performance` `async` |
| 07 | [Loading and error boundaries](#07-loading-and-error-boundaries) | `loading` `errors` `streaming` |
| 08 | [Caching and revalidation intent](#08-caching-and-revalidation-intent) | `caching` `revalidation` `performance` |
| 09 | [Metadata per route](#09-metadata-per-route) | `metadata` `seo` `routing` |
| 10 | [Route handlers as API boundaries](#10-route-handlers-as-api-boundaries) | `route-handlers` `api` `boundaries` |

---

## 01. App Router file conventions

> Use App Router folders and special files so routes match Next.js conventions.

**Tags:** `routing` `app-router` `files` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/app-router-file-conventions)

### ✅ Good Code — `app/projects/[projectId]/page.tsx`

```tsx
export default async function ProjectPage({
  params,
}: PageProps<"/projects/[projectId]">) {
  // Route props keep dynamic segments typed and aligned with the folder path.
  const { projectId } = await params;

  return <h1>Project {projectId}</h1>;
}
```

### ❌ Bad Code — `app/projects-projectId.tsx`

```tsx
export default function ProjectsProjectIdPage() {
  // Parsing the URL by hand bypasses App Router conventions and typing.
  const projectId = window.location.pathname.split("/").at(-1);

  return <h1>Project {projectId}</h1>;
}
```

### 🔍 What to review

- **Good Code:** The good version lets the folder path define the route and reads the dynamic segment through the route props that Next.js provides.
- **Bad Code:** The bad version invents a flat filename convention and parses the URL manually, which bypasses routing, typing, prerendering, and static params support.

### 📝 Review notes

In review, check whether route segments, dynamic params, and special files match the App Router model. The bad version is review-worthy because it makes routing look like ordinary React while losing the framework behavior that Next.js gives for free.

### 💡 Takeaway

> **Routes should be expressed with folders and special files, not invented naming schemes.**

---

## 02. Layout and page boundaries

> Keep shared chrome in layouts and route-specific data in pages.

**Tags:** `layouts` `pages` `routing` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/layout-page-boundaries)

### ✅ Good Code — `app/dashboard/layout.tsx`

```tsx
import Link from "next/link";

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // The layout owns shared navigation while child pages supply route content.
  return (
    <section>
      <nav aria-label="Dashboard">
        <Link href="/dashboard">Overview</Link>
        <Link href="/dashboard/reviews">Reviews</Link>
      </nav>
      <main>{children}</main>
    </section>
  );
}
```

### ❌ Bad Code — `app/dashboard/layout.tsx`

```tsx
import { getLatestReviews } from "@/lib/reviews";

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // Fetching leaf-page data here blocks child routes that may not need it.
  const reviews = await getLatestReviews();

  return (
    <section>
      <aside>{reviews.length} reviews need attention</aside>
      <main>{children}</main>
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version uses the layout for persistent navigation and places route-specific content in the child page.
- **Bad Code:** The bad version fetches leaf-page data inside the layout, so every child route can be blocked by data it may not need.

### 📝 Review notes

In review, ask whether a layout dependency is truly shared by all child routes. The bad version is review-worthy because layouts persist across navigation, so putting page-specific work there can make unrelated pages slower and harder to stream.

### 💡 Takeaway

> **Layouts should hold persistent UI while pages own the leaf route content.**

---

## 03. Server vs Client Components

> Use Server Components for data and secrets, and Client Components for interactivity.

**Tags:** `server-components` `client-components` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/server-client-components)

### ✅ Good Code — `app/reviews/[id]/page.tsx`

```tsx
import { VoteButton } from "./vote-button";
import { getReview } from "@/lib/reviews";

export default async function ReviewPage({
  params,
}: PageProps<"/reviews/[id]">) {
  const { id } = await params;
  // Server rendering fetches the review before handing votes to a client island.
  const review = await getReview(id);

  return (
    <article>
      <h1>{review.title}</h1>
      <p>{review.summary}</p>
      <VoteButton initialVotes={review.votes} />
    </article>
  );
}
```

### ❌ Bad Code — `app/reviews/[id]/page.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";

export default function ReviewPage() {
  const [review, setReview] = useState(null);

  // This client boundary moves page data fetching after hydration.
  useEffect(() => {
    fetch("/api/current-review")
      .then((response) => response.json())
      .then(setReview);
  }, []);

  if (!review) {
    return <p>Loading...</p>;
  }

  return <h1>{review.title}</h1>;
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the route as a Server Component, fetches the review on the server, and passes only interactive data to a Client Component.
- **Bad Code:** The bad version marks the whole page as client code, moves fetching after hydration, and gives up the server-rendered data path.

### 📝 Review notes

In review, look for `"use client"` at route or layout boundaries. The bad version is review-worthy because the directive affects the whole client module graph, so one interactive child can accidentally pull much more code into the browser.

### 💡 Takeaway

> **Do not move a whole route to the client just because one child needs state.**

---

## 04. Small client component islands

> Place client-only behavior in the smallest component that needs it.

**Tags:** `client-components` `bundle-size` `interactivity` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/small-client-islands)

### ✅ Good Code — `app/layout.tsx`

```tsx
import { SearchBox } from "./search-box";
import { SiteLogo } from "./site-logo";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // Only SearchBox needs client JavaScript; the root layout stays server-rendered.
  return (
    <html lang="en">
      <body>
        <header>
          <SiteLogo />
          <SearchBox />
        </header>
        {children}
      </body>
    </html>
  );
}
```

### ❌ Bad Code — `app/layout.tsx`

```tsx
"use client";

import { useState } from "react";
import { SiteLogo } from "./site-logo";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const [query, setQuery] = useState("");

  // One input state pulls the whole root layout into the client bundle.
  return (
    <html lang="en">
      <body>
        <header>
          <SiteLogo />
          <input value={query} onChange={(event) => setQuery(event.target.value)} />
        </header>
        {children}
      </body>
    </html>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the root layout server-rendered and imports a small Client Component for the search interaction.
- **Bad Code:** The bad version marks the root layout as a Client Component just to hold input state, making static shell and imported children part of the client graph.

### 📝 Review notes

In review, search for browser-only hooks in large files. The bad version is review-worthy because it solves a small interaction by moving a large boundary, increasing hydration work and bundle size.

### 💡 Takeaway

> **A tiny client island keeps static UI and server data out of the browser bundle.**

---

## 05. Server-side data fetching

> Fetch route data on the server when the page needs it before rendering.

**Tags:** `data-fetching` `server-components` `not-found` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/server-side-data-fetching)

### ✅ Good Code — `app/articles/[slug]/page.tsx`

```tsx
import { notFound } from "next/navigation";
import { getArticleBySlug } from "@/lib/articles";

export default async function ArticlePage({
  params,
}: PageProps<"/articles/[slug]">) {
  const { slug } = await params;
  // Fetch essential page data on the server and use notFound for missing slugs.
  const article = await getArticleBySlug(slug);

  if (!article) {
    notFound();
  }

  return (
    <article>
      <h1>{article.title}</h1>
      <p>{article.excerpt}</p>
    </article>
  );
}
```

### ❌ Bad Code — `app/articles/[slug]/article-client.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";

export function ArticleClient({ slug }: { slug: string }) {
  const [article, setArticle] = useState(null);

  // Hydration-gated fetching delays page data and adds an avoidable API hop.
  useEffect(() => {
    fetch("/api/articles/" + slug)
      .then((response) => response.json())
      .then(setArticle);
  }, [slug]);

  if (!article) {
    return <p>Loading article...</p>;
  }

  return <h1>{article.title}</h1>;
}
```

### 🔍 What to review

- **Good Code:** The good version fetches the article before rendering the page and uses `notFound()` for a missing slug.
- **Bad Code:** The bad version delays essential page data until after client hydration and adds an API hop for data the server route can already read.

### 📝 Review notes

In review, separate data needed for the first meaningful render from data that is truly client-only. The bad version is review-worthy because it creates slower loading, weaker SEO, and duplicate API surface without a real browser requirement.

### 💡 Takeaway

> **Server Components can fetch data close to the source without exposing secrets to the client.**

---

## 06. Parallel data fetching

> Start independent data requests before awaiting them to avoid waterfalls.

**Tags:** `data-fetching` `performance` `async` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/parallel-data-fetching)

### ✅ Good Code — `app/projects/[projectId]/page.tsx`

```tsx
import { getProject, getReviewSummary } from "@/lib/projects";

export default async function ProjectPage({
  params,
}: PageProps<"/projects/[projectId]">) {
  const { projectId } = await params;
  // Start independent requests before awaiting so rendering waits once.
  const projectPromise = getProject(projectId);
  const summaryPromise = getReviewSummary(projectId);
  const [project, summary] = await Promise.all([
    projectPromise,
    summaryPromise,
  ]);

  return (
    <section>
      <h1>{project.name}</h1>
      <p>{summary.openCount} open reviews</p>
    </section>
  );
}
```

### ❌ Bad Code — `app/projects/[projectId]/page.tsx`

```tsx
import { getProject, getReviewSummary } from "@/lib/projects";

export default async function ProjectPage({
  params,
}: PageProps<"/projects/[projectId]">) {
  const { projectId } = await params;
  // Awaiting each independent request in order creates a server-side waterfall.
  const project = await getProject(projectId);
  const summary = await getReviewSummary(projectId);

  return (
    <section>
      <h1>{project.name}</h1>
      <p>{summary.openCount} open reviews</p>
    </section>
  );
}
```

### 🔍 What to review

- **Good Code:** The good version starts both independent requests before waiting, so the route waits for the slower request rather than the sum of both.
- **Bad Code:** The bad version awaits each request in sequence even though the second request does not depend on the first result.

### 📝 Review notes

In review, trace whether an `await` depends on a previous result. The bad version is review-worthy because server rendering makes waterfalls easy to write and hard to notice without looking at request timing.

### 💡 Takeaway

> **Independent server requests should be initiated together, then awaited together.**

---

## 07. Loading and error boundaries

> Give dynamic routes immediate loading UI and route-level recovery.

**Tags:** `loading` `errors` `streaming` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/loading-error-boundaries)

### ✅ Good Code — `app/dashboard/reviews/error.tsx`

```tsx
"use client";

import { useEffect } from "react";

export default function ReviewsError({
  error,
  unstable_retry,
}: {
  error: Error & { digest?: string };
  unstable_retry: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  // A route error boundary can log, show fallback UI, and trigger retry.
  return (
    <section role="alert">
      <h2>Reviews could not load.</h2>
      <button type="button" onClick={() => unstable_retry()}>
        Try again
      </button>
    </section>
  );
}
```

### ❌ Bad Code — `app/dashboard/reviews/page.tsx`

```tsx
import { getReviews } from "@/lib/reviews";

export default async function ReviewsPage() {
  // Catching every error in the page hides failures from the route boundary.
  try {
    const reviews = await getReviews();
    return reviews.map((review) => <p key={review.id}>{review.title}</p>);
  } catch {
    return <p>Something went wrong.</p>;
  }
}
```

### 🔍 What to review

- **Good Code:** The good version uses a route-level `error.tsx` Client Component so unexpected render failures are contained and retryable.
- **Bad Code:** The bad version catches every exception inside the page, hiding failures from the route boundary and leaving navigation without a meaningful loading strategy.

### 📝 Review notes

In review, look for dynamic routes without `loading.tsx` or recoverable error boundaries. The bad version is review-worthy because it turns unexpected failures into silent UI states and gives users no clear recovery path.

### 💡 Takeaway

> **Use loading and error files to keep navigation responsive and failures contained.**

---

## 08. Caching and revalidation intent

> Make caching choices explicit so freshness and performance match the product need.

**Tags:** `caching` `revalidation` `performance` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/caching-revalidation-intent)

### ✅ Good Code — `lib/posts.ts`

```ts
import { cacheLife, cacheTag } from "next/cache";

export async function getPublishedPosts() {
  "use cache";
  cacheLife("hours");
  cacheTag("posts");

  // The cache directive, lifetime, and tag make freshness explicit.
  return db.post.findMany({
    where: { published: true },
    orderBy: { publishedAt: "desc" },
  });
}
```

### ❌ Bad Code — `lib/posts.ts`

```ts
export async function getPublishedPosts() {
  // This fetch has no stated freshness or revalidation policy.
  const response = await fetch("https://cms.example.com/posts");

  return response.json();
}
```

### 🔍 What to review

- **Good Code:** The good version states that published posts can be cached for hours and tagged for targeted revalidation.
- **Bad Code:** The bad version relies on an unstated assumption that framework fetches are cached, which is not true by default in this Next.js version.

### 📝 Review notes

In review, ask whether the data should be fresh per request, cached for a period, or revalidated by mutation. The bad version is review-worthy because hidden caching assumptions cause either slow pages or stale content depending on how the app evolves.

### 💡 Takeaway

> **Cache data with an explicit lifetime or tag when shared content can be reused.**

---

## 09. Metadata per route

> Use the Metadata API in Server Components instead of mutating the document head manually.

**Tags:** `metadata` `seo` `routing` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/metadata-per-route)

### ✅ Good Code — `app/articles/[slug]/page.tsx`

```tsx
import type { Metadata } from "next";
import { getArticleBySlug } from "@/lib/articles";

export async function generateMetadata({
  params,
}: PageProps<"/articles/[slug]">): Promise<Metadata> {
  const { slug } = await params;
  const article = await getArticleBySlug(slug);

  // generateMetadata makes head data part of the server route contract.
  return {
    title: article?.title ?? "Article not found",
    description: article?.excerpt ?? "This article is not available.",
  };
}

export default async function ArticlePage({
  params,
}: PageProps<"/articles/[slug]">) {
  const { slug } = await params;
  const article = await getArticleBySlug(slug);

  return <h1>{article?.title}</h1>;
}
```

### ❌ Bad Code — `app/articles/[slug]/article-title.tsx`

```tsx
"use client";

import { useEffect } from "react";

export function ArticleTitle({ title }: { title: string }) {
  // Updating document.title after hydration is too late for reliable previews.
  useEffect(() => {
    document.title = title;
  }, [title]);

  return <h1>{title}</h1>;
}
```

### 🔍 What to review

- **Good Code:** The good version lets Next.js generate route metadata from server-side article data.
- **Bad Code:** The bad version updates `document.title` after hydration, so crawlers and social previews can miss the route-specific metadata.

### 📝 Review notes

In review, check whether metadata is declared through `metadata` or `generateMetadata`. The bad version is review-worthy because head state is treated as a client effect instead of part of the route contract.

### 💡 Takeaway

> **Route metadata should live in metadata exports or generateMetadata.**

---

## 10. Route handlers as API boundaries

> Use route handlers for HTTP APIs and keep them separate from page UI.

**Tags:** `route-handlers` `api` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/nextjs/route-handlers-api-boundaries)

### ✅ Good Code — `app/api/reviews/[id]/route.ts`

```ts
import { NextResponse } from "next/server";
import { getReview } from "@/lib/reviews";

export async function GET(
  _request: Request,
  context: RouteContext<"/api/reviews/[id]">,
) {
  const { id } = await context.params;
  const review = await getReview(id);

  // The route handler validates lookup results and returns explicit HTTP status.
  if (!review) {
    return NextResponse.json(
      { error: "Review not found." },
      { status: 404 },
    );
  }

  return NextResponse.json({ review });
}
```

### ❌ Bad Code — `app/reviews/[id]/page.tsx`

```tsx
import { getReview } from "@/lib/reviews";

// Mixing GET and page UI in one file blurs the App Router boundary.
export async function GET() {
  const review = await getReview("current");
  return Response.json(review);
}

export default async function ReviewPage() {
  const review = await getReview("current");

  return <h1>{review.title}</h1>;
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the HTTP API in a `route.ts` file and returns explicit JSON with a 404 when the record is missing.
- **Bad Code:** The bad version mixes a route handler and page UI at the same segment, which conflicts with App Router route resolution and blurs the API boundary.

### 📝 Review notes

In review, verify whether the code is serving UI or HTTP data. The bad version is review-worthy because pages and route handlers are different routing primitives and should not be colocated at the same route segment.

### 💡 Takeaway

> **Route handlers should validate requests and return explicit HTTP responses.**

---

[← Back to index](README.md)
