# CP4-5 — Mobile Auth + Token Refresh — Proof Bundle

**Frozen:** 2026-05-14. Commit `352077a`. Branch `main` clean.

This document captures the verbatim proof artifacts produced while building CP4-5, so a future session (or auditor) can re-run each scenario and diff the output against this baseline. Where a curl returned an opaque token, the doc preserves the *prefix* needed to reason about it.

---

## 1. Pre-state and baseline

```
git log --oneline -1                  # 1ac3b8e Phase D Part 1 closeout — checkpoint
php artisan route:list | grep api/v1  # 90 endpoints
grep QUEUE_CONNECTION .env            # database
```

Canonical error contract still alive (`{error:{code,message,status,details}}`); `/health` still ships structured `checks.*`. Identical to `OPS_STATUS_SNAPSHOT_2026-05-13-phase-d-1.md`.

---

## 2. Migration applied

```
php artisan tenants:artisan "migrate --path=database/migrations/tenant/2026_05_14_080000_create_refresh_tokens_and_device_metadata.php"
# 2026_05_14_080000_create_refresh_tokens_and_device_metadata ... 85.83ms DONE
```

DB verify:
```sql
SET search_path = tenant_wamadat;
\d personal_access_tokens
# new columns present: device_name, device_type, device_id, last_ip, last_user_agent
\d refresh_tokens
# table exists with the 14 columns + indexes in the migration
```

---

## 3. Login — new response shape

Request:
```bash
curl -s -X POST http://127.0.0.1:8765/api/v1/auth/login \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -H "X-Tenant-Slug: wamadat" \
  -d '{
    "email":"student0@wamadat.demo",
    "password":"StudentPass123",
    "device_name":"Test-iPhone-15",
    "device_type":"ios",
    "device_id":"AAAA-1111-BBBB"
  }'
```

Response (key fields only):
```json
{
  "data": {
    "user": { "id": "a1c2ea63-…", "email": "student0@wamadat.demo", … },
    "access_token": "101|J4P4XCVQVOSzNHfVVOQBHdIyHH…",
    "refresh_token": "NTh04pNpETHO60voBcKE95qtYXMrq0…",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_expires_in": 5184000,
    "token": "101|J4P4XCVQVOSzNHfVVOQBHdIyHH…"     // deprecated alias
  }
}
```

Invariants verified:
- `expires_in == 3600` (60 minutes) ✓
- `refresh_expires_in == 5184000` (60 days) ✓
- `access_token` ≠ `refresh_token` (different formats: `id|opaque` vs 80-char random) ✓
- `data.token` mirrors `data.access_token` for transition only ✓

---

## 4. Use access on a protected endpoint

```bash
curl -s http://127.0.0.1:8765/api/v1/auth/me \
  -H "Authorization: Bearer 101|J4P4XCVQVOSzNHfVVOQBHdIyHH…"
```
→ `200 {"data": {…user…}}` ✓

---

## 5. Rotate — `/auth/refresh`

Request:
```bash
curl -s -X POST http://127.0.0.1:8765/api/v1/auth/refresh \
  -d '{"refresh_token":"NTh04pNpETHO60voBcKE95qtYXMrq0…"}'
```

Response:
```json
{
  "data": {
    "access_token": "102|U6bbV57WuVPhc6wYhHUMeiJXWYOJEG9DYl2coSWk7be1228a",
    "refresh_token": "MRwIAakHQYDwlyKe0jaJgH8PdCE5QwN8DTn60iD6TZBmohZyyY87…",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_expires_in": 5184000
  }
}
```

Invariants:
- `access_token` (102…) ≠ original (101…) ✓
- `refresh_token` ≠ original ✓
- Response carries NO `user` field (intentional — `/auth/me` is the source of truth)
- No legacy `token` alias on refresh response (only on login/register)

Rotation chain in DB:
```sql
SELECT id, access_token_id, replaced_by_id, revoked_at IS NOT NULL AS revoked
FROM refresh_tokens
WHERE user_id = '<student0>';
--  id | access_token_id | replaced_by_id | revoked
--   1 | 101             | 2              | t
--   2 | 102             |                | f
```
The OLD refresh row is `revoked_at != null` AND points at the NEW one via `replaced_by_id`. ✓

---

## 6. Reuse-detection — the brutal proof

The intended behaviour: if the OLD `refresh_token` (already rotated) is presented again, treat as token theft → revoke EVERY token the user owns → return `REFRESH_TOKEN_REUSED`.

