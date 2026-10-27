# Java — Code Review Handbook

> Review null boundaries, immutability, value objects, exceptions, resources, generics, streams, concurrency, records, and service boundaries.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/java](https://good-code-bad-code.pages.dev/tracks/java)
- **Related tracks:** [Kotlin](16-kotlin.md), [Go](25-go.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Null handling and Optional boundaries](#01-null-handling-and-optional-boundaries)** — A Java method should make missing data visible in its return type or exception path, not hide it behind a nullable value.
- [ ] **[Immutable data and defensive copies](#02-immutable-data-and-defensive-copies)** — Objects that represent a completed decision should not change because a caller still holds a mutable collection reference.
- [ ] **[equals, hashCode, and value objects](#03-equals-hashcode-and-value-objects)** — If a type can be used as a map key or set member, equality and hash behavior must be deliberate and consistent.
- [ ] **[Exception handling boundaries](#04-exception-handling-boundaries)** — Exceptions should carry useful meaning inside the system and become user-facing responses only at a clear boundary.
- [ ] **[try-with-resources cleanup](#05-try-with-resources-cleanup)** — Any Java object that must be closed should have an obvious owner and a cleanup path that survives exceptions.
- [ ] **[Generics and collection types](#06-generics-and-collection-types)** — A Java collection type should tell reviewers what it contains and what key shape it expects.
- [ ] **[Streams vs readable loops](#07-streams-vs-readable-loops)** — The best Java iteration shape is the one that makes intent and failure points easiest to review.
- [ ] **[Concurrency and shared state](#08-concurrency-and-shared-state)** — If multiple threads can touch a value, the code should show which primitive owns synchronization.
- [ ] **[Records and DTO boundaries](#09-records-and-dto-boundaries)** — Boundary data should cross the application as a named shape with validation close to the edge.
- [ ] **[Package and service boundaries](#10-package-and-service-boundaries)** — A Java class is easier to review when HTTP, business rules, and persistence are separated by clear package boundaries.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Null handling and Optional boundaries](#01-null-handling-and-optional-boundaries) | `null` `optional` `boundaries` |
| 02 | [Immutable data and defensive copies](#02-immutable-data-and-defensive-copies) | `immutability` `collections` `defensive-copies` |
| 03 | [equals, hashCode, and value objects](#03-equals-hashcode-and-value-objects) | `value-objects` `equals` `hashcode` |
| 04 | [Exception handling boundaries](#04-exception-handling-boundaries) | `exceptions` `boundaries` `errors` |
| 05 | [try-with-resources cleanup](#05-try-with-resources-cleanup) | `resources` `cleanup` `jdbc` |
| 06 | [Generics and collection types](#06-generics-and-collection-types) | `generics` `collections` `types` |
| 07 | [Streams vs readable loops](#07-streams-vs-readable-loops) | `streams` `loops` `readability` |
| 08 | [Concurrency and shared state](#08-concurrency-and-shared-state) | `concurrency` `shared-state` `thread-safety` |
| 09 | [Records and DTO boundaries](#09-records-and-dto-boundaries) | `records` `dto` `api-boundaries` |
| 10 | [Package and service boundaries](#10-package-and-service-boundaries) | `architecture` `packages` `services` |

---

## 01. Null handling and Optional boundaries

> Make absence explicit at method boundaries with Optional or exceptions, instead of returning null from finder code.

**Tags:** `null` `optional` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/null-handling-and-optional-boundaries)

### ✅ Good Code — `src/main/java/dev/review/user/UserService.java`

```java
public final class UserService {
    private final UserRepository users;

    public UserService(UserRepository users) {
        this.users = Objects.requireNonNull(users);
    }

    public UserProfile profileFor(UserId id) {
        Objects.requireNonNull(id, "id");

        // Optional makes the missing-user path explicit.
        User user = users.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));

        return UserProfile.from(user);
    }
}
```

### ❌ Bad Code — `src/main/java/UserService.java`

```java
public class UserService {
    private UserRepository users;

    public UserProfile profileFor(String id) {
        User user = users.findById(id);

        // A nullable user can fail later as a NullPointerException.
        return new UserProfile(user.name(), user.email());
    }
}
```

### 🔍 What to review

- **Good Code:** The good version treats missing users as part of the contract. The repository returns an `Optional`, the service checks its input, and the failure path is named.
- **Bad Code:** The bad version lets `null` travel through the application. Reviewers have to infer whether `findById` can miss, and the actual error appears later as a less helpful `NullPointerException`.

### 📝 Review notes

In Java reviews, ask where absence is allowed. If a method may not find a value, the caller should see that in the return type or in a clear exception at the boundary.

### 💡 Takeaway

> **A Java method should make missing data visible in its return type or exception path, not hide it behind a nullable value.**

---

## 02. Immutable data and defensive copies

> Protect domain snapshots from outside mutation by copying collections and exposing immutable views.

**Tags:** `immutability` `collections` `defensive-copies` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/immutable-data-and-defensive-copies)

### ✅ Good Code — `src/main/java/dev/review/orders/OrderSnapshot.java`

```java
public record OrderSnapshot(OrderId id, List<LineItem> items) {
    public OrderSnapshot {
        Objects.requireNonNull(id, "id");
        // Copy the incoming list so outside code cannot mutate this snapshot.
        items = List.copyOf(items);
    }

    public Money total() {
        return items.stream()
            .map(LineItem::subtotal)
            .reduce(Money.zero(), Money::plus);
    }
}
```

### ❌ Bad Code — `src/main/java/OrderSnapshot.java`

```java
public class OrderSnapshot {
    private final List<LineItem> items;

    public OrderSnapshot(List<LineItem> items) {
        // This keeps the caller's mutable list reference.
        this.items = items;
    }

    public List<LineItem> items() {
        return items;
    }
}
```

### 🔍 What to review

- **Good Code:** The good version copies the collection at construction time, so later caller mutation cannot change the snapshot. Its fields are final through the record shape.
- **Bad Code:** The bad version stores and returns the same mutable list. Any code with a reference can add, remove, or reorder items after the object was created.

### 📝 Review notes

Look closely at constructors and getters that accept or return collections. A defensive copy is often the difference between a stable value and a hidden shared state bug.

### 💡 Takeaway

> **Objects that represent a completed decision should not change because a caller still holds a mutable collection reference.**

---

## 03. equals, hashCode, and value objects

> Use small value objects or records for identity values so equality, hashing, and validation stay together.

**Tags:** `value-objects` `equals` `hashcode` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/equals-hashcode-and-value-objects)

### ✅ Good Code — `src/main/java/dev/review/user/UserId.java`

```java
public record UserId(UUID value) {
    public UserId {
        // Record equality and hashCode stay tied to this value.
        Objects.requireNonNull(value, "value");
    }

    public static UserId parse(String rawValue) {
        return new UserId(UUID.fromString(rawValue));
    }
}
```

### ❌ Bad Code — `src/main/java/User.java`

```java
public class User {
    private String id;
    private String displayName;

    @Override
    public boolean equals(Object other) {
        // Display names are not stable identity, and hashCode is missing.
        if (!(other instanceof User user)) {
            return false;
        }

        return displayName.equals(user.displayName);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version models identity as a value object. Java records give consistent equality and hash behavior, while the constructor keeps invalid values out.
- **Bad Code:** The bad version compares users by display name and overrides `equals` without a matching `hashCode`. Collections such as `HashSet` and `HashMap` can behave incorrectly.

### 📝 Review notes

When reviewing equality, check both the domain meaning and the Java contract. If equality is important, prefer a focused value object over spreading identity rules through larger mutable classes.

### 💡 Takeaway

> **If a type can be used as a map key or set member, equality and hash behavior must be deliberate and consistent.**

---

## 04. Exception handling boundaries

> Throw domain-specific exceptions inside the application and translate them at the outer boundary.

**Tags:** `exceptions` `boundaries` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/exception-handling-boundaries)

### ✅ Good Code — `src/main/java/dev/review/api/UserController.java`

```java
public final class UserController {
    private final UserService users;

    public ResponseEntity<UserResponse> show(UUID id) {
        try {
            UserProfile profile = users.profileFor(new UserId(id));
            return ResponseEntity.ok(UserResponse.from(profile));
        } catch (UserNotFoundException exception) {
            // Translate the domain exception at the HTTP boundary.
            return ResponseEntity.notFound().build();
        }
    }
}
```

### ❌ Bad Code — `src/main/java/UserController.java`

```java
public class UserController {
    public UserResponse show(String id) {
        try {
            return users.profileFor(id);
        } catch (Exception exception) {
            // Catching everything hides the original failure path.
            exception.printStackTrace();
            return null;
        }
    }
}
```

### 🔍 What to review

- **Good Code:** The good version keeps the service failure meaningful, then maps it to HTTP at the controller boundary. The response shape stays predictable.
- **Bad Code:** The bad version catches every exception, prints diagnostics to standard output, and returns `null`. The caller now has a second failure mode that hides the original problem.

### 📝 Review notes

Ask whether the catch block is at the right layer. Catching too low often hides bugs; catching at a boundary lets the application translate failures without losing intent.

### 💡 Takeaway

> **Exceptions should carry useful meaning inside the system and become user-facing responses only at a clear boundary.**

---

## 05. try-with-resources cleanup

> Use try-with-resources for files, connections, statements, and result sets so cleanup happens on success and failure.

**Tags:** `resources` `cleanup` `jdbc` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/try-with-resources-cleanup)

### ✅ Good Code — `src/main/java/dev/review/user/JdbcUserRepository.java`

```java
public Optional<User> findByEmail(String email) throws SQLException {
    String sql = "select id, email, name from users where email = ?";

    // try-with-resources owns cleanup for every closeable here.
    try (
        Connection connection = dataSource.getConnection();
        PreparedStatement statement = connection.prepareStatement(sql)
    ) {
        statement.setString(1, email);

        try (ResultSet rows = statement.executeQuery()) {
            if (!rows.next()) {
                return Optional.empty();
            }

            return Optional.of(mapUser(rows));
        }
    }
}
```

### ❌ Bad Code — `src/main/java/JdbcUserRepository.java`

```java
public User findByEmail(String email) throws SQLException {
    Connection connection = dataSource.getConnection();
    PreparedStatement statement = connection.prepareStatement(
        "select * from users where email = '" + email + "'"
    );

    ResultSet rows = statement.executeQuery();
    if (!rows.next()) {
        // Early returns skip manual cleanup.
        return null;
    }

    connection.close();
    return mapUser(rows);
}
```

### 🔍 What to review

- **Good Code:** The good version makes resource ownership visible. Connections, statements, and result sets are closed even when the query or mapping throws.
- **Bad Code:** The bad version manually closes only one resource on the happy path, skips cleanup on early returns, and mixes unsafe SQL construction into the same method.

### 📝 Review notes

Resource cleanup bugs are easy to miss in review because the happy path looks fine. Follow every return and exception path, then check that each closeable resource has an owner.

### 💡 Takeaway

> **Any Java object that must be closed should have an obvious owner and a cleanup path that survives exceptions.**

---

## 06. Generics and collection types

> Keep collection element types precise so casts, raw types, and runtime class errors do not leak through the codebase.

**Tags:** `generics` `collections` `types` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/generics-and-collection-types)

### ✅ Good Code — `src/main/java/dev/review/orders/OrderSummaryIndex.java`

```java
public final class OrderSummaryIndex {
    // The type says each owner can have many summaries.
    public Map<UserId, List<OrderSummary>> groupByOwner(List<OrderSummary> orders) {
        return orders.stream()
            .collect(Collectors.groupingBy(OrderSummary::ownerId));
    }

    public List<OrderSummary> summariesFor(
        Map<UserId, List<OrderSummary>> index,
        UserId ownerId
    ) {
        return index.getOrDefault(ownerId, List.of());
    }
}
```

### ❌ Bad Code — `src/main/java/OrderSummaryIndex.java`

```java
public class OrderSummaryIndex {
    public Map groupByOwner(List orders) {
        Map index = new HashMap();
        for (Object item : orders) {
            // Raw collections push type errors to runtime casts.
            OrderSummary order = (OrderSummary) item;
            index.put(order.ownerId().toString(), order);
        }
        return index;
    }
}
```

### 🔍 What to review

- **Good Code:** The good version names both the key and value types. Reviewers can see that one owner may have many summaries, and callers do not need casts.
- **Bad Code:** The bad version uses raw `List` and `Map`, stores only one order per owner, and pushes type mistakes to runtime.

### 📝 Review notes

Raw collections are a review smell in modern Java. They remove the compiler from the conversation and make reviewers simulate type checks by hand.

### 💡 Takeaway

> **A Java collection type should tell reviewers what it contains and what key shape it expects.**

---

## 07. Streams vs readable loops

> Use streams for clear transformations and loops when branching, validation, or naming intermediate steps makes the review easier.

**Tags:** `streams` `loops` `readability` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/streams-vs-readable-loops)

### ✅ Good Code — `src/main/java/dev/review/orders/DiscountCalculator.java`

```java
public Money discountFor(List<OrderLine> lines) {
    Money discount = Money.zero();

    // A loop gives the branch and accumulator clear names.
    for (OrderLine line : lines) {
        if (!line.isDiscountable()) {
            continue;
        }

        Money lineDiscount = policy.discountFor(line);
        discount = discount.plus(lineDiscount);
    }

    return discount;
}
```

### ❌ Bad Code — `src/main/java/DiscountCalculator.java`

```java
public Money discountFor(List<OrderLine> lines) {
    AtomicReference<Money> discount = new AtomicReference<>(Money.zero());

    lines.stream()
        .filter(line -> line.isDiscountable())
        // Side effects inside streams make the pipeline harder to reason about.
        .peek(line -> audit.log(line.id()))
        .map(line -> policy.discountFor(line))
        .forEach(value -> discount.set(discount.get().plus(value)));

    return discount.get();
}
```

### 🔍 What to review

- **Good Code:** The good version uses a loop because the operation has branching and a named accumulation step. Each review question has a place to attach.
- **Bad Code:** The bad version hides mutation inside a stream pipeline and uses `peek` for side effects. The stream looks functional, but it is harder to reason about.

### 📝 Review notes

Streams are not automatically cleaner than loops. Prefer streams for simple mapping, filtering, and collecting; use loops when the business rule needs names and branches.

### 💡 Takeaway

> **The best Java iteration shape is the one that makes intent and failure points easiest to review.**

---

## 08. Concurrency and shared state

> Use concurrent primitives for shared state so check-then-act races do not appear under load.

**Tags:** `concurrency` `shared-state` `thread-safety` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/concurrency-and-shared-state)

### ✅ Good Code — `src/main/java/dev/review/cache/ProfileCache.java`

```java
public final class ProfileCache {
    private final ConcurrentMap<UserId, UserProfile> profiles = new ConcurrentHashMap<>();
    private final UserService users;

    public UserProfile get(UserId id) {
        // computeIfAbsent makes load-and-store one atomic cache step.
        return profiles.computeIfAbsent(id, users::loadProfile);
    }
}
```

### ❌ Bad Code — `src/main/java/ProfileCache.java`

```java
public class ProfileCache {
    private final Map<String, UserProfile> profiles = new HashMap<>();

    public UserProfile get(String id) {
        // containsKey then put is a race on a shared HashMap.
        if (!profiles.containsKey(id)) {
            profiles.put(id, users.loadProfile(id));
        }

        return profiles.get(id);
    }
}
```

### 🔍 What to review

- **Good Code:** The good version uses a concurrent map and makes the atomic operation explicit with `computeIfAbsent`.
- **Bad Code:** The bad version uses a shared `HashMap` and a check-then-put sequence. Two threads can compute the same value, corrupt the map, or observe inconsistent state.

### 📝 Review notes

When reviewing shared state, find the synchronization owner. If the answer is "the timing usually works", the code needs a safer primitive or a narrower ownership model.

### 💡 Takeaway

> **If multiple threads can touch a value, the code should show which primitive owns synchronization.**

---

## 09. Records and DTO boundaries

> Use records or explicit DTOs for request and response shapes instead of untyped maps that push errors deeper into the application.

**Tags:** `records` `dto` `api-boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/records-and-dto-boundaries)

### ✅ Good Code — `src/main/java/dev/review/api/CreateUserRequest.java`

```java
public record CreateUserRequest(String email, String displayName) {
    public CreateUserRequest {
        // The request shape validates itself before reaching the service.
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("email is required");
        }

        if (displayName == null || displayName.isBlank()) {
            throw new IllegalArgumentException("displayName is required");
        }
    }
}
```

### ❌ Bad Code — `src/main/java/UserController.java`

```java
public UserResponse create(Map<String, Object> body) {
    // String keys and casts hide request-shape mistakes until runtime.
    String email = (String) body.get("email");
    String name = (String) body.get("name");

    User user = users.create(email.trim(), name.trim());

    return new UserResponse(user.id(), user.email(), user.name());
}
```

### 🔍 What to review

- **Good Code:** The good version gives the boundary a name and validates required fields before the service receives them.
- **Bad Code:** The bad version treats the request as a loose map, relies on casts, and uses a different key name than the domain concept. Missing values fail later in confusing ways.

### 📝 Review notes

DTOs are not ceremony when they protect a boundary. They make request shape, validation, and downstream expectations visible in one small place.

### 💡 Takeaway

> **Boundary data should cross the application as a named shape with validation close to the edge.**

---

## 10. Package and service boundaries

> Keep controllers, services, repositories, and packages focused so each layer owns one kind of decision.

**Tags:** `architecture` `packages` `services` · [View on site](https://good-code-bad-code.pages.dev/tracks/java/package-service-boundaries)

### ✅ Good Code — `src/main/java/dev/review/user/UserService.java`

```java
package dev.review.user;

public final class UserService {
    private final UserRepository users;
    private final WelcomeEmail welcomeEmail;

    public UserService(UserRepository users, WelcomeEmail welcomeEmail) {
        this.users = users;
        this.welcomeEmail = welcomeEmail;
    }

    public UserProfile register(CreateUserCommand command) {
        // The service owns the business flow and calls dependencies.
        User user = users.save(User.create(command.email(), command.displayName()));
        welcomeEmail.sendTo(user);
        return UserProfile.from(user);
    }
}
```

### ❌ Bad Code — `src/main/java/UserController.java`

```java
public class UserController {
    public ResponseEntity<?> register(HttpServletRequest request) throws SQLException {
        // HTTP, SQL, connection management, and email are mixed together.
        String email = request.getParameter("email");
        Connection connection = DriverManager.getConnection(databaseUrl);
        Statement statement = connection.createStatement();
        statement.executeUpdate("insert into users(email) values('" + email + "')");
        mailer.send(email, "Welcome");
        return ResponseEntity.ok().build();
    }
}
```

### 🔍 What to review

- **Good Code:** The good version gives business behavior a service boundary. Persistence and email are dependencies, not hidden work inside a controller.
- **Bad Code:** The bad version mixes request parsing, SQL construction, database connection management, email, and HTTP response code in one class.

### 📝 Review notes

For Java architecture reviews, trace one user action through packages. If a controller owns every decision, split by responsibility before adding more behavior.

### 💡 Takeaway

> **A Java class is easier to review when HTTP, business rules, and persistence are separated by clear package boundaries.**

---

[← Back to index](README.md)
