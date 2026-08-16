# Phase 3 — Architecture Rules

**Status:** Analysis only. These are the design constraints every Phase 3 subsystem must obey. The operator asked: what's reusable / generic / tightly coupled / config-driven / block-based?

## 1. Reusable (shared library; every subsystem consumes)

These live in a shared Filament/Laravel package and never get re-implemented per subsystem.

| Primitive | Owner subsystem | Consumed by |
|---|---|---|
| **Wizard component** (multi-step, progress bar, draft autosave, validation per step) | A1 Admin UX | B1 Program Builder, B2 Content (lesson-edit modal flow), E2 Consultations (consultant onboarding), C4 Refund (3-step modal) |
| **Drag-drop sortable list** | A1 | B2 Content (modules, lessons), D1 Landing (blocks), D2 CMS (blocks), D3 Partners (sort order) |
| **Asset picker** (uploads to Bunny + signed URLs) | A1 | B1 Program covers, B2 Content videos/PDFs, B3 Review videos, D1/D2 image blocks, D3 partner logos |
| **Color picker** (HSL + brand presets) | A1 | Plan branding, Tenant branding, block accent colors |
| **Quota/feature toggle picker** (replaces KeyValue JSONB inputs) | A1 | Plan quotas, Tenant settings |
| **Danger-action modal** (typed confirmation: "اكتب اسم البرنامج للتأكيد") | A2 Safety | Program archive, Tenant suspend, User ban, large refunds, bulk deletes |
| **Audit-wrapped form save** | A2 | every destructive resource action |
| **Separation-of-duties check** (`requester !== approver` + role gate) | A2 | C4 Refund, role change, plan upgrade |
| **Block renderer** (server-side renders block tree → HTML for SSR + JSON for client) | A3 Block System | D1, D2 |
| **Block schema validator** (zod-style PHP rule per block kind) | A3 | every block save |
| **Form schema validator** (field types, conditional logic, required rules) | A4 Forms | every form submit |
| **Template engine** (variable substitution + per-locale + per-channel) | A5 Communication | every email/SMS/in-app trigger |
| **Delivery tracker** (status webhook → row update + retry queue) | A5 | every outbound channel |
| **Audit log writer** (existing service, extend) | already exists | every destructive subsystem |

## 2. Generic (one model serves many concrete use-cases)

These are domain abstractions where one schema must serve multiple business cases — designed for evolution.

### A. The "Block" abstraction (used by D1 Landing + D2 CMS)
- One `blocks` table with `kind` enum + `payload` JSONB.
- 25-30 block kinds defined as separate factory classes implementing a `BlockType` interface.
- Adding a new block kind = adding a class + a payload schema + a Blade render template. No migration.
- Versioned: each block-edit creates a `block_version` row for rollback.

### B. The "Form" abstraction (used by 6 concrete forms)
- One `forms` table for definitions, one `form_submissions` for entries.
- Each form has a `target` (lead-list / consultation-booking / contact / custom-webhook / etc.).
- Field types are a fixed enum (text, email, phone, number, multi-select, file, date, consent, captcha).
- A "registration form" and a "contact form" are the same model with different field configs + different targets.

### C. The "Notification" abstraction (used by every outbound message)
- Channels: email, sms, whatsapp, in-app. Each is an adapter implementing `NotificationChannel` interface.
- Templates: one row per template-slug per locale per channel.
- Triggers: domain events emit "notify:order.abandoned" → registry maps to template + audience → dispatcher runs.

### D. The "Audit" abstraction (already exists; extend)
- `AuditAction` enum already covers 50+ actions.
- Phase 3 adds 30-40 more actions (program.draft_saved, lesson.added, block.published, form.submitted, etc.).
- Stays generic — one table, one writer service.

## 3. Tightly coupled (NOT generic — domain-specific by design)

Some things should stay specific. Forcing genericity here would lose semantic value.

