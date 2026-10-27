# Good Code Bad Code — Code Review Handbook

คู่มือรีวิวโค้ดแบบเทียบ **Good Code vs Bad Code** ครบ **27 tracks / 270 บทเรียน** เรียบเรียงจาก [https://good-code-bad-code.pages.dev](https://good-code-bad-code.pages.dev) เพื่อใช้เป็นมาตรฐานอ้างอิงในการเขียนและรีวิวโค้ดของทุกโปรเจกต์

## วิธีใช้คู่มือนี้

1. **เลือก Track** ตามภาษา / framework / เครื่องมือที่โปรเจกต์ใช้
2. **เทียบตัวอย่าง** อ่าน Good Code กับ Bad Code ก่อน แล้วถามตัวเองว่า *อะไรเปลี่ยน ลดความเสี่ยงอะไร และโค้ดเบสเรามีแพทเทิร์นนี้ไหม*
3. **ใช้ Takeaway เป็นเช็กลิสต์** — ทุกไฟล์มี *Review Checklist* ด้านบน คัดลอกไปใส่ PR template หรือ `CONTRIBUTING.md` ได้ทันที
4. รวมเช็กลิสต์ทุก track ไว้ที่ [CHECKLIST.md](CHECKLIST.md) สำหรับใช้ตอนรีวิว PR

**โครงสร้างของแต่ละบทเรียน:** Summary → Tags → ✅ Good Code → ❌ Bad Code → 🔍 What to review → 📝 Review notes → 💡 Takeaway

> เนื้อหาโค้ดและคำอธิบายคงไว้เป็นภาษาอังกฤษตามต้นฉบับ เพื่อความถูกต้องของศัพท์เทคนิค

## Foundations

| # | Track | Focus | Lessons |
|---|-------|-------|---------|
| 01 | [HTML](01-html.md) | Review structure, semantics, forms, and accessible document foundations. | 10 |
| 02 | [CSS](02-css.md) | Review layout, responsive styling, contrast, and maintainable selectors. | 10 |
| 03 | [JavaScript](03-javascript.md) | Review control flow, async behavior, naming, and side effects. | 10 |
| 04 | [TypeScript](04-typescript.md) | Review type narrowing, API boundaries, unions, and safer function shapes. | 10 |
| 22 | [Git](22-git.md) | Review status checks, staging, commits, branches, sync workflows, diffs, conflicts, history safety, ignores, and release tags. | 10 |

## Frontend

| # | Track | Focus | Lessons |
|---|-------|-------|---------|
| 05 | [React](05-react.md) | Review component boundaries, state, effects, and rendering decisions. | 10 |
| 06 | [Vue](06-vue.md) | Review single-file components, props, emits, reactivity, composables, stores, and slots. | 10 |
| 07 | [Next.js](07-nextjs.md) | Review App Router structure, server boundaries, data fetching, and route APIs. | 10 |
| 27 | [Tailwind CSS](27-tailwindcss.md) | Review utility composition, responsive variants, states, theme tokens, dark mode, reuse boundaries, accessibility, and class conflicts. | 10 |

## Backend

| # | Track | Focus | Lessons |
|---|-------|-------|---------|
| 08 | [Node.js](08-nodejs.md) | Review runtime boundaries, async I/O, HTTP lifecycle, logging, and process safety. | 10 |
| 09 | [Express](09-express.md) | Review app factories, routers, middleware order, validation, auth, and error handling. | 10 |
| 10 | [NestJS](10-nestjs.md) | Review modules, controllers, providers, validation pipes, guards, filters, interceptors, config, and tests. | 10 |
| 12 | [Python](12-python.md) | Review readable functions, data shapes, exceptions, typing, testability, and async boundaries. | 10 |
| 13 | [PHP](13-php.md) | Review strict types, request validation, output escaping, PDO queries, passwords, sessions, configuration, and template boundaries. | 10 |
| 14 | [Laravel](14-laravel.md) | Review routes, controllers, Form Requests, middleware, container bindings, Eloquent, migrations, resources, policies, queues, and feature tests. | 10 |
| 15 | [Java](15-java.md) | Review null boundaries, immutability, value objects, exceptions, resources, generics, streams, concurrency, records, and service boundaries. | 10 |
| 16 | [Kotlin](16-kotlin.md) | Review null safety, data classes, sealed states, scope functions, collections, coroutines, Flow, JVM interop, and tests. | 10 |
| 17 | [C](17-c.md) | Review pointers, buffer bounds, allocation ownership, const contracts, structs, error codes, file cleanup, integer limits, compiler flags, and tests. | 10 |
| 18 | [C++](18-cpp.md) | Review RAII, smart pointers, move semantics, const references, rule of zero, exception boundaries, templates, algorithms, locks, and tests. | 10 |
| 19 | [C#](19-csharp.md) | Review nullable references, records, async cancellation, dependency injection, disposal, exceptions, LINQ, pattern matching, options, and tests. | 10 |
| 20 | [Rust](20-rust.md) | Review ownership, borrowing, lifetimes, Result and Option boundaries, traits, async safety, Cargo features, and tests. | 10 |
| 21 | [Lua](21-lua.md) | Review table shapes, nil boundaries, modules, metatables, coroutines, sandboxed execution, embedded APIs, and tests. | 10 |
| 23 | [FastAPI](23-fastapi.md) | Review routes, Pydantic models, dependencies, auth, async handlers, database sessions, and API tests. | 10 |
| 24 | [Django](24-django.md) | Review URL patterns, models, QuerySets, forms, views, CSRF, settings, transactions, and tests. | 10 |
| 25 | [Go](25-go.md) | Review packages, exported APIs, errors, context, interfaces, goroutines, cleanup, tests, and HTTP handlers. | 10 |

## Data + Ops

| # | Track | Focus | Lessons |
|---|-------|-------|---------|
| 11 | [SQL](11-sql.md) | Review schemas, queries, joins, indexes, transactions, migrations, and data safety. | 10 |
| 26 | [Docker](26-docker.md) | Review build contexts, base images, cache layers, multi-stage builds, secrets, users, healthchecks, and Compose files. | 10 |

## Lesson Map

<details>
<summary><b>01. HTML</b> — 10 lessons</summary>

1. [Document language and metadata](01-html.md#01-document-language-and-metadata)
2. [Semantic document structure](01-html.md#02-semantic-document-structure)
3. [Heading hierarchy](01-html.md#03-heading-hierarchy)
4. [Links and navigation](01-html.md#04-links-and-navigation)
5. [Images and alternative text](01-html.md#05-images-and-alternative-text)
6. [Accessible form labels](01-html.md#06-accessible-form-labels)
7. [Form help text and errors](01-html.md#07-form-help-text-and-errors)
8. [Interactive elements](01-html.md#08-interactive-elements)
9. [Tables for tabular data](01-html.md#09-tables-for-tabular-data)
10. [ARIA restraint](01-html.md#10-aria-restraint)

</details>

<details>
<summary><b>02. CSS</b> — 10 lessons</summary>

1. [Box model sizing](02-css.md#01-box-model-sizing)
2. [Cascade and specificity](02-css.md#02-cascade-and-specificity)
3. [Flex layout boundaries](02-css.md#03-flex-layout-boundaries)
4. [Grid layout tracks](02-css.md#04-grid-layout-tracks)
5. [Responsive units](02-css.md#05-responsive-units)
6. [Color contrast states](02-css.md#06-color-contrast-states)
7. [State selectors and focus-visible](02-css.md#07-state-selectors-and-focus-visible)
8. [Logical properties](02-css.md#08-logical-properties)
9. [Custom properties as design tokens](02-css.md#09-custom-properties-as-design-tokens)
10. [Motion and reduced motion](02-css.md#10-motion-and-reduced-motion)

</details>

<details>
<summary><b>03. JavaScript</b> — 10 lessons</summary>

1. [Strict equality and nullish checks](03-javascript.md#01-strict-equality-and-nullish-checks)
2. [Guard clauses](03-javascript.md#02-guard-clauses)
3. [Validating data boundaries](03-javascript.md#03-validating-data-boundaries)
4. [Naming side effects](03-javascript.md#04-naming-side-effects)
5. [Array transformations](03-javascript.md#05-array-transformations)
6. [Async error handling](03-javascript.md#06-async-error-handling)
7. [Promise concurrency](03-javascript.md#07-promise-concurrency)
8. [Event listener cleanup](03-javascript.md#08-event-listener-cleanup)
9. [Module boundaries and globals](03-javascript.md#09-module-boundaries-and-globals)
10. [Dates and time zones](03-javascript.md#10-dates-and-time-zones)

</details>

<details>
<summary><b>04. TypeScript</b> — 10 lessons</summary>

1. [Type inference and explicit boundaries](04-typescript.md#01-type-inference-and-explicit-boundaries)
2. [Narrowing unknown](04-typescript.md#02-narrowing-unknown)
3. [Avoiding unsafe assertions](04-typescript.md#03-avoiding-unsafe-assertions)
4. [Discriminated unions](04-typescript.md#04-discriminated-unions)
5. [Exhaustive checks with never](04-typescript.md#05-exhaustive-checks-with-never)
6. [Precise function types](04-typescript.md#06-precise-function-types)
7. [Optional properties vs nullable values](04-typescript.md#07-optional-properties-vs-nullable-values)
8. [Generic constraints](04-typescript.md#08-generic-constraints)
9. [Utility types at API boundaries](04-typescript.md#09-utility-types-at-api-boundaries)
10. [Type-only imports and module boundaries](04-typescript.md#10-type-only-imports-and-module-boundaries)

</details>

<details>
<summary><b>05. React</b> — 10 lessons</summary>

1. [Props component boundaries](05-react.md#01-props-component-boundaries)
2. [Stable keys in lists](05-react.md#02-stable-keys-in-lists)
3. [Derived state](05-react.md#03-derived-state)
4. [Updating state from previous state](05-react.md#04-updating-state-from-previous-state)
5. [Controlled form inputs](05-react.md#05-controlled-form-inputs)
6. [Effect dependencies](05-react.md#06-effect-dependencies)
7. [Async effect cleanup](05-react.md#07-async-effect-cleanup)
8. [Context boundaries](05-react.md#08-context-boundaries)
9. [Memoization when it helps](05-react.md#09-memoization-when-it-helps)
10. [Composition over prop flags](05-react.md#10-composition-over-prop-flags)

</details>

<details>
<summary><b>06. Vue</b> — 10 lessons</summary>

1. [Single-file component boundaries](06-vue.md#01-single-file-component-boundaries)
2. [Props and emits contracts](06-vue.md#02-props-and-emits-contracts)
3. [Computed values versus watchers](06-vue.md#03-computed-values-versus-watchers)
4. [List rendering keys](06-vue.md#04-list-rendering-keys)
5. [Form v-model boundaries](06-vue.md#05-form-v-model-boundaries)
6. [Composables and state ownership](06-vue.md#06-composables-and-state-ownership)
7. [Async state, loading, and errors](06-vue.md#07-async-state-loading-and-errors)
8. [Pinia store actions](06-vue.md#08-pinia-store-actions)
9. [Provide and inject boundaries](06-vue.md#09-provide-and-inject-boundaries)
10. [Slots for component composition](06-vue.md#10-slots-for-component-composition)

</details>

<details>
<summary><b>07. Next.js</b> — 10 lessons</summary>

1. [App Router file conventions](07-nextjs.md#01-app-router-file-conventions)
2. [Layout and page boundaries](07-nextjs.md#02-layout-and-page-boundaries)
3. [Server vs Client Components](07-nextjs.md#03-server-vs-client-components)
4. [Small client component islands](07-nextjs.md#04-small-client-component-islands)
5. [Server-side data fetching](07-nextjs.md#05-server-side-data-fetching)
6. [Parallel data fetching](07-nextjs.md#06-parallel-data-fetching)
7. [Loading and error boundaries](07-nextjs.md#07-loading-and-error-boundaries)
8. [Caching and revalidation intent](07-nextjs.md#08-caching-and-revalidation-intent)
9. [Metadata per route](07-nextjs.md#09-metadata-per-route)
10. [Route handlers as API boundaries](07-nextjs.md#10-route-handlers-as-api-boundaries)

</details>

<details>
<summary><b>08. Node.js</b> — 10 lessons</summary>

1. [Runtime boundaries and entry points](08-nodejs.md#01-runtime-boundaries-and-entry-points)
2. [Async filesystem without blocking](08-nodejs.md#02-async-filesystem-without-blocking)
3. [Async failure boundaries](08-nodejs.md#03-async-failure-boundaries)
4. [Environment variables and configuration](08-nodejs.md#04-environment-variables-and-configuration)
5. [Module side effects and startup code](08-nodejs.md#05-module-side-effects-and-startup-code)
6. [Streams for large payloads](08-nodejs.md#06-streams-for-large-payloads)
7. [HTTP request lifecycle](08-nodejs.md#07-http-request-lifecycle)
8. [Graceful shutdown and process signals](08-nodejs.md#08-graceful-shutdown-and-process-signals)
9. [Structured logging](08-nodejs.md#09-structured-logging)
10. [Input validation and path safety](08-nodejs.md#10-input-validation-and-path-safety)

</details>

<details>
<summary><b>09. Express</b> — 10 lessons</summary>

1. [App factory vs server startup](09-express.md#01-app-factory-vs-server-startup)
2. [Router boundaries](09-express.md#02-router-boundaries)
3. [Middleware order](09-express.md#03-middleware-order)
4. [Request validation](09-express.md#04-request-validation)
5. [Async route error forwarding](09-express.md#05-async-route-error-forwarding)
6. [Central error handler](09-express.md#06-central-error-handler)
7. [Response shape consistency](09-express.md#07-response-shape-consistency)
8. [Auth middleware boundaries](09-express.md#08-auth-middleware-boundaries)
9. [Rate limiting and trust proxy](09-express.md#09-rate-limiting-and-trust-proxy)
10. [Avoiding business logic in routes](09-express.md#10-avoiding-business-logic-in-routes)

</details>

<details>
<summary><b>10. NestJS</b> — 10 lessons</summary>

1. [Module, controller, and provider boundaries](10-nestjs.md#01-module-controller-and-provider-boundaries)
2. [Dependency injection providers](10-nestjs.md#02-dependency-injection-providers)
3. [DTO validation pipes](10-nestjs.md#03-dto-validation-pipes)
4. [Guards for auth and authorization](10-nestjs.md#04-guards-for-auth-and-authorization)
5. [Exception filters and error shape](10-nestjs.md#05-exception-filters-and-error-shape)
6. [Interceptors for response and logging work](10-nestjs.md#06-interceptors-for-response-and-logging-work)
7. [Configuration module validation](10-nestjs.md#07-configuration-module-validation)
8. [Repository and service boundaries](10-nestjs.md#08-repository-and-service-boundaries)
9. [Request lifecycle: middleware and guards](10-nestjs.md#09-request-lifecycle-middleware-and-guards)
10. [Testing module overrides](10-nestjs.md#10-testing-module-overrides)

</details>

<details>
<summary><b>11. SQL</b> — 10 lessons</summary>

1. [Schema keys and constraints](11-sql.md#01-schema-keys-and-constraints)
2. [Explicit SELECT columns](11-sql.md#02-explicit-select-columns)
3. [Filtering and NULL handling](11-sql.md#03-filtering-and-null-handling)
4. [Join cardinality](11-sql.md#04-join-cardinality)
5. [Aggregation and grouping](11-sql.md#05-aggregation-and-grouping)
6. [Index-friendly predicates](11-sql.md#06-index-friendly-predicates)
7. [Transactions for atomic workflows](11-sql.md#07-transactions-for-atomic-workflows)
8. [Migration backfills](11-sql.md#08-migration-backfills)
9. [Parameterized queries](11-sql.md#09-parameterized-queries)
10. [Stable pagination](11-sql.md#10-stable-pagination)

</details>

<details>
<summary><b>12. Python</b> — 10 lessons</summary>

1. [Naming and readability](12-python.md#01-naming-and-readability)
2. [Truthy, falsy, and None checks](12-python.md#02-truthy-falsy-and-none-checks)
3. [Mutable default arguments](12-python.md#03-mutable-default-arguments)
4. [List and dict comprehensions](12-python.md#04-list-and-dict-comprehensions)
5. [Exception boundaries](12-python.md#05-exception-boundaries)
6. [Context managers for files](12-python.md#06-context-managers-for-files)
7. [Dataclasses for data shapes](12-python.md#07-dataclasses-for-data-shapes)
8. [Type hints at boundaries](12-python.md#08-type-hints-at-boundaries)
9. [Dependency injection for testability](12-python.md#09-dependency-injection-for-testability)
10. [Async and await boundaries](12-python.md#10-async-and-await-boundaries)

</details>

<details>
<summary><b>13. PHP</b> — 10 lessons</summary>

1. [Strict types and return types](13-php.md#01-strict-types-and-return-types)
2. [Input validation and filtering](13-php.md#02-input-validation-and-filtering)
3. [Output escaping for XSS](13-php.md#03-output-escaping-for-xss)
4. [PDO prepared statements](13-php.md#04-pdo-prepared-statements)
5. [Password hashing and verification](13-php.md#05-password-hashing-and-verification)
6. [Session handling and cookies](13-php.md#06-session-handling-and-cookies)
7. [Error handling and exceptions](13-php.md#07-error-handling-and-exceptions)
8. [Autoloading and namespaces](13-php.md#08-autoloading-and-namespaces)
9. [Configuration and secrets](13-php.md#09-configuration-and-secrets)
10. [Separating logic from templates](13-php.md#10-separating-logic-from-templates)

</details>

<details>
<summary><b>14. Laravel</b> — 10 lessons</summary>

1. [Routing and controller boundaries](14-laravel.md#01-routing-and-controller-boundaries)
2. [Form request validation](14-laravel.md#02-form-request-validation)
3. [Middleware and auth boundaries](14-laravel.md#03-middleware-and-auth-boundaries)
4. [Service container bindings](14-laravel.md#04-service-container-bindings)
5. [Eloquent query scopes](14-laravel.md#05-eloquent-query-scopes)
6. [Migrations and schema constraints](14-laravel.md#06-migrations-and-schema-constraints)
7. [API resources and response shape](14-laravel.md#07-api-resources-and-response-shape)
8. [Policies and authorization](14-laravel.md#08-policies-and-authorization)
9. [Queues and database commits](14-laravel.md#09-queues-and-database-commits)
10. [Feature tests with database assertions](14-laravel.md#10-feature-tests-with-database-assertions)

</details>

<details>
<summary><b>15. Java</b> — 10 lessons</summary>

1. [Null handling and Optional boundaries](15-java.md#01-null-handling-and-optional-boundaries)
2. [Immutable data and defensive copies](15-java.md#02-immutable-data-and-defensive-copies)
3. [equals, hashCode, and value objects](15-java.md#03-equals-hashcode-and-value-objects)
4. [Exception handling boundaries](15-java.md#04-exception-handling-boundaries)
5. [try-with-resources cleanup](15-java.md#05-try-with-resources-cleanup)
6. [Generics and collection types](15-java.md#06-generics-and-collection-types)
7. [Streams vs readable loops](15-java.md#07-streams-vs-readable-loops)
8. [Concurrency and shared state](15-java.md#08-concurrency-and-shared-state)
9. [Records and DTO boundaries](15-java.md#09-records-and-dto-boundaries)
10. [Package and service boundaries](15-java.md#10-package-and-service-boundaries)

</details>

<details>
<summary><b>16. Kotlin</b> — 10 lessons</summary>

1. [Null safety boundaries](16-kotlin.md#01-null-safety-boundaries)
2. [Data class immutability](16-kotlin.md#02-data-class-immutability)
3. [Sealed result states](16-kotlin.md#03-sealed-result-states)
4. [Scope function intent](16-kotlin.md#04-scope-function-intent)
5. [Collection transformations](16-kotlin.md#05-collection-transformations)
6. [Coroutine scope lifecycle](16-kotlin.md#06-coroutine-scope-lifecycle)
7. [Flow state streams](16-kotlin.md#07-flow-state-streams)
8. [Resource use boundaries](16-kotlin.md#08-resource-use-boundaries)
9. [JVM interop platform types](16-kotlin.md#09-jvm-interop-platform-types)
10. [Kotest fixtures and assertions](16-kotlin.md#10-kotest-fixtures-and-assertions)

</details>

<details>
<summary><b>17. C</b> — 10 lessons</summary>

1. [Pointer ownership at boundaries](17-c.md#01-pointer-ownership-at-boundaries)
2. [Buffer bounds and formatted output](17-c.md#02-buffer-bounds-and-formatted-output)
3. [Allocation and free ownership](17-c.md#03-allocation-and-free-ownership)
4. [Const correctness for read paths](17-c.md#04-const-correctness-for-read-paths)
5. [Struct ownership and borrowed views](17-c.md#05-struct-ownership-and-borrowed-views)
6. [Error codes that name failures](17-c.md#06-error-codes-that-name-failures)
7. [File I/O cleanup paths](17-c.md#07-file-io-cleanup-paths)
8. [Integer overflow before allocation](17-c.md#08-integer-overflow-before-allocation)
9. [Compiler warning contracts](17-c.md#09-compiler-warning-contracts)
10. [Unit tests with boundary cases](17-c.md#10-unit-tests-with-boundary-cases)

</details>

<details>
<summary><b>18. C++</b> — 10 lessons</summary>

1. [RAII resource ownership](18-cpp.md#01-raii-resource-ownership)
2. [Smart pointer ownership](18-cpp.md#02-smart-pointer-ownership)
3. [Move semantics and lifetimes](18-cpp.md#03-move-semantics-and-lifetimes)
4. [Const references and string_view](18-cpp.md#04-const-references-and-string_view)
5. [Rule of zero](18-cpp.md#05-rule-of-zero)
6. [Exception boundaries](18-cpp.md#06-exception-boundaries)
7. [Template constraints](18-cpp.md#07-template-constraints)
8. [Algorithm and iterator intent](18-cpp.md#08-algorithm-and-iterator-intent)
9. [Scoped locks for shared state](18-cpp.md#09-scoped-locks-for-shared-state)
10. [Unit tests for edge cases](18-cpp.md#10-unit-tests-for-edge-cases)

</details>

<details>
<summary><b>19. C#</b> — 10 lessons</summary>

1. [Nullable reference boundaries](19-csharp.md#01-nullable-reference-boundaries)
2. [Records and immutable DTOs](19-csharp.md#02-records-and-immutable-dtos)
3. [Async await and cancellation](19-csharp.md#03-async-await-and-cancellation)
4. [Dependency injection boundaries](19-csharp.md#04-dependency-injection-boundaries)
5. [Using declarations and disposal](19-csharp.md#05-using-declarations-and-disposal)
6. [Exceptions and result boundaries](19-csharp.md#06-exceptions-and-result-boundaries)
7. [LINQ query intent](19-csharp.md#07-linq-query-intent)
8. [Pattern matching switches](19-csharp.md#08-pattern-matching-switches)
9. [Options configuration](19-csharp.md#09-options-configuration)
10. [Unit tests with arrange act assert](19-csharp.md#10-unit-tests-with-arrange-act-assert)

</details>

<details>
<summary><b>20. Rust</b> — 10 lessons</summary>

1. [Ownership and borrowing](20-rust.md#01-ownership-and-borrowing)
2. [Lifetimes for returned data](20-rust.md#02-lifetimes-for-returned-data)
3. [Result error boundaries](20-rust.md#03-result-error-boundaries)
4. [Option instead of unwrap](20-rust.md#04-option-instead-of-unwrap)
5. [Pattern matching exhaustiveness](20-rust.md#05-pattern-matching-exhaustiveness)
6. [Traits and generic contracts](20-rust.md#06-traits-and-generic-contracts)
7. [Iterator ownership](20-rust.md#07-iterator-ownership)
8. [Async Send and Sync boundaries](20-rust.md#08-async-send-and-sync-boundaries)
9. [Cargo feature dependencies](20-rust.md#09-cargo-feature-dependencies)
10. [Tests, fixtures, and assertions](20-rust.md#10-tests-fixtures-and-assertions)

</details>

<details>
<summary><b>21. Lua</b> — 10 lessons</summary>

1. [Tables as records and arrays](21-lua.md#01-tables-as-records-and-arrays)
2. [Nil boundaries and defaults](21-lua.md#02-nil-boundaries-and-defaults)
3. [Module return contracts](21-lua.md#03-module-return-contracts)
4. [Metatable operator boundaries](21-lua.md#04-metatable-operator-boundaries)
5. [Coroutine lifecycle](21-lua.md#05-coroutine-lifecycle)
6. [pcall error handling](21-lua.md#06-pcall-error-handling)
7. [Sandboxed execution](21-lua.md#07-sandboxed-execution)
8. [Embedded API boundaries](21-lua.md#08-embedded-api-boundaries)
9. [Configuration validation](21-lua.md#09-configuration-validation)
10. [Busted test cases](21-lua.md#10-busted-test-cases)

</details>

<details>
<summary><b>22. Git</b> — 10 lessons</summary>

1. [Status before work](22-git.md#01-status-before-work)
2. [Interactive staging hunks](22-git.md#02-interactive-staging-hunks)
3. [Atomic commits and messages](22-git.md#03-atomic-commits-and-messages)
4. [Branch naming and upstream](22-git.md#04-branch-naming-and-upstream)
5. [Fetch, pull, and rebase sync](22-git.md#05-fetch-pull-and-rebase-sync)
6. [Reviewing diffs before push](22-git.md#06-reviewing-diffs-before-push)
7. [Conflict resolution without overwriting](22-git.md#07-conflict-resolution-without-overwriting)
8. [Revert vs reset](22-git.md#08-revert-vs-reset)
9. [.gitignore and secret safety](22-git.md#09-gitignore-and-secret-safety)
10. [Tags and release points](22-git.md#10-tags-and-release-points)

</details>

<details>
<summary><b>23. FastAPI</b> — 10 lessons</summary>

1. [Path operation order](23-fastapi.md#01-path-operation-order)
2. [Request models and validation](23-fastapi.md#02-request-models-and-validation)
3. [Response models as contracts](23-fastapi.md#03-response-models-as-contracts)
4. [Dependencies as boundaries](23-fastapi.md#04-dependencies-as-boundaries)
5. [Auth with security dependencies](23-fastapi.md#05-auth-with-security-dependencies)
6. [Async handler boundaries](23-fastapi.md#06-async-handler-boundaries)
7. [HTTPException handling](23-fastapi.md#07-httpexception-handling)
8. [Database session lifecycle](23-fastapi.md#08-database-session-lifecycle)
9. [Testing with dependency overrides](23-fastapi.md#09-testing-with-dependency-overrides)
10. [App structure and routers](23-fastapi.md#10-app-structure-and-routers)

</details>

<details>
<summary><b>24. Django</b> — 10 lessons</summary>

1. [URL pattern order](24-django.md#01-url-pattern-order)
2. [Model fields and constraints](24-django.md#02-model-fields-and-constraints)
3. [QuerySets and lazy evaluation](24-django.md#03-querysets-and-lazy-evaluation)
4. [Model manager boundaries](24-django.md#04-model-manager-boundaries)
5. [Forms and validation](24-django.md#05-forms-and-validation)
6. [Class-based view responsibilities](24-django.md#06-class-based-view-responsibilities)
7. [CSRF and unsafe methods](24-django.md#07-csrf-and-unsafe-methods)
8. [Settings by environment](24-django.md#08-settings-by-environment)
9. [Transactions and side effects](24-django.md#09-transactions-and-side-effects)
10. [Test client and fixtures](24-django.md#10-test-client-and-fixtures)

</details>

<details>
<summary><b>25. Go</b> — 10 lessons</summary>

1. [Package and file organization](25-go.md#01-package-and-file-organization)
2. [Naming and exported APIs](25-go.md#02-naming-and-exported-apis)
3. [Error handling and wrapping](25-go.md#03-error-handling-and-wrapping)
4. [Context cancellation](25-go.md#04-context-cancellation)
5. [Interfaces at boundaries](25-go.md#05-interfaces-at-boundaries)
6. [Struct validation and zero values](25-go.md#06-struct-validation-and-zero-values)
7. [Goroutines and channel ownership](25-go.md#07-goroutines-and-channel-ownership)
8. [Defer and resource cleanup](25-go.md#08-defer-and-resource-cleanup)
9. [Table-driven tests](25-go.md#09-table-driven-tests)
10. [JSON HTTP handler boundaries](25-go.md#10-json-http-handler-boundaries)

</details>

<details>
<summary><b>26. Docker</b> — 10 lessons</summary>

1. [Build context and .dockerignore](26-docker.md#01-build-context-and-dockerignore)
2. [Base images and tag pinning](26-docker.md#02-base-images-and-tag-pinning)
3. [Layer cache and dependency order](26-docker.md#03-layer-cache-and-dependency-order)
4. [Multi-stage builds](26-docker.md#04-multi-stage-builds)
5. [COPY, ADD, and WORKDIR](26-docker.md#05-copy-add-and-workdir)
6. [Non-root users and permissions](26-docker.md#06-non-root-users-and-permissions)
7. [ARG, ENV, and secrets](26-docker.md#07-arg-env-and-secrets)
8. [ENTRYPOINT, CMD, and signals](26-docker.md#08-entrypoint-cmd-and-signals)
9. [Healthchecks and runtime config](26-docker.md#09-healthchecks-and-runtime-config)
10. [Compose services, volumes, and networks](26-docker.md#10-compose-services-volumes-and-networks)

</details>

<details>
<summary><b>27. Tailwind CSS</b> — 10 lessons</summary>

1. [Utility composition readability](27-tailwindcss.md#01-utility-composition-readability)
2. [Responsive breakpoints](27-tailwindcss.md#02-responsive-breakpoints)
3. [State variants: hover, focus, and disabled](27-tailwindcss.md#03-state-variants-hover-focus-and-disabled)
4. [Spacing and layout consistency](27-tailwindcss.md#04-spacing-and-layout-consistency)
5. [Theme tokens and custom values](27-tailwindcss.md#05-theme-tokens-and-custom-values)
6. [Dark mode variants](27-tailwindcss.md#06-dark-mode-variants)
7. [Class reuse and component boundaries](27-tailwindcss.md#07-class-reuse-and-component-boundaries)
8. [Arbitrary values with restraint](27-tailwindcss.md#08-arbitrary-values-with-restraint)
9. [Accessibility: contrast and focus](27-tailwindcss.md#09-accessibility-contrast-and-focus)
10. [Conditional classes and conflicts](27-tailwindcss.md#10-conditional-classes-and-conflicts)

</details>

---

Source: [https://good-code-bad-code.pages.dev](https://good-code-bad-code.pages.dev) · Generated from the live site content.
