# Review Checklist

A structured lens for the `/reviewer` skill. Walk the diff with this
list, but only raise findings that actually apply. Don't enumerate
the checklist in the output.

---

## Correctness

- [ ] Does the change implement what the ticket / PR description says?
- [ ] Edge cases handled — empty inputs, null, max bounds, concurrency,
      race conditions
- [ ] Error paths return / throw correctly; no silent swallows
- [ ] No off-by-one, wrong operator, inverted condition
- [ ] Idempotency where it matters (retries, queue jobs, webhooks)
- [ ] Time zones / timestamp handling explicit (UTC vs local)

## Security

- [ ] Auth required where it should be (`auth` middleware, guard,
      authorization policy)
- [ ] Authorization (not just authentication) — does the user own /
      have permission for the resource?
- [ ] Input validated (FormRequest, DTO, schema) — never trust the
      client
- [ ] Mass assignment guarded (`$fillable`, explicit DTO mapping)
- [ ] SQL/NoSQL injection — parameterized queries, no string
      interpolation into queries
- [ ] XSS — output escaping where rendered to a browser
- [ ] CSRF on state-changing endpoints (for browser-facing apps)
- [ ] Secrets never committed; not logged; not in error responses
- [ ] Sensitive data not logged (PII, tokens, passwords, full card
      numbers)
- [ ] Rate limiting on auth / abuse-prone endpoints
- [ ] File uploads validated (type, size, storage path traversal)
- [ ] Redirects validated against open-redirect vectors

## Performance

- [ ] No N+1 queries — eager-load relationships where used
- [ ] Indexes exist on columns the new queries filter / join on
- [ ] Large dataset processing uses chunking / streaming, not full
      hydration
- [ ] Cache used where the data is read-heavy and stable
- [ ] Long-running work is queued, not synchronous in the request path
- [ ] No accidental quadratic loops over user-controlled data
- [ ] HTTP calls have timeouts and retry/backoff where appropriate

## Design (SOLID + boundaries)

- [ ] Single responsibility — each class / function does one thing
- [ ] Layer respect — controller doesn't query DB directly; service
      doesn't render HTTP responses
- [ ] Dependencies injected; no `new SomeClass()` for collaborators
      that should be DI-managed
- [ ] Depends on interfaces / contracts, not concrete implementations,
      where the codebase uses that pattern
- [ ] Composition over inheritance unless inheritance genuinely models
      "is-a"
- [ ] No god classes / multi-hundred-line functions introduced
- [ ] Early returns over deep nesting

## Project Conventions

- [ ] Naming matches the project (file names, class names, function
      names, DB columns)
- [ ] Folder placement matches similar features
- [ ] Layered architecture used the way the rest of the project uses
      it (Service / Repository / Action / Handler — whatever applies)
- [ ] Error handling style consistent (custom exceptions vs Result
      types vs error returns)
- [ ] Logging style consistent (structured fields, context)
- [ ] Existing helpers / facades / utilities reused — not reimplemented

## Database

- [ ] Migration present for schema changes
- [ ] Migration is reversible (`down()` implemented) or marked
      irreversible deliberately
- [ ] Foreign keys present and indexed
- [ ] Nullable / default values match the application's assumptions
- [ ] No data-loss-prone migration without explicit acknowledgement
      (e.g., `dropColumn` on a populated column)
- [ ] No raw SQL where the ORM / query builder would do
- [ ] Soft delete used consistently if the table is soft-deletable

## Tests

- [ ] New behavior has tests
- [ ] Tests assert observable behavior, not internal calls
- [ ] One assertion focus per test
- [ ] Edge cases covered, not just the happy path
- [ ] External I/O mocked (HTTP, queue, mail, third-party SDK)
- [ ] Database touched where the project favors feature/integration
      tests
- [ ] Test names follow the project's pattern
      (`it should ... when ...`, `Test_Foo`, etc.)
- [ ] No flaky time / random / network dependence

## Readability

- [ ] Names communicate intent — no `tmp`, `data2`, `doStuff`
- [ ] Comments explain *why*, not *what*
- [ ] Dead code removed (no leftover debug prints, commented-out
      blocks)
- [ ] Public APIs documented if the project documents them
- [ ] Magic numbers / strings extracted to named constants

## Compatibility & Rollout

- [ ] No breaking API change without a deprecation path or version
      bump
- [ ] Feature flags where the change is risky or partially rolled out
- [ ] Migration is safe to deploy ahead of code (or vice versa) per
      the project's deploy order
- [ ] Backwards compatibility for serialized data (queues, cache,
      sessions) preserved
- [ ] Env vars added are documented in `.env.example`

## Misc

- [ ] No accidentally committed files (`.env`, IDE config, logs,
      build artifacts)
- [ ] Dependencies added are actually needed and at an appropriate
      version
- [ ] Commit history is reasonably clean (no "fix typo fix typo fix
      typo" chains)
- [ ] PR description matches what the diff actually does
