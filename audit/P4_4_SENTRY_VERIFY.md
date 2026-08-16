# 🛰️ P4.4 — Sentry Verify

**Date:** 2026-05-13
**Goal:** Make sure that when a real production exception fires, Sentry captures it with enough context (user, tenant, release, environment) to triage without crawling logs.

---

## 1️⃣ Pre-existing state

| Item | Status |
|---|---|
| `sentry/sentry-laravel` in composer.json | ✅ `^4.10` |
| `config/sentry.php` | ✅ exists with safe defaults (`send_default_pii = false`, `sql_bindings = false`, ignore-list for noise) |
| `ReportingClient` façade | ✅ exists — safe no-op when SDK / DSN absent |
| `.env.example` Sentry keys | ❌ missing |
| Per-request user/tenant scope binding | ❌ missing |
| Test command | ❌ missing |

---

## 2️⃣ What was added

### `app/Modules/Observability/Infrastructure/Providers/SentryScopeServiceProvider.php`
A boot-time provider that stamps every Sentry event with the right context:

- **Static tags** (set once at boot): `service=wamadat-backend`, `release=<config>`, `environment=<config>`
- **On authentication** (`Illuminate\Auth\Events\Authenticated`): `scope->setUser(['id' => <user_id>])` — **id only, no email/name**
- **On tenant switch** (`Spatie\Multitenancy\Events\MadeTenantCurrentEvent`): adds `tenant_id`, `tenant_slug` tags + a `tenant` context block

Registered in `bootstrap/providers.php` so it boots on every request, queue worker, and CLI invocation.

### `app/Modules/Observability/Infrastructure/Console/SentryTestCommand.php`
`php artisan sentry:test` — runs end-to-end verification:

1. Asserts `SENTRY_LARAVEL_DSN` is set (fails with exit 1 if not)
2. Prints the configured environment + release + (masked) DSN
3. Picks an active tenant (or accepts `--tenant=<slug>` / `--no-tenant`)
4. Makes the tenant current → fires the `MadeTenantCurrentEvent` → scope stamped
5. Calls `ReportingClient::captureException()` with a verification context block
6. Tells the operator what to look for in Sentry

Registered in `AppServiceProvider::boot()` alongside the existing `DrainEmailOutbox` command.

### `.env.example`
New Sentry block:
```
SENTRY_LARAVEL_DSN=
SENTRY_ENVIRONMENT=local
SENTRY_RELEASE=
SENTRY_TRACES_SAMPLE_RATE=0.0
```

---

## 3️⃣ Sensitive-data scrubbing (verified)

| Field | Behavior |
|---|---|
| `send_default_pii` | `false` in `config/sentry.php` — IP, headers, server vars not auto-sent |
| `sql_bindings` breadcrumb | `false` — SQL query text yes, parameter values no |
| User identifier in Sentry user | `id` ONLY — `setUser(['id' => $userId])`. No email/name/phone. |
| Tenant context | `id` + `slug` only — no plan, no billing, no internal fields |
| Test command DSN echo | Masked: `https://***@<host>/<project>` |

The codebase has **zero direct `Sentry::setUser([...email...])` calls** (verified by `grep`).

---

## 4️⃣ Ignored exceptions (won't reach Sentry — intentional)

From `config/sentry.php`:
- `AuthenticationException` — too noisy, fires for every 401
- `ValidationException` — user-input errors, not bugs
- `NotFoundHttpException` — 404s from bots/typos
- `MethodNotAllowedHttpException` — malformed clients
- `AccessDeniedHttpException` — RBAC denials, expected by design

---

## 5️⃣ Operator verification steps

```powershell
cd C:\Users\U\wamadat-platform\backend

# 1. Set DSN in .env
# SENTRY_LARAVEL_DSN=https://<your-key>@<your-org>.ingest.sentry.io/<project>
# SENTRY_ENVIRONMENT=staging  (or production)
# SENTRY_RELEASE=2026.05.13-r1

# 2. Fire a test event with a tenant context
php artisan sentry:test --tenant=wamadat

# 3. In Sentry UI:
#    - The event should appear within ~30s
#    - Tags: service=wamadat-backend · environment=<set> · release=<set>
#           · tenant_id=<uuid> · tenant_slug=wamadat
#    - User: { id: "<user_id>" }  (only when an authenticated request)
#    - No email, name, phone, IP (PII off)
#    - Context > "verification" present
#    - Context > "tenant" present

# 4. Variants
php artisan sentry:test --no-tenant     # context without tenant tag
php artisan sentry:test --tenant=acme   # any active tenant
```

---

## 6️⃣ Lint / boot status

| Item | Result |
|---|---|
| PHP lint on 4 new/changed files | **CLEAN** |
| `php artisan list` shows `sentry:test` | ✅ |
| Existing `sentry:publish` from package | ✅ (untouched) |
| Existing test suite | unchanged (Sentry wiring is no-op when DSN empty, so tests don't reach out) |

---

## 7️⃣ What's still NOT verified (and why)

| Item | Why deferred |
|---|---|
| Real event in Sentry UI | Requires a live DSN — captured in the operator runbook above |
| Source-map upload for stack traces | Tied to the deploy pipeline (P4.5 + future) |
| Sentry user feedback widget | Frontend concern, optional |
| Performance tracing sample rate tuning | Start at `0.0` until production traffic baseline known |

---

## ✅ P4.4 closure

- DSN config wired (`.env.example` documents it)
- User scope stamped on every authenticated request (id only — no PII)
- Tenant scope stamped on every tenant-switch (id + slug only)
- Static tags (service + release + environment) set at boot
- `php artisan sentry:test` is the operator's one-step verification
- ignore_exceptions list keeps noise out
- `send_default_pii=false` + `sql_bindings=false` belt-and-suspenders against leaking user data

Sentry is ready to capture production exceptions the moment a DSN is set.
