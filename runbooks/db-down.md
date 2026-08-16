# 🔴 Runbook — Database Down

Postgres is either unreachable or returning errors. Affects ALL writes and most
reads. The platform degrades to read-from-cache only.

---

## Symptoms

- `HealthStatusWidget` flips landlord_db and/or tenant_db to red
- `GET /api/v1/health` returns 503 with the failed probe in the response body
- Sentry inbox fills with `PDOException` / `SQLSTATE[08006]` (connection failure)
  or `SQLSTATE[57P03]` (DB starting up)
- Users see Next.js error boundaries on dashboard / learn / checkout
- `/admin/failed-jobs` count climbs (jobs trying to write fail and queue)

---

## Verify (under 60 seconds)

```bash
# 1. Hit the health endpoint directly — bypasses cache
curl -sf https://api.wamadat.academy/api/v1/health | jq '.checks.landlord_db, .checks.tenant_db'

# 2. From an ops shell on the app server
psql "$DB_URL" -c "SELECT 1;" && echo "landlord OK"
psql "$DB_TENANT_URL" -c "SELECT 1;" && echo "tenant OK"

# 3. Check the managed Postgres provider's status page
#    (RDS / Cloud SQL / Neon / etc.)
```

If `psql` errors:
- **`could not connect`** → network / firewall / DNS issue (rare; managed Postgres) — escalate to infra
- **`password authentication failed`** → credentials rotated outside our knowledge — check the provider console
- **`the database system is starting up`** → recent failover/restart, wait 30-60s then re-probe
- **`too many connections`** → app is leaking connections OR provider is sized too small

---

## Mitigate (stop bleeding NOW)

1. **Put the frontend in maintenance mode** so users see a clean message, not
   stack traces:
   ```bash
   php artisan down --secret="<unique-passphrase>" --render="errors::503"
   ```
   You can still reach `/admin/<secret>` to operate while everyone else gets 503.

2. **Stop queue workers** so jobs stop racking up in `failed_jobs`:
   ```bash
   # systemd-managed worker (recommended deployment)
   systemctl stop wamadat-queue@*

   # OR — supervisor
   supervisorctl stop wamadat-queue:*

   # OR — kill the running worker processes (last resort)
   pkill -f 'artisan queue:work'
   ```
   (Note: `queue:pause` only exists when Laravel Horizon is installed. This
   project deliberately did NOT add Horizon — Filament ops panel covers the
   need. So pause the workers via the supervisor that started them.)

3. **DO NOT** retry failed jobs while the DB is still bad — that's how you
   create duplicate writes when it comes back.

---

## Recover

1. Wait for the provider's incident to resolve, OR fail over to the read replica:
   - Promote replica → update `DB_HOST` / `DB_TENANT_HOST` in production env
   - Reload app: `php artisan config:cache && php artisan octane:reload` (or
     restart php-fpm / Octane workers)

2. Re-probe health: `curl -sf https://api.wamadat.academy/api/v1/health | jq`
   Wait for all 8 probes to be green before lifting maintenance.

3. Bring services back up:
   ```bash
   php artisan up
   php artisan queue:work --tries=3 --max-time=3600     # or restart systemd unit
   ```

4. Retry failed jobs from the UI (`/admin/failed-jobs` → bulk retry). The retry
   action audits to `audit_logs` automatically.

---

## Post-incident

- [ ] Audit row: every retried job already writes `ops.failed_job.retried` — confirm in `/admin/audit-logs`
- [ ] Sentry release tag: filter to the outage window, verify no straggler errors
- [ ] Open a post-mortem note in `docs/audit/incidents/YYYY-MM-DD-db-down.md`
- [ ] If this was a **connection-limit** failure, raise the limit OR introduce
      PgBouncer before the next post-mortem
- [ ] If the **secondary** had stale data after failover, validate PITR settings
      with the provider

---

## Don'ts

- ❌ Don't `migrate --force` to "fix" a stuck migration during an outage — that's how Phase 4.2 mess happened on staging. The migration is fine; the DB isn't.
- ❌ Don't retry failed jobs while landlord_db is red — wait for the green probe first.
- ❌ Don't lift `php artisan down` until both `landlord_db` AND `tenant_db` probes are green for two consecutive minutes.
