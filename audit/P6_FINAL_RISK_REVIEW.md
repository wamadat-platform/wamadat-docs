# 🎯 P6 — Final Risk Review

**Date:** 2026-05-13
**Scope:** Holistic readiness assessment across every system area, with launch-gating verdict.

---

## ⏱ One-line verdict

> **Soft-launch ready** with a small known list. **Production-ready** (public marketing + paid acquisition) once the deferred items in §5 land. Backend is the strongest layer; the deferred items are mostly real-payment-gateway activation and a couple of UX polish items.

---

## 1️⃣ Scorecard (out of 10)

| Domain | Score | Notes |
|---|---|---|
| **Backend architecture** | 9 | Modular DDD layout, strict tenant isolation, append-only audit, idempotent webhooks, transactional writes, sane rate-limits. Tested at 107/107 on the standard suite. |
| **Frontend code quality** | 8 | TypeScript strict, shared design-system primitives (Skeleton/EmptyState/Toaster/FormField), RTL-clean, error boundaries per segment, offline awareness, dialog focus traps. |
| **UX (usability + flow)** | 8 | Learn area treated as a workspace, autosave drafts, video progress resilient to tab switch / refresh, quiz with kbd nav + autosave, AI assistant with retry. Notes/Q&A optimistic. |
| **UI (visual polish)** | 7 | Brand palette + Tajawal font consistent. No charts yet; mobile drawer + safe-area handled. Dark mode deferred. Skeleton/loading transitions soft. |
| **Security** | 9 | 0 IDORs across 8 resources, password hashing + HIBP guard, encrypted cookies, security headers, signed webhooks, signature failure → 400, RBAC via Spatie, defense-in-depth on delete-own (404 not 403). Sentry PII off. |
| **Payments** | 7 | Webhook idempotency rock-solid, refund flow exists, ZATCA invoice issued on payment, mock gateway works end-to-end. **Real gateways (Tap/Tamara) require live keys to score above 8.** |
| **Operations** | 8 | Filament control room (failed jobs / outbox / webhooks / queue depth / health) wired up in P4.3. Sentry scope binding in P4.4. Audit-trail on every action. No Horizon yet (deliberately deferred). |
| **Tests** | 7 | Standard suite green, parallel-safe TenantTestCase. **Module tests under `app/Modules/*/Tests/` still need a custom harness** (P4.2.A backlog). No E2E (Playwright) yet. |
| **Performance** | 8 | Eager-load discipline verified in P4.1, three new perf indexes, no N+1 across the audited flows. Slow `ILIKE` search noted for the catalog (acceptable until ~5k programs). |
| **Maintainability** | 8 | Single coherent style (pint preset clean), one toast helper, one form-submit hook, one dialog hook, one retry pattern. Audit docs in `docs/audit/` are the project's institutional memory. |
| **Launch readiness** | **8** | Soft-launch tomorrow with eyes open is fine. Public/paid marketing should wait for §5 items. |

**Average: 7.9 / 10.**

---

## 2️⃣ Soft launch — green flag?

✅ **YES** if "soft launch" means:
- Invited / private list of students enroll using existing emails
- Mock gateway or real Tap with a test merchant key
- 2-3 published programs (not hundreds)
- Operator on standby for the first 7 days to watch Sentry + Filament ops panel
- No press, no paid acquisition

What proves the verdict:
- End-to-end flow (signup → enroll → learn → certificate) covered by tests + manual checklist (P5)
- 0 critical placeholders in user-facing surfaces
- Operational visibility (failed jobs, outbox, webhooks, queue depth, health) all surface in admin within seconds
- Audit log is append-only, signed by user — every operator action traceable
- Sentry will catch the unknown unknowns (DSN-only switch — code is ready)

---

## 3️⃣ Full production — green flag?

🟡 **NOT YET.** Needs the §5 deferred items first. The single biggest gap is **live payment gateways under a production merchant account**. Everything else is non-blocking.

---

## 4️⃣ Blockers (must do before full production)

| # | Blocker | Why | Effort |
|---|---|---|---|
| B-1 | Live Tap / Tamara merchant account + production keys + webhook signature secret | Mock gateway is a demo, not a payment processor. Real money requires real credentials. | external — KYC + onboarding (1-2 weeks elapsed) |
| B-2 | Sentry DSN set in production `.env` + sourcemap upload from CI | We added the scope binding (P4.4); operators just need to flip the switch | minutes |
| B-3 | Domain + SSL + DNS for production + tenant subdomain wildcard | Subdomain routing already in code, just needs DNS | hours |
| B-4 | Production Postgres + Redis with backups + monitoring | Currently dev/staging only | infra — 1 day |
| B-5 | Resend API key (or alternative) + verified sender domain | Email delivery is the platform's lifeline (cert, receipt, password reset) | hours, depends on DNS |

