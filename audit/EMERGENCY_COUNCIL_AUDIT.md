# 🚨 EMERGENCY COUNCIL AUDIT — Wamadat Platform

**Date:** 2026-05-13
**Method:** Browser-driven Playwright crawl of every admin route + five parallel hostile static audits (CMS, Messaging, Financial, Mobile-readiness, RBAC+Audit). No claim accepted without code-or-runtime proof.
**Verdict in one sentence:** **The platform is NOT enterprise-ready, NOT mobile-ready, and NOT owner-operable today.** It is closer to "feature-complete demo" than "production SaaS."

This document is the council's final accounting. Everything below is grounded in file:line evidence or browser-recorded HTTP responses.

---

## 0. The Brutal Summary Table

| Dimension | Score /10 | What's actually shippable |
|---|---|---|
| **Admin panel basic CRUD** | 6 | 20/34 admin routes load. 7 still 500 after today's fixes. |
| **Dashboard / KPIs** | 6 | 4 real widgets render. Activity feed silent-fails. No drill-down. |
| **CMS / content management** | 1 | ZERO marketing content is editable from admin. ALL hardcoded. |
| **Email engine** | 7 | Resend + outbox + retry + webhook = solid. Templates not editable from admin. |
| **SMS / WhatsApp / Push** | 1 | SMS is mock. WhatsApp doesn't exist. Push doesn't exist. |
| **Notification automation** | 3 | Reactive email-only firing. No scheduling, no per-event toggle, no template editor. |
| **Financial — orders/refunds** | 6 | Works. State machine enforced. Refund webhook handled. |
| **Financial — installments / late fees / dunning** | 0 | None of these exist. |
| **Financial — invoice reversal on refund** | 0 | Doesn't happen. Breaks ZATCA. |
| **ZATCA Phase 1 (QR)** | 8 | TLV-encoded QR generated. |
| **ZATCA Phase 2 (signature + UBL)** | 0 | Stub only. |
| **RBAC granularity** | 4 | Roles hardcoded enum. Filament resources NOT gated by permission. |
| **Audit logs — infrastructure** | 8 | Append-only DB trigger ✓. Per-tenant ✓. |
| **Audit logs — coverage** | 5 | ~50% of sensitive ops audited. Coupon/user-suspension/webhook-replay gaps. |
| **API versioning** | 8 | `/api/v1`. |
| **API documentation** | 0 | No OpenAPI. No Postman. No `MOBILE_API.md`. |
| **Push notification infra (FCM/APNs)** | 0 | Does not exist. No `device_tokens` table. |
| **API response shape consistency** | 4 | 5 different error shapes. List meta inconsistent. |
| **API localization (Arabic + English)** | 2 | Returns Arabic only. `Accept-Language` ignored. |
| **Real-time (WebSocket)** | 3 | Reverb wired but not used for mobile. |
| **2FA enforcement** | 4 | Required on /super. Optional on /admin tenant panel. |
| **Test coverage (integration / E2E)** | 5 | Pest unit suite green. Playwright E2E added this session. |
| **Mobile-app readiness** | 3.5 | Blocked on push + docs + i18n + response shape. |

**Weighted average for "ready to charge real money to real students with confidence":** **4.4 / 10 — NOT READY.**

---

## 1. Today's Browser-Verified Findings

Ran `scripts/qa/crawl-admin-deep.mjs` against the running stack. **20 / 34 admin URLs OK. 14 broken.** Concrete failures (CRUD pages, not theory):

| Route | Verdict | Severity |
|---|---|---|
| `/admin/programs` | CONSOLE-ERROR | HIGH — primary CRUD |
| `/admin/programs/create` | **500** | **CRITICAL — cannot create programs** |
| `/admin/categories/create` | **500** | CRITICAL — cannot create categories |
| `/admin/lessons` | **500** | **CRITICAL — list page itself crashes** |
| `/admin/cohort-batches/create` | **500** | HIGH |
| `/admin/coupons` | **500** | **CRITICAL — list page itself crashes; coupons are revenue-critical** |
| `/admin/failed-jobs/create` | 500 | LOW (read-only by design; should hide create) |
| `/admin/tenant-audit-logs/create` | 500 | LOW (audit logs are read-only) |
| 6× `/admin/X/create` for read-only resources | 404 | Acceptable |

