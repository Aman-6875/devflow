# Example: planner with a ticket

## Input from user

> Plan this please:
>
> **PROJ-412 — Allow users to delete their account**
> As a user, I should be able to permanently delete my account from the
> settings page. Deletion should remove personal data but preserve
> aggregated analytics. A confirmation step is required.
>
> AC:
> - Settings page has a "Delete account" button
> - Clicking it requires typing the user's email to confirm
> - On confirm, soft-delete the user, scrub PII, keep aggregated counts
> - User is logged out and redirected to homepage

## Detected context

- Stack: Laravel 11 + PHP 8.3
- Test framework: Pest
- Existing patterns: Service layer, FormRequest validation, Spatie
  Permission, soft deletes on `users`

## Output plan

```markdown
# Plan: Allow users to delete their account (PROJ-412)

## Objective
Add a self-service account deletion flow that scrubs PII while
preserving aggregated analytics. Requires email-typed confirmation
before destructive action.

## Acceptance Criteria
- [ ] Settings page exposes a "Delete account" action
- [ ] Confirmation requires the user to type their exact email
- [ ] On confirm: user is soft-deleted, PII fields are nulled, aggregate
      counters preserved
- [ ] User is logged out and redirected to `/`

## Out of Scope
- Admin-initiated deletion
- GDPR data export (separate ticket)
- Hard deletion / scheduled purge

## Approach
Add a `DeleteAccountAction` invokable service that runs inside a DB
transaction: soft-delete the user, null PII columns, dispatch an audit
event. Front the action with a `DeleteAccountRequest` that validates the
typed email matches `auth()->user()->email`. Use the existing settings
controller; do not introduce a new module.

## Steps

### 1. Add the request validator
**Files:** `app/Http/Requests/Settings/DeleteAccountRequest.php` (new)
**Change:** Single rule — `confirm_email` must equal authenticated
user's email (case-insensitive).
**Reason:** Stops accidental deletes; matches existing FormRequest
pattern in `app/Http/Requests/Settings/`.

### 2. Implement the action
**Files:** `app/Actions/Settings/DeleteAccountAction.php` (new)
**Change:** Invokable class. Inside a transaction: null PII fields
(`email`, `phone`, `address`, etc.), call `$user->delete()` (soft),
dispatch `AccountDeleted` event.
**Reason:** Keeps controller thin; isolates the destructive logic for
testability.

### 3. Wire the controller endpoint
**Files:** `app/Http/Controllers/Settings/AccountController.php` (modify)
**Change:** Add `destroy(DeleteAccountRequest $request,
DeleteAccountAction $action)` method. Logs out, invalidates session,
redirects to `/`.
**Reason:** Existing controller already handles settings actions.

### 4. Route
**Files:** `routes/web.php` (modify)
**Change:** `Route::delete('/settings/account', [AccountController::class,
'destroy'])->middleware('auth')->name('settings.account.destroy');`
**Reason:** Matches existing settings route group conventions.

### 5. UI button + confirmation modal
**Files:** `resources/views/settings/account.blade.php` (modify)
**Change:** Add danger-zone section with the button and modal containing
the email-typed confirmation input.
**Reason:** AC requires the UI affordance.

### 6. Event + listener (audit only)
**Files:** `app/Events/AccountDeleted.php` (new),
`app/Listeners/LogAccountDeletion.php` (new)
**Change:** Fire event from action; listener writes to `audit_logs`.
**Reason:** Compliance trail without coupling to the action.

## Tests
- `tests/Feature/Settings/DeleteAccountTest.php`
  - it fails validation when typed email does not match
  - it soft-deletes the user when typed email matches
  - it nulls PII columns
  - it logs the user out and redirects to /
  - it dispatches the AccountDeleted event

## Risks / Open Questions
- Q: Should we offer a grace period (e.g. 30-day undelete window)?
- Risk: Soft-delete leaves rows reachable via `withTrashed()` queries.
  Audit existing usages of `withTrashed()` on `users` before merging.
- Risk: Aggregated counters living in `users` (e.g. `total_orders`)
  might leak indirect PII. Confirm with product.

## Estimated Effort
M — touches 6 files, no external integrations, but PII scrub list needs
product sign-off before implementation.
```
