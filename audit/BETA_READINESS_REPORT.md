# Beta Readiness Report — Closed Beta Verdict

**Authority:** consolidated go/no-go for Closed Beta Controlled Exposure (post-Phase-D).
**Captured:** 2026-05-14.
**Baseline commit:** `dc0acc4` (CP4-12 closeout — Phase D Part 2 complete).

---

## 1. Verdict

**GO for Closed Beta Controlled Exposure.**

Scope of "controlled exposure":
- Limited users (operator-issued invites).
- Limited published programs.
- Direct monitoring of logs / queues / financial / support channels.
- No public campaigns. No public app store launches. No open onboarding.

**NOT GO for public soft launch.** Public launch waits until Phase E (Performance + Scale Reality) + Phase F (Disaster Recovery + Operations).

---

## 2. What's locked + ready

### Backend platform
- Laravel 11, schema-per-tenant Postgres, Filament 3 admin (tenant + super).
- `/api/v1/*` — 90 documented operations, frozen contract per `PHASE_D_FINAL_API_AUDIT.md`.
- Canonical error envelope with closed-vocabulary 22 codes (CP4-9).
- Two-token auth model with refresh rotation + reuse detection (CP4-5).
- 24 rate-limited endpoints with mobile-safe `RATE_LIMITED` envelope (CP4-10).
- Signed, scoped, time-limited media URLs for lesson resources (CP4-11).
- Audit log append-only triggers (UPDATE/DELETE + TRUNCATE blocks).
- ZATCA Phase 1 invoice + credit note with TLV QR (B2 + B-H5).
- Per-IP + per-(email|IP) login rate limiting (B7).
- 2FA mandatory on `/admin` (B4 + B-H6).
- Refund approval threshold (>5000 SAR requires second approver, B1).
- Coupon-redemption race protection (B3 + B-H2 + CP2-5 UNIQUE constraint).
- Refund idempotency at gateway level (B-H1).
- Outbox + notification idempotency keys (CP2-1, CP2-2).
- Database queue with worker-managed retry + failed-job recovery (CP3-1…CP3-7).

### Documentation
- `docs/api/01-inventory.md` through `09-signed-media-urls.md` (9 files).
- `docs/api/CHANGELOG-v1.md` with every breaking + non-breaking change.
- `backend/public/docs/{openapi.yaml, collection.json, index.html}` machine-readable.
- `docs/audit/PHASE_D_FINAL_API_AUDIT.md` — consolidated Phase D outcome.

### Test surface
- 25 contract tests, 140 assertions, all green (CP4-11 + CP4-12).
- Live HTTP smoke proofs documented in each CP commit message.

---

## 3. Known limitations (operate around these)

| Limitation | Impact | Beta workaround |
|---|---|---|
| **Queue driver = `database`, no Redis/Horizon** | Lower throughput ceiling; failed-job retries delayed | Cron-managed `queue:work` (CP3-3). Monitor `failed_jobs` table; manual retry via `queue:retry`. Acceptable for <50 concurrent users. |
| **Filesystems = `local` (no S3/CDN)** | Single-server file storage; no edge caching | Lesson media volume kept under 5 GB during Beta. Storage symlink + signed URLs (CP4-11) protect access. |
| **53 pre-canonical Pest tests failing (R-OPEN-32)** | Legacy assertions don't match current v1 surface | Tests don't gate deployment. The 25-test CP4-11/12 suite is the going-forward safety net. Cleanup runs in parallel. |
| **Scribe OpenAPI has empty per-endpoint `responses` (R-OPEN-30)** | SDK codegen produces untyped response models | Canonical envelope is uniform, documented at meta level. Mobile SDK can wrap with a generic envelope type. |
| **Per-field FormRequest validation messages bypass Accept-Language (R-OPEN-28)** | `error.details.email[0]` stays Arabic when client requests EN | Mobile/SDK clients render `error.message` (which IS localized) as the headline; `details` is secondary. |
| **No load testing yet (Phase E)** | Unknown ceilings: DB connections, queue depth, memory under sustained traffic | Beta cohort capped at operator's tolerance for live tail; Phase E lifts the cap. |
| **No backup/restore drill yet (Phase F)** | Recovery procedures untested under fire | Daily Neon snapshot + weekly logical dump. Drill runs in Phase F. |
| **`/admin` is the operator console; no read-only user UI** | All operator actions go through Filament | Operator trained on Filament; this is acceptable for ~3-5 internal users. |

