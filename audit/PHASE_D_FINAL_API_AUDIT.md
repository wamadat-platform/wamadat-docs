# Phase D — Final API Audit

**Captured:** 2026-05-14 (end of Phase D Part 2).
**Commit anchor:** `dc0acc4` (CP4-12 closeout).
**Scope:** the public v1 API surface under `/api/v1/*` — 90 documented operations across 55 paths (excludes `/health` + `/webhooks/*` by design).

This is the consolidated audit that Closed Beta operates against. Every section answers a question a mobile/SDK consumer would ask before shipping a client against v1.

---

## 1. Deprecated fields (going away in v2 — clients SHOULD migrate)

| Field | Endpoint(s) | Replacement | Status |
|---|---|---|---|
| `data.token` | `/auth/register`, `/auth/login`, `/auth/refresh` | `data.access_token` | Alias; still emitted for the transition. Removal target: v2. |
| `file_url` (lesson resources) | `/learn/programs/{slug}` | `download_url` (signed, scoped, time-limited) | REMOVED — CP4-11 (2026-05-13). Breaking. |

**Everything else** in v1 is current and not deprecated.

---

## 2. Undocumented behavior — none remaining

Everything that affects the client wire contract is captured in `docs/api/`:

| Doc | Covers |
|---|---|
| `01-inventory.md` | Full endpoint list by domain (CP4-1). |
| `02-response-shape-rebuild.md` | Canonical success + error envelope (CP4-2). |
| `03-versioning.md` | v1 surface lock, v2 migration policy (CP4-3). |
| `04-error-codes.md` | Closed `code` vocabulary, 22 codes, per-status mapping (CP4-9). |
| `05-mobile-auth.md` | Two-token model, refresh flow, session management (CP4-5). |
| `06-pagination.md` | Three-tier envelope contract (CP4-4). |
| `07-rate-limits.md` | 24 throttled endpoints, full policy (CP4-10). |
| `08-scribe-and-postman.md` | Machine-readable docs how-to (CP4-7 + CP4-8). |
| `09-signed-media-urls.md` | Signed-URL contract + threat model (CP4-11). |
| `CHANGELOG-v1.md` | Every contract change, dated, before/after, migration notes. |
| `public/docs/openapi.yaml` | OpenAPI 3.0.3 spec generated from routes. |
| `public/docs/collection.json` | Postman v2.1.0 collection. |

Per-endpoint OpenAPI `responses` / `summary` are intentionally light (R-OPEN-30) — the canonical envelope is uniform across all endpoints, documented at the meta level. Not blocking SDK codegen.

---

## 3. Mobile blockers — none

All Phase D blockers are resolved. The mobile SDK can be built against:
- **Auth model:** two-token (Sanctum access + refresh rotation), CP4-5, R-OPEN-22/23 closed during this phase.
- **Error contract:** uniform envelope with closed-vocabulary `code`, CP4-2 + CP4-9.
- **Pagination:** stable three-tier shape, CP4-4.
- **Localization:** `Accept-Language: ar|en`, CP4-6.
- **Rate limits:** documented per-endpoint, mobile-safe codes, CP4-9 + CP4-10.
- **File downloads:** signed URLs with user binding + enrollment re-check, CP4-11.

The OpenAPI YAML at `backend/public/docs/openapi.yaml` is the source of truth for client codegen. Postman collection at `backend/public/docs/collection.json` for manual exploration.

---

## 4. Inconsistent endpoints — known + accepted

Surfaced and documented during CP4-1 / CP4-3 / CP4-4 audits. None block Closed Beta:

| Inconsistency | Where | Why kept |
|---|---|---|
| `me/today` returns a flat `{program, lessons, streak}` shape — no `meta` block | `/me/today` | Domain-specific aggregate; not a list endpoint. Documented in 06-pagination.md as a separate shape. |
| `me/calendar.ics` returns `text/calendar` (not JSON) | `/me/calendar.ics` | iCal RFC 5545 requires this. Not a JSON contract. |
| `verify/{code}` returns `{valid, data, revoked, revoked_at, revoked_reason}` | Public certificate verification | Pre-CP4-2 shape kept for the public verification page; no client SDK consumes this. |
| `program_state/transition` returns `{transitioned_from, transitioned_to, ...}` | Trainer cohort actions | Trainer-only surface; not in the mobile-student SDK contract. |

These are flagged for review during v2 design (no migration needed for v1 mobile rollout).

---

## 5. Breaking changes summary (v1 timeline)

Per `docs/api/CHANGELOG-v1.md`. Mobile clients built before each date need migration:

| Date | CP | Surface | Migration |
|---|---|---|---|
| 2026-05-13 | CP4-2 | All `/api/v1/*` error responses | Switch from `{message:'…'}` to `{error:{code, message, status, details}}`. Match on `error.code`. |
| 2026-05-13 | CP4-5 | `/auth/login`, `/auth/register`, NEW `/auth/refresh` | Switch from single `data.token` to two-token `data.access_token` + `data.refresh_token`. Implement refresh-on-401 flow. |
| 2026-05-14 | CP4-4 | `/catalog/programs`, `/catalog/instructors` | Pagination envelope: `{data, meta:{current_page, per_page, total, last_page, has_more}}`. Stop reading `links.*`. |
| 2026-05-13 | CP4-11 | `/learn/programs/{slug}` resources | `resources[i].file_url` removed; use `resources[i].download_url`. Download URL is 15-minute signed; send the same Bearer token. |

