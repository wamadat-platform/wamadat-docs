# Deployment Topology — Closed Beta

**Status:** Active risk class. Created 2026-05-14. Reviewed against [[beta-on-dev-machine-risk]].

## Where everything runs

| Component | Location | Process |
|---|---|---|
| Laravel backend | `C:\Users\U\wamadat-platform\backend` | `php artisan serve` (dev) or production-grade later |
| Next.js frontend | `C:\Users\U\wamadat-platform\frontend` | `pnpm dev` or `pnpm build && pnpm start` |
| PostgreSQL 16 | `C:\Program Files\PostgreSQL\16` | Windows service `postgresql-x64-16` |
| Queue worker | (planned) NSSM-wrapped Windows service | `php artisan queue:work` |
| Storage / uploads | `C:\Users\U\wamadat-platform\backend\storage` | Local filesystem |
| Git remote | `https://github.com/asseerimishal-coder/wamadat-platform.git` | Private repo |
| Offsite backup | `C:\Users\U\OneDrive\Wamadat-Backups` | OneDrive sync to Microsoft cloud |

**Single host:** one Windows 11 machine. No replicas. No geographic redundancy.

## Risk class

This is **dev-grade hosting for a 50-user Beta**. Acceptable for the cap, NOT acceptable beyond it.

### What's protected against

- ✅ Code loss (GitHub private remote)
- ✅ DB loss from accidental delete / corruption (daily pg_dump + OneDrive offsite)
- ✅ Restore correctness (weekly restore drill)
- ✅ Queue job loss (DB-backed jobs table + supervision)

### What's NOT protected against

- ❌ Machine downtime (machine off = Beta down — no uptime guarantee)
- ❌ Disk failure during the day (between backups → up to 24h data loss)
- ❌ ISP outage (no fallback connectivity)
- ❌ Geographic disaster (OneDrive is Microsoft region, not multi-region)
- ❌ Power outage (no UPS)

These risks are accepted under the Beta cap. They become unacceptable at scale.

## Migration trigger

Move to a real VPS when ANY of these is true:
- Sustained >20 concurrent active users
- First paid customer
- First uptime SLA request from a Beta user
- Beta extends beyond 90 days

Target: Hetzner CX22 (€5/mo) or DigitalOcean Basic ($6/mo). Migration plan in `docs/operations/06-vps-migration.md` (written when trigger fires).

## Daily operator routine

1. **Morning**: check `docs/operations/backup-log.txt` — last entry should be from last night.
2. **Anytime**: check `failed_jobs` count: `php artisan queue:failed` — should be 0.
3. **Weekly (Sunday)**: run `scripts/operations/restore-drill.bat` — must report PASS.
4. **Weekly**: glance at OneDrive folder size — should grow, not shrink.

See [[beta-on-dev-machine-risk]] memory and `docs/operations/03-db-recovery.md` for procedures.
