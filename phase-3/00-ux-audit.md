# Phase 3 — UX Audit of Current Filament Admin

**Date:** 2026-05-15. **Status:** Analysis only — no code changes from this document.

This is what exists today, what works, and what hurts a non-technical operator running an academy day-to-day. Phase 3 sequencing decisions are downstream of this audit.

## 1. What the operator actually has access to today

### `/admin` panel (tenant — academy_owner / admin / instructor / etc.)

| Resource | Module | Form shape | Operational maturity |
|---|---|---|---|
| Program | Catalog | 5-tab form (Basics / Details / Pricing / Media / Publish) | Wizard-ish, but no Draft→Preview→Publish flow |
| Category | Catalog | Flat form | CRUD only |
| Lesson | Learning | 4-section form (Context / Content / Media / Settings) | No drag-drop ordering, no module grouping in UI |
| CohortBatch | Learning | Flat form | CRUD only |
| Quiz | Assessment | Form + nested Repeater (Q→choices) | Decent, but no question bank, no preview |
| Assignment | Assessment | 4-section form | Separate `/grade` page for scoring (bouncing) |
| Order | Commerce | Read-mostly (form returns `[]`) + refund actions | Operational, but no inline customer timeline |
| Invoice | Commerce | Table-only, no form | CRUD only |
| Coupon | Commerce | 3-section form (Details / Discount / Scope+Usage) | Decent, but no campaign analytics |
| Review | Engagement | Flat form + publish/hide/reply | Moderation actions present |
| ContactMessage | Engagement | Read-only form + status | No "quick reply" channel |
| LiveSession | LiveSessions | 4-section form | Stores Zoom URL as string only — no SDK |
| FailedJob | Operations | Read-only + retry / bulk-retry | Operational |
| EmailOutbox | Operations | Read-only + retry | Operational |
| PaymentWebhook | Operations | Read-only + replay | Operational |
| TenantAuditLog | Operations | Read-only + JSON inspect | Strong, but no "rollback" |

Widgets on `/admin` dashboard: 8 (AdminStats, InstructorStats, TodayPending, RevenueChart, TopPrograms, HealthStatus, QueueDepth, RecentActivityFeed).

### `/super` panel (landlord — Wamadat platform staff)

4 resources (Tenant, SystemUser, Plan, AuditLogCentral) — used for SaaS multi-tenant management. Not directly relevant to academy daily ops, but exists.

### Custom pages

- `TrashPage` (admin + super) — 8-entity soft-delete recovery with restore + permanent delete + audit. Strong feature.
- `GradeAssignment` page — separate page per record for scoring.

## 2. Pain points — daily-use friction for the academy owner

Categorized, each tagged with severity (🔴 daily blocker / 🟡 weekly friction / 🟢 nice-to-have):

### A. Multi-step flows scattered across pages
- 🔴 **Refund approval requires two people on the same table row.** First operator requests; second approves via "اعتماد استرداد معلق". If first walks away, refund sits invisible until second happens to look. No async notification.
- 🟡 **Assignment grading bounces to a separate page.** Operator clicks "تصحيح" → new page. Cannot batch-score 5 submissions from one list.
- 🟡 **Lesson ordering inside a program is form-based,** not drag-drop. Re-sorting 20 lessons = 20 form opens.

### B. CRUD-only areas (where operator wants flows, not forms)
- 🔴 **Program creation** is a 5-tab form with all fields visible at once. No "save draft", no "preview as student", no "publish checklist".
- 🔴 **Lesson creation** is also flat-form. The 7 lesson types (video / audio / pdf / text / quiz / assignment / live) share one form, with conditional fields buried.
- 🟡 **Coupon scope assignment** uses raw JSONB key-value. Operator has to know to write `{"program_ids": ["uuid",...]}`. No picker UI.
- 🟡 **Plan quotas** also use KeyValue input. Same issue.
- 🟡 **Tenant branding/settings** use KeyValue. Brand colors should be a color picker, not a text key.

### C. Fake / partial flows (UI exists but doesn't reflect backend capability)
- 🔴 **Order resource** has form returning `[]` — disabled fields only, no view of line items, no view of related enrollments, no customer timeline. Operator has to query DB to investigate. Backend has rich data, UI doesn't surface it.
- 🟡 **PaymentWebhook resource** shows raw JSON payload — not a digestible "webhook timeline per order".
- 🟡 **Invoice resource** has no form, no line items view. Backend has the data.

