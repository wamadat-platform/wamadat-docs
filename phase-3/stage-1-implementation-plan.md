# Phase 3 — Stage 1 Implementation Plan (v3 — revised 2026-05-15)

**Stage 1 scope:** 7 cross-cutting foundation subsystems + Media Library + Mini Ops Dashboard + Program-edit UX refactor.
**Stage 1 budget:** 12 weeks realistic (10 dev + 2 buffer for 3 review-gate iterations).
**Branch:** `phase-3` (created off `main` at `b9e6850` as the first action of Week 1).
**Approval status:** operator approved v1+v2+v3 with iterative refinements. v3 is the working plan.

**Revision log:** see `revisions-history.md` for v1→v2→v3 changes.

---

## Design Principles (Stage 1 — apply universally)

### Principle #0 — Operational UX > developer convenience in every conflict
When the design choice is "elegant abstraction" vs "fewer keystrokes for operator" — fewer keystrokes wins. Document any elegance-loss in BACKLOG for later revisit if it bites.

### Principle #1 — Every contract works across domains
Every public-facing or admin-facing contract must serve: Programs, Pages, Consultations (placeholder), Campaigns, future mobile. Only generalize when ≥2 actual consumers exist. No future-proof abstractions for hypothetical consumers.

### Principle #2 — Block schema discipline is non-negotiable
Every BlockType implements 7 mandatory methods. PHPStan custom rule enforces. CI fails on missing method or invalid schema.

### Principle #3 — Operator review gates prove operational pain relief
Each Review Gate is a REAL operational scenario the operator runs against the platform. Not a component showcase. If a Gate doesn't demonstrate reduced daily friction, it fails.

### Principle #4 — Non-technical operator default
Every label, error message, empty state assumes the reader doesn't know what JSON is. RTL + Arabic-first. Single-step common paths.

### Principle #5 — Tenancy correctness
Every new tenant-scoped table in `migrations/tenant/`. Every new tenant-scoped model declares `UsesTenantConnection`. Two-tenant isolation test asserts no cross-bleed.

### Principle #6 — Single-screen-per-task
If a logical task forces the operator across more than one screen, the design is wrong. Modals stack max 2 deep before forcing a separate page. Deep workflows decompose into modals nested on the parent's edit page.

---

## What ships in Stage 1 (v3)

| # | Subsystem | Output | Weeks |
|---|---|---|---|
| A1 | Admin UX Library | 10 Filament/Livewire primitives | 1-2 |
| A2 | Safety Rails | SafeAction + RequiresSeparationOfDuty wired into existing destructive actions | 1-2 |
| **MED** | **Media Library** (expanded v3) | `media_assets` + `asset_references` tables; replace-without-break; "where used" inspector; folder/tag search; image+video preview | **2-4** |
| **PR** | **Program Resource refactor** (new in v3) | Existing Program create/edit migrated onto A1 wizard + A2 SafeAction + AssetPicker | 2 (overlap with A1) |
| A3 | Block System Core | `pages` + `page_versions` + 7 starter BlockTypes + builder UI + publish/rollback | 5-7 |
| A4 | Forms Builder (expanded v3) | `forms` + 4 sibling tables + 9 field types + spam abstraction + submission audit + export filtering | 7-8 |
| A5 | Communication polish (simplified v2) | Minimal-toolbar editor + categories + variable chips + test send + delivery dashboard | 9 |
| **POL** | **Publish Policies** (expanded v3) | `publish_policies` + scheduled publish/unpublish + per-entity publish-history timeline | 9-10 |
| **DSH** | **Mini Ops Dashboard** (expanded v3) | `NeedsActionWidget` with 8 actionable click-through widgets | 10 |
| Gate | Stage 1 Browser Proof | Playwright suite covering 4 checkpoints + operator pre-Exit walkthrough | 11-12 |
| | Buffer / iteration absorber | Distributed across the 12 weeks (~2w) | — |

---

## Review Gates — REAL operational scenarios (Principle #3)

### 🛑 Review Gate 1 — end of Week 2

**Operator-facing scenario** (you actually run this):

