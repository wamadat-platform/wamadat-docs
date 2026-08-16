# Executive Transformation Backlog — Wamadat Academy

> **Source of truth**: `docs/executive-transformation-audit.md` (2026-06-08). Every item here traces back to a finding (C-, H-, M-, L-) or a track-level capability gap.
> **Format**: Twelve tracks. Each item has: epic, user story, technical task, priority, owner role, files likely affected, dependencies, validation command, acceptance criteria, definition of done.
> **Priority scale**: P0 (launch blocker), P1 (do this sprint), P2 (do this quarter), P3 (do when load justifies).

---

## Track 1 — Security Hardening

### 1.1 — Enforce mandatory 2FA verified by Pest (closes H-001)
- **Epic**: Admin panel hardening
- **Story**: As a CISO, I need automated proof that a system_user without a confirmed 2FA secret cannot access any /super route, so an audit can rely on the control existing.
- **Technical task**: Read `App\Modules\Tenancy\Infrastructure\Http\Middleware\RequireTwoFactor`. Confirm it rejects (not redirects to setup) when secret is `NULL`. Add Pest test covering both the rejection path and the success path.
- **Priority**: P1
- **Owner**: Backend security
- **Files**: `backend/app/Modules/Tenancy/Infrastructure/Http/Middleware/RequireTwoFactor.php`, `backend/tests/Feature/Admin/SuperPanelTwoFactorTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=SuperPanelTwoFactor`
- **Acceptance**: Two assertions — user without secret → 403/redirect; user with active secret → 200 to /super dashboard.
- **DoD**: Test green in CI; finding H-001 closed in audit doc; commit message references H-001.

### 1.2 — IP allowlist middleware for /super (closes H-002)
- **Epic**: Admin panel hardening
- **Story**: As an operator, I want /super reachable only from my documented IPs so a leaked credential pair (password + 2FA) cannot be used remotely.
- **Technical task**: Create `Modules\Operations\Infrastructure\Http\Middleware\AdminIpAllowlist`. Read CIDRs from `config('security.admin_allowlist')` (new). If allowlist empty → middleware no-ops with a `Log::warning`. Register in `SuperPanelProvider::middleware()` ahead of `Authenticate`.
- **Priority**: P1 (Public Launch blocker, P2 for Beta)
- **Owner**: Backend security + Ops
- **Files**: `backend/app/Modules/Operations/Infrastructure/Http/Middleware/AdminIpAllowlist.php` (new), `backend/config/security.php` (new), `backend/app/Providers/Filament/SuperPanelProvider.php`, `backend/tests/Feature/Admin/AdminIpAllowlistTest.php` (new)
- **Dependencies**: Operator provides initial CIDR list
- **Validation command**: `vendor\bin\pest --filter=AdminIpAllowlist`
- **Acceptance**: Request from CIDR member → continues; non-member → 403 with code `IP_NOT_ALLOWED`.
- **DoD**: Test green; `config/security.php` documents env var; H-002 closed.

### 1.3 — /super idle session timeout ≤30 min (closes H-003)
- **Epic**: Admin panel hardening
- **Story**: As a CISO, I need /super to log out idle admins after 30 minutes independent of the global session lifetime, so an unattended browser is not a persistent attack surface.
- **Technical task**: Track `last_admin_activity_at` on the session, expire on next request if delta > 30 min, require re-auth. Use a Filament panel middleware.
- **Priority**: P1
- **Owner**: Backend security
- **Files**: `backend/app/Modules/IdentityAccess/Infrastructure/Http/Middleware/AdminIdleTimeout.php` (new), `backend/app/Providers/Filament/SuperPanelProvider.php`, `backend/app/Providers/Filament/AdminPanelProvider.php`, `backend/tests/Feature/Admin/IdleTimeoutTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=IdleTimeout`
- **Acceptance**: Travel time forward 31 min → /super request → 401 + redirect to login.
- **DoD**: Test green; H-003 closed.

### 1.4 — Payment webhook signature verification audit (verifies SEC-06)
- **Epic**: Payment integrity
- **Story**: As a CTO, I need confidence that Tap + Tamara webhooks reject any unsigned/forged payload, because a successful spoof grants free programs.
- **Technical task**: Read `Modules\Commerce\Infrastructure\Http\Controllers\PaymentWebhookController` + gateway adapter classes; verify HMAC check is the first step of `handle()`. Add Pest test with bad signature.
- **Priority**: P1
- **Owner**: Backend security
- **Files**: `backend/app/Modules/Commerce/Infrastructure/Http/Controllers/PaymentWebhookController.php`, gateway adapters under `Modules\Commerce\Infrastructure\PaymentGateways\*`, `backend/tests/Feature/Webhooks/PaymentWebhookSignatureTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=PaymentWebhookSignature`
- **Acceptance**: Webhook with tampered signature → 401, no order state change. Valid signature → 200, expected side effects.
- **DoD**: Test green for each enabled gateway (mock/tap/tamara); SEC-06 marked verified.

### 1.5 — Audit log on `programs.status` transitions (closes M-002)
- **Epic**: Compliance audit
- **Story**: As a compliance officer, I need every publish/unpublish action attributable to a user with before/after, so retroactive review is possible.
- **Technical task**: Register `Spatie\Activitylog\LogsActivity` on `ProgramModel`; restrict logged columns to `status, published_at, updated_by`; verify the activity log table is tenant-scoped.
- **Priority**: P2
- **Owner**: Backend
- **Files**: `backend/app/Modules/Catalog/Infrastructure/Eloquent/ProgramModel.php`, `backend/tests/Feature/Audit/ProgramTransitionAuditTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=ProgramTransitionAudit`
- **Acceptance**: Publish then unpublish → 2 entries in `activity_log` with actor + old/new.
- **DoD**: Test green; M-002 closed.

