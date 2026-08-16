# REBUILD REQUIRED — API Error Response Layer

**Raised:** 2026-05-13 during Phase D / CP4-2.
**Scope:** error response shape only. **Success responses STAY as-is** (already consistent under `{ data: ... }` wrapper).

---

## Why this is REBUILD and not a quiet patch

The user's Phase D charter explicitly authorizes raising REBUILD REQUIRED for any specific layer that isn't mobile-ready without restructuring. The error layer qualifies because:

1. **Six distinct error shapes** are currently emitted across the API (5 observed live + 1 in earlier B7 work). A mobile client cannot consume an API whose error contract is a runtime guess.
2. The variance hides a CURRENT FRONTEND BUG: `parseApiError` in `frontend/lib/api/client.ts` reads `{message, errors:{field:[]}, code}` (Laravel default). `AuthController::login` emits `{errors:[{code, title, detail}]}` (JSON:API array). They are structurally incompatible. The frontend currently CANNOT extract the INVALID_CREDENTIALS code on a wrong-password attempt and falls back to a generic message.
3. A mobile app cannot retry/recover/route based on a code that is sometimes `body.code`, sometimes `body.errors[0].code`, and sometimes absent entirely.

A patch that adds the new shape WITHOUT removing the old shape would multiply the variance, not reduce it. The rebuild is the smallest correct intervention.

---

## Observed variance — verbatim curl output (commit `4be7bc3`)

| # | Endpoint | Status | Shape |
|---|---|---|---|
| 1 | `GET /api/v1` | 200 | `{"api":"wamadat-v1","tenant":"wamadat"}` — flat |
| 2 | `GET /catalog/categories` | 200 | `{"data":[...]}` — data-wrapped (OK) |
| 3 | `POST /auth/login` 8-char password validation | 422 | `{"message":"...", "errors":{"password":[...]}}` |
| 4 | `POST /auth/login` empty body | 422 | `{"message":"البريد...", "errors":{"email":[...], "password":[...]}}` — Arabic |
| 5 | `GET /me/notifications` no auth | 401 | `{"message":"غير مصرَّح..."}` — no `errors`, no `code` |
| 6 | `POST /auth/login` rate-limited (B7 proof) | 429 | `{"errors":[{"code":"...", "title":"...", "detail":"..."}]}` |
| 7 | `POST /auth/login` wrong password | 401 | `{"errors":[{"code":"INVALID_CREDENTIALS","title":"...","detail":"..."}]}` |
| 8 | tenant header missing | 422 | `{"message":"...", "code":"TENANT_REQUIRED", "hint":"..."}` — top-level `code` |

That's 6 distinct shapes. `errors` is:
- absent in cases 1, 5, 8
- an object map in cases 3, 4
- an array of objects in cases 6, 7

`code` is:
- top-level in case 8
- nested inside `errors[0]` in cases 6, 7
- absent in cases 1, 2, 3, 4, 5

---

## Canonical shape (new)

Every error response from `/api/*` returns exactly:

```json
{
  "error": {
    "code": "STABLE_MACHINE_STRING",
    "message": "Localized human-readable string",
    "status": 422,
    "details": null | {field: [string,...]} | [{...}]
  }
}
```

Rules:
- `error` is always a single object, never an array, never absent.
- `code` is a `SCREAMING_SNAKE_CASE` string from a closed vocabulary (CP4-9 catalogues every code).
- `message` is the localized message; CP4-6 wires Accept-Language so it switches between AR/EN.
- `status` mirrors the HTTP status for clients that can't read response.status (some embedded mobile WebViews can't).
- `details`:
    - `null` for simple errors (auth failed, not found).
    - `{field: [string, ...]}` for validation — same shape as Laravel's `errors` block for ergonomics, but nested under `error.details`.
    - `[{...}]` for multi-error compound responses (rare — bulk endpoints).

### Examples — canonical shape applied to each case above

```jsonc
// Case 7: wrong password
{ "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "بيانات الدخول غير صحيحة",
    "status": 401,
    "details": null
}}

// Case 3: validation error
{ "error": {
    "code": "VALIDATION_FAILED",
    "message": "البريد الإلكتروني مطلوب",
    "status": 422,
    "details": {
      "email": ["البريد الإلكتروني مطلوب"],
      "password": ["كلمة المرور مطلوبة"]
    }
}}

// Case 5: unauthenticated
{ "error": {
    "code": "UNAUTHENTICATED",
    "message": "غير مصرَّح. سجّل الدخول من جديد.",
    "status": 401,
    "details": null
}}

// Case 6: rate limited
{ "error": {
    "code": "RATE_LIMITED",
    "message": "محاولات كثيرة. حاول لاحقاً.",
    "status": 429,
    "details": null
}}

// Case 8: tenant missing
{ "error": {
    "code": "TENANT_REQUIRED",
    "message": "هذا المسار يَتطلّب تَحديد المستأجر.",
    "status": 422,
    "details": null
}}
```

