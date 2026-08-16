# Ops Status Snapshot — 2026-05-14 (CP4-5 close)

**Supersedes:** `OPS_STATUS_SNAPSHOT_2026-05-13-phase-d-1.md` for any field marked **CHANGED**. Everything else inherits unchanged.

---

## Auth surface (NEW for CP4-5)

| Fact | Value | Verify with |
|---|---|---|
| Auth model | **Two-token** — access + refresh | `cat docs/api/05-cp4-5-auth-proof-bundle.md` |
| Access TTL | 60 minutes | `grep ACCESS_TTL_MIN backend/app/Modules/IdentityAccess/Application/Services/TokenService.php` |
| Refresh TTL | 60 days | `grep REFRESH_TTL_DAYS …/TokenService.php` |
| Refresh table | `tenant_<slug>.refresh_tokens` | `\d refresh_tokens` |
| Refresh transport | body of `POST /auth/refresh` OR `wamadat_refresh` httpOnly cookie path-scoped to `/api/v1/auth` | code review |
| Rotation policy | Single-use; new pair issued on every refresh; old marked `revoked_at` + `replaced_by_id` | inspect `refresh_tokens` rows |
| Reuse detection | Old refresh after rotation → `revokeAllForUser` + `REFRESH_TOKEN_REUSED` (401) | live proof in `05-cp4-5-auth-proof-bundle.md` §6 |
| Session inspection | `GET /me/sessions` | `curl …/me/sessions -H "Authorization: Bearer …"` |
| Session revoke (one device) | `DELETE /me/sessions/{id}` | proof §7b |
| Revoke-all-others | `DELETE /me/sessions` | proof §7c |
| Logout | `POST /auth/logout` — revokes the FULL pair (access + paired refresh) | proof §8 |

### New API endpoints

```
POST   /api/v1/auth/refresh
GET    /api/v1/me/sessions
DELETE /api/v1/me/sessions/{id}
DELETE /api/v1/me/sessions
```

Endpoint count: **94** (was 90 in Phase D Part 1 snapshot).

### Response shape — login / register (CHANGED, BREAKING — see CHANGELOG entry)

```jsonc
{
  "data": {
    "user": { ... },
    "access_token": "...",      // 60-min TTL
    "refresh_token": "...",     // 60-day TTL
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_expires_in": 5184000,
    "token": "..."              // DEPRECATED alias = access_token (R-OPEN-24)
  }
}
```

### Token vocabulary (additions to CP4-2 closed code list)

`REFRESH_TOKEN_MISSING`, `REFRESH_TOKEN_INVALID`, `REFRESH_TOKEN_EXPIRED`, `REFRESH_TOKEN_REUSED`, `ACCOUNT_INACTIVE`, `CANNOT_REVOKE_CURRENT_SESSION`, `SESSION_NOT_FOUND`. Full curated list lands in CP4-9.

### Audit actions emitted by TokenService

- `auth.token.issued` — on every login + register + rotation.
- `auth.token.rotated` — on `/auth/refresh` success, carries `old_refresh_id` + `new_refresh_id`.
- `auth.session.revoked` — three scopes:
  - `scope=current` (logout)
  - `scope=specific_device` (DELETE /me/sessions/{id})
  - `scope=all` (DELETE /me/sessions, OR reuse-detection; the latter sets `reason=refresh_token_reuse_detected`)

---

## DB rows (CHANGED — CP4-5 test runs)

| Table | Pre-CP4-5 | Post-CP4-5 |
|---|---|---|
| `personal_access_tokens` legacy (`expires_at IS NULL`) | 87 | **81** (R-OPEN-23 — operator-purgeable) |
| `personal_access_tokens` new (with `expires_at`) | 0 | 0 (test rows cleaned) |
| `refresh_tokens` | does not exist | exists, 0 active, 3 revoked (student0 test artifacts) |

---

## Everything else — UNCHANGED from Phase D Part 1 snapshot

