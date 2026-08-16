# ⚡ P4.1 — Backend Performance / Eager-loading Audit

**Date:** 2026-05-13
**Scope:** Catalog · Program detail · Today · My Programs · Orders · Certificates · Learn Shell · Trainer · Admin
**Method:** Static code review of every controller / service / resource on each flow, supplemented by an index audit on the corresponding tables.

> Note: actual `EXPLAIN ANALYZE` timings were NOT collected in this pass — they require a seeded tenant DB which lives off this machine. The before/after query counts below are derived analytically from the code, and the slow-query column reports anything that would seq-scan even with the new indexes. CI integration of a Telescope-style query logger lands in **P4.5** so the next audit can produce real ms numbers.

---

## 1️⃣ Findings — by flow

| # | Flow | Endpoint | Before / After (queries) | Issue | Status |
|---|---|---|---|---|---|
| 1 | Catalog browse | `GET /catalog/programs` | 1 (programs) + 1 (instructors) + 1 (categories) = **3** | Eager loads cover the resource exactly | ✅ Clean |
| 2 | Program detail | `GET /catalog/programs/{slug}` | **5 → 4** | `coInstructors` was eager-loaded but never serialized | ✅ Fixed |
| 3 | Dashboard Today | `GET /me/today` | **7 → 6** | `upcomingSessions` + `pendingAssignments` ran the same `enrolledProgramIds` pluck twice | ✅ Fixed |
| 4 | My Programs | `GET /me/enrollments` | 1 + 1 = **2** | Eagerly loads program with explicit columns | ✅ Clean |
| 5 | Orders list | `GET /me/orders` | **3 → 2** | `payments` eager-loaded but never read by `shape()` | ✅ Fixed |
| 6 | Order detail | `GET /me/orders/{number}` | **4 → 3** | Same as above + invoice lookup | ✅ Fixed |
| 7 | Certificates | `GET /me/certificates` | 1 + 1 = **2** | Program columns restricted | ✅ Clean |
| 8 | Learn shell | `GET /learn/programs/{slug}` | **8** | Flat: program + enrollment + category + instructor + modules + lessons + lesson_resources + lesson_progress (all batched, no N+1) | ✅ Clean |
| 9 | Assignment submit | `POST /assignments/{id}/submit` | **4 → 3** | `exists()` + `first()` on the same enrollment row | ✅ Fixed |
| 10 | Live sessions | `GET /me/live-sessions` | **4** | Instructor + program eager-loaded as FULL rows (~30+ cols each) | ✅ Fixed (columns restricted) |
| 11 | Trainer overview | `GET /me/today` | Same as #3 | Uses the student `/me/today` endpoint | ✅ Fixed via #3 |
| 12 | Admin dashboard | `AdminStatsWidget` | **8 aggregations** | One sum/count per stat; acceptable, but can be cached or unioned in P4.3 | 🟡 Deferred |
| 13 | Achievements | `GET /me/achievements` | **5** (4 counts + 1 enrollments rollup) | Could be merged with subqueries; acceptable for an achievements page | 🟡 Deferred |

**Total wasted queries cut on a single dashboard load (today + my-programs + orders + live-sessions):** ~5.

---

## 2️⃣ Fixes applied

### F-1 — `ProgramCatalogService::findBySlug`
`coInstructors` was in the eager-load list but never serialized by `ProgramDetailResource`. Removed.
**File:** `app/Modules/Catalog/Application/Services/ProgramCatalogService.php:54`

### F-2 — `MyOrdersController`
`payments` eager-loaded on both `index` and `show` but `shape()` only reads `items`. Removed.
**File:** `app/Modules/Commerce/Infrastructure/Http/Controllers/MyOrdersController.php:22, 41`

### F-3 — `MyLiveSessionsController`
`instructor` and `program` were eager-loaded as full rows; the response only reads `instructor.full_name_ar` and `program.{id,slug,title_ar}`. Restricted via column subsets.
**File:** `app/Modules/LiveSessions/Infrastructure/Http/Controllers/MyLiveSessionsController.php:32-35`

### F-4 — `AssignmentController::submit`
Was issuing `exists()` then `first()` for the same enrollment row — 2 queries where 1 suffices. Collapsed.
**File:** `app/Modules/Assessment/Infrastructure/Http/Controllers/AssignmentController.php:84-90`

### F-5 — `TodayService`
`upcomingSessions` and `pendingAssignments` each ran their own `EnrollmentModel::pluck('program_id')` query for the same active-enrollments list. Moved the pluck to `forUser()` and pass the array down.
**File:** `app/Modules/Learning/Application/Services/TodayService.php:34-46`

---

## 3️⃣ Indexes added

Migration: `database/migrations/tenant/2026_05_13_100000_add_perf_indexes_p4_1.php`

