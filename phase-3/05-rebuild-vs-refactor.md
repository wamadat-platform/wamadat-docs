# Phase 3 — Rebuild vs Refactor Matrix

**Status:** Analysis only. For each Phase 3 subsystem, what's the action against current code? **Keep / Extend / Refactor / Rebuild / New-build.**

## Definitions

- **Keep**: works as-is; no Phase 3 changes.
- **Extend**: add fields/methods without touching existing structure.
- **Refactor**: same data + same model, but reshape internals (cleaner separation, better tests, better UX wrap).
- **Rebuild**: replace existing implementation with a new one; migrate data.
- **New-build**: nothing exists; build from scratch.

The goal of this document: the operator decides per row whether the action level is acceptable. Disagreements drive scope conversation BEFORE coding.

---

## Domain-by-domain decisions

### 1. Programs (Catalog module)
**Current state:** `ProgramModel` + `programs` table with rich fields (status, mode, pricing, SEO JSONB, learning outcomes JSONB, etc.). 5-tab Filament form.

**Action: REFACTOR (model) + REBUILD (admin UX).**
- Model: keep schema. Add `draft_state` JSONB column for the approval workflow (instructor proposes → owner publishes) OR a `program_drafts` table. ← decision needed in Stage 0.
- Admin UX: replace 5-tab form with B1's wizard. Old form retired.
- Data migration: zero — schema additions only.

### 2. Categories
**Current state:** Single-section flat form. Works.

**Action: KEEP (model) + minor REFACTOR (admin UX).**
- Add: parent/child tree picker, sort_order drag-drop, cover_image_url asset picker.
- No model changes.

### 3. Instructors (InstructorProfileModel)
**Current state:** Separate profile model + program FKs. Adequate.

**Action: KEEP (model) + REFACTOR (admin UX).**
- Add: portfolio block on the instructor's public page, link to their consultant profile if exists.

### 4. Lessons + Modules
**Current state:** `LessonModel` supports 7 types (video / audio / pdf / text / quiz / assignment / live). `ProgramModuleModel` exists. Lesson resources table exists (PDF / doc / zip / link / image / code attachments). Drip rules JSONB.

**Action: EXTEND (model) + REBUILD (admin UX).**
- Model extensions:
  - `type` enum needs to grow: add `youtube`, `vimeo`, `zoom`, `embed`, `external_link`, `form`, `survey`. (`media_provider` field already exists — may absorb some of these without enum change.)
  - SCORM: defer to Phase 4.
- Admin UX: today's 4-section form is replaced by B2's drag-drop tree + modal-per-lesson.
- Data migration: enum migration adds values; no data backfill needed.

### 5. Cohort batches + Live Sessions
**Current state:** `CohortBatchModel` + `LiveSessionModel` both rich. Live session has room_provider field (`100ms` / `zoom` / `none`), room_external_id (URL string), recording fields, attendance records, polls, chat messages.

**Action: EXTEND (model) + REBUILD (Zoom integration in admin).**
- Model: add `zoom_meeting_id` (numeric), `zoom_webhook_event_log` (JSONB) on LiveSession.
- Today's Zoom integration: just storing a URL. Rebuild: full server-to-server OAuth + auto-create-meeting + webhook listener.
- Attendance/polls/chat: keep — already strong.

### 6. Orders + Cart + Abandoned
**Current state:** `OrderModel`, `CartModel`, `cart_items` table, `OrderReconciliationService` scheduled job marks abandoned after 24h.

**Action: KEEP (model + service) + REBUILD (admin UX) + EXTEND (recovery flow).**
- Model: add `recovery_emails_sent` smallint, `cancellation_reason` varchar, `recovery_offer_coupon_id` FK (nullable).
- Recovery flow: 3-email sequence built on A5 Communication Center.
- Admin UX: replace today's read-only form with the customer-timeline + quick-actions console.

### 7. Payment Gateways
**Current state:** `PaymentGatewayRegistry` with Tap, Tamara, Mock. Webhook controller. Payment + refund + reconciliation models.

