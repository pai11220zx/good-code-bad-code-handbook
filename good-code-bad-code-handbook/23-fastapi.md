# FastAPI — Code Review Handbook

> Review routes, Pydantic models, dependencies, auth, async handlers, database sessions, and API tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/fastapi](https://good-code-bad-code.pages.dev/tracks/fastapi)
- **Related tracks:** [Python](12-python.md), [Django](24-django.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Path operation order](#01-path-operation-order)** — FastAPI path operations are evaluated in order, so route order is behavior.
- [ ] **[Request models and validation](#02-request-models-and-validation)** — Request bodies should cross the API boundary as validated models, not raw dictionaries.
- [ ] **[Response models as contracts](#03-response-models-as-contracts)** — A response model is both documentation and a guardrail for data leaving the service.
- [ ] **[Dependencies as boundaries](#04-dependencies-as-boundaries)** — FastAPI dependencies make repeated request setup explicit, reusable, and visible in OpenAPI.
- [ ] **[Auth with security dependencies](#05-auth-with-security-dependencies)** — Auth should be a dependency boundary, not repeated header parsing in every route.
- [ ] **[Async handler boundaries](#06-async-handler-boundaries)** — An `async def` route should not hide blocking I/O inside the event loop.
- [ ] **[HTTPException handling](#07-httpexception-handling)** — Expected API failures should use explicit HTTP status codes and response shapes.
- [ ] **[Database session lifecycle](#08-database-session-lifecycle)** — Database sessions need request-scoped ownership, cleanup, and transaction intent.
- [ ] **[Testing with dependency overrides](#09-testing-with-dependency-overrides)** — FastAPI dependency overrides make integration-style API tests fast and focused.
- [ ] **[App structure and routers](#10-app-structure-and-routers)** — A FastAPI app should wire routers and middleware; feature behavior belongs in focused modules.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Path operation order](#01-path-operation-order) | `routing` `path parameters` `order` |
| 02 | [Request models and validation](#02-request-models-and-validation) | `pydantic` `validation` `request body` |
| 03 | [Response models as contracts](#03-response-models-as-contracts) | `response model` `contracts` `serialization` |
| 04 | [Dependencies as boundaries](#04-dependencies-as-boundaries) | `dependencies` `depends` `boundaries` |
| 05 | [Auth with security dependencies](#05-auth-with-security-dependencies) | `security` `auth` `dependencies` |
| 06 | [Async handler boundaries](#06-async-handler-boundaries) | `async` `await` `blocking` |
| 07 | [HTTPException handling](#07-httpexception-handling) | `errors` `http exception` `status codes` |
| 08 | [Database session lifecycle](#08-database-session-lifecycle) | `database` `session` `dependencies` |
| 09 | [Testing with dependency overrides](#09-testing-with-dependency-overrides) | `testing` `dependency overrides` `testclient` |
| 10 | [App structure and routers](#10-app-structure-and-routers) | `routers` `structure` `app factory` |

---

## 01. Path operation order

> Declare specific routes before dynamic path parameters so FastAPI matches requests as intended.

**Tags:** `routing` `path parameters` `order` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/path-operation-order)

### ✅ Good Code — `api/users.py`

```python
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["users"])


# Put fixed paths before dynamic parameters.
@router.get("/me")
async def read_current_user():
    return {"user_id": "current"}


@router.get("/{user_id}")
async def read_user(user_id: int):
    return {"user_id": user_id}
```

### ❌ Bad Code — `api/users.py`

```python
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["users"])


# Dynamic parameters above fixed paths can catch reserved words.
@router.get("/{user_id}")
async def read_user(user_id: int):
    return {"user_id": user_id}


@router.get("/me")
async def read_current_user():
    return {"user_id": "current"}
```

### 🔍 What to review

- **Good Code:** The good version declares the fixed `/me` route before the dynamic `/{user_id}` route.
- **Bad Code:** The bad version lets `/{user_id}` match `/me` first, so FastAPI tries to parse `"me"` as an integer and returns a validation error.

### 📝 Review notes

When reviewing FastAPI routing, read the file top to bottom. Specific routes, fixed words, and literal paths should usually appear before broader dynamic routes.

### 💡 Takeaway

> **FastAPI path operations are evaluated in order, so route order is behavior.**

---

## 02. Request models and validation

> Use Pydantic request models so FastAPI can parse, validate, document, and type the incoming body.

**Tags:** `pydantic` `validation` `request body` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/request-models-validation)

### ✅ Good Code — `api/reviews.py`

```python
from fastapi import APIRouter
from pydantic import BaseModel, Field

router = APIRouter()


class CreateReviewRequest(BaseModel):
    title: str = Field(min_length=3, max_length=120)
    author_id: int


# Pydantic validates the body before route logic runs.
@router.post("/reviews")
async def create_review(payload: CreateReviewRequest):
    return {
        "title": payload.title,
        "author_id": payload.author_id,
    }
```

### ❌ Bad Code — `api/reviews.py`

```python
from fastapi import APIRouter, Request

router = APIRouter()


@router.post("/reviews")
async def create_review(request: Request):
    # Manual JSON parsing moves validation into the handler.
    payload = await request.json()
    return {
        "title": payload["title"],
        "author_id": int(payload["author_id"]),
    }
```

### 🔍 What to review

- **Good Code:** The good version lets FastAPI and Pydantic validate the shape, types, and title length before business logic runs.
- **Bad Code:** The bad version pulls raw JSON out of the request and converts values manually. Missing keys, invalid types, and docs drift become route-handler problems.

### 📝 Review notes

In FastAPI, the function signature is the API contract. If a body is important enough to accept, it is usually important enough to model.

### 💡 Takeaway

> **Request bodies should cross the API boundary as validated models, not raw dictionaries.**

---

## 03. Response models as contracts

> Return response models that expose the public API shape and filter internal fields.

**Tags:** `response model` `contracts` `serialization` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/response-models-contracts)

### ✅ Good Code — `api/reviews.py`

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()


class ReviewResponse(BaseModel):
    id: int
    title: str
    status: str


# response_model keeps the public response shape explicit.
@router.get("/reviews/{review_id}", response_model=ReviewResponse)
async def get_review(review_id: int):
    review = await reviews.find(review_id)
    return review
```

### ❌ Bad Code — `api/reviews.py`

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/reviews/{review_id}")
async def get_review(review_id: int):
    review = await reviews.find(review_id)
    # __dict__ can leak internal fields into the API.
    return review.__dict__
```

### 🔍 What to review

- **Good Code:** The good version declares the public response shape and lets FastAPI serialize only the fields that belong in the contract.
- **Bad Code:** The bad version returns object internals. Private fields, database state, or unexpected attributes can leak into the API.

### 📝 Review notes

When reviewing FastAPI responses, ask what the client contract is. If the answer is "whatever the object currently has," the API is too accidental.

### 💡 Takeaway

> **A response model is both documentation and a guardrail for data leaving the service.**

---

## 04. Dependencies as boundaries

> Use Depends to share request parsing, context, and collaborators without hiding setup inside handlers.

**Tags:** `dependencies` `depends` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/dependencies-boundaries)

### ✅ Good Code — `api/reviews.py`

```python
from typing import Annotated

from fastapi import APIRouter, Depends, Query

router = APIRouter()


async def pagination(
    limit: Annotated[int, Query(le=100)] = 20,
    offset: int = 0,
):
    return {"limit": limit, "offset": offset}


# Depends keeps repeated query parsing reusable and documented.
@router.get("/reviews")
async def list_reviews(page: Annotated[dict, Depends(pagination)]):
    return await reviews.list(**page)
```

### ❌ Bad Code — `api/reviews.py`

```python
from fastapi import APIRouter, Request

router = APIRouter()


@router.get("/reviews")
async def list_reviews(request: Request):
    # Parsing query params by hand scatters defaults and validation.
    limit = int(request.query_params.get("limit", 20))
    offset = int(request.query_params.get("offset", 0))
    return await reviews.list(limit=limit, offset=offset)
```

### 🔍 What to review

- **Good Code:** The good version makes pagination a reusable dependency with validation and documentation.
- **Bad Code:** The bad version manually parses query parameters inside the route. Validation, defaults, and reuse are scattered across handlers.

### 📝 Review notes

Dependencies should clarify the boundary, not hide business logic. Look for repeated parsing or setup in routes and move that into a focused dependency.

### 💡 Takeaway

> **FastAPI dependencies make repeated request setup explicit, reusable, and visible in OpenAPI.**

---

## 05. Auth with security dependencies

> Use FastAPI security dependencies so auth is declared in the route contract and OpenAPI schema.

**Tags:** `security` `auth` `dependencies` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/auth-security-dependencies)

### ✅ Good Code — `api/auth.py`

```python
from typing import Annotated

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

# OAuth2PasswordBearer extracts the token and documents the auth scheme.
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


async def current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = await users.find_by_token(token)
    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid credentials",
        )
    return user
```

### ❌ Bad Code — `api/reviews.py`

```python
from fastapi import APIRouter, Request

router = APIRouter()


@router.post("/reviews/{review_id}/approve")
async def approve_review(review_id: int, request: Request):
    # Every route that parses headers itself can drift.
    header = request.headers.get("authorization", "")
    token = header.replace("Bearer ", "")
    user = await users.find_by_token(token)
    return await reviews.approve(review_id, user.id)
```

### 🔍 What to review

- **Good Code:** The good version centralizes token extraction and invalid credential handling in a security dependency.
- **Bad Code:** The bad version reparses the header in the route and assumes a user exists. Auth behavior drifts route by route and docs do not show the security scheme.

### 📝 Review notes

FastAPI can represent security dependencies in OpenAPI. In review, prefer one trusted auth boundary that routes depend on.

### 💡 Takeaway

> **Auth should be a dependency boundary, not repeated header parsing in every route.**

---

## 06. Async handler boundaries

> Keep async route handlers non-blocking and await asynchronous collaborators directly.

**Tags:** `async` `await` `blocking` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/async-handler-boundaries)

### ✅ Good Code — `api/reports.py`

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/reports/{report_id}")
async def get_report(report_id: int):
    # Await async I/O so the event loop can serve other requests.
    report = await report_service.fetch_report(report_id)
    audit = await audit_service.record_view(report_id)
    return {"report": report, "audit_id": audit.id}
```

### ❌ Bad Code — `api/reports.py`

```python
import requests
from fastapi import APIRouter

router = APIRouter()


@router.get("/reports/{report_id}")
async def get_report(report_id: int):
    # requests.get blocks the event loop inside an async route.
    response = requests.get(f"https://api.example.com/reports/{report_id}")
    # Missing await can leave side effects unfinished.
    audit_service.record_view(report_id)
    return response.json()
```

### 🔍 What to review

- **Good Code:** The good version awaits async collaborators so I/O yields control back to the event loop.
- **Bad Code:** The bad version calls a blocking HTTP client inside an async route and forgets to await the audit work.

### 📝 Review notes

FastAPI supports both `def` and `async def`. In review, check whether the handler and its collaborators agree about blocking versus non-blocking work.

### 💡 Takeaway

> **An `async def` route should not hide blocking I/O inside the event loop.**

---

## 07. HTTPException handling

> Raise HTTPException for expected client-facing failures instead of returning mixed success and error shapes.

**Tags:** `errors` `http exception` `status codes` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/http-exception-handling)

### ✅ Good Code — `api/reviews.py`

```python
from fastapi import APIRouter, HTTPException, status

router = APIRouter()


@router.get("/reviews/{review_id}")
async def get_review(review_id: int):
    review = await reviews.find(review_id)
    if review is None:
        # Use HTTP status codes for expected missing-resource cases.
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Review not found",
        )
    return review
```

### ❌ Bad Code — `api/reviews.py`

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/reviews/{review_id}")
async def get_review(review_id: int):
    review = await reviews.find(review_id)
    if review is None:
        # Returning 200 for an error makes clients trust the wrong status.
        return {"ok": False, "error": "Review not found"}
    return {"ok": True, "review": review}
```

### 🔍 What to review

- **Good Code:** The good version returns a real 404 with a clear detail when the resource does not exist.
- **Bad Code:** The bad version returns an error envelope with a successful HTTP status. Clients, logs, and monitoring can all misread the failure.

### 📝 Review notes

Use exceptions for expected HTTP failures at the route boundary. Keep success responses and error responses separate enough for clients to trust status codes.

### 💡 Takeaway

> **Expected API failures should use explicit HTTP status codes and response shapes.**

---

## 08. Database session lifecycle

> Provide database sessions through a dependency so each request gets a clear unit of work.

**Tags:** `database` `session` `dependencies` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/database-session-lifecycle)

### ✅ Good Code — `api/database.py`

```python
from typing import Annotated

from fastapi import Depends
from sqlmodel import Session


# Yield creates one session per request and closes it afterward.
def get_session():
    with Session(engine) as session:
        yield session


SessionDep = Annotated[Session, Depends(get_session)]


@router.post("/reviews")
def create_review(payload: ReviewCreate, session: SessionDep):
    review = Review.model_validate(payload)
    session.add(review)
    # The route makes the write and commit visible.
    session.commit()
    session.refresh(review)
    return review
```

### ❌ Bad Code — `api/reviews.py`

```python
from sqlmodel import Session

# A module-level session is shared across requests.
session = Session(engine)


@router.post("/reviews")
def create_review(payload: ReviewCreate):
    review = Review.model_validate(payload)
    session.add(review)
    session.commit()
    return review
```

### 🔍 What to review

- **Good Code:** The good version gives each request a session through a dependency and makes commit/refresh part of the route workflow.
- **Bad Code:** The bad version keeps a module-level session shared across requests. Cleanup, transaction boundaries, and concurrency behavior are unclear.

### 📝 Review notes

Database dependencies should have a visible lifecycle. In review, look for module-level sessions, hidden commits, and request handlers that cannot be tested with a replacement session.

### 💡 Takeaway

> **Database sessions need request-scoped ownership, cleanup, and transaction intent.**

---

## 09. Testing with dependency overrides

> Override dependencies in tests instead of calling external services or patching internals.

**Tags:** `testing` `dependency overrides` `testclient` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/testing-dependency-overrides)

### ✅ Good Code — `tests/test_reviews.py`

```python
from fastapi.testclient import TestClient

from app.main import app
from app.auth import current_user


def fake_current_user():
    return {"id": 1, "role": "reviewer"}


def test_approve_review():
    # Replace real auth with a fake user for this test.
    app.dependency_overrides[current_user] = fake_current_user
    client = TestClient(app)

    response = client.post("/reviews/10/approve")

    # Clear overrides so later tests do not inherit this setup.
    app.dependency_overrides = {}
    assert response.status_code == 200
```

### ❌ Bad Code — `tests/test_reviews.py`

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_approve_review():
    response = client.post(
        "/reviews/10/approve",
        # Production-like tokens make the test depend on real auth behavior.
        headers={"Authorization": "Bearer production-token"},
    )

    assert response.status_code == 200
```

### 🔍 What to review

- **Good Code:** The good version replaces the auth dependency with a test implementation and resets overrides after the assertion.
- **Bad Code:** The bad version depends on a production-like token and whatever auth service is behind it. Tests become slow, flaky, and hard to reason about.

### 📝 Review notes

FastAPI exposes `app.dependency_overrides` for tests. In review, prefer overriding dependencies over patching deep internals or calling real external services.

### 💡 Takeaway

> **FastAPI dependency overrides make integration-style API tests fast and focused.**

---

## 10. App structure and routers

> Organize feature routers and keep app creation focused on wiring rather than route implementation.

**Tags:** `routers` `structure` `app factory` · [View on site](https://good-code-bad-code.pages.dev/tracks/fastapi/app-structure-routers)

### ✅ Good Code — `app/main.py`

```python
from fastapi import FastAPI

from app.api import health, reviews, users


def create_app():
    app = FastAPI(title="Review API")

    # main.py wires routers; feature code stays in app.api modules.
    app.include_router(health.router)
    app.include_router(users.router, prefix="/users", tags=["users"])
    app.include_router(reviews.router, prefix="/reviews", tags=["reviews"])

    return app


app = create_app()
```

### ❌ Bad Code — `app/main.py`

```python
from fastapi import FastAPI

app = FastAPI()


# main.py now mixes app setup with feature handlers.
@app.get("/users/{user_id}")
async def read_user(user_id: int):
    return await users.find(user_id)


@app.post("/reviews/{review_id}/approve")
async def approve_review(review_id: int):
    review = await reviews.find(review_id)
    await notifications.send(review.author_email)
    return await reviews.approve(review_id)
```

### 🔍 What to review

- **Good Code:** The good version makes app creation a composition point and leaves feature routes in their own modules.
- **Bad Code:** The bad version turns `main.py` into a mix of app setup, route definitions, business flow, and side effects.

### 📝 Review notes

FastAPI makes it easy to start in one file. In review, watch for the moment where the file stops being an app entry point and becomes the whole application.

### 💡 Takeaway

> **A FastAPI app should wire routers and middleware; feature behavior belongs in focused modules.**

---

[← Back to index](README.md)