### Examples — success shape (UNCHANGED)

```jsonc
// list
{ "data": [...], "meta": {...} | undefined }

// single resource
{ "data": {...} }

// flat (root only — `GET /api/v1`)
{ "api": "...", "tenant": "..." }
```

Success shape decision: the existing `{data: ...}` envelope is consistent enough across the 90 endpoints to leave alone. Forcing it on the root `/api/v1` ping would be cosmetic.

---

## Implementation surface

Three files added, two modified:

### NEW: `app/Exceptions/Api/ApiException.php`
A `RuntimeException` subclass with named constructors for the common shapes:
```php
ApiException::unauthorized('INVALID_CREDENTIALS', 'بيانات الدخول غير صحيحة')
ApiException::notFound('RESOURCE_NOT_FOUND', 'الطلب غير موجود')
ApiException::unprocessable('VALIDATION_FAILED', 'البريد مطلوب', $details)
ApiException::forbidden('PERMISSION_DENIED', 'لا تَملك الصلاحية')
ApiException::rateLimited('RATE_LIMITED', 'محاولات كثيرة')
ApiException::serverError('SERVER_ERROR', 'خطأ في الخادم')
```

### NEW: `app/Exceptions/Api/ApiErrorRenderer.php`
Static method `render(Throwable, Request): JsonResponse`. Knows how to convert:
- `ApiException` → its own code/message
- `ValidationException` → `VALIDATION_FAILED` + details = `errors()`
- `AuthenticationException` → `UNAUTHENTICATED`
- `AccessDeniedHttpException` / `AuthorizationException` → `PERMISSION_DENIED`
- `NotFoundHttpException` / `ModelNotFoundException` → `RESOURCE_NOT_FOUND`
- `ThrottleRequestsException` → `RATE_LIMITED`
- `NoCurrentTenant` (Spatie) → `TENANT_REQUIRED`
- `HttpException` (generic) → uses its statusCode to pick a code
- Any other `Throwable` → `SERVER_ERROR`, message hidden in production

### MODIFIED: `bootstrap/app.php`
Replace the two existing `$exceptions->render(...)` closures with a single closure that routes every `/api/*` request through `ApiErrorRenderer::render`.

### MODIFIED: `AuthController::login`
Throw `ApiException::unauthorized('INVALID_CREDENTIALS', ...)` instead of building the response. The renderer handles it.

### MODIFIED: `frontend/lib/api/client.ts`
`parseApiError` now reads the new shape **first**, falls back to the legacy shapes for the transition window. After CP4-12 contract tests are green, the legacy-fallback can be deleted in a follow-up.

---

## Test plan (delivered by CP4-12)

7 Pest contract tests assert the EXACT shape:
- Login wrong password → `error.code='INVALID_CREDENTIALS'`, status 401.
- Login validation → `error.code='VALIDATION_FAILED'`, `error.details.email[]`, status 422.
- /me/notifications no auth → `error.code='UNAUTHENTICATED'`, status 401.
- /catalog/programs/{nope} → `error.code='RESOURCE_NOT_FOUND'`, status 404.
- Login rate limit → `error.code='RATE_LIMITED'`, status 429.
- Missing tenant header → `error.code='TENANT_REQUIRED'`, status 422.
- /me/notifications signed in → `data:[...]` shape, status 200 (success contract).

These freeze v1. Any future change that drifts the shape breaks the suite — that's the point.

---

## Security impact

POSITIVE: in production mode the renderer hides exception messages on `SERVER_ERROR` (no stack-trace leakage). Today the bare `{"message": $e->getMessage()}` patterns in some controllers can leak DB error wording.

NEUTRAL elsewhere — the canonical shape doesn't add or remove auth gates.

---

## Mobile impact

The whole point. After rebuild a Swift/Kotlin client can write:

```swift
struct ApiError: Codable {
  let code: String
  let message: String
  let status: Int
  let details: [String: [String]]?  // or AnyCodable
}
```

…and decode every error response from every endpoint with that single struct. Today they'd need 6 separate decoders + branching.

---

## Remaining risks after rebuild

- Frontend transition window: `parseApiError` reads both new and legacy shapes for the duration. Legacy fallback must be deleted in a follow-up commit once contract tests prove the new shape is the only one alive in CI.
- Third-party webhook responses (`POST /webhooks/{gateway}`, `POST /webhooks/resend`) are NOT canonicalized — Tap/Tamara/Resend expect their own response shapes (typically `200 {ok:true}`). The renderer skips webhook routes.

---

## Commit anchor
The rebuild lands as commit `???` (filled in after `git commit`).