The first implementation had a real bug — the throw was inside the rotation transaction, so the `revokeAllForUser` call inside the same transaction got rolled back. The client saw the right error code but other devices stayed logged in. Caught by the proof:

```sql
SELECT remaining_access, refresh_active, revoked_refresh
FROM (
  SELECT
    (SELECT count(*) FROM personal_access_tokens WHERE tokenable_id='…') AS remaining_access,
    (SELECT count(*) FROM refresh_tokens WHERE user_id='…' AND revoked_at IS NULL) AS refresh_active,
    (SELECT count(*) FROM refresh_tokens WHERE user_id='…' AND revoked_at IS NOT NULL) AS revoked_refresh
) x;
-- remaining_access: 3      ← BUG: should be 0
-- refresh_active:   2      ← BUG: should be 0
-- revoked_refresh:  1
```

Plus the Android-device token (untouched in the reuse flow) was still HTTP 200 — concrete evidence that "logout-everything" had been rolled back.

**Fix in commit `352077a`:** split `rotate()` into a pre-flight read (no transaction) that handles reuse-detection, followed by the locked rotation transaction. Reuse triggers `revokeAllForUser` BEFORE the throw, so the security action commits independently.

Post-fix re-proof:
```
=== state BEFORE reuse ===
access_tokens = 2, refresh_active = 2

=== first refresh on A (legit) ===
200 OK, new pair issued.

=== reuse old REFRESH_A — same plaintext, after rotation ===
401 {"error":{"code":"REFRESH_TOKEN_REUSED", …}}

=== state AFTER reuse ===
remaining_access  = 0       ✓
refresh_active    = 0       ✓
revoked_refresh   = 3       ✓ (the original A pair + the new A refresh + the B refresh)

=== Android device's untouched access ===
HTTP 401   ✓ (revoke-all caught it)

=== audit ===
auth.session.revoked | scope=all | reason=refresh_token_reuse_detected
```

The audit row commits BEFORE the throw, so it's persistent even though the controller returns an error.

---

## 7. Multi-device + sessions endpoints

Three logins with distinct `device_id`:

```bash
for D in "iPhone-Alice:ios:DEV-1" "Pixel-Alice:android:DEV-2" "MacBook-Alice:web:DEV-3"; do
  ...
done
```

```bash
curl -H "Authorization: Bearer <ACCESS_B>" http://127.0.0.1:8765/api/v1/me/sessions
```

Response (truncated, key fields):
```json
{
  "data": [
    {"id":104, "is_current":false, "device_name":"Pixel-B",        "device_type":"android", "device_id":"BBBB"},
    {"id":102, "is_current":false, "device_name":"Test-iPhone-15", "device_type":"ios",     "device_id":"AAAA-1111-BBBB"},
    {"id":103, "is_current":true,  "device_name":"iPhone-A",       "device_type":"ios",     "device_id":"AAAA"}
  ],
  "meta": {"total":3, "current_session_id":103}
}
```

Note `is_current` is correctly set only on the requesting device's row. ✓

### 7a — Refuse to revoke the current session via `/me/sessions/{id}`
```bash
curl -X DELETE http://127.0.0.1:8765/api/v1/me/sessions/103 -H "Authorization: Bearer <103's access>"
# 422 {"error":{"code":"CANNOT_REVOKE_CURRENT_SESSION", …}}
```
The endpoint refuses on purpose; `/auth/logout` is the canonical "kill my own session" path because it also clears cookies. ✓

### 7b — Revoke a different device's session
```bash
curl -X DELETE http://127.0.0.1:8765/api/v1/me/sessions/109 \
     -H "Authorization: Bearer <Pixel access>"
# 204
```
Then the revoked device's access token returns 401 on the next request. ✓

### 7c — Revoke all OTHER sessions
```bash
curl -X DELETE http://127.0.0.1:8765/api/v1/me/sessions \
     -H "Authorization: Bearer <Pixel access>"
# 200 {"data":{"access_revoked":1, "refresh_revoked":1}}
```
DB state shows only Pixel remains. MacBook access → 401. Pixel still 200. ✓

---

## 8. Logout — revokes the FULL pair

```bash
curl -X POST http://127.0.0.1:8765/api/v1/auth/logout \
     -H "Authorization: Bearer <Pixel access>"
# 204 + Set-Cookie clearing both wamadat_session and wamadat_refresh
```

DB state after logout: `access=0`, `refresh_active=0`. ✓

Trying to reuse the Pixel refresh:
```bash
curl -X POST http://127.0.0.1:8765/api/v1/auth/refresh \
     -d '{"refresh_token":"<old Pixel refresh>"}'
# 401 {"error":{"code":"REFRESH_TOKEN_REUSED", …}}
```

