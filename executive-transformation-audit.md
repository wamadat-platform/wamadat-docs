# Executive Transformation Audit — Wamadat Academy

> **Generated**: 2026-06-08 · **Scope**: full repository inspection · **Method**: file-level evidence only (no assumptions from stale memory or README claims). Findings cite paths and quote facts.
> **Auditor's note**: The triggering prompt assumed several gaps based on a 24-day-old BACKLOG.md + memory snapshots. Direct code inspection found that **some of those gaps are already closed** (admin 2FA, CI/CD, tenant-queue config, Sentry SDK wiring, subdomain finder). Those are documented here with the actual evidence. Real gaps are documented with the same rigor.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Actual Completion Percentage by Area](#2-actual-completion-percentage-by-area)
3. [Launch Readiness Score](#3-launch-readiness-score)
4. [Closed Beta Readiness Score](#4-closed-beta-readiness-score)
5. [Public Launch Readiness Score](#5-public-launch-readiness-score)
6. [Enterprise Readiness Score](#6-enterprise-readiness-score)
7. [Critical Findings](#7-critical-findings)
8. [High Findings](#8-high-findings)
9. [Medium Findings](#9-medium-findings)
10. [Low Findings](#10-low-findings)
11. [Technical Debt Register](#11-technical-debt-register)
12. [Security Risk Register](#12-security-risk-register)
13. [Tenant Isolation Risk Register](#13-tenant-isolation-risk-register)
14. [DevOps Readiness](#14-devops-readiness)
15. [QA Readiness](#15-qa-readiness)
16. [Frontend Readiness](#16-frontend-readiness)
17. [Backend Readiness](#17-backend-readiness)
18. [AI Readiness](#18-ai-readiness)
19. [Data Architecture](#19-data-architecture)
20. [Compliance Readiness](#20-compliance-readiness)
21. [Commercial SaaS Readiness](#21-commercial-saas-readiness)
22. [Required Fixes Before Closed Beta](#22-required-fixes-before-closed-beta)
23. [Required Fixes Before Public Launch](#23-required-fixes-before-public-launch)
24. [Required Fixes Before Enterprise Customers](#24-required-fixes-before-enterprise-customers)
25. [Final Verdict](#25-final-verdict)

---

## 1. Executive Summary

Wamadat Academy is a **multi-tenant Arabic-first LMS SaaS** that is **functionally complete for a single-tenant Closed Beta**. The platform is past `Foundation` (as the README claims) and past Phase D + Operational Hardening Sprint (both closed 2026-05-14). It now sits in a **soft-launch holding pattern**: infrastructure is green, the published-program cap has been brought into compliance (`5/5`), but **no real users have been invited** — only one verified user, zero payment webhooks in the last 25 days.

The codebase reflects a level of engineering discipline that is uncommon at this stage: 25 modules in DDD-style layout, 47 controllers, 23 Filament resources, ~75 API endpoints, 75 migrations, explicit canonical error envelope, signed-URL media, PDPL-compliant data export/delete, encrypted national ID with HMAC hash, mandatory 2FA on both admin panels, comprehensive CI pipeline.

The real risks today are concentrated in **three areas**:

1. **Frontend test coverage is effectively zero** despite Vitest + Playwright being installed and CI running `pnpm test`. CI passes trivially.
2. **Cache layer is not tenant-prefixed.** Spatie's tenant-aware caching needs explicit setup; defaults are in use. A cache-key collision between tenants would leak data.
3. **Documentation is stale and misleading.** README still says `Status: Foundation`; BACKLOG.md predates Phase D + Hardening Sprint + No-Code initiative; PROGRESS.md is from 2026-05-11 and contradicts itself.

Closing those three gaps + adding a small set of Playwright smoke tests + wiring Sentry DSN moves the platform from "**Beta-ready but not Beta-launched**" to "**Beta-launched with real safety net**".

For Public Launch and Enterprise readiness, the gaps are larger: real-traffic load testing (Phase E deferred by design), multi-tenant cache isolation, support tooling, DR drills against a second host, and a hardened CI/CD that actively blocks deploys on failed contract tests.

---

## 2. Actual Completion Percentage by Area

| Area | % | Evidence |
|---|---|---|
| Backend domain logic | **85%** | 25 modules, 47 controllers, 75 migrations, 23 Filament resources, 75+ API endpoints — all wired to real implementations (no placeholders, zero `TODO`/`FIXME`/`HACK` markers in `backend/app`) |
| Frontend pages | **80%** | 48 routes under `app/[locale]/` covering catalog, dashboard, learn, checkout, auth, page builder, design-system, offline |
| Database schema | **90%** | 9 landlord + 66 tenant migrations executed; `Tenancy Drift Check: aligned` per latest summary |
| Backend test files | **40%** | 17 Pest files (15 Feature + 2 Unit, 0 Arch); coverage unmeasured; 53 legacy tests known-failing (R-OPEN-32 deferred) |
| Frontend test files | **2%** | Vitest installed, `tests/` folder empty; Playwright installed, no `playwright.config.ts`, no test files |
| Ops / runbooks | **85%** | 7 runbooks in `docs/operations/`, 4 Windows Task Scheduler tasks, daily backup verified (last: today 09:22), restore drill log present |
| Security controls | **80%** | OWASP headers, signed URLs, encrypted PII + HMAC, 2FA mandatory (both panels), HIBP password check, PDPL endpoints, audit log w/ immutable trigger. Cache isolation gap is the residual risk. |
| Observability | **55%** | Sentry SDK wired (BE + FE), CI deploy workflow uploads sourcemaps. **DSN env vars empty** (`backend/.env.example` shows `SENTRY_LARAVEL_DSN=`). Better Stack not wired. |
| Documentation | **40%** | 10 reference docs in `docs/` are detailed; 4 runbooks added in Hardening Sprint; **but root-level README, BACKLOG, PROGRESS are stale by ≥22 days** and contradict actual state. |
| CI/CD | **70%** | `.github/workflows/ci.yml` runs Pint, PHPStan, Pest, vitest, build, audit on push/PR. `.github/workflows/deploy.yml` exists. Gaps: no Playwright in CI, no `composer audit` failing on high, no migration dry-run, branch protection not verified. |
| Compliance | **75%** | PDPL (export + delete) shipped, ZATCA Phase 2 lib in composer, audit logs immutable, MFA available. Cookie banner / consent flow / DPA workflow not verified. |
| AI service | **60%** | FastAPI scaffold exists with deps (Anthropic, OpenAI, pgvector, redis, structlog, sentry). Implementation depth not deeply inspected this audit — flagged. |

**Overall weighted completion**: ~70%, with weight on infrastructure + backend over tests + docs.

---

## 3. Launch Readiness Score

Composite score across the four launch tiers (each 0–100, hard floors apply):

| Tier | Score | Floor blockers |
|---|---|---|
| Closed Beta (single tenant, 50 users, 5 programs) | **88 / 100** | Cache isolation + Sentry DSN + 3 Playwright smoke |
| Public Launch (open registration, multiple tenants) | **52 / 100** | Cache isolation, subdomain DNS, Better Stack, real load test, full E2E suite, support tooling |
| Enterprise SaaS (SSO, SLA, audit reports) | **28 / 100** | SSO/SAML, SLA monitoring, customer-facing audit log export, DR drill against second host, multi-region, dedicated tenant DB option, compliance certificate package |
| Top-tier global LMS SaaS | **22 / 100** | All of the above + i18n beyond ar/en, advanced reporting, native mobile apps, marketplace, ML-powered recommendations measured against industry benchmarks |

---

## 4. Closed Beta Readiness Score: **88 / 100**

**What's solid**:
- Daily summary `2026-06-08 09:22` shows: Postgres UP, queue worker running, 0 errors in 24h, 0 failed jobs, latest backup 0h ago, programs at `5/5 (at cap)`, users at `1/50 (ok)`, tenancy drift `aligned`, DB sizes healthy (landlord 9 MB / tenants 17 MB), disk 541 GB free.
- 4 Task Scheduler tasks operational: backup (03:00), queue worker (logon-triggered loop), failed-jobs check (every 15 min — verified by 24-entry tail of `alerts-log.txt` all `OK`), daily summary (08:00).
- All 11 pre-launch checklist items from `docs/operations/06-closed-beta.md` are verifiable.

**What's missing for Beta**:
- **Sentry DSN** (`SENTRY_LARAVEL_DSN=` empty in `.env.example`). Without a DSN, the daily summary is the only error signal — fine for one user, blind for fifty.
- **One Playwright smoke** to prove the catalog → checkout → enrollment golden path works after each deploy. Manual testing of this on every change is operator burden.
- **Cache prefix tenant isolation** — currently low risk (one tenant exists) but the moment a second tenant is provisioned this becomes a critical leak vector.

**12-point deduction breakdown**: -5 Sentry DSN, -4 cache isolation, -3 no Playwright safety net.

---

## 5. Public Launch Readiness Score: **52 / 100**

**Categorical gaps**:
- **No real load test ever run.** Phase E (P1-4) deliberately deferred — measure from real traffic.
- **No DR drill against a second host.** Single-machine topology (`docs/operations/00-deployment-topology.md`); zero redundancy.
- **No analytics product layer.** PostHog/Plausible not present (L-10 backlog).
- **No support pipeline.** Tickets/CRM modules exist in code (`Support` module) but workflow + SLAs not validated.
- **CI does not block deploy on E2E failures.** No Playwright in pipeline.
- **Subdomain DNS not configured.** SubdomainTenantFinder exists, but the `TENANCY_SUBDOMAIN_BASE=wamadat.test` is dev-only. Wildcard DNS + cert + tenant onboarding flow are unproven in prod.

---

## 6. Enterprise Readiness Score: **28 / 100**

**Hard requirements not present**:
- SSO / SAML / OIDC integration (only Sanctum tokens today)
- Customer-facing audit log export
- Dedicated tenant DB option (today: schema-per-tenant in shared cluster)
- Multi-region failover
- Signed compliance package (SOC 2 / ISO 27001 / pen-test report)
- Contractual SLA + incident response runbook + status page
- Custom domain management UI (only cookie/header tenant resolution today)
- Tenant-level data residency controls (PDPL-only today)

---

## 7. Critical Findings

> **Format**: ID, Title, Priority, Area, Business Impact, Technical Impact, Security Impact, Complexity, Dependencies, Owner, Validation, Acceptance, Effort.

### C-001 — Cache layer is not tenant-prefixed; cross-tenant leak risk on day-two
- **Priority**: Critical (latent)
- **Area**: Backend / Tenancy
- **Business impact**: First multi-tenant collision = data leak between academies. Reputation/legal blast radius is total.
- **Technical impact**: `config/cache.php` does not exist. Laravel defaults apply. `Spatie\Multitenancy\Concerns\UsesTenantConnection` does not auto-namespace cache keys. Code that uses `Cache::remember('homepage_programs', ...)` is shared across tenants.
- **Security impact**: PDPL-violating data leak if a key contains tenant-specific content (user lists, program counts, etc.).
- **Complexity**: Medium — publish Laravel cache config + register Spatie's tenant-aware cache or use `prefix` derived from `current_tenant()->slug`.
- **Dependencies**: None.
- **Owner**: Backend lead.
- **Validation**: New Pest test — set tenant A, cache `foo=A_value`; switch to tenant B, attempt to read `foo`, assert miss.
- **Acceptance**: Two simultaneous tenants cannot read each other's cached values for any key.
- **Effort**: 3–4h (incl. test).

### C-002 — Frontend tests folder is empty; CI passes trivially on `pnpm test`
- **Priority**: Critical (process gap)
- **Area**: Frontend / QA
- **Business impact**: Every frontend change ships unverified. Regressions cost the operator hours of manual recheck or surface as user-reported bugs.
- **Technical impact**: `frontend/tests/` exists but contains zero files. Vitest is installed and `pnpm test` is a CI step — runs to success on no tests. False positive on every PR.
- **Security impact**: No automated check that auth cookies are httpOnly, that protected routes 401 unauthorized users, etc.
- **Complexity**: Low to start (≥10 critical tests in <2h).
- **Dependencies**: None.
- **Owner**: Frontend lead.
- **Validation**: `pnpm test` reports ≥10 tests passing; coverage ≥50% statements on auth + cart stores.
- **Acceptance**: Vitest suite covers auth store, cart store, API client signed-url builder, i18n routing helper.
- **Effort**: 1 day initial → recurring per feature.

### C-003 — No Playwright E2E tests; checkout golden path is unguarded
- **Priority**: Critical (regression risk)
- **Area**: Frontend / QA
- **Business impact**: Checkout is the revenue-bearing flow. A broken `/checkout` ships silently. The 25-day window of zero payment webhooks (per daily summary) means the operator may not notice a broken checkout for a long time.
- **Technical impact**: `@playwright/test` is installed, chromium 1217 browsers downloaded, but no `playwright.config.ts`, no test directory, no scripts in `package.json` for `playwright test`.
- **Security impact**: No verification that auth-required pages actually require auth in the browser context.
- **Complexity**: Medium — needs running BE + FE locally + test data setup that's deterministic.
- **Dependencies**: Sentry DSN (so smoke failures emit signals).
- **Owner**: QA lead.
- **Validation**: `pnpm e2e` runs ≥5 specs to green: home (ar) loads, sign-in, browse program, simulate checkout, view dashboard.
- **Acceptance**: Specs are reliable across 10 consecutive runs (flake rate <5%).
- **Effort**: 2 days for 5 smokes; 5 days for full 14-flow suite.

### C-004 — Sentry DSN environment variables are empty; error visibility is blind
- **Priority**: Critical (observability)
- **Area**: Observability
- **Business impact**: Operator finds out about errors when users complain. The daily summary `Errors 24h: 0` is a count of stack traces in `laravel.log`, not an aggregation. With one user, fine. With fifty, this is a black hole.
- **Technical impact**: `backend/.env.example` shows `SENTRY_LARAVEL_DSN=`. `frontend/sentry.server.config.ts` is wired but `enabled: Boolean(process.env.NEXT_PUBLIC_SENTRY_DSN)` evaluates false without the env var. Both Sentry packages are installed and configured — only the DSN is missing.
- **Security impact**: None directly; downstream effect is slower incident detection.
- **Complexity**: Trivial — create Sentry project (free tier), set `SENTRY_LARAVEL_DSN` + `NEXT_PUBLIC_SENTRY_DSN`, restart workers.
- **Dependencies**: External (Sentry account).
- **Owner**: Ops / DevOps.
- **Validation**: `php artisan sentry:test` (command exists at `SentryTestCommand.php`) returns success; frontend dummy error triggers an event in the Sentry dashboard.
- **Acceptance**: A handled error in BE and a thrown error in FE both appear in Sentry within 60s.
- **Effort**: 30 min including dashboard setup.

### C-005 — `Errors 24h: 0` is computed by simple log grep, will silently break under log rotation
- **Priority**: Critical (monitoring brittleness)
- **Area**: Observability
- **Business impact**: The morning summary is the operator's primary signal source. If it reports green for a wrong reason, the operator misses real fires.
- **Technical impact**: `daily-summary.ps1` reads `backend/storage/logs/laravel.log` directly; when Laravel rotates the log (daily channel), errors land in `laravel-YYYY-MM-DD.log` and the grep misses them. Spike of `Errors 24h: 0` for weeks may not reflect zero errors.
- **Security impact**: None directly.
- **Complexity**: Low — adjust grep to include rotated files modified within 24h.
- **Dependencies**: None.
- **Owner**: Ops.
- **Validation**: Inject test error into a rotated log file (yesterday); rerun summary; verify count > 0.
- **Acceptance**: Summary catches errors across all log files modified in last 24h.
- **Effort**: 1h.

---

## 8. High Findings

### H-001 — `system_users` 2FA implementation: status not measured against the policy
- **Priority**: High
- **Area**: Admin security
- **Business impact**: 2FA bypass on /super = total tenant compromise.
- **Technical impact**: `SuperPanelProvider::authMiddleware` includes `RequireTwoFactor::class` (line 86 with comment `ADR-019: 2FA mandatory for super admins`). Need to verify: (a) no system_user can exist with `two_factor_secret IS NULL` and access /super, (b) middleware rejects vs redirects to setup, (c) recovery codes generation is single-use.
- **Security impact**: High if middleware is permissive instead of strict.
- **Complexity**: Low — read middleware + write 1 Pest test.
- **Dependencies**: None.
- **Owner**: Backend security.
- **Validation**: Pest test — system_user without 2FA secret hits /super → expects 403 or redirect to setup, not bypass.
- **Acceptance**: 100% of /super routes require active 2FA secret.
- **Effort**: 2h.

### H-002 — No IP allowlist on /super or /admin
- **Priority**: High
- **Area**: Admin security
- **Business impact**: /super is internet-reachable. Any leaked password + leaked 2FA seed = panel compromise.
- **Technical impact**: No middleware in `SuperPanelProvider` enforces IP filter. `IpAllowlistMiddleware` does not exist (grep needed but not found in this audit).
- **Security impact**: High. Defense-in-depth gap.
- **Complexity**: Low — middleware + config list of CIDRs + 1 test.
- **Dependencies**: Operator must provide static IPs / VPN CIDRs.
- **Owner**: Backend security + Ops.
- **Validation**: Request from disallowed IP → 403; from allowed IP → continues to auth.
- **Acceptance**: /super accessible only from documented operator IPs.
- **Effort**: 3h.

### H-003 — Session timeout for /super not measurably ≤30 min
- **Priority**: High
- **Area**: Admin security
- **Business impact**: Idle admin sessions persist; physical access = panel access.
- **Technical impact**: `SESSION_LIFETIME=10080` in `.env.example` (one week). No /super-specific override. Filament session uses Laravel default lifetime.
- **Security impact**: High for shared/operator machines.
- **Complexity**: Low — `IdleSessionTimeoutMiddleware` or `auth.session` with `last_activity_at` check.
- **Dependencies**: None.
- **Owner**: Backend security.
- **Validation**: Inactive session >30 min → next /super request → 401.
- **Acceptance**: /super idle session timeout enforced at ≤30 min independent of global `SESSION_LIFETIME`.
- **Effort**: 3h.

### H-004 — Pest architecture tests are absent
- **Priority**: High
- **Area**: Backend quality
- **Business impact**: Architectural drift goes unnoticed (e.g., `Modules\X\Application` calling `Modules\Y\Infrastructure` directly, defeating DDD bounded contexts).
- **Technical impact**: `tests/Arch/` exists but is empty. `pest-plugin-arch` is in composer-require-dev. The "18 Bounded Contexts communicate via Domain Events only" README claim is not enforced by code.
- **Security impact**: Indirect — coupling makes future security patches risky.
- **Complexity**: Low — Pest arch is one-liner per rule.
- **Dependencies**: None.
- **Owner**: Backend.
- **Validation**: `pest --filter=Arch` runs ≥6 rules: each module's `Domain` layer cannot import `Infrastructure`, `Controllers` cannot import `Domain` types from another module, `Application` cannot import another module's `Application`, etc.
- **Acceptance**: 6+ arch rules pass on green main; failing rule blocks PR.
- **Effort**: 4h.

### H-005 — PHPStan baseline level not measured; ci.yml runs `analyse` without `--level` flag
- **Priority**: High
- **Area**: Backend quality
- **Business impact**: Type-related production bugs slip through; refactoring risk underestimated.
- **Technical impact**: `ci.yml` line 86: `./vendor/bin/phpstan analyse --memory-limit=2G --no-progress`. No `--level` so it uses the project default in `phpstan.neon`. We don't know the level. Larastan is installed.
- **Security impact**: Type-confusion bugs occasionally produce auth bypass (e.g., comparing `int` to `string`).
- **Complexity**: Medium — discover current level, attempt one bump, see error count, decide.
- **Dependencies**: None.
- **Owner**: Backend.
- **Validation**: `phpstan.neon` declares an explicit level; CI fails on regression.
- **Acceptance**: Level documented and stable; baseline file accepted for current debt.
- **Effort**: 4h to discover + lock; days if the bump triggers refactor.

### H-006 — Tenant isolation Pest test surface is single (TenantIsolationTest.php)
- **Priority**: High
- **Area**: Tenancy
- **Business impact**: Multi-tenant cross-leak = catastrophic. One test file is not "covered".
- **Technical impact**: Only `tests/Feature/TenantIsolationTest.php` exists. Need: cross-tenant API request rejection, cross-tenant signed-URL rejection, cross-tenant queue job context, cross-tenant cache key isolation.
- **Security impact**: High.
- **Complexity**: Medium — needs 2-tenant fixture in Pest.
- **Dependencies**: C-001 (cache isolation must exist first to test).
- **Owner**: Backend security.
- **Validation**: 4+ tenant-isolation tests in `tests/Feature/TenantIsolation*.php`.
- **Acceptance**: Each cross-tenant scenario has a deterministic fail-on-leak test.
- **Effort**: 6h.

### H-007 — Untracked operator scripts (`start.bat`, `stop.bat`, `scripts/start.ps1`, `scripts/stop.ps1`) and 13-line uncommitted change to `MediaAssetResource.php`
- **Priority**: High (repo hygiene + lost-work risk)
- **Area**: Repo safety
- **Business impact**: Dev machine wipe = loss of operator's daily workflow scripts. Uncommitted Filament resource changes might be a half-finished fix.
- **Technical impact**: `git status` confirms; the Filament edit (+13 lines) intent unknown without reading the diff.
- **Security impact**: None direct.
- **Complexity**: Trivial — diff, decide, commit.
- **Dependencies**: Operator clarification on intent of MediaAssetResource edit.
- **Owner**: Operator + AI executor.
- **Validation**: `git status` clean after.
- **Acceptance**: Either committed with rationale or reverted.
- **Effort**: 30 min.

### H-008 — Stale documentation cluster (README "Status: Foundation"; PROGRESS.md from 2026-05-11; BACKLOG.md from 2026-05-12)
- **Priority**: High (operator confusion + onboarding risk)
- **Area**: Documentation
- **Business impact**: Any new contractor/employee/auditor opens the repo and misreads project state. Decisions get made on wrong premises (as this very prompt did).
- **Technical impact**: README badge says `Status: Foundation`; reality is Closed-Beta-ready. PROGRESS.md self-contradicts ("7 phases complete" vs actual 25 modules in production-grade). BACKLOG.md lists items already shipped as open.
- **Security impact**: None.
- **Complexity**: Medium-low — write three accurate replacements.
- **Dependencies**: This audit serves as the source of truth.
- **Owner**: PMO / Tech writing.
- **Validation**: New contributor onboarding doc-review — they describe the project state correctly from docs alone.
- **Acceptance**: README, PROGRESS, BACKLOG reflect 2026-06-08 reality without exaggeration.
- **Effort**: 3h.

### H-009 — Beta-cap enforcement is observability-only (no code or DB constraints)
- **Priority**: High (policy-only enforcement is fragile)
- **Area**: Multi-tenancy
- **Business impact**: A bug in admin UI or a Filament action could publish a 6th program without warning until next daily summary (24h window).
- **Technical impact**: `docs/operations/06-closed-beta.md` line 62: caps are surfaced by daily summary; no middleware, no DB constraint. Operator-discipline gated.
- **Security impact**: None direct.
- **Complexity**: Low — `PublishProgramService` check + a feature flag for when caps lift.
- **Dependencies**: None.
- **Owner**: Backend.
- **Validation**: Attempting to publish 6th program returns a domain exception with translated message.
- **Acceptance**: Cap enforced at write-path, not read-path.
- **Effort**: 4h.

### H-010 — Queue worker supervision relies on at-logon Task Scheduler trigger
- **Priority**: High (single point of failure)
- **Area**: DevOps
- **Business impact**: If operator logs off the Windows box (e.g., remote desktop disconnect with "Sign out"), queue stops. Email outbox + signed-URL minting + future jobs all stall.
- **Technical impact**: `queue-worker.ps1` is registered "at logon" per `install-supervision.ps1`. Better Stack uptime check not wired.
- **Security impact**: None direct; secondary impact is delayed email delivery (auth OTP, password reset).
- **Complexity**: Medium — switch trigger to "at startup" running as a service account, or use NSSM/winsw.
- **Dependencies**: Operator decision on service account.
- **Owner**: Ops.
- **Validation**: After full Windows reboot without operator login → worker process running.
- **Acceptance**: Queue worker survives operator logoff.
- **Effort**: 4h.

---

## 9. Medium Findings

### M-001 — `phase-3` branch exists locally and on remote; intent unclear
- **Priority**: Medium
- **Area**: Repo hygiene
- **Business impact**: Confusion. Possible merge conflicts later.
- **Technical impact**: `git branch -a` shows `phase-3` + `remotes/origin/phase-3`. Last commit timing unknown without `git log phase-3`.
- **Owner**: Dev lead.
- **Validation**: Check `git log main..phase-3` for unmerged work.
- **Acceptance**: Either merged or explicitly deleted with note.
- **Effort**: 30 min.

### M-002 — `programs.status` write-path has no audit log entry
- **Priority**: Medium
- **Area**: Compliance / Audit
- **Business impact**: When operator unpublishes 25 programs (the actual fix that happened between 2026-05-14 and 2026-06-08), there is no per-program audit trail.
- **Technical impact**: `audit_logs_central` exists with immutable trigger but `programs` status transitions are not necessarily wired (need to check `Spatie\Activitylog` registration on `ProgramModel`).
- **Owner**: Backend.
- **Validation**: Publish then unpublish a program; query `activity_log`; assert two entries.
- **Acceptance**: Every state transition is logged with actor + before/after.
- **Effort**: 3h.

### M-003 — `composer audit` runs with `|| true` in CI (line 92); high vulns don't fail build
- **Priority**: Medium
- **Area**: CI/CD security
- **Business impact**: A high-severity Laravel/Filament/Spatie vuln ships unchallenged.
- **Technical impact**: `ci.yml` line 92 + similar on frontend line 140 (`pnpm audit --audit-level=high || true`).
- **Owner**: DevOps.
- **Validation**: Inject a known-vulnerable package version; CI fails.
- **Acceptance**: `high` vuln in any dep blocks deploy (with explicit allowlist for known-accepted CVEs).
- **Effort**: 1h fix + recurring allowlist maintenance.

### M-004 — `route:list --columns` failed with "option does not exist" — Laravel 11 changed the flag
- **Priority**: Medium
- **Area**: Developer ergonomics
- **Business impact**: Internal scripts that grep route list may be broken.
- **Technical impact**: Laravel 11 removed `--columns`; the equivalent is `--format` or `--except-vendor`. Any script invoking this with `--columns` will fail.
- **Owner**: Backend.
- **Validation**: `grep -r "route:list --columns" scripts/`.
- **Acceptance**: All ops scripts use the current flags.
- **Effort**: 30 min.

### M-005 — `ai-service/` test coverage and runtime not validated this audit
- **Priority**: Medium
- **Area**: AI service
- **Business impact**: AI assistant + recommendations rely on this service. If it's stale or broken, the `me/learn/lessons/{lesson}/ask` endpoint fails.
- **Technical impact**: Folder structure exists (`app/` + `tests/`), `pyproject.toml` is current, but uvicorn was not started + no test run this audit.
- **Owner**: AI lead.
- **Validation**: Start service, hit `/health`, run pytest suite, measure coverage.
- **Acceptance**: AI service runs locally + has ≥1 smoke test for `/health` + `/embed`.
- **Effort**: 4h.

### M-006 — `audit_logs_central_immutable` trigger over-engineered (BACKLOG DB-03)
- **Priority**: Medium
- **Area**: Data
- **Business impact**: Tenant deletion is harder than necessary; legitimate cleanups blocked.
- **Technical impact**: Per BACKLOG DB-03, trigger complicates tenant deletion. Needs softening without losing append-only audit guarantee.
- **Owner**: Backend.
- **Validation**: Tenant deletion test works without trigger-bypass hacks.
- **Acceptance**: Audit log remains append-only via app-level guard; physical row deletion allowed for GDPR/PDPL-mandated tenant erasure.
- **Effort**: 4h.

### M-007 — `categories.parent_id` + `discussion_replies.parent_reply_id` lack DB-level integrity (BACKLOG DB-04)
- **Priority**: Medium
- **Area**: Data
- **Business impact**: Orphaned rows possible; tree-walk queries crash on bad data.
- **Technical impact**: Foreign-key enforcement disabled at DB level by prior decision; app layer expected to enforce. Monitoring not present.
- **Owner**: Backend.
- **Validation**: Pest test attempts orphan insert + asserts app exception.
- **Acceptance**: Orphan path detected at write OR a nightly check alerts.
- **Effort**: 4h.

### M-008 — Partial indexes on soft-deleted rows missing (BACKLOG DB-05)
- **Priority**: Medium
- **Area**: Performance
- **Business impact**: Slow queries as soft-delete count grows.
- **Owner**: Backend.
- **Validation**: `EXPLAIN ANALYZE` on representative query before/after.
- **Acceptance**: Index added; query plan uses it.
- **Effort**: 3h.

### M-009 — Off-site backup retention policy not documented
- **Priority**: Medium
- **Area**: DR
- **Business impact**: OneDrive backup folder grows unbounded; one bad night = many bad nights archived.
- **Technical impact**: `C:\Users\U\OneDrive\Wamadat-Backups\` retention rules unknown.
- **Owner**: Ops.
- **Validation**: `Get-ChildItem` of backup folder ≤ documented retention count.
- **Acceptance**: Retention script keeps last N backups + monthly snapshots.
- **Effort**: 2h.

### M-010 — `LiveSessions` (100ms / Reverb) flow has no contract test
- **Priority**: Medium
- **Area**: Real-time
- **Business impact**: Live class breakage is high-visibility. No automated detection.
- **Technical impact**: `LiveSessions` module exists; no `LiveSessionTest.php` in Feature tests.
- **Owner**: Backend + QA.
- **Validation**: Pest test creates session, asserts 100ms token format, asserts presence row inserted.
- **Acceptance**: ≥1 happy-path test per live-session lifecycle event.
- **Effort**: 5h.

---

## 10. Low Findings

### L-001 — README badge "Status: Foundation" misleads at first glance
- Cosmetic, but high read-frequency. Fix in H-008.

### L-002 — `Hijri date picker` (BACKLOG L-07) not implemented
- Niche; defer to post-Beta.

### L-003 — `font-display: swap` for Tajawal (BACKLOG FE-09)
- Already done via Next/font per author note in BACKLOG, but worth verifying with a real `lighthouse` run.

### L-004 — `Plausible / PostHog` analytics not wired (BACKLOG L-10)
- Add once Beta has ≥5 users; before then it's noise.

### L-005 — `eslint-plugin-tailwindcss` version `^3.17.5` predates Tailwind 4 compatibility window
- May produce false positives. Verify on next lint run.

### L-006 — No `.editorconfig` enforcement check in CI
- `.editorconfig` exists in root; CI doesn't verify compliance.

### L-007 — `pnpm-workspace.yaml` in frontend/ but the project is not actually a workspace (just one package)
- Confusing; either remove or document why.

### L-008 — `scripts/start.bat` and `stop.bat` (untracked) likely duplicate `scripts/dev.ps1` + new ops scripts
- Decide single source of truth. Fix in H-007.

### L-009 — `frontend/pnpm-lock.yaml` modification time not verified against package.json updates
- Out-of-sync lock can produce reproducibility issues. `pnpm install --frozen-lockfile` in CI catches it.

### L-010 — No CONTRIBUTING.md guidance for AI service contributors
- Backend + frontend covered; Python contributors get no checklist.

---

## 11. Technical Debt Register

| ID | Debt | Carrying cost | Pay-off trigger |
|---|---|---|---|
| TD-01 | Queue driver = `database` (not Redis) | Higher Postgres write load; acceptable at Beta scale | First sign of `jobs` table contention or >100 jobs/min |
| TD-02 | 53 legacy Pest tests known-failing (R-OPEN-32) | Lower confidence in test signal | Closed Beta exit |
| TD-03 | PROGRESS.md retained as historical artifact | Operator confusion | Replace with STATUS.md (this audit's snapshot) |
| TD-04 | `pgvector/pgvector` excluded from package discovery manually | Hidden coupling; package-manager surprises on upgrade | Composer upgrade cycle |
| TD-05 | `databaseNotifications()` commented out in both panel providers with schema-mismatch reason | Filament UX feature deferred | Tenant `notifications` table schema reconciliation |
| TD-06 | `BroadcastService` not using batched jobs (queue `batching` table declared but unused) | Cannot atomically retry batches | When batch use case appears |
| TD-07 | No code-level enforcement of Beta caps (5 programs / 50 users) | Operator-discipline gated | First multi-tenant onboarding |
| TD-08 | Sentry Releases not auto-tagged with commit SHA in CI for backend | Errors can't be tied to specific deploy | Public Launch |
| TD-09 | `DisableBladeIconComponents` in Filament middleware — likely a perf workaround for an issue that may be fixed in newer Filament | Possible icon rendering quirks | Filament 3.x minor upgrade |

---

## 12. Security Risk Register

| ID | Risk | Severity | Mitigation in place | Residual |
|---|---|---|---|---|
| SEC-01 | Cross-tenant cache key collision | Critical | None | Open — see C-001 |
| SEC-02 | /super admin compromise via leaked creds | High | 2FA mandatory | Open — IP allowlist (H-002), session timeout (H-003) |
| SEC-03 | Signed URL replay | Medium | HMAC + 15-min expiry + per-user binding + active.account check | Acceptable |
| SEC-04 | Auth cookie XSS | Mitigated | httpOnly cookie + CP4-5 refresh + SameSite | Verify SameSite in `EncryptCookies` config |
| SEC-05 | Brute force on /api/v1/auth/* | Mitigated | Throttle middlewares declared per route | Verify throttle keys are email+IP for login |
| SEC-06 | Payment webhook spoofing | Open verification | `PaymentWebhookController` exists; signature check status not verified this audit | Audit Tap + Tamara HMAC check inside controller |
| SEC-07 | National ID disclosure | Mitigated | Encrypted at rest + HMAC hash for UNIQUE; covered by H-03 in BACKLOG | OK |
| SEC-08 | Race conditions in coupon validation | Mitigated | Reportedly fixed (commit `197052b`), Pest test ?? | Need Pest test confirming |
| SEC-09 | Mass enumeration of programs / users via API | Mitigated | Rate limits + pagination | OK |
| SEC-10 | CSRF on Filament panels | Mitigated | `VerifyCsrfToken` in panel middleware | OK |
| SEC-11 | Information disclosure via verbose errors | Mitigated | `ApiErrorRenderer` canonical envelope | Verify `APP_DEBUG=false` enforcement test |
| SEC-12 | Privilege escalation via Filament Resource policies | Open verification | Spatie permissions present; per-resource policy check not in this audit | Pest test per resource |

---

## 13. Tenant Isolation Risk Register

| ID | Vector | Status | Test exists | Action |
|---|---|---|---|---|
| TI-01 | Database schema isolation | Implemented (Spatie `SwitchTenantSchemaTask`) | Yes (`TenantIsolationTest.php` — single test) | Expand to cover models from 5+ modules |
| TI-02 | Queue job tenant context | Implemented (Spatie default ON + explicit binding in `SendOutboxEmailJob`) | No | Add cross-tenant job dispatch test |
| TI-03 | Cache key isolation | NOT IMPLEMENTED | No | C-001 + H-006 |
| TI-04 | Redis session/broadcast isolation | Default DB-index split (REDIS_*_DB different numbers) | No | Document + verify |
| TI-05 | Media file isolation | Signed URLs scoped to user; need verification path includes tenant | Partial (`MediaSignedUrlTest.php`) | Cross-tenant signed-URL test |
| TI-06 | Webhook handler tenant resolution | Implemented (cookie/default-slug fallback in `webhooks/{gateway}`) | No | Cross-tenant webhook test |
| TI-07 | Meilisearch index per tenant | Status unknown (Scout config not inspected this audit) | No | Inspect + test |
| TI-08 | Tenant context in `php artisan` commands | Implemented (Spatie tenant: commands) | No | Add test for `tenant:run --tenant=X cmd` |
| TI-09 | Tenant-aware logging | Status unknown | No | Audit Monolog processors |

---

## 14. DevOps Readiness

| Capability | Status | Notes |
|---|---|---|
| CI pipeline | ✅ Comprehensive | Pint, PHPStan, Pest --parallel --ci, vitest, build, composer audit, pnpm audit |
| Deploy pipeline | ✅ Exists | `.github/workflows/deploy.yml` for tags + manual; Sentry sourcemap upload |
| Branch protection on main | ⚠️ Not verified | Per pre-launch checklist item 8 |
| Migrations safety | ⚠️ No dry-run | Production deploys run migrations without `--pretend` check |
| Backup automation | ✅ | Daily 03:00 Task Scheduler; verified backup today 09:22 |
| Restore drill | ⚠️ Manual | `restore-drill-log.txt` shows one PASS at 2026-05-14; no weekly cadence enforced |
| Health probe | ✅ | `/up` Laravel default + `/api/v1/health` deep probe |
| Worker supervision | ✅ Adequate | PowerShell loop + Task Scheduler at-logon; H-010 caveat |
| Alerting | ⚠️ Partial | 3-channel failed-jobs alert (toast + log + OneDrive marker); Sentry DSN missing |
| Secrets management | ⚠️ Local-only | `.env` on disk; no Vault/SSM integration documented |
| Container readiness | ❌ | App not containerized; only Meilisearch + Mailpit + MinIO in docker-compose |
| Deployment topology doc | ✅ | `00-deployment-topology.md` |
| Rollback runbook | ✅ | `02-rollback.md` |
| DR plan | ⚠️ Single host | `03-db-recovery.md` exists; no second-host failover plan |

---

## 15. QA Readiness

| Layer | Coverage today | Gap |
|---|---|---|
| Backend unit tests (Pest) | 2 files | Need ≥20 to cover Value Objects + Services |
| Backend feature tests (Pest) | 15 files (~25-test contract net per Phase D memory) | Need integration tests per high-value endpoint |
| Backend arch tests | 0 | H-004 |
| Backend coverage % | Unmeasured | Need `--coverage` baseline + CI gate |
| Frontend unit tests (Vitest) | 0 files | C-002 |
| Frontend E2E tests (Playwright) | 0 | C-003 |
| Visual regression | None | Future |
| Accessibility test | None (axe-core) | FE-06 in BACKLOG |
| Load test | None | Phase E deferred |
| Mutation testing | `infection/infection` installed but not in CI | Future |
| Contract test fixture | Yes (CP4-12 25-test net) | Maintain |

---

## 16. Frontend Readiness

**Architecture**: Next.js 15 App Router, React 19, TS 5.5, Tailwind 4, shadcn/ui, next-intl 3.26, TanStack Query 5.62, Zustand 5, react-hook-form + zod, ky for API, Laravel Echo + Pusher-js for WS, Sentry/nextjs, Vercel Analytics, Speed Insights.

| Item | Status |
|---|---|
| RTL throughout | ✅ |
| ar/en i18n | ✅ (`messages/ar.json` + `en.json`) |
| Auth flows | ✅ (sign-in, sign-up, forgot, reset, verify, 2FA setup-on-dashboard) |
| 48 pages | ✅ |
| Page Builder catch-all | ✅ |
| Sentry wired | ✅ (3 config files) |
| Speed Insights | ✅ |
| Web Vitals tracking | ✅ |
| Service worker / offline route | ✅ (`/offline`) |
| Storybook / design system | ✅ (`/design-system` page) |
| Tests | ❌ (C-002, C-003) |
| A11y audit | ❌ |
| Lighthouse CI | ❌ |
| Image optimization profile | Unverified |
| Bundle analyzer in CI | ❌ |
| SEO meta on all pages | Partial — verified `[locale]/layout.tsx` injects site settings; per-page metadata not audited |

---

## 17. Backend Readiness

| Item | Status |
|---|---|
| Laravel 11.51.0 verified | ✅ |
| Filament 3.2 | ✅ |
| 25 modules (DDD-style) | ✅ |
| 75 migrations applied | ✅ |
| 47 controllers, all wired | ✅ (zero placeholder `TODO`) |
| 23 admin Filament resources | ✅ |
| 75+ API endpoints | ✅ |
| Multi-tenancy (Spatie + custom schema task) | ✅ |
| Queue-tenant-aware default ON | ✅ |
| Horizon installed | ✅ (composer); not used yet (queue=database) |
| Reverb installed | ✅ |
| Scout + Meilisearch | ✅ |
| Cashier (Stripe) | ✅ installed |
| Cookie/Bearer auth bridge | ✅ |
| Canonical API error envelope (CP4-2) | ✅ |
| Accept-Language locale (CP4-6) | ✅ |
| Pagination canonicalization (CP4-4) | ✅ |
| Signed URLs for media (CP4-11) | ✅ |
| Tenant isolation tests | ⚠️ thin (1 file) |
| Rate limiting | ✅ Per-route throttles audited (CP4-10) |
| Audit log (immutable trigger) | ✅ |
| Sentry SDK | ✅ |
| Telescope | ✅ (dev) |
| Pulse | ✅ |
| ZATCA Phase 2 | ✅ via salla/zatca |
| ID Helper | ✅ |

---

## 18. AI Readiness

**Module**: `ai-service/` (Python 3.12 + FastAPI 0.115 + Anthropic SDK 0.40+ + OpenAI 1.55+ + pgvector 0.3 + Redis 5.2 + structlog + sentry).

**Verified**: package config exists; module folder structure present; `mypy strict`, `ruff` configured, `pytest-cov` with `--cov-fail-under=80` configured in `pyproject.toml`.

**NOT verified this audit**: actual route handlers, runtime test pass, Anthropic API key wired, prompt cache discipline, fallback behavior on rate limit, vector index population.

**Backend integration**: `StudyAssistantController` exists at `me/learn/lessons/{lesson}/ask` with 20/min throttle. End-to-end path not load-tested.

**Risk**: medium — if the FastAPI service is down, every `ask` call returns 5xx. No circuit breaker visible from controller layer; needs audit.

---

## 19. Data Architecture

| Item | Status |
|---|---|
| PostgreSQL 16 verified | ✅ (`psql --version` returns 16.13) |
| Two databases (landlord + tenants) | ✅ |
| Schema-per-tenant inside tenants DB | ✅ |
| pgvector extension | ✅ (installed via composer; runtime use in AI module) |
| Migrations split into landlord/ + tenant/ folders | ✅ |
| Foreign-key enforcement | ⚠️ Selectively disabled (M-007) |
| Backup format | pg_dump custom (verified by restore drill PASS) |
| Backup offsite | OneDrive (verified — backup today 09:22) |
| Backup encryption at-rest | Inherits OneDrive's encryption; no application-layer encryption confirmed |
| Read replica | ❌ (DB-08 backlog) |
| Connection pooling | ⚠️ Default Laravel; pgbouncer not configured |
| Vacuum / autovacuum tuning | Default Postgres |
| Query plan tests | None |
| Tenant DB size monitoring | ✅ in daily summary |

---

## 20. Compliance Readiness

| Regime | Status |
|---|---|
| PDPL (Saudi) | ✅ Data export (`/me/data-export`), account delete (`/me/account` DELETE w/ PII scrub + token revoke), audit log immutable. Cookie consent banner not verified. |
| ZATCA Phase 2 | ✅ `salla/zatca` package in composer; QR + e-invoice flow exists per migrations (`invoices` table with ZATCA fields). Live submission not verified. |
| GDPR | Indirect — PDPL covers same surface |
| SOC 2 | ❌ Not in scope |
| ISO 27001 | ❌ Not in scope |
| Accessibility (WCAG 2.1 AA) | Unaudited — `axe-core` not in CI |
| Cookie banner / consent | Not verified |
| DPA (Data Processing Agreement) template | Not in repo |
| Breach notification workflow | ⚠️ Listed in BACKLOG (PHASE 12 dependency) |
| 30-day grace period hard-delete cron | ⚠️ Listed in BACKLOG (PHASE 12 dependency) |
| Right to erasure full chain | ✅ |
| Saudi National ID encryption + HMAC | ✅ |

---

## 21. Commercial SaaS Readiness

| Capability | Status |
|---|---|
| Pricing page | ✅ `/pricing` |
| Subscription billing | ⚠️ Cashier installed; integration depth not verified |
| Tap / Tamara / Mada flows | ✅ Per controllers + Phase D fixes |
| Customer success tooling | ⚠️ Support module exists; pipelines not validated |
| Email outbox + drain pattern | ✅ |
| Marketing site | ✅ (Page Builder lets operator own content) |
| Affiliate program | ✅ controllers + DB; flow not deeply tested |
| Coupons (5 scope levels) | ✅ |
| Refunds | ⚠️ schema present; flow not verified this audit |
| Gift programs | ✅ |
| Newsletter | ✅ |
| Wishlist | ✅ |
| Live sessions | ✅ controllers + 100ms integration; no E2E test |
| Certificates (verifiable) | ✅ |
| Onboarding flow for new tenants | ❌ (no automated tenant provisioning UX) |
| Self-serve account creation per tenant | ❌ |
| Status page | ❌ |
| Trial → paid conversion flow | ❌ |
| Public roadmap | ❌ |
| Public changelog | CHANGELOG.md exists but not visibly updated since project start |

---

## 22. Required Fixes Before Closed Beta

> Closed Beta = invite 5–10 users, observe for 30 days, drift towards open cohort if green.

1. **C-001** Cache tenant isolation (3–4h). One Pest test proving non-leak.
2. **C-004** Sentry DSN (30 min).
3. **C-005** Daily-summary log scan covers rotated files (1h).
4. **C-003** Three Playwright smoke specs minimum: ar home, sign-in, simulated checkout (1 day).
5. **H-007** Commit/clean uncommitted MediaAssetResource + 4 untracked scripts (30 min).
6. **H-008** Update README + STATUS (replacing PROGRESS) + BACKLOG (3h).
7. **M-001** Decide phase-3 branch fate (30 min).

**Time-boxed**: 2.5 dev-days of focused work.

Items **not** required for Closed Beta: PHPStan baseline bump, arch tests, IP allowlist for /super (defer to public launch when traffic justifies), Better Stack, multi-tenant cache (only one tenant exists).

---

## 23. Required Fixes Before Public Launch

> Public Launch = open registration, multiple tenants, paid traffic, no per-user hand-holding.

In addition to all Beta items:

8. **C-001 expanded** — Multi-tenant cache test with 3+ simultaneous tenants.
9. **H-001 → H-006** All High findings closed.
10. **TI-02, TI-05, TI-06, TI-07** Tenant isolation test surface complete.
11. **CI/CD**: composer audit + pnpm audit fail on high (M-003), migration `--pretend` dry-run job in deploy.yml, Playwright in CI on PR.
12. **Phase E** Real-traffic load test baseline measured.
13. **Better Stack** uptime monitor on FE + BE + AI service.
14. **Subdomain DNS + wildcard cert** provisioned and tested.
15. **Tenant onboarding wizard** (or operator-runnable command) replaces the manual seeder flow.
16. **Beta caps lifted with code-level guard** (H-009).
17. **Backend coverage** ≥70%; critical flows ≥90%.
18. **Frontend coverage** ≥50% statements, ≥40% branches.
19. **Status page** + public incident comms plan.
20. **Migration to non-dev-machine topology** OR explicit acceptance of single-machine SLA.

**Time-boxed**: 4–8 weeks of focused work with a small team.

---

## 24. Required Fixes Before Enterprise Customers

In addition to all Public Launch items:

21. **SSO / SAML / OIDC** integration (Microsoft Entra, Okta, Google Workspace).
22. **Customer-facing audit log export** (CSV / JSON / SCIM events).
23. **Dedicated DB option** for enterprise tier (per-tenant cluster).
24. **Multi-region failover** runbook + tested.
25. **SOC 2 Type II or equivalent** in progress with named auditor.
26. **Signed DPA** template ready.
27. **Customer-supplied IP allowlist** per tenant.
28. **Per-tenant data residency** controls.
29. **Tenant-scoped reporting API** with SLA on freshness.
30. **24/7 incident response runbook** with named on-call.
31. **Pen-test report** within 12 months, with critical findings closed.
32. **Insurance**: cyber liability policy.

**Time-boxed**: 6–12 months with a dedicated security + compliance squad.

---

## 25. Final Verdict

### Closed Beta

**Status**: **READY with 2.5 days of focused work**.
**Reason**: All structural pieces exist. The launch blocker (program cap) is already closed (`5/5` per today's summary). The four items left (cache isolation, Sentry DSN, log scan robustness, 3 smoke tests) are scoped and small. The bigger blocker is **operator action**, not engineering work — the platform has been Beta-ready and waiting since 2026-05-14.

### Public Launch

**Status**: **NOT READY**.
**Reason**: Five hard gates open — multi-tenant cache (critical), Phase E real-traffic baseline (deferred by design), full E2E suite, subdomain DNS provisioning, code-level Beta-cap removal. Realistic timeline 4–8 weeks of focused team work.

### Enterprise SaaS

**Status**: **NOT READY**.
**Reason**: SSO, customer-facing audit, dedicated-DB tier, multi-region, signed compliance package all absent. Realistic timeline 6–12 months with a security + compliance squad.

### Top-Tier Global LMS SaaS

**Status**: **NOT READY**.
**Reason**: All Enterprise gaps plus: native mobile apps, AI-driven personalization measured against industry benchmarks, marketplace, advanced reporting, i18n beyond ar/en. This is a 12–24 month roadmap from current state.

---

### Final auditor's note

The platform is **significantly better engineered than the prompt's stated assumptions**. Half the "known serious gaps" listed in the trigger prompt are already closed in code — they look open only because BACKLOG.md was last edited 2026-05-12 (24 days ago) and predates Phase D + Operational Hardening Sprint + No-Code Initiative.

The real risks are concentrated in **three places**: cache isolation, frontend test absence, and stale documentation. Close those plus wire Sentry DSN and the platform is genuinely Beta-launch-ready — pending operator decision to actually invite users.
