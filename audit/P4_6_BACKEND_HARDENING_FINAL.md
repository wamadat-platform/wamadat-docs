# 🛡️ P4.6 — Backend Hardening Final Pass

**Date:** 2026-05-13
**Method:** Domain-by-domain code review against the 11-point hardening checklist. Flag real issues only — no speculative "could be tighter" without a concrete attack.

---

## 1️⃣ Authorization

| Concern | Status |
|---|---|
| Role-based middleware | ✅ Spatie `role:` + `permission:` + `role_or_permission:` aliases registered in `bootstrap/app.php` |
| Per-resource scoping | ✅ Every `/me/*` controller scopes by `$request->user()->id` (verified in Phase 2.5 — 0 IDORs found across 8 resources) |
| Admin panel gating | ✅ `/admin` uses Filament's `authGuard('web')` against tenant users + role gates on each resource |
| Suspended/locked users | ✅ `active.account` middleware kicks them out on the very next request regardless of token validity |
| Defense-in-depth for delete-own-only | ✅ ReviewController scopes the delete query by `user_id` so the API returns 404 (not 403) and never leaks the row's existence |

**No changes needed.**

---

## 2️⃣ Validation

| Concern | Status |
|---|---|
| Form-request validation on every write endpoint | ✅ Inline `$request->validate(...)` or dedicated FormRequest used consistently |
| Strict types on validators | ✅ `nullable`, `required`, `max`, `min` applied where appropriate |
| URL fields | ⚠️ **`AssignmentController::submit`** accepted any string for `files.*.url` — could be filed as content from a non-S3 origin |
| Mass assignment | ✅ Models use `protected $guarded = []` or `protected $fillable = [...]` depending on the table's risk profile; no `Model::create($request->all())` patterns found |

**Fix applied:**
- `app/Modules/Assessment/Infrastructure/Http/Controllers/AssignmentController.php` — tightened file/url validators to `url:http,https` with `max:1000`. Origin allowlist (must point to our S3/CDN) is captured for Phase 10.5 in BACKLOG.

---

## 3️⃣ Transactions

Audited every multi-write service. **16 of 16** wrap their writes in `DB::transaction`:

```
TenantProvisioner       LearningService       PasswordResetService
EnrollmentService       AccountService        AuthService
ReviewService           RefundService         InvoiceService
GiftService             CheckoutService       CertificateIssuer
QuizAttemptService      StreakService         AffiliateService
CohortFinalizationService
```

Each transaction is on the appropriate connection (`tenant` for tenant data, `landlord` for tenant provisioning).

**No changes needed.**

---

## 4️⃣ Webhooks

`PaymentWebhookController::handle`:

| Check | Status |
|---|---|
| Gateway whitelist | ✅ `gateways->has($gatewayName)` — unknown gateway → 400 |
| Signature verification | ✅ `$gateway->verifyWebhook($request)` — failure → 400 + warning log |
| Idempotency | ✅ `(gateway, external_id, event_type)` unique index + `firstOrCreate` |
| Fast 2xx on duplicates | ✅ `wasRecentlyCreated` check → returns `{duplicate: true}` with 200 |
| Full payload preserved | ✅ Raw payload stored in `payment_webhooks.raw_payload` before processing |
| Failure recording | ✅ Errors update the row's `result` + `error_message`; replay action exists in Filament (P4.3) |

Resend (email events) webhook follows the same shape.

**No changes needed.**

---

## 5️⃣ Queues

| Check | Status |
|---|---|
| Driver | Redis (per `.env.example`) |
| Tenant-aware | ✅ `queues_are_tenant_aware_by_default: true` in `config/multitenancy.php` |
| Failed-job persistence | ✅ landlord `failed_jobs` table (added in P4.3) |
| Failed-job UI + retry | ✅ Filament resource added in P4.3 |
| Health probe for queue depth | ✅ HealthController.probeQueue + QueueDepthWidget (P4.3) |

**No changes needed.**

---

## 6️⃣ Exceptions

`bootstrap/app.php → withExceptions`:

| Concern | Status |
|---|---|
| 401 for API auth failures | ✅ Clean JSON response, no redirect-to-login leak |
| 422 for missing tenant context | ✅ Clean JSON with `code: TENANT_REQUIRED` + hint |
| Per-segment Next.js error.tsx | ✅ Added during Phase 3.15 for /learn, /dashboard, /checkout |
| Sentry capture on unhandled | ✅ ReportingClient façade + new SentryScopeServiceProvider (P4.4) |
| `ignore_exceptions` filters noise | ✅ Auth / Validation / 404 / 405 / 403 all skipped from Sentry |

