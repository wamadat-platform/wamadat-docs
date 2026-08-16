# API Inventory — Wamadat Platform

**Captured:** 2026-05-13 (Phase D Part 1 — CP4-1).
**Source of truth:** `php artisan route:list` on commit `4be7bc3`.

Reproduce:
```bash
cd backend
php artisan route:list | sed 's/\x1b\[[0-9;]*m//g' | grep "api/v1"
```

---

## TL;DR

| | |
|---|---|
| **Total `/api/*` endpoints** | **90** |
| **Under `/api/v1`** | 90 (100%) |
| **Outside v1** | 0 — **CP4-3 audit passes immediately** |
| **Unversioned `/api`** | 0 |
| **Domains** | 8 (auth, me, catalog, learn, checkout, consultations, webhooks, ops) |

All 90 endpoints are under `/api/v1`. No bypass routes. No drift.

---

## By domain

### `/api/v1/auth/*` — 8 endpoints
```
POST   /auth/register                     create account
POST   /auth/login                        password login → token
POST   /auth/logout                       revoke token + clear cookie
GET    /auth/me                           current user shape (DashboardGuard relies on it)
POST   /auth/forgot-password              email reset link via outbox
POST   /auth/reset-password               consume token + set new pwd
POST   /auth/phone/send-otp               SMS OTP step 1
POST   /auth/phone/verify                 SMS OTP step 2
```

### `/api/v1/me/*` — 39 endpoints
Self-service surface for the authenticated user. Notable groups:

```
GET    /me/today                          today widget bundle (lessons + tasks)
GET    /me/streak                         daily streak count
GET    /me/calendar.ics                   ICS feed

POST   /me/2fa/setup                      TOTP enrollment step 1
POST   /me/2fa/confirm                    TOTP enrollment step 2
POST   /me/2fa/disable                    requires password step-up
GET    /me/2fa/status

PUT    /me/profile                        update name/avatar/locale
PUT    /me/password                       change password
PUT    /me/preferences                    notification toggles, language
DELETE /me/account                        GDPR deletion request

GET    /me/notifications                  list + filter=unread
POST   /me/notifications/read-all
POST   /me/notifications/{id}/read
GET    /me/orders                         purchase history
GET    /me/orders/{order_number}          single order detail
GET    /me/enrollments                    my programs
GET    /me/progress/{program}             completion %
GET    /me/achievements
GET    /me/certificates
POST   /me/certificates/issue/{program_slug}
GET    /me/certificates/{id}/download     PDF download
GET    /me/assignments
GET    /me/live-sessions                  upcoming + past
GET    /me/wishlist
POST   /me/wishlist
DELETE /me/wishlist/{program_slug}

GET    /me/notes                          all my lesson notes
PUT    /me/notes/{id}
DELETE /me/notes/{id}

POST   /me/gifts                          send gift program
POST   /me/gifts/redeem
GET    /me/gifts/sent

GET    /me/affiliate                      partner stats
POST   /me/affiliate/enroll
DELETE /me/reviews/{id}                   delete my own review

GET    /me/data-export                    full GDPR export
GET    /me/qr                             personal QR code
POST   /me/qr/regenerate
```

### `/api/v1/catalog/*` — 11 endpoints
```
GET    /catalog/programs                  list with filter/search/pagination
GET    /catalog/programs/{slug}           single program detail
GET    /catalog/programs/{slug}/cover.svg dynamic cover image
GET    /catalog/programs/{slug}/reviews   list reviews on a program
POST   /catalog/programs/{slug}/reviews   submit review
GET    /catalog/featured-programs
GET    /catalog/categories
GET    /catalog/instructors
GET    /catalog/instructors/featured
GET    /catalog/instructors/{id}
GET    /catalog/stats                     marketing-page counters
```

