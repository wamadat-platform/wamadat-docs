# Phase 3 — Subsystem Matrix (19 subsystems)

**Status:** Analysis only. Per-subsystem deep profile across every dimension the operator asked for: goal / primary user / complexity / duration / blockers / hidden risks / DB impact / API impact / admin UX impact / mobile-app impact later.

**Conventions:**
- Complexity: S (small, 1-2w), M (medium, 3-4w), L (large, 5-7w), XL (>7w)
- Duration: realistic, includes integration + browser-proof
- "Mobile-app impact later" = what the future mobile API will need from this subsystem

---

## Category A — Cross-Cutting Foundations (everything depends on these)

### A1. Admin UX Library (RTL primitives, wizards, blocks, dangerous-action confirmations)
- **Goal:** Reusable Filament components — wizard stepper, draft/preview/publish state machine, danger-action modal (with explicit dollar amounts/data scope), drag-drop sortable list, color picker, asset picker, audit-aware form save. Arabic-first, RTL-clean.
- **Primary user:** Every Phase 3 subsystem's developer; downstream every admin operator using it.
- **Complexity:** M (3-4w)
- **Blockers:** None — pure additive Filament work.
- **Hidden risks:** Filament 3 cast a wide UX net; some primitives (wizards, drag-drop) need Livewire customizations. Easy to under-scope.
- **DB impact:** None.
- **API impact:** None.
- **Admin UX impact:** ⚠️ Foundational — every other Phase 3 file uses these primitives.
- **Mobile-app impact later:** N/A (admin-only).

### A2. Safety Rails (audit hooks, double-confirm for money/users, separation of duties)
- **Goal:** A standard `DangerousAction` Filament action class wired to AuditLog automatically. Separation-of-duties check (requester ≠ approver) on refunds, role changes, user bans, money operations. Soft-delete-first by default.
- **Primary user:** Operator (won't see it directly, but won't accidentally delete production data).
- **Complexity:** S (1-2w)
- **Blockers:** Audit log model already exists.
- **Hidden risks:** Migrating existing destructive actions to the new safe-action class requires sweep + test.
- **DB impact:** Schema change to add `audit_required` metadata on critical fields (or convention-based).
- **API impact:** None today (API layer is read-mostly).
- **Admin UX impact:** ⚠️ Universal — every destructive action goes through this.
- **Mobile-app impact later:** Server-side enforcement (separation of duties) is independent of client.

### A3. Block System Core (page composition primitive — used by Landing & Website CMS)
- **Goal:** Polymorphic Block table + 30 block types' shapes defined as Eloquent factories. Each block: kind, position, status, version, payload (JSONB validated by zod-style PHP rule). Reusable across "any page". Drag-drop ordering. Variant-per-language slots (ar/en).
- **Primary user:** Operator composes landing pages + CMS pages from these blocks.
- **Complexity:** L (5-7w) — block contract design is the make-or-break decision.
- **Blockers:** Decision on whether blocks live in DB rows (slow but flexible) or in versioned JSONB documents (fast but stricter migrations).
- **Hidden risks:** Block schema versioning. If a block kind evolves later (Hero v1 → Hero v2 with new field), need migration path that doesn't break old pages.
- **DB impact:** New tables: `pages`, `blocks`, `block_versions`, `block_translations`.
- **API impact:** New read endpoint: `GET /api/v1/pages/{slug}` returning rendered block tree. Cache-friendly.
- **Admin UX impact:** ⚠️ Foundational — Landing + CMS builders are skins on top of this.
- **Mobile-app impact later:** Mobile probably won't render landing pages (deep-link to web instead). But block-based content (e.g., lesson "rich text" block) may reuse the kind library.

