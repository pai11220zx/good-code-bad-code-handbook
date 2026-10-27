# Code Review Checklist — All Tracks

เช็กลิสต์รวม Takeaway ทั้งหมด ใช้คัดลอกเฉพาะ track ที่โปรเจกต์ใช้ไปใส่ PR template

[← Back to index](README.md)

## HTML

- [ ] Start every page with language, charset, viewport, and a useful title. — [Document language and metadata](01-html.md#01-document-language-and-metadata)
- [ ] Prefer meaningful landmarks over anonymous wrappers. — [Semantic document structure](01-html.md#02-semantic-document-structure)
- [ ] Heading levels should outline the content in order, not act as font-size shortcuts. — [Heading hierarchy](01-html.md#03-heading-hierarchy)
- [ ] If it takes the user somewhere, make it an anchor with a meaningful destination and label. — [Links and navigation](01-html.md#04-links-and-navigation)
- [ ] Alt text should explain the image purpose in context, not repeat the file name. — [Images and alternative text](01-html.md#05-images-and-alternative-text)
- [ ] Use a real label so the input keeps its name before, during, and after editing. — [Accessible form labels](01-html.md#06-accessible-form-labels)
- [ ] Use aria-describedby and aria-invalid to keep field guidance tied to the input. — [Form help text and errors](01-html.md#07-form-help-text-and-errors)
- [ ] Reach for a native button when the user is triggering an action. — [Interactive elements](01-html.md#08-interactive-elements)
- [ ] When content is rows and columns of data, prefer table, caption, th, and scope. — [Tables for tabular data](01-html.md#09-tables-for-tabular-data)
- [ ] Prefer native semantics; ARIA should clarify, not paper over the wrong element. — [ARIA restraint](01-html.md#10-aria-restraint)

## CSS

- [ ] Prefer border-box sizing so declared component width includes padding and borders. — [Box model sizing](02-css.md#01-box-model-sizing)
- [ ] Prefer component classes and modifiers over deep selectors, IDs, and repeated !important fixes. — [Cascade and specificity](02-css.md#02-cascade-and-specificity)
- [ ] Prefer local flex gaps over selectors that push margins onto every child. — [Flex layout boundaries](02-css.md#03-flex-layout-boundaries)
- [ ] Let grid define track rules with minmax and gaps instead of distributing width through floats or nth-child margins. — [Grid layout tracks](02-css.md#04-grid-layout-tracks)
- [ ] Use bounded responsive values so scaling cannot make text too small or too large. — [Responsive units](02-css.md#05-responsive-units)
- [ ] Pair state changes with contrast and focus treatment that users can perceive. — [Color contrast states](02-css.md#06-color-contrast-states)
- [ ] Use :focus-visible for keyboard focus and avoid removing outlines without a visible replacement. — [State selectors and focus-visible](02-css.md#07-state-selectors-and-focus-visible)
- [ ] Prefer inline and block logical properties when left/right/top/bottom are not the real intent. — [Logical properties](02-css.md#08-logical-properties)
- [ ] Name reusable color, spacing, radius, and component accents with custom properties. — [Custom properties as design tokens](02-css.md#09-custom-properties-as-design-tokens)
- [ ] Pair animated UI with a prefers-reduced-motion path that keeps the state change clear. — [Motion and reduced motion](02-css.md#10-motion-and-reduced-motion)

## JavaScript

- [ ] Use strict equality for comparisons and nullish coalescing when only null or undefined should fall back. — [Strict equality and nullish checks](03-javascript.md#01-strict-equality-and-nullish-checks)
- [ ] Let the common path stay shallow by returning early for invalid inputs. — [Guard clauses](03-javascript.md#02-guard-clauses)
- [ ] Validate API, storage, and message payloads at the boundary before using nested fields. — [Validating data boundaries](03-javascript.md#03-validating-data-boundaries)
- [ ] Make mutation visible in the function name or return a new value instead. — [Naming side effects](03-javascript.md#04-naming-side-effects)
- [ ] Prefer filter, map, and reduce when they describe the transformation without mutating input data. — [Array transformations](03-javascript.md#05-array-transformations)
- [ ] Check both network failures and response status before trusting fetched data. — [Async error handling](03-javascript.md#06-async-error-handling)
- [ ] Use Promise.all for independent required work and reserve sequential awaits for true dependencies. — [Promise concurrency](03-javascript.md#07-promise-concurrency)
- [ ] Keep a stable handler reference and return a cleanup function when attaching listeners. — [Event listener cleanup](03-javascript.md#08-event-listener-cleanup)
- [ ] Expose small functions from modules and pass dependencies in when code needs storage, network, or browser APIs. — [Module boundaries and globals](03-javascript.md#09-module-boundaries-and-globals)
- [ ] Use Intl date formatting and pass the intended time zone when the displayed day matters. — [Dates and time zones](03-javascript.md#10-dates-and-time-zones)

## TypeScript

- [ ] Annotate function inputs, return contracts, and exported APIs while letting local implementation details infer naturally. — [Type inference and explicit boundaries](04-typescript.md#01-type-inference-and-explicit-boundaries)
- [ ] Narrow unknown input to make invalid states harder to express after validation. — [Narrowing unknown](04-typescript.md#02-narrowing-unknown)
- [ ] Treat `as` as a last resort at trusted boundaries, not a way to skip validation. — [Avoiding unsafe assertions](04-typescript.md#03-avoiding-unsafe-assertions)
- [ ] Use discriminated unions to make invalid states harder to express. — [Discriminated unions](04-typescript.md#04-discriminated-unions)
- [ ] Use a `never` assignment in the final branch to make missing union cases a type error. — [Exhaustive checks with never](04-typescript.md#05-exhaustive-checks-with-never)
- [ ] Use precise function signatures to make invalid calls harder to express. — [Precise function types](04-typescript.md#06-precise-function-types)
- [ ] Use `?` when a property may be omitted and `null` when the property is present but intentionally empty. — [Optional properties vs nullable values](04-typescript.md#07-optional-properties-vs-nullable-values)
- [ ] Use `extends` to keep generic functions flexible without losing the properties they rely on. — [Generic constraints](04-typescript.md#08-generic-constraints)
- [ ] Use Pick, Omit, Partial, and Readonly to express boundary-specific contracts from existing domain types. — [Utility types at API boundaries](04-typescript.md#09-utility-types-at-api-boundaries)
- [ ] Use `import type` when a module only needs a type and keep runtime imports for values that execute. — [Type-only imports and module boundaries](04-typescript.md#10-type-only-imports-and-module-boundaries)

## React

- [ ] Prefer explicit props for presentational components over hidden app-state reach-ins. — [Props component boundaries](05-react.md#01-props-component-boundaries)
- [ ] Prefer durable IDs for list keys instead of array positions. — [Stable keys in lists](05-react.md#02-stable-keys-in-lists)
- [ ] Prefer render-time derived values when state can be calculated from existing data. — [Derived state](05-react.md#03-derived-state)
- [ ] When state depends on prior state, pass an updater function to the setter. — [Updating state from previous state](05-react.md#04-updating-state-from-previous-state)
- [ ] Controlled inputs make the displayed value and submitted value come from one source. — [Controlled form inputs](05-react.md#05-controlled-form-inputs)
- [ ] Complete dependency arrays make effects follow the data they read. — [Effect dependencies](05-react.md#06-effect-dependencies)
- [ ] Clean up async effects when inputs change or the component unmounts. — [Async effect cleanup](05-react.md#07-async-effect-cleanup)
- [ ] Keep context narrow so components do not subscribe to unrelated app state. — [Context boundaries](05-react.md#08-context-boundaries)
- [ ] Memoization should have a clear cost to avoid and a complete dependency list. — [Memoization when it helps](05-react.md#09-memoization-when-it-helps)
- [ ] Use children and named slots when a component has multiple content variants. — [Composition over prop flags](05-react.md#10-composition-over-prop-flags)

## Vue

- [ ] A Vue SFC should show what it receives, what it renders, and which events it can send back. — [Single-file component boundaries](06-vue.md#01-single-file-component-boundaries)
- [ ] Props describe input from the parent; emits describe output back to the parent. — [Props and emits contracts](06-vue.md#02-props-and-emits-contracts)
- [ ] If a value can be derived from reactive inputs, prefer computed over a watcher that copies state. — [Computed values versus watchers](06-vue.md#03-computed-values-versus-watchers)
- [ ] A v-for key should identify the data item, not its current position in the array. — [List rendering keys](06-vue.md#04-list-rendering-keys)
- [ ] A form component should own draft input state and send a committed value back through an event. — [Form v-model boundaries](06-vue.md#05-form-v-model-boundaries)
- [ ] A composable should return the state and actions one caller needs, without sharing mutable module state by accident. — [Composables and state ownership](06-vue.md#06-composables-and-state-ownership)
- [ ] Async Vue components need visible loading and error paths, not only a happy-path ref assignment. — [Async state, loading, and errors](06-vue.md#07-async-state-loading-and-errors)
- [ ] A Pinia store should expose state, derived getters, and named actions instead of letting components mutate shared state from any file. — [Pinia store actions](06-vue.md#08-pinia-store-actions)
- [ ] Injected values should have a small contract, a typed key, and a provider close to the feature that owns the context. — [Provide and inject boundaries](06-vue.md#09-provide-and-inject-boundaries)
- [ ] Slots let the parent own variable content while the child owns shared structure and accessibility. — [Slots for component composition](06-vue.md#10-slots-for-component-composition)

## Next.js

- [ ] Routes should be expressed with folders and special files, not invented naming schemes. — [App Router file conventions](07-nextjs.md#01-app-router-file-conventions)
- [ ] Layouts should hold persistent UI while pages own the leaf route content. — [Layout and page boundaries](07-nextjs.md#02-layout-and-page-boundaries)
- [ ] Do not move a whole route to the client just because one child needs state. — [Server vs Client Components](07-nextjs.md#03-server-vs-client-components)
- [ ] A tiny client island keeps static UI and server data out of the browser bundle. — [Small client component islands](07-nextjs.md#04-small-client-component-islands)
- [ ] Server Components can fetch data close to the source without exposing secrets to the client. — [Server-side data fetching](07-nextjs.md#05-server-side-data-fetching)
- [ ] Independent server requests should be initiated together, then awaited together. — [Parallel data fetching](07-nextjs.md#06-parallel-data-fetching)
- [ ] Use loading and error files to keep navigation responsive and failures contained. — [Loading and error boundaries](07-nextjs.md#07-loading-and-error-boundaries)
- [ ] Cache data with an explicit lifetime or tag when shared content can be reused. — [Caching and revalidation intent](07-nextjs.md#08-caching-and-revalidation-intent)
- [ ] Route metadata should live in metadata exports or generateMetadata. — [Metadata per route](07-nextjs.md#09-metadata-per-route)
- [ ] Route handlers should validate requests and return explicit HTTP responses. — [Route handlers as API boundaries](07-nextjs.md#10-route-handlers-as-api-boundaries)

## Node.js

- [ ] A Node module should be importable without starting the process or opening sockets. — [Runtime boundaries and entry points](08-nodejs.md#01-runtime-boundaries-and-entry-points)
- [ ] Avoid synchronous filesystem calls inside HTTP handlers and other hot paths. — [Async filesystem without blocking](08-nodejs.md#02-async-filesystem-without-blocking)
- [ ] Every async request path should have a clear failure path and one response. — [Async failure boundaries](08-nodejs.md#03-async-failure-boundaries)
- [ ] Config should fail fast with clear messages instead of scattering process.env reads. — [Environment variables and configuration](08-nodejs.md#04-environment-variables-and-configuration)
- [ ] Side effects should live behind explicit start functions or entry points. — [Module side effects and startup code](08-nodejs.md#05-module-side-effects-and-startup-code)
- [ ] For large payloads, prefer pipeline-based streaming with an explicit error path. — [Streams for large payloads](08-nodejs.md#06-streams-for-large-payloads)
- [ ] A request handler should validate early, read bounded input, and return after sending a response. — [HTTP request lifecycle](08-nodejs.md#07-http-request-lifecycle)
- [ ] A production Node process should drain HTTP and dependency connections before exit. — [Graceful shutdown and process signals](08-nodejs.md#08-graceful-shutdown-and-process-signals)
- [ ] Logs should help correlate behavior without leaking secrets or personal data. — [Structured logging](08-nodejs.md#09-structured-logging)
- [ ] Resolve user-controlled paths against an allowed root and reject traversal attempts. — [Input validation and path safety](08-nodejs.md#10-input-validation-and-path-safety)

## Express

- [ ] An Express app should be buildable for tests without opening a port. — [App factory vs server startup](09-express.md#01-app-factory-vs-server-startup)
- [ ] Routers should own one resource area and receive services instead of reaching into globals. — [Router boundaries](09-express.md#02-router-boundaries)
- [ ] Express middleware order is behavior, not decoration. — [Middleware order](09-express.md#03-middleware-order)
- [ ] Route handlers should pass validated values to services, not raw request bodies. — [Request validation](09-express.md#04-request-validation)
- [ ] Async routes should either be awaited by Express or explicitly forwarded to next. — [Async route error forwarding](09-express.md#05-async-route-error-forwarding)
- [ ] A central error handler keeps status codes, logging, and response bodies consistent. — [Central error handler](09-express.md#06-central-error-handler)
- [ ] Clients should not need route-specific parsing rules for common API responses. — [Response shape consistency](09-express.md#07-response-shape-consistency)
- [ ] Routes should consume trusted auth context instead of reparsing credentials. — [Auth middleware boundaries](09-express.md#08-auth-middleware-boundaries)
- [ ] Rate limiting should use a deliberate client identity, especially behind proxies. — [Rate limiting and trust proxy](09-express.md#09-rate-limiting-and-trust-proxy)
- [ ] Routes should translate HTTP to application calls, not contain the whole use case. — [Avoiding business logic in routes](09-express.md#10-avoiding-business-logic-in-routes)

## NestJS

- [ ] Controllers should translate HTTP requests while providers own application work and modules wire the feature together. — [Module, controller, and provider boundaries](10-nestjs.md#01-module-controller-and-provider-boundaries)
- [ ] Constructor injection makes dependencies visible and gives tests a stable place to replace infrastructure. — [Dependency injection providers](10-nestjs.md#02-dependency-injection-providers)
- [ ] A controller should receive data that already passed DTO validation and transformation rules. — [DTO validation pipes](10-nestjs.md#03-dto-validation-pipes)
- [ ] Authorization checks belong before the route body, not halfway through controller logic. — [Guards for auth and authorization](10-nestjs.md#04-guards-for-auth-and-authorization)
- [ ] Exception filters should translate failures into consistent responses without leaking internal details. — [Exception filters and error shape](10-nestjs.md#05-exception-filters-and-error-shape)
- [ ] Interceptors should wrap route execution without forcing every controller method to repeat the same timing or response code. — [Interceptors for response and logging work](10-nestjs.md#06-interceptors-for-response-and-logging-work)
- [ ] Configuration should be parsed once at startup and injected as typed values instead of read ad hoc from process.env. — [Configuration module validation](10-nestjs.md#07-configuration-module-validation)
- [ ] Services should express use cases while repositories express database queries and persistence details. — [Repository and service boundaries](10-nestjs.md#08-repository-and-service-boundaries)
- [ ] Middleware is for low-level request work; guards decide access before pipes and controller logic run. — [Request lifecycle: middleware and guards](10-nestjs.md#09-request-lifecycle-middleware-and-guards)
- [ ] A test should replace external providers at the module boundary instead of reaching into service internals. — [Testing module overrides](10-nestjs.md#10-testing-module-overrides)

## SQL

- [ ] A schema should enforce the data rules the application depends on. — [Schema keys and constraints](11-sql.md#01-schema-keys-and-constraints)
- [ ] A query result is an API contract, so keep its shape intentional. — [Explicit SELECT columns](11-sql.md#02-explicit-select-columns)
- [ ] NULL is not equal to anything, so review nullable filters with extra care. — [Filtering and NULL handling](11-sql.md#03-filtering-and-null-handling)
- [ ] A join changes row count, so review the relationship before reviewing the selected columns. — [Join cardinality](11-sql.md#04-join-cardinality)
- [ ] Every non-aggregated selected value should be part of the grouping intent. — [Aggregation and grouping](11-sql.md#05-aggregation-and-grouping)
- [ ] A correct query can still be review-worthy if it prevents useful index access. — [Index-friendly predicates](11-sql.md#06-index-friendly-predicates)
- [ ] If one business action writes multiple rows, review where the transaction begins and ends. — [Transactions for atomic workflows](11-sql.md#07-transactions-for-atomic-workflows)
- [ ] A safe migration respects the data that already exists. — [Migration backfills](11-sql.md#08-migration-backfills)
- [ ] User input should become query data, not query text. — [Parameterized queries](11-sql.md#09-parameterized-queries)
- [ ] Pagination needs a stable order, not just a limit. — [Stable pagination](11-sql.md#10-stable-pagination)

## Python

- [ ] Readable Python starts with names that describe the domain, not the implementation shortcut. — [Naming and readability](12-python.md#01-naming-and-readability)
- [ ] Use `is None` when absence is different from an empty value. — [Truthy, falsy, and None checks](12-python.md#02-truthy-falsy-and-none-checks)
- [ ] Default arguments are evaluated once, so mutable defaults can leak state between calls. — [Mutable default arguments](12-python.md#03-mutable-default-arguments)
- [ ] A comprehension should make simple filtering and mapping easier to read, not denser to parse. — [List and dict comprehensions](12-python.md#04-list-and-dict-comprehensions)
- [ ] Specific exceptions make failure modes reviewable; blanket catches hide the bug you need to see. — [Exception boundaries](12-python.md#05-exception-boundaries)
- [ ] Resource lifetime should be visible in the code shape, not hidden in manual cleanup. — [Context managers for files](12-python.md#06-context-managers-for-files)
- [ ] A named data shape makes fields, defaults, and equality behavior easier to review. — [Dataclasses for data shapes](12-python.md#07-dataclasses-for-data-shapes)
- [ ] Type hints are most valuable at module boundaries, not as decoration on obvious local variables. — [Type hints at boundaries](12-python.md#08-type-hints-at-boundaries)
- [ ] Injected dependencies make side effects explicit and keep tests focused on behavior. — [Dependency injection for testability](12-python.md#09-dependency-injection-for-testability)
- [ ] Calling an async function creates work to await; it does not complete the work by itself. — [Async and await boundaries](12-python.md#10-async-and-await-boundaries)

## PHP

- [ ] A PHP file that owns business rules should make parameter and return expectations visible at the function boundary. — [Strict types and return types](13-php.md#01-strict-types-and-return-types)
- [ ] Superglobals are untrusted boundaries; normalize them into explicit values before using them. — [Input validation and filtering](13-php.md#02-input-validation-and-filtering)
- [ ] Prepared statements protect SQL, but HTML output still needs context-aware escaping. — [Output escaping for XSS](13-php.md#03-output-escaping-for-xss)
- [ ] SQL structure and SQL values should stay separate; PDO placeholders make that boundary reviewable. — [PDO prepared statements](13-php.md#04-pdo-prepared-statements)
- [ ] Passwords should go through password_hash and password_verify, never fast general-purpose hashes. — [Password hashing and verification](13-php.md#05-password-hashing-and-verification)
- [ ] Session code should make cookie flags and identity transitions visible to reviewers. — [Session handling and cookies](13-php.md#06-session-handling-and-cookies)
- [ ] Good PHP error handling separates developer diagnostics from user-facing responses. — [Error handling and exceptions](13-php.md#07-error-handling-and-exceptions)
- [ ] Modern PHP code should make class ownership visible through namespaces instead of require chains. — [Autoloading and namespaces](13-php.md#08-autoloading-and-namespaces)
- [ ] Secrets should be runtime configuration, not strings committed next to application logic. — [Configuration and secrets](13-php.md#09-configuration-and-secrets)
- [ ] Templates should render a prepared view model, not query databases or decide application flow. — [Separating logic from templates](13-php.md#10-separating-logic-from-templates)

## Laravel

- [ ] Routes should describe how HTTP enters the app while controllers and services own the work behind that route. — [Routing and controller boundaries](14-laravel.md#01-routing-and-controller-boundaries)
- [ ] A controller action should work with validated input instead of reading unchecked request payloads directly. — [Form request validation](14-laravel.md#02-form-request-validation)
- [ ] Access and request-wide rules should be visible where the route is declared so protected endpoints are easy to audit. — [Middleware and auth boundaries](14-laravel.md#03-middleware-and-auth-boundaries)
- [ ] Application code should depend on contracts or services that can be replaced in tests and configured at provider boundaries. — [Service container bindings](14-laravel.md#04-service-container-bindings)
- [ ] Use database queries for filtering and eager loading so request handlers do not pull more rows than they need. — [Eloquent query scopes](14-laravel.md#05-eloquent-query-scopes)
- [ ] Important data rules should be enforced by the database, not only remembered by controllers or jobs. — [Migrations and schema constraints](14-laravel.md#06-migrations-and-schema-constraints)
- [ ] Responses should expose a deliberate shape instead of leaking every model field and relationship. — [API resources and response shape](14-laravel.md#07-api-resources-and-response-shape)
- [ ] Authorization should read as a named rule at the controller boundary instead of scattered ownership checks. — [Policies and authorization](14-laravel.md#08-policies-and-authorization)
- [ ] A queued job should not observe data that the current database transaction has not committed yet. — [Queues and database commits](14-laravel.md#09-queues-and-database-commits)
- [ ] A useful feature test proves both the HTTP result and the persistent behavior users depend on. — [Feature tests with database assertions](14-laravel.md#10-feature-tests-with-database-assertions)

## Java

- [ ] A Java method should make missing data visible in its return type or exception path, not hide it behind a nullable value. — [Null handling and Optional boundaries](15-java.md#01-null-handling-and-optional-boundaries)
- [ ] Objects that represent a completed decision should not change because a caller still holds a mutable collection reference. — [Immutable data and defensive copies](15-java.md#02-immutable-data-and-defensive-copies)
- [ ] If a type can be used as a map key or set member, equality and hash behavior must be deliberate and consistent. — [equals, hashCode, and value objects](15-java.md#03-equals-hashcode-and-value-objects)
- [ ] Exceptions should carry useful meaning inside the system and become user-facing responses only at a clear boundary. — [Exception handling boundaries](15-java.md#04-exception-handling-boundaries)
- [ ] Any Java object that must be closed should have an obvious owner and a cleanup path that survives exceptions. — [try-with-resources cleanup](15-java.md#05-try-with-resources-cleanup)
- [ ] A Java collection type should tell reviewers what it contains and what key shape it expects. — [Generics and collection types](15-java.md#06-generics-and-collection-types)
- [ ] The best Java iteration shape is the one that makes intent and failure points easiest to review. — [Streams vs readable loops](15-java.md#07-streams-vs-readable-loops)
- [ ] If multiple threads can touch a value, the code should show which primitive owns synchronization. — [Concurrency and shared state](15-java.md#08-concurrency-and-shared-state)
- [ ] Boundary data should cross the application as a named shape with validation close to the edge. — [Records and DTO boundaries](15-java.md#09-records-and-dto-boundaries)
- [ ] A Java class is easier to review when HTTP, business rules, and persistence are separated by clear package boundaries. — [Package and service boundaries](15-java.md#10-package-and-service-boundaries)

## Kotlin

- [ ] Kotlin review should make nullable contracts visible where request or Java data enters the code. — [Null safety boundaries](16-kotlin.md#01-null-safety-boundaries)
- [ ] Kotlin data classes should protect value state from caller mutation after construction. — [Data class immutability](16-kotlin.md#02-data-class-immutability)
- [ ] A sealed result lets reviewers see every branch the caller must render or log. — [Sealed result states](16-kotlin.md#03-sealed-result-states)
- [ ] Kotlin scope functions should reveal whether code transforms a value, configures an object, or records a side effect. — [Scope function intent](16-kotlin.md#04-scope-function-intent)
- [ ] Kotlin collection review should connect `filter`, `map`, and `mapNotNull` to the domain rule being applied. — [Collection transformations](16-kotlin.md#05-collection-transformations)
- [ ] Kotlin coroutine review should check which scope owns each child job and how cancellation reaches it. — [Coroutine scope lifecycle](16-kotlin.md#06-coroutine-scope-lifecycle)
- [ ] Kotlin Flow review should separate stream transformation from the lifecycle that collects it. — [Flow state streams](16-kotlin.md#07-flow-state-streams)
- [ ] Kotlin resource review should pair every opened stream with `use` or another explicit close path. — [Resource use boundaries](16-kotlin.md#08-resource-use-boundaries)
- [ ] Kotlin review should add a boundary check when Java APIs return values without nullability metadata. — [JVM interop platform types](16-kotlin.md#09-jvm-interop-platform-types)
- [ ] Kotlin tests should assert the result shape that callers depend on, not only that a function returned a value. — [Kotest fixtures and assertions](16-kotlin.md#10-kotest-fixtures-and-assertions)

## C

- [ ] A C function that writes through a pointer should say who owns the storage and how large that storage is. — [Pointer ownership at boundaries](17-c.md#01-pointer-ownership-at-boundaries)
- [ ] A bounded write still needs a return-value check, because truncation can be a data bug rather than a safe result. — [Buffer bounds and formatted output](17-c.md#02-buffer-bounds-and-formatted-output)
- [ ] Heap ownership should appear as a contract: the code that creates memory must name the code that releases it. — [Allocation and free ownership](17-c.md#03-allocation-and-free-ownership)
- [ ] Const is part of the API contract in C; it separates read-only review paths from functions that may change data. — [Const correctness for read paths](17-c.md#04-const-correctness-for-read-paths)
- [ ] A struct that borrows memory should say so through const pointers and lengths, while stack storage must never escape its function. — [Struct ownership and borrowed views](17-c.md#05-struct-ownership-and-borrowed-views)
- [ ] A C error code is useful when it preserves the reason a caller needs for logging, retrying, or showing a message. — [Error codes that name failures](17-c.md#06-error-codes-that-name-failures)
- [ ] File cleanup should be visible on success and failure paths, especially when the function can return before the bottom. — [File I/O cleanup paths](17-c.md#07-file-io-cleanup-paths)
- [ ] Allocation math must be checked before `malloc`, because overflow can turn a large request into a small allocation. — [Integer overflow before allocation](17-c.md#08-integer-overflow-before-allocation)
- [ ] Build assumptions belong near the code they protect, because reviewers cannot infer warning policy from a function body alone. — [Compiler warning contracts](17-c.md#09-compiler-warning-contracts)
- [ ] A C test should fail by itself when the code breaks, without asking a reviewer to read terminal output. — [Unit tests with boundary cases](17-c.md#10-unit-tests-with-boundary-cases)

## C++

- [ ] C++ resource code should tie acquisition to an object lifetime, so exits through return or exception still release the resource. — [RAII resource ownership](18-cpp.md#01-raii-resource-ownership)
- [ ] A C++ API should show ownership in its type: `unique_ptr` for one owner, `shared_ptr` for shared lifetime, and raw pointers for observation. — [Smart pointer ownership](18-cpp.md#02-smart-pointer-ownership)
- [ ] Reference members are lifetime contracts; prefer owned values when the object needs data after the caller returns. — [Move semantics and lifetimes](18-cpp.md#03-move-semantics-and-lifetimes)
- [ ] Read-only C++ parameters should say whether they borrow data, copy data, or mutate data. — [Const references and string_view](18-cpp.md#04-const-references-and-string_view)
- [ ] If a class does not own raw resources directly, it usually should not define a destructor, copy constructor, or move constructor. — [Rule of zero](18-cpp.md#05-rule-of-zero)
- [ ] Exceptions should travel with meaning until a boundary can translate them into a status, response, or log record. — [Exception boundaries](18-cpp.md#06-exception-boundaries)
- [ ] Template code should name the operations it needs, because unconstrained templates fail far from the call site. — [Template constraints](18-cpp.md#07-template-constraints)
- [ ] A standard algorithm can make a range operation read as a single reviewable action instead of a custom loop. — [Algorithm and iterator intent](18-cpp.md#08-algorithm-and-iterator-intent)
- [ ] Mutex code should make the lock lifetime visible in scope, not depend on matching manual lock and unlock calls. — [Scoped locks for shared state](18-cpp.md#09-scoped-locks-for-shared-state)
- [ ] A C++ test should encode the boundary expectation in assertions so CI can fail without human log reading. — [Unit tests for edge cases](18-cpp.md#10-unit-tests-for-edge-cases)

## C#

- [ ] A C# method should make absence visible in its return type so callers choose the missing-data response at the boundary. — [Nullable reference boundaries](19-csharp.md#01-nullable-reference-boundaries)
- [ ] DTOs are easier to review when construction and mutation are separate: map once, then pass a stable value. — [Records and immutable DTOs](19-csharp.md#02-records-and-immutable-dtos)
- [ ] Async C# code should carry `CancellationToken` through every operation that can wait on I/O. — [Async await and cancellation](19-csharp.md#03-async-await-and-cancellation)
- [ ] A C# service should reveal the collaborators it needs instead of constructing database, clock, or HTTP clients inside methods. — [Dependency injection boundaries](19-csharp.md#04-dependency-injection-boundaries)
- [ ] Disposable resources should have a scope-owned lifetime so exceptions and early returns still release them. — [Using declarations and disposal](19-csharp.md#05-using-declarations-and-disposal)
- [ ] C# exception handling should preserve the failure reason until a boundary can map it to HTTP, UI, or logging. — [Exceptions and result boundaries](19-csharp.md#06-exceptions-and-result-boundaries)
- [ ] LINQ works best when each step names a collection transformation, not when the query hides mutation or logging. — [LINQ query intent](19-csharp.md#07-linq-query-intent)
- [ ] A C# switch expression should show each known state and make unknown states visible instead of silently falling through. — [Pattern matching switches](19-csharp.md#08-pattern-matching-switches)
- [ ] Configuration should cross into services as typed values that were validated during startup. — [Options configuration](19-csharp.md#09-options-configuration)
- [ ] A C# test should name the behavior, create only the data it needs, execute one action, and assert the observable result. — [Unit tests with arrange act assert](19-csharp.md#10-unit-tests-with-arrange-act-assert)

## Rust

- [ ] A Rust function should take ownership only when it stores, transforms, or consumes the value. — [Ownership and borrowing](20-rust.md#01-ownership-and-borrowing)
- [ ] A returned reference should point to data owned outside the function, with a lifetime that makes that relationship visible. — [Lifetimes for returned data](20-rust.md#02-lifetimes-for-returned-data)
- [ ] Rust boundary code should return typed errors instead of panicking on expected failure paths. — [Result error boundaries](20-rust.md#03-result-error-boundaries)
- [ ] A Rust lookup that can miss should keep the miss path visible instead of unwrapping inside helper code. — [Option instead of unwrap](20-rust.md#04-option-instead-of-unwrap)
- [ ] Rust enum reviews should prefer exhaustive matching over string flags or default branches that hide new states. — [Pattern matching exhaustiveness](20-rust.md#05-pattern-matching-exhaustiveness)
- [ ] A Rust generic should declare the smallest trait contract needed by the function body. — [Traits and generic contracts](20-rust.md#06-traits-and-generic-contracts)
- [ ] A Rust loop should make collection ownership visible through the iterator method it chooses. — [Iterator ownership](20-rust.md#07-iterator-ownership)
- [ ] Rust async review should check which values cross await points and task boundaries. — [Async Send and Sync boundaries](20-rust.md#08-async-send-and-sync-boundaries)
- [ ] A Rust crate should make optional integrations explicit through Cargo features and guarded code paths. — [Cargo feature dependencies](20-rust.md#09-cargo-feature-dependencies)
- [ ] A Rust test should name the input fixture and assert the observable result that protects the review rule. — [Tests, fixtures, and assertions](20-rust.md#10-tests-fixtures-and-assertions)

## Lua

- [ ] Lua table review should name whether a table is a record, array, map, or mixed structure. — [Tables as records and arrays](21-lua.md#01-tables-as-records-and-arrays)
- [ ] Lua code should choose defaults at input boundaries instead of scattering nil checks through business logic. — [Nil boundaries and defaults](21-lua.md#02-nil-boundaries-and-defaults)
- [ ] A Lua module should return the functions it wants callers to use and avoid mutating global state during import. — [Module return contracts](21-lua.md#03-module-return-contracts)
- [ ] Metatable review should check whether metamethods keep domain operations visible or hide side effects. — [Metatable operator boundaries](21-lua.md#04-metatable-operator-boundaries)
- [ ] Lua coroutine code should inspect resume success and status before assuming a task advanced. — [Coroutine lifecycle](21-lua.md#05-coroutine-lifecycle)
- [ ] Lua host code should catch plugin failures at the boundary and return a result the caller can classify. — [pcall error handling](21-lua.md#06-pcall-error-handling)
- [ ] Dynamic Lua execution should run with a small explicit environment instead of the full global table. — [Sandboxed execution](21-lua.md#07-sandboxed-execution)
- [ ] An embedded Lua API should expose narrow commands rather than raw host objects with broad mutation power. — [Embedded API boundaries](21-lua.md#08-embedded-api-boundaries)
- [ ] Lua config review should check required fields and value types at load time. — [Configuration validation](21-lua.md#09-configuration-validation)
- [ ] Lua tests should name the behavior, build a small fixture, and assert the exact output or error shape. — [Busted test cases](21-lua.md#10-busted-test-cases)

## Git

- [ ] A good Git workflow begins by making the current branch and uncommitted changes visible before editing files. — [Status before work](22-git.md#01-status-before-work)
- [ ] Reviewable commits come from reviewable staging, not from adding every file by habit. — [Interactive staging hunks](22-git.md#02-interactive-staging-hunks)
- [ ] A commit should describe one coherent change and make the reason visible without reading the whole diff first. — [Atomic commits and messages](22-git.md#03-atomic-commits-and-messages)
- [ ] A branch should tell the team what kind of work it contains and where it syncs by default. — [Branch naming and upstream](22-git.md#04-branch-naming-and-upstream)
- [ ] Sync commands should show what changed upstream before they rewrite or merge local history. — [Fetch, pull, and rebase sync](22-git.md#05-fetch-pull-and-rebase-sync)
- [ ] A push should come after a local review pass, not be the first time anyone sees the final diff. — [Reviewing diffs before push](22-git.md#06-reviewing-diffs-before-push)
- [ ] A conflict is a design review moment; choosing ours or theirs blindly can delete another person's intent. — [Conflict resolution without overwriting](22-git.md#07-conflict-resolution-without-overwriting)
- [ ] History commands should match whether the commits are already shared with other people. — [Revert vs reset](22-git.md#08-revert-vs-reset)
- [ ] A repository should make accidental secret and build artifact commits hard to create and easy to catch. — [.gitignore and secret safety](22-git.md#09-gitignore-and-secret-safety)
- [ ] A release tag should point to a verified commit and carry a clear versioned meaning. — [Tags and release points](22-git.md#10-tags-and-release-points)

## FastAPI

- [ ] FastAPI path operations are evaluated in order, so route order is behavior. — [Path operation order](23-fastapi.md#01-path-operation-order)
- [ ] Request bodies should cross the API boundary as validated models, not raw dictionaries. — [Request models and validation](23-fastapi.md#02-request-models-and-validation)
- [ ] A response model is both documentation and a guardrail for data leaving the service. — [Response models as contracts](23-fastapi.md#03-response-models-as-contracts)
- [ ] FastAPI dependencies make repeated request setup explicit, reusable, and visible in OpenAPI. — [Dependencies as boundaries](23-fastapi.md#04-dependencies-as-boundaries)
- [ ] Auth should be a dependency boundary, not repeated header parsing in every route. — [Auth with security dependencies](23-fastapi.md#05-auth-with-security-dependencies)
- [ ] An `async def` route should not hide blocking I/O inside the event loop. — [Async handler boundaries](23-fastapi.md#06-async-handler-boundaries)
- [ ] Expected API failures should use explicit HTTP status codes and response shapes. — [HTTPException handling](23-fastapi.md#07-httpexception-handling)
- [ ] Database sessions need request-scoped ownership, cleanup, and transaction intent. — [Database session lifecycle](23-fastapi.md#08-database-session-lifecycle)
- [ ] FastAPI dependency overrides make integration-style API tests fast and focused. — [Testing with dependency overrides](23-fastapi.md#09-testing-with-dependency-overrides)
- [ ] A FastAPI app should wire routers and middleware; feature behavior belongs in focused modules. — [App structure and routers](23-fastapi.md#10-app-structure-and-routers)

## Django

- [ ] Django checks URL patterns in order, so route order is application behavior. — [URL pattern order](24-django.md#01-url-pattern-order)
- [ ] A Django model should express business invariants, not just provide loose columns. — [Model fields and constraints](24-django.md#02-model-fields-and-constraints)
- [ ] QuerySets are lazy; review where they are built, sliced, and finally evaluated. — [QuerySets and lazy evaluation](24-django.md#03-querysets-and-lazy-evaluation)
- [ ] Repeated filters are a sign that a query rule needs a named boundary. — [Model manager boundaries](24-django.md#04-model-manager-boundaries)
- [ ] Request data should become cleaned data before it changes the database. — [Forms and validation](24-django.md#05-forms-and-validation)
- [ ] A class-based view should clarify the request flow, not become a dumping ground for every concern. — [Class-based view responsibilities](24-django.md#06-class-based-view-responsibilities)
- [ ] Unsafe methods such as POST should keep CSRF protection unless there is a deliberate API boundary. — [CSRF and unsafe methods](24-django.md#07-csrf-and-unsafe-methods)
- [ ] Django settings are operational code; review them with security and deployment in mind. — [Settings by environment](24-django.md#08-settings-by-environment)
- [ ] Database changes and external notifications need an explicit ordering contract. — [Transactions and side effects](24-django.md#09-transactions-and-side-effects)
- [ ] A useful Django view test proves the request behavior, not just that some text appeared. — [Test client and fixtures](24-django.md#10-test-client-and-fixtures)

## Go

- [ ] A Go package is an API boundary; review whether its name and exported surface make that boundary clear. — [Package and file organization](25-go.md#01-package-and-file-organization)
- [ ] Exported names are part of the package contract, so they should be minimal, clear, and caller-oriented. — [Naming and exported APIs](25-go.md#02-naming-and-exported-apis)
- [ ] Errors are values in Go; review both the message for humans and the wrapping contract for callers. — [Error handling and wrapping](25-go.md#03-error-handling-and-wrapping)
- [ ] Context belongs on work that can block, wait, or leave the process. — [Context cancellation](25-go.md#04-context-cancellation)
- [ ] In Go, interfaces are most useful when they describe what the caller needs right now. — [Interfaces at boundaries](25-go.md#05-interfaces-at-boundaries)
- [ ] A zero value can be a feature, but domain objects still need clear validity rules. — [Struct validation and zero values](25-go.md#06-struct-validation-and-zero-values)
- [ ] The code that starts a goroutine should make its stop condition and channel ownership visible. — [Goroutines and channel ownership](25-go.md#07-goroutines-and-channel-ownership)
- [ ] Cleanup should sit close to acquisition so reviewers can see every path releases the resource. — [Defer and resource cleanup](25-go.md#08-defer-and-resource-cleanup)
- [ ] A test table should improve coverage and readability, not hide which case failed. — [Table-driven tests](25-go.md#09-table-driven-tests)
- [ ] A Go HTTP handler should make request parsing and response writing explicit at the edge. — [JSON HTTP handler boundaries](25-go.md#10-json-http-handler-boundaries)

## Docker

- [ ] The build context is part of the input to your image, so review what files are allowed into it. — [Build context and .dockerignore](26-docker.md#01-build-context-and-dockerignore)
- [ ] A base image is a dependency; review it with the same care as package versions. — [Base images and tag pinning](26-docker.md#02-base-images-and-tag-pinning)
- [ ] Docker cache follows instruction order, so dependency files should usually be copied before application source. — [Layer cache and dependency order](26-docker.md#03-layer-cache-and-dependency-order)
- [ ] Multi-stage builds reduce final image size and separate build concerns from runtime concerns. — [Multi-stage builds](26-docker.md#04-multi-stage-builds)
- [ ] Prefer explicit paths and explicit download or extraction steps over surprising Dockerfile side effects. — [COPY, ADD, and WORKDIR](26-docker.md#05-copy-add-and-workdir)
- [ ] Containers should not need root at runtime unless the application has a specific reason. — [Non-root users and permissions](26-docker.md#06-non-root-users-and-permissions)
- [ ] Secrets should be supplied at runtime or through secret mounts, not stored in image layers. — [ARG, ENV, and secrets](26-docker.md#07-arg-env-and-secrets)
- [ ] The process started by Docker should receive signals directly and shut down predictably. — [ENTRYPOINT, CMD, and signals](26-docker.md#08-entrypoint-cmd-and-signals)
- [ ] A running container is not always a healthy application; expose a meaningful health signal. — [Healthchecks and runtime config](26-docker.md#09-healthchecks-and-runtime-config)
- [ ] A Compose file should make service relationships explicit while keeping persistence and exposure deliberate. — [Compose services, volumes, and networks](26-docker.md#10-compose-services-volumes-and-networks)

## Tailwind CSS

- [ ] A utility-first class list still needs structure; readable grouping makes design intent reviewable. — [Utility composition readability](27-tailwindcss.md#01-utility-composition-readability)
- [ ] Responsive utilities should describe how a small-screen layout grows, not how a large layout is squeezed down. — [Responsive breakpoints](27-tailwindcss.md#02-responsive-breakpoints)
- [ ] Buttons and links should expose hover, focus-visible, and disabled states with the same care as their default style. — [State variants: hover, focus, and disabled](27-tailwindcss.md#03-state-variants-hover-focus-and-disabled)
- [ ] Repeated spacing utilities should reveal a rhythm; random values usually reveal local visual patching. — [Spacing and layout consistency](27-tailwindcss.md#04-spacing-and-layout-consistency)
- [ ] A repeated color, spacing, or radius should usually become a token that reviewers can discuss by name. — [Theme tokens and custom values](27-tailwindcss.md#05-theme-tokens-and-custom-values)
- [ ] Dark mode should cover background, text, border, and state colors, not only the most visible foreground color. — [Dark mode variants](27-tailwindcss.md#06-dark-mode-variants)
- [ ] Repeated Tailwind strings are still duplication; extract them when they represent the same UI concept. — [Class reuse and component boundaries](27-tailwindcss.md#07-class-reuse-and-component-boundaries)
- [ ] Arbitrary values are a precision tool; overuse makes the design system invisible to reviewers. — [Arbitrary values with restraint](27-tailwindcss.md#08-arbitrary-values-with-restraint)
- [ ] Utility classes can make accessibility explicit when contrast, focus, and labels are part of the class list. — [Accessibility: contrast and focus](27-tailwindcss.md#09-accessibility-contrast-and-focus)
- [ ] Conditional Tailwind should make variant precedence obvious; conflicting utilities turn state review into guesswork. — [Conditional classes and conflicts](27-tailwindcss.md#10-conditional-classes-and-conflicts)