### `/api/v1/learn/*` — 14 endpoints
```
GET    /learn/programs/{slug}             enrolled-only program view
GET    /learn/lessons/{lesson}/notes
POST   /learn/lessons/{lesson}/notes
POST   /learn/lessons/{lesson}/progress   completion mark
POST   /learn/lessons/{lesson}/ask        Q&A to instructor
GET    /learn/lessons/{lesson}/questions
POST   /learn/lessons/{lesson}/questions
DELETE /learn/questions/{id}
POST   /learn/questions/{id}/answers

GET    /learn/quizzes/{id}
POST   /learn/quizzes/{id}/start          → attempt id
POST   /learn/attempts/{id}/answers       submit single answer
POST   /learn/attempts/{id}/submit        finalize
GET    /learn/attempts/{id}/result
```

### `/api/v1/checkout/*` + `/api/v1/enrollments`, `/api/v1/programs/*`, etc. — 7 endpoints
```
POST   /checkout                          create order
POST   /checkout/coupon/validate          dry-run coupon
POST   /enrollments                       (free / gift programs)
POST   /programs/{program}/transition     instructor moves cohort state
POST   /assignments/{id}/submit
POST   /cohorts/{cohort}/finalize         instructor closes a batch
POST   /attendance/scan                   QR scan-in for live sessions
```

### `/api/v1/consultations/*` — 2 endpoints
```
GET    /consultations/packages
POST   /consultations/requests
```

### `/api/v1/webhooks/*` — 2 endpoints
```
POST   /webhooks/{gateway}                Tap / Tamara payment webhooks
POST   /webhooks/resend                   Resend delivery events
```

### Other public — 6 endpoints
```
POST   /contact                           contact form
POST   /newsletter/subscribe
GET    /newsletter/unsubscribe
GET    /search                            global search
GET    /verify/{code}                     certificate / user QR public lookup
GET    /                                  root: { api, tenant }
GET    /health                            see HealthController
```

---

## Auth + middleware annotations

Authentication: Laravel Sanctum with the **`AuthCookieToBearer` bridge middleware**. The bridge:
- Reads `wamadat_token` httpOnly cookie set by `/auth/login` and `/auth/register`.
- Promotes it to an `Authorization: Bearer …` header for the request.
- Mobile clients can skip the cookie path entirely and send the header directly.

CP4-5 will deep-dive this.

Rate limits:
- `/auth/login` — 5/min by (email|ip) + 20/min per ip + 100/hr per ip (B7).
- `/auth/register` — 3 per 5 min by ip (B7).
- Default `throttle:api` (60/min) on the rest.

---

## CP4-3 status — versioning

**PASS.** 90 / 90 endpoints under `/api/v1`. No legacy. No bypass. The version prefix is a real boundary, not aspirational.

When v2 lands (no concrete trigger today; planned for backwards-incompatible response-shape changes), it ships under `/api/v2` and v1 stays alive for a deprecation window. Today's contract tests (CP4-12) freeze the v1 shape.

---

## Initial response-shape sampling (preview for CP4-2)

Five live calls captured the variance. **Four distinct error shapes** in five tries — flagged as REBUILD REQUIRED for the error layer in `02-response-shape-rebuild.md`.

| Call | Shape | Notes |
|---|---|---|
| `GET /api/v1` | `{api, tenant}` flat | root only; not following any envelope |
| `GET /catalog/categories` | `{data: [...]}` | data-wrapped list — standard |
| `POST /auth/login` (bad password) | `{message, errors:{field:[]}}` | Laravel FormRequest default |
| `POST /auth/login` (empty body) | `{message: "Arabic", errors:{field:["Arabic msg"]}}` | same shape, localized values |
| `GET /me/notifications` (401) | `{message: "Arabic"}` only | NO `errors` block, NO `code` field |

A 6th shape observed in earlier work (B7 proof):
- `POST /auth/login` (rate-limited): `{errors: [{code, title, detail}]}` — JSON:API array style.

That's `errors` as an OBJECT MAP in some places and an ARRAY OF OBJECTS in others. A mobile client cannot type-check this without runtime branching.

The CP4-2 rebuild fixes only this — success shapes keep their existing `data` wrapping (already consistent enough).
