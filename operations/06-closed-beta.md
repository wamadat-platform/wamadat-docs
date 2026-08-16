# Closed Beta Operations Readiness

**Status:** Beta NOT yet launched as of 2026-06-08. This document is the launch gate.

**Update 2026-06-08** — Original 2026-05-14 program-cap blocker is **CLOSED**. The
unpublishing of 25 surplus seed programs was performed between 2026-05-14 and
2026-06-08. Today's daily summary (`summary-20260608.txt`) reports
`Active programs: 5 / 5 (!! at cap)` and `Beta verified users: 1 / 50 (ok)`.
Item 9 in the pre-launch checklist below now passes.

Remaining pre-launch tasks live in `docs/executive-transformation-backlog.md`
Track 11 (Closed Beta Launch): Sentry DSN, daily-summary rotated-log fix,
3 Playwright smoke specs, then operator action (curating first invitees).

## Beta scope (locked)

| Constraint | Value | Why |
|---|---|---|
| **Max verified users** | 50 | Dev-machine topology has zero redundancy; 50 is the cap where 24h support response is realistic for the solo operator. |
| **Max published programs** | 5 | Limits content-error blast radius. Quality > volume during Beta. |
| **Cohort acquisition** | Direct invitation only | No public marketing. No paid ads. Each invite is hand-curated. |
| **Scale assumptions** | None | The system has NOT been load-tested (Phase E deferred). Anything beyond the caps is unsupported behavior. |
| **Uptime SLA** | None | Single-machine topology; no commitments. |
| **Support window** | Operator-defined | Documented to invitees before they accept. |

## Pre-launch checklist (run through ALL before sending the first invite)

| # | Item | Verify with | Pass condition |
|---|---|---|---|
| 1 | Git remote in place | `git remote -v` | `origin = github.com/asseerimishal-coder/wamadat-platform` |
| 2 | Daily backup task active | `Get-ScheduledTask Wamadat-Daily-Backup` | State = `Ready`, NextRunTime tomorrow 03:00 |
| 3 | Latest restore drill PASS | `Get-Content docs/operations/restore-drill-log.txt -Tail 1` | Last entry = `PASS` within 7 days |
| 4 | Queue worker active | `Get-ScheduledTask Wamadat-Queue-Worker` + tail `worker-log.txt` | Recent CYCLE entries |
| 5 | Failed-jobs check active | `Get-ScheduledTask Wamadat-Failed-Jobs-Check` | State = `Ready` |
| 6 | Daily summary task active | `Get-ScheduledTask Wamadat-Daily-Summary` | State = `Ready` |
| 7 | Today's summary clean | `summaries/summary-<today>.txt` | No `*** OVER CAP ***`, no `STALE`, no `DOWN` |
| 8 | Branch protection on `main` | https://github.com/asseerimishal-coder/wamadat-platform/settings/branches | Required checks listed |
| 9 | Published programs ≤ 5 | Daily summary `[Beta Caps]` line | `Active programs: N / 5  (ok)` |
| 10 | Seed/test users isolated | Daily summary `Beta verified users` line | `0 / 50  (ok)` at launch moment |
| 11 | Sentry release wired | `php artisan about` shows `SENTRY_RELEASE` | (deferred — wire on first VPS deploy) |

Items 9, 10, 11 were the ones most likely to fail at the time this document
was first written (2026-05-14). As of 2026-06-08 only item 11 remains
likely-to-fail (Sentry DSN env var still empty — tracked as audit finding
C-004 / backlog item 4.1).

~~Specifically: **30 programs are currently `status='published'` — 25 must be
unpublished before launch.**~~ **[Closed 2026-06-08 — daily summary confirms 5/5.]**

## Unpublishing surplus programs (before launch)

```sql
-- Identify the 5 to KEEP (operator decides which)
SELECT id, slug, title_ar, status, created_at
FROM tenant_wamadat.programs
WHERE status = 'published'
ORDER BY created_at DESC;

-- Unpublish everything except the chosen 5
UPDATE tenant_wamadat.programs
SET status = 'draft', updated_at = NOW()
WHERE status = 'published'
  AND id NOT IN (
    '<keep-id-1>',
    '<keep-id-2>',
    '<keep-id-3>',
    '<keep-id-4>',
    '<keep-id-5>'
  );
```

Re-run the daily summary after — the `[Beta Caps]` line must read `Active programs: 5 / 5  (!! at cap)` or less.

## Operational gates (during Beta)

### Cap enforcement is **observability-only** — no code blocks

The 50/5 caps are NOT enforced by code (no middleware, no DB constraint). They are surfaced by the daily summary. When the operator sees:

- `Beta verified users: 48 / 50  (!! near cap)` → stop sending invitations until existing users churn out.
- `Active programs: 5 / 5  (!! at cap)` → unpublish one before publishing another.

This is intentional. Beta is small enough that policy-level enforcement is sufficient. Code-level enforcement is added if a SECOND tenant goes live or Beta extends past 90 days.

### Manual kill switch (if registration must be paused)

There is no built-in flag yet. If registration must stop NOW:

```bash
# Quick measure: take the app down briefly
cd C:/Users/U/wamadat-platform/backend
php artisan down --message="Closed Beta is full"
```

A proper feature flag (`BETA_REGISTRATION_OPEN=false`) is a future change — track in BACKLOG.md.

### Communication discipline

- ❌ No public posts about Wamadat being "live"
- ❌ No SEO indexing (verify `robots.txt` disallows public crawlers if applicable)
- ❌ No paid acquisition
- ✅ Direct invitation only. Each invite includes the Beta caveats (single-host, daily backups, no SLA).
- ✅ Daily operator presence (the morning summary IS the presence — see `05-monitoring.md`).

## Beta exit criteria (decide before launch)

The exit from Beta to "Open Cohort" happens when ALL of these are true:

1. 30 days of zero P0/P1 incidents.
2. Daily summary has trended green every day for 14 consecutive days.
3. Restore drill has passed 4 consecutive weekly runs.
4. Phase E real-traffic baseline has been measured (see [[next-session-operational-hardening]] P1-4).
5. Operator has decided whether to migrate to a VPS or scale dev-host further.

If ANY of these is unmet, Beta extends.

## Escalation triggers (when Beta itself is in trouble)

| Signal | Response |
|---|---|
| Daily summary shows `PostgreSQL: DOWN` | Stop accepting writes. Run `03-db-recovery.md` if data appears corrupt. |
| `Failed 24h: > 10` payments | Pause checkout (artisan down with payment-specific message). Investigate Tap/Tamara API status. |
| Errors 24h > 200 | Same as above, but for general traffic. |
| `webhook STALE while captures > 0` | Gateway integration broken. Stop accepting new orders. |
| `Latest offsite bk: NONE FOUND` or `> 36h` | Investigate Task Scheduler `Wamadat-Daily-Backup`. Run `backup-now.bat` manually. |
| Restore drill FAIL (weekly) | Highest priority — backups may not be recoverable. Halt new feature work until resolved. |

## Going forward

After 14 days of green operation, the next memo to write is `07-beta-to-open.md` covering the migration plan to a real VPS (per `00-deployment-topology.md` migration trigger).

Linked memory: [[beta-on-dev-machine-risk]], [[closed-beta-plan]], [[next-session-operational-hardening]].