**Action: EXTEND (add Moyasar + HyperPay gateways) + REFACTOR (config from env to admin).**
- New gateway classes: `MoyasarPaymentGateway`, `HyperPayPaymentGateway` implementing existing `PaymentGateway` interface.
- New table: `payment_gateway_settings` (per-tenant config: api_key encrypted, mode test/live, enabled bool).
- Migrate existing env-only config to seed entries in the new table.
- Refund implementations per gateway.

### 8. Coupons
**Current state:** `CouponModel` rich (percentage / fixed / free_shipping, scope enum, usage limits, auto-apply, redemption tracking). `CouponService` validates + applies.

**Action: KEEP (core) + EXTEND (scope additions) + REBUILD (admin UX).**
- Model: scope enum gains `cohort`, `consultation`, `campaign`, `affiliate_code`. Add `affiliate_code` FK (nullable).
- Admin UX: scope picker (not JSON), redemption dashboard per coupon, simulate-impact preview.

### 9. Refunds
**Current state:** `RefundModel` with multi-step state machine (pending → approved → processing → completed → rejected). Requester + approver fields.

**Action: KEEP (model) + REFACTOR (workflow UX).**
- Add: `dry_run` boolean, `gateway_simulation` JSONB.
- UX: 3-step modal replaces today's single button. Sep-of-duty enforcement.

### 10. Reviews
**Current state:** `ReviewModel` rich (rating, body, moderation_status, instructor_response, helpful_count, verified_learner). `ReviewVoteModel` exists.

**Action: EXTEND (video) + REFACTOR (moderation UX).**
- Model: add `video_url`, `video_thumbnail_url`, `video_duration_seconds`.
- UX: moderation queue page (filter unmoderated), inline approve/reject, featured-review toggle.
- New job: 7-day-post-completion auto-request review email.

### 11. Consultations
**Current state:** `ConsultationRequestModel` — but consultations are FORMS, not first-class products. No consultant profile, no calendar.

**Action: NEW-BUILD (full consultant module).**
- New models: `ConsultantProfileModel`, `ConsultantAvailabilityRuleModel`, `ConsultationBookingModel`, `ConsultationReviewModel`.
- Keep existing `ConsultationRequestModel` as a legacy lead-capture form, separate from the new booking system.
- After 30 days of new system live, migrate old requests to bookings (one-time data move).

### 12. Forms
**Current state:** No generic forms. Hardcoded contact_messages table + consultation_requests + newsletter_subscribers.

**Action: NEW-BUILD (forms module) + REFACTOR (migrate existing hardcoded forms).**
- New tables: `forms`, `form_fields`, `form_submissions`, `form_field_responses`.
- Existing tables stay (contact_messages etc.) — new submissions route to `form_submissions` for any new form.
- After 30 days, decide whether to back-fill the legacy tables into the new submissions table.

### 13. Pages / CMS
**Current state:** `/app/Modules/Cms` folder exists, completely empty. Next.js frontend has hardcoded routes (homepage, about, etc.).

**Action: NEW-BUILD (pages + blocks) + REFACTOR (frontend rendering).**
- New tables: `pages`, `blocks`, `block_versions`, `block_translations`.
- Next.js: change hardcoded route pages to `[...slug].tsx` server-rendering from `GET /api/v1/pages/{slug}`.
- One-time migration: copy current hardcoded copy into seed `blocks` rows for each existing page.

### 14. Partners / Sponsors
**Current state:** `AffiliatePartnerModel` exists (for revenue-share affiliate system). No general partner/sponsor model.

**Action: NEW-BUILD (partners module).**
- New tables: `partners`, `partner_placements`.
- Affiliate system stays separate — different domain.

### 15. Marketing / Pixels / UTM
**Current state:** No marketing settings, no UTM capture, no conversion event dispatcher. `/app/Modules/Analytics` empty.

**Action: NEW-BUILD (marketing + analytics module).**
- New tables: `marketing_settings`, `marketing_events`, `utm_attribution` (per user/order).
- New service: `MarketingEventDispatcher`.
- New admin page: settings + attribution dashboard.

### 16. Communication (Email + In-App + SMS + WhatsApp)
**Current state:** `EmailOutboxModel`, `NotificationTemplateModel`, `NotificationModel` (in-app), Resend integration.

**Action: EXTEND (SMS + WhatsApp adapters) + REFACTOR (template editor).**
- New tables: `sms_outbox`, `whatsapp_outbox`. Adapter classes for vendors.
- Existing tables stay; gain `channel` enrichment.
- Admin template editor replaces today's "edit JSON".