---

## Track 2 — Tenant Isolation

### 2.1 — Tenant-aware cache prefix (closes C-001)
- **Epic**: Multi-tenant data integrity
- **Story**: As a tenant, I need certainty that another tenant cannot read my cached homepage/dashboard/program lists.
- **Technical task**: Create `backend/config/cache.php` (publish Laravel's default + customize). Register a custom CacheStore that prefixes keys with `current_tenant()?->slug . ':'`. Apply on the default store. Document in `02-architecture.md`.
- **Priority**: **P0** (multi-tenant safety)
- **Owner**: Backend lead
- **Files**: `backend/config/cache.php` (new), `backend/app/Modules/Tenancy/Infrastructure/Cache/TenantAwareCacheManager.php` (new), `backend/app/Providers/AppServiceProvider.php`, `backend/tests/Feature/Tenant/CacheIsolationTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=CacheIsolation`
- **Acceptance**: Test that sets `foo` under tenant A, asserts cache MISS for `foo` under tenant B. Passes deterministically.
- **DoD**: Test green; C-001 closed; relevant ADR added.

### 2.2 — Cross-tenant queue job dispatch test (closes TI-02 visibility)
- **Epic**: Multi-tenant data integrity
- **Story**: As a backend lead, I need automated proof that a job dispatched in tenant A's context executes in tenant A's schema even after re-queue.
- **Technical task**: Pest test that dispatches `SendOutboxEmailJob` under tenant A (with an outbox row in tenant A schema), runs the worker, asserts the row is updated in tenant A and that NO write happened in tenant B's empty schema.
- **Priority**: P1
- **Owner**: Backend
- **Files**: `backend/tests/Feature/Tenant/QueueJobTenantContextTest.php` (new)
- **Dependencies**: 2-tenant Pest fixture (factory)
- **Validation command**: `vendor\bin\pest --filter=QueueJobTenantContext`
- **Acceptance**: Test asserts payload survives serialize/re-queue/retry, lands in correct tenant.
- **DoD**: Test green; TI-02 has automated coverage.

### 2.3 — Cross-tenant signed-URL rejection test (closes TI-05 verification)
- **Epic**: Multi-tenant data integrity
- **Story**: As a CISO, I need proof that a signed lesson-resource URL minted for tenant A cannot be redeemed under tenant B's hostname/header.
- **Technical task**: Pest test: enroll user in tenant A; mint signed URL via `MediaController`; replay against tenant B context; assert 404 or 403.
- **Priority**: P1
- **Owner**: Backend security
- **Files**: `backend/tests/Feature/Tenant/SignedUrlCrossTenantTest.php` (new), reuses helpers from `tests/Feature/Tenant/MediaSignedUrlTest.php`
- **Dependencies**: 2-tenant Pest fixture
- **Validation command**: `vendor\bin\pest --filter=SignedUrlCrossTenant`
- **Acceptance**: Cross-tenant replay returns canonical error envelope, no file bytes.
- **DoD**: Test green; TI-05 hardened.

### 2.4 — Cross-tenant API request rejection test (TI-01 coverage expansion)
- **Epic**: Multi-tenant data integrity
- **Story**: As a CTO, I need each module's controllers covered by an isolation test, not just one example.
- **Technical task**: Parameterized Pest test that hits 6 high-value endpoints under tenant B with a tenant A bearer token / cookie. Each call must return 401/403 or 404.
- **Priority**: P1
- **Owner**: Backend
- **Files**: `backend/tests/Feature/Tenant/CrossTenantApiAccessTest.php` (new)
- **Dependencies**: 2-tenant fixture
- **Validation command**: `vendor\bin\pest --filter=CrossTenantApiAccess`
- **Acceptance**: 6 endpoints × deny → 6 green assertions.
- **DoD**: Test green; TI-01 coverage no longer single-file.

---

## Track 3 — Quality Engineering

### 3.1 — Frontend Vitest critical-paths suite (closes C-002)
- **Epic**: Frontend safety net
- **Story**: As a frontend lead, I need a green Vitest suite that covers auth, cart, API client, and i18n routing so refactors don't ship regressions.
- **Technical task**: Write tests covering: auth store login/logout/refresh side effects; cart store add/remove/coupon; ky API client header injection (auth + tenant); next-intl routing localization helpers; form validation for sign-up + checkout.
- **Priority**: P1
- **Owner**: Frontend lead
- **Files**: `frontend/tests/stores/auth.test.ts`, `frontend/tests/stores/cart.test.ts`, `frontend/tests/lib/api.test.ts`, `frontend/tests/lib/i18n.test.ts`, `frontend/tests/components/sign-up-form.test.tsx`, `frontend/tests/components/checkout-form.test.tsx` (all new)
- **Dependencies**: None
- **Validation command**: `pnpm test`
- **Acceptance**: ≥10 tests green. Coverage ≥50% statements, ≥40% branches on covered files. CI passes on real tests, not 0.
- **DoD**: Tests green; coverage report committed; C-002 closed.

### 3.2 — Playwright smoke suite (closes C-003 for Beta)
- **Epic**: E2E safety net
- **Story**: As a CTO, I need the catalog → simulated checkout → enrollment golden path verified after every deploy so revenue-bearing flow can't ship broken.
- **Technical task**: Create `playwright.config.ts` targeting `http://localhost:3000`. Add `pnpm e2e` + `pnpm e2e:ui` scripts. Specs: home (ar) loads with site settings, sign-in, browse program, simulate checkout (env-flag mock gateway), view dashboard. Use deterministic seed data via API setup hooks.
- **Priority**: P0 for Beta launch
- **Owner**: QA lead
- **Files**: `frontend/playwright.config.ts` (new), `frontend/tests/e2e/home.spec.ts`, `frontend/tests/e2e/sign-in.spec.ts`, `frontend/tests/e2e/catalog-to-enrollment.spec.ts`, `frontend/tests/e2e/_fixtures.ts`
- **Dependencies**: BE + FE local stack reachable; deterministic seed user with known password
- **Validation command**: `pnpm e2e`
- **Acceptance**: 3 specs green on 10 consecutive local runs; flake <5%.
- **DoD**: Specs green; CI workflow draft added (gated, opt-in for first 30 days).

### 3.3 — Pest architecture tests (closes H-004)
- **Epic**: Backend architectural integrity
- **Story**: As an architect, I need automated detection of cross-module imports that violate the bounded-context contract.
- **Technical task**: Write 6 arch rules: (1) each module's `Domain/` cannot import any `Infrastructure/`; (2) `Application/` cannot reach into another module's `Application/`; (3) `Http/Controllers/` may only inject `Application/`; (4) `Eloquent/` lives only under `Infrastructure/Eloquent/`; (5) value objects are `final`; (6) `Modules\X` cannot import `Modules\Y\Domain\*` (Domain types are private).
- **Priority**: P1
- **Owner**: Backend architect
- **Files**: `backend/tests/Arch/ModuleBoundariesTest.php` (new), `backend/tests/Arch/ValueObjectsTest.php` (new)
- **Dependencies**: None (`pest-plugin-arch` already in composer)
- **Validation command**: `vendor\bin\pest --filter=Arch`
- **Acceptance**: ≥6 rules pass; any violation blocks PR.
- **DoD**: Tests green; H-004 closed.

### 3.4 — PHPStan level baseline lock (closes H-005)
- **Epic**: Backend type safety
- **Story**: As a backend lead, I need to know the current PHPStan level and lock it to prevent regression.
- **Technical task**: Run `vendor\bin\phpstan analyse` at successive levels (5 → 6 → 7 → 8) until first error. Generate baseline file at highest passing level. Declare in `phpstan.neon`. Document remaining baseline items.
- **Priority**: P1
- **Owner**: Backend
- **Files**: `backend/phpstan.neon`, `backend/phpstan-baseline.neon` (new), `backend/docs/quality.md` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\phpstan analyse --memory-limit=2G`
- **Acceptance**: Clean run at declared level; baseline file accepted; CI step uses declared level.
- **DoD**: Lock committed; H-005 closed.

---

## Track 4 — DevOps and Observability

### 4.1 — Wire Sentry DSN (closes C-004)
- **Epic**: Error observability
- **Story**: As an on-call operator, I want every error in BE + FE surfaced in a single dashboard so the daily summary stops being the only signal.
- **Technical task**: Create Sentry projects (BE + FE). Populate `SENTRY_LARAVEL_DSN` + `NEXT_PUBLIC_SENTRY_DSN` + `SENTRY_RELEASE` (from `git rev-parse --short HEAD` in deploy). Restart workers. Validate with `php artisan sentry:test` (existing `SentryTestCommand`).
- **Priority**: P0 for Beta launch
- **Owner**: Ops
- **Files**: `backend/.env` (not committed), `frontend/.env.local` (not committed), `docs/operations/observability.md` (new)
- **Dependencies**: Sentry account (free tier OK at Beta scale)
- **Validation command**: `php artisan sentry:test`
- **Acceptance**: Test event appears in Sentry within 60s; both projects healthy.
- **DoD**: DSNs set in all environments (local, CI optional, prod); C-004 closed.

### 4.2 — Daily summary covers rotated logs (closes C-005)
- **Epic**: Monitoring robustness
- **Story**: As an operator, I want the morning summary to count errors across all log files from the last 24h, not just `laravel.log`.
- **Technical task**: Update `daily-summary.ps1` to scan `Get-ChildItem -Path "$LogsDir\laravel*.log" | Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-24) }`. Sum `ERROR` matches across all matching files.
- **Priority**: P0 for Beta launch
- **Owner**: Ops
- **Files**: `scripts/operations/daily-summary.ps1`
- **Dependencies**: None
- **Validation command**: Inject test error into `laravel-yesterday.log`; rerun summary; verify count > 0.
- **Acceptance**: Errors counted across all rotated files modified in last 24h.
- **DoD**: Script updated; C-005 closed; one validation run shows correct count.

### 4.3 — Sentry release auto-tagging in CI deploy (closes TD-08)
- **Epic**: Release tracking
- **Story**: As an on-call operator, I want every error tagged with the git SHA that introduced it.
- **Technical task**: Update `.github/workflows/deploy.yml` to compute `SENTRY_RELEASE=$(git rev-parse --short HEAD)` and pass to both BE + FE build/run steps.
- **Priority**: P2 (until Public Launch)
- **Owner**: DevOps
- **Files**: `.github/workflows/deploy.yml`
- **Dependencies**: 4.1
- **Validation command**: Inspect new deploy run; verify Sentry release is created.
- **Acceptance**: Sentry shows new release on each deploy, tied to commit SHA.
- **DoD**: TD-08 closed.

### 4.4 — Better Stack uptime for BE + FE + AI service
- **Epic**: External uptime signal
- **Story**: As a CTO, I need independent confirmation that the platform is reachable, because internal health probes don't fire if the host is down.
- **Technical task**: Set up 3 Better Stack heartbeat monitors hitting `/up`, `/`, `:8001/health`. Alert via email + SMS at 2 consecutive failures.
- **Priority**: P1 for Public Launch
- **Owner**: Ops
- **Files**: `docs/operations/observability.md`
- **Dependencies**: External account
- **Validation command**: Stop one service temporarily; verify alert fires.
- **Acceptance**: All 3 endpoints monitored; alert chain validated.
- **DoD**: Account live; runbook updated.

---

## Track 5 — Multi-Tenancy Production Readiness

### 5.1 — Subdomain DNS + wildcard cert provisioning runbook
- **Epic**: Public-launch tenant onboarding
- **Story**: As a SaaS operator, I need `<tenant>.wamadat.academy` to resolve and serve TLS so the `SubdomainTenantFinder` (already implemented) can do its job in production.
- **Technical task**: Document DNS wildcard `*.wamadat.academy → app IP`, Let's Encrypt wildcard via DNS-01 challenge, automated renewal cron. Add to runbook.
- **Priority**: P1 for Public Launch
- **Owner**: DevOps
- **Files**: `docs/operations/07-subdomain-onboarding.md` (new)
- **Dependencies**: Domain control; ACME-capable cert tool
- **Validation command**: `curl -k https://test-tenant.wamadat.academy/api/v1/health`
- **Acceptance**: Random new subdomain resolves + serves TLS within 5 min of tenant creation.
- **DoD**: Runbook published; first non-flagship tenant onboarded as proof.

