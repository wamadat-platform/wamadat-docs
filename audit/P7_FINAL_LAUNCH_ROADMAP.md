# 🗺️ P7 — Final Launch Roadmap

**Date:** 2026-05-13
**Owner:** Wamadat platform team
**Status:** Code is launch-ready. This roadmap is operational, not engineering.

---

## Stage 0 — Pre-launch (days T-7 to T-0)

**Goal:** Flip from "code complete" to "running in production with operators watching."

### Hard prerequisites (cannot launch without)
- [ ] Production Postgres provisioned (managed, with automated daily backups + PITR)
- [ ] Production Redis provisioned (separate DB indices per role per `config/database.php`)
- [ ] Production domain + wildcard SSL (`*.wamadat.academy`) issued and on the LB
- [ ] DNS pointing the apex + the first tenant subdomain
- [ ] Resend (or equivalent) sender domain verified (SPF + DKIM)
- [ ] Tap merchant account approved + production keys in `.env`
- [ ] Sentry DSN in production `.env` + `SENTRY_RELEASE` stamped by CI on deploy
- [ ] Production `.env` reviewed — `APP_ENV=production`, `APP_DEBUG=false`, `LOG_LEVEL=info`
- [ ] `php artisan migrate --database=landlord --force` succeeds against prod
- [ ] First tenant provisioned via `php artisan tenant:provision` (NOT via raw `migrate --database=tenant`)
- [ ] Cron `* * * * * php artisan schedule:run` wired on the prod box
- [ ] At least one queue worker running (`php artisan queue:work --tries=3 --max-time=3600`)

### Verification rituals (run once, document outcome)
- [ ] `php artisan sentry:test --tenant=<slug>` → event visible in Sentry within 30s with the right tags
- [ ] `curl https://wamadat.academy/api/v1/health` → 200 + all 8 probes green
- [ ] Full pre-launch smoke checklist from P5 §6 — the 10-step user journey
- [ ] Force a job failure (kill Redis briefly) → confirm it lands in `/admin/failed-jobs` with the right tenant tag → retry from the UI → confirm audit row in `audit_logs`
- [ ] `php artisan outbox:drain` runs cleanly + email lands in a real inbox