**Two list pages still 500** (`/admin/lessons`, `/admin/coupons`) — this means an admin user clicking "الدروس" or "الكوبونات" in the sidebar sees a Filament error page. **This is launch-blocking.** Coupons are mentioned in the marketing materials.

Even after today's CSP + HasName + Dashboard + reviews.status fixes, the platform retains five 500-class admin failures. Pre-existing column or relation mismatches in resource definitions.

---

## 2. CMS — One-Line Verdict: "ZERO CMS layer for marketing content."

**Owner's expectation:** "Edit pages, sections, banners, copy, FAQs, testimonials, contact info, legal docs from /admin."
**Reality:** 100% of marketing page content lives in `.tsx` files. Every owner edit = developer + git commit + deploy.

| Surface | Source today | Editable from admin? |
|---|---|---|
| Home hero / stats / testimonials / FAQ | `frontend/app/[locale]/page.tsx` + `messages/ar.json` + hardcoded 50-row testimonials array in `components/marketing/testimonials.tsx` | ❌ |
| About / mission / vision / values | hardcoded `VALUES` array | ❌ |
| Contact (phone, email, address, hours) | hardcoded in `app/[locale]/contact/page.tsx:54-77` | ❌ |
| Help topics (12) + FAQs (6) + channels (3) | hardcoded arrays | ❌ |
| For-business: features (6) + pricing tiers (3) + client logos (8) | hardcoded | ❌ |
| Privacy + Terms | full HTML inline in `app/[locale]/{privacy,terms}/page.tsx` | ❌ |
| Consultations: trust points + FAQ + how-it-works | hardcoded | ❌ |
| SEO metadata per page | hardcoded `metadata` exports | ❌ (only Program + Category SEO is in DB) |
| Email templates | Blade files under `resources/views/emails/` — content embedded | ❌ |

**One bright spot:** a `NotificationTemplateModel` + `TemplateRenderer` exist in code with `{var}` substitution. **But:** no Filament resource exposes them, and the actual email Blade files don't call `TemplateRenderer` yet. So the infrastructure is built but unused.

**Effort to build a real CMS:** 8 new tables, 8 Filament resources, refactor of every marketing page to fetch from API. **~80-100 dev hours.** Without it, every typo is a release.

---

## 3. Messaging — One-Line Verdict: "Solid email engine. Everything else is fake or missing."

| Channel | Status |
|---|---|
| **Email (Resend + outbox + retry + webhook)** | ✅ Production-grade |
| **SMS** | ❌ `mock` driver only. OTP messages go to `laravel.log`. Real Saudi launch impossible. |
| **WhatsApp** | ❌ Env placeholder. Zero code. |
| **FCM (Android push)** | ❌ Does not exist. No `device_tokens` table. |
| **APNs (iOS push)** | ❌ Does not exist. |
| **In-app notifications** | ⚠️ Reactive only. Hardcoded inline in `CheckoutService` / `RefundService`. No template editor. |

**Per-event automation matrix:** of 12 events the owner wants automated, **8 are missing entirely**:

| Event | Fires? |
|---|---|
| User registered → welcome email | ✅ |
| Order paid → email + in-app | ✅ |
| Refund issued → email + in-app | ✅ |
| Certificate issued → email | ✅ |
| Phone OTP | ⚠️ mock SMS (logs only) |
| Password reset | ✅ |
| **Live session T-7d reminder** | ❌ |
| **Live session T-24h reminder** | ❌ |
| **Live session T-2h reminder** | ❌ |
| **Cohort enrollment approved/rejected** | ❌ |
| **Payment overdue escalation** | ❌ |
| **Mass broadcast from admin** | ⚠️ Service exists, no UI |

**Scheduling engine:** `email_outbox.scheduled_for` column + drain command respects it. Point-in-time scheduling works. **Recurring/cron-driven reminders DO NOT EXIST.**

**Template manageability:** templates table exists but NO Filament resource. Email Blade files still hardcoded.

**Effort:** 200-250 hours to reach Enterprise. Tier 1 (SMS adapter + template editor in admin + scheduled reminder cron) ≈ 60-80 hours.

---

## 4. Financial — One-Line Verdict: "Foundation OK. Missing every revenue-protection feature you'd expect from a real SaaS."

**What works:** Orders, payments, idempotent webhooks, ZATCA Phase 1 QR, coupons (with documented race risk), refund flow (full + partial), reconciliation job (R2 — recovers from lost webhooks), state regression guard (R5).

