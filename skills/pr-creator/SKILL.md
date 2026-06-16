---
name: pr-creator
description: >
  Use when the user is ready to open a pull request after implementation
  and tests are done. Triggers on phrases like "create a PR", "open a
  pull request", "raise a PR", "ship this", or whenever the branch is
  ready and the user wants the PR opened. Inspects the branch's diff
  and commits, detects the project's PR conventions, ties the PR to a
  ticket if one exists, and opens the PR with a clean title, structured
  description, and a test plan. Runs pre-flight checks first.
license: MIT
---

# pr-creator

Opens a clean, well-structured pull request from the current branch.
Pulls ticket context from the user, derives the PR title and body from
the actual diff and commits, and runs pre-flight checks before pushing.

---

## When to Use

Fire this skill when:

- Implementation and tests are done on a feature branch
- User says "create a PR", "open a pull request", "raise a PR",
  "ship this", or explicitly invokes `/pr-creator`

Do NOT fire this skill when:

- The branch has no commits beyond the base branch (nothing to PR)
- Implementation is incomplete or tests are failing
- The user is mid-review of someone else's PR (use `/reviewer`)

---

## Process

### Step 1 — Verify Branch State

Run these checks and STOP if anything is off:

1. `git status` — there must be no uncommitted changes, or ask the
   user whether to commit them first
2. `git rev-parse --abbrev-ref HEAD` — confirm we're not on the base
   branch (`main` / `master` / `develop`)
3. Determine the base branch (usually `main`, but check repo defaults
   and existing PRs via `gh pr list --limit 1 --json baseRefName` if
   `gh` is available)
4. `git log <base>..HEAD --oneline` — confirm there's at least one
   commit to ship
5. `git diff <base>...HEAD --stat` — get the file change summary

### Step 2 — Get Ticket Context (Optional)

Ask the user if there is a ticket to link. They can provide it any way:

- Paste ticket title + description
- Share CLI output (`az boards work-item show`, `jira issue view`,
  `gh issue view`, etc.)
- Share a URL with a summary
- Or say "no ticket"

Never assume a specific ticket system. Never try to fetch tickets
yourself.

### Step 3 — Detect PR Conventions

Inspect the repo for existing patterns:

1. **PR template** — `.github/pull_request_template.md` or
   `docs/pull_request_template.md`. If present, USE IT.
2. **Recent PR titles** — `gh pr list --state all --limit 10 --json
   title,number`. Match the style (Conventional Commits, ticket-key
   prefix, plain, etc.).
3. **Branch naming** — already in the branch name, but useful to
   confirm scope
4. **Labels** — `gh label list` — apply matching labels if obvious

### Step 4 — Draft Title and Body

**Title** (keep under 70 characters):

- Match repo style. Examples:
  - `feat(account): allow self-service account deletion`
  - `[PROJ-412] Allow users to delete their account`
  - `Add self-service account deletion flow`
- Include the ticket key if the repo style uses one and a ticket
  exists

**Body** — if a PR template exists, fill it. Otherwise use:

```markdown
## Summary
<2–4 sentences. What changes, why, what user-visible behavior is
different.>

## Ticket
<URL or key, or "n/a">

## Changes
- <bullet — high-level change 1>
- <bullet — high-level change 2>
- <bullet — etc.>

## Test Plan
- [ ] <how to manually verify behavior 1>
- [ ] <how to verify behavior 2>
- [ ] Automated tests in `<path>` cover <what>

## Out of Scope
- <if applicable>

## Risks / Rollout Notes
- <migration consideration, feature flag, breaking change, etc.>
```

Derive bullets from actual commits and the diff. Do not pad. Do not
fabricate test plan steps that the changes don't actually allow.

### Step 5 — Pre-flight Checks

Before pushing, run whatever the project provides — if any of these
exist as scripts, run them:

- Linter / formatter (`./vendor/bin/pint --test`, `gofmt -l`,
  `eslint`, etc.)
- Type check (`phpstan`, `tsc --noEmit`)
- Fast test suite (`composer test`, `go test ./...`, `npm test`)

If any fail:

- Surface the failure to the user
- STOP — do not open a PR with red checks unless the user explicitly
  overrides

### Step 6 — Push and Open the PR

1. Push the branch with upstream tracking:
   ```bash
   git push -u origin <branch>
   ```
2. Open the PR via `gh`:
   ```bash
   gh pr create --base <base> --title "<title>" --body "$(cat <<'EOF'
   <body>
   EOF
   )"
   ```
3. Apply labels if detected as relevant:
   ```bash
   gh pr edit <number> --add-label "<label>"
   ```
4. Return the PR URL to the user

### Step 7 — Hand Off

Report:

- PR URL
- Title and final body used
- Labels applied
- Any pre-flight check warnings (non-blocking ones)
- Suggested next: ask a teammate (or run `/reviewer`) for a review pass

---

## Output Format

```
### Pre-flight
- Linter: pass | fail
- Types:  pass | fail
- Tests:  pass | fail

### PR Opened

URL:    <url>
Title:  <title>
Base:   <branch>
Branch: <branch>
Labels: <labels>

### Body Used

<full PR body>
```

---

## Anti-patterns

- ❌ Opening a PR with failing pre-flight checks (unless user
  explicitly overrides)
- ❌ Padding the PR body with sections that don't apply
- ❌ Fabricating a test plan that the diff doesn't actually support
- ❌ Ignoring the repo's `pull_request_template.md`
- ❌ Using a title style that doesn't match recent PRs
- ❌ Skipping the ticket link when the repo convention is to include
  one
- ❌ Auto-merging or assigning reviewers without explicit user request
- ❌ Trying to fetch ticket data via MCP / HTTP / CLI — always ask the
  user for ticket context
- ❌ Force-pushing or rewriting history during PR creation