This is technically the reuse path (the token's `revoked_at` was set by logout). The behavioural result is correct — the token is dead — but the error code is `REUSED` rather than `INVALID`. Acceptable for v1; tracked as a precision improvement in remaining risks.

---

## 9. What this proof did NOT cover (intentional, deferred)

- **Real concurrent-rotation proof.** The race-loss path (`REFRESH_TOKEN_INVALID` when another rotation wins the lock) is established by code reading + lockForUpdate semantics, not by an actual two-process race. A pgbench harness against `/auth/refresh` (analogous to the B-H2 coupon harness) is tracked as a CP4-12 sub-item.
- **Access-token-natural-expiry behaviour.** The 60-minute TTL is set on the row and Sanctum respects `expires_at` at resolution time. Not exercised in a live `sleep 3600` proof; relies on Sanctum's built-in check. The check is unit-testable in CP4-12.
- **Refresh-token-natural-expiry behaviour.** Same posture — `expires_at` set, `rotate()` checks `isFuture()`, not exercised by a 60-day sleep.
- **Password-change-revokes-other-sessions.** NOT implemented in CP4-5 — tracked as R-OPEN-22.
- **Sustained reuse under load.** A flapping client that retries refresh during a flaky network may trigger reuse-detection on a legitimate user. The current strict-reuse policy is industry-standard but Auth0-style "10-second grace window" is a known refinement, deferred.

---

## 10. Re-running this bundle in the next session

The exact command sequence below reproduces every step. Run it before any auth-related change to verify the contract hasn't drifted.

```bash
cd backend
PHP=/c/laragon/bin/php/php-8.3.30-Win32-vs16-x64/php.exe

# Reset baseline for student0
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_tenants <<'SQL'
SET search_path = tenant_wamadat;
UPDATE users SET failed_login_attempts=0, locked_until=NULL WHERE email='student0@wamadat.demo';
DELETE FROM personal_access_tokens WHERE tokenable_id=(SELECT id FROM users WHERE email='student0@wamadat.demo');
DELETE FROM refresh_tokens         WHERE user_id   =(SELECT id FROM users WHERE email='student0@wamadat.demo');
SQL

# Server
$PHP artisan serve --host=127.0.0.1 --port=8765 &
sleep 4

# Login → keep both tokens
RESP=$(curl -s -X POST http://127.0.0.1:8765/api/v1/auth/login \
  -H "Content-Type: application/json" -H "Accept: application/json" -H "X-Tenant-Slug: wamadat" \
  -d '{"email":"student0@wamadat.demo","password":"StudentPass123","device_name":"Re-test","device_type":"ios","device_id":"RE-TEST"}')
ACCESS=$(echo "$RESP" | grep -oP '"access_token":"\K[^"]+')
REFRESH=$(echo "$RESP" | grep -oP '"refresh_token":"\K[^"]+')

# 1. /me works
curl -s -o /dev/null -w "GET /me → %{http_code}\n" \
  http://127.0.0.1:8765/api/v1/auth/me \
  -H "Authorization: Bearer $ACCESS" -H "X-Tenant-Slug: wamadat"
# Expected: 200

# 2. Rotate
RESP2=$(curl -s -X POST http://127.0.0.1:8765/api/v1/auth/refresh \
  -H "Content-Type: application/json" -H "X-Tenant-Slug: wamadat" \
  -d "{\"refresh_token\":\"$REFRESH\"}")
[ "$(echo "$RESP2" | grep -c access_token)" = "1" ] && echo "rotate OK"

# 3. Reuse old refresh → must revoke everything
curl -s -X POST http://127.0.0.1:8765/api/v1/auth/refresh \
  -H "Content-Type: application/json" -H "X-Tenant-Slug: wamadat" \
  -d "{\"refresh_token\":\"$REFRESH\"}" | grep REFRESH_TOKEN_REUSED && echo "reuse detected"

# 4. DB confirms zero active tokens for the user
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_tenants -tAc "SET search_path=tenant_wamadat;
SELECT (SELECT count(*) FROM personal_access_tokens WHERE tokenable_id=(SELECT id FROM users WHERE email='student0@wamadat.demo'))
  ||','||(SELECT count(*) FROM refresh_tokens WHERE user_id=(SELECT id FROM users WHERE email='student0@wamadat.demo') AND revoked_at IS NULL);"
# Expected: 0,0
```

If any of these drift from the expected values, something else has changed in auth — investigate before any further work.