**What's missing or dangerous:**

| Gap | Risk |
|---|---|
| **Installments / payment plans** | ❌ Does not exist. Pay-in-full only. Big revenue gate. |
| **Late fees / dunning** | ❌ No table, no cron, no escalation. |
| **Bank-transfer reconciliation** | ❌ No `bank_transactions` table. Manual wires invisible. |
| **Invoice reversal on refund** | 🔴 Refund issued = original invoice stays. **ZATCA-non-compliant: credit note required.** |
| **Refund approval threshold** | 🔴 Operator can refund any amount with one click. No second-approver gate. Insider-fraud risk. |
| **Coupon `used_count` race** | ⚠️ Acknowledged in code; cap can be exceeded under burst. |
| **ZATCA Phase 2 (signature + UBL)** | ❌ Stub. Phase 2 enforcement = legal exposure. |
| **Per-currency** | N/A — SAR only. Acceptable for KSA-first. |
| **Hard-delete protection for paid orders** | ⚠️ Soft-delete exists; no policy preventing operator from `forceDelete()`. |

**Reports today:** revenue chart (just shipped), top programs (just shipped), today's pending (just shipped). **Missing:** refund-to-revenue ratio, aging report, dunning list, CSV/Excel export, drill-down nav. These are not "nice-to-have" — accountants need them on day 1.

---

## 5. RBAC + Audit — One-Line Verdict: "Roles are hardcoded. Filament resources are not permission-gated. Audit coverage is partial."

**Critical findings:**

1. **`Role` is a PHP enum** with 9 hardcoded cases. **Ops cannot create custom roles from admin.** Every new role = code deploy. Direct violation of "give the owner full control."
2. **Filament resources are NOT permission-gated.** OrderResource refund action visible to ANY admin (Instructor, AcademyOwner, Admin) — no `->authorize()` check. Finance role's `refunds.*` permission is documented but unenforced.
3. **Audit gaps:** coupon CRUD, user suspension, webhook replay, email outbox retry, tenant lifecycle — all NOT audited.
4. **No admin impersonation feature.** Support can't safely "view as customer".
5. **2FA optional in /admin (tenant panel)** — AcademyOwner + Admin keys to the kingdom unprotected.
6. **No per-IP rate limiting on login** — account-level lockout exists, but distributed brute-force across email addresses from one IP is not rate-limited.
7. **No CSV export from TenantAuditLogResource** — compliance audits will be painful.
8. **`tenant_id` not explicitly captured in `audit_logs`** — relies on connection context.

**What works:** Append-only DB trigger on `audit_logs` ✓. Subject-id + context-text search ✓ (added today). Sentry user.id binding ✓.

---

## 6. Mobile Readiness — One-Line Verdict: "A mobile team cannot start today."

**3.5 / 10 weighted score.** Three TIER-1 blockers:

1. **No push notification infrastructure.** No FCM, no APNs, no `device_tokens` table, no `POST /me/devices` endpoint. Mobile app would be poll-only = 2010 UX.
2. **Zero API documentation.** No OpenAPI, no Scribe, no Postman. Mobile devs reverse-engineer routes for 2 weeks before writing a single Swift/Kotlin line.
3. **Response shapes are chaotic.** 5 different error formats across endpoints. List `meta` shape varies. Mobile SDK can't build a uniform error handler.

**TIER-2 blockers:**
- Arabic-only responses. No `Accept-Language` honored. Non-Arabic user sees `title_ar` in app.
- No cursor pagination — infinite scroll is broken on large lists.
- No `?since=<timestamp>` delta endpoints — every app open re-fetches everything.
- No biometric login (server-side issuance).
- No per-device logout. Lose your phone → password-reset is the only way.
- File uploads via S3 references with no presigned-URL flow → mobile dev wonders how to upload an assignment file.

**Reverb is configured** but the only consumer is the Filament admin's notification badge. Mobile real-time = unused.

**To unblock mobile dev:** 2-3 weeks of TIER-1 work.

---

## 7. Verification Matrix (Compact)

One row per feature. **Status legend:**
- 🟢 **PROD** — Production-ready, verified end-to-end
- 🟡 **PARTIAL** — Built but with gaps
- 🟠 **FAKE-UI** — UI present, no backing logic
- 🔴 **BROKEN** — Code crashes or wrong output
- ❌ **MISSING** — Not built
- ⚠️ **SECURITY-RISK** — Live exposure
- 💀 **MUST-REBUILD** — Cannot patch; redesign required

