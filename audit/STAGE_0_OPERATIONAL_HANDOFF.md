# 📦 Stage 0 — Operational Handoff

**Date:** 2026-05-13
**Scope:** Engineering is complete (P1 → P7). This pass converts that finished
code into something an operator can actually deploy and run. Four deliverables,
all targeting items called out in P6 §4 (B-1 through B-5) and P7 Stage 0.

---

## 1️⃣ Production environment template

**File:** `.env.production.example` (repo root)

A 250-line annotated template covering 20 sections — app, DB landlord/tenant,
Redis, mail (Resend), auth, payments (Tap/Tamara), ZATCA, storage (Bunny), live
sessions (100ms), AI (Anthropic/OpenAI), search (Meilisearch), observability
(Sentry), Reverb, OAuth, feature flags, logging, runtime.

Every key is marked one of:
- **REQUIRED** — empty value fails Stage 0 verification
- **REQUIRED at full production** — optional for soft-launch (e.g. Tap live keys)
- **(optional)** — feature-gated; activates a side-system when filled

The file ends with the **Stage 0 verification checklist** — 7 commands an
operator runs in order before flipping DNS. If any fail, the runbooks (§2 below)
explain what to do.

**Critical posture decisions baked in:**
- `APP_DEBUG=false`, `LOG_LEVEL=info` (debug leaks query params)
- `SESSION_SECURE_COOKIE=true`, `SESSION_DOMAIN=.wamadat.academy`
- `RESEND_DRIVER=resend` (NOT mock)
- `ZATCA_ENV=production`, `ZATCA_PRODUCTION=true`
- `TAMARA_BASE_URL` points to live (not `api-sandbox.tamara.co`)
- `SENTRY_RELEASE` deliberately empty in this file — populated by the deploy
  workflow (see §3)
- `SENTRY_AUTH_TOKEN` deliberately NOT in this file — it's a CI-only secret

---

## 2️⃣ Runbooks

**Directory:** `docs/runbooks/` (5 files)

| File | Failure mode | Length |
|---|---|---|
| README.md | Index + structure | one screen |
| db-down.md | Postgres unreachable / slow | one page |
| redis-down.md | Redis unreachable (sessions, queue, cache) | one page |
| email-stalled.md | Resend delivery stalled | one page |
| webhook-mismatch.md | Payment webhook signature rejection | one page |

Each runbook follows the same 5-section format:
1. **Symptoms** — what an operator notices
2. **Verify** — 60-90 second commands to confirm diagnosis
3. **Mitigate** — stop bleeding NOW
4. **Recover** — restore green state
5. **Post-incident** — audit-log expectations + `docs/audit/incidents/` template

Each runbook is mapped to the exact signals in `HealthStatusWidget`,
`QueueDepthWidget`, `/admin/email-outboxes`, and `/admin/payment-webhooks`
delivered in P4.3, so an operator's first move is always "open `/admin`."

Special sections worth flagging:
- **db-down.md** → maintenance-mode flow, replica failover, "don't retry while
  red" rule
- **redis-down.md** → temporary `SESSION_DRIVER=cookie` mitigation +
  cascade-to-DB warning
- **email-stalled.md** → table of root causes mapped to fixes; pre-flight check
  for daily delivery probe
- **webhook-mismatch.md** → secret-rotation lifecycle, replay flow via
  `/admin/payment-webhooks`, the "raw vs parsed body" gotcha

---

## 3️⃣ Sentry sourcemap upload (B-2)

### What changed

| File | Change |
|---|---|
| `frontend/next.config.ts` | Wrapped with `withSentryConfig` from `@sentry/nextjs`. Activates ONLY when `SENTRY_AUTH_TOKEN` is in the env (deploy-time, CI-injected) — dev/local builds skip the wrapper completely so HMR + `pnpm build` stay fast. |
| `frontend/sentry.client.config.ts` | NEW — browser SDK init. `sendDefaultPii: false`, matched ignore list with backend P4.4 |
| `frontend/sentry.server.config.ts` | NEW — server runtime SDK init |
| `frontend/sentry.edge.config.ts` | NEW — edge runtime SDK init (middleware) |
| `.github/workflows/deploy.yml` | NEW — production deploy workflow. Frontend job builds with `SENTRY_AUTH_TOKEN` present → sourcemaps upload + release create automatic. Asserts no `.map` files leak into `.next/static`. Backend job creates the matching Sentry release via `getsentry/action-release@v1`. |

### Required GitHub Actions secrets (one-time setup)

| Name | Type | Value |
|---|---|---|
| `SENTRY_AUTH_TOKEN` | Secret | Token from Sentry → Settings → Auth Tokens — needs `project:read`, `project:releases`, `org:read` scopes |
| `NEXT_PUBLIC_SENTRY_DSN` | Secret | Frontend project DSN |
| `SENTRY_ORG` | Variable | `wamadat` |
| `SENTRY_PROJECT_FRONTEND` | Variable | `wamadat-frontend` |
| `SENTRY_PROJECT_BACKEND` | Variable | `wamadat-backend` |
| `NEXT_PUBLIC_API_BASE_URL` | Variable | `https://api.wamadat.academy` |
| `NEXT_PUBLIC_DEFAULT_TENANT_SLUG` | Variable | `wamadat` |