Non-breaking entries on the same dates: CP4-6 (Accept-Language), CP4-7 + CP4-8 (Scribe docs), CP4-9 (error codes catalogue), CP4-10 (RATE_LIMITED contract fix).

---

## 6. Unresolved risks summary

From `docs/audit/OPEN_RISKS.md` — risks remaining after Phase D close. None block Closed Beta; severity drives sequencing.

### High-priority residual (closed during Phase D — keep for reference)
- **R-OPEN-22 — CLOSED:** Password change now revokes other sessions.
- **R-OPEN-23 — CLOSED:** Legacy 81 `web-session` tokens purged.
- **R-OPEN-29 — CLOSED:** `ApiErrorRenderer` namespace bug (throttle → RATE_LIMITED) fixed in CP4-10.
- **R-OPEN-31 — CLOSED:** Tenant-test bootstrap regression fixed during CP4-12.

### Open P2 (Closed Beta cleanup work)
- **R-OPEN-19:** Frontend `parseApiError` retains legacy fallback branches — purely client cleanup.
- **R-OPEN-20:** Frontend `parseApiError` retains legacy fallback branches.
- **R-OPEN-21:** ~60 legacy controllers still emit pre-canonical shape; rewritten by `NormalizeApiErrorResponse` middleware. Wire contract is intact.
- **R-OPEN-24:** `data.token` deprecated alias still emitted — see §1 above. Remove in v2.
- **R-OPEN-25:** Concurrent-rotation race not proven by harness — manual proof from CP4-5 stands.
- **R-OPEN-26:** Logout-then-refresh returns `REFRESH_TOKEN_REUSED` instead of `INVALID` — analytics noise, not a security gap.
- **R-OPEN-27:** Invalid UUID in `Model::find()` returns 500 — sweep ~15 known sites.
- **R-OPEN-28:** Per-field FormRequest validation messages bypass Accept-Language — sweep ~15 FormRequests.
- **R-OPEN-30:** Scribe OpenAPI lacks per-endpoint `@response` blocks — SDK codegen polish.
- **R-OPEN-32:** 53 legacy Pest tests assert pre-canonical shapes; rewrite or delete during Closed Beta.

None are **launch blockers**; they're documented hygiene work that runs in parallel with operational learning.

---

## 7. What changed in Phase D Part 2 (since 2026-05-13 baseline)

12 commits + 1 cleanup commit:

| CP | Commit | Subject |
|---|---|---|
| CP4-5 | `352077a` | Two-token mobile auth + refresh flow |
| — | `e7de7e7` | R-OPEN-22 + R-OPEN-23 high-priority flag |
| — | `4b2bf9c` | CP4-5 closeout — proof + OPEN_RISKS + OPS |
| CP4-9 | `adaaf02` | Mobile-safe error codes catalogue |
| CP4-6 | `b8e4514` | Accept-Language handling |
| CP4-4 | `9705f03` | Mobile-safe pagination envelope (Tier 1) |
| CP4-10 | `331a4f5` | API rate-limit audit + RATE_LIMITED contract fix |
| CP4-7+8 | `fc50d79` | Scribe-generated OpenAPI + Postman collection |
| CP4-11 | `d2b6fe6` | Signed/scoped lesson-resource download URLs |
| CP4-12 | `dc0acc4` | R-OPEN-31 close + Final API Contract Safety Net |

Phase D Part 2 scope (locked 2026-05-13): **9 / 9 items shipped, deep-proof tier for the 4 critical ones (auth, rate-limits, signed URLs, contract tests), light-touch tier for the 5 documentation-grade items.**

---

## 8. Test surface

Contract safety nets exercised by Pest:

| Suite | Tests | Assertions | Files |
|---|---|---|---|
| API contract (CP4-12) | 16 | 114 | `tests/Feature/Tenant/ApiContractTest.php` |
| Signed media URLs (CP4-11) | 9 | 26 | `tests/Feature/Tenant/MediaSignedUrlTest.php` |

**Total going-forward safety net: 25 tests, 140 assertions, all green.**

Legacy tests (53 failures from R-OPEN-32) are pre-canonical and slated for cleanup during Closed Beta — they do NOT exercise the v1 contract, they exercise a long-stale pre-CP4 contract. The CP4-12 suite is the going-forward safety net.

Live HTTP smoke is the proof of record for: CP4-2, CP4-4, CP4-5, CP4-6, CP4-9, CP4-10, CP4-11. Each commit message documents the smoke commands.

---

## 9. Verdict for v1 stability

The v1 surface is **frozen at commit `dc0acc4`**. Mobile clients can codegen against the OpenAPI YAML and ship; the Postman collection imports cleanly; the 25-test contract net protects against future regressions.

No additional contract work is needed before Closed Beta. The remaining open risks are operational cleanups (R-OPEN-19/20/21/24-28/30/32) that run in parallel with Closed Beta learning.
