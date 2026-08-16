# 🟠 Runbook — Redis Down

Redis is the queue broker, cache, and session store. If it's down, three things
fail in this order:

1. **Sessions** → users get logged out
2. **Queue** → emails, certificates, webhooks stop processing (jobs queue in memory and lost on restart)
3. **Cache** → app falls back to DB → DB load spikes

---

## Symptoms

- `HealthStatusWidget` flips `cache` and/or queue probe to red
- `QueueDepthWidget` shows `?` instead of a number (can't read queue depth)
- Sentry: `Predis\Connection\ConnectionException` or `RedisException: read error on connection`
- Users complain they're being logged out
- Outbox emails stop draining: `/admin/email-outboxes` shows `queued` count climbing without `sent` count moving

---

## Verify (under 60 seconds)

```bash
# 1. Health endpoint
curl -sf https://api.wamadat.academy/api/v1/health | jq '.checks.cache, .checks.queue'

# 2. Direct PING (use the production REDIS_HOST/PORT/PASSWORD).
# Omit `--tls` if your Redis is not behind TLS (managed providers usually use
# TLS in production; self-hosted on a private VPC may not). If `--tls` fails
# with "wrong version number", retry without it.
redis-cli -h $REDIS_HOST -p $REDIS_PORT -a "$REDIS_PASSWORD" --tls PING
# → expect: PONG

# 3. Queue depth from Laravel
php artisan tinker --execute="echo Redis::connection()->llen('queues:default');"
```

If `redis-cli` errors:
- **`Connection refused`** → instance down — check the managed Redis provider's status page
- **`NOAUTH Authentication required`** → password env mismatch — check `REDIS_PASSWORD` in `.env.production`
- **`MOVED <slot> <ip>`** → cluster failover, client should follow but reload if it doesn't
- **OOM** → memory exhausted — see Recover §3

---

## Mitigate (stop bleeding NOW)

1. **Switch sessions to cookie driver temporarily** so users stop getting logged out:
   ```bash
   # In production .env (or override via systemd Environment=)
   SESSION_DRIVER=cookie
   php artisan config:cache
   php artisan octane:reload    # or restart php-fpm
   ```
   This is reversible. Cookies are signed + encrypted, so it's secure for a brief window.

2. **Stop queue workers** so they stop crash-looping:
   ```bash
   systemctl stop wamadat-queue@*
   ```

3. **Disable the Sentry breadcrumb noise** by raising log level on Redis errors
   in `config/logging.php` if it's drowning the inbox. Optional — only if Sentry
   is unusable.

---

## Recover

1. Restore Redis (provider-side) OR fail over to replica.

2. Re-probe: `curl -sf https://api.wamadat.academy/api/v1/health | jq`

3. Restart workers + flip sessions back to redis:
   ```bash
   # Revert SESSION_DRIVER=redis in .env.production
   php artisan config:cache
   php artisan octane:reload
   systemctl start wamadat-queue@worker1 wamadat-queue@worker2
   ```

4. Drain the outbox manually to catch up:
   ```bash
   php artisan outbox:drain
   ```
   Watch `/admin/email-outboxes` — `queued` should decrement, `sent` increment.

5. Failed jobs from the outage period: bulk-retry from `/admin/failed-jobs` (audited automatically as `ops.failed_job.retried_bulk`).

---

## Post-incident

- [ ] Capacity check: was this OOM? If yes, size up Redis or split cache + queue into two instances
- [ ] Persistence audit: confirm AOF / RDB snapshots are configured on the managed instance (default OFF on some providers!)
- [ ] If sessions were forcibly invalidated, post a status-page note ("users may have been logged out briefly")
- [ ] Open `docs/audit/incidents/YYYY-MM-DD-redis-down.md`

---

## Don'ts

- ❌ Don't `FLUSHALL` "to start clean" — you'll delete cache + sessions + queue at once
- ❌ Don't leave `SESSION_DRIVER=cookie` permanently — cookies have a 4KB limit and slow every request
- ❌ Don't restart Redis without checking what's in memory if you suspect a wedged queue — `redis-cli --scan --pattern 'queues:*'` first

---

## Side-effect to watch: DB load

While Redis is down, every cache miss hits Postgres. If `db-down.md` symptoms
start to appear during a Redis outage, that's a **cascade** — page someone
immediately and consider maintenance mode.