| Subsystem | Why tightly coupled |
|---|---|
| **Payment gateway integrations** (C2) | Each gateway has a unique webhook signature scheme, refund delay, fee structure. The `PaymentGateway` interface is the abstraction; individual gateways stay specific. |
| **Refund workflow** (C4) | Tied to Order + Payment + Gateway. Generic "approval workflow" engine would over-abstract — a refund is not a leave-request. |
| **Zoom integration** (E1) | Zoom-specific OAuth flow + webhook events. Trying to abstract "any video conferencing" prematurely will block when we adopt 100ms or Twilio. Start Zoom-coupled, abstract later if needed. |
| **Curriculum / Modules / Lessons** (B2) | Programs have a specific structure (modules contain lessons; lessons have types). Generic "tree of nodes" is too abstract — we'd lose validation power. |
| **Saudi VAT + ZATCA invoicing** (C5) | Country-specific compliance. Don't generalize. |

## 4. Config-driven (operator changes via admin, not code change)

The operator's spec says "ممنوع hardcoded". These items must be config from day one:

| Config | Stored in | Operator-edits via |
|---|---|---|
| Tenant branding (colors, logo, fonts) | `tenants.branding` JSONB (exists) | A1 picker UI replaces today's KeyValue |
| Plan quotas + features | `plans.quotas`, `plans.features` JSONB (exists) | A1 picker UI replaces today's KeyValue |
| Marketing pixel IDs | `marketing_settings` table (new in D4) | D4 settings page |
| Email/SMS template content + variables | `notification_templates` (exists) | A5 template editor |
| Form definitions | `forms` (new in A4) | A4 builder |
| Block payloads on pages | `blocks` (new in A3) | D1/D2 builders |
| Payment gateway selection + test/live mode | `payment_gateway_settings` (new in C2) | C2 admin page |
| Coupon rules (scope, limits) | `coupons` (exists) + scope picker | C1 picker UI |
| Page SEO (title, description, OG image) | `pages.seo` JSONB | D1/D2 SEO panel per page |
| Partner display targets | `partner_placements` (new in D3) | D3 multi-target picker |
| Consultant availability rules | `consultant_availability_rules` (new in E2) | E2 availability editor |
| Audit retention (days) | new tenant setting | Filament setting page |
| Beta caps (50 users / 5 programs) | new tenant setting | Filament setting page |
| Webhook URLs (Zapier, Make.com) | new `integrations` table | Settings page |
| Storefront CTAs, hero copy | A3 blocks on D2's home page | D2 builder |

**Anti-pattern to avoid:** any string visible to a customer must NOT be in code. Includes program category names, FAQ items, footer links, social links, contact emails, brand slogans. All in the database, all editable via admin.

## 5. Block-based (composable from a library, not bespoke)

These layers use the Block abstraction (A3):

| Surface | Composition |
|---|---|
| Per-program landing page (D1) | Blocks from the 20-block library; each program gets a starter template + can customize |
| Homepage (D2) | Block tree |
| About / For-companies / Careers / Contact / FAQ pages (D2) | Block trees |
| Terms / Privacy (D2) | Block trees, but a simpler "rich text" + "TOC" pattern (legal pages = mostly long-form) |
| Campaign landing pages (D2) | Block trees, with countdown / urgency blocks more common |
| Consultant profile pages (E2) | Block tree (smaller subset: Hero, Bio, Availability, Reviews, CTA) |
| Email templates (A5) | NOT block-based — stays MJML-style for email client compat |
| Lesson "text content" type (B2) | Rich-text editor, not full block tree — too heavy for a single lesson |

**Block library starter set (20 + 5 internal):**

External-facing (used in D1/D2 admin UI):
1. Hero (image + headline + sub + CTAs)
2. Headline + Body (text section)
3. Image (full-width / inset)
4. Image Grid / Gallery
5. Video (Bunny / YouTube / Vimeo)
6. Features Grid (icon + title + body × N)
7. Benefits List (checkmark list)
8. Audience ("لمن هذا البرنامج")
9. Instructor Card
10. Curriculum (auto-pulls program modules)
11. Pricing Card (auto-pulls program pricing + active coupons)
12. Schedule (auto-pulls cohort dates)
13. FAQ accordion
14. Testimonials Carousel (auto-pulls featured reviews)
15. Registration Form (embeds an A4 form)
16. CTA Banner
17. Countdown Timer
18. Map (Saudi cities preset)
19. Partners Grid (auto-pulls D3 partners for this target)
20. Related Programs (auto-pulls by category)