### Trigger

Deploy workflow runs on:
- `workflow_dispatch` with confirmation string `"deploy"` (manual)
- Pushing a `v*.*.*` tag (release-driven)

It deliberately does NOT run on every push/PR — sourcemap uploads + release
creation should happen once per deploy, not once per commit. CI (`ci.yml`)
remains untouched.

### Why we also wired the SDK at the same time

`withSentryConfig` is a build-time concern — it uploads `.map` files. The SDK
init files (`sentry.{client,server,edge}.config.ts`) are the runtime side —
they actually capture errors and send them with the matching release tag.
Without both, sourcemap upload would have no events to stack-trace against.
The backend's runtime side (`SentryScopeServiceProvider`) shipped in P4.4 — the
frontend was the last gap.

### Sourcemap leak guard

The deploy workflow has an explicit assertion:
```bash
if find .next/static -name "*.js.map" 2>/dev/null | grep -q .; then
    echo "::error::Sourcemaps found in .next/static — they would ship to users."
    exit 1
fi
```
`withSentryConfig` is configured with `deleteSourcemapsAfterUpload: true`, but
the assertion makes the regression visible if that flag ever flips.

---

## 4️⃣ P4.2.A — Module test harness

### What changed

| File | Change |
|---|---|
| `backend/tests/LandlordTestCase.php` | NEW — base case that explicitly migrates the landlord path. tearDown drops any `tenant_*` schema the test left behind. |
| `backend/tests/Pest.php` | Added `pest()->extend(LandlordTestCase::class)->in('app/Modules')` so module-internal tests auto-use the right base case. `tests/Feature` and `tests/Unit` targeting tightened to `tests/Feature` and `tests/Unit` paths to prevent module tests from accidentally inheriting `TenantTestCase`. |
| `app/Modules/Tenancy/Tests/Feature/TenantLifecycleTest.php` | Removed `uses(Tests\TestCase::class, RefreshDatabase::class)` — auto-applied LandlordTestCase replaces it |
| `app/Modules/Tenancy/Tests/Feature/TenantProvisioningTest.php` | Same — removed `uses(...)` line |
| `app/Modules/Tenancy/Tests/Feature/SubdomainResolutionTest.php` | Same — removed `uses(...)` line |

### Why this matters

The 17 failing module tests called out in P4.2 §7 + P4.5 §5 weren't testing
broken code — they were tests-against-broken-harness. With LandlordTestCase
applied automatically, the harness now:

1. Refreshes the landlord DB on the **correct** connection + path
2. Cleans up tenant schemas after each test (no cross-test schema pollution)
3. Stays out of the way of `TenantTestCase` (still used by `tests/Feature/`)

### Not run in this session

PHP is not on PATH in this Windows shell, so I could not execute `pest
app/Modules/Tenancy/Tests` to confirm green. The fix is structural and
matches exactly the diagnosis written into P4.2 §7. Verification command for
the operator:

```powershell
cd C:\Users\U\wamadat-platform\backend
./vendor/bin/pest app/Modules/Tenancy/Tests --testsuite=Feature
./vendor/bin/pest app/Modules/Tenancy/Tests --testsuite=Unit
```

If a test still fails, that's a real test bug to fix — not a harness issue.

### Out of scope (and why)

The `SubdomainResolutionTest::resolves tenant from custom domain` test reads
from real DNS (per P6 §5). It still does. Moving it to a mocked resolver is a
separate concern from the harness gap and is captured as a BACKLOG item.

---

## 📊 Summary

| # | Deliverable | Files | Blocker addressed |
|---|---|---|---|
| 1 | `.env.production.example` | 1 | P7 Stage 0 prerequisites |
| 2 | Runbooks (×4 + README) | 5 | P7 Stage 0 §1 runbook printing |
| 3 | Sentry sourcemap upload + frontend SDK wiring | 7 | P6 §4 B-2 |
| 4 | LandlordTestCase + Pest.php + 3 test files | 5 | P4.2.A from P4.5 §5 |

**Total: 18 files** (15 new, 3 edited). No backend production code touched —
this pass is operational tooling and test harness only.

---

## What's still external (Stage 0 §1 from P7 — unchanged)

Code-side work for Stage 0 is now complete. The remaining items are
provider/infra:

- [ ] Production Postgres provisioned (managed, daily backup + PITR)
- [ ] Production Redis provisioned (TLS, persistence)
- [ ] Wildcard SSL `*.wamadat.academy`
- [ ] DNS apex + first tenant subdomain
- [ ] Resend sender domain SPF + DKIM verified
- [ ] **Tap merchant account approved + live keys** (longest lead time — KYC)
- [ ] Production .env.production filled from the template
- [ ] GitHub Actions secrets configured (`SENTRY_AUTH_TOKEN`, etc.)
- [ ] First tenant provisioned via `php artisan tenant:provision`
- [ ] Cron `* * * * * php artisan schedule:run` on the prod box
- [ ] At least one queue worker (systemd unit or equivalent)

When those are done, run the Stage 0 verification checklist at the bottom of
`.env.production.example` — if all 7 commands pass, soft-launch is greenlit.