### 5.2 — Code-level Beta cap enforcement (closes H-009)
- **Epic**: Multi-tenant safety
- **Story**: As an operator, I need publishing a 6th program to be rejected with a clear error, not relying on me reading tomorrow's summary.
- **Technical task**: Add `PublishProgramService::guardCap()` reading `config('beta.published_program_cap', 5)`. Throw `BetaCapExceededException` → translated to canonical envelope with `code=BETA_CAP_EXCEEDED`.
- **Priority**: P1 for Public Launch (P2 for Beta where one operator is the cap)
- **Owner**: Backend
- **Files**: `backend/app/Modules/Catalog/Application/Services/PublishProgramService.php`, `backend/config/beta.php` (new), `backend/tests/Feature/Catalog/BetaCapTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=BetaCap`
- **Acceptance**: 6th publish attempt → 422 with code `BETA_CAP_EXCEEDED`; 5th succeeds.
- **DoD**: Test green; H-009 closed.

### 5.3 — Tenant onboarding command (operator-runnable)
- **Epic**: Public-launch tenant onboarding
- **Story**: As an operator, I want to create a new tenant + run migrations + seed default plan + create owner user with one command, so onboarding doesn't require manual seeder edits.
- **Technical task**: Wrap existing tenant creation into `php artisan tenant:onboard {slug} {owner-email}` console command. Idempotent. Outputs login link.
- **Priority**: P1 for Public Launch
- **Owner**: Backend
- **Files**: `backend/app/Modules/Tenancy/Infrastructure/Console/OnboardTenantCommand.php` (new), `backend/tests/Feature/Console/OnboardTenantTest.php` (new)
- **Dependencies**: None
- **Validation command**: `php artisan tenant:onboard demo demo@example.com` → DB has `tenant_demo` schema with applied migrations + owner user.
- **Acceptance**: Re-running with same args is a no-op (idempotent).
- **DoD**: Test green; runbook updated.

