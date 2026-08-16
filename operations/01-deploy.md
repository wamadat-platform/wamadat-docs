# Deploy Checklist

**Topology:** Closed Beta runs on the operator's Windows 11 dev machine (see `00-deployment-topology.md`). "Deploy" = pulling latest `origin/main` into the same machine the dev edits land on.

**Status:** Manual deploy via `scripts/operations/deploy-now.bat`. CI runs on every push (see `.github/workflows/ci.yml`); branch protection enforces it (see step 0 below).

## Step 0 — One-time GitHub branch protection (operator, manual)

Branch protection is set via GitHub web UI; do it once.

1. Open https://github.com/asseerimishal-coder/wamadat-platform/settings/branches
2. Click **Add branch ruleset** (or "Add classic branch protection rule" if shown).
3. Branch name pattern: `main`
4. Enable:
   - ✅ Require a pull request before merging (optional — operator is sole maintainer, so direct push is acceptable for Closed Beta, BUT once a second contributor joins flip this on)
   - ✅ **Require status checks to pass before merging**
     - Add required checks: `Backend (Laravel + Pest)` and `Frontend (Next.js)`
   - ✅ Require branches to be up to date before merging
   - ✅ Do not allow bypassing the above settings (toggle if admin override should also be blocked)
5. Save.

After saving, the operator can still `git push origin main` directly (no PR required) BUT a push that breaks CI will surface red on the GitHub repo page — that's the gate for Closed Beta.

## Pre-flight (always, before any deploy)

| # | Check | Pass condition |
|---|---|---|
| 1 | CI green on `origin/main` | https://github.com/asseerimishal-coder/wamadat-platform/actions — latest workflow run is ✓ |
| 2 | No active customers mid-flow | (judgment call — visible from Filament admin) |
| 3 | Working tree clean | `git status` shows "nothing to commit" |
| 4 | Recent backup exists | `Get-ChildItem backups/` shows entry within last 24h |
| 5 | Worker queue drained | `php artisan queue:size` returns 0 (or low — pending jobs replay after) |

If any check fails: fix it OR explicitly accept the risk in the deploy log.

## Standard deploy

```cmd
scripts\operations\deploy-now.bat
```

This script runs the steps below in order; each step that mutates state is logged to `docs/operations/deploy-log.txt`. If any step fails, deploy aborts and the rollback runbook (`02-rollback.md`) is the next read.

1. **Verify working tree clean** — refuses to deploy if uncommitted changes exist (would clobber).
2. **Pre-deploy DB backup** — invokes the P0-2 script. Backup is the rollback anchor.
3. **`git pull --ff-only origin main`** — refuses non-FF (which would mean local commits aren't pushed).
4. **`composer install --no-dev --optimize-autoloader`** — production-mode deps.
5. **`php artisan migrate --force`** — DB schema changes. Forward-only; rollback uses the backup snapshot.
6. **Cache rebuilds** — `config:cache`, `route:cache`, `view:cache`.
7. **Frontend build** — `pnpm install --frozen-lockfile && pnpm build`.
8. **Restart queue worker** — `schtasks /End` then `/Run` for `Wamadat-Queue-Worker`.
9. **Smoke check** — runs `check-failed-jobs.ps1`; reports anomalies.

## Post-deploy verification (operator, 5 min)

1. `php artisan about` — confirms env, version, queue connection.
2. Open the storefront in browser → load a course page → confirm no 500.
3. Open Filament admin → confirm login works, payment webhooks list loads.
4. Check `docs/operations/summaries/summary-<today>.txt` (if past 08:00) for anomalies.
5. Tail `backend/storage/logs/laravel.log` for new ERROR/CRITICAL.

If any of these fail → follow `02-rollback.md`.

## When NOT to deploy

- During a payment burst (week-of-month checkout peak).
- Within 1 hour of a backup window (03:00 daily) — competing DB load.
- When the daily summary already shows an anomaly that has not been investigated.
- During the Beta cap window without explicit operator decision.

## Migrating from this checklist to a real CI/CD pipeline

When the VPS migration happens (see `00-deployment-topology.md` trigger), the steps above become a SHA-pinned GitHub Actions deploy job (one already scaffolded in `.github/workflows/deploy.yml` — replace the `TODO` step with SSH-driven `deploy.sh` calling the same sequence). The local `deploy.ps1` then becomes a dev-only convenience and the GitHub Actions job becomes authoritative.