### A4. Forms Builder (generic, used by 6+ flows)
- **Goal:** Operator-defined forms with: text, email, phone, multi-select, file upload, date, ToS checkbox, reCAPTCHA. Each form: title, slug, target (lead-list / consultation / contact / custom), notification template, auto-reply template, integrations (Webhook/Zapier later). Submissions table viewable in admin + Excel export.
- **Primary user:** Marketing/operator builds; public website + landing pages embed.
- **Complexity:** L (5-7w)
- **Blockers:** Block System Core (A3) — forms are embeddable as a `FormBlock`.
- **Hidden risks:** File upload security (signed URLs, virus scanning), GDPR/PDPL consent capture, spam protection without reCAPTCHA in Beta.
- **DB impact:** `forms`, `form_fields`, `form_submissions`, `form_field_responses`.
- **API impact:** `POST /api/v1/forms/{slug}/submit`, `GET /api/v1/forms/{slug}` (public).
- **Admin UX impact:** Big — new resource + a builder wizard.
- **Mobile-app impact later:** Mobile renders forms via API (block-based) and submits via the same endpoint.

### A5. Communication Center (multi-channel: email exists; SMS/WhatsApp/in-app extend)
- **Goal:** Unified message templating + delivery + tracking. Channels: email (exists, Resend), in-app (exists), SMS (new, gateway TBD — Unifonic or Taqnyat), WhatsApp (new, Cloud API later). Operator-edits templates with variable picker + preview in-language. Delivery logs filterable by template/user/channel/status.
- **Primary user:** Operator (templates, delivery diagnosis); system (every other subsystem calls into this).
- **Complexity:** M (3-4w to ship email + in-app polish; SMS/WhatsApp deferred to Phase 3.X)
- **Blockers:** Email outbox already exists. SMS/WhatsApp need vendor selection + integration shell.
- **Hidden risks:** Saudi PDPL marketing-consent rules; SMS sender-ID approval (CITC); WhatsApp template approval (Meta).
- **DB impact:** Extend `notification_templates` with per-channel body + locale; add `sms_outbox`, `whatsapp_outbox` shells.
- **API impact:** New read endpoint for in-app notifications (exists); webhook receivers for delivery status per provider.
- **Admin UX impact:** Big — template editor with live preview, channel toggles, delivery dashboard.
- **Mobile-app impact later:** In-app notifications are the same data. Push notifications (future) will reuse the channel pattern.

---

## Category B — Content & Catalog (operator's daily authoring)

### B1. Program Builder (Draft → Preview → Publish wizard)
- **Goal:** Replace the 5-tab form with a true wizard: 7 steps (Basics → Category & Instructor → Pricing & Tax → Type/Mode/Schedule → Media → SEO & Landing → Publish checklist). Draft autosaves. "Preview as student" opens read-only storefront. Publish checks (cover image present? at least 1 lesson? pricing set?) gate the publish button.
- **Primary user:** academy_owner / admin / instructor (instructor proposes; owner approves).
- **Complexity:** M (3-4w)
- **Blockers:** A1 (UX library wizard primitive).
- **Hidden risks:** Approval workflow (instructor proposes → owner publishes) is in the spec but doesn't exist in the model. Need a `proposed_changes` JSONB column or a separate `program_drafts` table.
- **DB impact:** Add `programs.draft_state` (JSONB) OR `program_drafts` table for approval workflow.
- **API impact:** `GET /api/v1/programs/{slug}` already exists; preview adds an authenticated variant with draft data.
- **Admin UX impact:** ⚠️ The single highest-leverage daily-workflow change.
- **Mobile-app impact later:** Mobile reads programs the same way. Draft preview not needed on mobile.

