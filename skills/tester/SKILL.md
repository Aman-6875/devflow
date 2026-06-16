---
name: tester
description: >
  Use when the user wants to add tests — for newly written code, for an
  existing untested function, or for a ticket's acceptance criteria.
  Triggers on phrases like "write tests for", "add test coverage",
  "test this", "cover the AC", or whenever a ticket includes testable
  acceptance criteria and the user asks for the test suite. Detects
  the project's existing test framework and conventions, writes tests
  that target behavior (not implementation details), and ensures one
  clear assertion focus per test. Does NOT modify production code.
license: MIT
---

# tester

Writes behavior-focused tests for code or for a ticket's acceptance
criteria. Uses whatever test framework the project already has.

---

## When to Use

Fire this skill when:

- New code has been implemented (typically after `/executor`) and
  needs test coverage
- A ticket has acceptance criteria and the user wants AC-driven tests
- An existing function/endpoint is untested and the user wants
  coverage added
- User explicitly invokes `/tester`

Do NOT fire this skill when:

- The user is asking for help debugging a *failing* test (different
  task)
- The user wants production code written (use `/executor` instead)
- The project has no test setup at all and the user has not asked you
  to set one up

---

## Process

### Step 1 — Gather Input

You need one or both of:

- **The code under test** — file paths the user mentions, or the
  recent diff
- **The acceptance criteria** — paste from a ticket, or plain-text
  description from the user

If a ticket exists, ask the user to share it however they prefer
(paste, CLI output, URL with summary, etc.). Never assume a specific
ticket system or try to fetch it yourself.

If neither code nor AC is provided, ask which one the user wants to
test against.

### Step 2 — Detect Test Framework and Conventions

Inspect the project to learn:

1. **Test framework**:
   - Laravel: `pest` in `composer.json` → Pest; otherwise PHPUnit
   - Go: `*_test.go` files with `testing` package — table-driven or
     not, check existing style
   - Node: `jest.config.*`, `vitest.config.*`, or `package.json`
     scripts → Jest / Vitest / Mocha
   - NestJS: usually Jest with `@nestjs/testing`
2. **Test folder layout** — mirror it (`tests/Feature/`,
   `tests/Unit/`, `internal/foo/foo_test.go`, `__tests__/`, etc.)
3. **Naming convention** — `it should ... when ...`, `Test_Foo`,
   `describe/it`, etc. Match it exactly.
4. **Test helpers / factories** — model factories in Laravel,
   `testify` in Go, MSW in Node — use what the project uses.
5. **Setup / teardown patterns** — `RefreshDatabase`, fixtures,
   in-memory DB, Docker-Compose-backed integration setup, etc.

Write tests idiomatic to the **detected** framework and version. Do not
introduce a new test library.

### Step 3 — Map AC and Behavior to Test Cases

For each acceptance criterion or behavior to test, plan one (or a
small cluster of) test cases. Each test must:

- Target **behavior**, not implementation details
- Have **one assertion focus** (a single conceptual thing under test)
- Have a name in the format `it should <behavior> when <condition>`
  (or the project's equivalent)
- Mock external I/O (HTTP, queue, mail, third-party SDKs); do NOT
  mock the system under test or its direct collaborators if the
  project favors integration over unit testing

Before writing any test, list the planned test cases briefly so the
user can confirm coverage matches what they expect.

### Step 4 — Write the Tests

For each planned case:

1. Place the file at the path matching project layout
2. Use existing factories / fixtures
3. Arrange / Act / Assert — visually separated (blank lines between
   sections is fine)
4. Assert observable outcomes (response status, DB row state, emitted
   event, return value), not internal calls

Cover both:

- **Happy path** — primary behavior described by the AC
- **Edge cases** — invalid input, unauthorized access, conflicting
  state, boundary values

If a critical edge case isn't covered by the AC, surface it and ask
whether to add a test.

### Step 5 — Run the Tests

Run the tests and confirm they pass (or, for new behavior the executor
hasn't written yet, that they fail for the right reason — TDD style).

If a test fails unexpectedly:

- Investigate root cause; do NOT loosen the test to make it pass
- If the production code has a real bug, surface it — that may need
  `/executor` to fix
- If the test is wrong, fix the test

### Step 6 — Summarize

Report:

- Test files created or modified
- Number of tests added
- What was NOT covered and why
- Command to run the suite

---

## Output Format

```
### Test Plan

For AC / behavior: <summary>

Cases:
- it should <behavior> when <condition>
- ...

### Tests Written

Files:
- <path> (created | modified)

Cases added: <N>
All passing: yes / no (with details if no)

### Not Covered (and Why)
- <case> — <reason>

### Run

    <project's test command>
```

---

## Anti-patterns

- ❌ Introducing a new test framework instead of using what's already
  there
- ❌ Testing implementation details (e.g. `expect(spy).toHaveBeenCalled`
  on internal helpers) instead of observable behavior
- ❌ Multiple unrelated assertions in one test
- ❌ Skipping edge cases because they "aren't in the AC" — surface
  them, don't silently omit
- ❌ Mocking the database or the system under test (project
  conventions usually want real DB in feature tests)
- ❌ Loosening a failing test to make it pass
- ❌ Modifying production code in this skill — surface the bug, hand
  off to `/executor`
- ❌ Generating snapshot tests by default — they encode
  implementation, not behavior
