# Example: planner without a ticket

When the user has no formal ticket, ask for a plain-text description
and treat it the same way.

## Input from user

> Plan this — we need to add rate limiting to our API. Auth endpoints
> at 5 req/min, other public endpoints at 60 req/min. Use Redis.

## Planner's clarifying question

Before producing a plan, ask ONE focused question if anything critical
is missing. Here, the user didn't specify what should happen on limit
exceeded, or whether limits are per-IP or per-user.

> One question: what response should we return when the limit is
> exceeded (HTTP 429 with `Retry-After`?), and should the limit be
> per-IP or per-authenticated-user? Once you confirm these two, I'll
> produce the plan.

After user clarifies, produce the plan in the same structure as the
with-ticket example.

## Output plan

```markdown
# Plan: API rate limiting

## Objective
Protect API endpoints from abuse with two tiers: 5 req/min on auth
endpoints (per-IP), 60 req/min on public endpoints (per-authenticated-
user, falls back to per-IP for unauthenticated requests). Backed by
Redis.

## Acceptance Criteria
- [ ] Auth routes (`login`, `register`, `password/reset`) limited to
      5 req/min per IP
- [ ] Other API routes limited to 60 req/min per user (or IP if
      unauthenticated)
- [ ] Response on limit: HTTP 429 with `Retry-After` header and JSON
      body `{ "message": "Too many requests", "retry_after": <s> }`
- [ ] Redis-backed counters with sliding window

## Out of Scope
- Per-endpoint custom limits beyond the two tiers
- Quota plans / monetized tiers
- WAF-level protection

## Approach
Use Laravel's built-in `RateLimiter` facade (which is Redis-backed when
the cache store is Redis). Define two named limiters in
`AppServiceProvider::boot()`. Apply via `throttle:<name>` middleware on
route groups. Customize the 429 response with a custom exception
handler entry.

## Steps

### 1. Register named limiters
**Files:** `app/Providers/AppServiceProvider.php` (modify)
**Change:** In `boot()`, register `auth` and `public-api` limiters
using `RateLimiter::for(...)`.
**Reason:** Centralizes limit config; idiomatic Laravel 11.

### 2. Apply middleware to routes
**Files:** `routes/api.php` (modify), `routes/web.php` (modify)
**Change:** Wrap auth routes in `Route::middleware('throttle:auth')`,
other API routes in `throttle:public-api`.
**Reason:** Declarative, easy to audit.

### 3. Customize the 429 response
**Files:** `bootstrap/app.php` (modify)
**Change:** In the exception handler closure, render
`ThrottleRequestsException` as JSON with `retry_after`.
**Reason:** AC requires structured JSON; Laravel 11 puts exception
config here.

### 4. Confirm Redis is the cache store
**Files:** `config/cache.php` (read-only check), `.env.example` (modify)
**Change:** Document `CACHE_STORE=redis` requirement.
**Reason:** RateLimiter performance depends on Redis.

## Tests
- `tests/Feature/RateLimitingTest.php`
  - it allows 5 auth requests per minute then 429s
  - it allows 60 public requests per minute per user
  - it returns `Retry-After` header on 429
  - it scopes auth limit per IP, not per user

## Risks / Open Questions
- Q: Behind a load balancer, how does the request see the real client
  IP? Confirm `TrustProxies` is configured.
- Risk: Shared Redis across environments will leak limit counters.
  Use a key prefix per environment.

## Estimated Effort
S — three file touches, well-trodden Laravel path.
```
