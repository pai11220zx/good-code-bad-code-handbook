# PHP — Code Review Handbook

> Review strict types, request validation, output escaping, PDO queries, passwords, sessions, configuration, and template boundaries.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/php](https://good-code-bad-code.pages.dev/tracks/php)
- **Related tracks:** [Laravel](14-laravel.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Strict types and return types](#01-strict-types-and-return-types)** — A PHP file that owns business rules should make parameter and return expectations visible at the function boundary.
- [ ] **[Input validation and filtering](#02-input-validation-and-filtering)** — Superglobals are untrusted boundaries; normalize them into explicit values before using them.
- [ ] **[Output escaping for XSS](#03-output-escaping-for-xss)** — Prepared statements protect SQL, but HTML output still needs context-aware escaping.
- [ ] **[PDO prepared statements](#04-pdo-prepared-statements)** — SQL structure and SQL values should stay separate; PDO placeholders make that boundary reviewable.
- [ ] **[Password hashing and verification](#05-password-hashing-and-verification)** — Passwords should go through password_hash and password_verify, never fast general-purpose hashes.
- [ ] **[Session handling and cookies](#06-session-handling-and-cookies)** — Session code should make cookie flags and identity transitions visible to reviewers.
- [ ] **[Error handling and exceptions](#07-error-handling-and-exceptions)** — Good PHP error handling separates developer diagnostics from user-facing responses.
- [ ] **[Autoloading and namespaces](#08-autoloading-and-namespaces)** — Modern PHP code should make class ownership visible through namespaces instead of require chains.
- [ ] **[Configuration and secrets](#09-configuration-and-secrets)** — Secrets should be runtime configuration, not strings committed next to application logic.
- [ ] **[Separating logic from templates](#10-separating-logic-from-templates)** — Templates should render a prepared view model, not query databases or decide application flow.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Strict types and return types](#01-strict-types-and-return-types) | `types` `strict-types` `contracts` |
| 02 | [Input validation and filtering](#02-input-validation-and-filtering) | `input` `validation` `request` |
| 03 | [Output escaping for XSS](#03-output-escaping-for-xss) | `xss` `escaping` `templates` |
| 04 | [PDO prepared statements](#04-pdo-prepared-statements) | `pdo` `sql` `prepared-statements` |
| 05 | [Password hashing and verification](#05-password-hashing-and-verification) | `passwords` `security` `hashing` |
| 06 | [Session handling and cookies](#06-session-handling-and-cookies) | `sessions` `cookies` `auth` |
| 07 | [Error handling and exceptions](#07-error-handling-and-exceptions) | `errors` `exceptions` `logging` |
| 08 | [Autoloading and namespaces](#08-autoloading-and-namespaces) | `autoloading` `namespaces` `structure` |
| 09 | [Configuration and secrets](#09-configuration-and-secrets) | `configuration` `secrets` `environment` |
| 10 | [Separating logic from templates](#10-separating-logic-from-templates) | `templates` `separation` `mvc` |

---

## 01. Strict types and return types

> Make PHP function contracts explicit with strict scalar types and return types so reviewers can catch coercion bugs at boundaries.

**Tags:** `types` `strict-types` `contracts` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/strict-types-and-return-types)

### ✅ Good Code — `src/Billing/InvoiceTotal.php`

```php
<?php declare(strict_types=1);

// Strict types make cents stay integers at the function boundary.
function invoiceTotalCents(int $subtotalCents, int $taxCents): int
{
    if ($subtotalCents < 0 || $taxCents < 0) {
        throw new InvalidArgumentException('Invoice amounts must be positive.');
    }

    return $subtotalCents + $taxCents;
}
```

### ❌ Bad Code — `invoice.php`

```php
<?php

// Untyped request values let PHP coerce strings during math.
function invoiceTotal($subtotal, $tax)
{
    return $subtotal + $tax;
}

echo invoiceTotal($_POST['subtotal'], $_POST['tax']);
```

### 🔍 What to review

- **Good Code:** The good version declares strict typing, names the unit of money, returns a specific type, and rejects invalid values before computing.
- **Bad Code:** The bad version relies on PHP coercion and untyped request values. A reviewer has to infer whether strings, floats, negative numbers, or missing fields are valid.

### 📝 Review notes

In PHP reviews, look for contracts at the edge of every important function. Strict types do not replace validation, but they make accidental coercion much harder to miss.

### 💡 Takeaway

> **A PHP file that owns business rules should make parameter and return expectations visible at the function boundary.**

---

## 02. Input validation and filtering

> Validate request input at the boundary before it reaches query, domain, or template code.

**Tags:** `input` `validation` `request` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/input-validation-filtering)

### ✅ Good Code — `public/orders.php`

```php
<?php declare(strict_types=1);

// Validate and normalize the boundary value before using it.
$orderId = filter_input(
    INPUT_GET,
    'order_id',
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1]]
);

if ($orderId === false || $orderId === null) {
    http_response_code(400);
    echo 'Invalid order id';
    return;
}

$order = $orders->findById($orderId);
```

### ❌ Bad Code — `public/orders.php`

```php
<?php

// Raw $_GET values mix missing, invalid, and empty cases.
$orderId = $_GET['order_id'];

if (!$orderId) {
    echo 'Missing order id';
}

$order = $orders->findById($orderId);
```

### 🔍 What to review

- **Good Code:** The good version reads the original request value, validates that it is a positive integer, and stops early with a clear HTTP response.
- **Bad Code:** The bad version pulls directly from `$_GET`, treats empty and invalid values the same, and passes a raw boundary value deeper into the application.

### 📝 Review notes

Validation should happen once near the boundary. After that point, downstream code should receive typed, normalized values instead of reading superglobals again.

### 💡 Takeaway

> **Superglobals are untrusted boundaries; normalize them into explicit values before using them.**

---

## 03. Output escaping for XSS

> Escape untrusted values when rendering HTML so user content cannot become markup or script.

**Tags:** `xss` `escaping` `templates` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/output-escaping-xss)

### ✅ Good Code — `templates/profile.php`

```php
<?php declare(strict_types=1);

// Escape text before it becomes HTML output.
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
?>

<h1><?= e($user['name']) ?></h1>
<p><?= e($user['bio']) ?></p>
```

### ❌ Bad Code — `templates/profile.php`

```php
<?php
// Direct output treats user content as trusted HTML.
?>

<h1><?= $user['name'] ?></h1>
<p><?= $_GET['bio'] ?></p>
```

### 🔍 What to review

- **Good Code:** The good version uses a small escaping helper for text rendered into HTML, including quotes and invalid byte substitution.
- **Bad Code:** The bad version outputs database and request values directly. If those values contain HTML, the template will treat them as markup.

### 📝 Review notes

Review output by context. Text nodes need HTML escaping, attributes need attribute-safe escaping, and JavaScript contexts need a different strategy.

### 💡 Takeaway

> **Prepared statements protect SQL, but HTML output still needs context-aware escaping.**

---

## 04. PDO prepared statements

> Use prepared statements with bound values instead of interpolating request data into SQL strings.

**Tags:** `pdo` `sql` `prepared-statements` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/pdo-prepared-statements)

### ✅ Good Code — `src/UserRepository.php`

```php
<?php declare(strict_types=1);

final class UserRepository
{
    public function __construct(private PDO $pdo) {}

    public function findActiveByEmail(string $email): ?array
    {
        // Placeholders keep SQL structure separate from user data.
        $stmt = $this->pdo->prepare(
            'SELECT id, email, name FROM users WHERE email = :email AND active = 1'
        );
        $stmt->execute(['email' => $email]);

        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        return $user === false ? null : $user;
    }
}
```

### ❌ Bad Code — `users.php`

```php
<?php

$email = $_GET['email'];
// Interpolating request data makes the query boundary unsafe.
$sql = "SELECT * FROM users WHERE email = '$email' AND active = 1";
$user = $pdo->query($sql)->fetch();
```

### 🔍 What to review

- **Good Code:** The good version prepares SQL once, binds the email as data, selects explicit columns, and returns `null` when no row matches.
- **Bad Code:** The bad version interpolates a request value into SQL and selects every column. A reviewer cannot trust the query boundary or the response shape.

### 📝 Review notes

Prepared statements are more than a security habit. They expose query intent, parameter names, and result handling during review.

### 💡 Takeaway

> **SQL structure and SQL values should stay separate; PDO placeholders make that boundary reviewable.**

---

## 05. Password hashing and verification

> Store password hashes with PHP password APIs and verify them without inventing custom hashing rules.

**Tags:** `passwords` `security` `hashing` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/password-hashing-verification)

### ✅ Good Code — `src/Auth/Passwords.php`

```php
<?php declare(strict_types=1);

function hashPassword(string $plainPassword): string
{
    // Let PHP choose the current password-safe algorithm and cost.
    return password_hash($plainPassword, PASSWORD_DEFAULT);
}

function verifyPassword(string $plainPassword, string $storedHash): bool
{
    return password_verify($plainPassword, $storedHash);
}

function shouldRehash(string $storedHash): bool
{
    return password_needs_rehash($storedHash, PASSWORD_DEFAULT);
}
```

### ❌ Bad Code — `login.php`

```php
<?php

// md5 is too fast and not designed for stored passwords.
$hash = md5($_POST['password']);

if ($hash === $user['password_hash']) {
    $_SESSION['user_id'] = $user['id'];
}
```

### 🔍 What to review

- **Good Code:** The good version relies on PHP password APIs, which store the algorithm and cost information in the hash and can signal when rehashing is needed.
- **Bad Code:** The bad version uses a fast hash that is not designed for passwords, and it couples login success directly to session mutation.

### 📝 Review notes

Password review should be boring. If the code has custom salts, fast hashes, manual comparisons, or homegrown upgrade logic, slow down and ask why.

### 💡 Takeaway

> **Passwords should go through password_hash and password_verify, never fast general-purpose hashes.**

---

## 06. Session handling and cookies

> Configure session cookies deliberately and regenerate session IDs when authentication state changes.

**Tags:** `sessions` `cookies` `auth` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/session-handling-cookies)

### ✅ Good Code — `src/Auth/Session.php`

```php
<?php declare(strict_types=1);

// Cookie flags are configured before the session starts.
session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();

function signIn(int $userId): void
{
    session_regenerate_id(true);
    $_SESSION['user_id'] = $userId;
}
```

### ❌ Bad Code — `login.php`

```php
<?php

session_start();

if ($_POST['password'] === $user['password']) {
    $_SESSION['user_id'] = $user['id'];
    // Request-controlled roles should not become authorization state.
    $_SESSION['role'] = $_GET['role'];
}
```

### 🔍 What to review

- **Good Code:** The good version sets cookie flags before starting the session and regenerates the session ID when the user signs in.
- **Bad Code:** The bad version uses default cookie behavior, does not rotate the session ID, and trusts a request parameter for authorization state.

### 📝 Review notes

Authentication changes are session boundary changes. Review login, logout, and privilege changes for cookie flags, ID rotation, and where session values come from.

### 💡 Takeaway

> **Session code should make cookie flags and identity transitions visible to reviewers.**

---

## 07. Error handling and exceptions

> Catch exceptions at a clear boundary, log useful context, and return safe responses to users.

**Tags:** `errors` `exceptions` `logging` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/error-handling-exceptions)

### ✅ Good Code — `public/index.php`

```php
<?php declare(strict_types=1);

try {
    $response = $router->dispatch($request);
    $response->send();
} catch (Throwable $error) {
    // Log details for developers but send users a safe response.
    $logger->error('Unhandled request failure', [
        'path' => $request->path(),
        'exception' => $error,
    ]);

    http_response_code(500);
    echo 'Something went wrong.';
}
```

### ❌ Bad Code — `public/index.php`

```php
<?php

try {
    // This lets the request choose a file and exposes raw failures.
    include $_GET['page'] . '.php';
} catch (Exception $error) {
    die($error->getMessage());
}
```

### 🔍 What to review

- **Good Code:** The good version catches failures at the front controller boundary, logs internal details, and returns a generic response.
- **Bad Code:** The bad version mixes routing with unsafe include behavior and sends raw exception messages to the browser.

### 📝 Review notes

Review where exceptions are caught. Catching too low hides failures; catching at the boundary lets the app log context and keep user output safe.

### 💡 Takeaway

> **Good PHP error handling separates developer diagnostics from user-facing responses.**

---

## 08. Autoloading and namespaces

> Use namespaces and autoloaded classes so file structure, ownership, and dependencies are explicit.

**Tags:** `autoloading` `namespaces` `structure` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/autoloading-and-namespaces)

### ✅ Good Code — `src/Service/UserProfileService.php`

```php
<?php declare(strict_types=1);

namespace App\Service;

use App\Repository\UserRepository;

// Namespace and constructor injection make ownership visible.
final class UserProfileService
{
    public function __construct(private UserRepository $users) {}

    public function profileFor(int $userId): array
    {
        return $this->users->findProfile($userId);
    }
}
```

### ❌ Bad Code — `profile.php`

```php
<?php

// Manual includes and globals hide where dependencies come from.
require_once 'db.php';
require_once 'users.php';
require_once 'helpers.php';

function profile_for($id) {
    global $db;
    return find_profile($db, $id);
}
```

### 🔍 What to review

- **Good Code:** The good version gives the class a namespace, injects its dependency, and lets autoloading connect files to class names.
- **Bad Code:** The bad version depends on include order, global state, and free functions whose ownership is hard to trace.

### 📝 Review notes

Autoloading is a maintainability boundary. If a reviewer has to follow a chain of `require_once` calls to understand a dependency, the structure is doing too little work.

### 💡 Takeaway

> **Modern PHP code should make class ownership visible through namespaces instead of require chains.**

---

## 09. Configuration and secrets

> Load configuration from the environment and fail clearly when required secrets are missing.

**Tags:** `configuration` `secrets` `environment` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/configuration-and-secrets)

### ✅ Good Code — `config/database.php`

```php
<?php declare(strict_types=1);

$databaseUrl = getenv('DATABASE_URL');

// Required secrets should fail fast when the environment is missing.
if ($databaseUrl === false || $databaseUrl === '') {
    throw new RuntimeException('DATABASE_URL is required.');
}

return [
    'url' => $databaseUrl,
    'timeout_seconds' => 3,
];
```

### ❌ Bad Code — `config/database.php`

```php
<?php

// Credentials in code and request-driven config are both unsafe.
return [
    'url' => 'mysql://root:secret-password@localhost/app',
    'timeout_seconds' => $_GET['timeout'] ?? 30,
];
```

### 🔍 What to review

- **Good Code:** The good version reads a required secret from the runtime environment and fails during startup if it is missing.
- **Bad Code:** The bad version commits credentials and allows a request value to influence infrastructure configuration.

### 📝 Review notes

Configuration review should answer two questions quickly: where does this value come from, and can user input change it?

### 💡 Takeaway

> **Secrets should be runtime configuration, not strings committed next to application logic.**

---

## 10. Separating logic from templates

> Keep request handling and data access outside templates so rendering code stays small and easy to review.

**Tags:** `templates` `separation` `mvc` · [View on site](https://good-code-bad-code.pages.dev/tracks/php/separating-logic-from-templates)

### ✅ Good Code — `src/Controller/ProfileController.php`

```php
<?php declare(strict_types=1);

final class ProfileController
{
    public function __construct(private UserRepository $users) {}

    public function show(int $userId): array
    {
        // The controller prepares data before the template renders it.
        $user = $this->users->findProfile($userId);

        return [
            'title' => $user['name'],
            'user' => $user,
        ];
    }
}
```

### ❌ Bad Code — `profile.php`

```php
<?php

// Request input, SQL, and output are mixed in one template file.
$userId = $_GET['id'];
$user = $pdo->query("SELECT * FROM users WHERE id = $userId")->fetch();
?>

<h1><?= $user['name'] ?></h1>
<p><?= $user['bio'] ?></p>
```

### 🔍 What to review

- **Good Code:** The good version prepares a view model before rendering. The template can focus on display and escaping.
- **Bad Code:** The bad version mixes request input, SQL, database access, and raw output in one file. Every change becomes a cross-cutting review.

### 📝 Review notes

PHP can embed HTML naturally, but that convenience should not erase boundaries. Ask whether the file is handling a request, fetching data, or rendering a view; if it is doing all three, split it.

### 💡 Takeaway

> **Templates should render a prepared view model, not query databases or decide application flow.**

---

[← Back to index](README.md)