1. Open `/admin/programs/{id}/edit` for one of your 5 published programs.
2. Notice the form now appears as a 5-step **wizard** (A1.1), not a flat 5-tab form.
3. In Step 4 "Media", click the cover-image picker. AssetPicker opens.
4. Browse the Media Library — see images organized by folder (sidebar tree). Filter by tag.
5. Upload a new cover image inline. It lands in Media Library + auto-selected.
6. Continue wizard to Step 5 "Publish". Click "نشر".
7. A SafeAction modal (A2.6) demands typed program-slug confirmation. Type, confirm.
8. Audit log entry created. Program updated. Wizard exits cleanly.
9. Try "حذف" (delete) on a draft program — SafeAction demands typed confirmation again.
10. Open `/admin/dev/ux-library` as a sanity check — every primitive renders correctly in RTL.
11. On mobile viewport (Chrome DevTools 390×844): the wizard, AssetPicker, and SafeAction modal are all usable.

**Pass criteria:**
- Wizard renders correctly RTL across all 5 steps
- AssetPicker browses, filters, uploads, selects
- SafeAction modal prevents accidental destructive actions
- Mobile viewport: every interaction works with thumb-friendly targets
- Operator marks 0 components "redo" without a buffer-absorbed fix plan

**This is NOT a component demo. It's the operator's daily program-edit flow, now refactored.**

### 🛑 Review Gate 2 — end of Week 6

**Operator-facing scenario:**

1. `/admin/pages` → "New page" → name "صفحة هبوط الحملة"
2. Add 4 blocks via drag-drop: Hero + Text + Image + CTA
3. For Hero image: pick from Media Library (reuses asset from Gate 1)
4. Click "Save draft"
5. Click "Preview" → iframe shows desktop + mobile toggle, page renders correctly RTL
6. Click "Publish" (operator has direct publish via policy)
7. Public visitor opens `/p/{slug}` → page renders identically to preview
8. Edit Hero headline → save draft → publish → page_versions row v2 created
9. Click "Rollback to v1" → confirmation → live page reverts
10. Replace the Hero image in Media Library with a different file → the published page now shows the new image (because URL lookup is by asset_id, not URL)
11. Click the image's "where used" inspector → shows it's used in this page + on the program edit'd in Gate 1

**Pass criteria:**
- Block builder is drag-drop in RTL (or up/down buttons if RTL drag fallback was used)
- Live preview shows desktop + mobile faithfully
- Publish, rollback, version snapshots work
- Asset replacement doesn't break references
- Where-used inspector accurately tracks placement
- 0 critical bugs

### 🛑 Review Gate 3 — end of Week 11 (dry-run of Exit Gate)

Operator informally walks the full 14-step Exit Gate scenario (see below) before formal ratification in Week 12. Anything flagged "redo" gets absorbed in Week 12 buffer days before official sign-off.

---

## Week-by-week execution (v3)

### Week 1 — A1 + A2 start, branch created

**Day 1 setup:**
- `phase-3` branch created off `main` (tip `b9e6850`)
- All Stage 1 work commits to `phase-3`
- New module directories scaffolded (empty + .gitkeep)

**A1 deliverables (Week 1):**
- `WizardStep.php` + Blade
- `SortableList.php` + Livewire (POC drag-drop in RTL; fallback to up/down buttons if Filament RTL drag fails)
- `ColorPicker.php`
- Tests

**A2 deliverables (Week 1):**
- `SafeAction.php`, `RequiresSeparationOfDuty.php`, `ApproverIsRequesterException.php`
- Test suite

**Daily reporting starts Day 1.**

### Week 2 — A1 finish + A2 wire-in + MED start + PR refactor + Review Gate 1

**A1 finish:**
- `DangerActionModal.php`, `AuditAwareForm.php`, `QuotaFeaturePicker.php`, `StatusBadge.php`, `BlockPreview.php`
- Demo at `/admin/dev/ux-library` (sanity-check route, not the Gate)

