# Git — Code Review Handbook

> Review status checks, staging, commits, branches, sync workflows, diffs, conflicts, history safety, ignores, and release tags.

- **Category:** Foundations
- **Lessons:** 10
- **Source:** [https://good-code-bad-code.pages.dev/tracks/git](https://good-code-bad-code.pages.dev/tracks/git)
- **Related tracks:** [SQL](11-sql.md), [Docker](26-docker.md)

[← Back to index](README.md)

## Review Checklist

ใช้เช็กลิสต์นี้ตอนรีวิวโค้ด / เปิด PR — แต่ละข้อลิงก์ไปยังบทเรียนที่อธิบายเหตุผลและตัวอย่าง

- [ ] **[Status before work](#01-status-before-work)** — A good Git workflow begins by making the current branch and uncommitted changes visible before editing files.
- [ ] **[Interactive staging hunks](#02-interactive-staging-hunks)** — Reviewable commits come from reviewable staging, not from adding every file by habit.
- [ ] **[Atomic commits and messages](#03-atomic-commits-and-messages)** — A commit should describe one coherent change and make the reason visible without reading the whole diff first.
- [ ] **[Branch naming and upstream](#04-branch-naming-and-upstream)** — A branch should tell the team what kind of work it contains and where it syncs by default.
- [ ] **[Fetch, pull, and rebase sync](#05-fetch-pull-and-rebase-sync)** — Sync commands should show what changed upstream before they rewrite or merge local history.
- [ ] **[Reviewing diffs before push](#06-reviewing-diffs-before-push)** — A push should come after a local review pass, not be the first time anyone sees the final diff.
- [ ] **[Conflict resolution without overwriting](#07-conflict-resolution-without-overwriting)** — A conflict is a design review moment; choosing ours or theirs blindly can delete another person's intent.
- [ ] **[Revert vs reset](#08-revert-vs-reset)** — History commands should match whether the commits are already shared with other people.
- [ ] **[.gitignore and secret safety](#09-gitignore-and-secret-safety)** — A repository should make accidental secret and build artifact commits hard to create and easy to catch.
- [ ] **[Tags and release points](#10-tags-and-release-points)** — A release tag should point to a verified commit and carry a clear versioned meaning.

## Lessons

| # | Lesson | Tags |
|---|--------|------|
| 01 | [Status before work](#01-status-before-work) | `status` `workflow` `safety` |
| 02 | [Interactive staging hunks](#02-interactive-staging-hunks) | `staging` `patches` `commits` |
| 03 | [Atomic commits and messages](#03-atomic-commits-and-messages) | `commits` `messages` `history` |
| 04 | [Branch naming and upstream](#04-branch-naming-and-upstream) | `branches` `upstream` `collaboration` |
| 05 | [Fetch, pull, and rebase sync](#05-fetch-pull-and-rebase-sync) | `fetch` `pull` `rebase` |
| 06 | [Reviewing diffs before push](#06-reviewing-diffs-before-push) | `diff` `push` `review` |
| 07 | [Conflict resolution without overwriting](#07-conflict-resolution-without-overwriting) | `conflicts` `merge` `rebase` |
| 08 | [Revert vs reset](#08-revert-vs-reset) | `revert` `reset` `history` |
| 09 | [.gitignore and secret safety](#09-gitignore-and-secret-safety) | `gitignore` `secrets` `security` |
| 10 | [Tags and release points](#10-tags-and-release-points) | `tags` `release` `versioning` |

---

## 01. Status before work

> Start every change by checking branch, cleanliness, and upstream state so you do not build on the wrong base.

**Tags:** `status` `workflow` `safety` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/status-before-work)

### ✅ Good Code — `review-start.sh`

```bash
# Confirm branch and local changes before editing.
git status --short --branch
git fetch origin
git status --short --branch
git diff --stat
```

### ❌ Bad Code — `review-start.sh`

```bash
# Pulling first can hide which base changed under you.
git pull
code src/app.ts
git add .
git commit -m "updates"
```

### 🔍 What to review

- **Good Code:** The good version checks the current branch, working tree, remote state, and visible diff before making a decision.
- **Bad Code:** The bad version pulls first, starts editing, and commits everything without proving which branch it is on or what changed.

### 📝 Review notes

When reviewing Git workflow, start with observability. If a developer cannot say what branch they are on and whether the tree is clean, later Git commands become harder to trust.

### 💡 Takeaway

> **A good Git workflow begins by making the current branch and uncommitted changes visible before editing files.**

---

## 02. Interactive staging hunks

> Stage changes deliberately so each commit contains only the hunks that belong together.

**Tags:** `staging` `patches` `commits` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/interactive-staging-hunks)

### ✅ Good Code — `stage-review.sh`

```bash
# Inspect first, then stage only related hunks.
git diff
git add -p src/review.ts
git diff --staged
git commit -m "fix review navigation labels"
```

### ❌ Bad Code — `stage-review.sh`

```bash
# This stages unrelated edits, generated files, and debug changes.
git add .
git commit -m "fix stuff"
git push
```

### 🔍 What to review

- **Good Code:** The good version inspects the unstaged diff, stages only relevant hunks, then reviews the staged diff before committing.
- **Bad Code:** The bad version adds unrelated files and gives reviewers no signal about the intent of the change.

### 📝 Review notes

`git add -p` is especially useful when formatting, debugging, and feature work are mixed locally. It helps the commit tell one story.

### 💡 Takeaway

> **Reviewable commits come from reviewable staging, not from adding every file by habit.**

---

## 03. Atomic commits and messages

> Keep commits small enough to review and give each one a message that explains the user-facing or maintenance reason.

**Tags:** `commits` `messages` `history` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/atomic-commits-messages)

### ✅ Good Code — `commit-plan.sh`

```bash
# One commit explains one behavior change.
git add components/lesson/lesson-pager.tsx
git commit -m "improve lesson navigation controls"

# A separate commit keeps test intent easy to review.
git add tests/content/navigation.test.ts
git commit -m "cover lesson navigation boundaries"
```

### ❌ Bad Code — `commit-plan.sh`

```bash
# Reviewers cannot tell which concern this commit owns.
git add .
git commit -m "work"
git push
```

### 🔍 What to review

- **Good Code:** The good version separates UI behavior from test coverage and gives each commit a message that explains the change.
- **Bad Code:** The bad version creates a grab-bag commit. Reviewers have to understand unrelated edits at once, and reverting one part becomes riskier.

### 📝 Review notes

Atomic does not mean tiny for its own sake. It means the commit has one reason to exist and can be reviewed, reverted, or cherry-picked as a unit.

### 💡 Takeaway

> **A commit should describe one coherent change and make the reason visible without reading the whole diff first.**

---

## 04. Branch naming and upstream

> Create topic branches with clear names and set upstream tracking so future pushes and pulls target the right remote branch.

**Tags:** `branches` `upstream` `collaboration` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/branch-naming-upstream)

### ✅ Good Code — `branch-start.sh`

```bash
# Topic branch name tells teammates what this work is about.
git switch -c feature/git-track-lessons
git push -u origin feature/git-track-lessons
git status --short --branch
```

### ❌ Bad Code — `branch-start.sh`

```bash
# Committing directly on main raises the cost of mistakes.
git switch main
git commit -am "changes"
git push origin HEAD
```

### 🔍 What to review

- **Good Code:** The good version creates a named topic branch, publishes it with upstream tracking, and checks the branch state.
- **Bad Code:** The bad version commits on `main` and pushes `HEAD` without a clear target. That makes accidental pushes harder to spot in review.

### 📝 Review notes

Branch names are lightweight documentation. They should help a reviewer understand whether the work is a feature, fix, chore, spike, or release change.

### 💡 Takeaway

> **A branch should tell the team what kind of work it contains and where it syncs by default.**

---

## 05. Fetch, pull, and rebase sync

> Fetch before integrating upstream work, then choose merge or rebase intentionally for the branch you are on.

**Tags:** `fetch` `pull` `rebase` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/fetch-pull-rebase-sync)

### ✅ Good Code — `sync-feature.sh`

```bash
# Fetch makes upstream changes visible before integration.
git fetch origin
git log --oneline --decorate --max-count=5 HEAD..origin/main
git rebase origin/main
npm test
```

### ❌ Bad Code — `sync-feature.sh`

```bash
# Pulling without inspecting upstream makes the integration implicit.
git pull
git push --force
git status
```

### 🔍 What to review

- **Good Code:** The good version fetches first, inspects upstream commits, rebases the feature branch intentionally, and runs tests after integration.
- **Bad Code:** The bad version lets `pull` decide the integration behavior and follows with a force push without checking what changed.

### 📝 Review notes

`git fetch` is the safe observation step. In review, ask whether the command sequence makes upstream changes visible before it changes local history.

### 💡 Takeaway

> **Sync commands should show what changed upstream before they rewrite or merge local history.**

---

## 06. Reviewing diffs before push

> Inspect unstaged, staged, and committed changes before pushing so accidental files and debug edits do not leave your machine.

**Tags:** `diff` `push` `review` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/reviewing-diffs-before-push)

### ✅ Good Code — `pre-push-review.sh`

```bash
# Check both file content and the commits about to leave the branch.
git diff --check
git diff
git diff --staged
git log --oneline origin/main..HEAD
git push
```

### ❌ Bad Code — `pre-push-review.sh`

```bash
# The first real review happens after the push, which is too late.
git add .
git commit -m "final"
git push
```

### 🔍 What to review

- **Good Code:** The good version checks whitespace issues, reviews both unstaged and staged changes, and confirms which commits are about to be pushed.
- **Bad Code:** The bad version sends the final state upstream without a local review pass.

### 📝 Review notes

Before pushing, reviewers should be able to ask "what exactly leaves this branch?" The answer should come from `diff` and `log`, not memory.

### 💡 Takeaway

> **A push should come after a local review pass, not be the first time anyone sees the final diff.**

---

## 07. Conflict resolution without overwriting

> Resolve conflicts by understanding both sides, then stage only the corrected files instead of blindly taking one version.

**Tags:** `conflicts` `merge` `rebase` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/conflict-resolution-without-overwriting)

### ✅ Good Code — `resolve-conflict.sh`

```bash
# Read the conflicted file before choosing a resolution.
git status
git diff --merge src/lesson.ts
code src/lesson.ts
npm test
git add src/lesson.ts
git rebase --continue
```

### ❌ Bad Code — `resolve-conflict.sh`

```bash
# Taking one side for every file can erase local intent.
git checkout --theirs .
git add .
git rebase --continue
git push --force
```

### 🔍 What to review

- **Good Code:** The good version identifies conflicted files, compares both sides, edits the resolved file, runs tests, and stages only the resolution.
- **Bad Code:** The bad version takes the other side for every file and force-pushes, which can silently discard local work or domain decisions.

### 📝 Review notes

Conflict resolution should preserve intent from both branches. If the command sequence never opens the conflicted file, it is probably too mechanical.

### 💡 Takeaway

> **A conflict is a design review moment; choosing ours or theirs blindly can delete another person's intent.**

---

## 08. Revert vs reset

> Use revert for shared history and reserve reset for local cleanup before commits leave your machine.

**Tags:** `revert` `reset` `history` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/revert-vs-reset)

### ✅ Good Code — `undo-shared-change.sh`

```bash
# Revert preserves shared history while undoing behavior.
git fetch origin
git switch main
git pull --ff-only
git revert 4f3a2b1
git push
```

### ❌ Bad Code — `undo-shared-change.sh`

```bash
# Resetting shared main rewrites history for everyone else.
git switch main
git reset --hard HEAD~1
git push --force
```

### 🔍 What to review

- **Good Code:** The good version creates a new commit that undoes a shared change, preserving history that teammates may already have.
- **Bad Code:** The bad version rewrites shared history on `main`, forcing everyone else to reconcile their local repositories.

### 📝 Review notes

The first review question for undo commands is simple: has this commit been pushed? If yes, prefer `revert` unless the team has explicitly agreed to rewrite history.

### 💡 Takeaway

> **History commands should match whether the commits are already shared with other people.**

---

## 09. .gitignore and secret safety

> Keep generated files and secrets out of version control, and remove tracked secrets from the index immediately.

**Tags:** `gitignore` `secrets` `security` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/gitignore-and-secret-safety)

### ✅ Good Code — `secret-safety.sh`

```bash
# Prevent local secrets and generated output from being staged.
printf ".env\n.env.local\ndist/\nnode_modules/\n" >> .gitignore
git rm --cached .env
git status --short
git add .gitignore
git commit -m "ignore local secrets and build output"
```

### ❌ Bad Code — `secret-safety.sh`

```bash
# Adding everything can publish secrets or local machine state.
git add .
git commit -m "add config"
git push
```

### 🔍 What to review

- **Good Code:** The good version ignores local secret files and removes an already-tracked secret from the index before committing.
- **Bad Code:** The bad version stages every file, which can include credentials, generated artifacts, editor files, or local environment settings.

### 📝 Review notes

`.gitignore` is prevention, not cleanup after the fact. If a secret was committed, removing it from a later commit is not enough; the team may need key rotation and history cleanup.

### 💡 Takeaway

> **A repository should make accidental secret and build artifact commits hard to create and easy to catch.**

---

## 10. Tags and release points

> Create intentional annotated tags for release points after verifying the exact commit that should ship.

**Tags:** `tags` `release` `versioning` · [View on site](https://good-code-bad-code.pages.dev/tracks/git/tags-and-release-points)

### ✅ Good Code — `release-tag.sh`

```bash
# Verify the exact commit before creating the release tag.
git status --short --branch
npm test
git log --oneline --max-count=1
git tag -a v1.4.0 -m "Release v1.4.0"
git push origin v1.4.0
```

### ❌ Bad Code — `release-tag.sh`

```bash
# Vague tags and bulk pushes can publish accidental release points.
git tag latest
git push --tags
```

### 🔍 What to review

- **Good Code:** The good version verifies the working tree, runs tests, checks the exact commit, creates an annotated version tag, and pushes only that tag.
- **Bad Code:** The bad version creates a vague moving name and pushes every local tag, including tags that may not be ready to publish.

### 📝 Review notes

Tags are part of the release contract. Review the commit they point at, the tag name, and whether the push command publishes only the intended release.

### 💡 Takeaway

> **A release tag should point to a verified commit and carry a clear versioned meaning.**

---

[← Back to index](README.md)
