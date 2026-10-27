# Docker — Code Review Handbook

> Review build contexts, base images, cache layers, multi-stage builds, secrets, users, healthchecks, and Compose files.

- **Category:** Data + Ops
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/docker](https://good-code-bad-code.pages.dev/tracks/docker)
- **Related tracks:** [SQL](11-sql.md), [Git](22-git.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Build context and .dockerignore](#01-build-context-and-dockerignore)** — The build context is part of the input to your image, so review what files are allowed into it.
- [ ] **[Base images and tag pinning](#02-base-images-and-tag-pinning)** — A base image is a dependency; review it with the same care as package versions.
- [ ] **[Layer cache and dependency order](#03-layer-cache-and-dependency-order)** — Docker cache follows instruction order, so dependency files should usually be copied before application source.
- [ ] **[Multi-stage builds](#04-multi-stage-builds)** — Multi-stage builds reduce final image size and separate build concerns from runtime concerns.
- [ ] **[COPY, ADD, and WORKDIR](#05-copy-add-and-workdir)** — Prefer explicit paths and explicit download or extraction steps over surprising Dockerfile side effects.
- [ ] **[Non-root users and permissions](#06-non-root-users-and-permissions)** — Containers should not need root at runtime unless the application has a specific reason.
- [ ] **[ARG, ENV, and secrets](#07-arg-env-and-secrets)** — Secrets should be supplied at runtime or through secret mounts, not stored in image layers.
- [ ] **[ENTRYPOINT, CMD, and signals](#08-entrypoint-cmd-and-signals)** — The process started by Docker should receive signals directly and shut down predictably.
- [ ] **[Healthchecks and runtime config](#09-healthchecks-and-runtime-config)** — A running container is not always a healthy application; expose a meaningful health signal.
- [ ] **[Compose services, volumes, and networks](#10-compose-services-volumes-and-networks)** — A Compose file should make service relationships explicit while keeping persistence and exposure deliberate.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Build context and .dockerignore](#01-build-context-and-dockerignore) | `build context` `dockerignore` `security` |
| 02 | [Base images and tag pinning](#02-base-images-and-tag-pinning) | `base image` `tags` `supply chain` |
| 03 | [Layer cache and dependency order](#03-layer-cache-and-dependency-order) | `cache` `layers` `dependencies` |
| 04 | [Multi-stage builds](#04-multi-stage-builds) | `multi-stage` `runtime` `image size` |
| 05 | [COPY, ADD, and WORKDIR](#05-copy-add-and-workdir) | `copy` `add` `workdir` |
| 06 | [Non-root users and permissions](#06-non-root-users-and-permissions) | `security` `user` `permissions` |
| 07 | [ARG, ENV, and secrets](#07-arg-env-and-secrets) | `secrets` `env` `arg` |
| 08 | [ENTRYPOINT, CMD, and signals](#08-entrypoint-cmd-and-signals) | `entrypoint` `cmd` `signals` |
| 09 | [Healthchecks and runtime config](#09-healthchecks-and-runtime-config) | `healthcheck` `runtime` `operations` |
| 10 | [Compose services, volumes, and networks](#10-compose-services-volumes-and-networks) | `compose` `volumes` `networks` |

---

## 01. Build context and .dockerignore

> Keep the Docker build context small and intentional so builds are faster, safer, and easier to reproduce.

**Tags:** `build context` `dockerignore` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/build-context-dockerignore)

### ✅ Good Code — `.dockerignore`

```gitignore
# Send only the files needed to build the image.
node_modules
.git
.env
.next
coverage
dist
*.log
# Keep local Docker and Compose files out unless the image needs them.
Dockerfile*
compose*.yaml
```

### ❌ Bad Code — `.dockerignore`

```gitignore
# No ignore rules means Docker sends the whole repository to the builder.
```

### 🔍 What to review

- **Good Code:** The good version excludes dependency folders, local build output, secrets, logs, and repository metadata from the build context.
- **Bad Code:** The bad version sends the whole repository to the builder. That makes cache invalidation noisy and increases the chance of copying sensitive or irrelevant files.

### 📝 Review notes

Start Docker reviews at the build context. If `.dockerignore` is missing or too small, a clean Dockerfile can still be slow, flaky, or unsafe.

### 💡 Takeaway

> **The build context is part of the input to your image, so review what files are allowed into it.**

---

## 02. Base images and tag pinning

> Choose a narrow base image and pin versions deliberately so rebuilds do not change underneath you without review.

**Tags:** `base image` `tags` `supply chain` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/base-images-and-tag-pinning)

### ✅ Good Code — `Dockerfile`

```dockerfile
# Pin Node and Alpine versions so rebuilds stay predictable.
FROM node:22.11.0-alpine3.20 AS runtime

WORKDIR /app
ENV NODE_ENV=production

COPY package.json package-lock.json ./
# Install only production dependencies for this runtime image.
RUN npm ci --omit=dev

COPY server.js ./server.js
CMD ["node", "server.js"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
# latest can move to a different Node or OS package set later.
FROM node:latest

WORKDIR /app
COPY . .
# npm install may change dependency resolution during image builds.
RUN npm install

CMD ["node", "server.js"]
```

### 🔍 What to review

- **Good Code:** The good version uses a specific runtime base and keeps production dependency installation explicit.
- **Bad Code:** The bad version depends on `latest`, so rebuilding the same commit can silently pull a different Node version and operating system packages.

### 📝 Review notes

Pinning does not mean never updating. It means updates become visible changes that CI, security scanners, and reviewers can evaluate.

### 💡 Takeaway

> **A base image is a dependency; review it with the same care as package versions.**

---

## 03. Layer cache and dependency order

> Order Dockerfile instructions so dependency layers are reused when source files change.

**Tags:** `cache` `layers` `dependencies` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/layer-cache-dependency-order)

### ✅ Good Code — `Dockerfile`

```dockerfile
FROM node:22-alpine AS app
WORKDIR /app

# Copy dependency files first so npm ci can stay cached.
COPY package.json package-lock.json ./
RUN npm ci

# Source changes happen after dependency install.
COPY src ./src
COPY public ./public

RUN npm run build
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM node:22-alpine AS app
WORKDIR /app

# Copying everything first makes any source edit break the install cache.
COPY . .
RUN npm install
RUN npm run build
```

### 🔍 What to review

- **Good Code:** The good version lets the dependency install layer stay cached when only source files change.
- **Bad Code:** The bad version copies the whole project before installing dependencies. Any source edit can invalidate the expensive install layer.

### 📝 Review notes

Cache-friendly Dockerfiles are also review-friendly. Look for the smallest stable inputs before expensive `RUN` steps.

### 💡 Takeaway

> **Docker cache follows instruction order, so dependency files should usually be copied before application source.**

---

## 04. Multi-stage builds

> Separate build tooling from the final runtime image so production containers contain only what they need to run.

**Tags:** `multi-stage` `runtime` `image size` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/multi-stage-builds)

### ✅ Good Code — `Dockerfile`

```dockerfile
# Build stages keep install and build tools out of the final image.
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM deps AS build
COPY . .
RUN npm run build

# The runtime stage receives only the files needed to run the app.
FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
CMD ["node", "server.js"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
# One stage ships source, dev dependencies, and build tools to production.
FROM node:22-alpine
WORKDIR /app

COPY . .
RUN npm install
RUN npm run build

CMD ["npm", "start"]
```

### 🔍 What to review

- **Good Code:** The good version keeps install and build tooling in earlier stages and copies only runtime artifacts into the final image.
- **Bad Code:** The bad version ships source files, dev dependencies, and build tools in the same image that runs production.

### 📝 Review notes

For compiled or bundled apps, ask what the final stage actually needs. Anything only needed to build should usually stay out of the runtime stage.

### 💡 Takeaway

> **Multi-stage builds reduce final image size and separate build concerns from runtime concerns.**

---

## 05. COPY, ADD, and WORKDIR

> Use WORKDIR and COPY intentionally so filesystem paths are clear and remote side effects are not hidden in ADD.

**Tags:** `copy` `add` `workdir` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/copy-add-and-workdir)

### ✅ Good Code — `Dockerfile`

```dockerfile
FROM python:3.12-slim

# WORKDIR gives every following path one clear base directory.
WORKDIR /app

# Copy known inputs before installing dependencies.
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY app ./app
CMD ["python", "-m", "app"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM python:latest

RUN mkdir app
# cd inside RUN does not set the working directory for later layers.
RUN cd app && pip install flask

# ADD can download or extract content, hiding extra behavior in one line.
ADD https://example.com/app.tar.gz /app
COPY . /
```

### 🔍 What to review

- **Good Code:** The good version uses an absolute `WORKDIR`, copies known files, and keeps installation inputs visible.
- **Bad Code:** The bad version relies on `cd` inside individual layers, fetches remote content through `ADD`, and copies the repository into the image root.

### 📝 Review notes

`COPY` should be the default for local files because it only copies bytes from the build context. Reach for `ADD` only when remote fetch or archive extraction is intentional and reviewed.

### 💡 Takeaway

> **Prefer explicit paths and explicit download or extraction steps over surprising Dockerfile side effects.**

---

## 06. Non-root users and permissions

> Run application processes as a non-root user and make file ownership match the runtime user.

**Tags:** `security` `user` `permissions` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/non-root-user-permissions)

### ✅ Good Code — `Dockerfile`

```dockerfile
FROM node:22-alpine AS runtime

WORKDIR /app
# Create a dedicated user for the app process.
RUN addgroup -S app && adduser -S app -G app

# chown makes copied files readable by the runtime user.
COPY --chown=app:app package.json package-lock.json ./
RUN npm ci --omit=dev
COPY --chown=app:app server.js ./server.js

# Switch away from root before the app starts.
USER app
CMD ["node", "server.js"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

WORKDIR /app
COPY . .
RUN npm install

# Without USER, the app runs as root by default.
CMD ["node", "server.js"]
```

### 🔍 What to review

- **Good Code:** The good version creates a dedicated user, assigns ownership during copy, and switches to that user before the process starts.
- **Bad Code:** The bad version runs as the default root user. A compromised process gets more privileges inside the container than it needs.

### 📝 Review notes

When adding `USER`, also review writable directories. A non-root container that cannot write its required cache, temp, or upload paths will fail at runtime.

### 💡 Takeaway

> **Containers should not need root at runtime unless the application has a specific reason.**

---

## 07. ARG, ENV, and secrets

> Separate build-time arguments, runtime environment variables, and secrets so sensitive values are not baked into images.

**Tags:** `secrets` `env` `arg` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/args-env-and-secrets)

### ✅ Good Code — `compose.yaml`

```yaml
services:
  app:
    build:
      context: .
      args:
        # Build args are for build-time choices, not secrets.
        NODE_VERSION: "22"
    environment:
      NODE_ENV: production
      DATABASE_URL_FILE: /run/secrets/database_url
    # Secrets are mounted at runtime instead of baked into the image.
    secrets:
      - database_url

secrets:
  database_url:
    file: ./secrets/database_url.txt
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

# Passing secrets through ARG can leave them in build history.
ARG DATABASE_URL
# ENV keeps the secret in the final image config.
ENV DATABASE_URL=$DATABASE_URL
ENV API_TOKEN=super-secret-token

COPY . .
RUN npm run build
CMD ["npm", "start"]
```

### 🔍 What to review

- **Good Code:** The good version treats build arguments, runtime environment, and secrets as different concerns.
- **Bad Code:** The bad version passes secrets through `ARG` and stores them in `ENV`, making sensitive values part of the image configuration or build history.

### 📝 Review notes

If a value is secret, assume image layers, logs, and metadata can expose it. Prefer Compose secrets, orchestrator secrets, or BuildKit secret mounts.

### 💡 Takeaway

> **Secrets should be supplied at runtime or through secret mounts, not stored in image layers.**

---

## 08. ENTRYPOINT, CMD, and signals

> Use exec-form commands and keep startup wrappers signal-safe so containers stop gracefully.

**Tags:** `entrypoint` `cmd` `signals` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/entrypoint-cmd-and-signals)

### ✅ Good Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY server.js ./server.js

# Exec form lets node receive Docker stop signals directly.
ENTRYPOINT ["node"]
CMD ["server.js"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

WORKDIR /app
COPY . .
RUN npm install

# Shell form wraps the app in a shell, making stop signals harder to handle.
CMD npm start
```

### 🔍 What to review

- **Good Code:** The good version uses exec form, so the application process receives termination signals directly.
- **Bad Code:** The bad version uses shell form. The shell can become PID 1 and make signal forwarding and graceful shutdown less predictable.

### 📝 Review notes

If an entrypoint script is needed, check that it ends with `exec "$@"`. Startup scripts should prepare the environment, then hand off to the real process.

### 💡 Takeaway

> **The process started by Docker should receive signals directly and shut down predictably.**

---

## 09. Healthchecks and runtime config

> Describe runtime health and configuration explicitly so operators can tell whether the container is ready and healthy.

**Tags:** `healthcheck` `runtime` `operations` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/healthchecks-and-runtime-config)

### ✅ Good Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

WORKDIR /app
ENV NODE_ENV=production
ENV PORT=3000

COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY server.js ./server.js

# Declare the port the app is expected to listen on.
EXPOSE 3000
# Check the app endpoint, not just whether the process exists.
HEALTHCHECK --interval=30s --timeout=3s --retries=3   CMD wget -qO- http://127.0.0.1:3000/health || exit 1

CMD ["node", "server.js"]
```

### ❌ Bad Code — `Dockerfile`

```dockerfile
FROM node:22-alpine

WORKDIR /app
COPY . .
RUN npm install

# EXPOSE only documents the intended port; it does not prove the app is healthy.
EXPOSE 3000
CMD ["node", "server.js"]
```

### 🔍 What to review

- **Good Code:** The good version declares runtime defaults, exposes the expected port, and provides a healthcheck tied to the app's health endpoint.
- **Bad Code:** The bad version can be running while the application is unable to serve requests. Operators get less signal when the HTTP server or a required dependency fails.

### 📝 Review notes

Healthchecks should be cheap and meaningful. Avoid checks that only prove the process exists if the real failure mode is a broken dependency or dead HTTP server.

### 💡 Takeaway

> **A running container is not always a healthy application; expose a meaningful health signal.**

---

## 10. Compose services, volumes, and networks

> Use Compose to define service boundaries, persistent volumes, and internal networks without overexposing ports or state.

**Tags:** `compose` `volumes` `networks` · [View on site](https://good-code-bad-code.pages.dev/tracks/docker/compose-services-volumes-networks)

### ✅ Good Code — `compose.yaml`

```yaml
services:
  app:
    build: .
    environment:
      DATABASE_HOST: db
    depends_on:
      db:
        condition: service_healthy
    # Only expose the web port the host needs.
    ports:
      - "3000:3000"
    networks:
      - internal

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: reviews
      POSTGRES_USER: reviews
    # Named volumes keep database data after container restarts.
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U reviews"]
      interval: 10s
      timeout: 5s
      retries: 5
    # Services talk to each other on a private network.
    networks:
      - internal

volumes:
  postgres_data:

networks:
  internal:
```

### ❌ Bad Code — `compose.yaml`

```yaml
services:
  app:
    build: .
    # Host networking removes the boundary between container and host.
    network_mode: host
    volumes:
      - .:/app
      # Mounting / exposes the host filesystem to the container.
      - /:/host

  db:
    image: postgres:latest
    # Opening the database port is often unnecessary for app-only access.
    ports:
      - "5432:5432"
```

### 🔍 What to review

- **Good Code:** The good version names persistence, keeps services on an internal network, and exposes only the application port needed by the host.
- **Bad Code:** The bad version uses host networking, mounts the host filesystem, and exposes the database without a clear reason.

### 📝 Review notes

Compose review is boundary review. Look at which services are reachable from the host, which data survives restarts, and which mounts can mutate the developer machine.

### 💡 Takeaway

> **A Compose file should make service relationships explicit while keeping persistence and exposure deliberate.**

---

[← Back to index](README.md)
