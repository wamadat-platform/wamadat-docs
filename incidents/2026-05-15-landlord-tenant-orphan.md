# Incident: Landlord Tenant Orphan (Diag / wamadat divergence)

**Date detected:** 2026-05-15 00:05 (operator-discovered while running `create-real-admins.bat`).
**Severity:** P1 — would have blocked Closed Beta launch (no operator could create accounts; the `/admin` panel would have been unreachable via the normal tenant-resolution path).
**Resolved:** 2026-05-15 00:11 with a single transactional UPDATE.
**Root-cause category:** test-state drift into the running dev DB.

## What was wrong

`landlord.tenants` carried exactly one active row, but it pointed at a Postgres schema that did not exist:

| field | value (BEFORE fix) |
|---|---|
| id | `a1c5bf77-b141-474c-a617-7dfea790399e` |
| name | `Diag` |
| slug | `diaglog` |
| subdomain | `diaglog` |
| database_schema | **`tenant_diaglog`** ← does not exist in `wamadat_tenants` DB |
| status | `active` |

Meanwhile the only actual tenant schema (`tenant_wamadat`, 78 tables, all Phase D data, all 30 programs, all 246 lessons, all 225 seed users) was **orphaned** — referenced by zero rows in `landlord.tenants`.

Concrete consequence: `php artisan wamadat:create-tenant-owner --tenant=wamadat` failed at the `TenantModel::query()->where('slug', 'wamadat')->first()` lookup. The same lookup runs in the Spatie multitenancy `SubdomainTenantFinder` middleware — meaning ANY incoming request to `/admin` with `X-Tenant-Slug: wamadat` (or to `wamadat.platform.test`) would also fail to resolve a tenant.

## How it happened (most likely sequence)

The codebase already has the right scaffolding to prevent this from being possible — and yet it happened. Reconstruction:

1. **Initial seed (early sprint):** `php artisan db:seed --class=TenancyFlagshipSeeder` ran on the dev DB. The seeder is idempotent on `slug='wamadat'` — it called the `TenantProvisioner`, which inserted a row into `landlord.tenants` AND created the `tenant_wamadat` schema with the canonical structure. State at this point: aligned.

2. **A test or diagnostic run later:** somebody (Pest test, manual SQL, or a one-off seeder/factory call) overwrote the single tenant row to a `Diag` / `diaglog` placeholder. Three plausible mechanisms:
   - A `DatabaseTransactions`/`RefreshDatabase` test that truncated `landlord.tenants` mid-run and only rolled back the *outer* transaction, leaving inner factory-created Diag row visible.
   - A diagnostic script that ran `TenantModel::query()->update([...])` against the first row to test the resolver, and was never undone.
   - A factory call (`TenantModel::factory()->create([...])`) inside a `tinker` or test that wasn't wrapped in a transaction.
3. **The `tenant_wamadat` schema was untouched.** Phase D feature tests don't drop schemas — they only truncate tables inside them or create ephemeral test schemas. So the orphaned data survived.

4. **All subsequent work proceeded blind to the breakage** because:
   - Phase D feature tests build their own ephemeral tenants per test and don't depend on the persistent state.
   - `daily-summary.ps1` discovered schemas directly via `information_schema.schemata LIKE 'tenant_%'` — bypassing landlord — so it kept reporting 1 tenant correctly.
   - No one tried to log into `/admin` between the breakage and the cleanup attempt — that's the path that would have failed loudly.

Search of the current code for `'Diag'`, `'diaglog'`, or any reference to the orphan row turned up **zero hits**. Whatever produced it is no longer in the codebase. The remnant is purely DB state.

## Why it wasn't detected

| Detection layer | What it would have caught | Why it didn't |
|---|---|---|
| Pest feature test suite | Path-level requests to `/admin` resolving the wrong tenant | Tests create per-test ephemeral tenants; persistent state irrelevant |
| `ci.yml` | Broken end-to-end auth flow | No HTTP-level `/admin` smoke test; CI only runs Pest + Pint + PHPStan |
| `daily-summary.ps1` (pre-fix) | landlord ↔ schema drift | Queried schemas directly; never cross-checked with `landlord.tenants` |
| `restore-drill.ps1` | Restoring would surface the same Diag row | Drill ran, but only verified row counts in the dump round-trip — didn't validate cross-table integrity |
| Manual `/admin` smoke | Login redirect / tenant not found | No daily routine yet that hits the panel |
| `TenancyFlagshipSeeder` re-run | Would have re-provisioned wamadat | Skips on `slug='wamadat'` exists — but the existing row was Diag, slug='diaglog', so the check passed and seed was never re-run |

