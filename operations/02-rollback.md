# Rollback Runbook

**When to read this:** a deploy via `01-deploy.md` failed mid-flow, OR a deploy succeeded but broke production behavior discovered after the fact. This document is the single source of truth for restoring known-good state.

**Hard rule:** if the failure was during/after `php artisan migrate`, the rollback path **MUST** use the pre-deploy backup snapshot. Forward migrations are not symmetric with `migrate:rollback` in this codebase — never assume `migrate:rollback` will work cleanly.

## Decision tree — pick exactly one path

```
Did the deploy reach step 5 (php artisan migrate)?
├── No  → Path A: Code-only rollback (fast, no data risk)
├── Yes → Was the migration successful?
│         ├── No  → Path B: Mid-migration recovery (DB is in unknown state — use backup)
│         └── Yes → Are there user writes since the migration ran?
│                   ├── No  → Path B: Restore from pre-deploy backup
│                   └── Yes → Path C: Forward-fix only (rollback would lose user data)
```

## Path A — Code-only rollback (no DB changes happened)

Migration step never ran. Just rewind code.

```powershell
cd C:\Users\U\wamadat-platform
$lastGood = git log --oneline -n 5      # pick the SHA you were running before deploy
git reset --hard <lastGood-SHA>
git push origin main --force-with-lease  # operator confirms; rewinds remote
schtasks /End  /TN 'Wamadat-Queue-Worker'
Start-Sleep -Seconds 2
schtasks /Run  /TN 'Wamadat-Queue-Worker'
```

Verify with the post-deploy verification checklist in `01-deploy.md`.

## Path B — Restore from pre-deploy backup (DB was touched)

The pre-deploy backup taken by `deploy.ps1` step 2 is the anchor. Find it:

```powershell
Get-ChildItem C:\Users\U\wamadat-platform\backups -Filter 'wamadat-backup-*.zip' |
    Sort-Object LastWriteTime -Descending | Select-Object -First 5
```

Pick the entry whose timestamp is closest to the deploy START in `docs/operations/deploy-log.txt` (and EARLIER than it). Then run the **Full restore procedure** in `03-db-recovery.md` — it handles forensic rename, fresh DB, restore, sanity check.

Then code-rewind:

```powershell
cd C:\Users\U\wamadat-platform
git reset --hard <lastGood-SHA>
git push origin main --force-with-lease
schtasks /End  /TN 'Wamadat-Queue-Worker'
Start-Sleep -Seconds 2
schtasks /Run  /TN 'Wamadat-Queue-Worker'
```

⚠ **User writes between the migration and your rollback are LOST.** Document the time window in the incident log and reach out to any affected user.

## Path C — Forward-fix only (cannot afford to lose user data)

The migration completed, real users wrote real data on top of it, and rolling back to backup would discard those writes. Rollback is no longer an option — you must fix forward.

Steps:
1. Put the app in maintenance mode: `php artisan down --message="Maintenance"`.
2. Identify the specific broken behavior; write a targeted fix in a new commit on `main`.
3. Run the **full** `deploy-now.bat` again with the fix.
4. `php artisan up`.

If the bug is data-shaped (corrupt rows from the bad migration) rather than code-shaped, you may need a one-off remediation migration. Capture every SQL statement in `docs/incidents/<date>-<short-name>.md` before running it.

## Common gotcha: queue worker holding stale code

Even after `git reset`, the queue worker process loaded into memory still has the OLD code until it cycles. The bounce commands above (`schtasks /End` + `/Run`) handle this — DON'T skip them.

## After any rollback

1. **New baseline backup** — run `backup-now.bat` once the system is verified-stable so future rollbacks have a fresh anchor.
2. **Write incident note** — `docs/incidents/<YYYY-MM-DD>-<short-name>.md`:
   - What broke
   - Which path you took (A / B / C)
   - Affected user window (for Path B)
   - What change goes into `01-deploy.md` to prevent recurrence
3. **Update memory if structural** — if the rollback exposed a class of risk not captured in the existing memory files, write or update a memory.

## Recovery from machine loss

If the machine is unavailable (theft, dead disk, etc.), this is a recovery operation, not a rollback. Follow `03-db-recovery.md` § "Restoring from offsite" — it covers the bootstrap path on a replacement Windows host.