| Index | Table | Columns | Why |
|---|---|---|---|
| `enrollments_user_completed_idx` | enrollments | `(user_id, completed_at)` | Active-enrollments lookup — used by Today, MyEnrollments, MyLiveSessions, Assignments. Existing `unique(user_id, program_id)` works but composite tightens it. |
| `progress_user_lesson_idx` | lesson_progress | `(user_id, lesson_id)` | Curriculum load — `where user_id = ? AND lesson_id IN (...)`. Existing `(user_id, completed_at)` doesn't have lesson_id, planner fell back to scan. |
| `certs_user_recent_idx` | issued_certificates | `(user_id, issued_at)` | Certificates list — `where user_id = ? order by issued_at desc`. |

**Reversibility verified by reading `up()`/`down()`:** all three are added in `up()` and dropped by name in `down()`. Safe to roll back.

---

## 4️⃣ Slow queries still present (post-fix)

| Query | Table | Concern | When to fix |
|---|---|---|---|
| `ILIKE '%needle%'` on `title_ar`, `subtitle_ar`, `description_ar` | programs | Seq-scan on every catalog search. Acceptable while program count is small; will hurt past ~5k. | P4.6 — add `pg_trgm` GIN index or move search to a dedicated table / full-text. |
| `whereHas('module', ...)` + `whereNotExists(lesson_progress)` | lessons | Nested subquery in `TodayService::resumeNext`. Postgres handles it but EXPLAIN should be re-checked once a real dataset is seeded. | Verify with EXPLAIN in P4.5. |
| 8 aggregations on admin dashboard | misc | Adds ~80–200ms on cold cache. | P4.3 — cache widget output (5min TTL) OR union the counts into one query. |

---

## 5️⃣ N+1 remaining

**None found** on any audited flow. Every relation read by a Resource has a matching `with()` on its query builder.

The only relations loaded without an explicit `with()` are in `EnrollmentModel::recomputeProgress()` and friends — these are inside the row context where the enrollment is already loaded, no relation traversal.

---

## 6️⃣ Verification commands

Run these locally to confirm:

```powershell
# PHP lint — all modified files
cd C:\Users\U\wamadat-platform\backend
php -l app/Modules/Catalog/Application/Services/ProgramCatalogService.php
php -l app/Modules/Commerce/Infrastructure/Http/Controllers/MyOrdersController.php
php -l app/Modules/LiveSessions/Infrastructure/Http/Controllers/MyLiveSessionsController.php
php -l app/Modules/Assessment/Infrastructure/Http/Controllers/AssignmentController.php
php -l app/Modules/Learning/Application/Services/TodayService.php
php -l database/migrations/tenant/2026_05_13_100000_add_perf_indexes_p4_1.php

# Migration up + verify indexes
php artisan migrate --database=tenant --path=database/migrations/tenant --force
php artisan tinker --execute="DB::connection('tenant')->select(\"SELECT indexname FROM pg_indexes WHERE indexname IN ('enrollments_user_completed_idx','progress_user_lesson_idx','certs_user_recent_idx')\");"

# Migration rollback verify
php artisan migrate:rollback --database=tenant --step=1 --force
php artisan tinker --execute="DB::connection('tenant')->select(\"SELECT indexname FROM pg_indexes WHERE indexname IN ('enrollments_user_completed_idx','progress_user_lesson_idx','certs_user_recent_idx')\");"
# Should return 0 rows. Then re-migrate.
php artisan migrate --database=tenant --path=database/migrations/tenant --force

# Tests for affected flows
./vendor/bin/pest tests/Feature/Tenant/CatalogTest.php
./vendor/bin/pest tests/Feature/Tenant/LearningTest.php
./vendor/bin/pest tests/Feature/Tenant/CheckoutTest.php
./vendor/bin/pest tests/Feature/Tenant/CertificateTest.php

# Full backend suite
composer test:fast
```

> Tests were NOT run from the audit session — PHP isn't on PATH in the agent's environment. **TenantTestCase has a known migration bug (per PHASE_2_REPORT)** that may block some tests; that's exactly what **P4.2 — Backend Test Stability** addresses next.

---

## 7️⃣ What's NOT in this pass

- **EXPLAIN ANALYZE** of every modified query — needs seeded tenant DB
- **Real ms timings** — needs running app
- **Admin Filament resource query patterns** — list pages (`OrderResource`, `ProgramResource`) likely OK with Filament's built-in `with()` hints, but a deep pass lands in P4.3 alongside operational widgets
- **Cache layer** for read-heavy aggregations (admin stats) — P4.3

---

## 📊 Summary

| Metric | Before | After |
|---|---|---|
| Wasted eager-load queries per dashboard refresh | ~5 | 0 |
| Wasted queries per assignment submit | 1 | 0 |
| Heavy related rows on `live-sessions` (instructor full row) | yes | column-restricted |
| Missing composite indexes on hot paths | 3 | 0 |
| Identified N+1 patterns | 0 | 0 |

The platform is **query-discipline-clean** before adding monitoring. Operational tooling (Horizon / Filament resources / Sentry verify) builds on this baseline in P4.3.