The strongest single contributor: **no end-to-end "operator opens /admin and lands on the dashboard" check exists**. Every layer measured pieces of the system that worked fine in isolation.

## How it was fixed

```sql
BEGIN;
UPDATE tenants
SET name = 'Wamadat Academy',
    slug = 'wamadat',
    subdomain = 'wamadat',
    database_schema = 'tenant_wamadat',
    status = 'active',
    updated_at = NOW()
WHERE id = 'a1c5bf77-b141-474c-a617-7dfea790399e';
-- Gate: exactly one row matching the expected state
DO $$ ... RAISE EXCEPTION IF COUNT(*) <> 1 ... $$;
COMMIT;
```

Pre-update anchor backup: `wamadat-backup-20260515-001044.zip` (local + OneDrive).

Verified post-fix:
- `php artisan tenant:list` → 1 row, `wamadat`/`tenant_wamadat`, status active
- `php artisan tenant:info wamadat` → "Schema contains 78 tables" ✓
- `daily-summary.ps1` [Tenancy Drift Check] section → `status: aligned`

## How to prevent recurrence

Five layers, ordered by leverage:

### 1. Drift check in daily summary — DONE 2026-05-15

`daily-summary.ps1` now has a `[Tenancy Drift Check]` section that compares:
- count of `landlord.tenants` rows where `deleted_at IS NULL`
- count of actual `tenant_*` schemas
- whether each landlord row's `database_schema` value corresponds to an existing schema

Output: `aligned` OR `*** DRIFT ***`. Operator's morning routine catches it within 24h of recurrence.

### 2. End-to-end `/admin` smoke test in CI — TODO (R-OPEN: smoke-admin-panel)

Add to `ci.yml` Backend job AFTER Pest:
```yaml
- name: Smoke /admin tenant resolution
  run: |
    php artisan tenant:list | grep -q "wamadat" || (echo "Missing wamadat tenant" && exit 1)
    php artisan tenant:info wamadat | grep -q "Schema contains" || (echo "wamadat schema unreachable" && exit 1)
```

This is a 10-second check that fails CI if the landlord ↔ schema mapping is broken on any future commit.

### 3. Pest test isolation hardening — TODO (R-OPEN: pest-landlord-isolation)

The `Tests/TestCase.php` `tearDown()` should re-assert that the landlord baseline (the `wamadat` row) is restored after any test that touches `landlord.tenants`. Two paths:
- Use a global `beforeEach` in the Pest suite that runs `TenancyFlagshipSeeder` IF the row is missing.
- OR a `tearDown` that compares pre/post tenants table state and fails the test that introduces drift.

The second is more strict — drift becomes a test failure that names the offending test directly.

### 4. Make `TenancyFlagshipSeeder` repair-aware — TODO (R-OPEN: flagship-seeder-self-heal)

Current logic:
```php
if (TenantModel::query()->where('slug', 'wamadat')->exists()) return;
```

Stronger logic: also detect orphan-schema and Diag-style placeholder rows, and either repair them in place or raise a loud error pointing at this incident doc. Repair-in-place is more operator-friendly for dev DBs.

### 5. `tenant:info` and `tenant:list` integrated into the deploy checklist — TODO

`docs/operations/01-deploy.md` already has a 5-item pre-flight. Add a 6th:
- Step 6: `php artisan tenant:list` shows the expected tenant set; no Diag/test placeholders.

## Open follow-ups

These are added to BACKLOG with the IDs above:
- `R-OPEN-smoke-admin-panel` — Add CI smoke for `/admin` tenant resolution.
- `R-OPEN-pest-landlord-isolation` — Pest tearDown must protect landlord baseline.
- `R-OPEN-flagship-seeder-self-heal` — Flagship seeder repairs orphaned/Diag state.
- `R-OPEN-deploy-checklist-tenant-step` — Add `tenant:list` verification to deploy pre-flight.

## Lessons

- "All tests pass" did not mean "the system works." It meant "the tests pass." Real end-to-end coverage of the auth + tenant path is missing.
- Per-test ephemeral fixtures isolate tests from each other but also isolate them from drift in the persistent state — a useful property for test stability that became a blind spot here.
- Daily summary is now the front-line drift detector. Treat its red lines as Beta-blockers, not noise.
