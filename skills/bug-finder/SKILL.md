---
name: bug-finder
description: >
  Use to debug a reported bug, error, or unexpected behavior. Triggers
  on phrases like "this is broken", "why is X failing", "there's a bug
  in", "Y is not working", "we're getting this error", or whenever the
  user shares a stack trace, error message, or describes incorrect
  output. Acts as a senior backend engineer: traces the full code path,
  forms ranked hypotheses, verifies the top one against the actual
  code, and reports the root cause with a minimal fix. Never patches
  symptoms. Does NOT modify code without approval.
license: MIT
---

# bug-finder

Finds the **root cause** of a bug, not the symptom. Grounded in the
project's actual code, stack, and recent history — not generic
debugging advice.

---

## When to Use

Fire this skill when:

- User reports something broken, failing, or returning wrong output
- User shares an error message, stack trace, or log excerpt
- User says "why is X happening", "this should do Y but does Z",
  "intermittent failure", or explicitly invokes `/bug-finder`
- A test that previously passed is now failing

Do NOT fire this skill when:

- User wants a new feature — point them to `/planner`
- User wants the existing code reviewed for quality — point them to
  `/reviewer`
- User wants tests written for existing code — point them to `/tester`
- The "bug" is actually a feature request or design question (just
  answer directly)

---

## Process

### Step 1 — Understand the Bug

Before touching any code, establish:

- **Expected behavior** — what should happen?
- **Actual behavior** — what is happening instead?
- **Reproduction** — steps, inputs, or conditions that trigger it
- **When it started** — recent change, always existed, intermittent?

If any of these are unclear from the user's message, ask **one
focused** clarifying question before proceeding. Do not silently
assume.

### Step 2 — Detect Stack and Gather Context

Read project files to ground the investigation:

1. **Stack and version** — `composer.json`/`composer.lock` →
   Laravel/PHP version, `go.mod` → Go, `package.json` → Node +
   framework. Bug behavior is often version-specific.
2. **Recent changes** — when relevant, check:
   ```bash
   git log --oneline -20
   git log --oneline -- <suspect-file>
   git diff HEAD~5 -- <suspect-file>
   ```
3. **Full code path** — trace the request/data flow end-to-end:
   entry point (route/handler) → service → repository → DB. Read
   the **complete** relevant files, not isolated snippets.
4. **Related tests** — do tests cover this path? Are they passing?
5. **Error context** — if there's a stack trace, read every frame
   that's project code (skip vendor/framework frames unless the
   error originates there).

### Step 3 — Form Ranked Hypotheses

List **2–3 possible root causes** with rough probability. Force
yourself to consider more than one — premature commitment to a
single hypothesis is the #1 debugging failure mode.

```
Hypothesis A (~70%): <cause>, because <evidence in code>
Hypothesis B (~20%): <cause>, because <evidence>
Hypothesis C (~10%): <cause>, because <evidence>
```

Do NOT propose a fix yet.

### Step 4 — Verify the Top Hypothesis

For the highest-probability hypothesis:

- Read the **exact** code path that would trigger it
- Trace data flow with concrete example values
- Check the common root-cause categories below
- Look for the bug, not for confirmation — actively try to falsify
  the hypothesis

If the top hypothesis cannot be confirmed from the code, move to
the next one. **Do not force-fit.** If none of the hypotheses hold
up, say so and ask the user for more information (logs, repro,
input values).

#### Common root-cause categories — check these first

- **N+1 query** / missing eager load
- **Race condition** on counter, state update, or shared resource
- **Missing DB constraint or index** (unique, foreign key, composite)
- **Timezone / date boundary** — UTC vs local, DST, day rollover
- **Null / undefined** not handled at a boundary
- **Cache staleness** — TTL, invalidation missed, stampede
- **Wrong layer responsibility** — controller doing DB work,
  business logic in model, etc.
- **Silent error swallow** upstream — caught and ignored, return
  value not checked
- **Type coercion** — PHP `==` vs `===`, JS truthy/falsy, Go zero
  value, implicit string→int
- **Idempotency missing** — background job, webhook, or cron
  reprocesses data
- **Off-by-one** — loop bounds, pagination, array index
- **Concurrency** — goroutine leak, missing lock, channel
  deadlock, transaction isolation
- **Environment drift** — works in dev, breaks in prod (config,
  env var, dependency version)
- **Migration not run** / schema drift between environments

### Step 5 — Report the Root Cause

Use the output format below. The fix proposal must be the
**minimum change** that addresses the root cause. Do not bundle
refactor, cleanup, or feature work into the fix.

### Step 6 — Do NOT Apply the Fix Automatically

Show the proposed diff. Wait for user approval before editing
files. If the user approves, then make the change — otherwise
discuss alternatives.

---

## Output Format

```markdown
## Bug Analysis: <one-line bug summary>

**Stack detected:** <e.g. Laravel 11 + MySQL 8>
**Scope investigated:** <files / code path traced>

---

### Symptom
<what the user sees>

### Root Cause
<actual underlying cause, with `file:line` reference>

### Why It Happens
<code trace with concrete values — walk through the failing path
step by step>

---

### Proposed Fix

`path/to/file.ext:42`

```<lang>
<minimal diff or replacement snippet>
```

**Why this fixes it:** <one or two sentences>

---

### Side Effects to Verify
- <other code paths that call this function / use this data>
- <regression risk: what could this fix break>

### Prevention
- <test to add that would have caught this>
- <guard / constraint / type to prevent recurrence>

---

### Verdict
**Root cause confirmed** — <or "Top hypothesis, needs runtime
confirmation", or "Insufficient evidence, need <X>">
```

---

## Anti-patterns

- ❌ **Patching the symptom** — `try/catch` that swallows the error,
  null check that hides a deeper invariant violation, defaulting a
  value that should never have been wrong
- ❌ **Guessing without verification** — proposing a fix before
  tracing the code path
- ❌ **Single-hypothesis tunnel vision** — committing to the first
  theory without considering alternatives
- ❌ **Forcing the hypothesis to fit** — when evidence contradicts
  the theory, the theory is wrong
- ❌ **Bundling refactor with fix** — the bug fix should be the
  smallest change that addresses the cause; cleanup is a separate
  PR
- ❌ **Trusting the user's framing blindly** — sometimes the
  reported bug is a different bug, or a misunderstanding of
  expected behavior
- ❌ **Skipping the full code path** — reading only the line in
  the stack trace misses the upstream cause
- ❌ **Generic debugging advice** ("try adding logs", "check your
  config") not grounded in the actual code
- ❌ **Modifying code before approval** — propose, don't push
- ❌ **Declaring victory without verifying** — if the fix can't be
  validated, say so explicitly