### B2. Program Content Builder (sections + 15 lesson types + drag-drop)
- **Goal:** Per-program, a tree view: Modules → Lessons. Drag-drop ordering at both levels. Inline "add lesson" with type picker (video / audio / pdf / text / quiz / assignment / live / YouTube / Vimeo / Zoom / link / file / form / survey / embed / SCORM later). Each lesson edit is a modal, not a page bounce. Free-preview toggle, required-for-completion toggle, drip rules, attached resources.
- **Primary user:** Instructor + admin.
- **Complexity:** L (5-7w)
- **Blockers:** B1, A1 (drag-drop primitive).
- **Hidden risks:** Large file uploads (video) need signed URLs to Bunny/Mux (already integrated for Phase D, but UI doesn't surface it). SCORM is a rabbit hole — defer.
- **DB impact:** Lesson model supports 7 types today; extend `type` enum or use `provider` polymorphism for YouTube/Vimeo/Zoom URLs (mostly via existing `media_provider` field).
- **API impact:** `GET /api/v1/programs/{slug}/curriculum` exists; needs structured response for nested modules.
- **Admin UX impact:** ⚠️ Huge — replaces the 20-form marathon.
- **Mobile-app impact later:** Curriculum API drives mobile lesson player.

### B3. Reviews + Testimonials (extend existing: video, moderation queue, request-after-completion)
- **Goal:** Existing `ReviewModel` already has moderation, helpful votes, instructor response. Add: video review upload (Bunny), moderation queue UI (filter unmoderated), automated review request 7 days after program completion. Featured reviews. Video testimonial section block (used by Landing Page Builder).
- **Primary user:** Operator (moderate); student (submit); landing pages (display).
- **Complexity:** S (1-2w)
- **Blockers:** None (ReviewModel rich already).
- **Hidden risks:** Video moderation cost (manual review hours). Need profanity filter / auto-flag.
- **DB impact:** Add `reviews.video_url`, `reviews.video_thumbnail_url`, `reviews.video_duration_seconds`.
- **API impact:** Existing public reviews endpoint; add video upload signed-URL endpoint.
- **Admin UX impact:** Medium — moderation queue is a new page.
- **Mobile-app impact later:** Mobile shows reviews; video plays via Bunny player.

---

## Category C — Commerce (revenue path)

### C1. Coupons + Promotion Codes (extend scope, add analytics)
- **Goal:** `CouponModel` exists with scope (platform / category / program / instructor / user) + usage limits + `is_auto_apply`. Add: cohort-scope, consultation-scope, campaign-scope (tied to a marketing campaign), influencer/affiliate code linkage. Operator UI: scope picker (real picker, not JSON), live coupon-impact preview ("this would have applied to 47 orders last month"), redemption dashboard per coupon.
- **Primary user:** Marketing operator.
- **Complexity:** M (3-4w)
- **Blockers:** A1 (picker primitive), C3 (Orders) for impact preview.
- **Hidden risks:** Stacking rules (can a user combine 2 coupons?) — needs explicit decision matrix.
- **DB impact:** Extend `scope` enum; add `affiliate_code` FK; consider `coupon_campaigns` table later.
- **API impact:** `POST /api/v1/checkout/apply-coupon` exists; needs scope-aware validation.
- **Admin UX impact:** Medium — picker UI + redemption dashboard.
- **Mobile-app impact later:** Mobile checkout calls the same coupon-apply endpoint.

### C2. Payment Integrations (extend: Moyasar, HyperPay)
- **Goal:** PaymentGatewayRegistry pattern exists (Tap, Tamara, Mock). Add MoyasarPaymentGateway + HyperPayPaymentGateway implementing the same `PaymentGateway` contract. Test/live mode config from admin (not env), webhook URL registration, signature verification, refund implementation.
- **Primary user:** Operator (config); system (process payments).
- **Complexity:** M (3-4w per gateway, parallelizable)
- **Blockers:** Vendor accounts (operator must sign up + share API keys via secure channel, not chat).
- **Hidden risks:** Each gateway has its own webhook signature scheme + refund delay (Tamara holds for 24h before approving). Edge cases multiply.
- **DB impact:** None (CommerceServiceProvider registration only).
- **API impact:** Per-gateway webhook endpoint already pattern-matched in PaymentWebhookController.
- **Admin UX impact:** Medium — gateway config page in admin (replaces env-only config).
- **Mobile-app impact later:** Mobile checkout uses same `POST /api/v1/checkout/initiate-payment` — gateway-agnostic.

### C3. Orders + Abandoned Checkout (UX over existing service)
- **Goal:** `OrderReconciliationService` already detects abandoned carts after 24h. Build the UI: dedicated tabs (New / Pending payment / Abandoned / Paid / Refunded / Failed), customer timeline per order (signed up → added to cart → checkout started → abandoned → recovery email sent), one-click "resend payment link", "apply coupon manually", "cancel with reason", "refund with approval gate". Abandoned-cart recovery sequence (3 emails over 7 days).
- **Primary user:** Operator (recover lost revenue), customer (receives recovery emails).
- **Complexity:** M (3-4w)
- **Blockers:** A5 (Communication Center for recovery email sequence).
- **Hidden risks:** Recovery email frequency cap (don't spam someone with 3 emails if they cancelled deliberately).
- **DB impact:** Add `orders.recovery_emails_sent` counter, `orders.cancellation_reason`.
- **API impact:** No new endpoints — admin-only.
- **Admin UX impact:** ⚠️ Big — Order resource becomes the operator's revenue-recovery console.
- **Mobile-app impact later:** Cart state mirrors on mobile; recovery emails deep-link back.

### C4. Refund Workflow Hardening
- **Goal:** Multi-step refund: Request (reason required, amount field requires explicit choice "full / partial / custom") → Approve (different user, gated by role + sep-of-duty) → Process (gateway call) → Settle (audit log captures gateway response). Dry-run mode (calculates fees + final refund without calling gateway).
- **Primary user:** Finance + admin.
- **Complexity:** S (1-2w) — mostly UX over existing service.
- **Blockers:** A2 (separation-of-duties primitive).
- **Hidden risks:** Partial refund accounting (which line item gets refunded? coupons?).
- **DB impact:** Refunds table already has approval fields; add `dry_run` flag + `gateway_simulation` JSONB.
- **API impact:** None (admin-only).
- **Admin UX impact:** Medium — replaces today's single "refund" button with a 3-step modal.
- **Mobile-app impact later:** None.

### C5. Financial Operations Dashboard (reporting)
- **Goal:** Daily/weekly/monthly revenue, top programs by revenue, failed-payment funnel by gateway, coupon impact, refund-rate trend, abandoned-cart conversion-after-recovery rate, payout tracking per gateway, VAT summary, export CSV/Excel/PDF.
- **Primary user:** academy_owner + finance.
- **Complexity:** M (3-4w)
- **Blockers:** C1-C4 must be in place (data they aggregate).
- **Hidden risks:** Saudi VAT (15%) accounting + ZATCA Phase 2 e-invoicing (tenant settings already flag `zatca_phase_2: true`).
- **DB impact:** Read-only views; potentially materialized views for performance.
- **API impact:** None (admin-only).
- **Admin UX impact:** Big — new dashboard page with charts + filters.
- **Mobile-app impact later:** N/A (operator-only).

---

## Category D — Marketing & Public Surface (acquisition)

### D1. Landing Page Builder (per-program + standalone campaign pages)
- **Goal:** Operator composes per-program landing pages from 20 block types: Hero, Benefits, Instructor, Curriculum, Pricing, Schedule, FAQ, Testimonials, Registration Form, Gallery, Video, CTAs, Countdown, Map, Embed, Custom HTML, Banner, Partners, Certificate Preview, Related Programs. Drag-drop, live preview (desktop + mobile), draft/publish, version history, rollback, SEO controls, RTL-clean.
- **Primary user:** Marketing + academy_owner.
- **Complexity:** XL (>7w)
- **Blockers:** A3 (Block System Core), A4 (Forms — Registration Form is a Form-Block), B3 (Reviews — Testimonials block).
- **Hidden risks:** Mobile rendering parity. Custom HTML/Embed = XSS risk — needs sanitization library + CSP. Version-history storage size.
- **DB impact:** Uses A3's tables. Add `program_landing_pages` join (or store landing-page-id on program).
- **API impact:** Public `GET /api/v1/landing/{program-slug}` returns block tree.
- **Admin UX impact:** ⚠️ Largest single visual builder.
- **Mobile-app impact later:** Mobile deep-links to web landing pages (not native-rendered).

### D2. Website CMS (all marketing pages as blocks)
- **Goal:** Replace Next.js hardcoded routes (homepage, about, programs index, instructors index, for-companies, consultations, careers, contact, FAQ, terms, privacy, campaign landing pages) with block-composed pages stored in `pages` table. Each page editable from admin.
- **Primary user:** Marketing.
- **Complexity:** L (5-7w)
- **Blockers:** A3 (Block System Core); D1 lessons learned (reuse block library).
- **Hidden risks:** SEO migration — old URLs must keep 200 responses. URL slugs are content-addressable.
- **DB impact:** Uses A3's `pages` table; add `pages.template` (home / about / generic).
- **API impact:** `GET /api/v1/pages/{slug}` (public).
- **Admin UX impact:** Big — operator can edit any page without dev.
- **Mobile-app impact later:** Mobile would still deep-link to web for static content.

### D3. Partners + Sponsors (new module)
- **Goal:** New `PartnerModel` (separate from `AffiliatePartnerModel` which is for revenue-share). Each partner: name, logo, link, type (sponsor / academic-partner / tech-partner), display targets (homepage / program-X / campaign-Y), active toggle, sort order. Partners block (D1/D2) renders them.
- **Primary user:** Marketing.
- **Complexity:** S (1-2w)
- **Blockers:** A3 (for the rendering block).
- **Hidden risks:** Logo asset management (already covered by media-upload pattern).
- **DB impact:** New tables: `partners`, `partner_placements` (M:N to pages/programs/campaigns).
- **API impact:** None — operator-managed config exposed via page blocks.
- **Admin UX impact:** Medium — new resource + multi-target picker.
- **Mobile-app impact later:** Mobile renders partners block via API.

### D4. Marketing + Tracking (pixels, UTM, conversion events)
- **Goal:** Admin settings page: paste Meta Pixel ID, TikTok Pixel ID, Snapchat Pixel ID, GA4 measurement ID, GTM container ID, custom scripts. Server-side event dispatcher emits standard events (PageView, ViewContent, AddToCart, InitiateCheckout, Purchase, Lead, CompleteRegistration, BookConsultation, Subscribe) to every configured pixel. UTM capture on every landing page → stored on user/order. Conversion-attribution dashboard.
- **Primary user:** Marketing + academy_owner.
- **Complexity:** L (5-7w)
- **Blockers:** A4 (Forms — Lead events), C3 (Orders — Purchase events), D1/D2 (PageView from landing pages).
- **Hidden risks:** Pixel consent gating (PDPL — opt-in cookies). Conversions API (server-side) for iOS-14-style attribution loss.
- **DB impact:** `marketing_settings` (single row keyed by tenant), `utm_parameters` columns on orders + users, `marketing_events` (event log).
- **API impact:** Pixel script tags injected in public layout; server emits events via background queue.
- **Admin UX impact:** Big — settings panel + attribution dashboard.
- **Mobile-app impact later:** Mobile attribution requires native SDKs (AppsFlyer / Adjust) — deferred.

---

## Category E — Live Operations (real-time / scheduled)

### E1. Zoom + Live Sessions (SDK integration over existing model)
- **Goal:** `LiveSessionModel` exists with room_provider field. Build Zoom server-to-server OAuth integration: create-meeting from admin (auto-fills LiveSession), send join link to enrolled students 24h before, send reminder 1h before, capture attendance via Zoom webhook (start/end events), store recording link.
- **Primary user:** Instructor + operator (schedule); students (attend).
- **Complexity:** L (5-7w)
- **Blockers:** A5 (Communication for reminders), Zoom server-to-server app registration (vendor).
- **Hidden risks:** Recording-link expiry handling; Zoom webhook signature verification; concurrent-meeting limits per Zoom plan.
- **DB impact:** Use existing `room_external_id` + `room_metadata`. Add `zoom_meeting_id`, `zoom_webhook_event_log`.
- **API impact:** Webhook endpoint for Zoom events; student-facing `GET /api/v1/me/live-sessions/{id}/join-url` (signed, expiring).
- **Admin UX impact:** Big — replaces today's "paste Zoom URL" textbox.
- **Mobile-app impact later:** Mobile launches Zoom app via deep link.

### E2. Consultation System (Bookings, calendar, packaged service)
- **Goal:** Consultation as a first-class product alongside programs. Each consultant: profile (photo, bio, specialty, hourly rate, session duration, types online/in-person/phone, available time slots), Zoom-link-per-session, booking form, payment, reminders, post-session review. Consultant landing page.
- **Primary user:** Consultants (manage availability), students (book).
- **Complexity:** L (5-7w)
- **Blockers:** A4 (Forms — booking form), A5 (Communication — reminders), E1 (Zoom integration).
- **Hidden risks:** Calendar conflicts (consultant's availability across multiple bookings); cancellation policy (refund window?); time-zone handling.
- **DB impact:** New tables: `consultants`, `consultant_availability_rules`, `consultations`, `consultation_bookings`, `consultation_reviews`.
- **API impact:** `GET /api/v1/consultants`, `POST /api/v1/consultations/book`.
- **Admin UX impact:** Big — new resource cluster.
- **Mobile-app impact later:** Mobile booking flow reads/writes via API.

---

## Category F — Operator Cockpit (top-of-funnel visibility)

### F1. Operations Dashboard (the home page operator sees)
- **Goal:** A real owner dashboard, not a tile of widgets. Sections: revenue at-a-glance (today / this week / MTD vs same period last year), pending operator actions (unmoderated reviews count, abandoned-cart count, refund-approval count, failed-payment count), live signals (programs starting today, consultations today, live sessions today), failure surface (failed jobs count, failed-email count, failed-webhook count), top-performing program of the week.
- **Primary user:** academy_owner (1 page, glanceable).
- **Complexity:** M (3-4w)
- **Blockers:** Most other subsystems must exist (it aggregates them).
- **Hidden risks:** Cache staleness vs real-time accuracy tradeoff.
- **DB impact:** Read-only; potentially Redis-cached counters.
- **API impact:** None (admin-only).
- **Admin UX impact:** ⚠️ The first thing the operator sees every morning.
- **Mobile-app impact later:** Mobile "admin lite" could mirror; deferred.

---

## Category G — Quality Gate (last)

### G1. Browser Proof Suite (end-to-end validation)
- **Goal:** Playwright/Cypress tests covering the operator's full Phase 3 flow + Beta-user flow: create program → add modules → upload lesson video → add Zoom session → add PDF → add embed → build landing page → publish → public visitor opens page → registers via form → checks out (Tap or Mock gateway) → coupon applied → order paid → enrollment created → reminder email sent → live session attended → review submitted → moderated → published on landing.
- **Primary user:** Verification of all 18 prior subsystems.
- **Complexity:** M (3-4w)
- **Blockers:** All prior subsystems.
- **Hidden risks:** Test environment parity with prod (especially Zoom + payment gateway sandboxes).
- **DB impact:** Test fixtures; no schema change.
- **API impact:** Tests exercise API surface.
- **Admin UX impact:** Tests assert UX flows visually.
- **Mobile-app impact later:** Mobile e2e is a separate suite.

---

## Summary table — totals

| Category | Subsystems | Total realistic duration |
|---|---:|---:|
| A. Cross-cutting foundations | 5 | 13-19w |
| B. Content & catalog | 3 | 9-15w |
| C. Commerce | 5 | 13-17w |
| D. Marketing & public surface | 4 | 18-26w |
| E. Live operations | 2 | 10-14w |
| F. Operator cockpit | 1 | 3-4w |
| G. Quality gate | 1 | 3-4w |
| **TOTAL realistic, sequential** | **21** | **69-99w** |

Note: 19 subsystems in the operator's spec → 21 in my breakdown (Refund Workflow + Operations Dashboard split off as standalone because their dependency profiles differ from the parent category). Many subsystems can parallelize once foundations land — see `03-sequencing-plan.md` for the calendar.