Internal-only (programmatic use, not in builder UI):
- Custom HTML (admin role gate, sanitized) — safety net
- Embed Code (sanitized) — safety net
- Certificate Preview (per-program auto-render)
- Newsletter Subscribe (special form)
- Spacer / Divider (layout primitive)

## 6. Multi-tenant correctness (every subsystem)

Phase D established the tenant-schema-per-academy pattern. Phase 3 inherits these invariants:

1. Every new table on the tenant connection lives in `tenant_<slug>` schema and uses `UsesTenantConnection` trait on the model.
2. Every landlord table on the landlord connection.
3. No cross-schema joins. No raw queries that hardcode `tenant_wamadat.` — use the model's connection.
4. Background jobs that touch tenant data must be queued with tenant context (Spatie's `MakeQueueTenantAwareAction` handles this — don't bypass).
5. New API endpoints exposed on `/api/v1/` MUST respect tenant resolution (subdomain or X-Tenant-Slug header).
6. Admin actions write to tenant `AuditLogModel`; landlord/super actions write to `AuditLogCentralModel`.

## 7. RTL + Arabic-first

The operator's spec says "العربية أولا". This is more than translation:

1. All admin Filament pages: `dir="rtl"` set, Tajawal font already configured in both panels. ✅ today.
2. **All form labels Arabic by default**, with English as the i18n alternate.
3. **Mixed-direction strings handled correctly** — when "https://example.com" appears in an Arabic paragraph, isolated via `<bdi>`.
4. **Date format**: Hijri toggle available, default Gregorian per current setting.
5. **Numbers**: Latin numerals by default (per existing tenant setting `numerals: latin`). Don't enforce Hindi-Arabic numerals unless tenant flips it.
6. **Block builder**: every block has `_ar` and `_en` slots; switcher in builder. Empty `_en` → frontend falls back to `_ar`.
7. **Phone format**: E.164 storage (`+9665...`), display with grouping (`+966 5XX XXX XXX`).
8. **SEO meta**: `lang` attribute matches block content language; canonical URLs per locale.
9. **Email + SMS template language**: tenant setting drives default; per-user preference overrides.

## 8. Non-technical operator default

Every UI decision answers: "would a non-technical academy owner understand this label/flow on first sight?"

| Anti-pattern | Replacement |
|---|---|
| "Polymorphic relation" | "Linked content" |
| "JSONB payload" | a real form with labeled fields |
| "Status: published / archived" | "حالة: منشور / مؤرشف" (Arabic-first) + colored badge |
| "Cron expression" | "Every day at 8am" (preset list + custom) |
| "Webhook signing secret" | "Connect [Service]" button + Wamadat handles the secret internally |
| "Bulk action" | "تطبيق على المحدّد" with explicit count "(12)" shown |
| Generic "Save" | Specific verb: "نشر" / "حفظ كمسودة" / "أرسل للمراجعة" |

## 9. Future-proofing — what must scale without rebuild

The operator's spec says "كل شيء قابل للتوسع لاحقا بدون rebuild كامل". The four pillars:

1. **Block library** — adding a block kind in 1 day, not 1 month.
2. **Form fields** — adding a field type in 1 day.
3. **Payment gateways** — adding a gateway in 1 week.
4. **Notification channels** — adding a channel in 1 week.

These four are the dimensions where the platform will need to expand most. The Phase 3 architecture must keep them open. Everything else (program model, order model, refund model) can stay tightly typed — they don't grow new variants at the same rate.

## 10. The "would I bet 6 months on it?" rule

Before any subsystem's PR ships, the developer must answer:
- Could a non-technical operator use this without help?
- Could a new developer onboard to this subsystem in 1 day?
- Could we 2x the data volume (programs, users, orders) without changing the design?
- Could we add the obvious next feature (e.g., bulk import, second tenant, payment-on-installments) without rebuilding?

If any answer is no, the subsystem is not ready to ship. This is the bar.