### 5.4 — Meilisearch index-per-tenant verification (closes TI-07)
- **Epic**: Multi-tenant data integrity
- **Story**: As a CTO, I need certainty that search on tenant A does not return tenant B documents.
- **Technical task**: Inspect Scout config for index naming. If shared index, switch to `wamadat_{slug}_programs` index per tenant. Add Pest test seeding documents in 2 tenants, searching from each, asserting non-leak.
- **Priority**: P0 for Public Launch (no risk at single-tenant Beta)
- **Owner**: Backend
- **Files**: `backend/config/scout.php` (likely modify), `backend/app/Modules/Search/...`, `backend/tests/Feature/Tenant/SearchIsolationTest.php` (new)
- **Dependencies**: Local Meilisearch running
- **Validation command**: `vendor\bin\pest --filter=SearchIsolation`
- **Acceptance**: Each tenant searches own index only.
- **DoD**: Test green; TI-07 closed.

---

## Track 6 — Frontend Stability

### 6.1 — Accessibility audit + axe-core in CI (BACKLOG FE-06)
- **Epic**: Inclusive product
- **Story**: As a Saudi user with assistive tech, I expect dashboard + checkout to be operable with keyboard + screen reader.
- **Technical task**: Add `@axe-core/playwright` to Playwright specs. Run accessibility scan on home, sign-in, catalog, program detail, checkout, dashboard. Fail on critical violations.
- **Priority**: P1 for Public Launch
- **Owner**: Frontend + QA
- **Files**: `frontend/tests/e2e/a11y.spec.ts` (new), `frontend/package.json`
- **Dependencies**: 3.2
- **Validation command**: `pnpm e2e -- --grep a11y`
- **Acceptance**: 6 critical pages: 0 critical/serious axe violations.
- **DoD**: a11y spec green; FE-06 closed.

