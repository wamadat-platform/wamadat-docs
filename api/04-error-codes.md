# API v1 — Error Codes Catalogue

**Authority:** This document is the **single source of truth** for `error.code` strings emitted by `/api/v1/*` responses. New code? Add a row here BEFORE adding the emit site. Removing a code? See "deprecation" at the bottom.

**Last reconciled:** 2026-05-14 (CP4-9). Branch tip: `e7de7e7`.

---

## How to read this catalogue

Every `error.code` belongs to exactly one **family** (prefix). Every code has:

- **Code** — the SCREAMING_SNAKE_CASE machine string. Immutable; if semantics change, ship a NEW code and deprecate the old one.
- **HTTP** — the status the response carries.
- **Class** — `retryable` (a future request with no change in client state might succeed) / `fatal` (no point retrying the same request unchanged) / `recoverable` (retry succeeds only after a specific client action — e.g. refresh, re-login).
- **UI surface** — where a typical client should route the user. `inline` = next to a form field; `toast` = transient banner; `screen` = navigate (login/error page); `silent` = retry/refresh logic, no user message.
- **AR / EN message keys** — the i18n bundle paths. The Arabic strings are the current production wording. The English keys are reserved here for CP4-6 to populate when the Accept-Language middleware lands.
- **Emit site** — file + symbol that throws or returns the code.

A code MAY be emitted from multiple sites; family + meaning must stay identical across all of them.

---

## Family: AUTH_*  — authentication + session

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `UNAUTHENTICATED` | 401 | recoverable (refresh or login) | silent then screen | غير مصرَّح. سجّل الدخول من جديد. | `errors.unauthenticated` | `ApiErrorRenderer` (AuthenticationException) |
| `INVALID_CREDENTIALS` | 401 | fatal-to-this-request | inline (login form) | بيانات الدخول غير صحيحة | `errors.invalid_credentials` | `AuthController::login` |
| `ACCOUNT_INACTIVE` | 401 | fatal | screen (support contact) | الحساب غير نشط. | `errors.account_inactive` | `TokenService::rotate` |
| `REFRESH_TOKEN_MISSING` | 422 | fatal-to-this-request | silent then screen | لم يتم إرسال رمز التحديث. | `errors.refresh_missing` | `AuthController::refresh` |
| `REFRESH_TOKEN_INVALID` | 401 | fatal | screen (login) | رمز التحديث غير صالح. | `errors.refresh_invalid` | `TokenService::rotate` (×2) |
| `REFRESH_TOKEN_EXPIRED` | 401 | fatal | screen (login) | رمز التحديث منتهي الصلاحية. الرجاء تسجيل الدخول مجدداً. | `errors.refresh_expired` | `TokenService::rotate` |
| `REFRESH_TOKEN_REUSED` | 401 | fatal — **security event** | screen (login) + security audit | تم استخدام رمز التحديث بشكل مكرر. تم تسجيل خروج كل الجلسات لأمان حسابك. | `errors.refresh_reused` | `TokenService::rotate` |
| `CANNOT_REVOKE_CURRENT_SESSION` | 422 | fatal-to-this-request | toast | لا يمكن إنهاء الجلسة الحالية من هنا. استخدم `/auth/logout`. | `errors.cannot_revoke_current` | `SessionsController::destroy` |
| `SESSION_NOT_FOUND` | 404 | fatal | toast | الجلسة المطلوبة غير موجودة. | `errors.session_not_found` | `SessionsController::destroy` |

**Family rules:**
- Anything 401 that surfaces from an authenticated endpoint should be `UNAUTHENTICATED`. The client's job: attempt one refresh, retry the original request once, then route to login if still 401.
- `REFRESH_TOKEN_REUSED` is the **only** 401 in this family that is also a security event — analytics / Sentry alerts should fire on it.

---

## Family: PERMISSION_*  — authorization

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `PERMISSION_DENIED` | 403 | fatal | toast OR screen | لا تَملك الصلاحية لهذا الإجراء. | `errors.permission_denied` | `ApiErrorRenderer` (AuthorizationException / AccessDeniedHttpException), 11 legacy controller emits (canonicalized by middleware) |

**Family rules:**
- 403 NEVER means "log in again" — it means the user IS authenticated but lacks the permission. The client should NOT call `/auth/refresh` on this code.

---

## Family: VALIDATION_*  — request shape rejected

| Code | HTTP | Class | UI surface | AR (per-field) | EN key | Emit site |
|---|---|---|---|---|---|---|
| `VALIDATION_FAILED` | 422 | fatal-to-this-request | inline (per field from `error.details`) | first field's first message | `errors.validation_failed` | `ApiErrorRenderer` (ValidationException) |

