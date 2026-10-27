# Node.js — Code Review Handbook

> Review runtime boundaries, async I/O, HTTP lifecycle, logging, and process safety.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/nodejs](https://good-code-bad-code.pages.dev/tracks/nodejs)
- **Related tracks:** [JavaScript](03-javascript.md), [TypeScript](04-typescript.md), [React](05-react.md), [Vue](06-vue.md), [Next.js](07-nextjs.md), [Express](09-express.md), [NestJS](10-nestjs.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Runtime boundaries and entry points](#01-runtime-boundaries-and-entry-points)** — A Node module should be importable without starting the process or opening sockets.
- [ ] **[Async filesystem without blocking](#02-async-filesystem-without-blocking)** — Avoid synchronous filesystem calls inside HTTP handlers and other hot paths.
- [ ] **[Async failure boundaries](#03-async-failure-boundaries)** — Every async request path should have a clear failure path and one response.
- [ ] **[Environment variables and configuration](#04-environment-variables-and-configuration)** — Config should fail fast with clear messages instead of scattering process.env reads.
- [ ] **[Module side effects and startup code](#05-module-side-effects-and-startup-code)** — Side effects should live behind explicit start functions or entry points.
- [ ] **[Streams for large payloads](#06-streams-for-large-payloads)** — For large payloads, prefer pipeline-based streaming with an explicit error path.
- [ ] **[HTTP request lifecycle](#07-http-request-lifecycle)** — A request handler should validate early, read bounded input, and return after sending a response.
- [ ] **[Graceful shutdown and process signals](#08-graceful-shutdown-and-process-signals)** — A production Node process should drain HTTP and dependency connections before exit.
- [ ] **[Structured logging](#09-structured-logging)** — Logs should help correlate behavior without leaking secrets or personal data.
- [ ] **[Input validation and path safety](#10-input-validation-and-path-safety)** — Resolve user-controlled paths against an allowed root and reject traversal attempts.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Runtime boundaries and entry points](#01-runtime-boundaries-and-entry-points) | `runtime` `startup` `architecture` |
| 02 | [Async filesystem without blocking](#02-async-filesystem-without-blocking) | `filesystem` `async` `performance` |
| 03 | [Async failure boundaries](#03-async-failure-boundaries) | `errors` `async` `http` |
| 04 | [Environment variables and configuration](#04-environment-variables-and-configuration) | `configuration` `environment` `startup` |
| 05 | [Module side effects and startup code](#05-module-side-effects-and-startup-code) | `modules` `side-effects` `startup` |
| 06 | [Streams for large payloads](#06-streams-for-large-payloads) | `streams` `memory` `http` |
| 07 | [HTTP request lifecycle](#07-http-request-lifecycle) | `http` `requests` `validation` |
| 08 | [Graceful shutdown and process signals](#08-graceful-shutdown-and-process-signals) | `process` `shutdown` `signals` |
| 09 | [Structured logging](#09-structured-logging) | `logging` `observability` `security` |
| 10 | [Input validation and path safety](#10-input-validation-and-path-safety) | `validation` `security` `filesystem` |

---

## 01. Runtime boundaries and entry points

> Keep process startup separate from reusable application wiring.

**Tags:** `runtime` `startup` `architecture` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/runtime-boundaries-entry-points)

### ✅ Good Code — `src/server.ts`

```ts
import { createApp } from "./app";
import { readConfig } from "./config";

const config = readConfig(process.env);
const app = createApp({ logger: console });

// Process startup is isolated to the entry point.
const server = app.listen(config.port, () => {
  console.info("Server listening on port " + config.port);
});

export { server };
```

### ❌ Bad Code — `src/app.ts`

```ts
import http from "node:http";
import { handleRequest } from "./handler";

const server = http.createServer(handleRequest);

// Importing this module immediately opens a socket.
server.listen(process.env.PORT || 3000);

export default server;
```

### 🔍 What to review

- **Good Code:** The good version keeps startup in one entry point and lets the app wiring be imported without immediately listening on a port.
- **Bad Code:** The bad version starts the server as a side effect of importing `app.ts`, which makes tests, scripts, and workers unexpectedly open sockets.

### 📝 Review notes

In review, ask what happens when this module is imported by a test or another process. The bad version is review-worthy because startup side effects make the runtime harder to compose and harder to shut down cleanly.

### 💡 Takeaway

> **A Node module should be importable without starting the process or opening sockets.**

---

## 02. Async filesystem without blocking

> Use async filesystem APIs in request paths so one slow disk read does not block the event loop.

**Tags:** `filesystem` `async` `performance` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/async-filesystem-without-blocking)

### ✅ Good Code — `src/routes/config.ts`

```ts
import { readFile } from "node:fs/promises";
import type { ServerResponse } from "node:http";

export async function sendPublicConfig(response: ServerResponse) {
  // Async I/O lets the event loop keep serving other requests.
  const file = await readFile("public-config.json", "utf8");

  response.setHeader("content-type", "application/json");
  response.end(file);
}
```

### ❌ Bad Code — `src/routes/config.ts`

```ts
import { readFileSync } from "node:fs";
import type { ServerResponse } from "node:http";

export function sendPublicConfig(response: ServerResponse) {
  // Sync I/O blocks the whole process during a request.
  const file = readFileSync("public-config.json", "utf8");

  response.setHeader("content-type", "application/json");
  response.end(file);
}
```

### 🔍 What to review

- **Good Code:** The good version awaits an async file read and lets the event loop continue accepting other request work while the disk is busy.
- **Bad Code:** The bad version uses `readFileSync` in a request path, blocking every other request in the process until the read finishes.

### 📝 Review notes

In review, synchronous filesystem APIs are usually acceptable during startup or build tooling, but suspicious in request handlers. The bad version is review-worthy because it can pass local testing while causing latency spikes under concurrent traffic.

### 💡 Takeaway

> **Avoid synchronous filesystem calls inside HTTP handlers and other hot paths.**

---

## 03. Async failure boundaries

> Catch rejected promises at the HTTP boundary where the route can log context and send one response.

**Tags:** `errors` `async` `http` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/async-failure-boundaries)

### ✅ Good Code — `src/routes/reports.ts`

```ts
import type { ServerResponse } from "node:http";
import { createReport } from "../services/reports";

export async function sendReport(response: ServerResponse) {
  // The HTTP boundary owns both success and failure responses.
  try {
    const report = await createReport();

    response.setHeader("content-type", "application/json");
    response.end(JSON.stringify({ report }));
  } catch (error) {
    console.error({ error }, "Could not create report");
    response.statusCode = 500;
    response.end(JSON.stringify({ error: "Could not create report." }));
  }
}
```

### ❌ Bad Code — `src/routes/reports.ts`

```ts
import type { ServerResponse } from "node:http";
import { createReport } from "../services/reports";

export function sendReport(response: ServerResponse) {
  // Rejection has no owner, so the client may never get a response.
  createReport().then((report) => {
    response.setHeader("content-type", "application/json");
    response.end(JSON.stringify({ report }));
  });
}
```

### 🔍 What to review

- **Good Code:** The good version catches async failures at the HTTP boundary, logs context, and returns a single error response.
- **Bad Code:** The bad version leaves the rejected promise without an owner, so the process can emit an unhandled rejection while the client waits or disconnects.

### 📝 Review notes

In review, follow every promise created in request code to find the rejection owner. The bad version is review-worthy because successful paths look clean while failure behavior is undefined.

### 💡 Takeaway

> **Every async request path should have a clear failure path and one response.**

---

## 04. Environment variables and configuration

> Read and validate environment configuration once at startup.

**Tags:** `configuration` `environment` `startup` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/environment-configuration)

### ✅ Good Code — `src/config.ts`

```ts
type AppConfig = {
  port: number;
  databaseUrl: string;
};

export function readConfig(env: NodeJS.ProcessEnv): AppConfig {
  const databaseUrl = env.DATABASE_URL;
  const port = Number(env.PORT ?? "3000");

  // Validate configuration before the app accepts traffic.
  if (!databaseUrl) {
    throw new Error("DATABASE_URL is required.");
  }

  if (!Number.isInteger(port) || port < 1) {
    throw new Error("PORT must be a positive integer.");
  }

  return { databaseUrl, port };
}
```

### ❌ Bad Code — `src/db.ts`

```ts
export function connectDatabase() {
  // Hidden defaults can point production at the wrong service.
  return connect({
    url: process.env.DATABASE_URL || "postgres://localhost/dev",
    poolSize: Number(process.env.POOL_SIZE || "not-a-number"),
  });
}
```

### 🔍 What to review

- **Good Code:** The good version turns environment variables into a typed config object and fails during startup if required values are missing.
- **Bad Code:** The bad version reads `process.env` deep inside infrastructure code, silently falls back to a local database, and accepts invalid numeric config.

### 📝 Review notes

In review, look for scattered `process.env` reads and defaults that could point production at the wrong service. The bad version is review-worthy because configuration bugs should fail loudly before the app starts serving traffic.

### 💡 Takeaway

> **Config should fail fast with clear messages instead of scattering process.env reads.**

---

## 05. Module side effects and startup code

> Avoid starting timers, connections, or jobs when a module is merely imported.

**Tags:** `modules` `side-effects` `startup` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/module-side-effects-startup)

### ✅ Good Code — `src/jobs/cleanup.ts`

```ts
type CleanupJob = {
  stop: () => void;
};

// Callers decide when the background job starts and stops.
export function startCleanupJob(runCleanup: () => Promise<void>): CleanupJob {
  const interval = setInterval(() => {
    runCleanup().catch((error) => {
      console.error({ error }, "Cleanup job failed");
    });
  }, 60_000);

  return {
    stop: () => clearInterval(interval),
  };
}
```

### ❌ Bad Code — `src/jobs/cleanup.ts`

```ts
import { deleteExpiredSessions } from "../sessions";

// Importing the helper also starts a permanent background loop.
setInterval(() => {
  deleteExpiredSessions();
}, 60_000);

export function cleanupNow() {
  return deleteExpiredSessions();
}
```

### 🔍 What to review

- **Good Code:** The good version makes startup explicit and returns a stop function for the background job.
- **Bad Code:** The bad version starts a timer at import time, so tests or scripts that only need `cleanupNow` also start an ongoing background loop.

### 📝 Review notes

In review, scan module top levels for timers, connections, subscriptions, and network calls. The bad version is review-worthy because implicit startup makes lifecycle control and test isolation much harder.

### 💡 Takeaway

> **Side effects should live behind explicit start functions or entry points.**

---

## 06. Streams for large payloads

> Stream large files or responses instead of loading everything into memory.

**Tags:** `streams` `memory` `http` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/streams-large-payloads)

### ✅ Good Code — `src/routes/download.ts`

```ts
import { createReadStream } from "node:fs";
import { pipeline } from "node:stream/promises";
import type { ServerResponse } from "node:http";

export async function sendExport(filePath: string, response: ServerResponse) {
  response.setHeader("content-type", "text/csv");

  try {
    // pipeline coordinates backpressure and stream errors.
    await pipeline(createReadStream(filePath), response);
  } catch (error) {
    console.error({ error, filePath }, "Export stream failed");
    if (!response.headersSent) {
      response.statusCode = 500;
      response.end("Could not stream export.");
    }
  }
}
```

### ❌ Bad Code — `src/routes/download.ts`

```ts
import { readFile } from "node:fs/promises";
import type { ServerResponse } from "node:http";

export async function sendExport(filePath: string, response: ServerResponse) {
  // Loading the whole export scales memory with file size.
  const file = await readFile(filePath);

  response.setHeader("content-type", "text/csv");
  response.end(file);
}
```

### 🔍 What to review

- **Good Code:** The good version streams the file through `pipeline`, which manages backpressure and surfaces stream failures.
- **Bad Code:** The bad version reads the entire file into memory before sending it, which can pressure memory or crash the process for large exports.

### 📝 Review notes

In review, ask how large the payload can become and whether multiple users can request it at once. The bad version is review-worthy because memory usage grows with payload size and concurrency.

### 💡 Takeaway

> **For large payloads, prefer pipeline-based streaming with an explicit error path.**

---

## 07. HTTP request lifecycle

> Order method checks, body limits, and response exits so invalid requests stop early.

**Tags:** `http` `requests` `validation` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/http-request-lifecycle)

### ✅ Good Code — `src/routes/webhook.ts`

```ts
import type { IncomingMessage, ServerResponse } from "node:http";

async function readBody(request: IncomingMessage, limitBytes: number) {
  let body = "";

  for await (const chunk of request) {
    body += chunk;
    if (Buffer.byteLength(body) > limitBytes) {
      throw new Error("Body too large.");
    }
  }

  return body;
}

export async function handleWebhook(
  request: IncomingMessage,
  response: ServerResponse,
) {
  // Reject unsupported methods before reading the body.
  if (request.method !== "POST") {
    response.statusCode = 405;
    response.end("Method not allowed.");
    return;
  }

  const body = await readBody(request, 64_000);
  response.end("Received " + body.length + " bytes.");
}
```

### ❌ Bad Code — `src/routes/webhook.ts`

```ts
import type { IncomingMessage, ServerResponse } from "node:http";

export async function handleWebhook(
  request: IncomingMessage,
  response: ServerResponse,
) {
  let body = "";

  // Unbounded reads happen before the method is even checked.
  for await (const chunk of request) {
    body += chunk;
  }

  if (request.method !== "POST") {
    response.statusCode = 405;
    response.end("Method not allowed.");
  }

  response.end("Received " + body.length + " bytes.");
}
```

### 🔍 What to review

- **Good Code:** The good version rejects unsupported methods before reading the body, enforces a body limit, and returns after sending an error.
- **Bad Code:** The bad version reads unlimited input before checking the method and can attempt to write a second response after a 405.

### 📝 Review notes

In review, trace the request from entry to response. The bad version is review-worthy because lifecycle bugs often appear only on invalid methods, large payloads, or clients that disconnect mid-request.

### 💡 Takeaway

> **A request handler should validate early, read bounded input, and return after sending a response.**

---

## 08. Graceful shutdown and process signals

> Stop accepting work and close resources when the process receives shutdown signals.

**Tags:** `process` `shutdown` `signals` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/graceful-shutdown-signals)

### ✅ Good Code — `src/shutdown.ts`

```ts
import type { Server } from "node:http";

export function installShutdown(server: Server, closeDatabase: () => Promise<void>) {
  async function shutdown(signal: string) {
    console.info("Received " + signal + ", shutting down");

    // Stop accepting new work before closing dependencies.
    server.close(async () => {
      await closeDatabase();
      process.exit(0);
    });

    setTimeout(() => {
      console.error("Shutdown timed out");
      process.exit(1);
    }, 10_000).unref();
  }

  process.once("SIGTERM", () => shutdown("SIGTERM"));
  process.once("SIGINT", () => shutdown("SIGINT"));
}
```

### ❌ Bad Code — `src/server.ts`

```ts
// Immediate exit can drop in-flight requests.
process.on("SIGTERM", () => {
  process.exit(0);
});

process.on("SIGINT", () => {
  process.exit(0);
});
```

### 🔍 What to review

- **Good Code:** The good version stops accepting new HTTP work, closes dependencies, and keeps a timeout in case shutdown hangs.
- **Bad Code:** The bad version exits immediately, which can drop in-flight requests and leave external work half-complete.

### 📝 Review notes

In review, ask what happens during deploys, container restarts, and local Ctrl+C. The bad version is review-worthy because the signal path controls production correctness, not just operational polish.

### 💡 Takeaway

> **A production Node process should drain HTTP and dependency connections before exit.**

---

## 09. Structured logging

> Log machine-readable events with request context and without sensitive data.

**Tags:** `logging` `observability` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/structured-logging)

### ✅ Good Code — `src/logging.ts`

```ts
type LogEvent = {
  level: "info" | "error";
  message: string;
  requestId: string;
  durationMs?: number;
};

export function logEvent(event: LogEvent) {
  // Stable fields make logs searchable and correlatable.
  console.log(JSON.stringify({
    time: new Date().toISOString(),
    ...event,
  }));
}
```

### ❌ Bad Code — `src/login.ts`

```ts
export function logLoginAttempt(email: string, password: string) {
  // Secrets in logs become production data leaks.
  console.log("Login failed for " + email + " with password " + password);
}
```

### 🔍 What to review

- **Good Code:** The good version emits structured JSON with stable fields that log systems can search and aggregate.
- **Bad Code:** The bad version logs a free-form string and includes a password, turning logs into a security incident.

### 📝 Review notes

In review, look for correlation IDs, event names, levels, and sensitive fields. The bad version is review-worthy because logs are production data and should be useful without becoming a place where secrets are copied.

### 💡 Takeaway

> **Logs should help correlate behavior without leaking secrets or personal data.**

---

## 10. Input validation and path safety

> Validate user input before using it in filesystem paths or other sensitive operations.

**Tags:** `validation` `security` `filesystem` · [View on site](https://good-code-bad-code.pages.dev/tracks/nodejs/input-validation-path-safety)

### ✅ Good Code — `src/files.ts`

```ts
import path from "node:path";

const uploadRoot = path.resolve("uploads");

export function resolveUploadPath(fileName: string) {
  // Validate and resolve user input against the allowed root.
  if (!/^[a-z0-9._-]+$/i.test(fileName)) {
    throw new Error("Invalid file name.");
  }

  const targetPath = path.resolve(uploadRoot, fileName);

  if (!targetPath.startsWith(uploadRoot + path.sep)) {
    throw new Error("Path escapes upload directory.");
  }

  return targetPath;
}
```

### ❌ Bad Code — `src/files.ts`

```ts
import path from "node:path";

export function resolveUploadPath(fileName: string) {
  // User input can traverse outside the intended directory.
  return path.join("uploads", fileName);
}
```

### 🔍 What to review

- **Good Code:** The good version validates the filename, resolves it against a known root, and rejects paths that escape the upload directory.
- **Bad Code:** The bad version joins user input directly into a filesystem path, allowing traversal input to point outside the intended folder.

### 📝 Review notes

In review, treat file paths, shell arguments, URLs, and database filters as sensitive boundaries. The bad version is review-worthy because a short helper can become the place where user input crosses into the operating system.

### 💡 Takeaway

> **Resolve user-controlled paths against an allowed root and reject traversal attempts.**

---

[← Back to index](README.md)
