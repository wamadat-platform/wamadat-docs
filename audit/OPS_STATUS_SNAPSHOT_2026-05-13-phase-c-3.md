# Ops Status Snapshot — 2026-05-13 (Phase C Part 3 close)

**Captured:** end of session that delivered queue productionization.
**Supersedes:** `OPS_STATUS_SNAPSHOT_2026-05-13.md` (Phase C Part 1 baseline). That earlier snapshot is kept for diff'ing the trajectory.

All facts below are direct DB reads, `artisan` outputs, or filesystem grep — no inference. Reproduce with the commands in the right column.

---

## Queue infrastructure

| Fact | Value | Verify with |
|---|---|---|
| `QUEUE_CONNECTION` | `database` | `grep QUEUE_CONNECTION backend/.env` |
| `DB_QUEUE_CONNECTION` | `landlord` | same |
| `DB_QUEUE_TABLE` | `jobs` | same |
| `config/queue.php` | published, default=database | `head -30 backend/config/queue.php` |
| Jobs table | `wamadat_landlord.jobs` | `psql -d wamadat_landlord -c "\d jobs"` |
| Failed-jobs table | `wamadat_landlord.failed_jobs` | already existed pre-CP3 |
| Job classes count | 1 (`SendOutboxEmailJob`) | `find app/Modules -path "*/Jobs/*.php"` |
| Worker driver | database (Postgres SELECT … FOR UPDATE SKIP LOCKED) | implicit in driver |
| Worker process | dev: `php artisan queue:work` (manual); prod: supervisor / systemd | `docs/runbooks/06-queue-worker.md` |

### Queue lanes (priority high → low)

| Lane | Owner | Current usage |
|---|---|---|
| `critical` | reserved | empty — no job classes yet |
| `payments` | reserved | empty — RefundService still calls gateway inline |
| `notifications` | active | `SendOutboxEmailJob` |
| `default` | catch-all | empty |

Worker command (production): `php artisan queue:work --queue=critical,payments,notifications,default --tries=3 --timeout=120 --sleep=3 --max-time=3600`.

### Retry / backoff policy

`SendOutboxEmailJob`:
- `public int $tries = 3;`
- `backoff(): [10, 30, 90]` — exponential-ish (seconds).
- `public int $timeout = 60;` — Resend usually < 3s; 60s is generous.
- After 3 exhausted attempts → row lands in `failed_jobs` AND outbox row flips to `status=failed` via `failed()` hook.

Recovery: `/admin/failed-jobs` UI → `Retry` action OR `php artisan queue:retry all`. Proven end-to-end in commit `57ea535`.

---

## Scheduled jobs (cron)

```
*/5 * * * *  php artisan outbox:drain --limit=100         (safety net for queue worker down)
*   * * * *  php artisan ops:scheduler-heartbeat
*/5 * * * *  php artisan orders:reconcile
*/2 * * * *  php artisan webhooks:replay-queued --limit=50
0   3 * * 0  php artisan ops:failed-jobs:purge --days=30
```

Reproduce: `php artisan schedule:list`.

The `outbox:drain` cron is **belt-and-braces** post-CP3 — the queue worker is the primary mover; drain handles "worker is down" recovery.

---

## Email outbox