### 6.2 — Analytics (Plausible) wired with consent flow (BACKLOG L-10)
- **Epic**: Product analytics
- **Story**: As a product lead, I need page-view + conversion funnel data so Beta retention is measurable.
- **Technical task**: Add Plausible (or PostHog) script. Wrap in consent banner (PDPL compliance). Track 5 critical events: program_view, add_to_cart, checkout_start, checkout_complete, lesson_complete.
- **Priority**: P1 for Beta exit
- **Owner**: Frontend + Marketing
- **Files**: `frontend/app/[locale]/layout.tsx`, `frontend/components/analytics/*` (new), `frontend/lib/analytics.ts` (new)
- **Dependencies**: Plausible account
- **Validation command**: Browser DevTools network — events firing as expected.
- **Acceptance**: 5 events visible in Plausible within 1h of deploy.
- **DoD**: L-10 closed; consent banner shows once per browser.

### 6.3 — Bundle analyzer in CI
- **Epic**: Frontend performance
- **Story**: As a frontend lead, I want PR comments showing bundle delta to catch accidental ballooning.
- **Technical task**: Add `next-bundle-analyzer` + GitHub Action that comments on PRs.
- **Priority**: P2
- **Owner**: Frontend
- **Files**: `frontend/next.config.ts`, `.github/workflows/bundle-size.yml` (new)
- **Dependencies**: None
- **Validation command**: PR shows comment with bundle stats.
- **Acceptance**: PR comment visible; threshold for warning configurable.
- **DoD**: First PR with measurable change shows correct comment.

---

## Track 7 — Backend Stability

### 7.1 — Soft-delete partial indexes (BACKLOG DB-05)
- **Epic**: Backend performance
- **Story**: As a backend lead, I want active-row queries fast even as soft-deleted rows accumulate.
- **Technical task**: For each table with `deleted_at`, add `CREATE INDEX ... WHERE deleted_at IS NULL` on common filter columns.
- **Priority**: P2
- **Owner**: Backend
- **Files**: `backend/database/migrations/tenant/2026_06_*_add_soft_delete_partial_indexes.php` (new)
- **Dependencies**: None
- **Validation command**: `EXPLAIN ANALYZE` before/after on `programs` listing query
- **Acceptance**: New plan uses partial index.
- **DoD**: Migration runs in dev + CI; DB-05 closed.

### 7.2 — Soften `audit_logs_central_immutable` trigger (BACKLOG DB-03)
- **Epic**: Compliance + tenant deletion ergonomics
- **Story**: As an operator, I need to be able to fully delete a tenant for PDPL erasure without trigger gymnastics.
- **Technical task**: Replace `BEFORE DELETE` trigger with app-layer guard. Verify activity log integrity through hash chain alternative (or accept trade-off with deletion log).
- **Priority**: P2
- **Owner**: Backend + Compliance
- **Files**: `backend/database/migrations/landlord/2026_06_*_soften_audit_immutable_trigger.php` (new), `backend/app/Modules/Tenancy/Application/Services/TenantDeletionService.php`
- **Dependencies**: M-002 (audit logging for app-layer guard)
- **Validation command**: `vendor\bin\pest --filter=TenantDeletion`
- **Acceptance**: Tenant deletion succeeds; audit log retains historical entries.
- **DoD**: DB-03 closed.

### 7.3 — Live session contract test (closes M-010)
- **Epic**: Real-time integrity
- **Story**: As a QA lead, I want a green test exercising the 100ms token + presence path so a broken live class is caught pre-deploy.
- **Technical task**: Pest test mocks the 100ms HTTP client, creates a live session, asserts token payload shape, asserts presence row insert, asserts QR/attendance flow.
- **Priority**: P1
- **Owner**: Backend + QA
- **Files**: `backend/tests/Feature/LiveSessions/LiveSessionFlowTest.php` (new)
- **Dependencies**: None (mock HTTP)
- **Validation command**: `vendor\bin\pest --filter=LiveSessionFlow`
- **Acceptance**: Test green; covers happy path + 100ms 500 error fallback.
- **DoD**: M-010 closed.

---

## Track 8 — Data and Analytics

### 8.1 — Backup retention policy (closes M-009)
- **Epic**: Data safety
- **Story**: As an operator, I want unbounded OneDrive growth limited and weekly snapshots retained for 1 year.
- **Technical task**: PowerShell script: keep last 14 daily + last 8 weekly (Sunday) + last 12 monthly (1st of month). Delete the rest. Schedule weekly.
- **Priority**: P2
- **Owner**: Ops
- **Files**: `scripts/operations/backup-retention.ps1` (new), Task Scheduler entry
- **Dependencies**: None
- **Validation command**: Dry-run script reports planned deletes
- **Acceptance**: Backup folder respects retention plan after one run.
- **DoD**: M-009 closed.

