# Queue Worker Supervision + Failed-Jobs Alerting

**Status:** Active. Installed 2026-05-14 via `scripts/operations/install-supervision.bat`. Drill-validated end-to-end.

## What runs and where

| Task Scheduler task | Trigger | Script | Purpose |
|---|---|---|---|
| `Wamadat-Queue-Worker`      | At logon (current user) | `queue-worker.ps1`      | Loop: spawn `php artisan queue:work`, restart on exit, log each cycle |
| `Wamadat-Failed-Jobs-Check` | Every 15 min            | `check-failed-jobs.ps1` | Alert (toast + log + offsite marker) when `failed_jobs > 0` |
| `Wamadat-Daily-Backup`      | Daily 03:00             | `backup-databases.ps1`  | (See `00-deployment-topology.md` / P0-2 — listed for completeness) |

All tasks run as `$env:USERDOMAIN\$env:USERNAME` at standard privilege. None require admin.

## Queue worker supervisor — design notes

- Outer PowerShell loop calls `php artisan queue:work --tries=3 --timeout=120 --max-jobs=1000 --max-time=3600 --sleep=3`.
- `--max-time=3600` cycles the worker hourly so code/config changes get picked up without manual restart.
- `--max-jobs=1000` caps memory growth per worker process.
- If the worker exits in under 30 seconds, the loop waits 30 seconds before restarting (crash-loop protection). Otherwise it restarts after 2 seconds.
- Each cycle's start/exit/duration is appended to `docs/operations/worker-log.txt` (gitignored, operator-local).
- Log file self-rotates to `.1` when it exceeds 5 MB. Only two generations are kept — older runs are overwritten.

## Failed-jobs alert — design notes

Three independent channels so a missed channel doesn't equal a silent failure:

1. **Windows toast notification** — visible to the logged-in operator. Best-effort (no failure if WinRT can't bind).
2. **Local log** — `docs/operations/alerts-log.txt` (gitignored). Every run appends one line: `OK` (count=0), `ALERT` (count>0), or `ERROR` (script failed).
3. **Offsite marker** — `C:\Users\U\OneDrive\Wamadat-Backups\ALERTS\alert-YYYYMMDD.txt`. Created only when `count>0`. OneDrive sync makes this visible from another device.

When you see an alert:
```powershell
cd C:\Users\U\wamadat-platform\backend
php artisan queue:failed              # show failures
php artisan queue:retry all           # retry everything
php artisan queue:flush               # drop them all (only if expected)
```

## Operator checklist

**Daily:**
- Glance at `docs/operations/worker-log.txt` tail — should show recent CYCLE entries from today.
- Confirm no `alert-*.txt` files in `OneDrive\Wamadat-Backups\ALERTS\` dated today.

**Weekly:**
- Run `scripts/operations/restore-drill.bat` — must report PASS.
- `php artisan queue:failed` — should be empty.

**Monthly:**
- Re-run `scripts/operations/install-supervision.bat` (idempotent) — refreshes any task definitions that may have drifted.

## Validation performed at install time (2026-05-14)

- Worker supervisor: started, logged `SUPERVISOR_START` + `CYCLE_1`, terminated cleanly on `schtasks /End`.
- Failed-jobs check, OK path: failed_jobs=0 → logged `OK`, exit 0.
- Failed-jobs check, ALERT path: injected drill row → logged `ALERT failed_jobs=1`, OneDrive marker created.
- Failed-jobs check, recovery: drill row deleted → next run logged `OK`, no new marker.

## Uninstall

```powershell
Unregister-ScheduledTask -TaskName Wamadat-Queue-Worker, Wamadat-Failed-Jobs-Check, Wamadat-Daily-Backup -Confirm:$false
```

## Migration to VPS (future)

When this Beta outgrows the dev machine (see `00-deployment-topology.md` — migration trigger):
- Replace `queue-worker.ps1` with a systemd unit (`/etc/systemd/system/wamadat-queue.service`).
- Replace `check-failed-jobs.ps1` with a cron job calling Healthchecks.io or Slack webhook.
- Replace Task Scheduler `Wamadat-Daily-Backup` with cron + offsite via `rclone` or AWS CLI.

Reference systemd unit template will be added to `docs/operations/06-vps-migration.md` when the migration is scheduled.
