---
name: executor
description: >
  Use after a plan has been approved (typically from the planner skill)
  and the user is ready to implement. Triggers on phrases like "execute
  the plan", "implement this", "now build it", "start coding", or
  whenever the user references an approved plan and asks for code.
  Detects the project's stack and framework version, follows the plan
  step-by-step, writes idiomatic code, and commits in logical chunks.
  Does NOT design — that is the planner's job. Does NOT write tests as
  the primary deliverable — that is the tester's job.
license: MIT
---

# executor

Implements code from an approved plan. Stack-aware, version-aware,
convention-aware. Operates in small, reviewable steps.

---

## When to Use

Fire this skill when:

- A plan exists (from `/planner` or written by the user) and the user
  wants implementation to begin
- User says "implement this", "build it", "execute the plan", "start
  coding", or explicitly invokes `/executor`

Do NOT fire this skill when:

- No plan exists and the task is non-trivial — call `/planner` first
- The user is mid-debugging — they likely want investigation, not bulk
  implementation
- The task is a single trivial edit — just do the edit directly

---

## Process

### Step 1 — Locate the Approved Plan

The plan can come from:

- The current conversation (output of `/planner` earlier in the session)
- A file the user points to
- Plain text the user pastes

If no plan is present, STOP and tell the user to run `/planner` first.
Do not improvise a plan inside the executor.

### Step 2 — Detect Project Stack and Version

Read project root files to identify:

1. **Stack**:
   - `composer.json` → PHP / Laravel
   - `go.mod` → Go
   - `package.json` → Node / TypeScript (inspect dependencies for
     NestJS, Express, Next.js, etc.)
   - Mixed or unclear → ask the user which target the plan addresses

2. **Framework version** — read the lockfile (`composer.lock`,
   `go.sum`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`) for
   the exact installed version. Fall back to the manifest constraint
   if no lockfile exists.

3. **Conventions in use** — scan a few existing files in the relevant
   layer to identify:
   - Naming and folder structure
   - Use of layered architecture (Controller / Service / Repository,
     Handler / Service / Repository, etc.)
   - Test framework (Pest vs PHPUnit, table-driven Go tests, Jest vs
     Vitest, etc.)
   - Linter / formatter config (Pint, gofmt, ESLint, Prettier)

Write code idiomatic to the **detected** stack and version. Do not
invent syntax. Do not use deprecated patterns from older versions, and
do not use features unavailable in the detected version.

### Step 3 — Implement Step-by-Step

Work through the plan in order. For each plan step:

1. State which step you're starting ("Step 2: implementing
   `DeleteAccountAction`").
2. Make the file changes (create or modify) using the editor tool.
3. Briefly note what you did and any deviation from the plan, with
   reason.
4. After each logical unit (one plan step or a tight cluster of
   related changes), pause to let the user review before continuing.

If you discover the plan is wrong or incomplete mid-execution:

- STOP implementing
- Surface the issue clearly with what you found and why the plan needs
  to change
- Ask the user to update the plan (or update it together)
- Do not silently deviate

### Step 4 — Commit in Logical Chunks

After each plan step (or related cluster) is implemented and verified:

- Stage only the files for that step
- Write a commit message in the project's existing style (inspect
  `git log` to match — Conventional Commits, prefixed scopes, plain
  imperative, etc.)
- Reference the ticket key if one exists (e.g. `PROJ-412`)

Do not bundle unrelated steps into one commit.

### Step 5 — Run Project Checks

Before declaring done, run whatever the project provides:

- Linter / formatter (`./vendor/bin/pint`, `gofmt`, `eslint`, etc.)
- Type checker if applicable (`phpstan`, `tsc --noEmit`)
- The fast test suite (do NOT add new tests here — that's the tester
  skill's job, but existing tests must pass)

Fix anything that breaks. Do not declare done with red checks.

### Step 6 — Hand Off

Summarize:

- Files created / modified (with paths)
- Commits made
- What the user should verify manually
- Suggested next step: usually `/tester` to add tests, then
  `/pr-creator` to open the PR

---

## Output Format

For each step:

```
### Step <N>: <title from plan>

Files:
- <path> (created | modified)
- ...

Notes:
- <anything noteworthy, deviation, assumption>

Commit: "<commit message used>"
```

At the end, a brief summary with the suggested next skill.

---

## Anti-patterns

- ❌ Improvising a plan inside the executor (call `/planner` instead)
- ❌ Writing code for a framework version different from what's
  installed
- ❌ Introducing patterns or libraries not used elsewhere in the project
- ❌ Silently deviating from the plan — always surface and ask
- ❌ Bundling unrelated changes into one commit
- ❌ Skipping linter / type-check / existing tests before hand-off
- ❌ Writing extensive new tests in this skill — that is the tester
  skill's job (a quick smoke test inline is fine, but the test suite
  belongs to `/tester`)
- ❌ Auto-chaining into `/tester` or `/pr-creator` without explicit
  user request