### 8.2 — Read replica plan (BACKLOG DB-08)
- **Epic**: Performance + DR
- **Story**: As a CTO, I want analytics queries to run against a replica so production write path isn't impacted.
- **Technical task**: Document the deploy topology change (deferred until VPS migration). Add `read` connection to `database.php` config skeleton (gated on `DB_READ_HOST`).
- **Priority**: P3
- **Owner**: DBA + Backend
- **Files**: `backend/config/database.php`, `docs/operations/07-read-replica-plan.md` (new)
- **Dependencies**: Hosting decision
- **Validation command**: N/A until host change
- **Acceptance**: Plan committed; config supports the toggle.
- **DoD**: DB-08 partially closed (plan committed).

### 8.3 — Tenant-scoped reporting endpoint
- **Epic**: Customer-facing analytics
- **Story**: As a tenant academy owner, I need a JSON endpoint listing daily new enrollments, revenue, refunds for any 30-day window, so I can build my own dashboard.
- **Technical task**: `GET /api/v1/tenant/reports/summary?from=&to=` returning aggregates from existing `enrollments`/`orders` tables. Auth required, owner role only.
- **Priority**: P1 for Public Launch
- **Owner**: Backend
- **Files**: `backend/app/Modules/Analytics/Infrastructure/Http/Controllers/TenantReportController.php` (new), `backend/routes/api.php`, `backend/tests/Feature/Analytics/TenantReportTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=TenantReport`
- **Acceptance**: Endpoint returns canonical envelope; non-owner gets 403.
- **DoD**: Endpoint live + tested.

---

## Track 9 — Compliance

### 9.1 — Cookie consent banner with PDPL-compliant copy
- **Epic**: Compliance
- **Story**: As a Saudi visitor, I need an explicit consent prompt before non-essential cookies/analytics load.
- **Technical task**: Add consent banner (Arabic-first), gate analytics + non-essential cookies on consent state.
- **Priority**: P1 for Public Launch
- **Owner**: Frontend + Compliance
- **Files**: `frontend/components/consent/*` (new), `frontend/app/[locale]/layout.tsx`
- **Dependencies**: 6.2 (analytics)
- **Validation command**: Manual — first visit shows banner; consent persists per browser.
- **Acceptance**: Plausible/PostHog only fires post-consent.
- **DoD**: Banner copy reviewed by legal/compliance.

### 9.2 — 30-day grace-period hard-delete cron (BACKLOG PHASE 12)
- **Epic**: PDPL right to erasure
- **Story**: As a deleted user, my data should be hard-removed 30 days after my account is soft-deleted, per PDPL.
- **Technical task**: Console command `php artisan accounts:hard-purge --since=30d`. Scheduled daily. Removes PII + cascades.
- **Priority**: P1 for Public Launch
- **Owner**: Backend + Compliance
- **Files**: `backend/app/Modules/IdentityAccess/Infrastructure/Console/HardPurgeCommand.php` (new), `backend/routes/console.php`, `backend/tests/Feature/Console/HardPurgeTest.php` (new)
- **Dependencies**: None
- **Validation command**: `vendor\bin\pest --filter=HardPurge`
- **Acceptance**: 31-day-old soft-deleted user → hard-removed; 29-day-old preserved.
- **DoD**: Command + schedule wired; test green.

### 9.3 — DPA template + breach notification workflow
- **Epic**: Compliance
- **Story**: As a legal officer, I need standard DPA + breach response runbook documented.
- **Technical task**: Add `docs/compliance/dpa-template.md` + `docs/operations/07-breach-response.md`.
- **Priority**: P2 for Public Launch; P1 for Enterprise
- **Owner**: Legal + Ops
- **Files**: docs only
- **Validation**: Legal review
- **Acceptance**: Templates filed.
- **DoD**: Docs committed.

---

## Track 10 — Documentation Cleanup

