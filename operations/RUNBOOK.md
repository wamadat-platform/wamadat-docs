# Wamadat — Operations Runbook (S4-D9)

> Goal: any competent operator — **not only the founder** — can run, recover,
> and monitor Wamadat from this one page. Reduces bus-factor from 1.
> Paths assume Windows + Laragon PHP + native PostgreSQL 16 (current host).

## 1. Start / Stop the platform
```powershell
# Backend (Laravel + Filament + API) — port 8000
cd backend; php artisan serve --host=127.0.0.1 --port=8000
# Queue worker (jobs: emails, reconcile, webhooks)
cd backend; php artisan queue:work --tries=3
# Scheduler — MUST run for health to stay green (see §5)
#   Register ONE Windows Task: every 1 min -> `php artisan schedule:run`
# Frontend (Next.js storefront) — port 3000
cd frontend; npm run dev      # prod: npm run build && npm run start
```

## 2. Backup (daily)
```powershell
.\scripts\backup.ps1                       # dumps landlord + tenants -> backups\db\
# Schedule it: Windows Task Scheduler, daily, runs backup.ps1 (KeepDays=14).
```

## 3. Restore — DRILL (safe, run weekly) & REAL
```powershell
# DRILL (no real DB touched — proves the backup works):
.\scripts\restore.ps1 -DumpFile .\backups\db\wamadat_tenants-<stamp>.dump
# REAL restore (destructive — take a fresh backup FIRST):
.\scripts\restore.ps1 -DumpFile <file> -Target wamadat_tenants -DrillOnly:$false
```
Verified drill on test: dump 314.7 KB -> restore -> users/programs counts matched original.

## 4. Health / Uptime monitoring
- **Liveness (LB / uptime monitor):** `GET /up` (1-byte) and `GET /api/v1/health`.
- `/api/v1/health` returns **200** when CRITICAL deps are up (landlord_db,
  tenant_db, cache) — even if scheduler/queue are "degraded" (shown in body).
  Returns **503** only when a critical dep is down. Point UptimeRobot/Pingdom
  at `/up`; alert on the `status` field of `/api/v1/health` for degraded.
- **Errors:** Sentry is configured (`SENTRY_LARAVEL_DSN`). Confirm DSN is set in
  `.env`; test with `php artisan sentry:test`.

## 5. Scheduler (cron)
Five jobs run via `php artisan schedule:run` (must fire every minute):
`outbox:drain` (5m), `ops:scheduler-heartbeat` (1m), `orders:reconcile` (5m),
`webhooks:replay-queued` (2m), `ops:failed-jobs:purge` (weekly).
- Verify: `php artisan schedule:list`.
- If `/health` shows `scheduler: degraded`, the 1-minute Windows Task that runs
  `schedule:run` has stopped — re-enable it.

## 6. Logs
- Now rotates daily, 14-day retention (`config/logging.php` stack -> daily).
- Location: `backend/storage/logs/laravel-YYYY-MM-DD.log`.

## 7. Rollback
- **Code:** changes are git-tracked. `git revert <sha>` (or `git checkout <sha> -- <path>`)
  then redeploy. No change in this transformation touched production data.
- **DB migration:** `php artisan migrate:rollback` (landlord) / re-run the tenant
  migration's `down()`. The S4 drift migrations are idempotent + reversible.
- **Data:** restore the latest dump (§3, REAL restore) — only after a fresh backup.
- **Tenant lifecycle:** `tenant:suspend` / `tenant:archive` / `tenant:restore`.

## 8. CI gate (merge protection)
`.github/workflows/ci.yml` runs **pint --test → phpstan (L8) → pest** on every PR.
Enable branch protection on `main` requiring the `quality` check. Any failure
blocks the merge. Locally: `composer qa`.

## 9. Common ops
- Provision a new tenant: `php artisan tenant:provision` (runs schema + tenant migrations).
- Seed roles into a tenant: `php artisan tenant:seed-roles <slug>`.
- Create super admin: `php artisan wamadat:create-super-admin`.
- Drain stuck emails: `php artisan outbox:drain`.
- Retry failed webhooks: `php artisan webhooks:replay-queued`.
