---
name: planner
description: >
  Use when the user wants to plan an implementation before writing code.
  Triggers on phrases like "plan this ticket", "write a plan for", "how
  should I approach", "break this down", "what's the implementation
  plan", or whenever the user provides a ticket/task and asks for a plan.
  Analyzes the task, detects the project's stack and conventions, and
  produces a step-by-step implementation plan with file-level detail.
  Does NOT write code — that is the executor skill's job.
license: MIT
---

# planner

Produces a concrete, file-level implementation plan from a ticket or task
description. Stops at the plan — never writes implementation code.

---

## When to Use

Fire this skill when:

- User pastes a ticket (Jira/Azure/Linear/GitHub Issue/plain text)
- User asks "how should I approach X" or "plan this feature"
- User explicitly invokes `/planner`
- A new feature or non-trivial change is about to start

Do NOT fire this skill for:

- Trivial single-line edits
- Bug fixes where the location is already known and the fix is obvious
- Questions about existing code (use a different skill or answer directly)

---

## Process

### Step 1 — Get the Ticket Context

Ask the user to provide ticket information using ANY method they prefer:

- Paste ticket title + description + acceptance criteria
- Share output from CLI tools (`az boards work-item show`, `jira issue
  view`, `gh issue view`, etc.)
- Share a ticket URL **with a summary they've copied**
- Or, if no ticket exists, ask them to describe the task in plain text

Do NOT assume any specific ticket system. Do NOT try to fetch tickets
yourself. Do NOT suggest installing MCP servers or CLI tools.

### Step 2 — Confirm the Essentials

Before planning, verify these are clear. If anything is missing or
ambiguous, ask ONE specific question and wait:

- **Objective** — what needs to exist after this is done
- **Acceptance criteria** — how we'll know it's done
- **Scope boundary** — what is explicitly NOT in scope
- **Constraints** — deadlines, dependencies, performance/security
  requirements, anything that shapes the approach

If the ticket says something ambiguous like *"improve user experience"*,
ask what specific behavior change is expected.

### Step 3 — Detect Project Context

Read the project to ground the plan in reality:

1. **Stack and framework version** — check `composer.json`, `go.mod`,
   `package.json`, etc. Identify the framework and major version.
2. **Folder structure** — note where similar features live. Mirror that.
3. **Existing patterns** — does the project use Service/Repository layers?
   Anonymous migrations? Pest or PHPUnit? Adapt the plan to fit.
4. **Existing related code** — find files that will likely need to change
   and list them by path.

Write code suggestions idiomatic to the **detected** stack and version. Do
not invent syntax. Do not push patterns that conflict with the project's
existing conventions.

### Step 4 — Produce the Plan

Output a plan with this structure:

```markdown
# Plan: <one-line title matching the ticket>

## Objective
<2–3 sentences. What and why.>

## Acceptance Criteria
- [ ] <criterion 1>
- [ ] <criterion 2>

## Out of Scope
- <thing 1>
- <thing 2>

## Approach
<One paragraph describing the strategy and why this approach over
alternatives. Mention any architectural decision and its trade-off.>

## Steps

### 1. <Step title>
**Files:** `path/to/file.ext` (new | modify)
**Change:** <what changes, briefly>
**Reason:** <why this step>

### 2. <Step title>
...

## Tests
- <test 1 — what behavior, which file>
- <test 2>

## Risks / Open Questions
- <risk or question 1>
- <risk or question 2>

## Estimated Effort
<S / M / L — and one sentence justifying>
```

### Step 5 — Stop and Wait

After producing the plan, STOP. Do not start implementing. Ask the user
to review the plan and confirm before any code is written.

Suggest: *"Let me know if any adjustments are needed. Once you approve,
run `/executor` to start the implementation."*

---

## Output Format

A single markdown document following the structure in Step 4. No code
blocks with implementation. No diffs. Only file paths + intent.

---

## Anti-patterns

- ❌ Writing actual implementation code in the plan (that is the
  executor's job)
- ❌ Skipping ambiguity — never invent missing acceptance criteria
- ❌ Generic plans that could apply to any project (always ground in the
  project's actual structure)
- ❌ Using framework patterns from a version the project does not run
- ❌ Suggesting refactors unrelated to the ticket scope
- ❌ Auto-chaining into the executor without explicit user approval
- ❌ Trying to fetch tickets via MCP/HTTP/CLI — always ask the user
