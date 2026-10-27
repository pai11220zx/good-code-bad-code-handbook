# Express — Code Review Handbook

> Review app factories, routers, middleware order, validation, auth, and error handling.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/express](https://good-code-bad-code.pages.dev/tracks/express)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [React](05-react.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[App factory vs server startup](#01-app-factory-vs-server-startup)** — An Express app should be buildable for tests without opening a port.
- [ ] **[Router boundaries](#02-router-boundaries)** — Routers should own one resource area and receive services instead of reaching into globals.
- [ ] **[Middleware order](#03-middleware-order)** — Express middleware order is behavior, not decoration.
- [ ] **[Request validation](#04-request-validation)** — Route handlers should pass validated values to services, not raw request bodies.
- [ ] **[Async route error forwarding](#05-async-route-error-forwarding)** — Async routes should either be awaited by Express or explicitly forwarded to next.
- [ ] **[Central error handler](#06-central-error-handler)** — A central error handler keeps status codes, logging, and response bodies consistent.
- [ ] **[Response shape consistency](#07-response-shape-consistency)** — Clients should not need route-specific parsing rules for common API responses.
- [ ] **[Auth middleware boundaries](#08-auth-middleware-boundaries)** — Routes should consume trusted auth context instead of reparsing credentials.
- [ ] **[Rate limiting and trust proxy](#09-rate-limiting-and-trust-proxy)** — Rate limiting should use a deliberate client identity, especially behind proxies.
- [ ] **[Avoiding business logic in routes](#10-avoiding-business-logic-in-routes)** — Routes should translate HTTP to application calls, not contain the whole use case.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [App factory vs server startup](#01-app-factory-vs-server-startup) | `startup` `architecture` `testing` |
| 02 | [Router boundaries](#02-router-boundaries) | `routing` `routers` `boundaries` |
| 03 | [Middleware order](#03-middleware-order) | `middleware` `routing` `errors` |
| 04 | [Request validation](#04-request-validation) | `validation` `requests` `security` |
| 05 | [Async route error forwarding](#05-async-route-error-forwarding) | `async` `errors` `routes` |
| 06 | [Central error handler](#06-central-error-handler) | `errors` `middleware` `responses` |
| 07 | [Response shape consistency](#07-response-shape-consistency) | `responses` `api` `contracts` |
| 08 | [Auth middleware boundaries](#08-auth-middleware-boundaries) | `auth` `middleware` `security` |
| 09 | [Rate limiting and trust proxy](#09-rate-limiting-and-trust-proxy) | `security` `rate-limiting` `proxy` |
| 10 | [Avoiding business logic in routes](#10-avoiding-business-logic-in-routes) | `architecture` `routes` `services` |

---

## 01. App factory vs server startup

> Create the Express app separately from the process that starts listening.

**Tags:** `startup` `architecture` `testing` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/app-factory-server-startup)

### ✅ Good Code — `src/app.ts`

```ts
import express from "express";
import { createReviewsRouter } from "./reviews/router";

export function createApp() {
  // Tests can import this factory without opening a port.
  const app = express();

  app.use(express.json());
  app.use("/reviews", createReviewsRouter());

  return app;
}
```

### ❌ Bad Code — `src/app.ts`

```ts
import express from "express";
import { createReviewsRouter } from "./reviews/router";

const app = express();

app.use(express.json());
app.use("/reviews", createReviewsRouter());
// Importing this module starts listening as a side effect.
app.listen(process.env.PORT || 3000);

export default app;
```

### 🔍 What to review

- **Good Code:** The good version exports an app factory, so tests and workers can create the same app without starting a server.
- **Bad Code:** The bad version starts listening as a module side effect. Importing the app for tests or scripts now opens a port unexpectedly.

### 📝 Review notes

In review, check whether the Express app can be constructed independently from `listen()`. The bad version is review-worthy because startup side effects make testing, shutdown, and multi-process usage harder than they need to be.

### 💡 Takeaway

> **An Express app should be buildable for tests without opening a port.**

---

## 02. Router boundaries

> Group related routes behind routers with explicit dependencies.

**Tags:** `routing` `routers` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/router-boundaries)

### ✅ Good Code — `src/reviews/router.ts`

```ts
import { Router } from "express";
import type { ReviewsService } from "./service";

export function createReviewsRouter(service: ReviewsService) {
  // The router owns review endpoints and receives its dependency.
  const router = Router();

  router.get("/", async (_req, res) => {
    const reviews = await service.listOpenReviews();
    res.json({ data: reviews });
  });

  router.post("/", async (req, res) => {
    const review = await service.createReview(req.body);
    res.status(201).json({ data: review });
  });

  return router;
}
```

### ❌ Bad Code — `src/app.ts`

```ts
import express from "express";
import { reviewService, userService, billingService } from "./services";

const app = express();

// Unrelated resources pile into app.ts and share global services.
app.get("/reviews", async (_req, res) => {
  res.json(await reviewService.listOpenReviews());
});
app.post("/reviews", async (req, res) => {
  res.json(await reviewService.createReview(req.body));
});
app.get("/users", async (_req, res) => {
  res.json(await userService.listUsers());
});
app.post("/billing/retry", async (req, res) => {
  res.json(await billingService.retryCharge(req.body.invoiceId));
});
```

### 🔍 What to review

- **Good Code:** The good version keeps review routes in a focused router and receives the service dependency explicitly.
- **Bad Code:** The bad version piles unrelated resources into the app entry point and reaches into shared service singletons.

### 📝 Review notes

In review, ask whether this route file has one clear resource boundary. The bad version is review-worthy because Express makes it easy to keep adding endpoints to `app.ts` until routing, dependencies, and ownership become blurry.

### 💡 Takeaway

> **Routers should own one resource area and receive services instead of reaching into globals.**

---

## 03. Middleware order

> Order middleware so parsing, security, routes, 404s, and errors run in the intended sequence.

**Tags:** `middleware` `routing` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/middleware-order)

### ✅ Good Code — `src/app.ts`

```ts
import express from "express";
import { requireUser } from "./auth";
import { errorHandler, notFoundHandler } from "./errors";
import { createReviewsRouter } from "./reviews/router";

export function createApp() {
  const app = express();

  // Parsing and auth run before routes; 404 and errors run after.
  app.use(express.json());
  app.use("/reviews", requireUser, createReviewsRouter());
  app.use(notFoundHandler);
  app.use(errorHandler);

  return app;
}
```

### ❌ Bad Code — `src/app.ts`

```ts
import express from "express";
import { requireUser } from "./auth";
import { errorHandler } from "./errors";
import { createReviewsRouter } from "./reviews/router";

export function createApp() {
  const app = express();

  // Earlier middleware cannot handle routes that run before it.
  app.use(errorHandler);
  app.use("/reviews", createReviewsRouter());
  app.use(requireUser);
  app.use(express.json());

  return app;
}
```

### 🔍 What to review

- **Good Code:** The good version parses JSON before routes, protects the router before handlers run, and places the 404 and error handlers after routes.
- **Bad Code:** The bad version registers the error handler before routes, applies auth after the router, and parses JSON too late.

### 📝 Review notes

In review, read Express middleware top to bottom like a pipeline. The bad version is review-worthy because every `app.use()` changes what later handlers can see and whether earlier handlers ever run.

### 💡 Takeaway

> **Express middleware order is behavior, not decoration.**

---

## 04. Request validation

> Validate request input at the route boundary before calling business services.

**Tags:** `validation` `requests` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/request-validation)

### ✅ Good Code — `src/reviews/router.ts`

```ts
import { Router } from "express";
import { parseCreateReviewBody } from "./validation";

export function createReviewsRouter(service: ReviewsService) {
  const router = Router();

  router.post("/", async (req, res) => {
    // Validate at the route edge before calling business logic.
    const result = parseCreateReviewBody(req.body);

    if (!result.ok) {
      res.status(400).json({ error: result.error });
      return;
    }

    const review = await service.createReview(result.value);
    res.status(201).json({ data: review });
  });

  return router;
}
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
import { Router } from "express";

export function createReviewsRouter(service: ReviewsService) {
  const router = Router();

  router.post("/", async (req, res) => {
    // Raw body fields reach the service without shape checks.
    const review = await service.createReview({
      title: req.body.title,
      reviewerIds: req.body.reviewerIds,
      dueDate: req.body.dueDate,
    });

    res.status(201).json(review);
  });

  return router;
}
```

### 🔍 What to review

- **Good Code:** The good version validates the body once at the route boundary and only passes trusted values into the service.
- **Bad Code:** The bad version copies raw body fields directly into business logic and leaves missing, malformed, or extra input to fail later.

### 📝 Review notes

In review, route boundaries are where untrusted input should become validated application data. The bad version is review-worthy because it spreads input assumptions across services and makes error responses inconsistent.

### 💡 Takeaway

> **Route handlers should pass validated values to services, not raw request bodies.**

---

## 05. Async route error forwarding

> Let async route failures reach the Express error pipeline instead of becoming unhandled work.

**Tags:** `async` `errors` `routes` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/async-route-error-forwarding)

### ✅ Good Code — `src/reviews/router.ts`

```ts
import { Router } from "express";
import { NotFoundError } from "../errors";

export function createReviewsRouter(service: ReviewsService) {
  const router = Router();

  router.get("/:id", async (req, res) => {
    // Thrown async errors flow to the central error handler.
    const review = await service.getReview(req.params.id);

    if (!review) {
      throw new NotFoundError("Review not found.");
    }

    res.json({ data: review });
  });

  return router;
}
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
import { Router } from "express";

export function createReviewsRouter(service: ReviewsService) {
  const router = Router();

  router.get("/:id", (req, res) => {
    // This promise chain is not returned or caught.
    service.getReview(req.params.id).then((review) => {
      if (!review) {
        res.status(404).json({ error: "Review not found." });
        return;
      }

      res.json({ data: review });
    });
  });

  return router;
}
```

### 🔍 What to review

- **Good Code:** The good version uses an async handler and lets thrown errors move to the central error handler.
- **Bad Code:** The bad version starts promise work without returning or catching it, so service failures can become unhandled while the request remains unfinished.

### 📝 Review notes

In review, make sure every async route has a path to the error middleware. Express 5 forwards rejected promises from async handlers, but manually started promise chains still need to be returned or caught.

### 💡 Takeaway

> **Async routes should either be awaited by Express or explicitly forwarded to next.**

---

## 06. Central error handler

> Use one Express error handler to map application errors into HTTP responses.

**Tags:** `errors` `middleware` `responses` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/central-error-handler)

### ✅ Good Code — `src/errors.ts`

```ts
import type { ErrorRequestHandler } from "express";

export const errorHandler: ErrorRequestHandler = (error, _req, res, next) => {
  // Never write a second response after headers were sent.
  if (res.headersSent) {
    next(error);
    return;
  }

  if (error.name === "ValidationError") {
    res.status(400).json({ error: error.message });
    return;
  }

  console.error({ error }, "Unhandled request error");
  res.status(500).json({ error: "Internal server error." });
};
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
router.post("/", async (req, res) => {
  try {
    const review = await service.createReview(req.body);
    res.json(review);
  } catch (error) {
    // Each route invents its own error shape and status code.
    console.error(error);
    res.status(500).send("Something broke");
  }
});

router.get("/:id", async (req, res) => {
  try {
    res.json(await service.getReview(req.params.id));
  } catch {
    res.status(400).json({ message: "Bad request" });
  }
});
```

### 🔍 What to review

- **Good Code:** The good version centralizes error mapping, checks `headersSent`, and keeps unexpected error output generic.
- **Bad Code:** The bad version repeats catch blocks in routes and returns different shapes and status codes for similar failures.

### 📝 Review notes

In review, repeated route-level catch blocks are often a sign that the error pipeline is missing or underused. The bad version is review-worthy because inconsistency makes clients harder to write and production failures harder to diagnose.

### 💡 Takeaway

> **A central error handler keeps status codes, logging, and response bodies consistent.**

---

## 07. Response shape consistency

> Return predictable success and error envelopes across routes.

**Tags:** `responses` `api` `contracts` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/response-shape-consistency)

### ✅ Good Code — `src/http/respond.ts`

```ts
import type { Response } from "express";

// Small helpers keep response envelopes consistent.
export function ok<T>(res: Response, data: T) {
  return res.json({ data });
}

export function created<T>(res: Response, data: T) {
  return res.status(201).json({ data });
}

export function fail(res: Response, status: number, message: string) {
  return res.status(status).json({ error: { message } });
}
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
router.get("/", async (_req, res) => {
  // Neighboring routes now need different client parsing rules.
  res.json(await service.listOpenReviews());
});

router.post("/", async (req, res) => {
  res.status(201).json({ review: await service.createReview(req.body) });
});

router.get("/:id", async (req, res) => {
  const review = await service.getReview(req.params.id);

  if (!review) {
    res.status(404).send("not found");
    return;
  }

  res.json({ data: { review } });
});
```

### 🔍 What to review

- **Good Code:** The good version gives routes small helpers for success and error responses, making the API contract consistent.
- **Bad Code:** The bad version returns arrays, objects, strings, and nested data shapes depending on the route.

### 📝 Review notes

In review, compare response bodies across neighboring endpoints. The bad version is review-worthy because inconsistent API envelopes push accidental complexity onto every client and test.

### 💡 Takeaway

> **Clients should not need route-specific parsing rules for common API responses.**

---

## 08. Auth middleware boundaries

> Authenticate once in middleware and pass a safe user context to route handlers.

**Tags:** `auth` `middleware` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/auth-middleware-boundaries)

### ✅ Good Code — `src/auth/require-user.ts`

```ts
import type { NextFunction, Request, Response } from "express";

export async function requireUser(
  req: Request,
  res: Response,
  next: NextFunction,
) {
  // Middleware turns credentials into a small trusted user context.
  const session = await sessions.findFromHeader(req.get("authorization"));

  if (!session) {
    res.status(401).json({ error: { message: "Authentication required." } });
    return;
  }

  res.locals.user = {
    id: session.userId,
    role: session.role,
  };
  next();
}
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
router.post("/:id/approve", async (req, res) => {
  // Auth, authorization, and approval logic are mixed together.
  const token = req.get("authorization");
  const session = await sessions.findFromHeader(token);

  if (!session || session.role !== "maintainer") {
    res.status(403).json({ error: "Forbidden" });
    return;
  }

  const review = await service.approveReview(req.params.id, session.userId);
  res.json({ data: review });
});
```

### 🔍 What to review

- **Good Code:** The good version authenticates in middleware and stores only the user context downstream routes need.
- **Bad Code:** The bad version reparses credentials inside the route, mixing authentication, authorization, and review approval in one handler.

### 📝 Review notes

In review, auth code should have a clear boundary and consistent failure response. The bad version is review-worthy because each route can drift into its own auth logic, increasing security risk and duplication.

### 💡 Takeaway

> **Routes should consume trusted auth context instead of reparsing credentials.**

---

## 09. Rate limiting and trust proxy

> Configure client IP trust deliberately before using IP-based controls.

**Tags:** `security` `rate-limiting` `proxy` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/rate-limiting-trust-proxy)

### ✅ Good Code — `src/app.ts`

```ts
import express from "express";
import { rateLimit } from "express-rate-limit";

export function createApp() {
  const app = express();

  // Trust the proxy hop count used in this deployment.
  app.set("trust proxy", 1);
  app.use(rateLimit({
    windowMs: 60_000,
    limit: 120,
    standardHeaders: true,
    legacyHeaders: false,
  }));

  return app;
}
```

### ❌ Bad Code — `src/app.ts`

```ts
import express from "express";
import { rateLimit } from "express-rate-limit";

export function createApp() {
  const app = express();

  // Trusting every proxy hop makes IP controls easier to spoof.
  app.set("trust proxy", true);
  app.use(rateLimit({ windowMs: 60_000, limit: 10_000 }));

  return app;
}
```

### 🔍 What to review

- **Good Code:** The good version sets a specific proxy trust depth and applies a clear rate limit policy.
- **Bad Code:** The bad version trusts every proxy hop and sets a limit so high that the control is mostly decorative.

### 📝 Review notes

In review, security middleware needs deployment context. The bad version is review-worthy because IP-based controls depend on whether Express can trust the source address it sees.

### 💡 Takeaway

> **Rate limiting should use a deliberate client identity, especially behind proxies.**

---

## 10. Avoiding business logic in routes

> Keep route handlers thin by delegating business decisions to services.

**Tags:** `architecture` `routes` `services` · [View on site](https://good-code-bad-code.pages.dev/tracks/express/business-logic-out-of-routes)

### ✅ Good Code — `src/reviews/router.ts`

```ts
router.post("/:id/approve", async (req, res) => {
  // The route only translates HTTP data into a workflow call.
  const review = await reviewWorkflow.approve({
    reviewId: req.params.id,
    approverId: res.locals.user.id,
  });

  res.json({ data: review });
});
```

### ❌ Bad Code — `src/reviews/router.ts`

```ts
router.post("/:id/approve", async (req, res) => {
  // Database writes, rules, and notifications are hidden here.
  const review = await db.review.findUnique({ where: { id: req.params.id } });

  if (!review || review.status !== "pending") {
    res.status(409).json({ error: "Review cannot be approved." });
    return;
  }

  await db.review.update({
    where: { id: req.params.id },
    data: { status: "approved", approvedBy: res.locals.user.id },
  });
  await notifications.sendApprovalEmail(review.authorId);

  res.json({ ok: true });
});
```

### 🔍 What to review

- **Good Code:** The good version keeps the route focused on HTTP input and output while the workflow owns approval rules and side effects.
- **Bad Code:** The bad version performs database reads, business checks, writes, notification side effects, and response formatting in one route.

### 📝 Review notes

In review, route handlers should be easy to scan even when the use case is complex. The bad version is review-worthy because business rules hidden inside Express handlers are harder to reuse, test, and transact safely.

### 💡 Takeaway

> **Routes should translate HTTP to application calls, not contain the whole use case.**

---

[← Back to index](README.md)
