# NestJS — Code Review Handbook

> Review modules, controllers, providers, validation pipes, guards, filters, interceptors, config, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/nestjs](https://good-code-bad-code.pages.dev/tracks/nestjs)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [React](05-react.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Node.js](08-nodejs.md), [Express](09-express.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Module, controller, and provider boundaries](#01-module-controller-and-provider-boundaries)** — Controllers should translate HTTP requests while providers own application work and modules wire the feature together.
- [ ] **[Dependency injection providers](#02-dependency-injection-providers)** — Constructor injection makes dependencies visible and gives tests a stable place to replace infrastructure.
- [ ] **[DTO validation pipes](#03-dto-validation-pipes)** — A controller should receive data that already passed DTO validation and transformation rules.
- [ ] **[Guards for auth and authorization](#04-guards-for-auth-and-authorization)** — Authorization checks belong before the route body, not halfway through controller logic.
- [ ] **[Exception filters and error shape](#05-exception-filters-and-error-shape)** — Exception filters should translate failures into consistent responses without leaking internal details.
- [ ] **[Interceptors for response and logging work](#06-interceptors-for-response-and-logging-work)** — Interceptors should wrap route execution without forcing every controller method to repeat the same timing or response code.
- [ ] **[Configuration module validation](#07-configuration-module-validation)** — Configuration should be parsed once at startup and injected as typed values instead of read ad hoc from process.env.
- [ ] **[Repository and service boundaries](#08-repository-and-service-boundaries)** — Services should express use cases while repositories express database queries and persistence details.
- [ ] **[Request lifecycle: middleware and guards](#09-request-lifecycle-middleware-and-guards)** — Middleware is for low-level request work; guards decide access before pipes and controller logic run.
- [ ] **[Testing module overrides](#10-testing-module-overrides)** — A test should replace external providers at the module boundary instead of reaching into service internals.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Module, controller, and provider boundaries](#01-module-controller-and-provider-boundaries) | `modules` `controllers` `providers` |
| 02 | [Dependency injection providers](#02-dependency-injection-providers) | `providers` `dependency-injection` `testing` |
| 03 | [DTO validation pipes](#03-dto-validation-pipes) | `dto` `validation` `pipes` |
| 04 | [Guards for auth and authorization](#04-guards-for-auth-and-authorization) | `guards` `auth` `authorization` |
| 05 | [Exception filters and error shape](#05-exception-filters-and-error-shape) | `errors` `filters` `http` |
| 06 | [Interceptors for response and logging work](#06-interceptors-for-response-and-logging-work) | `interceptors` `logging` `responses` |
| 07 | [Configuration module validation](#07-configuration-module-validation) | `configuration` `environment` `validation` |
| 08 | [Repository and service boundaries](#08-repository-and-service-boundaries) | `repositories` `services` `database` |
| 09 | [Request lifecycle: middleware and guards](#09-request-lifecycle-middleware-and-guards) | `middleware` `guards` `lifecycle` |
| 10 | [Testing module overrides](#10-testing-module-overrides) | `testing` `providers` `overrides` |

---

## 01. Module, controller, and provider boundaries

> Separate NestJS modules, controllers, and providers so routing, application logic, and feature wiring stay reviewable.

**Tags:** `modules` `controllers` `providers` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/module-controller-provider-boundaries)

### ✅ Good Code — `reviews.module.ts`

```ts
import { Body, Controller, Injectable, Module, Post } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  constructor(private readonly reviews: ReviewsService) {}

  // The controller reads HTTP input and delegates application work.
  @Post()
  create(@Body() body: CreateReviewDto) {
    return this.reviews.create(body);
  }
}

@Injectable()
export class ReviewsService {
  create(input: CreateReviewDto) {
    return { id: crypto.randomUUID(), ...input };
  }
}

@Module({
  controllers: [ReviewsController],
  providers: [ReviewsService],
})
export class ReviewsModule {}
```

### ❌ Bad Code — `reviews.controller.ts`

```ts
import { Body, Controller, Post } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  @Post()
  async create(@Body() body: any) {
    // The controller owns persistence, ids, and HTTP input in one method.
    const review = { id: Date.now(), ...body };
    await db.collection("reviews").insertOne(review);
    return review;
  }
}
```

### 🔍 What to review

- **Good Code:** The good version gives the controller one job: accept HTTP input and call the service. The module declares the controller and provider that make the feature available.
- **Bad Code:** The bad version puts database writes and id generation inside the controller. That makes route code harder to test and couples HTTP concerns to persistence decisions.

### 📝 Review notes

In NestJS reviews, follow a request from module to controller to provider. If a controller method grows into business logic or persistence logic, move that work behind an injected provider.

### 💡 Takeaway

> **Controllers should translate HTTP requests while providers own application work and modules wire the feature together.**

---

## 02. Dependency injection providers

> Use NestJS providers and injection tokens so services declare their collaborators instead of constructing infrastructure directly.

**Tags:** `providers` `dependency-injection` `testing` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/dependency-injection-providers)

### ✅ Good Code — `reviews.service.ts`

```ts
import { Inject, Injectable } from "@nestjs/common";

export const REVIEWS_REPOSITORY = Symbol("REVIEWS_REPOSITORY");

@Injectable()
export class ReviewsService {
  constructor(
    @Inject(REVIEWS_REPOSITORY)
    private readonly reviews: ReviewsRepository,
  ) {}

  // The service declares the repository it needs.
  findPending() {
    return this.reviews.findByStatus("pending");
  }
}
```

### ❌ Bad Code — `reviews.service.ts`

```ts
import { Injectable } from "@nestjs/common";
import { MongoClient } from "mongodb";

@Injectable()
export class ReviewsService {
  async findPending() {
    // Direct construction hides connection and test boundaries.
    const client = new MongoClient(process.env.MONGO_URL!);
    const reviews = client.db().collection("reviews");
    return reviews.find({ status: "pending" }).toArray();
  }
}
```

### 🔍 What to review

- **Good Code:** The good version declares a repository dependency at the constructor boundary. Tests can provide a fake repository without opening a database connection.
- **Bad Code:** The bad version creates a database client inside the method. Every call now depends on environment variables, network access, and driver setup.

### 📝 Review notes

Provider injection is the main seam for testability in NestJS. When a service calls `new` for infrastructure, ask whether that collaborator should be a provider registered in the module.

### 💡 Takeaway

> **Constructor injection makes dependencies visible and gives tests a stable place to replace infrastructure.**

---

## 03. DTO validation pipes

> Validate request DTOs at the NestJS boundary before controller methods receive untrusted input.

**Tags:** `dto` `validation` `pipes` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/dto-validation-pipes)

### ✅ Good Code — `main.ts`

```ts
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // The global pipe rejects unknown fields before controllers run.
  app.useGlobalPipes(
    new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }),
  );

  await app.listen(3000);
}
bootstrap();
```

### ❌ Bad Code — `reviews.controller.ts`

```ts
import { Body, Controller, Post } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  @Post()
  create(@Body() body: any) {
    // Validation is scattered inside route code and misses extra fields.
    if (!body.title) throw new Error("title is required");
    return this.reviews.create(body);
  }
}
```

### 🔍 What to review

- **Good Code:** The good version applies `ValidationPipe` at the application boundary, so DTO rules run before controller methods.
- **Bad Code:** The bad version validates one field by hand inside the route. Extra request fields can still pass through to the service.

### 📝 Review notes

Review NestJS input by finding the pipe boundary first. If DTO validation is scattered inside methods, errors become inconsistent and request shapes are harder to trust.

### 💡 Takeaway

> **A controller should receive data that already passed DTO validation and transformation rules.**

---

## 04. Guards for auth and authorization

> Use guards to approve or reject a request before protected controller code runs.

**Tags:** `guards` `auth` `authorization` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/guards-auth-authorization)

### ✅ Good Code — `reviews.controller.ts`

```ts
import { Controller, Get, UseGuards } from "@nestjs/common";

@Controller("reviews")
@UseGuards(SessionGuard, ReviewerRoleGuard)
export class ReviewsController {
  // Guards decide access before this method runs.
  @Get("pending")
  findPending() {
    return this.reviews.findPending();
  }
}
```

### ❌ Bad Code — `reviews.controller.ts`

```ts
import { Controller, Get, Req } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  @Get("pending")
  findPending(@Req() request: any) {
    // Access checks are mixed into route behavior.
    if (!request.user || request.user.role !== "reviewer") {
      return [];
    }
    return this.reviews.findPending();
  }
}
```

### 🔍 What to review

- **Good Code:** The good version declares authentication and role checks at the controller boundary. A reviewer can see access policy before reading route behavior.
- **Bad Code:** The bad version hides authorization inside the route method and returns an empty list on denial. That can mask a security bug as normal application behavior.

### 📝 Review notes

In NestJS, guards are the review point for access control. Look for routes that read `request.user` directly and decide whether that logic belongs in a reusable guard.

### 💡 Takeaway

> **Authorization checks belong before the route body, not halfway through controller logic.**

---

## 05. Exception filters and error shape

> Use exception filters when an API needs one response shape for logged and user-facing errors.

**Tags:** `errors` `filters` `http` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/exception-filters-error-shape)

### ✅ Good Code — `http-exception.filter.ts`

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
} from "@nestjs/common";

@Catch(HttpException)
export class ApiExceptionFilter implements ExceptionFilter {
  // The filter owns the public error response shape.
  catch(error: HttpException, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();
    const status = error.getStatus();

    response.status(status).json({
      statusCode: status,
      error: "REQUEST_FAILED",
      message: error.message,
    });
  }
}
```

### ❌ Bad Code — `reviews.controller.ts`

```ts
import { Controller, Get, Param } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  @Get(":id")
  async findOne(@Param("id") id: string) {
    try {
      return await this.reviews.findOne(id);
    } catch (error) {
      // Raw errors can leak internal messages to clients.
      return { ok: false, error: String(error) };
    }
  }
}
```

### 🔍 What to review

- **Good Code:** The good version puts API error formatting in a filter. Route methods can throw framework exceptions and still return a consistent JSON shape.
- **Bad Code:** The bad version catches unknown errors inside a route and serializes them. Internal messages can leak to clients and logs may lose the original stack.

### 📝 Review notes

Use filters for response shape and logging policy. Avoid route-level catch blocks that turn every failure into a success response with an error object.

### 💡 Takeaway

> **Exception filters should translate failures into consistent responses without leaking internal details.**

---

## 06. Interceptors for response and logging work

> Use interceptors for cross-cutting response mapping, timing, and logging around controller method execution.

**Tags:** `interceptors` `logging` `responses` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/interceptors-response-logging)

### ✅ Good Code — `request-timing.interceptor.ts`

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from "@nestjs/common";
import { tap } from "rxjs";

@Injectable()
export class RequestTimingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const startedAt = Date.now();

    // The interceptor measures every decorated route the same way.
    return next.handle().pipe(
      tap(() => {
        const request = context.switchToHttp().getRequest();
        logger.info({ path: request.url, durationMs: Date.now() - startedAt });
      }),
    );
  }
}
```

### ❌ Bad Code — `reviews.controller.ts`

```ts
import { Controller, Get } from "@nestjs/common";

@Controller("reviews")
export class ReviewsController {
  @Get()
  async findAll() {
    // Repeating timing code in each route invites drift.
    const startedAt = Date.now();
    const result = await this.reviews.findAll();
    logger.info({ durationMs: Date.now() - startedAt });
    return { data: result };
  }
}
```

### 🔍 What to review

- **Good Code:** The good version puts request timing in an interceptor that wraps the controller call. The controller can focus on returning reviews.
- **Bad Code:** The bad version repeats timing and response wrapping inside a route. Other routes can drift in field names, timing behavior, or log context.

### 📝 Review notes

Interceptors fit cross-cutting work around handler execution. If many controllers repeat response mapping, timing, or cache checks, review whether an interceptor would make the policy visible once.

### 💡 Takeaway

> **Interceptors should wrap route execution without forcing every controller method to repeat the same timing or response code.**

---

## 07. Configuration module validation

> Validate NestJS configuration at startup so missing environment variables fail before requests arrive.

**Tags:** `configuration` `environment` `validation` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/config-module-validation)

### ✅ Good Code — `app.module.ts`

```ts
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validate(config) {
        // Startup fails if required runtime config is absent.
        if (!config.DATABASE_URL) {
          throw new Error("DATABASE_URL is required");
        }
        return config;
      },
    }),
  ],
})
export class AppModule {}
```

### ❌ Bad Code — `reviews.repository.ts`

```ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class ReviewsRepository {
  connect() {
    // A missing env var appears only when this method is called.
    return connectToDatabase(process.env.DATABASE_URL!);
  }
}
```

### 🔍 What to review

- **Good Code:** The good version validates required runtime configuration while the app boots. A bad deployment fails before serving traffic.
- **Bad Code:** The bad version reads `process.env` inside a repository and asserts that the value exists. Missing configuration becomes a runtime request failure.

### 📝 Review notes

Configuration is part of the application boundary. In NestJS reviews, prefer a config module that validates and exposes values through injection over scattered `process.env` reads.

### 💡 Takeaway

> **Configuration should be parsed once at startup and injected as typed values instead of read ad hoc from process.env.**

---

## 08. Repository and service boundaries

> Keep NestJS services focused on application rules and put database details behind repository providers.

**Tags:** `repositories` `services` `database` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/repository-service-boundaries)

### ✅ Good Code — `reviews.service.ts`

```ts
import { Injectable, NotFoundException } from "@nestjs/common";

@Injectable()
export class ReviewsService {
  constructor(private readonly reviews: ReviewsRepository) {}

  // The service owns the use case and the repository owns the query.
  async approve(reviewId: string, reviewerId: string) {
    const review = await this.reviews.findPendingById(reviewId);
    if (!review) throw new NotFoundException("Review not found");

    return this.reviews.markApproved(reviewId, reviewerId);
  }
}
```

### ❌ Bad Code — `reviews.service.ts`

```ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class ReviewsService {
  async approve(reviewId: string, reviewerId: string) {
    // SQL details and application rules are tangled in one method.
    const review = await db.query("select * from reviews where id = $1", [reviewId]);
    await db.query("update reviews set status = 'approved' where id = $1", [
      reviewId,
    ]);
    return review.rows[0];
  }
}
```

### 🔍 What to review

- **Good Code:** The good version lets the service read like a use case: find a pending review, reject missing data, then approve it.
- **Bad Code:** The bad version mixes SQL details with application decisions. Query shape, status transitions, and error semantics are harder to test separately.

### 📝 Review notes

Repository boundaries are useful when query details start hiding the use case. Keep database-specific code behind a provider so service tests can focus on behavior.

### 💡 Takeaway

> **Services should express use cases while repositories express database queries and persistence details.**

---

## 09. Request lifecycle: middleware and guards

> Place middleware, guards, pipes, and interceptors according to the NestJS request lifecycle.

**Tags:** `middleware` `guards` `lifecycle` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/request-lifecycle-middleware-guards)

### ✅ Good Code — `request-context.middleware.ts`

```ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  NestMiddleware,
} from "@nestjs/common";
import type { NextFunction, Response } from "express";

@Injectable()
export class RequestContextMiddleware implements NestMiddleware {
  // Middleware attaches low-level request metadata.
  use(request: RequestWithContext, _response: Response, next: NextFunction) {
    request.requestId = crypto.randomUUID();
    next();
  }
}

@Injectable()
export class AdminGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    const request = context.switchToHttp().getRequest<RequestWithContext>();
    return request.user?.role === "admin";
  }
}
```

### ❌ Bad Code — `auth.middleware.ts`

```ts
import { Injectable, NestMiddleware } from "@nestjs/common";

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(request: any, response: any, next: () => void) {
    // Access policy hidden in middleware can bypass controller metadata.
    if (request.path.startsWith("/admin") && request.user?.role !== "admin") {
      response.status(403).send("Forbidden");
      return;
    }
    next();
  }
}
```

### 🔍 What to review

- **Good Code:** The good version uses middleware for request metadata and a guard for authorization. Each concern maps to a lifecycle stage.
- **Bad Code:** The bad version hides route access policy inside path-string checks. Controller metadata and route decorators no longer tell the full security story.

### 📝 Review notes

NestJS request flow has distinct review points. Use middleware for raw request preparation, guards for access decisions, pipes for input validation, and interceptors for work around handler execution.

### 💡 Takeaway

> **Middleware is for low-level request work; guards decide access before pipes and controller logic run.**

---

## 10. Testing module overrides

> Use NestJS testing modules and provider overrides so tests exercise application wiring without real infrastructure.

**Tags:** `testing` `providers` `overrides` · [View on site](https://good-code-bad-code.pages.dev/tracks/nestjs/testing-module-overrides)

### ✅ Good Code — `reviews.service.spec.ts`

```ts
import { Test } from "@nestjs/testing";

it("returns pending reviews", async () => {
  const repository = { findByStatus: vi.fn().mockResolvedValue([]) };

  // The testing module replaces infrastructure at the provider boundary.
  const moduleRef = await Test.createTestingModule({
    providers: [ReviewsService, { provide: REVIEWS_REPOSITORY, useValue: repository }],
  }).compile();

  const service = moduleRef.get(ReviewsService);
  await service.findPending();

  expect(repository.findByStatus).toHaveBeenCalledWith("pending");
});
```

### ❌ Bad Code — `reviews.service.spec.ts`

```ts
it("returns pending reviews", async () => {
  const service = new ReviewsService();

  // The test patches a private field after construction.
  (service as any).reviews = { findByStatus: vi.fn().mockResolvedValue([]) };

  await service.findPending();
});
```

### 🔍 What to review

- **Good Code:** The good version builds a testing module and replaces the repository provider. The test uses the same injection path as the application.
- **Bad Code:** The bad version constructs the service manually and writes a private dependency after the fact. That can hide broken module registration.

### 📝 Review notes

For NestJS tests, prefer overriding providers at the module boundary. If tests patch private fields, they may pass while production dependency injection is broken.

### 💡 Takeaway

> **A test should replace external providers at the module boundary instead of reaching into service internals.**

---

[← Back to index](README.md)