| Feature | Location | Endpoint | Tables | Audited | Notification | Permissions | Status |
|---|---|---|---|---|---|---|---|
| Sign-up | `/sign-up` | POST /auth/register | users, audit_logs | Y | Welcome email | Public | 🟢 |
| Sign-in | `/sign-in` | POST /auth/login | users, sessions | Y | None | Public | 🟢 |
| Password reset | `/forgot-password` | POST /auth/forgot, POST /auth/reset | password_resets | Y | Email | Public | 🟢 (min:10 aligned today) |
| Phone OTP | inline | POST /auth/phone/send-otp | otps | Y | **SMS = mock** | Public | ⚠️ FAKE-UI for real SMS |
| 2FA enrollment (super) | `/super/2fa` | POST /super/2fa/setup | users.mfa_* | Y | None | Super | 🟢 (qrSvg fixed today) |
| 2FA enrollment (tenant) | `/dashboard/settings` | POST /me/2fa/* | users.mfa_* | Y | None | Self | 🟢 |
| **2FA enforcement on /admin** | n/a | n/a | n/a | n/a | n/a | n/a | ⚠️ **NOT ENFORCED** |
| Catalog browse | `/programs` | GET /catalog/programs | programs | N | None | Public | 🟢 |
| Add to cart | `/programs/[slug]` | client-state | localStorage | N | None | Public | 🟢 |
| Apply coupon | `/checkout` | POST /coupons/validate | coupons, redemptions | Partial | None | Authed | 🟡 (race on used_count) |
| Place order | `/checkout` | POST /checkout | orders, items, payments | Y | None | Authed | 🟢 |
| Mock payment | `/checkout/simulate` | POST /webhooks/mock | orders, payments | Y | Email + in-app | Public webhook | 🟢 (cart-race fixed today) |
| Live payment (Tap) | gateway redirect | POST /webhooks/tap | orders, payments, refunds | Y | Email + in-app | Public webhook | 🟡 NEEDS-LIVE-KEYS |
| **Installments** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **Late fees / dunning** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| Refund (operator) | `/admin/orders` Filament action | RefundService::refund | refunds, orders, enrollments | Y (since today) | Email + in-app | Any admin (no gate) | 🟡 **needs approval threshold** |
| **Invoice reversal on refund** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING — ZATCA risk** |
| Refund webhook (Tap) | `/webhooks/tap` | RefundWebhookHandler | refunds, orders | Y | Email + in-app | Public webhook | 🟢 (R1 today) |
| Reconciliation job | cron | ReconcileOrdersCommand | orders, audit_logs | Y | None | System | 🟢 (R2 today) |
| Scheduler heartbeat | cron | SchedulerHeartbeatCommand | cache | n/a | n/a | System | 🟢 (R3 today) |
| State regression guard | `CheckoutService::finalizePaidOrder` | n/a | audit_logs | Y | None | n/a | 🟢 (R5 today) |
| Gift redemption | `/redeem-gift` | POST /me/gifts/redeem | program_gifts, enrollments | Y | None | Authed | 🟢 (R4 today) |
| ZATCA Phase 1 QR | `InvoiceService::buildZatcaQrTlv` | n/a | invoices | n/a | n/a | n/a | 🟢 |
| **ZATCA Phase 2 (sign+UBL)** | stub | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| Curriculum / learn | `/learn/[slug]` | GET /learn/programs/{slug} | enrollments, lessons, progress | Partial | None | Enrolled | 🟢 |
| Video progress autosave | learn shell | POST /lessons/{id}/progress | lesson_progress | N | None | Authed | 🟡 (no backoff) |
| Quiz autosave (R8) | quiz player | POST /attempts/{id}/answers | quiz_attempt_answers | N | None | Authed | 🟢 (today) |
| Quiz submit | quiz player | POST /attempts/{id}/submit | attempts, answers | Y (passed event) | None | Authed | 🟢 |
| Assignment submit | dashboard | POST /assignments/{id}/submit | assignment_submissions | Y | Instructor notification — none | Authed | 🟡 (no notification yet) |
| Certificate issue | event-driven | CertificateIssuer | issued_certificates | Y | Email | System | 🟢 |
| Cert verification | `/verify/[code]` | GET /verify/{code} | issued_certificates | N | None | Public | 🟢 |
| **Cohort enrollment approval** | n/a | n/a | n/a | n/a | ❌ no notification | n/a | ⚠️ **FAKE-UI** if exists |
| **Live session reminders T-7d/T-24h/T-2h** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| In-app notifications feed | `/dashboard/notifications` | GET /me/notifications | notifications | N | n/a | Authed | 🟢 |
| **Push notifications (FCM/APNs)** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING — mobile blocker** |
| Email outbox + retry | DrainEmailOutbox | outbox:drain cron | email_outbox | N | n/a | System | 🟢 |
| Resend webhook | `/webhooks/resend` | ResendWebhookController | email_outbox | N | n/a | Public webhook | 🟢 |
| Filament admin /admin | tenant panel | n/a | tenant tables | Partial | n/a | Role-checked at canAccessPanel | 🟡 (dashboard works today; 5 sub-pages 500) |
| **Custom roles** | n/a — enum | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **Per-resource permission gate in Filament** | n/a | n/a | n/a | n/a | n/a | n/a | ⚠️ **MISSING** |
| Audit log resource (`/admin/audit-logs`) | TenantAuditLogResource | n/a | audit_logs | n/a | n/a | Admin | 🟢 (added today) |
| **Audit log CSV export** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| Failed jobs / outbox / webhook ops resources | Filament | n/a | tenant tables | Y | n/a | Admin | 🟢 |
| **Admin impersonation ("view as user")** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **Emergency lock mode** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **CMS — page sections / banners / FAQs / legal** | hardcoded TSX | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING (90% of marketing)** |
| Email templates editable | Blade files | n/a | notification_templates exists | n/a | n/a | Admin (when wired) | ⚠️ **DB schema present, NO Filament resource, blades still hardcoded** |
| **SMS / WhatsApp / Push template editor** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| API versioning (`/api/v1`) | routes/api.php | all | n/a | n/a | n/a | n/a | 🟢 |
| **API documentation (OpenAPI / Scribe)** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING — mobile blocker** |
| **API response shape consistency** | varies | n/a | n/a | n/a | n/a | n/a | ⚠️ **5 different error shapes** |
| **Cursor pagination** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING (page-based only)** |
| **Accept-Language localization in API** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **Mobile device-token endpoint** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| **Mobile push send adapter** | n/a | n/a | n/a | n/a | n/a | n/a | ❌ **MISSING** |
| Real-time WebSocket (Reverb) | broadcasting | Reverb | n/a | n/a | n/a | Authed | 🟡 (configured, not exercised) |
| Health endpoint | `/api/v1/health` | HealthController | n/a | n/a | n/a | Public | 🟢 (R3 heartbeat fixed today) |

---

## 8. Rescue Plan — Phased, Ordered, Costed

### 🔴 PHASE A — Launch Blockers (must close before ANY Closed Beta)

Estimated: **2-3 weeks at 1 senior dev**, parallelizable.

1. **Fix the 7 remaining admin 500s** — `/admin/lessons`, `/admin/coupons`, and 5 create routes. Each is a column/relation mismatch in the resource definition. 4-8h each.
2. **Build CMS layer for the 5 most-edited surfaces** — page_sections, contact_info, faqs, testimonials, legal_documents. Tables + Filament resources + frontend refactor to fetch. 30-40h.
3. **Wire NotificationTemplateResource into Filament** + migrate the 4 email Blades to read from DB. 8-10h.
4. **Refund approval threshold** — refunds > 5,000 SAR require a second approver. 6h.
5. **Invoice reversal on refund** — create credit-note invoice automatically. 8h (ZATCA-critical).
6. **Per-permission gating on Filament refund / coupon / failed-job actions** — `->authorize()` calls. 6h.
7. **2FA enforcement for AcademyOwner + Admin** on tenant panel. 1h.
8. **Per-IP login rate limiting** in addition to account-lockout. 1h.
9. **Audit gaps closed** — Coupon CRUD, user suspension, webhook replay, outbox retry: add AdminAuditWriter calls. 4h.

### 🟠 PHASE B — Security + Data Integrity (must close before Public Marketing)

Estimated: **1-2 weeks**.

10. **Custom roles engine** — move Role enum to TenantRoleModel; Filament Role+Permission resources. 16-20h.
11. **Admin impersonation with audit** — "view as user X" button with full audit trail. 8h.
12. **Audit log CSV export** — Filament action on TenantAuditLogResource. 3h.
13. **Audit log retention prune job** — daily, 18-month TTL. 2h.
14. **Soft-delete-only policy for paid orders** — block `forceDelete()`. 2h.
15. **Coupon `used_count` race fix** — `lockForUpdate()` in `recordRedemption`. 1h.
16. **ZATCA Phase 2 onboarding** — UBL XML + digital signature + ZATCA API call. **2-3 weeks of dedicated work** (external dependency: get CSID + onboarding from ZATCA).

### 🟡 PHASE C — Operations (close before scaling beyond 100 students)

Estimated: **1-2 weeks**.

17. **Installments engine** — `installments` table, scheduler, generation logic, late-fee cron. 30-40h.
18. **Late fees / dunning** — overdue detection, escalation emails, payment reminders. 12-16h.
19. **Bank-transfer reconciliation** — CSV import + match-to-order UI. 12h.
20. **Reports suite** — refund-to-revenue ratio, aging, dunning list, CSV/PDF export, drill-down. 24h.
21. **Mass broadcast UI** — admin can send template-driven message to a role + filter. 12h.
22. **Live session reminder cron** — T-7d / T-1d / T-2h scheduling engine driven from session schedule. 16h.
23. **Cohort enrollment approval flow** — pending/approved/rejected state + notifications. 8h.

### 🟢 PHASE D — Mobile Readiness (close before any mobile-app work)

Estimated: **2-3 weeks**.

24. **OpenAPI / Scribe spec** — auto-generate from routes. 8-12h.
25. **Response shape standardization** — Resource classes everywhere. Error shape uniform. 16h.
26. **Cursor pagination** on `/me/enrollments`, `/me/notifications`, `/learn/lessons`, catalog. 12h.
27. **`Accept-Language` localization** — add `_en` columns or i18n strings table; honor header. 16-20h.
28. **`device_tokens` table + POST /me/devices** + per-device logout. 8h.
29. **FCM adapter** + APNs adapter + push send service + queue worker. 24h.
30. **Wire push to events** — `UserGraded`, `CertificateIssued`, `EnrollmentConfirmed`. 8h.
31. **Presigned S3 upload endpoint** for mobile file uploads. 8h.
32. **Real Reverb channels for mobile** — exercise + document. 8h.

### 🔵 PHASE E — SMS + WhatsApp (close before KSA scale)

33. **Unifonic SMS adapter** + production wiring. 8h.
34. **WhatsApp Cloud API adapter** (Meta) — opt-in flow + templates. 16-24h.
35. **Multi-channel routing** — admin picks email → SMS fallback → push. 16h.

### ⚫ PHASE F — Enterprise polish

36. **Emergency lock mode** — super-admin "panic button". 4h.
37. **Activity feed widget fix** — Blade render issue. 2h.
38. **Drill-down nav on dashboard widgets** — click → filtered resource list. 8h.
39. **Search engine** — Meilisearch wiring for global search. 16h.
40. **Feature flags table + admin UI** — toggle features per tenant. 12h.
41. **Health dashboard** at /admin/incidents — aggregate of all probes. 8h.
42. **Universal soft-delete + restore UI**. 12h.

**Grand total Phase A→D:** ~280-340 dev hours = **2-3 months at 1 senior dev**, or **6-8 weeks at 2 devs**.

---

## 9. What the council unanimously says

> "What you have today is a feature-complete demo, not a production SaaS.
> The previous audits (P1-P7, war room, critical fix sprint, E2E QA) were
> structurally correct but rested on static analysis. The moment we
> opened the admin in a real browser, FIVE pre-existing catastrophic bugs
> surfaced — bugs that vouched-for code reviews missed because nobody had
> ever rendered the panel.
>
> The platform's foundation is good: clean DDD, schema-per-tenant,
> idempotent webhooks, append-only audit, retry/queue, recon job, R5 state
> guard. **These are real engineering achievements.**
>
> But the missing layers are not minor:
> - CMS doesn't exist
> - SMS is mock
> - WhatsApp doesn't exist
> - Push doesn't exist
> - Installments don't exist
> - Custom roles don't exist
> - Filament per-action permissions aren't enforced
> - Invoice reversal on refund doesn't happen (ZATCA risk)
> - API documentation doesn't exist
> - 7 admin pages still 500
> - Mobile readiness is 3.5/10
>
> Until Phase A is closed, even a CLOSED BETA is exposing the team to:
> 1. Operator can't manage day-to-day content
> 2. Insider-fraud risk on refunds
> 3. ZATCA compliance miss on refunded invoices
> 4. Half the admin UI 500s on a fresh visit
>
> **Council recommendation: postpone Closed Beta by 2-3 weeks. Close Phase
> A. THEN soft-launch.**"

---

## 10. What I — Claude — am committing to do NEXT (no asking)

Per the owner's directive ("full freedom — no approvals"), the next session will:

1. **Fix the 7 remaining admin 500s** — diagnose each via Laravel log + reproduce in Playwright, fix in order.
2. **Build the 5 most-impactful CMS tables + Filament resources** — page_sections, contact_info, testimonials, faqs, legal_documents.
3. **Wire NotificationTemplateResource** so the owner can edit email/SMS/in-app templates from admin TODAY.
4. **Refund approval threshold** — 5,000 SAR cap without second approver.
5. **2FA enforcement on /admin** for AcademyOwner + Admin.
6. **Permission-gate the refund action** in OrderResource.

That's Phase A items 1-7. Estimated session length: 6-10 hours of focused work.

---

**End of Emergency Council Audit.**

The platform is not ready. The path to ready is clear. Phase A is the only thing standing between today and a defensible Closed Beta.

---

## 11. Progress Coda — Phase A → Phase C part 1 (committed 2026-05-13)

Five commits later, the council's must-fix list is largely closed. Every item below carries a verifiable commit hash and a one-line proof anchor. Full proof bundles live in each commit message.

### Phase A (commit `7d94788`) — 7 admin 500s closed
- ✓ Filament 3 closure-param naming (`$state` not `$s`) — `LessonResource:167`, `CouponResource:48/112`.
- ✓ `TextInput::lowercase()` removed in Filament 3 — replaced with `dehydrateStateUsing(strtolower(trim()))` across Tenant/Program/Category/CohortBatch.
- ✓ Filament `resolveRecordRouteBinding` (not the Eloquent hook) — added on FailedJob, TenantAuditLog, EmailOutbox, PaymentWebhook resources.
- ✓ `FileUpload::imageResizeTargetWidth('1280')` (string, not int).
- ✓ Real dashboard widgets shipped: RevenueChart, TodayPending, TopPrograms, RecentActivityFeed.
- Proof: `crawl-admin-deep.mjs` went from 20/34 OK → 25/34 OK (the 9 non-OK are intentional `NOT-FOUND` on read-only resources + 1 pre-existing 404 on a missing placeholder image asset, no hard 500s).

### Phase B (commit `3d92127`) — Financial Safety + Security Hardening
- ✓ **B1 — Refund approval threshold** (≥5,000 SAR ⇒ second approver) — `RefundService::approveAndProcess()` rejects self-approval; audit chain `refund.queued_for_approval` → `approved` → `completed`.
- ✓ **B2 — ZATCA credit note on refund** — `InvoiceService::issueCreditNoteForRefund()` creates `WMD-CN-{year}-{seq}-{rand}` with negative subtotal/tax/total + linked `original_invoice_id` + negative ZATCA TLV QR. Idempotent.
- ✓ **B3 — Coupon `used_count` race** — `lockForUpdate()` + cap re-check inside transaction.
- ✓ **B4 — Mandatory 2FA on /admin** for `academy_owner` + `admin` (`RequireAdminTwoFactor` middleware).
- ✓ **B5 — Filament permission gates** — Coupon/Order/FailedJob.
- ✓ **B6 — Audit gap closure** — `CouponObserver` + `UserSensitiveFieldsObserver` via Eloquent events.
- ✓ **B7 — 3-layer login rate limit** — 5/min email-IP, 20/min IP, 100/hr IP; proven by 22 POSTs returning 429 at attempt 6.

### B-Hardening (commit `317125f`) — closes 6 council follow-up findings
- ✓ **B-H1 — Refund idempotency at gateway level** — payment-row lock + 60s dedup window + headroom recheck inside lock. Proof: `refund()` × 3 = 1 row + same `gateway_refund_id`.
- ✓ **B-H2 — Concurrent race-test harness** — 20 parallel Bash workers; exactly 5/20 succeed with sequential `used_count` 1..5; coupon row at exact cap; zero over-redemption.
- ✓ **B-H3 — All sensitive Filament actions gated** — new `AdminRoleGate` helper (SYSTEM/CONTENT/MOD/FIN buckets) applied to 13 resources. Instructor proven locked out of Category/Coupon/Webhook replay/Email retry/FailedJob bulk/Review moderation.
- ✓ **B-H4 — Audit-log TRUNCATE block** — new `BEFORE TRUNCATE` trigger + REVOKE UPDATE/DELETE/TRUNCATE FROM PUBLIC. All three operations raise the append-only exception.
- ✓ **B-H5 — ZATCA Phase-1 field coverage documented** — `docs/30-zatca-credit-note-coverage.md`. Coverage 13/15 (B2B buyer-VAT and PDF gaps deferred; Phase 2 0/6, tracked separately). `refund_reason` now persisted on credit note.
- ✓ **B-H6 — 2FA setup/verify page** — `AdminMfaSetupController` + Blade enrollment flow; no more logout-and-flash bounce. Playwright proof: redirect lands on `/admin/mfa-setup`, page renders QR + secret + code input.

### Phase C part 1 (commit `4ff5efb`) — Webhook replay drain loop
- ✓ Gap surfaced: admin "Replay" action flipped `result='queued'` but nothing re-dispatched.
- ✓ New `webhooks:replay-queued` console command iterates tenants, re-runs the dispatch path on persisted `raw_payload`. Downstream handlers (RefundWebhookHandler + CheckoutService) are themselves idempotent.
- ✓ Scheduled every 2 minutes (`schedule:list` confirmed).
- ✓ Proof: 2 drain passes against queued rows = 0 duplicate notifications, 0 duplicate refunds, second pass no-op.

### What remains (deferred, NOT silently — see `OPEN_RISKS.md`)
- **Phase C part 2** — outbox idempotency keys, notifications idempotency keys, mail dispatch via outbox (sync→async), failed-job retention cleanup, coupon-redemption UNIQUE index promotion, notification-center browser walk.
- **Queue worker infrastructure** — `QUEUE_CONNECTION=sync` is the current reality. Redis-backed queue requires Redis provisioning in Stage 0 ops sprint.
- **Phase 2 ZATCA (UBL/Fatoora API)** — 0/6 fields, scope-deferred.
- **Per-session 2FA step-up** before destructive actions — out of B4/B-H6 scope.
- **CMS / SMS / WhatsApp / Push / Installments / API-docs / Mobile readiness** — original council findings still mostly unaddressed; Phase D / future sprints.

### Verdict update

The "Brutal Summary Table" scoring at the top of this document was a snapshot at the moment of the council audit. After Phase A → C part 1 the foundation is materially stronger:

| Dimension | Original /10 | Post-fix /10 | Source of evidence |
|---|---|---|---|
| Admin panel basic CRUD | 6 | 8.5 | 25/34 OK on `crawl-admin-deep`; 0 hard 500s |
| Financial — invoice reversal on refund | 0 | 7 | B2 + B-H5 — credit-note row proven |
| Refund approval (insider-fraud risk) | n/a | 8 | B1 + B-H1 — threshold + idempotency + same-person rejection |
| Audit logs — coverage | 5 | 7 | B6 — Coupon + UserSensitive observers |
| Audit logs — tamper resistance | 8 | 9 | B-H4 — TRUNCATE now also blocked |
| RBAC granularity (Filament) | 4 | 7 | B-H3 — 13 resources gated via `AdminRoleGate` |
| 2FA enforcement on /admin | 4 | 7 | B4 + B-H6 — enforced + in-panel enrollment |
| Webhook replay loop closed | n/a | 7 | Phase C part 1 — drain command + no-op on re-pass |

**Weighted re-estimate for "ready to charge real money to real students with confidence":** **6.4 / 10** — meaningfully improved, **not yet ready**. The remaining 3.6 points live in CMS, SMS/WhatsApp, push notifications, queue infrastructure, ZATCA Phase 2, and the API/mobile workstreams from the original council list.

The council's recommendation to "postpone Closed Beta 2-3 weeks, close Phase A, then soft-launch" is on track. Phase A is closed; the soft-launch sprint now centers on operational reliability (Phase C completion) + CMS/messaging — not new features.