**Family rules:**
- `error.details` carries `{field: [string, ...]}` — the per-field messages. The client renders these next to their respective inputs.
- `error.message` is the headline (first field's first message) so clients can show a generic toast in addition to inline.

---

## Family: RESOURCE_*  — resource not found / unavailable

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `RESOURCE_NOT_FOUND` | 404 | fatal-to-this-request | screen (404) OR toast | المورد المطلوب غير موجود. | `errors.resource_not_found` | `ApiErrorRenderer` (ModelNotFound/NotFoundHttp), 32 legacy controller emits |
| `NOT_ENROLLED` | 403 | fatal-to-this-request, then route to enroll | screen (program catalog) | لم تَسجّل في هذا البرنامج. | `errors.not_enrolled` | `QuizController` (×2) |
| `PROGRAM_INCOMPLETE` | 422 | fatal-to-this-request | toast | البرنامج لم يكتمل بعد. | `errors.program_incomplete` | `CertificateController` |

**Family rules:**
- `NOT_ENROLLED` is the canonical "you don't have access to this paid program yet" code. UI should deep-link to the program's purchase / enrollment page.

---

## Family: RATE_*  — throttling

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `RATE_LIMITED` | 429 | **retryable after delay** (read `Retry-After` header) | toast (with countdown) | محاولات كثيرة. حاول لاحقاً. | `errors.rate_limited` | `ApiErrorRenderer` (ThrottleRequestsException) |

**Family rules:**
- 429 ALWAYS carries `Retry-After` (seconds). The client SHOULD respect it; manually retrying before that triggers another 429.
- Mobile clients implementing automatic backoff: minimum 5s + jitter regardless of `Retry-After`.

---

## Family: TENANT_*  — multi-tenant routing

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `TENANT_REQUIRED` | 422 | fatal — client config bug | screen (error) | هذا المسار يَتطلّب تَحديد المستأجر. | `errors.tenant_required` | `ApiErrorRenderer` (NoCurrentTenant) |

**Family rules:**
- Only ever surfaces when a mobile/SDK build forgets the `X-Tenant-Slug` header. Treat as a build-time bug, not a runtime UI condition.
- `error.details.hint` carries a developer-targeted hint (visible only in dev/testing envs in v2).

---

## Family: OPERATION_*  — business-rule rejection

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `OPERATION_FAILED` | 422 | fatal — depends on `error.message` | toast | varies per call site | `errors.operation_failed` | 13 legacy 422 controller emits (CheckoutService, CouponService, GiftService, etc.) |
| `CONFLICT` | 409 | fatal-to-this-request | toast | (depends on emitter) | `errors.conflict` | `ApiErrorRenderer` (HttpException with status 409) |

**Family rules:**
- `OPERATION_FAILED` is the **least specific** code — it means "the request was syntactically valid but the business rule says no". The client SHOULD render `error.message` directly because there's no machine-actionable subtype.
- This family will be **decomposed** over time as call sites gain their own specific codes (e.g. `COUPON_EXPIRED`, `INSUFFICIENT_HEADROOM`, etc.). When that happens, the row stays here as the fallback and the new codes get their own rows.

---

## Family: HTTP_*  — protocol errors

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `METHOD_NOT_ALLOWED` | 405 | fatal — client bug | screen (error) | — | `errors.method_not_allowed` | `ApiErrorRenderer` (HttpException status 405) |
| `INVALID_REQUEST` | 400 | fatal-to-this-request | toast | (depends on emitter) | `errors.invalid_request` | NewsletterController (legacy 400 emit, normalized by middleware) |
| `HTTP_ERROR_{N}` | N | fatal | screen (error) | (depends) | `errors.http_generic` | `ApiErrorRenderer` fallback for unmapped 4xx |

**Family rules:**
- `HTTP_ERROR_{N}` is the open-ended escape hatch. Seeing it in logs means a new emit site should get a real code added to this catalogue. Treat as a code-hygiene to-do.

---

## Family: SERVER_*  — anything else

| Code | HTTP | Class | UI surface | AR | EN key | Emit site |
|---|---|---|---|---|---|---|
| `SERVER_ERROR` | 500 | retryable (with backoff) | toast | حدث خطأ في الخادم. حاول لاحقاً. | `errors.server_error` | `ApiErrorRenderer` catch-all |

**Family rules:**
- In production environments, `error.message` is REPLACED with a generic Arabic line to prevent stack-trace leakage. In `local` / `testing` the raw exception message is preserved for debugging.
- Mobile clients SHOULD retry once after 5-10 seconds (`SERVER_ERROR` is the most common transient-flake bucket).

---

## Retryable-vs-fatal mobile decision tree

```
HTTP status received
  │
  ├─ 401
  │    ├─ code = UNAUTHENTICATED      → silently call /auth/refresh, retry once
  │    ├─ code = INVALID_CREDENTIALS  → show on login form, do NOT refresh
  │    ├─ code = REFRESH_TOKEN_*       → drop to login screen, clear keychain
  │    └─ code = ACCOUNT_INACTIVE     → screen "contact support"
  │
  ├─ 403  →   never refresh on 403
  │           render error.message, route to upgrade/enroll if applicable
  │
  ├─ 404
  │    ├─ code = RESOURCE_NOT_FOUND   → in-context 404 view
  │    ├─ code = SESSION_NOT_FOUND    → silently refresh sessions list
  │    └─ code = NOT_ENROLLED         → deep-link to program purchase
  │
  ├─ 422  →  render error.details per-field, fall back to error.message
  │
  ├─ 429  →  respect Retry-After header + jitter
  │
  ├─ 5xx  →  retry once with 5-10s backoff, then surface a toast
  │
  └─ anything else → log + show error.message
```

---

## Adding a new code

1. Edit this file FIRST — add the row in the correct family with all 6 columns filled.
2. If the family doesn't exist yet, add the family section with its rules.
3. Throw `ApiException::<constructor>('YOUR_CODE', 'Arabic message')` from the call site.
4. If the code maps to a custom HTTP status that doesn't fit existing constructors, add a constructor on `ApiException` first.
5. Run the smoke check (next section). It MUST pass.

## Deprecating a code

1. Add `(DEPRECATED — use X)` to the row's Code column. Keep the row for one release.
2. Update emit sites to throw the replacement.
3. Add a CHANGELOG-v1.md entry under "Non-breaking — deprecated".
4. Remove the row in the next minor release.

---

## Smoke check (light-touch contract proof)

This bundle reproduces every code currently in the catalogue. Run before merging any change to error emission.

```bash
$ host=http://127.0.0.1:8765
$ tenant='-H "X-Tenant-Slug: wamadat"'

# UNAUTHENTICATED
curl -s -o /dev/null -w "%{http_code}\n" $host/api/v1/me/notifications $tenant
# 401 with body.error.code = UNAUTHENTICATED

# INVALID_CREDENTIALS
curl -s -X POST $host/api/v1/auth/login -d '{"email":"x@x","password":"abcdefgh"}' \
  -H "Content-Type: application/json" $tenant | grep -o '"code":"INVALID_CREDENTIALS"'

# VALIDATION_FAILED
curl -s -X POST $host/api/v1/auth/login -d '{}' -H "Content-Type: application/json" $tenant \
  | grep -o '"code":"VALIDATION_FAILED"'

# RESOURCE_NOT_FOUND (router miss)
curl -s $host/api/v1/zzz-no-such $tenant | grep -o '"code":"RESOURCE_NOT_FOUND"'

# REFRESH_TOKEN_MISSING
curl -s -X POST $host/api/v1/auth/refresh -d '{}' -H "Content-Type: application/json" $tenant \
  | grep -o '"code":"REFRESH_TOKEN_MISSING"'

# REFRESH_TOKEN_INVALID
curl -s -X POST $host/api/v1/auth/refresh -d '{"refresh_token":"garbage"}' \
  -H "Content-Type: application/json" $tenant | grep -o '"code":"REFRESH_TOKEN_INVALID"'

# TENANT_REQUIRED — use a deliberately-unknown slug.
# (Omitting the header in dev resolves the default "wamadat" tenant silently;
#  in production where subdomain resolution is on, omitting it would fail.)
curl -s $host/api/v1/catalog/categories -H "X-Tenant-Slug: zzz-no-such-tenant" \
  | grep -o '"code":"TENANT_REQUIRED"'

# INVALID_REQUEST — newsletter unsubscribe with empty token.
curl -s "$host/api/v1/newsletter/unsubscribe?token=" $tenant \
  | grep -o '"code":"INVALID_REQUEST"'

# METHOD_NOT_ALLOWED — wrong verb on a known route.
curl -s -X DELETE $host/api/v1/catalog/categories $tenant \
  | grep -o '"code":"METHOD_NOT_ALLOWED"'

# RATE_LIMITED — replay /auth/login 21+ times from same IP (proven in B7).
```

**Smoke-check anti-pattern:** if a row in this catalogue lists an emit site but the smoke check can't reach the code locally (e.g. TENANT_REQUIRED relies on the production subdomain resolver), document the alternative provocation here. Never lower the catalogue's claim — fix the test.

Each grep above should print the expected code exactly once. If any code is missing from the live API, the catalogue is out of sync with reality.

---

## Open items

- **Localization completion (CP4-6):** the `EN` columns are placeholders. Wiring an `Accept-Language` middleware + the EN messages is CP4-6's deliverable. The keys above are FROZEN now; CP4-6 only writes the English strings against them.
- **OPERATION_FAILED decomposition:** as Phase D unfolds, individual call sites should graduate to their own family-specific codes (e.g. `COUPON_EXPIRED`). Mark the row "(historical)" once empty.
- **`error.details.hint` exposure:** `TENANT_REQUIRED` currently surfaces the developer hint in all environments. Wrap it behind `app.debug` (or a separate env flag) once OpenAPI docs are reachable for dev clients.

---

## CHANGELOG anchor

This file ships in commit `<filled at commit time>`. CHANGELOG-v1.md carries the migration note.
