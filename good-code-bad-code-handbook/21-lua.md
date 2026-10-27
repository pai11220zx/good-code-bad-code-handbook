# Lua — Code Review Handbook

> Review table shapes, nil boundaries, modules, metatables, coroutines, sandboxed execution, embedded APIs, and tests.

- **Category:** Backend
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/lua](https://good-code-bad-code.pages.dev/tracks/lua)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Tables as records and arrays](#01-tables-as-records-and-arrays)** — Lua table review should name whether a table is a record, array, map, or mixed structure.
- [ ] **[Nil boundaries and defaults](#02-nil-boundaries-and-defaults)** — Lua code should choose defaults at input boundaries instead of scattering nil checks through business logic.
- [ ] **[Module return contracts](#03-module-return-contracts)** — A Lua module should return the functions it wants callers to use and avoid mutating global state during import.
- [ ] **[Metatable operator boundaries](#04-metatable-operator-boundaries)** — Metatable review should check whether metamethods keep domain operations visible or hide side effects.
- [ ] **[Coroutine lifecycle](#05-coroutine-lifecycle)** — Lua coroutine code should inspect resume success and status before assuming a task advanced.
- [ ] **[pcall error handling](#06-pcall-error-handling)** — Lua host code should catch plugin failures at the boundary and return a result the caller can classify.
- [ ] **[Sandboxed execution](#07-sandboxed-execution)** — Dynamic Lua execution should run with a small explicit environment instead of the full global table.
- [ ] **[Embedded API boundaries](#08-embedded-api-boundaries)** — An embedded Lua API should expose narrow commands rather than raw host objects with broad mutation power.
- [ ] **[Configuration validation](#09-configuration-validation)** — Lua config review should check required fields and value types at load time.
- [ ] **[Busted test cases](#10-busted-test-cases)** — Lua tests should name the behavior, build a small fixture, and assert the exact output or error shape.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Tables as records and arrays](#01-tables-as-records-and-arrays) | `tables` `shape` `records` |
| 02 | [Nil boundaries and defaults](#02-nil-boundaries-and-defaults) | `nil` `defaults` `boundaries` |
| 03 | [Module return contracts](#03-module-return-contracts) | `modules` `require` `globals` |
| 04 | [Metatable operator boundaries](#04-metatable-operator-boundaries) | `metatable` `operators` `domain` |
| 05 | [Coroutine lifecycle](#05-coroutine-lifecycle) | `coroutines` `lifecycle` `errors` |
| 06 | [pcall error handling](#06-pcall-error-handling) | `pcall` `errors` `plugins` |
| 07 | [Sandboxed execution](#07-sandboxed-execution) | `sandbox` `environment` `security` |
| 08 | [Embedded API boundaries](#08-embedded-api-boundaries) | `embedded` `api` `host` |
| 09 | [Configuration validation](#09-configuration-validation) | `configuration` `validation` `tables` |
| 10 | [Busted test cases](#10-busted-test-cases) | `tests` `busted` `assertions` |

---

## 01. Tables as records and arrays

> Keep table shape clear when a value behaves like a record, array, or map.

**Tags:** `tables` `shape` `records` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/table-shapes)

### ✅ Good Code — `src/review_summary.lua`

```lua
local function build_review_summary(title, score)
  -- Named fields make this table read like a stable record.
  return {
    title = title,
    score = score,
    approved = score >= 4,
  }
end
```

### ❌ Bad Code — `review_summary.lua`

```lua
local function build_review_summary(title, score)
  -- Positional fields hide what each slot means to the caller.
  return { title, score, score >= 4 }
end
```

### 🔍 What to review

- **Good Code:** The good version uses named fields, so callers can read the table as a record and access values without remembering index positions.
- **Bad Code:** The bad version mixes unrelated values into positional slots. A caller has to know that index `3` means approval state, which is easy to break during later edits.

### 📝 Review notes

Lua tables can represent many shapes. In review, ask which shape the code intends and whether the access pattern makes that shape visible.

### 💡 Takeaway

> **Lua table review should name whether a table is a record, array, map, or mixed structure.**

---

## 02. Nil boundaries and defaults

> Normalize nil at the boundary so the rest of the code can distinguish missing data from empty values.

**Tags:** `nil` `defaults` `boundaries` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/nil-boundary-defaults)

### ✅ Good Code — `src/review_options.lua`

```lua
local function normalize_options(options)
  options = options or {}

  -- Defaults are chosen once before the review job runs.
  return {
    limit = options.limit or 20,
    include_archived = options.include_archived == true,
  }
end
```

### ❌ Bad Code — `review_options.lua`

```lua
local function fetch_reviews(options)
  -- Every read repeats a nil decision with slightly different rules.
  local limit = options and options.limit or 20
  local archived = options and options.include_archived

  return query_reviews(limit, archived)
end
```

### 🔍 What to review

- **Good Code:** The good version turns optional input into one normalized table. Later code can rely on `limit` and `include_archived` having predictable types.
- **Bad Code:** The bad version repeats nil checks inside the workflow. Each line can drift into a different default rule, especially when booleans are involved.

### 📝 Review notes

In Lua reviews, watch for `nil` flowing deep into the code path. Defaults are easier to audit when they live at the boundary where input enters the module.

### 💡 Takeaway

> **Lua code should choose defaults at input boundaries instead of scattering nil checks through business logic.**

---

## 03. Module return contracts

> Return an explicit module table so require calls expose a stable API instead of hidden globals.

**Tags:** `modules` `require` `globals` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/module-return-contracts)

### ✅ Good Code — `src/review_rules.lua`

```lua
local ReviewRules = {}

function ReviewRules.is_approved(review)
  -- The exported function is visible on the returned module table.
  return review.score ~= nil and review.score >= 4
end

return ReviewRules
```

### ❌ Bad Code — `review_rules.lua`

```lua
function is_approved(review)
  -- Global function names can collide across unrelated modules.
  return review.score >= 4
end
```

### 🔍 What to review

- **Good Code:** The good version builds and returns a module table. A caller using `require` can see the exported surface and keep names scoped.
- **Bad Code:** The bad version writes a global function during load. Another file can replace it, and tests can pass or fail depending on import order.

### 📝 Review notes

Lua modules should make their public API explicit. Review top-level assignments carefully because missing `local` often means global state changed during import.

### 💡 Takeaway

> **A Lua module should return the functions it wants callers to use and avoid mutating global state during import.**

---

## 04. Metatable operator boundaries

> Use metatables for clear domain behavior, not for surprising mutation or hidden lookup rules.

**Tags:** `metatable` `operators` `domain` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/metatable-operator-boundaries)

### ✅ Good Code — `src/review_score.lua`

```lua
local Score = {}
Score.__index = Score

function Score.new(value)
  return setmetatable({ value = value }, Score)
end

function Score:passes()
  -- Method syntax keeps the approval rule explicit.
  return self.value >= 4
end
```

### ❌ Bad Code — `review_score.lua`

```lua
local score_meta = {
  __add = function(left, right)
    -- Addition mutates the left operand instead of returning a new score.
    left.value = left.value + right.value
    return left
  end,
}
```

### 🔍 What to review

- **Good Code:** The good version uses a metatable to attach methods to a score object. The approval rule is still a named method with visible intent.
- **Bad Code:** The bad version overloads addition and mutates the left value. A reader expecting arithmetic gets hidden state changes.

### 📝 Review notes

Metatables are powerful enough to make code elegant or opaque. In review, prefer named methods unless a metamethod matches a familiar, side-effect-free operation.

### 💡 Takeaway

> **Metatable review should check whether metamethods keep domain operations visible or hide side effects.**

---

## 05. Coroutine lifecycle

> Check coroutine status and resume results so yielded work cannot fail silently.

**Tags:** `coroutines` `lifecycle` `errors` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/coroutine-lifecycle)

### ✅ Good Code — `src/review_worker.lua`

```lua
local function resume_worker(worker)
  if coroutine.status(worker) == "dead" then
    return nil, "worker already finished"
  end

  -- resume returns success separately from yielded values.
  local ok, result = coroutine.resume(worker)
  if not ok then
    return nil, result
  end

  return result
end
```

### ❌ Bad Code — `review_worker.lua`

```lua
local function resume_worker(worker)
  -- The error value is treated like normal work output.
  local _, result = coroutine.resume(worker)
  return result
end
```

### 🔍 What to review

- **Good Code:** The good version checks whether the coroutine is already dead and keeps the `ok` flag from `resume`. Errors return as error values, not normal task output.
- **Bad Code:** The bad version ignores the success flag. A runtime error inside the coroutine can look like a normal yielded result.

### 📝 Review notes

For Lua coroutine code, review both lifecycle and resume result handling. A coroutine scheduler should make finished, yielded, and failed states distinct.

### 💡 Takeaway

> **Lua coroutine code should inspect resume success and status before assuming a task advanced.**

---

## 06. pcall error handling

> Wrap untrusted or plugin code with pcall and convert failures into typed results for the host app.

**Tags:** `pcall` `errors` `plugins` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/pcall-error-handling)

### ✅ Good Code — `src/plugin_runner.lua`

```lua
local function run_plugin(plugin, review)
  -- pcall keeps plugin errors from unwinding the host process.
  local ok, result = pcall(plugin.review, review)
  if not ok then
    return nil, { code = "plugin_error", message = result }
  end

  return result
end
```

### ❌ Bad Code — `plugin_runner.lua`

```lua
local function run_plugin(plugin, review)
  -- A plugin error escapes without host context.
  return plugin.review(review)
end
```

### 🔍 What to review

- **Good Code:** The good version catches plugin failures and turns them into a structured error table. The host can log the plugin name, disable it, or show a controlled message.
- **Bad Code:** The bad version calls plugin code directly. A plugin error escapes through the host path without enough context for recovery.

### 📝 Review notes

Use `pcall` at trust boundaries, not everywhere. Review whether the code adds context after catching the error so callers can decide what to do next.

### 💡 Takeaway

> **Lua host code should catch plugin failures at the boundary and return a result the caller can classify.**

---

## 07. Sandboxed execution

> Limit the environment for dynamic Lua chunks so scripts can use approved helpers without reaching host globals.

**Tags:** `sandbox` `environment` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/sandboxed-execution)

### ✅ Good Code — `src/rule_sandbox.lua`

```lua
local function compile_rule(source)
  local env = {
    tonumber = tonumber,
    math = { min = math.min, max = math.max },
  }

  -- The chunk sees only the helpers copied into env.
  return load(source, "review_rule", "t", env)
end
```

### ❌ Bad Code — `rule_sandbox.lua`

```lua
local function compile_rule(source)
  -- The chunk can read and change the host global environment.
  return load(source)
end
```

### 🔍 What to review

- **Good Code:** The good version gives the loaded chunk a small environment with only approved helpers. The script cannot reach unrelated globals by default.
- **Bad Code:** The bad version loads code into the default environment. A rule script can access global functions or mutate state the host never intended to expose.

### 📝 Review notes

Dynamic Lua execution is a security boundary. Review the environment table, the chunk mode, and which helpers are copied into the sandbox.

### 💡 Takeaway

> **Dynamic Lua execution should run with a small explicit environment instead of the full global table.**

---

## 08. Embedded API boundaries

> Keep host-provided APIs small and explicit when Lua scripts extend an application.

**Tags:** `embedded` `api` `host` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/embedded-api-boundaries)

### ✅ Good Code — `src/review_script_api.lua`

```lua
local api = {}

function api.add_note(review, text)
  -- The host API validates script input before mutating review state.
  assert(type(text) == "string", "note text must be a string")
  review.notes[#review.notes + 1] = text
end

return api
```

### ❌ Bad Code — `review_script_api.lua`

```lua
local api = {}

function api.expose_review(review)
  -- Scripts receive the full mutable host object.
  return review
end

return api
```

### 🔍 What to review

- **Good Code:** The good version exposes a narrow command and validates script input before changing host state.
- **Bad Code:** The bad version hands the whole host object to scripts. A script can mutate fields the host does not expect, making failures hard to trace.

### 📝 Review notes

Embedded Lua APIs stay reviewable when each exported function names one host action, validates script input, and limits the state it can mutate.

### 💡 Takeaway

> **An embedded Lua API should expose narrow commands rather than raw host objects with broad mutation power.**

---

## 09. Configuration validation

> Validate Lua configuration tables before using them to build routes, plugins, or runtime behavior.

**Tags:** `configuration` `validation` `tables` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/configuration-validation)

### ✅ Good Code — `src/config.lua`

```lua
local function read_config(config)
  assert(type(config) == "table", "config must be a table")
  assert(type(config.review_path) == "string", "review_path must be a string")

  -- Validated config returns a predictable shape to runtime code.
  return {
    review_path = config.review_path,
    max_reviews = tonumber(config.max_reviews) or 100,
  }
end
```

### ❌ Bad Code — `config.lua`

```lua
local function read_config(config)
  -- Runtime code receives whatever shape the config file happened to define.
  return config
end
```

### 🔍 What to review

- **Good Code:** The good version checks that the config is a table and that required fields have the expected type. Runtime code receives a normalized shape.
- **Bad Code:** The bad version passes the raw config table through. Typos and wrong types fail later, often far away from the file that introduced them.

### 📝 Review notes

Lua config is code, but it still needs a contract. Validate the table shape when loading config so runtime paths do not become type checks by accident.

### 💡 Takeaway

> **Lua config review should check required fields and value types at load time.**

---

## 10. Busted test cases

> Write Lua tests around observable module behavior instead of only checking that functions return truthy values.

**Tags:** `tests` `busted` `assertions` · [View on site](https://good-code-bad-code.pages.dev/tracks/lua/busted-test-cases)

### ✅ Good Code — `spec/review_rules_spec.lua`

```lua
describe("review rules", function()
  it("rejects a missing title", function()
    -- The fixture names the invalid input the module must reject.
    local ok, error = ReviewRules.validate({ title = nil, score = 5 })

    assert.is_false(ok)
    assert.are.equal("title_required", error.code)
  end)
end)
```

### ❌ Bad Code — `spec/review_rules_spec.lua`

```lua
describe("review rules", function()
  it("validates reviews", function()
    -- Truthy checks miss the actual validation contract.
    assert.is_truthy(ReviewRules.validate({}))
  end)
end)
```

### 🔍 What to review

- **Good Code:** The good version names the invalid input and asserts the exact error code. A regression in title validation will fail clearly.
- **Bad Code:** The bad version only checks for a truthy result. It does not protect the validation contract that callers rely on.

### 📝 Review notes

Lua tests should read like review notes for the module. Prefer one focused invalid input and a precise assertion over a broad truthy check.

### 💡 Takeaway

> **Lua tests should name the behavior, build a small fixture, and assert the exact output or error shape.**

---

[← Back to index](README.md)
