# Ops Status Snapshot — 2026-05-13 (Phase D Part 1 close)

**Supersedes:** `OPS_STATUS_SNAPSHOT_2026-05-13-phase-c-3.md` for any field marked **CHANGED** below. Everything else inherits unchanged.

Reproduce: every line shows the command in the right column.

---

## API surface (NEW for Phase D)

| Fact | Value | Verify with |
|---|---|---|
| Total endpoints under `/api/v1` | 90 | `php artisan route:list \| grep "api/v1" \| wc -l` |
| Endpoints outside `/api/v1` | 0 | `php artisan route:list \| grep -E "^[A-Z]+\s+api/" \| grep -v "api/v1"` |
| Inventory doc | `docs/api/01-api-inventory.md` | view file |
| Canonical error shape spec | `docs/api/02-response-shape-rebuild.md` | view file |
| Breaking-changes log | `docs/api/CHANGELOG-v1.md` | view file |
| Phase D remaining scope | `docs/api/03-phase-d-remaining-scope.md` | view file |

### API response shape contract

**SUCCESS** — unchanged (`{data: …}` envelope or domain-specific flat shape).

**ERROR** — canonical (**CHANGED** by commit `ae2dd5e`):
```json
{
  "error": {
    "code": "STABLE_SCREAMING_SNAKE_CASE",
    "message": "Localized human-readable",
    "status": 422,
    "details": null | {field: [string, ...]} | [{...}]
  }
}
```

**Exclusions** (deliberately not normalized):
- `POST /api/v1/webhooks/{gateway}` — gateway contract.
- `POST /api/v1/webhooks/resend` — gateway contract.
- `GET /api/v1/health` — monitoring contract (503 with structured `checks.*`).

Verify with:
```bash
# canonical error
curl -s http://localhost:8000/api/v1/zzz-no-such -H "X-Tenant-Slug: wamadat"
# → {"error":{"code":"RESOURCE_NOT_FOUND",...}}

# success unchanged
curl -s http://localhost:8000/api/v1 -H "X-Tenant-Slug: wamadat"
# → {"api":"wamadat-v1","tenant":"wamadat"}

# /health excluded — keeps structured checks shape
curl -s http://localhost:8000/api/v1/health -H "X-Tenant-Slug: wamadat"
# → {"status":"degraded","checks":{...}}
```

### Code vocabulary used in this release
`INVALID_CREDENTIALS`, `UNAUTHENTICATED`, `PERMISSION_DENIED`, `RESOURCE_NOT_FOUND`, `CONFLICT`, `METHOD_NOT_ALLOWED`, `VALIDATION_FAILED`, `OPERATION_FAILED`, `TENANT_REQUIRED`, `RATE_LIMITED`, `SERVER_ERROR`, `HTTP_ERROR_{N}`.

The full catalogue with AR + EN messages lands in CP4-9 next session.

---

## Queue infrastructure (UNCHANGED from Phase C Part 3)

`QUEUE_CONNECTION=database`. Landlord `jobs` + `failed_jobs` tables. One job class (`SendOutboxEmailJob`) on `notifications` lane. Worker runbook: `docs/runbooks/06-queue-worker.md`.

---

## Scheduled jobs (UNCHANGED from Phase C Part 3 — 5 jobs)

```
*/5 * * * *  outbox:drain --limit=100
*   * * * *  ops:scheduler-heartbeat
*/5 * * * *  orders:reconcile
*/2 * * * *  webhooks:replay-queued --limit=50
0   3 * * 0  ops:failed-jobs:purge --days=30
```

---

## Audit log integrity (UNCHANGED)

Both `audit_logs_no_update_delete` and `audit_logs_no_truncate` triggers ENABLED. `PUBLIC` revoked from UPDATE/DELETE/TRUNCATE.

---

## DB rows (CHANGED — Phase D test runs)

