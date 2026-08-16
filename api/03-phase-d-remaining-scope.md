# Phase D — Remaining Scope (status-per-item)

**Generated:** 2026-05-13 (Phase D Part 1 checkpoint).
**Status at checkpoint:** commit `ae2dd5e` + the /health bypass fix landing in the closeout commit.
**Policy:** every remaining item resumes at the same proof depth as Phase B / B-Hardening / Phase C. No shallow proof.

---

## Done

| # | Item | Status | Anchor |
|---|---|---|---|
| **CP4-1** | Full API inventory | ✅ DONE | `docs/api/01-api-inventory.md` + commit `ae2dd5e` |
| **CP4-2** | Unified response shape (success + error) | ✅ DONE — **REBUILD REQUIRED** raised + delivered | `docs/api/02-response-shape-rebuild.md` + commit `ae2dd5e` + closeout `1ac3b8e` |
| **CP4-3** | Versioning audit (`/api/v1`) | ✅ DONE — all 94/94 endpoints under v1 | section 4 of `01-api-inventory.md` |
| **CP4-5** | Mobile auth + token refresh | ✅ DONE — two-token model, sessions, reuse-detect | `docs/api/05-cp4-5-auth-proof-bundle.md` + commit `352077a` + this closeout |

---

## Remaining locked scope — 8 items

Order in this list is the recommended execution order for the next session.

### CP4-9 — Mobile-safe error codes catalogue ← START HERE in the next session
- **Status:** PENDING
- **Why it's next:** the canonical shape now ships AND CP4-5 added 7 more codes (`REFRESH_TOKEN_*`, `CANNOT_REVOKE_CURRENT_SESSION`, `SESSION_NOT_FOUND`, `ACCOUNT_INACTIVE`). The closed vocabulary of `error.code` strings is still scattered across `ApiErrorRenderer`, `NormalizeApiErrorResponse`, `TokenService`, `SessionsController`. Mobile devs need one authoritative table.
- **Expected deliverable:** `docs/api/04-error-codes.md` enumerating every code, its HTTP status, AR + EN message, and the emit sites. Pest test that grep-asserts no code is emitted from outside this file (or at minimum a CI check).
- **Estimated depth:** 2-3 hours.

### CP4-4 — Pagination strategy on every list endpoint
- **Status:** PENDING
- **Today:** unaudited. Some endpoints page (catalog/programs), some don't, meta shape inconsistent.
- **Decisions:** cursor vs offset per endpoint. Cursor preferred for feeds (notifications, orders, enrollments). Offset OK for fixed-cap admin lists.
- **Expected deliverable:** audit table per endpoint + retrofit for the ones that need cursor + meta-shape normalization (`{data: [...], meta: {next_cursor, has_more}}`).
- **Estimated depth:** 4-6 hours — most expensive remaining item.

### CP4-6 — Accept-Language handling
- **Status:** PENDING
- **Today:** all messages return AR. `App::setLocale` is never set from request.
- **Expected deliverable:** middleware that parses `Accept-Language` header, sets `App::setLocale('ar'|'en')` for the request, AR-EN parallel translation files seeded for the closed code vocabulary (CP4-9 dependency).
- **Estimated depth:** 2-3 hours.

### CP4-10 — Rate-limit audit
- **Status:** PARTIAL (login + register limits exist from B7)
- **Today:** `/auth/login` 3-layer, `/auth/register` 3 per 5 min, everything else default 60/min (`throttle:api`).
- **Expected deliverable:** per-endpoint review — flag any write/sensitive endpoint without an explicit limit; add per-route throttles for refund-trigger, password-reset, 2FA setup, OTP send, etc.
- **Estimated depth:** 2-3 hours.

### CP4-11 — Signed / scoped file URLs
- **Status:** PENDING
- **Today:** `cover_image_url`, `video_url`, `certificate_url` in API responses — most point at public Filament `/storage/` paths. Sensitive (lesson videos, invoice PDFs, certificates) need signed temporary URLs.
- **Expected deliverable:** per-file-class decision (public / signed / role-gated). Signed URL helper for sensitive paths. Test that an unauthenticated client cannot fetch a lesson video URL.
- **Estimated depth:** 3-4 hours.

### CP4-7 — OpenAPI / Scribe documentation
- **Status:** PENDING
- **Decision in next session:** install `knuckleswtf/scribe` (annotation-driven, easy) vs hand-author OpenAPI 3.1 YAML (precise, more effort).
- **Expected deliverable:** generated spec covering all 90 endpoints. Committed under `docs/api/openapi.yaml` (or Scribe's published HTML).
- **Estimated depth:** 4-6 hours.

### CP4-8 — Postman collection
- **Status:** PENDING
- **Expected deliverable:** Postman collection generated from the OpenAPI spec OR hand-built. Covers the 7 priority flows with environment variables + auth header pre-script.
- **Estimated depth:** 1-2 hours (after CP4-7).

### CP4-12 — Contract tests for 7 priority flows
- **Status:** PENDING — depends on EVERYTHING above being stable.
- **Tests required:** login (success + wrong password + validation), programs list (shape + pagination meta), enrollment create, payment status read, invoice fetch, notifications list, profile fetch.
- **Expected deliverable:** Pest feature tests under `tests/Feature/Api/Contracts/` that assert exact response shape (keys, types, status code, error code). Tests fail if any contract drifts.
- **Estimated depth:** 3-4 hours.

---

## Cumulative remaining-effort estimate

~25-35 hours of focused work for the 9 items. Realistic across 3-4 working sessions if proof depth stays at Phase B / B-Hardening / Phase C level. **No shortcuts.**

---

## Anti-pattern guardrails for the next session

The Phase D charter explicitly forbade:
- New features → still in force.
- UI polish → still in force.
- "Works locally" without proof → every item in the table above must ship its proof bundle (`current risk / implementation / affected endpoints / API before/after / tests / docs / security impact / mobile impact / remaining risks / commit hash`).

If the next session attempts an item and discovers it ALSO needs REBUILD (e.g. pagination retrofit reveals models without proper indices), it raises REBUILD REQUIRED for that specific layer rather than ship a quiet patch.

---

## Cross-references

- API inventory: `docs/api/01-api-inventory.md`
- Canonical error rebuild spec: `docs/api/02-response-shape-rebuild.md`
- Breaking changes log: `docs/api/CHANGELOG-v1.md` (new in closeout)
- Open risks: `docs/audit/OPEN_RISKS.md`
- Ops snapshot tip: `docs/audit/OPS_STATUS_SNAPSHOT_2026-05-13-phase-c-3.md`