Five external/operational items. No code work.

---

## 5️⃣ Residual risks (open, watched, non-blocking)

| Risk | Mitigation in place | Mitigation gap |
|---|---|---|
| Single payment gateway provider | Mock fallback exists | Only Tap on day-1; Tamara when ramped |
| ILIKE catalog search seq-scans past ~5k programs | Postgres FTS already in `PostgresSearchEngine` | Add `pg_trgm` GIN index OR cut over to Meilisearch (already installed) once volume warrants |
| Module tests under `app/Modules/*/Tests/` still red | Standard `tests/` suite is green and CI-gated | Custom harness — P4.2.A backlog |
| No E2E tests (Playwright) | Pest + manual smoke checklist | Add Playwright in P5 ops phase |
| No load test | Architecture is conservative (DB-backed, indexed) | First public push will be the real test; HealthStatusWidget + QueueDepthWidget surface stress fast |
| `app/Modules/Tenancy/Tests/Feature/SubdomainResolutionTest` reads from real DNS in tests | — | Move to mocked resolver |
| AI assistant rate limit hard-coded 20/min | Cleanly tunable via route middleware | If Claude bill spikes, lower the cap |
| Single-region Postgres | OK for KSA users; latency ok | Multi-region failover is Phase 3 from the master roadmap (`docs/02-architecture.md`) |
| Dark mode deferred | Light-only UI works fine for both modes | P22 cosmetic |
| Video provider is YouTube | Free, works, but not DRM/anti-piracy | Bunny Stream / Mux in P10.5 once content licensing matters |

---

## 6️⃣ What to defer (consciously)

| Feature | Why defer |
|---|---|
| Horizon dashboard | Filament ops panel covers the actual operator needs cheaper |
| Telescope in prod | Sentry is the production-grade analog; Telescope is dev-only |
| Pulse | Will plug in if/when load makes us care about microsecond breakdowns |
| Charts on admin dashboard | Stats strip is enough for soft launch — measure before optimizing |
| Resumable uploads (tus) | Plain S3 upload works for the assignment sizes we target |
| Wishlist / Affiliate UI pages | Backend exists; UI was removed; reactivate once retention data justifies |
| Bundles | DORMANT per owner decision; backend + frontend pages live in BACKLOG for revival |
| Multi-region | Mawhiba's actual SaaS-launch trigger — not relevant pre-100k MAU |
| AI streaming | Backend would need SSE refactor; non-streaming works |

---

## 7️⃣ What to delete (dead weight)

Already removed in earlier phases:
- BundleResource / BundleModel — gone
- Deleted dashboard pages (gifts, affiliate, wishlist, calendar) — gone, no orphan imports
- Deleted public pages (`/apps`, `/workshops`, `/affiliate`) — gone, sidebar updated

Recommended additional deletes (cosmetic only):
- The `seedBundles()` historical comment in `BigDemoSeeder` — small but signals dead intent
- The `// PHASE NN follow-up` comments scattered across the code — convert to `BACKLOG.md` entries to keep code clean

---

## 8️⃣ Post-launch improvements (priorities ranked)

1. **Activate real payment gateway** (B-1 from §4) — gates revenue, days of effort
2. **Sourcemap upload to Sentry in CI** — bad stacks suck to debug, hours of effort
3. **Module test harness fix (P4.2.A)** — currently 17 red tests, days
4. **Playwright E2E** — 4-6 critical journeys; days
5. **Subdomain DNS + SSL automation** for new tenant provisioning — important when 2nd tenant onboards
6. **Bunny Stream cutover** for any program with paid content rights
7. **pg_trgm / Meilisearch search upgrade** when catalog passes ~1k programs
8. **Dark mode + avatar upload** (PHASE 22 polish)
9. **Camera-based QR attendance scan** (was the original P12 stub)
10. **AI streaming response** (UX-only improvement; backend SSE refactor)

---

## ✅ P6 closure — bottom line

The platform is **soft-launch ready today**. The remaining gap to full production is 5 operational items (none of them code work). Every layer scored 7/10 or higher; nothing is structurally broken.

The risk profile is honest: known stubs are documented, deferred items have reasons, and the operator's control room (P4.3 + P4.4) gives the team eyes on the production state the moment it starts taking traffic.