**A2 wire-in:**
- RefundOrder → SafeAction + RequiresSeparationOfDuty
- SystemUserResource role-change → SafeAction
- TrashPage permanent-delete → SafeAction
- Permission seeds: `*.propose`, `*.publish`, `*.schedule_publish`, `*.schedule_unpublish`

**MED start (new in v3 — expanded scope):**
- Migration: `media_assets` table (tenant)
- Migration: `asset_references` table (tenant) — tracks every entity using each asset
- `MediaAssetModel`, `AssetReferenceModel` with UsesTenantConnection
- `MediaUploadService` (uploads to Bunny, returns Asset with thumbnail/medium/large URL variants)
- Async observer: when entities save with asset fields, write reference rows

**PR — Program Resource refactor (new in v3):**
- ProgramResource form() restructured to use A1 WizardStep
- Cover image field → AssetPicker (pulls from Media Library)
- Trailer video field → AssetPicker (video type filter)
- Publish action → SafeAction with typed-slug confirmation
- Delete (soft) action → SafeAction
- Mobile viewport tested

**🛑 REVIEW GATE 1 — Week 2 end** (see scenario above)

### Week 3 — MED expanded features

- `MediaLibraryResource` (Filament): folder tree sidebar + grid view + tag filter + search
- Folder navigation via string path (lean — no separate folders table)
- Tag operations on assets (Postgres text[] array)
- Image variants via Bunny URL params (auto-thumbnail / medium / large)
- Video preview using HTML5 video element + Bunny streaming
- **Copy URL action** (Change #3 — v3): operator clicks to copy any variant URL
- **Where-Used inspector** (Change #3): clicking an asset shows list of every entity using it (Page X, Program Y, Lesson Z) with deep-links
- **Replace File action**: keeps asset UUID stable, replaces underlying Bunny URL; all referencing entities auto-update on next render
- Search combines: folder filter + tag filter + filename search + type filter

### Week 4 — MED finish + AssetPicker integration + UX polish iteration

- AssetPicker (A1.3) integrated:
  - Browse-library tab (default)
  - Upload-new tab (inline upload writes to library + selects)
  - Filters by media type per consumer context (image-only for cover; video-only for trailer; any for downloadable file)
- All consumers of file/image uploads migrated to AssetPicker:
  - Program covers, trailers, promo videos
  - Page block images
  - Lesson media (placeholder fields — actual Content Builder is Stage 2)
- Iteration buffer from Review Gate 1 feedback absorbed

### Week 5-7 — A3 Block System Core

**Week 5 — model + interface + 7 BlockTypes:**
- Migrations: `pages`, `page_versions` (tenant); `pages.owner_type` + `pages.owner_id` (polymorphic, for program landings)
- Migration: `pages.scheduled_publish_at`, `pages.scheduled_unpublish_at` (Change #6 v3)
- Models: `PageModel`, `PageVersionModel`
- `BlockType` interface (7 methods)
- `BlockContext` value object
- 7 starter `BlockType` implementations: Hero, Text, Image, ImageGrid, Video, FAQ, CTA
- PHPStan custom rule enforcing contract
- `BlockRegistry`, `BlockValidator`

**Week 6 — services + builder UI + Review Gate 2:**
- `PageDraftService`, `PagePublishService` (handles scheduled too), `PageRollbackService`
- `PageUnpublishService` (handles scheduled unpublish — Change #6 v3)
- `BlockBuilder.php` (Livewire drag-drop list + per-block edit modal)
- `BlockPreview.php` (desktop + mobile toggle, iframe)
- `PageResource` (Filament)

**🛑 REVIEW GATE 2 — Week 6 end** (see scenario above)

**Week 7 — A3 finish + public render + Browser Proof 2:**
- Public read endpoint: `GET /api/v1/pages/{slug}` (HTML for browser, JSON for mobile via Accept header)
- Program landing endpoint: `GET /api/v1/programs/{slug}/landing`
- GIN indexes on JSONB
- 7 BlockType contract tests pass

### Week 8 — A4 Forms Builder (expanded v3)

- Migrations: `forms`, `form_fields`, `form_submissions`, `form_field_responses` (tenant)
- Migration: `form_submission_audit` (tenant) — every submission writes one row with IP, UA, field diff
- Models with UsesTenantConnection
- 9 `FormFieldType` implementations
- `SpamCheck` interface — Honeypot impl bundled; reCAPTCHA stub for Phase 4
- `FormBuilderService`
- Rate limit per-form configurable (default: 5/min IP, 1/min email)
- Public submission endpoint with audit logging
- File uploads route through Media Library service
- `FormResource` + `FormSubmissionResource` (with export filtering UI — date range, status, field-value filters)
- `FormBlockType` (A3 block kind)

**Browser Proof Checkpoint 3 (end of Week 8):**
- Form built → embedded in page → public submit (with file) → submission lands in admin → file appears in Media Library tagged "form-upload" with `asset_references` entry → in-app notification → CSV export with filters applied works → audit log row written per submission

### Week 9 — A5 Communication + POL Publish Policies

**A5 simplified editor (Change #7 v2):**
- `NotificationTemplateResource` rebuilt with minimal toolbar (bold/italic/link/lists)
- Category filter pills (transactional / marketing / notification / system)
- Variable picker as inline chip-pills
- Live preview with sample data
- "Test send to me" button
- `DeliveryDashboardPage` combined email outbox + in-app + delivery rates 7d/30d

**POL (v3 expanded):**
- Migration: `publish_policies` table (tenant) — (entity_kind, role_name) → mode
- Migration: `publish_history` table (tenant) — every publish/unpublish/schedule/cancel/fire emits a row
- Seeded policies: academy_owner = direct on all; instructor = review_required on Program/Lesson; marketing = review_required on Page/Block
- `RunScheduledPublishes` console command (cron every 5 min)
- `RunScheduledUnpublishes` console command (cron every 5 min)
- `PublishHistoryService` writer
- `PublishHistoryTimelineWidget` — Filament widget showing entity's publish history on its edit page
- `PublishModePicker` settings page (academy_owner only)

### Week 10 — DSH Mini Ops Dashboard (expanded v3) + integration polish

**DSH (v3 expanded — 8 widgets, all click-through actionable):**

`NeedsActionWidget` at top of `/admin` dashboard. Each row:

| Widget | Count source | Sub-text (1 line) | Click-through |
|---|---|---|---|
| Pending approvals | proposed_changes pending publish for current user's role | "5 pending — oldest 2d" | filtered list |
| Failed email sends | EmailOutbox status=failed | "3 failures in 24h" | EmailOutboxResource |
| **Abandoned checkouts** | Cart abandoned_at < 24h, converted_at null | "12 abandoned — 8.5K SAR" | OrderResource filtered |
| Draft pages | Page status=draft (CMS + program landings) | "4 drafts — oldest 3d" | PageResource filtered |
| **Unpublished programs** | Program status=draft (separate from pages) | "2 drafts" | ProgramResource filtered |
| **Upcoming live sessions** | LiveSession scheduled_at in next 24h | "1 session in 3h" | LiveSessionResource filtered |
| Failed jobs | failed_jobs count | "0 failures" or "2 failures in last hour" | FailedJobResource |
| Recent refunds | Refund last 7 days | "3 refunds — 450 SAR" | RefundResource (or order filter) |

All widget rows cached 60s; manual refresh button.

### Week 11 — Cross-stage polish + Review Gate 3 (dry-run of Exit Gate)

- Integration testing across all subsystems
- Fix anything Review Gate 2 flagged
- Operator dry-runs the full Exit Gate scenario informally

**🛑 REVIEW GATE 3 — Week 11 end:** operator walks the formal 14-step scenario informally, marks issues, we fix before Week 12 ratification.

### Week 12 — Stage 1 Exit Gate + closeout + merge

**Stage 1 Browser Proof Checkpoint 4 — formal 14-step scenario:**

1. Operator uploads a hero image to Media Library, tags "campaign-may", folder `marketing/campaigns`
2. Operator creates a generic site landing page with Hero (picks the image) + Text + Image + CTA + FAQ
3. Operator creates a program landing page bound to an existing published program: Hero + Pricing (auto-pulls from program) + FAQ
4. Operator creates a form "تواصل معنا" with 4 fields (name, email, phone, message + file attachment), spam protection honeypot on
5. Operator embeds the form as FormBlock on the site landing
6. Operator edits email template "form.submission_received" — adds welcome line + previews + test-sends; receives email
7. Operator publishes site landing directly (policy=direct); schedules program landing for +30 min publish; site landing visible immediately
8. Wait 30 min OR manually trigger scheduler → program landing publishes; publish_history shows scheduled-fire event
9. Operator schedules site landing for unpublish at +60 min from now (operator wants the campaign to expire)
10. Public visitor opens site landing → submits form (with attached file)
11. Submission lands → file in Media Library tagged "form-upload" with asset_references entry → in-app notification → email reaches admin → delivery dashboard "sent"
12. Operator opens the uploaded image's "where used" → sees it's referenced by the site landing page; replaces the image with a new file; published page now renders new image (URL lookup stable)
13. Mini Ops Dashboard shows: 1 submission pending review, 0 pending approvals, 0 failed sends, 1 scheduled-unpublish in 60min, etc.
14. Operator tries to delete the form → SafeAction demands typed slug → form moves to trash → restore from TrashPage → form back; page block reference intact

**Pass conditions:**
- All 14 steps pass on clean environment
- All BlockType contract tests pass
- PHPStan custom-rule run clean
- Cross-tenant isolation test passes
- Pint/Pest/PHPStan all green
- Mini Ops Dashboard shows accurate counts throughout
- No regression in Phase D contract tests (25 still pass)

**Stage 1 closeout doc** generated in `docs/phase-3/stage-1-closeout.md`.

**Merge sequence:**
- `backup-now.bat` before merge
- Merge `phase-3` → `main`
- Tag `phase-3.1-foundations`
- Push main + tag
- Operator sign-off → Stage 2 begins

---

## Estimated timeline (v3) with distributed buffer

| Week | Focus | Working days |
|---|---|---:|
| 1 | A1 + A2 start | 5 |
| 2 | A1 finish + A2 wire-in + MED start + PR refactor + **Gate 1** | 5 |
| 3 | MED expanded features (where-used, replace, search) | 5 |
| 4 | MED finish + AssetPicker integration + iteration | 5 |
| 5 | A3 model + 7 BlockTypes + interface | 5 |
| 6 | A3 services + BlockBuilder + **Gate 2** | 5 |
| 7 | A3 public render + Browser Proof 2 | 5 |
| 8 | A4 Forms expanded + Browser Proof 3 | 5 |
| 9 | A5 editor + POL publish policies + scheduler + history | 5 |
| 10 | DSH 8-widget dashboard + integration polish | 5 |
| 11 | **Gate 3 dry-run** + buffer iteration | 5 |
| 12 | Exit Gate + closeout + merge | 5 |
| **Total** | | **60 working days = 12 weeks** |

Confidence: **medium**. Risks H1-H12 all carried forward with mitigations.

---

## What I report daily

```
Day N — Week M / 12
✅ shipped: [1-3 items]
🔨 in flight: [1-2 items]
⛔ blocked: [if any]
ETA Stage 1 close: [day from start, +/- 2d]
```

---

## What I will NOT do during Stage 1

- Touch Stage 2+ subsystem code (full Program Builder wizard, full Content Builder, etc. — only Program-edit refactor on existing form happens in Stage 1)
- Modify Phase D-frozen `/api/v1/*` endpoints (only ADD new ones)
- Run destructive DB ops on production data
- Open Closed Beta
- Commit to `main` (except via Stage 1 closeout merge)
- Skip a Review Gate or Browser Proof checkpoint
- Add SMS/WhatsApp/marketing pixel work
- Decide an architectural variant unilaterally
- Build features beyond what's listed (scope creep gets caught at Review Gates)

---

**Next action:** I'm creating `phase-3` branch off `main` (`b9e6850`) now and starting Week 1 Day 1 (module scaffolding + A1.1 WizardStep). Daily reports begin tomorrow.
