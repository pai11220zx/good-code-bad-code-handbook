# Django — Code Review Handbook

> Review URL patterns, models, QuerySets, forms, views, CSRF, settings, transactions, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/django](https://good-code-bad-code.pages.dev/tracks/django)
- **Related tracks:** [Python](12-python.md), [FastAPI](23-fastapi.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[URL pattern order](#01-url-pattern-order)** — Django checks URL patterns in order, so route order is application behavior.
- [ ] **[Model fields and constraints](#02-model-fields-and-constraints)** — A Django model should express business invariants, not just provide loose columns.
- [ ] **[QuerySets and lazy evaluation](#03-querysets-and-lazy-evaluation)** — QuerySets are lazy; review where they are built, sliced, and finally evaluated.
- [ ] **[Model manager boundaries](#04-model-manager-boundaries)** — Repeated filters are a sign that a query rule needs a named boundary.
- [ ] **[Forms and validation](#05-forms-and-validation)** — Request data should become cleaned data before it changes the database.
- [ ] **[Class-based view responsibilities](#06-class-based-view-responsibilities)** — A class-based view should clarify the request flow, not become a dumping ground for every concern.
- [ ] **[CSRF and unsafe methods](#07-csrf-and-unsafe-methods)** — Unsafe methods such as POST should keep CSRF protection unless there is a deliberate API boundary.
- [ ] **[Settings by environment](#08-settings-by-environment)** — Django settings are operational code; review them with security and deployment in mind.
- [ ] **[Transactions and side effects](#09-transactions-and-side-effects)** — Database changes and external notifications need an explicit ordering contract.
- [ ] **[Test client and fixtures](#10-test-client-and-fixtures)** — A useful Django view test proves the request behavior, not just that some text appeared.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [URL pattern order](#01-url-pattern-order) | `urls` `routing` `order` |
| 02 | [Model fields and constraints](#02-model-fields-and-constraints) | `models` `constraints` `fields` |
| 03 | [QuerySets and lazy evaluation](#03-querysets-and-lazy-evaluation) | `querysets` `orm` `performance` |
| 04 | [Model manager boundaries](#04-model-manager-boundaries) | `models` `managers` `querysets` |
| 05 | [Forms and validation](#05-forms-and-validation) | `forms` `validation` `views` |
| 06 | [Class-based view responsibilities](#06-class-based-view-responsibilities) | `views` `cbv` `responsibility` |
| 07 | [CSRF and unsafe methods](#07-csrf-and-unsafe-methods) | `csrf` `security` `forms` |
| 08 | [Settings by environment](#08-settings-by-environment) | `settings` `security` `deployment` |
| 09 | [Transactions and side effects](#09-transactions-and-side-effects) | `transactions` `atomic` `side effects` |
| 10 | [Test client and fixtures](#10-test-client-and-fixtures) | `tests` `client` `fixtures` |

---

## 01. URL pattern order

> Put specific URL patterns before broad dynamic patterns so Django routes requests to the intended view.

**Tags:** `urls` `routing` `order` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/url-pattern-order)

### ✅ Good Code — `reviews/urls.py`

```python
from django.urls import path

from . import views

app_name = "reviews"

urlpatterns = [
    # Put reserved paths before broad slug captures.
    path("settings/", views.review_settings, name="settings"),
    path("<slug:username>/", views.reviewer_profile, name="profile"),
]
```

### ❌ Bad Code — `reviews/urls.py`

```python
from django.urls import path

from . import views

app_name = "reviews"

urlpatterns = [
    # Broad slug captures can swallow reserved paths.
    path("<slug:username>/", views.reviewer_profile, name="profile"),
    path("settings/", views.review_settings, name="settings"),
]
```

### 🔍 What to review

- **Good Code:** The good version makes the fixed settings route win before the broad profile route can capture the same path segment.
- **Bad Code:** The bad version makes `settings/` look like a username. The view might run with the wrong assumptions, and the route table becomes harder to scan.

### 📝 Review notes

When reviewing Django URLconfs, read them from top to bottom. Specific routes, reserved words, and admin-like actions should usually appear before catch-all dynamic patterns.

### 💡 Takeaway

> **Django checks URL patterns in order, so route order is application behavior.**

---

## 02. Model fields and constraints

> Use precise fields, validation intent, and database constraints so the model protects real domain rules.

**Tags:** `models` `constraints` `fields` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/models-fields-constraints)

### ✅ Good Code — `reviews/models.py`

```python
from django.conf import settings
from django.db import models
from django.db.models import Q


class Review(models.Model):
    class Status(models.TextChoices):
        DRAFT = "draft", "Draft"
        PUBLISHED = "published", "Published"

    title = models.CharField(max_length=120)
    slug = models.SlugField(max_length=140)
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.DRAFT,
    )
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    published_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        # Database constraints protect rules even outside this code path.
        constraints = [
            models.UniqueConstraint(
                fields=["author", "slug"],
                name="unique_review_slug_per_author",
            ),
            models.CheckConstraint(
                condition=Q(status="draft") | Q(published_at__isnull=False),
                name="published_reviews_have_date",
            ),
        ]
```

### ❌ Bad Code — `reviews/models.py`

```python
from django.db import models


class Review(models.Model):
    # Loose nullable fields move data rules out of the model.
    title = models.TextField(null=True, blank=True)
    slug = models.CharField(max_length=255, null=True, blank=True)
    status = models.CharField(max_length=255, null=True, blank=True)
    author_id = models.IntegerField()
    published_at = models.DateTimeField(null=True, blank=True)
```

### 🔍 What to review

- **Good Code:** The good version chooses field types that match the data, links the author through a real relationship, and pushes uniqueness and publication rules into the database.
- **Bad Code:** The bad version stores almost everything as optional text or raw IDs. It leaves important invariants to scattered application code.

### 📝 Review notes

In Django model review, ask what the database is allowed to protect. If a rule must always be true, prefer an explicit field choice, relationship, index, or constraint over a comment in a service function.

### 💡 Takeaway

> **A Django model should express business invariants, not just provide loose columns.**

---

## 03. QuerySets and lazy evaluation

> Compose QuerySets before evaluation and load relationships intentionally to avoid hidden N+1 queries.

**Tags:** `querysets` `orm` `performance` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/querysets-lazy-evaluation)

### ✅ Good Code — `reviews/selectors.py`

```python
# Build the QuerySet before evaluating it.
def recent_reviews():
    return (
        Review.objects.filter(status=Review.Status.PUBLISHED)
        .select_related("author")
        .prefetch_related("comments")
        .order_by("-published_at")[:20]
    )


def review_cards():
    # Related rows are already loaded before this loop runs.
    return [
        {
            "title": review.title,
            "author": review.author.username,
            "comment_count": len(review.comments.all()),
        }
        for review in recent_reviews()
    ]
```

### ❌ Bad Code — `reviews/selectors.py`

```python
def review_cards():
    rows = []
    reviews = Review.objects.filter(status="published").order_by("-published_at")[:20]

    for review in reviews:
        rows.append(
            {
                "title": review.title,
                # Relationship access in the loop can create N+1 queries.
                "author": review.author.username,
                "comment_count": review.comments.count(),
            }
        )

    return rows
```

### 🔍 What to review

- **Good Code:** The good version composes the query, selects the related author, prefetches comments, and evaluates at the edge where cards are built.
- **Bad Code:** The bad version looks small but can trigger extra queries for every review. The performance cost is hidden inside attribute access.

### 📝 Review notes

When reading Django ORM code, trace the QuerySet lifecycle: filtering, ordering, slicing, relationship loading, and evaluation. The query plan is often more important than the line count.

### 💡 Takeaway

> **QuerySets are lazy; review where they are built, sliced, and finally evaluated.**

---

## 04. Model manager boundaries

> Put common query rules in QuerySet or manager methods so views do not repeat domain filters.

**Tags:** `models` `managers` `querysets` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/model-manager-boundaries)

### ✅ Good Code — `reviews/models.py`

```python
from django.db import models


class ReviewQuerySet(models.QuerySet):
    # Name the visibility rule once so views do not copy filters.
    def visible_to(self, user):
        base = self.filter(deleted_at__isnull=True)
        if user.is_staff:
            return base
        return base.filter(status=Review.Status.PUBLISHED)

    def with_author(self):
        # Keep relationship loading behind a readable query name.
        return self.select_related("author")


class Review(models.Model):
    objects = ReviewQuerySet.as_manager()


def review_list(request):
    reviews = Review.objects.visible_to(request.user).with_author()
    return render(request, "reviews/list.html", {"reviews": reviews})
```

### ❌ Bad Code — `reviews/views.py`

```python
# Duplicated filters drift when the rule changes.
def review_list(request):
    reviews = Review.objects.filter(
        deleted_at__isnull=True,
        status="published",
    ).select_related("author")
    return render(request, "reviews/list.html", {"reviews": reviews})


def review_feed(request):
    reviews = Review.objects.filter(
        deleted_at__isnull=True,
        status="published",
    ).order_by("-published_at")
    return render(request, "reviews/feed.html", {"reviews": reviews})
```

### 🔍 What to review

- **Good Code:** The good version names the visibility rule once and lets views ask for the query they need.
- **Bad Code:** The bad version repeats filters in multiple views. When the visibility rule changes, one path can easily drift.

### 📝 Review notes

Managers and custom QuerySets are not just convenience. They are a good place to capture vocabulary such as visible, published, archived, or owned by this user.

### 💡 Takeaway

> **Repeated filters are a sign that a query rule needs a named boundary.**

---

## 05. Forms and validation

> Use forms or ModelForms to validate request data before creating or updating model instances.

**Tags:** `forms` `validation` `views` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/forms-and-validation)

### ✅ Good Code — `reviews/views.py`

```python
from django import forms
from django.shortcuts import redirect, render

from .models import Review


class ReviewForm(forms.ModelForm):
    class Meta:
        # ModelForm owns which fields can be submitted.
        model = Review
        fields = ["title", "body"]

    def clean_title(self):
        title = self.cleaned_data["title"].strip()
        if "draft" in title.lower():
            raise forms.ValidationError("Use a final review title.")
        return title


def create_review(request):
    form = ReviewForm(request.POST or None)
    if request.method == "POST" and form.is_valid():
        review = form.save(commit=False)
        review.author = request.user
        review.save()
        return redirect("reviews:detail", pk=review.pk)

    # Invalid POST returns the form with validation errors.
    return render(request, "reviews/form.html", {"form": form})
```

### ❌ Bad Code — `reviews/views.py`

```python
from django.shortcuts import redirect, render

from .models import Review


def create_review(request):
    if request.method == "POST":
        # Direct request.POST access skips form validation.
        review = Review.objects.create(
            title=request.POST["title"],
            body=request.POST["body"],
            author=request.user,
        )
        return redirect("reviews:detail", pk=review.pk)

    return render(request, "reviews/form.html")
```

### 🔍 What to review

- **Good Code:** The good version centralizes validation in a form, uses cleaned data through `form.save()`, and keeps invalid submissions on the form page.
- **Bad Code:** The bad version trusts `request.POST` directly. Required fields, trimming, error messages, and domain validation are missing from the boundary.

### 📝 Review notes

In Django, forms are part of the request boundary. Review whether the view returns invalid input through the form error path, not as an exception that happens later.

### 💡 Takeaway

> **Request data should become cleaned data before it changes the database.**

---

## 06. Class-based view responsibilities

> Keep class-based views focused on HTTP orchestration and move query or side-effect-heavy work behind named methods.

**Tags:** `views` `cbv` `responsibility` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/class-based-view-responsibilities)

### ✅ Good Code — `reviews/views.py`

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView

from .models import Review


class ReviewListView(LoginRequiredMixin, ListView):
    model = Review
    template_name = "reviews/list.html"
    context_object_name = "reviews"
    paginate_by = 20

    def get_queryset(self):
        # The override only customizes the list query.
        return Review.objects.visible_to(self.request.user).with_author()
```

### ❌ Bad Code — `reviews/views.py`

```python
from django.views import View
from django.shortcuts import render


class ReviewDashboardView(View):
    def get(self, request):
        reviews = Review.objects.all()
        # GET now sends email and builds reports, not just a response.
        for review in reviews:
            if review.needs_digest:
                send_digest_email(review.author)

        stats = expensive_report(reviews)
        return render(
            request,
            "reviews/dashboard.html",
            {"reviews": reviews, "stats": stats},
        )
```

### 🔍 What to review

- **Good Code:** The good version uses the generic view for the list workflow and keeps the customization to the QuerySet.
- **Bad Code:** The bad version puts querying, reporting, and email side effects inside a GET handler. The class name hides more than it explains.

### 📝 Review notes

Class-based views are easiest to review when each override has a clear reason. If `get()` or `post()` reads like a service object, move the behavior behind a named function.

### 💡 Takeaway

> **A class-based view should clarify the request flow, not become a dumping ground for every concern.**

---

## 07. CSRF and unsafe methods

> Protect state-changing requests with Django's CSRF flow instead of exempting views for convenience.

**Tags:** `csrf` `security` `forms` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/csrf-and-unsafe-methods)

### ✅ Good Code — `reviews/views.py`

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import get_object_or_404, redirect
from django.views.decorators.http import require_POST

from .models import Review


@require_POST
@login_required
def publish_review(request, pk):
    # POST plus login keeps this state change protected.
    review = get_object_or_404(Review, pk=pk, author=request.user)
    review.publish()
    return redirect("reviews:detail", pk=review.pk)
```

### ❌ Bad Code — `reviews/views.py`

```python
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt

from .models import Review


@csrf_exempt
def publish_review(request, pk):
    # csrf_exempt removes a major browser-side protection.
    review = Review.objects.get(pk=pk)
    review.publish()
    return JsonResponse({"ok": True})
```

### 🔍 What to review

- **Good Code:** The good version requires POST, requires a logged-in user, and leaves CSRF middleware active for the state-changing action.
- **Bad Code:** The bad version disables CSRF and skips ownership checks. It makes a sensitive action easy to call from places the app did not intend.

### 📝 Review notes

When you see `csrf_exempt`, slow down. It might be valid for a signed webhook or separate API auth boundary, but it should never be a quick fix for a broken form.

### 💡 Takeaway

> **Unsafe methods such as POST should keep CSRF protection unless there is a deliberate API boundary.**

---

## 08. Settings by environment

> Load sensitive and environment-specific settings from the environment instead of hard-coding production behavior.

**Tags:** `settings` `security` `deployment` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/settings-environments)

### ✅ Good Code — `config/settings.py`

```python
import os

# Required secrets fail at startup when missing.
SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
DEBUG = os.environ.get("DJANGO_DEBUG") == "1"
ALLOWED_HOSTS = os.environ["DJANGO_ALLOWED_HOSTS"].split(",")

# Host and CSRF rules come from each environment.
CSRF_TRUSTED_ORIGINS = [
    origin
    for origin in os.environ.get("DJANGO_CSRF_TRUSTED_ORIGINS", "").split(",")
    if origin
]
```

### ❌ Bad Code — `config/settings.py`

```python
# Wildcard settings are too open for production.
SECRET_KEY = "dev-secret"
DEBUG = True
ALLOWED_HOSTS = ["*"]
CSRF_TRUSTED_ORIGINS = ["https://*"]
```

### 🔍 What to review

- **Good Code:** The good version makes secrets and host rules explicit per environment. Production can fail fast when required values are missing.
- **Bad Code:** The bad version ships permissive defaults. A reviewer cannot tell whether it is local-only code or production configuration waiting to leak.

### 📝 Review notes

Settings review is security review. Check `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, CSRF origins, database URLs, email backends, and anything that changes between local, staging, and production.

### 💡 Takeaway

> **Django settings are operational code; review them with security and deployment in mind.**

---

## 09. Transactions and side effects

> Wrap related database writes in atomic transactions and run external side effects after commit.

**Tags:** `transactions` `atomic` `side effects` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/transactions-and-side-effects)

### ✅ Good Code — `reviews/services.py`

```python
from django.db import transaction


def approve_review(review_id, moderator):
    with transaction.atomic():
        # Lock the row while this approval is being saved.
        review = Review.objects.select_for_update().get(pk=review_id)
        review.approve(moderator=moderator)
        review.save(update_fields=["status", "approved_by", "approved_at"])

        AuditLog.objects.create(
            actor=moderator,
            action="review.approved",
            review=review,
        )

        # Send email only after the transaction commits.
        transaction.on_commit(
            lambda: send_review_approved_email(review.pk)
        )

    return review
```

### ❌ Bad Code — `reviews/services.py`

```python
def approve_review(review_id, moderator):
    review = Review.objects.get(pk=review_id)
    review.approve(moderator=moderator)
    review.save()

    # Email can be sent before the database work fully succeeds.
    send_review_approved_email(review.pk)

    AuditLog.objects.create(
        actor=moderator,
        action="review.approved",
        review=review,
    )
    return review
```

### 🔍 What to review

- **Good Code:** The good version locks the row, saves the review and audit log atomically, and sends email only after the database commit succeeds.
- **Bad Code:** The bad version can send an email before the audit row is written. If the final write fails, the outside world has already been told the change happened.

### 📝 Review notes

Review multi-step workflows by asking what happens halfway through. If a user, email, queue, cache, and database must agree, transaction boundaries should be visible.

### 💡 Takeaway

> **Database changes and external notifications need an explicit ordering contract.**

---

## 10. Test client and fixtures

> Use Django's test client with realistic data setup and assertions that cover status, template, and database changes.

**Tags:** `tests` `client` `fixtures` · [View on site](https://good-code-bad-code.pages.dev/tracks/django/tests-client-and-fixtures)

### ✅ Good Code — `reviews/tests/test_views.py`

```python
from django.contrib.auth import get_user_model
from django.test import TestCase
from django.urls import reverse

from reviews.models import Review


class PublishReviewTests(TestCase):
    def test_author_can_publish_own_review(self):
        # Build the data this request needs inside the test.
        user = get_user_model().objects.create_user(
            username="mira",
            password="secret",
        )
        review = Review.objects.create(title="Draft", author=user)
        self.client.force_login(user)

        response = self.client.post(
            reverse("reviews:publish", args=[review.pk]),
            follow=True,
        )

        # Assert the database and rendered response after the request.
        review.refresh_from_db()
        self.assertEqual(response.status_code, 200)
        self.assertTrue(review.is_published)
        self.assertTemplateUsed(response, "reviews/detail.html")
```

### ❌ Bad Code — `reviews/tests/test_views.py`

```python
from django.test import TestCase


class PublishReviewTests(TestCase):
    def test_publish_review(self):
        # Hard-coded IDs and GET state changes hide workflow gaps.
        response = self.client.get("/reviews/1/publish/")
        self.assertContains(response, "Review published")
```

### 🔍 What to review

- **Good Code:** The good version creates the data it needs, logs in the user, sends the intended method, and checks the database state after the request.
- **Bad Code:** The bad version depends on a magic ID, uses GET for a state change, and only checks text. It can pass while the real workflow is broken.

### 📝 Review notes

Django's test client is most valuable when tests describe behavior from the outside. Prefer setup, request, and observable assertions over testing only implementation details.

### 💡 Takeaway

> **A useful Django view test proves the request behavior, not just that some text appeared.**

---

[← Back to index](README.md)