| Table | Pre-D | Post-D-Part-1 |
|---|---|---|
| `email_outbox` | 13 sent | 13 sent (no new test rows committed) |
| `payment_webhooks` | 6 ok | 6 ok (unchanged) |
| `notifications` | varied | varied (no Phase D writes) |
| `jobs` | 0 | 0 (cleared after CP3-9 proof) |
| `failed_jobs` | 0 | 0 |

---

## Build / branch / version

| Fact | Value |
|---|---|
| Branch | `main` |
| Tip commit at this snapshot | the Phase D Part 1 closeout commit (this one) |
| Working tree | clean after this commit |
| Last 7 commits | see below |

```
<closeout>  Phase D Part 1 — checkpoint: remaining-scope doc, breaking-changes log, OPEN_RISKS update
ae2dd5e     CP4-1 + CP4-2 + CP4-3 — API inventory, canonical error shape, versioning audit
4be7bc3     CP3-10 — Update OPEN_RISKS + OPS snapshot for Phase C Part 3
8431ec8     CP3-3 + CP3-7 + CP3-8 + CP3-9 — Worker ops + health + broadcast dedup + async proof
57ea535     CP3-1 + CP3-2/4/5/6 — Queue productionization foundation
eeb834a     CP2-6 — Notification-center Playwright verification
69d9dfd     CP2-5 — Coupon redemptions UNIQUE constraint
```

---

## Diff vs Phase C Part 3 snapshot

| Surface | Phase C Part 3 | Phase D Part 1 |
|---|---|---|
| API endpoints | 90 (unaudited inventory) | 90 (canonical inventory `docs/api/01-api-inventory.md`) |
| /api/v1 coverage | unaudited | 100% (90/90) |
| Error response shape | 6 distinct shapes observed | 1 canonical shape (`{error: …}`) |
| Frontend error consumer | latent bug — couldn't parse INVALID_CREDENTIALS | reads canonical first, legacy fallback for transition window |
| API breaking-changes log | not maintained | `docs/api/CHANGELOG-v1.md` started |
| Open risks | 12 (R-OPEN-1..12, several closed) | 12 plus 9 new (R-OPEN-13..21) for Phase D items |

---

## Pre-flight for the next session

Run these before any code changes — they should match the values above. Drift means something silently changed and needs investigation:

```bash
cd /c/Users/U/wamadat-platform/backend
php artisan schedule:list                                         # 5 jobs
grep QUEUE_CONNECTION .env                                        # database
php artisan route:list | sed 's/\x1b\[[0-9;]*m//g' | grep "api/v1" | wc -l   # 90

# canonical error contract still intact
php artisan serve --host=127.0.0.1 --port=8765 &
sleep 3
curl -s http://127.0.0.1:8765/api/v1/zzz-no-such -H "X-Tenant-Slug: wamadat" | head -c 200
# expected: {"error":{"code":"RESOURCE_NOT_FOUND",...}}

curl -s http://127.0.0.1:8765/api/v1/health -H "X-Tenant-Slug: wamadat" | head -c 80
# expected: {"status":"degraded","checks":{...}}  (NOT canonical error shape)

curl -s http://127.0.0.1:8765/api/v1 -H "X-Tenant-Slug: wamadat" | head -c 80
# expected: {"api":"wamadat-v1","tenant":"wamadat"}
```

If any of those drift unexpectedly, find out what else changed first.

---

## Open Phase D items (re-stated for convenience)

In recommended execution order for the next session:

1. **CP4-9** — Error codes catalogue (depends on canonical shape — ready).
2. **CP4-5** — Mobile auth flow + token refresh (depends on nothing).
3. **CP4-4** — Pagination retrofit across list endpoints (heaviest).
4. **CP4-6** — Accept-Language middleware (depends on CP4-9 for translation keys).
5. **CP4-10** — Rate-limit audit (lightweight, can interleave).
6. **CP4-11** — Signed / scoped file URLs.
7. **CP4-7** — OpenAPI / Scribe.
8. **CP4-8** — Postman collection (depends on CP4-7).
9. **CP4-12** — Contract tests (depends on EVERYTHING above).

Estimated total: 25-35 hours of focused work at Phase B / B-Hardening / Phase C proof depth.