---

## 4. Launch blockers — none active

All P0 blockers from previous phases are closed:
- Phase A — 7 admin 500s fixed with browser proof.
- Phase B — 7 security + financial hardening items + 6 hardening follow-ups closed.
- Phase C — queue + notification + outbox reliability + 10 ops items closed.
- Phase D Part 1 — API surface inventory + canonical shape + versioning closed.
- Phase D Part 2 — 12 mobile-readiness items closed.

R-OPEN entries currently P2 or below — none block Closed Beta.

---

## 5. Monitoring checklist (Beta-first 30 days)

Watch daily; investigate same-day if anything breaches.

### Application health
- [ ] `/api/v1/health` returns 200 from monitoring (NOT 503).
- [ ] Sentry error rate stays under 1 issue per 10 active users per day.
- [ ] No 5xx responses on `/api/v1/*` for 24h windows (treat as P0).
- [ ] No new R-OPEN entries promoted to P0.

### Database
- [ ] Neon connection pool usage < 60% peak.
- [ ] No locking incidents in `pg_stat_activity` longer than 5s.
- [ ] `audit_logs` table growth tracked; row count graphed weekly.
- [ ] Daily Neon snapshot completed (Neon dashboard).

### Queues
- [ ] `failed_jobs` table count: target 0, alert at > 5 per 24h.
- [ ] `jobs` table: no queue older than 60s.
- [ ] `outbox_emails`: zero rows stuck in `pending` > 5 min.
- [ ] `queue:work` cron alive (check `php artisan queue:health`).

### Authentication
- [ ] `REFRESH_TOKEN_REUSED` emit rate: alert at > 3 per 24h (security signal).
- [ ] Login throttle 429 rate: trend, not threshold (anomaly detection).
- [ ] `2fa_enabled` count for admin users: must be 100%.

### Financial
- [ ] `orders` vs `invoices` row counts match daily.
- [ ] Refunds approval workflow: every >5000 SAR refund has two approvers logged.
- [ ] ZATCA invoice serial sequence has no gaps (cron task verifies daily).
- [ ] `refunds.refund_idempotency_key` UNIQUE — alert on UNIQUE violation events.

### Behavior / business
- [ ] Active programs count vs. published count (no leftover drafts).
- [ ] Daily active users (DAU) trend in Filament dashboard.
- [ ] Support ticket volume + median time-to-first-response.
- [ ] Email delivery rate (Resend dashboard); bounces under 2%.

### Mobile API consumers
- [ ] Catalog of clients calling `/api/v1/*` (User-Agent + IP).
- [ ] `error.code: VALIDATION_FAILED` frequency by endpoint (UX signal).
- [ ] `error.code: SERVER_ERROR` rate (any non-zero needs investigation).

---

## 6. Rollback plan

### Code rollback (deploy-time issue)
```bash
# Revert to last known-good commit (Phase D baseline)
git revert --no-commit dc0acc4..HEAD
git commit -m "rollback to Phase D baseline"
git push origin main
# Deploy pipeline picks up the revert.
```

### Migration rollback (schema regression)
- Each tenant migration ships with `down()`. Run `php artisan migrate:rollback --database=tenant --step=N` against the specific tenant.
- Landlord migrations: same with `--database=landlord`.
- DO NOT rollback `audit_logs` triggers without an export — trigger drops would orphan the append-only chain.

### Data rollback (financial or sensitive write)
- Neon point-in-time recovery (24h window on Free tier, 7 days on Launch tier).
- Audit log forensics first — confirm scope before restore.
- Restore to a NEW schema, validate, then `pg_dump | psql` to swap.
- Communicate to affected users within 24h (PDPL requirement).

### Feature rollback (one CP misbehaves)
- Each CP is one commit. Cherry-pick revert is the safe path.
- DO NOT revert CP4-2 (canonical envelope) without coordinating with frontend — every client reads `error.code` now.
- DO NOT revert CP4-5 (two-token auth) without revoking all existing refresh tokens first.

---

## 7. Backup / recovery status