### 10.1 — Rewrite README + replace PROGRESS.md with STATUS.md (closes H-008)
- **Epic**: Truthful first impression
- **Story**: As a new contributor, I need the root README to describe the platform's actual state (Closed-Beta-ready) instead of "Foundation".
- **Technical task**: Rewrite README ditching the `Status: Foundation` badge, replace with `Status: Closed-Beta-ready`. Replace PROGRESS.md with STATUS.md (this audit's summary section + last 10 commits). Trim BACKLOG.md to only open items.
- **Priority**: P0 for Beta launch
- **Owner**: Tech writer / PMO
- **Files**: `README.md`, `PROGRESS.md` (delete), `STATUS.md` (new), `BACKLOG.md`
- **Dependencies**: This audit
- **Validation**: Reviewer reads STATUS.md + README and accurately describes project state
- **Acceptance**: No claims contradicted by code.
- **DoD**: H-008 closed.

### 10.2 — Onboarding doc for new developers
- **Epic**: Contributor velocity
- **Story**: As a new developer joining, I need a step-by-step setup that works end-to-end on Windows.
- **Technical task**: `docs/contributing/01-developer-setup.md` covering PHP via Laragon, Composer (separate install), pnpm, Postgres 16, Meilisearch via Docker, env files, first migration, first test run.
- **Priority**: P2
- **Owner**: Tech writer
- **Files**: docs only
- **Dependencies**: None
- **Validation**: Run all commands on a clean checkout.
- **Acceptance**: New dev reaches `pnpm dev` + `php artisan serve` green within 90 min.
- **DoD**: Doc committed; one dev does the run.

---

## Track 11 — Closed Beta Launch

### 11.1 — Confirm pre-launch checklist (08:00 daily summary green)
- **Epic**: Beta go/no-go
- **Story**: As an operator, I want all 11 items in `06-closed-beta.md` verified PASS before sending the first invite.
- **Technical task**: Run each verify command; record PASS/FAIL per item; address any FAIL.
- **Priority**: P0
- **Owner**: Operator + AI executor
- **Files**: `docs/operations/06-closed-beta.md` (annotate with run date + result)
- **Dependencies**: All Track 1, 2, 3, 4 P0 items
- **Validation command**: `Get-ScheduledTask`, `Get-Content summary-*.txt`, etc., per checklist
- **Acceptance**: 11/11 PASS.
- **DoD**: Annotated checklist committed.

### 11.2 — Curate first 5 invitees + invitation message
- **Epic**: Beta go-live
- **Story**: As an operator, I have 5 hand-picked invitees with the Beta caveat letter prepared.
- **Technical task**: Operator drafts list + message (in Arabic, includes Beta caveats: single-host, daily backups, no SLA, direct support channel).
- **Priority**: P0
- **Owner**: Operator + Customer Success
- **Files**: `docs/operations/closed-beta-invitees.md` (gitignored or stored in private notes)
- **Dependencies**: 11.1
- **Validation**: Operator confirms ready
- **Acceptance**: List + message exists
- **DoD**: First invite sent.

### 11.3 — Daily morning routine documented
- **Epic**: Beta operations
- **Story**: As an operator, I want a 5-minute morning checklist embedded into my routine.
- **Technical task**: Add `docs/operations/operator-morning-routine.md` linking to: today's summary, alerts log tail, queue worker log tail, recent backup file, Sentry dashboard.
- **Priority**: P1
- **Owner**: Ops
- **Files**: `docs/operations/operator-morning-routine.md` (new)
- **Dependencies**: 4.1 (Sentry)
- **Validation**: Operator runs through it in <5 min
- **Acceptance**: Routine completable in 5 min
- **DoD**: Doc committed.

---

## Track 12 — Public Launch

### 12.1 — Phase E real-traffic load test baseline
- **Epic**: Performance baseline
- **Story**: As a CTO, I need real-traffic p50/p95/p99 latency + queue depth + Postgres connection pool baseline before opening registration.
- **Technical task**: Capture 1 week of Beta traffic; baseline against k6/locust scenarios approximating 10x growth. Document breakpoints.
- **Priority**: P1
- **Owner**: DevOps + Backend
- **Files**: `docs/operations/08-load-baseline.md` (new)
- **Dependencies**: 30 days of real Beta traffic
- **Validation**: Baseline doc with numbers + breakpoints
- **Acceptance**: Operator can answer "at what concurrency does the system degrade?"
- **DoD**: Baseline committed.

### 12.2 — Playwright in CI on PR
- **Epic**: CI/CD safety
- **Story**: As a CTO, I want PRs blocked if E2E smoke tests fail.
- **Technical task**: Add `e2e` job to `.github/workflows/ci.yml`. Use Playwright `--reporter=html`. Cache browsers.
- **Priority**: P1 for Public Launch
- **Owner**: DevOps
- **Files**: `.github/workflows/ci.yml`
- **Dependencies**: 3.2
- **Validation command**: Push PR; verify e2e job runs.
- **Acceptance**: Failing E2E blocks merge.
- **DoD**: First PR with broken E2E correctly blocked.

### 12.3 — Migration `--pretend` gate in deploy.yml
- **Epic**: Production safety
- **Story**: As a DevOps lead, I want every production deploy to dry-run migrations and fail if SQL looks unsafe.
- **Technical task**: Add `php artisan migrate --pretend --database=landlord` + same for tenants; assert output size > 0 only when expected; surface in deploy summary.
- **Priority**: P1
- **Owner**: DevOps
- **Files**: `.github/workflows/deploy.yml`
- **Dependencies**: None
- **Validation**: PR introducing destructive migration → deploy blocked.
- **Acceptance**: Dry-run output published as deploy artifact.
- **DoD**: First destructive-looking migration correctly halted.

### 12.4 — composer/pnpm audit gate (closes M-003)
- **Epic**: Supply-chain security
- **Story**: As a CISO, I want HIGH-severity dependency vulns to block deploy.
- **Technical task**: Remove `|| true` from CI audit steps. Add allowlist file for accepted CVEs with sunset date.
- **Priority**: P1
- **Owner**: DevOps + Security
- **Files**: `.github/workflows/ci.yml`, `security/audit-allowlist.json` (new)
- **Dependencies**: None
- **Validation command**: Inject known-vulnerable dep version; CI fails.
- **Acceptance**: HIGH vuln in any dep blocks merge unless allowlisted.
- **DoD**: M-003 closed.

### 12.5 — Status page + incident comms plan
- **Epic**: Customer trust
- **Story**: As a public user, I want a single URL to check service health + read incident updates.
- **Technical task**: Provision status page (Better Stack or Statuspage.io). Wire health probes. Write incident comms template.
- **Priority**: P1 for Public Launch
- **Owner**: Ops + Customer Success
- **Files**: `docs/operations/incident-comms.md` (new)
- **Dependencies**: 4.4
- **Validation**: Test incident → page updates within 5 min
- **Acceptance**: Probes live; template ready.
- **DoD**: Status page URL published.

---

## Track 13 — Header & First Impression 2.0 (DEFERRED to charter)

> **All items below are sealed in `docs/strategy/04-experience-2-0-charter.md`**.
> Do NOT pull any of these into a sprint unless the charter's gate conditions are met (≥30 days real Beta data + ≥5 completions + ≥50 FE Sentry events + ≥3 user interviews + ≥20 Tally responses + ≥10 documented UX pains).
> Captured here for traceability only.

### 13.1 — Hide NotificationBell from guest visitors
- **Item**: gate `<NotificationBell />` on `isAuthenticated`
- **Effort**: 5 min
- **ROI**: ⚡⚡⚡⚡ (-12% header visual noise)
- **Status**: DEFERRED until charter opens

### 13.2 — Logo + header height reduction
- **Item**: `h-9 lg:h-10` logo, `h-14 lg:h-16` header
- **Effort**: 10 min
- **ROI**: ⚡⚡⚡ (+10% premium perception, +8% above-the-fold content)
- **Status**: DEFERRED

### 13.3 — Search → Command Palette (Cmd+K)
- **Item**: replace inline `SearchTypeahead` with command-palette pattern (`cmdk` already in deps)
- **Effort**: 1–2 days
- **ROI**: ⚡⚡⚡⚡ (+20% engagement with non-search nav)
- **Status**: DEFERRED

### 13.4 — Trust badge in header
- **Item**: small icon + tooltip linking to `/trust`, visible always (all user states)
- **Effort**: 1 hour
- **ROI**: ⚡⚡⚡⚡⚡ (+5–8% first-visit → sign-up in Saudi market)
- **Status**: DEFERRED

### 13.5 — UserMenu (avatar + dropdown) replaces 3 role-buttons
- **Item**: single avatar trigger, dropdown contains `لوحتي / لوحة المدرّب / لوحة الإدارة` per role
- **Effort**: 2–3 hours
- **ROI**: ⚡⚡⚡ (-20% multi-role confusion)
- **Status**: DEFERRED

### 13.6 — Mobile search icon + modal
- **Item**: tap-icon opens full-screen search on `< md` viewports
- **Effort**: 4–6 hours
- **ROI**: ⚡⚡⚡⚡⚡ (+15–20% mobile search usage)
- **Status**: DEFERRED

### 13.7 — Role-aware navigation (5 variants)
- **Item**: guest / student / instructor / admin / enterprise headers (see `04-experience-2-0-charter.md` § Track A)
- **Effort**: 2–3 days
- **ROI**: ⚡⚡⚡⚡ (compound — depends on role mix)
- **Status**: DEFERRED

### 13.8 — Sign-up CTA hierarchy boost
- **Item**: `sign-in` becomes text-only link, `sign-up` stays as orange button — larger visual delta
- **Effort**: 15 min
- **ROI**: ⚡⚡⚡ (+5–7% sign-up clicks)
- **Status**: DEFERRED

---

## Appendix — Item ↔ Audit Finding Cross-Reference

| Track | Item | Closes |
|---|---|---|
| 1.1 | RequireTwoFactor test | H-001 |
| 1.2 | Admin IP allowlist | H-002 |
| 1.3 | Admin idle timeout | H-003 |
| 1.4 | Payment webhook signature test | SEC-06 |
| 1.5 | Program transition audit | M-002 |
| 2.1 | Cache tenant prefix | **C-001** |
| 2.2 | Queue tenant context test | TI-02 |
| 2.3 | Signed URL cross-tenant test | TI-05 |
| 2.4 | Cross-tenant API test | TI-01 expansion |
| 3.1 | Vitest critical suite | **C-002** |
| 3.2 | Playwright smoke suite | **C-003** |
| 3.3 | Pest arch tests | H-004 |
| 3.4 | PHPStan baseline | H-005 |
| 4.1 | Sentry DSN | **C-004** |
| 4.2 | Log scan rotated files | **C-005** |
| 4.3 | Sentry release tag in CI | TD-08 |
| 4.4 | Better Stack uptime | Public Launch gate |
| 5.1 | Subdomain DNS runbook | Public Launch gate |
| 5.2 | Beta cap enforcement | H-009 |
| 5.3 | Tenant onboarding command | Public Launch gate |
| 5.4 | Search isolation | TI-07 |
| 6.1 | a11y axe-core | FE-06 |
| 6.2 | Plausible analytics | L-010 |
| 6.3 | Bundle analyzer | Performance |
| 7.1 | Soft-delete partial indexes | DB-05 |
| 7.2 | Soften audit trigger | DB-03 |
| 7.3 | Live session test | M-010 |
| 8.1 | Backup retention | M-009 |
| 8.2 | Read replica plan | DB-08 |
| 8.3 | Tenant report endpoint | Public Launch capability |
| 9.1 | Cookie consent | PDPL Public Launch gate |
| 9.2 | Hard-purge cron | PDPL grace period |
| 9.3 | DPA template | Enterprise gate |
| 10.1 | README/STATUS/BACKLOG rewrite | **H-008** |
| 10.2 | Dev onboarding doc | Velocity |
| 11.1 | Pre-launch checklist run | Beta GO |
| 11.2 | First invitees | Beta GO |
| 11.3 | Morning routine | Beta ops |
| 12.1 | Phase E load baseline | Public Launch gate |
| 12.2 | Playwright in CI | Public Launch gate |
| 12.3 | Migration pretend gate | Public Launch gate |
| 12.4 | Audit gate | M-003 |
| 12.5 | Status page | Public Launch gate |
