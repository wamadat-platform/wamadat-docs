# 🚀 Launch Engineering — Master Orchestrator

**Phase:** Production Infrastructure Finalization (Stage 0 §1 from P7)
**Date opened:** 2026-05-13
**Mindset:** This is not feature work. Every step here is one of: provision a thing, verify a thing, wire a key. The platform is ready; the infrastructure isn't yet.

---

## The 5 external items, ordered by criticality + dependency

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1   POSTGRES PRODUCTION (Neon)                              │
│          → everything else depends on the DB existing            │
│          ⏱ 30-60 min                                             │
├─────────────────────────────────────────────────────────────────┤
│ STEP 2   REDIS PRODUCTION (Upstash)                              │
│          → queue / sessions / cache. Required before deploy      │
│          ⏱ 15-30 min                                             │
├─────────────────────────────────────────────────────────────────┤
│ STEP 3   RESEND PRODUCTION                                       │
│          → auth flows + receipts + certificates. Email is        │
│          critical-path. Domain SPF/DKIM propagation: 1-24h       │
│          ⏱ 30-60 min setup + DNS propagation                     │
├─────────────────────────────────────────────────────────────────┤
│ STEP 4   DNS + SSL (Cloudflare)                                  │
│          → apex + wildcard + email records. Many other steps     │
│          need DNS records (Resend DKIM, Vercel verification)     │
│          ⏱ 30 min (depends on Step 3 records too)                │
├─────────────────────────────────────────────────────────────────┤
│ STEP 5   TAP KYC + LIVE KEYS                                     │
│          → Tap account approval is the longest lead time.        │
│          Submit KYC docs IMMEDIATELY in parallel with above.     │
│          ⏱ 1-2 weeks (mostly waiting on Tap)                     │
└─────────────────────────────────────────────────────────────────┘
```

### Critical insight

**Tap KYC starts ON DAY 1 in parallel with everything else.** Submitting KYC docs takes ~30 minutes; Tap's approval cycle is days-to-weeks. Don't serialize it — it'll be the long pole.

By the time Postgres + Redis + Resend + DNS are wired (typically 1-2 days of focused work), Tap KYC will hopefully be close to approved. If it isn't, the platform can still soft-launch with mock gateway + invite-only users while waiting for Tap.

---

## Why this order (and not another)

| Order rationale | Why |
|---|---|
| Postgres first | Auth, sessions, tenants, every API route depends on it. Zero work happens without DB. Get this right; if it's wrong nothing else works. |
| Redis second | Queue + sessions live here. Email outbox drains via queue. Email tests need queue worker. So Redis before Resend. |
| Resend third | Welcome / verify / password-reset emails are part of the auth flow. Auth flow can't be tested without email. |
| DNS+SSL fourth | DNS records for Resend DKIM/SPF are written in Cloudflare. Vercel project verification also needs Cloudflare access. So Cloudflare account ready by step 4, but DNS records may be written progressively from step 3 onward. |
| Tap last | KYC is bureaucratic, not engineering. Kick it off day 1 but don't BLOCK on it. |

---

## What you (the human) do vs what I (Claude) do

**You do** (anything requiring credentials, billing, or KYC):
- Create accounts (Neon, Upstash, Resend, Cloudflare, Vercel, Hetzner+Forge, Tap)
- Connect billing
- Click through provisioning wizards
- Paste DNS records into Cloudflare
- Submit KYC docs to Tap

**I do** (anything that's code or text):
- Detailed runbooks for each step (per provider — already written)
- `.env.production` updates as you give me credentials
- Verification scripts (already written under `scripts/launch/`)
- The actual `deploy.yml` GitHub Actions workflow once we know the hosting target
- Cloudflare API calls IF you give me an API token (optional but speeds things up)
- Post-provisioning smoke tests against the live system

---

## The 5 runbooks (in execution order)

1. **[01-postgres-neon.md](./01-postgres-neon.md)** — Provision Neon project, parse connection string, configure schema-per-tenant, run landlord + tenant migrations, verify with smoke test.
2. **[02-redis-upstash.md](./02-redis-upstash.md)** — Provision Upstash database, allocate DB indices (cache/session/queue/broadcast), wire TLS endpoint into `.env.production`, verify with PING + persistence check.
3. **[03-resend.md](./03-resend.md)** — Resend account, sender domain verification via Cloudflare DNS records (SPF + DKIM + DMARC), API key + webhook secret, synthetic send test.
4. **[04-dns-cloudflare.md](./04-dns-cloudflare.md)** — Apex A/CNAME, wildcard `*.wamadat.academy`, email auth records (Resend), Vercel/host verification records, Cloudflare SSL mode = Full (strict), page rules.
5. **[05-tap-kyc.md](./05-tap-kyc.md)** — KYC documents checklist, Tap merchant onboarding, live keys, webhook URL configuration, signature secret, first live charge dry-run.

---

## Verification scripts (under `scripts/launch/`)

Run each after the corresponding step. **All must pass before proceeding.** If any fails, the runbook for that step has a Troubleshooting section.

```
scripts/launch/
├── verify-prod-postgres.sh   ← after step 1
├── verify-prod-redis.sh      ← after step 2
├── verify-prod-email.sh      ← after step 3
└── verify-dns.sh             ← after step 4 (covers DNS for all prior records too)
```

---

## Go / No-Go gates

These are hard gates. **DO NOT** proceed to the next step if any item in the current gate is red.

### Gate 1 — Database ready
- [ ] `verify-prod-postgres.sh` passes (8 checks green)
- [ ] Landlord migrations applied to `wamadat_landlord` schema
- [ ] First tenant (`wamadat`) provisioned with all tenant migrations applied
- [ ] PITR confirmed enabled on the Neon dashboard
- [ ] Connection string saved to a secrets manager (NOT just in `.env.production`)

### Gate 2 — Cache + Queue ready
- [ ] `verify-prod-redis.sh` passes
- [ ] Queue worker can connect (test with `php artisan queue:work --once --tries=1`)
- [ ] Session driver writes and reads back successfully
- [ ] TLS endpoint verified (`rediss://` scheme working)