| Backup | Driver | Cadence | Retention | Drill status |
|---|---|---|---|---|
| Landlord DB | Neon snapshot | Daily | 7 days (Launch tier) | Not yet drilled — Phase F. |
| Tenant DBs (per schema) | Neon snapshot | Same as landlord | Same | Not yet drilled — Phase F. |
| Object storage (lesson files) | `disk('public')` on app server | None (single point of failure) | N/A | Acceptable for Beta volume; Phase H migrates to S3 + CDN. |
| Audit logs | DB only | Inherited from Neon | Inherited | Append-only triggers protect against tampering; restore = restore the whole DB. |
| Filament admin uploads | Same as object storage | Same | Same | Same. |
| Secrets (.env) | Vault NOT in use; .env on app server | Manual | Manual | Phase F deliverable: move to a managed secret store. |

**Phase F (Disaster Recovery + Operations) is queued to harden this surface.** For Closed Beta, the operator accepts the single-server file storage risk because total volume is small and a manual `rsync` per day is feasible.

---

## 8. Operational readiness

| Area | State | Owner |
|---|---|---|
| Deployment | Manual SSH + `git pull` + `php artisan migrate --force` | Operator |
| Cron jobs | `* * * * * php artisan schedule:run` configured | Operator |
| Queue worker | `php artisan queue:work` under cron-managed loop | Operator |
| Logs | `storage/logs/laravel.log` rotated nightly | Operator |
| Sentry | Wired (`SENTRY_LARAVEL_DSN` in .env) | Operator |
| Resend (email) | Wired (`MAIL_MAILER=resend`, `RESEND_API_KEY`) | Operator |
| Tap (payments) | Webhook configured at `/api/v1/webhooks/tap` | Operator |
| Twilio (SMS/WhatsApp) | Configured + verified during B-final | Operator |
| Cloudflare (DNS/SSL) | Already proxied | Operator |
| Neon (Postgres) | Already provisioned | Operator |
| Monitoring | Sentry + manual log tail. No PagerDuty yet. | Operator |
| On-call | Operator only (no rotation) — appropriate for Beta cohort size | — |
| Runbook | None yet — Phase F deliverable | — |

---

## 9. Support-risk areas (operator should watch these closely)

These are the surfaces where a Beta-cohort user complaint is most likely:

1. **Refresh-token flow on slow networks** — a mobile app on poor Wi-Fi can hit the 60-min access token expiry mid-action. UX: surface the refresh as transparent retry.
2. **`RATE_LIMITED` UX** — login throttle is aggressive (5 per (email\|IP) per minute). User retypes password → blocked → support ticket. Operator: educate Beta cohort.
3. **Lesson video playback** — external Bunny/Mux provider IDs. If those tokens are misconfigured for a tenant, students see "video unavailable" with no clean fallback message.
4. **Certificate generation timing** — PDF render is sync; >10s renders block the user. CP4-10 throttled to 5 per 5 min per user to absorb retries. Operator: graph p95 render time.
5. **`SendOutboxEmailJob`** — under sync queue (NOT in production: only in test), it has tenant-context churn. Production is async + isolated. But monitor outbox stale rows.
6. **Search latency** — hybrid FTS + ILIKE. No materialized view yet. Sub-200ms for current dataset; will degrade at scale (Phase E).
7. **Cart-clear-before-webhook race** — H-1 fixed but the test under load isn't done (Phase E).
8. **2FA enforcement on admin** — operator can lock themselves out if they lose the secret. Recovery codes are emailed at setup; operator must save them somewhere external.

---

## 10. Beta cohort guidance

- Start with **3-5 internal users** for the first 7 days. Operator on the line.
- Add **10-20 invited Beta users** after Day 7, only if Section 5 monitoring shows no anomalies.
- Cap at **50 users / 5 programs** before Phase E load testing completes.
- Suspend new invites if `failed_jobs` > 10 per day OR `5xx` count > 5 per day.

---

## 11. Phase D — CLOSED

This report supersedes any earlier go/no-go for v1. The platform is shipped to Closed Beta status as of **2026-05-14**.

The next workstream is **Closed Beta Controlled Exposure**. Operator runs the cohort. Phase E (Performance + Scale) starts when usage signals justify it. Phase F (DR + Ops) starts when Beta stability is proven. Phase G (UX + feedback) and H (expansion + mobile launch) follow.

`Phase D = CLOSED`.
