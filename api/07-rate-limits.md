# API v1 — Rate Limit Policy

**Authority:** Single source of truth for every throttle applied to `/api/v1/*` endpoints.

**Captured:** 2026-05-13 (CP4-10). Commit anchor: `<filled at commit time>`.

---

## Wire contract

When a client trips a throttle, the response is **always** the canonical envelope:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json; charset=utf-8

{
  "error": {
    "code": "RATE_LIMITED",
    "message": "محاولات كثيرة. حاول لاحقاً.",
    "status": 429,
    "details": null
  }
}
```

- `error.code` is **always** `RATE_LIMITED` for 429 responses on `/api/*` (CP4-9 catalogue row). Mobile/SDK clients should match on the code, not on the message.
- `error.message` localizes via `Accept-Language: ar|en`. EN: `Too many requests. Please try again later.`
- Throttle responses carry the standard `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `Retry-After` headers. Clients should honor `Retry-After` for backoff.

> **Bug fixed in CP4-10:** prior to this CP, throttle responses incorrectly carried `code: "HTTP_ERROR_429"` because `App\Exceptions\Api\ApiErrorRenderer` had imported `ThrottleRequestsException` from the wrong namespace (Symfony rather than `Illuminate\Http\Exceptions`). The `instanceof` check silently returned false and execution fell through to the generic HttpException branch. Patched in commit `<anchor>`. See OPEN_RISKS entry R-CLOSED-CP4-10-namespace.

---

## Public endpoints (no auth required)

These throttles defend the **unauthenticated** attack surface — anti-bot, anti-brute-force, anti-enumeration. Keyed primarily by IP (and email for login, where applicable).

| Method | Path | Throttle | Window | Key |
|---|---|---|---|---|
| POST | `/auth/register` | `throttle:register` | 3 per 5 min | per IP |
| POST | `/auth/login` | `throttle:login` (3-layer) | 5 per min + 20 per min + 100 per hour | (email\|ip), IP-min, IP-hour |
| POST | `/auth/refresh` | `throttle:login` (3-layer) | same as login | same as login |
| POST | `/auth/phone/send-otp` | `5,10` | 5 per 10 min | per IP |
| POST | `/auth/phone/verify` | `10,5` | 10 per 5 min | per IP |
| POST | `/auth/forgot-password` | `5,15` | 5 per 15 min | per IP |
| POST | `/auth/reset-password` | `5,15` | 5 per 15 min | per IP |
| POST | `/contact` | `6,1` | 6 per min | per IP |
| POST | `/newsletter/subscribe` | `10,1` | 10 per min | per IP |
| POST | `/consultations/requests` | `5,1` | 5 per min | per IP |

**Login throttle composition** (`AppServiceProvider::configureRateLimiters`):

```php
RateLimiter::for('login', fn (Request $r) => [
    Limit::perMinute(5)->by(strtolower($r->input('email','')) . '|' . $r->ip()),  // surgical
    Limit::perMinute(20)->by('ip-min:'  . $r->ip()),                              // distributed
    Limit::perHour(100)->by('ip-hour:'  . $r->ip()),                              // slow-spread
]);
```

Three layers compose so an attacker can't beat any single one by varying email or pacing requests. See B7 for the original threat model.

---

## Authenticated endpoints

These throttles defend **per-user** abuse vectors — token-bearer enumeration, payment-gateway abuse, brute-force OTP confirmation, expensive renders. Laravel's default key for authenticated requests is the user's primary key, so each user has their own bucket.

| Method | Path | Throttle | Window | Rationale |
|---|---|---|---|---|
| DELETE | `/me/account` | `2,60` | 2 per hour | A leaked bearer must not nuke the account in seconds — PDPL right of erasure must remain user-deliberate. |
| PUT | `/me/password` | `5,15` | 5 per 15 min | Credentials surface; matches reset-password cadence. |
| POST | `/me/2fa/confirm` | `10,5` | 10 per 5 min | OTP enumeration during setup. |
| POST | `/me/2fa/disable` | `5,5` | 5 per 5 min | Highest-risk 2FA endpoint — turns the control OFF. |
| POST | `/me/qr/regenerate` | `10,5` | 10 per 5 min | Rotates a sensitive token. |
| POST | `/me/certificates/issue/{slug}` | `5,5` | 5 per 5 min | PDF render + signature is expensive. |
| POST | `/catalog/programs/{slug}/reviews` | `5,60` | 5 per hour | Comfortable for legit users; slow review-spam. |
| POST | `/learn/lessons/{lesson}/ask` | `20,1` | 20 per min | AI study assistant — caps LLM cost per user. |
| POST | `/learn/attempts/{id}/answers` | `60,1` | 60 per min | Per-answer autosave, debounced ~500ms client-side. |
| POST | `/attendance/scan` | `60,1` | 60 per min | High legit throughput at door check-ins. |
| POST | `/checkout` | `10,1` | 10 per min | Payment-gateway abuse vector. |
| POST | `/checkout/coupon/validate` | `30,1` | 30 per min | Coupon-code enumeration vector. |
| POST | `/me/gifts` | `10,1` | 10 per min | Per-user gift creation pace. |
| POST | `/me/gifts/redeem` | `10,1` | 10 per min | Gift-code enumeration vector. |

**Endpoints intentionally NOT throttled** (low cost OR already gated by enrollment/ownership/admin role): `/me/notifications`, `/me/wishlist`, `/me/orders`, `/me/notes`, `/me/today`, `/me/calendar.ics`, `/me/sessions`, `/me/profile`, `/me/preferences`, `/me/affiliate`, `/me/enrollments`, `/me/streak`, `/me/achievements`, `/me/data-export` (the export job itself is rate-limited at the queue level via per-user concurrency).

---

## Reading the throttle string

Laravel's `throttle:N,M` means **N attempts per M minutes**:
- `throttle:5,60` → 5 requests per 60 minutes (per hour)
- `throttle:5,15` → 5 requests per 15 minutes
- `throttle:10,1` → 10 requests per 1 minute
- `throttle:2,60` → 2 requests per hour

The key is automatic: the authenticated user's ID for `auth:sanctum` routes, the client IP for public routes.

---

## Smoke check

The proof that a throttle is alive and returns the canonical shape:

```bash
host=http://127.0.0.1:8765
tenant='X-Tenant-Slug: wamadat'

# Public — trip the login throttle (5 per (email|ip) per minute)
for i in $(seq 1 7); do
  code=$(curl -s -o /tmp/r.json -w "%{http_code}" \
    -X POST "$host/api/v1/auth/login" \
    -H "$tenant" -H "Content-Type: application/json" \
    -d '{"email":"probe-smoke@wmt.sa","password":"wrong"}')
  echo "#$i → $code"
  [ "$code" = "429" ] && { cat /tmp/r.json; break; }
done

# Expected on attempt #6:
# {"error":{"code":"RATE_LIMITED","message":"محاولات كثيرة. حاول لاحقاً.","status":429,"details":null}}
```

If any smoke step regresses — `code` is anything other than `RATE_LIMITED`, the envelope is missing `error.*`, or the throttle never trips — the regression has broken the public API contract. Open a CHANGELOG-v1.md entry.

---

## Adding a new throttled endpoint

1. Identify the abuse class:
    - **Bot/anti-enumeration on public surface** → low cap, by IP, e.g. `throttle:5,1`.
    - **Per-user expensive operation** → moderate cap, by user, e.g. `throttle:5,5` or `throttle:20,1`.
    - **Sensitive credentials/security control** → tight cap, e.g. `throttle:5,15` or `throttle:2,60`.
    - **High legit throughput operation** (autosave, scan) → high cap with short window, e.g. `throttle:60,1`.
2. Apply on the route:
   ```php
   Route::post('me/foo', [FooController::class, 'store'])
       ->middleware('throttle:N,M');
   ```
3. Add a row to the table above **before** merging.
4. If the limit needs a composed key (e.g. email+IP), define it in `AppServiceProvider::configureRateLimiters` and use `throttle:my-named-limiter`.

Do not invent new error codes for 429 — `RATE_LIMITED` is the catalogue row, always.