**No changes needed.**

---

## 7️⃣ Rate limits

`AppServiceProvider::configureRateLimiters` + per-route `throttle:*`:

| Surface | Limit |
|---|---|
| `POST auth/login` | 5/min per email+IP, 20/min per IP fallback |
| `POST auth/register` | 3 per 5min per IP |
| `POST auth/phone/send-otp` | 5 per 10min |
| `POST auth/phone/verify` | 10 per 5min |
| `POST auth/forgot-password` | 5 per 15min |
| `POST auth/reset-password` | 5 per 15min |
| `POST contact` | 6/min |
| `POST newsletter/subscribe` | 10/min |
| `POST consultations/requests` | 5/min |
| `POST learn/lessons/{id}/ask` (AI) | 20/min |
| `POST attendance/scan` | 60/min |

Coverage of every public + sensitive endpoint. **No changes needed.**

---

## 8️⃣ File uploads

| Concern | Status |
|---|---|
| Backend never accepts multipart from users | ✅ Frontend uploads to S3 directly (CDN URL only reaches backend) |
| Assignment attachments | ✅ Hardened in §2 — must be `url:http,https`, length-capped |
| Filament admin uploads (program covers, etc.) | ✅ Uses Filament's `FileUpload` component which has built-in MIME + size validation |
| Profile avatar | Future (PHASE 22) — not active |

**No changes needed (beyond §2 fix).**

---

## 9️⃣ API consistency

| Convention | Status |
|---|---|
| Snake-case JSON fields | ✅ Consistent |
| Errors always `{ message: ..., errors?: {...}, code?: ... }` | ✅ Centralized via Laravel validators + custom render in bootstrap |
| Pagination: `data + meta` | ✅ Used in catalog + instructors + my-programs |
| Auth: Bearer or cookie | ✅ Cookie-bridge middleware unwraps the SPA cookie into a Bearer header so Sanctum sees one shape |
| HTTP status codes | ✅ 200/201 success, 401 auth, 403 authz, 404 not found, 422 validation, 429 throttled, 500 server |

**No changes needed.**

---

## 🔟 Tenant isolation

Verified in PHASE 2.5 integrity patch + revalidated here:

| Concern | Status |
|---|---|
| Tenant tables use `UsesTenantConnection` trait | ✅ 100% — every model in tenant modules |
| Landlord tables use `protected $connection = 'landlord'` | ✅ Including new `FailedJobModel` from P4.3 |
| Search_path switched per request | ✅ `SwitchTenantSchemaTask` rewrites it for the `tenant` connection |
| Cross-tenant query test | ✅ `tests/Feature/Tenant/TenantIsolationTest` — passing |
| Filament admin scoping | ✅ All new operations resources tenant-scope (P4.3 documents the failed-jobs payload filter) |

**No changes needed.**

---

## 1️⃣1️⃣ Logging

| Concern | Status |
|---|---|
| No password / token in logs | ✅ Grep found zero `Log::*(password\|token)` patterns |
| Sentry PII off | ✅ `send_default_pii: false` + `sql_bindings: false` |
| Append-only audit log | ✅ Per-tenant `audit_logs` table has DB-level trigger blocking UPDATE/DELETE |
| Audit trail on operational actions | ✅ Every retry/replay/delete in the Filament ops panel writes via `AdminAuditWriter` (P4.3) |
| Sentry user scope is id-only | ✅ Verified in P4.4 — no email, name, phone ever sent to Sentry |

**No changes needed.**

---

## 📊 Summary

| Area | Findings | Fixed | Deferred |
|---|---|---|---|
| Authorization | 0 issues | — | — |
| Validation | 1 (assignment URL too permissive) | ✅ | Origin allowlist → PHASE 10.5 |
| Transactions | 0 issues | — | — |
| Webhooks | 0 issues | — | — |
| Queues | 0 issues | — | — |
| Exceptions | 0 issues | — | — |
| Rate limits | 0 issues | — | — |
| File uploads | covered by §2 fix | ✅ | — |
| API consistency | 0 issues | — | — |
| Tenant isolation | 0 issues | — | — |
| Logging | 0 issues | — | — |

**Net real fixes in P4.6: 1** — tightened assignment-submit URL validators.

The backend was already production-grade going into this audit (the prior phases did the heavy lifting). This pass confirms no critical gaps remain across the 11-point checklist.

---

## ✅ P4.6 closure

The backend has no outstanding hardening blockers. Single line of code changed (validator), one BACKLOG item logged (S3 origin allowlist for future tightening).
