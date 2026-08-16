# Monitoring Baseline — Closed Beta

**Status:** Installed 2026-05-14. Single daily snapshot covers logs, queue, payments, storage, and basic health. Uptime checks deferred until external tunnel is provisioned.

## What's monitored and how

| Signal | Source | Frequency | Alert channel |
|---|---|---|---|
| Failed jobs > 0 | `failed_jobs` table | every 15 min | toast + log + offsite marker (see `04-worker-supervision.md`) |
| Daily health snapshot | logs/queue/payments/storage | daily 08:00 | `docs/operations/summaries/summary-YYYYMMDD.txt` + OneDrive |
| Backup PASS/FAIL | `backup-databases.ps1` | daily 03:00 | `docs/operations/backup-log.txt` |
| Restore drill PASS/FAIL | operator-triggered | weekly | `docs/operations/restore-drill-log.txt` |

The daily summary is the operator's morning routine — one glance answers "did anything go wrong overnight?".

## Daily summary contents

| Section | What it shows | Thresholds for concern |
|---|---|---|
| Logs | laravel.log size, ERROR/CRITICAL count last 24h, top 3 unique signatures | >100 errors/24h OR same error repeated >20x → investigate |
| Queue | pending jobs, total failed, failed last 24h, worker cycles last 24h | failed > 0 → see `04-worker-supervision.md`; cycles = 0 → worker not running |
| Payments | tenants scanned, captures + failed + SAR total last 24h, last webhook timestamp + freshness | webhook STALE while captures > 0 → gateway broken; failed/captures > 10% → payment pipeline issue |
| Storage | DB sizes (landlord/tenants), backend storage, disk C: free | disk < 20 GB → urgent cleanup |
| Health | PostgreSQL UP/DOWN, queue worker process, latest offsite backup age | any DOWN → immediate; backup age > 30h → check Task Scheduler |

## Operator routine

**Every morning (2 min):**
```powershell
notepad "C:\Users\U\wamadat-platform\docs\operations\summaries\summary-$(Get-Date -Format yyyyMMdd).txt"
```
Or just check OneDrive folder `Wamadat-Backups\SUMMARIES\` from any device.

Scan for:
- `Errors 24h:` line → high count?
- `Failed 24h:` line → any payment failures?
- `webhook STALE` warning?
- `PostgreSQL: DOWN` or `Latest offsite bk: NONE FOUND`?

**Triggering the summary on demand:**
- Double-click `scripts\operations\summary-now.bat`
- Or: `Start-ScheduledTask -TaskName Wamadat-Daily-Summary`

## What is NOT monitored yet (known gaps)

These are intentional deferrals — close them before / during Closed Beta growth, not before launch.

### Uptime (external)
- **Gap:** the app is on `localhost`. External monitors (UptimeRobot / Better Uptime) cannot reach it.
- **Trigger to close:** Cloudflare Tunnel provisioning during Closed Beta launch.
- **Plan:** install `cloudflared`, expose backend on a public hostname, register the hostname with UptimeRobot free tier (50 checks, 5 min interval). Failure email goes to operator.

### Real-time error aggregation
- **Gap:** Sentry SDK is wired in code + CI (`deploy.yml` uploads sourcemaps), but no production deploy has triggered a Sentry release yet — events go to the SDK but there's no dashboard being watched.
- **Trigger to close:** first deploy after the tunnel is up. Then sign in to Sentry, configure alert rules.

### APM / latency tracking
- **Gap:** no p50/p95/p99 endpoint latency captured.
- **Trigger to close:** real users hit the app. Add Sentry Performance OR Laravel Telescope for local dev visibility.

### Payment gateway connectivity probe
- **Gap:** if Tap/Tamara API goes down, the daily summary won't notice until payments start failing.
- **Plan:** add a daily probe that calls each gateway's status endpoint (Tap: `https://api.tap.company/v2/ping`, Tamara: TBD) — add when first real payment is processed.

### Memory / CPU baseline
- **Gap:** no historical baseline for "normal" resource usage.
- **Trigger to close:** Phase E (real-traffic measurement) — deferred until Beta has real users, per [[next-session-operational-hardening]].

## Future migration to VPS

When the dev-machine topology is replaced (see `00-deployment-topology.md` migration trigger):
- Replace Task Scheduler `Wamadat-Daily-Summary` with cron @ 08:00.
- Pipe the summary to Slack/Telegram instead of (or in addition to) OneDrive.
- Add Prometheus + Grafana once Beta scale demands time-series visibility.
