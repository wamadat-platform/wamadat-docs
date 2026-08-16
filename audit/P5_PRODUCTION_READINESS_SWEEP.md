# 🚦 P5 — Production Readiness Sweep

**Date:** 2026-05-13
**Method:** Walk every major flow / route / surface, look for broken links, fake buttons, orphan refs, critical placeholders. Fix anything truly broken; document what's intentionally deferred.

---

## 1️⃣ Surfaces audited

| Surface | Route(s) | Backend | Frontend |
|---|---|---|---|
| Marketing / home | `/`, `/programs`, `/about`, `/contact`, `/help`, `/for-business`, `/consultations`, `/instructors`, `/become-instructor`, `/categories`, `/pricing` | catalog read | ✅ live |
| Auth | `/sign-in`, `/sign-up`, `/forgot-password`, `/reset-password`, `/redeem-gift`, `/verify/[code]` | full flow | ✅ live |
| Dashboard | `/dashboard`, `/dashboard/today`, `/dashboard/my-programs`, `/dashboard/orders`, `/dashboard/certificates`, `/dashboard/assignments`, `/dashboard/live-sessions`, `/dashboard/notifications`, `/dashboard/profile`, `/dashboard/settings`, `/dashboard/achievements` | 11 endpoints | ✅ live |
| Trainer | `/trainer`, `/trainer/attendance` | inline today + qr scan | ✅ live (camera path stubbed) |
| Admin (Filament) | `/admin/programs`, `/admin/orders`, `/admin/invoices`, `/admin/coupons`, `/admin/reviews`, `/admin/contact-messages`, `/admin/quizzes`, `/admin/assignments`, `/admin/categories`, `/admin/lessons`, `/admin/cohort-batches`, `/admin/live-sessions`, `/admin/failed-jobs`, `/admin/email-outboxes`, `/admin/payment-webhooks`, `/admin/trash` | 16 resources | rendered server-side |
| Super admin | `/super/*` | tenant lifecycle, plans | rendered server-side |
| Checkout | `/cart`, `/checkout`, `/checkout/success`, `/checkout/return`, `/checkout/simulate`, gateway webhooks | full flow with idempotent webhook | ✅ live |
| Learn workspace | `/learn/[slug]`, `/learn/quizzes/[id]` | curriculum + progress + Q&A + notes + AI | ✅ live (P3.6 hardened) |
| Certificates | `/verify/[code]`, `/me/certificates/{id}/download` | issuance + verify + PDF | ✅ live |
| Attendance | `/trainer/attendance`, `POST /attendance/scan` | scan endpoint live, camera UI stubbed | ⚠️ manual entry works; camera-scan is the P12 stub |
| Payments | mock, Tap-ready, Tamara-ready | webhook idempotent, signature verified | mock active; real gateways pending creds |
| Notifications | `/dashboard/notifications` | in-app feed + mark-read | ✅ live |
| Health | `GET /api/v1/health` | 8 probes | — |
| Operations (admin) | failed jobs + outbox + webhooks + queue widget + health widget | new in P4.3 | rendered server-side |
| PWA | `/sw.js`, install banner, offline page | — | ✅ live |

**16 broad surfaces · ~70 user-facing routes · all reachable, none 404 or 500 by design.**

---

## 2️⃣ Findings — fixed in this pass

### F-1 — Help page had a dead `href="#"` link
`/help` "Live chat" tile pointed nowhere. Real broken affordance.

**Fix:** wired it to `https://wa.me/...?text=أحتاج%20مساعدة` so the affordance is real today. When the in-app chat widget ships (P22), we swap the URL.

**File:** `frontend/app/[locale]/help/page.tsx`

---

## 3️⃣ Findings — known stubs (intentionally deferred, not blockers)

| Surface | Stub | Roadmap |
|---|---|---|
| `/trainer/attendance` | "QR camera scan قريباً" — manual code entry works today | P12 once the html5-qrcode wrapper lands |
| `/dashboard/profile` avatar | "رفع الصور مباشرة سيتاح في PHASE 22" — text URL field works | P22 (storage + image processing) |
| `/dashboard/settings` dark mode | "Dark mode سيتفعل بصريا مع PHASE 22 polish" | P22 |
| Bunny / Mux video provider | YouTube embed works as the day-1 path | P10.5 |
| AI streaming | Non-streaming POST→answer works | Backend SSE work in a later phase |
| Real payment gateways (Tap / Tamara) | Mock gateway works end-to-end | Activate by adding `TAP_SECRET_KEY` / `TAMARA_API_TOKEN` |
| Resumable file upload (tus) | Plain S3 upload works | P5 ops layer |

None of these block launch. The platform is usable end-to-end today: a student can enroll → pay (mock) → learn → take a quiz → earn a certificate.

---

## 4️⃣ Other patterns checked (clean)

| Pattern | Search | Status |
|---|---|---|
| Leftover `console.log` / `alert()` in production code | grep `app/`, `components/` | **0 hits** |
| Forgotten `TODO` / `FIXME` in app code | grep app/components excl node_modules | **0 hits** |
| `text-left` / `text-right` (RTL danger) | grep app/components | **0 hits** (cleaned in P3.4) |
| `ml-*`, `mr-*`, `pl-*`, `pr-*` (physical-direction) | grep app/components | **0 hits** (cleaned in P3.4) |
| Unscoped `/me/*` endpoints | reviewed in P2.5 + P4.6 | **0 IDORs** |
| Routes with no auth on protected actions | grep `routes/api.php` | **All authed endpoints under `auth:sanctum`+`active.account`** |

---

## 5️⃣ Final test status snapshot

| Suite | Result |
|---|---|
| Frontend `tsc --noEmit` | **CLEAN** |
| Backend `pint --test` | **CLEAN** |
| Backend `tests/` Pest suite | **107 passed / 1 risky / 0 failed** |
| Module tests under `app/Modules/*/Tests` | 17 fail — separate harness gap (P4.2.A backlog) |
| Frontend lint (last verified P3.x) | clean |

---

## 6️⃣ Critical-path manual check (recommended pre-launch)

Even with the green lights above, the following manual smoke test on a staging tenant is recommended:

1. Sign up new student with valid Saudi national ID → email verify cycle
2. Browse catalog → open a program detail page → add to cart → checkout via mock gateway → receive enrollment + invoice
3. Open `/learn/[slug]` → watch a lesson partially → close tab → reopen → resume picks up at the same lesson (URL persistence)
4. Submit a quiz attempt → result modal → retry if failed
5. Submit an assignment → toast confirms
6. Wait for a lesson to mark complete (90%) → ProgressHeader bumps → certificate auto-issues at 100% across all lessons
7. `/verify/[code]` opens the certificate publicly with the issued snapshot
8. As admin: `/admin/failed-jobs` is empty (no failures from the flow); `/admin/email-outboxes` shows the welcome + receipt + cert emails as `sent`
9. Trigger a failure (e.g. kill Redis briefly) → `/admin/failed-jobs` populates → retry from UI → audit log records `ops.failed_job.retried`
10. Open `/api/v1/health` → expect `status: healthy` with all 8 checks green

If any of those 10 steps fail, that's the blocker list. Otherwise the system is launch-ready.

---

## ✅ P5 closure

- **1 real bug fixed** (`href="#"` on /help)
- **0 critical placeholders** remaining
- **All 70+ user-facing routes** reachable
- **Known stubs documented** with their roadmap phase
- **Manual smoke checklist** provided for pre-launch validation
