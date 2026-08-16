# Ops Status Snapshot — 2026-05-13

**Captured:** end of session that delivered Phase A → Phase C part 1.
**Purpose:** ground-truth picture of the queue/cron/failed-job/outbox/webhook subsystems so the next session starts from facts, not assumptions.

All facts below are direct DB reads, `artisan` outputs, or filesystem grep — no inference. Reproduce with the commands shown in the right column.

---

## Queue infrastructure

| Fact | Value | Verify with |
|---|---|---|
| `QUEUE_CONNECTION` | `sync` | `grep QUEUE_CONNECTION backend/.env` |
| `config/queue.php` | does not exist | `ls backend/config/queue.php` (returns ENOENT) |
| `app/Jobs/` directory | does not exist | `ls backend/app/Jobs` (returns ENOENT) |
| Queue worker process | none | `ps -ef \| grep "queue:work"` |
| `failed_jobs` migration | applied | `landlord/2026_05_13_110000_create_failed_jobs_table.php` exists |
| `failed_jobs` row count | 0 | `psql -d wamadat_landlord -tAc "SELECT count(*) FROM failed_jobs"` |
| Filament `FailedJobResource` | wired, tenant-scoped | `app/Modules/Operations/Infrastructure/Filament/Resources/FailedJobResource.php` |

**Interpretation:** the queue layer is *structurally complete* (table + Filament UI + retry plumbing) but *operationally inactive* (no driver, no worker, no jobs). When Redis lands in Stage 0, the layer activates without code changes other than `.env`.

---

## Scheduled jobs (cron)

`php artisan schedule:list` returned (exact, post Phase C part 1):

```
*/5 * * * *  php artisan outbox:drain --limit=100      ........ outbox drain
*   * * * *  php artisan ops:scheduler-heartbeat       ........ heartbeat writer
*/5 * * * *  php artisan orders:reconcile              ........ order reconciliation
*/2 * * * *  php artisan webhooks:replay-queued --limit=50 .... webhook replay (NEW)
```

| Job | Purpose | File |
|---|---|---|
| `outbox:drain` | Drain `email_outbox` rows where `status=queued` AND `scheduled_for <= now()` | `app/Modules/Marketing/Infrastructure/Console/DrainEmailOutbox.php` |
| `ops:scheduler-heartbeat` | Writes `ops:scheduler:heartbeat` cache key (TTL 600s) for HealthController | `app/Modules/Operations/Infrastructure/Console/Commands/SchedulerHeartbeatCommand.php` |
| `orders:reconcile` | Query gateway for `awaiting_payment` orders older than 2 min and promote/fail/abandon | `app/Modules/Commerce/Infrastructure/Console/Commands/ReconcileOrdersCommand.php` |
| `webhooks:replay-queued` | Re-dispatch `payment_webhooks` rows with `result='queued'` | `app/Modules/Commerce/Infrastructure/Console/Commands/ReplayQueuedWebhooksCommand.php` |

**Local dev state:** no cron daemon is running on the dev machine. `Cache::get('ops:scheduler:heartbeat')` returns `null` ⇒ HealthController would report `scheduler: degraded`. **This is expected in dev** — production deployment runbook must register `* * * * * php artisan schedule:run` in system cron.

All four jobs use `->withoutOverlapping()` so a slow run never spawns a parallel duplicate.

---

## Email outbox

| Fact | Value |
|---|---|
| Table | `email_outbox` (tenant connection) |
| Total rows | 6 |
| `status='sent'` | 6 |
| `status='queued'` / `failed` / `bounced` / `complained` | 0 |
| Drain command max attempts | 5 (per `DrainEmailOutbox::MAX_ATTEMPTS`) |
| Dedup constraint | none (R-OPEN-6 — `idempotency_key` not yet added) |

Reproduce:
```sql
SET search_path = tenant_wamadat;
SELECT status, count(*) FROM email_outbox GROUP BY status;
```

---

## Payment webhooks

| Fact | Value |
|---|---|
| Table | `payment_webhooks` (tenant connection) |
| UNIQUE constraint | `(gateway, external_id, event_type)` |
| Total rows | 6 |
| `result='ok'` | 6 |
| `result='queued'` / `failed' / 'invalid_signature'` | 0 |
| Replay-drain command | `webhooks:replay-queued`, scheduled every 2 min |

Reproduce:
```sql
SET search_path = tenant_wamadat;
SELECT result, count(*) FROM payment_webhooks GROUP BY result;
```

Proof from Phase C part 1 (commit `4ff5efb`):
- Inserted a fake `result='failed'` row, flipped to `queued`, ran drain → row became `ok` with `processed_at` set.
- Second drain pass: `Replayed 0 ok, 0 failed.` (correct no-op).
- Re-running with the same `gateway_refund_id`: notifications & refunds counts unchanged ⇒ handler dedup held.

---

## Audit log integrity

| Fact | Value |
|---|---|
| Table | `audit_logs` (tenant connection) |
| Triggers | `audit_logs_no_update_delete` (BEFORE UPDATE/DELETE), `audit_logs_no_truncate` (BEFORE TRUNCATE) |
| Both ENABLED | yes (`tgenabled='O'`) |
| Privileges granted to PUBLIC | none of UPDATE, DELETE, TRUNCATE (revoked in B-H4 migration) |
| Caveat | Superuser DB roles bypass GRANT/REVOKE; trigger remains the last line. R-OPEN-2. |

Reproduce:
```sql
SET search_path = tenant_wamadat;
SELECT t.tgname, t.tgenabled
FROM pg_trigger t JOIN pg_class c ON c.oid = t.tgrelid
WHERE c.relname = 'audit_logs' AND NOT t.tgisinternal;
```

---

## Tenancy

| Fact | Value |
|---|---|
| Default driver | Spatie Multitenancy, schema-per-tenant |
| Tenant model | `App\Modules\Tenancy\Infrastructure\Eloquent\TenantModel` |
| Tenant slug used in dev | `wamadat` |
| Schema | `tenant_wamadat` |
| `current_tenant()` helper | hardened post B-H5 to re-resolve via `getKey()` when Spatie returns the base class (tinker/console paths) |

---

## Build / branch / version

| Fact | Value | Verify |
|---|---|---|
| Branch | `main` | `git branch --show-current` |
| Working tree | clean | `git status --short` |
| Tip commit | `4ff5efb` Phase C / part 1 — Webhook replay drain loop | `git log -1 --oneline` |
| Last 5 commits | see below | `git log --oneline -5` |

```
4ff5efb Phase C / part 1 — Webhook replay drain loop
317125f B-Hardening — closes 6 council findings before Phase C
3d92127 Phase B — Financial Safety + Security Hardening with proof
7d94788 Phase A — 7 admin 500s closed with browser proof
b517469 Emergency Council Audit: brutal verification matrix + rescue plan
```

---

## What to verify FIRST in the next session

Before any new code lands in Phase C part 2, re-run this snapshot script and diff. If any of these change unexpectedly, something else changed too:

```bash
cd /c/Users/U/wamadat-platform/backend
php artisan schedule:list                                   # 4 jobs expected
grep QUEUE_CONNECTION .env                                  # sync
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_tenants -c "SET search_path=tenant_wamadat; SELECT status, count(*) FROM email_outbox GROUP BY status; SELECT result, count(*) FROM payment_webhooks GROUP BY result;"
PGPASSWORD=postgres psql -h 127.0.0.1 -U postgres -d wamadat_landlord -tAc "SELECT count(*) FROM failed_jobs;"
node scripts/qa/crawl-admin-deep.mjs | tail -10              # 25/34 OK baseline
```