### Soft cutoff
- [ ] Internal-only Filament login (super) reviewed — strong password + 2FA enabled
- [ ] Backup restore tested at least once on a staging copy (don't trust untested backups)
- [ ] Runbook printed / pinned: "DB down", "Redis down", "Email delivery stalled", "Webhook signature mismatch" — 1-page each

---

## Stage 1 — Soft Launch (T+0 → T+7 days)

**Goal:** First 25-50 real students. Watch for stress on every layer.

### Posture
- Closed list / invite-only — no paid acquisition yet
- 1-3 published programs (real content, not seeds)
- Operator on call 8-hour window each day, eyes on `/admin` operations panel
- Daily 15-minute standup: review Sentry events, failed jobs, outbox failures, webhook failures

### What to watch
| Signal | Source | Reaction threshold |
|---|---|---|
| Failed jobs > 0 | `/admin/failed-jobs` | Investigate same-day. Retry from UI if root cause was transient. |
| Sentry new issue | Sentry inbox | Triage within 2 hours during operator window |
| Outbox `failed`/`bounced` > 5/day | `/admin/email-outboxes` | Verify Resend deliverability + recipient quality |
| Webhook failure rate > 5% | `/admin/payment-webhooks` filtered `failed_only` | Check gateway dashboard, replay from UI |
| Queue depth > 200 | `QueueDepthWidget` | Spin up a second worker |
| Health endpoint flips to `503` | `/api/v1/health` | Page someone |

### What to NOT do
- Don't add features
- Don't change the brand palette / typography mid-flight
- Don't ship the deferred PHASE 22 stuff (avatar upload, dark mode) — keep the surface stable

### Exit criteria for moving to Stage 2
- 5 consecutive days with `failed_jobs.count = 0` at midnight
- 5 consecutive days with `health = healthy` 100% of the minute-by-minute polls
- ≥10 students completed a program (got a certificate)
- No P0/P1 Sentry issue in the last 72 hours
- At least one paid order processed end-to-end via the real gateway

---

## Stage 2 — First 7 Days Post-Soft-Launch (T+7 → T+14)

**Goal:** Stabilize, harden, fix the small things that the first cohort surfaces.

### Recurring jobs
- Daily Sentry review (group by tag = tenant)
- Daily outbox failed-or-bounced audit (delivery rate ≥ 95%)
- Run `composer audit` + `pnpm audit` weekly; patch high-severity CVEs

### Code priorities (in order)
1. **B-2 sourcemap upload** to Sentry — operator must pick from the deploy CI step (~1h work)
2. **AssignmentController S3-origin allowlist** (BACKLOG from P4.6 — tighten beyond "must be a URL")
3. **Module-test harness fix** (P4.2.A) — currently 17 red tests not in main suite
4. **Playwright E2E smoke** for the 5 critical journeys (signup, enrollment, learn, quiz, certificate)
5. Any small UX nicks reported by the cohort

### Don't touch
- Schema changes
- Migration order
- Existing tests (only add)

---

## Stage 3 — First 30 Days (T+14 → T+30)

**Goal:** Move from "operating on watchfulness" to "operating on dashboards." Start adding the second tier of operational tools.

### Adds (in order of value)
1. **Daily backup verification job** — restore from yesterday's backup to a staging DB, assert row counts; alert on miss
2. **Source-map + release notes** to Sentry on every deploy (CI step)
3. **Camera-based QR attendance scan** (P12 — was stubbed in P5)
4. **Avatar upload** (PHASE 22 — sensible default + manageable scope)
5. **Search latency probe** — start tracking p95 for `/search` to know when to flip to Meilisearch
6. **Status page** (statuspage.io or self-hosted) reading from the `/api/v1/health` endpoint

### Operational additions
- Weekly metrics email (DAU, enrollments, certificates issued, failed jobs, gateway failure rate, email delivery rate)
- Quarterly access review (who in the `/admin` panel has `admin` role)
- Quarterly secrets rotation reminder (Sanctum signing key, Resend, Tap webhook signing secret)

---

## Stage 4 — Stabilization (Month 2-3)

**Goal:** Pay down the deferred items, expand surface area, prepare for paid acquisition.

### Engineering
- **Real video provider cutover** (Bunny / Mux) for premium content
- **AI streaming response** (SSE) — improves perceived latency
- **Dark mode** (PHASE 22 polish — completes the design-system story)
- **Module-test harness completion** (P4.2.A)
- **Multi-tenant onboarding flow** (super admin self-service) — currently `php artisan tenant:provision` is the only path

### Product
- Run a controlled paid acquisition campaign (≤ 100 USD/day)
- Add a second category of content (cross-vertical learning)
- Open Wishlist / Affiliate UI back up (backend was kept alive intentionally)

### Process
- First fire drill: simulate Redis outage during business hours; measure MTTR
- First chaos exercise: revoke a tenant's owner user; confirm audit + lockout flow

---

## Stage 5 — Growth (Month 4+)

**Goal:** Scale curves get steep. The platform's design (per `docs/02-architecture.md`) anticipates this — schema-per-tenant Postgres, Redis queue, Filament admin, Sentry visibility.

### Engineering tracks
- **Search** → Meilisearch when catalog passes ~5k programs
- **Charts** on admin dashboard — when we have weeks of historical data worth visualizing
- **Horizon** — only if/when Redis queue ops complexity outgrows the Filament resource (a real signal, not a checkbox)
- **Sentry tracing** — flip `SENTRY_TRACES_SAMPLE_RATE` from 0.0 to 0.1 once we have baseline latency targets
- **Pulse** — when we want per-route ms breakdowns
- **Multi-region** — explicitly NOT now. Reactivate when KSA latency outliers + UAE/Egypt registrations demand it (per master roadmap Phase 3 — Month 9-12)

### Business tracks
- Open marketplace for 3rd-party instructors
- Partner with universities for accredited cohorts
- Mobile-native shell (PWA + Capacitor) once retention data justifies it

---

## 📌 What to NOT do (durable advice)

- **Don't add Horizon "because Laravel has it"** — operator pain is the trigger, not platform completeness
- **Don't ship a chart dashboard before chart data is meaningful** — the Stat strip is enough at 50 students
- **Don't migrate to microservices** — schema-per-tenant + a single Laravel monolith handles 100k MAU comfortably
- **Don't bypass the audit log** — every operator action should be reproducible from `audit_logs` alone
- **Don't relax `send_default_pii: false`** in Sentry, ever
- **Don't let the deferred items rot** — pick one per week from §3 to retire

---

## 📅 At-a-glance timeline

```
T-7d                 T-0         T+7d            T+30d          T+90d
│                    │           │               │              │
│ Stage 0  ───────►  │ Soft launch (Stage 1)     │              │
│ Provision +        │ Invite-only, watching     │              │
│ verify             │                           │ Stabilize +  │
│                    │                           │ harden       │
│                                                │ (Stage 3)    │
│                                                                │ Growth
│                                                                │ (Stage 5)
```

Code is ready. Operate it.
