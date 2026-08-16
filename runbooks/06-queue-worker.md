# Queue Worker Runbook — CP3-3

## What this runbook covers
- Local dev: how to run the worker on Windows / Laragon.
- Production: supervisor and systemd alternatives, graceful restart,
  monitoring.
- Recovery: failed-job retry, draining stuck queues, killing a hung
  worker.

## Architecture in one paragraph
The platform uses Laravel's `database` queue driver against the
landlord Postgres connection (`jobs` and `failed_jobs` tables). Stage 0
chose database over Redis to avoid Memurai/Redis provisioning during
the soft-launch sprint. Switching to Redis later is one env-var flip
(see "Migrating to Redis" below). Queue lanes in priority order:
**critical → payments → notifications → default**.

## Local dev

Run a worker in a separate terminal:

```bash
cd backend
php artisan queue:work \
  --queue=critical,payments,notifications,default \
  --tries=3 \
  --timeout=120 \
  --sleep=3
```

Flags:
- `--queue=...` — comma-separated priority order; left lane drained first.
- `--tries=3` — fallback if the job class has no `$tries`; our jobs
  set their own.
- `--timeout=120` — kill a job after 2 minutes (Postgres `retry_after=90`
  must be smaller than the worker timeout in order for the job to
  actually re-release before the next pickup; the 90/120 split is
  deliberate).
- `--sleep=3` — when there are no jobs, sleep 3s before polling again
  (keeps idle CPU near zero).

For one-off testing (no daemon):
```bash
php artisan queue:work --queue=notifications --once --verbose
```

`--once` exits after the first batch. `--verbose` prints `RUNNING`/`DONE`/
`FAIL` lines so you can watch progress.

## Production — supervisor (recommended)

Create `/etc/supervisor/conf.d/wamadat-worker.conf`:

```ini
[program:wamadat-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/wamadat-platform/backend/artisan queue:work
  --queue=critical,payments,notifications,default
  --tries=3
  --timeout=120
  --sleep=3
  --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/log/supervisor/wamadat-worker.log
stopwaitsecs=125
```

Notes:
- `numprocs=2` — two parallel workers consuming from all lanes. Scale
  up by raising this number; Postgres `SELECT … FOR UPDATE SKIP LOCKED`
  inside Laravel's queue driver keeps jobs from being claimed twice.
- `--max-time=3600` — each worker process exits after 1 hour, supervisor
  immediately respawns it. This bounds memory leaks from PHP long-running
  processes.
- `stopwaitsecs=125` — must be ≥ `--timeout` so supervisor doesn't kill
  the worker mid-job during a deploy.

Reload + restart on deploy:

```bash
sudo supervisorctl reread
sudo supervisorctl update
php artisan queue:restart       # tells each worker to exit cleanly after the current job
```

`queue:restart` is the canonical graceful-restart signal. Supervisor
notices the exit and respawns within seconds.

## Production — systemd (alternative)

`/etc/systemd/system/wamadat-worker@.service`:

```ini
[Unit]
Description=Wamadat queue worker %i
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
Restart=always
RestartSec=2
WorkingDirectory=/var/www/wamadat-platform/backend
ExecStart=/usr/bin/php artisan queue:work \
  --queue=critical,payments,notifications,default \
  --tries=3 --timeout=120 --sleep=3 --max-time=3600
StandardOutput=journal
StandardError=journal
TimeoutStopSec=125

[Install]
WantedBy=multi-user.target
```

Enable two instances:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wamadat-worker@1.service
sudo systemctl enable --now wamadat-worker@2.service
```

Inspect:
```bash
sudo systemctl status wamadat-worker@1
sudo journalctl -u wamadat-worker@1 -f
```

Graceful redeploy: `php artisan queue:restart` works the same way under
systemd as supervisor.

## Monitoring (CP3-7 also covers this)

The `/health` endpoint exposes per-queue depth + scheduler heartbeat
freshness. Watch for:
- `queue.depths.critical > 0` for more than a minute → critical work
  isn't being consumed.
- `queue.depths.notifications > 50` sustained → worker can't keep up;
  scale `numprocs` up.
- `failed_jobs.count_24h > 5` → something's broken upstream; check
  `/admin/failed-jobs`.
- `scheduler.heartbeat_age_seconds > 90` → cron is dead; check
  `* * * * * php artisan schedule:run`.

## Failed-job recovery (CP3-6)

Standard sequence:
1. Open `/admin/failed-jobs` — inspect exception + payload.
2. Fix the root cause (config, schema, etc.).
3. Single retry: row's action button (calls `queue:retry {uuid}`).
4. Bulk retry: row checkboxes → "إعادة المحددة" (calls
   `queue:retry {uuids}`).
5. Worst case (mass clear): `php artisan queue:retry all` from the
   server. Idempotent — re-runs only the failed rows.

Proven in commit 57ea535: 3 forced failures → 1 failed_jobs row →
`queue:retry all` → worker pass → success. End-to-end works.

## Stuck jobs / hung workers

A "stuck" job is one whose `reserved_at` is in the past but the worker
that claimed it died without writing back. Laravel's database driver
re-releases such rows after `queue.retry_after` (90s in our config).
So a true hang clears in ≤ 90 seconds without intervention.

Manual unstick if needed:
```bash
# See currently-reserved jobs
psql wamadat_landlord -c \
  "SELECT id, queue, attempts, to_timestamp(reserved_at) FROM jobs
   WHERE reserved_at IS NOT NULL ORDER BY reserved_at;"

# Force re-release one row (use sparingly)
psql wamadat_landlord -c \
  "UPDATE jobs SET reserved_at=NULL, available_at=extract(epoch from now())::int
   WHERE id=<row_id>;"
```

Drain the entire queue (NOT typical — only during incidents):
```bash
php artisan queue:clear database --queue=notifications
```

## Killing a runaway worker

```bash
# Supervisor
sudo supervisorctl stop wamadat-worker:*

# systemd
sudo systemctl stop wamadat-worker@1 wamadat-worker@2

# Last resort (process)
pgrep -f "queue:work" | xargs -r kill
```

The current job will be re-released after `retry_after=90s`.

## Migrating to Redis (Stage 1)

Once Memurai/Redis is provisioned:

1. Verify Redis is reachable: `php artisan tinker --execute='Redis::ping();'`
2. Flip env:
   ```
   QUEUE_CONNECTION=redis
   REDIS_QUEUE_CONNECTION=default
   REDIS_QUEUE=default
   ```
3. Drain the database queue first (`queue:work` until `jobs` table is
   empty) to avoid orphaning rows.
4. Restart workers: `php artisan queue:restart`.
5. Failed-jobs storage stays on `database-uuids` regardless of driver
   (so the existing FailedJobResource keeps working).

Rollback: flip `QUEUE_CONNECTION` back to `database` and restart. No
data migration needed — the two drivers are fully independent.