### Gate 3 — Email ready
- [ ] `verify-prod-email.sh` passes
- [ ] Real email lands in a real inbox within 60 seconds
- [ ] SPF + DKIM + DMARC all show "pass" in the received email's raw headers
- [ ] Webhook secret matches Resend dashboard

### Gate 4 — DNS + SSL ready
- [ ] `verify-dns.sh` passes
- [ ] `wamadat.academy` resolves correctly globally
- [ ] `*.wamadat.academy` wildcard works
- [ ] SSL certificate valid for both apex and wildcard
- [ ] HTTPS enforced (no http:// access)

### Gate 5 — Tap live
- [ ] Live keys received from Tap
- [ ] Webhook URL configured in Tap dashboard pointing to production
- [ ] First test charge with $1 SAR succeeds + webhook lands + order moves to paid
- [ ] Refund the test charge → refund webhook lands + order moves to refunded
- [ ] All audit log entries present for both events

### Final gate — Production ready for soft launch
- [ ] All 5 gates above green
- [ ] `php artisan sentry:test` returns event visible in Sentry within 30s
- [ ] `curl https://api.wamadat.academy/api/v1/health` returns 200 with all 8 probes green
- [ ] Full Stage 0 manual smoke checklist from `P5_PRODUCTION_READINESS_SWEEP.md` §6 passes (10 steps)
- [ ] Backup-restore drill performed on staging copy
- [ ] Runbooks printed / pinned for the operator

---

## Decisions already made (don't re-litigate)

| Decision | Choice | Why |
|---|---|---|
| DB provider | Neon | Schema-per-tenant native, PITR, branching, pgvector built-in, Singapore region |
| Cache/queue | Upstash | Serverless pricing, TLS default, free tier covers soft launch |
| DNS | Cloudflare | Free SSL, fast propagation, CDN bonus |
| Email | Resend | Already in code, KSA-friendly, modern API |
| Payments | Tap | Already in code, KSA-native, Mada support |
| Frontend host | Vercel | Best-in-class for Next.js 15 |
| Backend host | Laravel Forge + Hetzner CCX23 | Best Laravel deploy experience, $45/mo combined |

Approximate monthly cost: **~$110/month** all-in (Vercel + Neon + Upstash + Forge + Hetzner + Cloudflare + Resend Pro).

---

## What gets touched / created in this phase

Code-side: minimal. The `.env.production.example` template was written in Stage 0 Handoff already. This phase mostly _fills in values_ that were left blank.

```
.env.production                   ← created by you, never committed
docs/launch/00-LAUNCH_ENGINEERING.md  ← this file
docs/launch/01-postgres-neon.md       ← Postgres runbook
docs/launch/02-redis-upstash.md       ← Redis runbook
docs/launch/03-resend.md              ← Resend runbook
docs/launch/04-dns-cloudflare.md      ← DNS runbook
docs/launch/05-tap-kyc.md             ← Tap runbook
scripts/launch/verify-prod-postgres.sh
scripts/launch/verify-prod-redis.sh
scripts/launch/verify-prod-email.sh
scripts/launch/verify-dns.sh
```

---

## What happens AFTER all 5 are done

That's Stage 1 — Soft Launch. See `P7_FINAL_LAUNCH_ROADMAP.md`. Specifically:

- Invite 10-30 real students (closed list)
- 1-3 published programs (real content)
- Operator on call 8h/day
- Daily 15-min standup: Sentry inbox + failed jobs + outbox failures + webhook failures
- Watch the 6 signals from P7 Stage 1 table
- 5-day green run = move to Stage 2

But that's later. Right now: 5 boxes to tick.

---

**Start here:** open `01-postgres-neon.md` and follow it top-to-bottom. Each runbook has a "When done" section that tells you what to confirm before moving to the next.