- QUEUE_CONNECTION = `database`. Cron schedule = 5 jobs. Audit-log triggers ENABLED.
- Canonical error shape live for all `/api/*` except `/health` and `/webhooks/*`.
- `NormalizeApiErrorResponse` middleware rewrites legacy `{message}` shapes.
- Health endpoint reports per-lane queue depths + heartbeat.

---

## Build / branch / version

| Fact | Value |
|---|---|
| Branch | `main` |
| Tip commit before closeout | `352077a CP4-5 — Mobile auth + token refresh` |
| Tip at this snapshot | the closeout commit you're reading (writes this file + OPEN_RISKS update) |
| Working tree | clean after the closeout commit |

```
<closeout>  Phase D / CP4-5 closeout — proof bundle + OPEN_RISKS + OPS snapshot
352077a     CP4-5 — Mobile auth + token refresh
1ac3b8e     Phase D Part 1 closeout — checkpoint
ae2dd5e     CP4-1 + CP4-2 + CP4-3 — API inventory, canonical error shape, versioning audit
4be7bc3     CP3-10 — Update OPEN_RISKS + OPS snapshot for Phase C Part 3
```

---

## Diff vs Phase D Part 1 snapshot

| Surface | Phase D Part 1 | Phase D Part 2 (CP4-5) |
|---|---|---|
| API endpoint count | 90 | **94** |
| Auth response shape | `{data:{user,token,token_type}}` | `{data:{user,access_token,refresh_token,token_type,expires_in,refresh_expires_in,token<deprecated>}}` |
| Token lifetime | no expiry (`expires_at IS NULL`) | 60-min access, 60-day refresh |
| Token rotation | none | single-use, rotation chain, reuse-detect-and-revoke-all |
| Multi-device tracking | none | `device_name`, `device_type`, `device_id`, `last_ip`, `last_user_agent` |
| Sessions endpoint | none | `GET/DELETE /me/sessions` + `DELETE /me/sessions/{id}` |
| Logout behaviour | revoked current access only | revokes the full pair (access + refresh) |
| Refresh-on-401 (frontend) | none | single-flight `attemptRefresh()` + retry-once hook in `lib/api/client.ts` |
| Audit coverage | unchanged | `auth.token.issued`, `auth.token.rotated`, `auth.session.revoked` (3 scopes) |
| Open risks | R-OPEN-13..21 + base 1..12 | + R-OPEN-22..26 (CP4-5 residual) |

---

## Pre-flight for the next session

```bash
cd /c/Users/U/wamadat-platform/backend
PHP=/c/laragon/bin/php/php-8.3.30-Win32-vs16-x64/php.exe

$PHP artisan route:list | sed 's/\x1b\[[0-9;]*m//g' | grep "api/v1" | wc -l    # 94
$PHP artisan route:list | sed 's/\x1b\[[0-9;]*m//g' | grep -E "auth/refresh|me/sessions"
# expected: 4 lines — POST auth/refresh, GET me/sessions, DELETE me/sessions, DELETE me/sessions/{id}

grep QUEUE_CONNECTION .env                                                       # database

# Live contract still alive
$PHP artisan serve --host=127.0.0.1 --port=8765 &
sleep 3

curl -s http://127.0.0.1:8765/api/v1/zzz-no-such -H "X-Tenant-Slug: wamadat" | head -c 100
# {"error":{"code":"RESOURCE_NOT_FOUND",...

curl -s http://127.0.0.1:8765/api/v1/health -H "X-Tenant-Slug: wamadat" | head -c 80
# {"status":"degraded","checks":{...

curl -s -X POST http://127.0.0.1:8765/api/v1/auth/refresh \
  -H "Content-Type: application/json" -H "X-Tenant-Slug: wamadat" -d '{}' | head -c 150
# {"error":{"code":"REFRESH_TOKEN_MISSING",...   (proves the endpoint is wired + canonical)
```

If any of these drift, investigate before any new code.

---

## Open Phase D items (re-stated)

Locked order per the operator: CP4-9 → CP4-4 → CP4-11 → CP4-7 → CP4-8 → CP4-12. CP4-6 / CP4-10 interleave. CP4-5 closed in this commit chain.