### D. Dangerous admin flows (high-stakes ops with weak guardrails)
- 🔴 **Program "منشور" toggle** is live the moment it's set. Instructors can publish unfinished content. No approval workflow despite code comments referencing one.
- 🔴 **Refund amount default = full total.** If operator hits "استرداد" without editing the amount, full refund is queued. Confirmation modal exists but doesn't ask "partial or full?" as an explicit choice.
- 🟡 **Refund approval gate is role-based, not transaction-based.** Two finance officers can mutually approve each other's refunds — no separation-of-duty check.
- 🟡 **No "dry run" for refund** — clicking refund immediately calls the gateway. No simulation.

### E. Duplicated / overlapping concerns
- 🟡 **Two audit log tables** (AuditLogModel tenant + AuditLogCentralModel landlord) — admins on /super and /admin can't see each other's actions. For Closed Beta single-tenant, this duplicates effort.
- 🟡 **Notifications** flow through `NotificationModel` (in-app) but also `EmailOutboxModel` (transactional email) but also `notification_templates` (template registry). Three places to look when an email doesn't reach a user.

### F. Operational bottlenecks (slow tasks for the operator)
- 🔴 **Adding a new program with content takes ~20+ separate form saves** (program → category → lessons one-by-one → quiz → assignment → publish). No "bulk import", no template, no clone-existing.
- 🟡 **Searching across resources** is per-resource. No global search.
- 🟡 **Filter UX** on tables: filters exist but aren't sticky / saved per-user.

### G. Missing dashboards (operator-level visibility gaps)
- 🔴 **Abandoned cart panel** doesn't exist as a UI. Backend has `OrderReconciliationService` marking carts abandoned after 24h — operator can't see them.
- 🔴 **Failed payment funnel** — payments table has status field, but no UI cross-section (e.g., "all 'failed' payments last 7 days grouped by gateway + reason").
- 🟡 **Coupon performance dashboard** — `CouponRedemptionModel` tracks usage, but operator has no UI showing "which coupon drove which revenue".
- 🟡 **Live session attendance** — `AttendanceRecordModel` exists, but no UI showing per-session attendance %.
- 🟡 **Email deliverability** — `EmailOutboxModel` tracks bounces/complaints, no UI surfacing this signal.

### H. Missing quick actions (one-tap operator tasks)
- 🔴 **No "resend payment link"** from order row.
- 🔴 **No "apply coupon manually"** from order row.
- 🟡 **No "quick reply"** from ContactMessage row.
- 🟡 **No "publish program now"** primary CTA — buried in tab navigation.
- 🟡 **No "clone program"** action.

## 3. What works well today (don't break these in Phase 3)

- ✅ **Audit log inspector** — JSON before/after view with context. Solid foundation, just needs wider adoption.
- ✅ **Refund + retry + replay confirmation modals** — gated by role, capture reason. Pattern to reuse.
- ✅ **Trash + recovery page** — 8 entities, restore + permanent delete + audit log entry. Strong.
- ✅ **Failed-job retry** (single + bulk).
- ✅ **2FA enforcement on AcademyOwner + Admin** — `RequireAdminTwoFactor` middleware. Phase 3 should preserve this.
- ✅ **Spatie roles + permissions** (9 roles seeded) — already in place; Phase 3 should layer on, not replace.
- ✅ **Multi-tab forms on Program / Plan / Tenant** — better than single long form. Phase 3 should evolve these into proper wizards.

## 4. Operator pain summary (non-technical reading)

If you sit at this admin for a full day as an academy owner, three things will frustrate you:

1. **Adding a new program with full content is a 20-form marathon.** You'll switch screens for the program, then for each lesson, then for the quiz, then for the assignment. There is no "guided builder" — there's a flat catalog of CRUD forms you tile together yourself.

2. **You cannot see what your customer sees.** No "preview as student" anywhere. No mobile preview. Landing pages don't exist as a manageable thing — they live in the frontend code. Reviews don't have a moderation queue you scan, just a flat list.

3. **High-stakes operations look the same as low-stakes ones.** A refund and a category rename use the same edit-form-then-save pattern. The refund has a confirmation, but it doesn't visibly distinguish "this will move money out of your account in 3 seconds." Operator pace is the same for both — and that's a long-term safety issue.

Everything else is solvable. These three shape the Phase 3 priority list.