### 17. Audit Log
**Current state:** `AuditLogModel` (tenant) + `AuditLogCentralModel` (landlord). `AuditAction` enum. `AdminAuditWriter`, `AuditWriter` services. Strong.

**Action: KEEP + EXTEND (new actions for Phase 3 subsystems).**
- Add 30-40 new actions: program.draft_saved, lesson.added_to_module, lesson.video_uploaded, page.published, block.added, form.submitted, refund.dry_run_executed, etc.
- No structural change.

### 18. Roles / Permissions
**Current state:** Full Spatie/Permission. 9 roles seeded. `RolesProvisioner` per-tenant seed.

**Action: KEEP + minor EXTEND.**
- Add: marketing role's permission for D4 settings, page-author role for D2 CMS edits (or use marketing role).
- No new role tables.

### 19. Trash / Soft-Delete Recovery
**Current state:** `TrashPage` covers 8 entities. Restore + permanent-delete + audit.

**Action: KEEP + EXTEND (add new entities).**
- Add to trash: pages, blocks, forms, consultants, partners.
- No structural change.

### 20. Health / Failed-job / Outbox monitoring (Operations module)
**Current state:** `FailedJobResource`, `EmailOutboxResource`, `PaymentWebhookResource`, `TenantAuditLogResource`. Read-only + retry/replay actions.

**Action: KEEP + EXTEND (add SMS/WhatsApp outbox).**

---

## Aggregate table

| Domain | Action | Phase 3 effort |
|---|---|---|
| Programs (model) | REFACTOR | small |
| Programs (UX) | REBUILD | large |
| Categories | KEEP+ | small |
| Instructors | KEEP | small |
| Lessons (model) | EXTEND | small |
| Lessons (UX) | REBUILD | large |
| Cohorts + Live Sessions (model) | EXTEND | small |
| Zoom integration | REBUILD | large |
| Orders (model) | EXTEND | small |
| Orders (UX) | REBUILD | large |
| Cart + Abandoned | KEEP+ | medium (recovery flow) |
| Payment Gateways | EXTEND (Moyasar+HyperPay) + REFACTOR (config) | medium |
| Coupons (model) | EXTEND | small |
| Coupons (UX) | REBUILD | medium |
| Refunds | REFACTOR | small |
| Reviews | EXTEND + REFACTOR | medium |
| Consultations | NEW-BUILD | large |
| Forms | NEW-BUILD | large |
| Pages / CMS | NEW-BUILD | extra-large |
| Partners | NEW-BUILD | small |
| Marketing / Pixels | NEW-BUILD | large |
| Communication (SMS/WhatsApp) | EXTEND | medium |
| Audit Log | KEEP+ | tiny |
| Roles | KEEP | tiny |
| Trash | KEEP+ | tiny |
| Operations resources | KEEP+ | tiny |

**Reading the matrix:**
- 9 domains are **KEEP / KEEP+** (no real work). Phase 3 inherits and doesn't disturb these.
- 6 domains need **EXTEND** (schema additions, no restructure). Low-risk.
- 4 domains need **REFACTOR** (internals reshape, same data). Medium-risk if rushed.
- 8 domains need **REBUILD or NEW-BUILD** (admin UX or fresh module). Where Phase 3 effort concentrates.

**Big-effort centers (the actual Phase 3 work):**
1. Program/Lesson/Order UX rebuilds (operator-facing daily tools)
2. Pages/CMS new-build (largest entirely-new module)
3. Forms new-build (foundational for everything else)
4. Marketing new-build (revenue measurement)
5. Consultations new-build (new product line)
6. Zoom integration rebuild (live ops backbone)

---

## What the operator decides

For each "REBUILD" and "NEW-BUILD" line, the operator confirms:
1. Is this acceptable in scope for Phase 3?
2. Should any of these defer to Phase 4 (smaller Phase 3)?
3. Any "KEEP" line they think actually needs work?

Default recommendation (if operator says "approve all"): full Phase 3 = ~11 months serial / ~8 months with parallelism. See `03-sequencing-plan.md` for the calendar + the recommended Phase-3-trimmed-to-6-months cuts.