| Fact | Value |
|---|---|
| Total rows | 13 |
| `status='sent'` | 13 |
| `status='queued'` / `failed' / `bounced` / `complained` | 0 |
| Dedup constraint | `idempotency_key` partial UNIQUE (CP2-1) |
| Auto-dispatch | Yes — `OutboxService` dispatches `SendOutboxEmailJob` after insert |

Reproduce:
```sql
SET search_path = tenant_wamadat;
SELECT status, count(*) FROM email_outbox GROUP BY status;
```

---

## Notifications (in-app feed)

| Fact | Value |
|---|---|
| Dedup constraint | `idempotency_key` partial UNIQUE (CP2-2) |
| `createIdempotent()` helper | `NotificationModel::createIdempotent($attrs, $key)` |
| Call sites using key | CheckoutService (order.paid, enrollment.created) + RefundService + RefundWebhookHandler (shared `order.refunded:{refund_id}` key) |

---

## Payment webhooks

| Fact | Value |
|---|---|
| Table | tenant `payment_webhooks` |
| UNIQUE | `(gateway, external_id, event_type)` |
| Replay drain | `webhooks:replay-queued` every 2 min |

---

## Audit log integrity

| Fact | Value |
|---|---|
| Triggers | `audit_logs_no_update_delete` (UPDATE/DELETE) + `audit_logs_no_truncate` (TRUNCATE) |
| Both ENABLED | yes (`tgenabled='O'`) |
| PUBLIC grants | UPDATE/DELETE/TRUNCATE REVOKED |

---

## Coupon redemptions

| Fact | Value |
|---|---|
| UNIQUE | `(coupon_id, user_id, order_id)` (CP2-5) |
| Race protection | `lockForUpdate` in `CouponService::recordRedemption` (primary) + UNIQUE (backstop) |

---

## /api/v1/health output shape (post-CP3-7)

```json
{
  "status": "healthy|degraded",
  "checks": {
    "landlord_db":  { "ok": true },
    "tenant_db":    { "ok": true },
    "cache":        { "ok": true },
    "resend_configured": { "ok": true },
    "queue": {
      "ok": true,
      "total": 0,
      "depths": { "critical": 0, "payments": 0, "notifications": 0, "default": 0 },
      "oldest_reserved_age_sec": 0
    },
    "failed_jobs": {
      "ok": true,
      "count": 0,
      "count_last_24h": 0
    },
    "scheduler": {
      "ok": false|true,
      "last_beat_at": "...",
      "last_beat_ago_sec": 0,
      "window_sec": 90
    },
    "webhooks": { "ok": true, "last_hour": 0, "failed": 0 }
  }
}
```

Dev environments report `status=degraded` because the scheduler heartbeat is missing (no system cron in dev). Production with cron returns `healthy`.

---

## Build / branch / version

| Fact | Value |
|---|---|
| Branch | `main` |
| Tip commit | `8431ec8 CP3-3 + CP3-7 + CP3-8 + CP3-9 — Worker ops + health + broadcast dedup + async proof` |
| Last 6 commits | see below |

```
8431ec8 CP3-3 + CP3-7 + CP3-8 + CP3-9 — Worker ops + health + broadcast dedup + async proof
57ea535 CP3-1 + CP3-2/4/5/6 — Queue productionization foundation
eeb834a CP2-6 — Notification-center Playwright verification
69d9dfd CP2-5 — Coupon redemptions UNIQUE constraint
402ed3b CP2-4 — Failed-job retention cleanup cron
98ba6c0 CP2-3 — Mail dispatch via outbox (sync → async)
```

---

## Diff vs OPS_STATUS_SNAPSHOT_2026-05-13.md (Part 1 baseline)

| Surface | Before (Part 1) | After (Part 3) |
|---|---|---|
| QUEUE_CONNECTION | sync | **database** |
| Jobs table | absent | **present (landlord)** |
| Job classes | 0 | 1 (`SendOutboxEmailJob`) |
| Worker process | none | documented + provable; supervisor + systemd alternatives |
| Email outbox idempotency | none | `idempotency_key` partial UNIQUE |
| Notification idempotency | none | `idempotency_key` partial UNIQUE |
| Mail dispatch | sync `Mail::send()` | async via outbox → worker |
| Failed-job retention | none | weekly purge cron |
| Coupon redemption UNIQUE | (coupon_id, user_id) non-unique | (coupon_id, user_id, order_id) UNIQUE |
| Notification bell verification | manual | Playwright E2E |
| BroadcastService idempotency | none (admin-confirm only) | minute-bucket batch key |
| /health queue check | total only | total + per-lane depths + oldest-reserved-age |
| /health failed_jobs | total only | total + last-24h gating |
| Cron jobs | 4 | 5 |
| Scheduler heartbeat | dev-only known | unchanged (R-OPEN-9 production cron item) |

---

## What to re-verify FIRST in the next session

```bash
cd /c/Users/U/wamadat-platform/backend
php artisan schedule:list                                   # 5 jobs expected
grep QUEUE_CONNECTION .env                                  # database
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_landlord -c "SELECT (SELECT count(*) FROM jobs) AS jobs, (SELECT count(*) FROM failed_jobs) AS failed_jobs;"
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_tenants -c "SET search_path=tenant_wamadat; SELECT status, count(*) FROM email_outbox GROUP BY status; SELECT result, count(*) FROM payment_webhooks GROUP BY result;"
curl -s http://localhost:8000/api/v1/health | head -c 300
```

If any of these change unexpectedly (e.g. `QUEUE_CONNECTION` flipped, or `jobs` table missing), something else likely changed too — investigate.
