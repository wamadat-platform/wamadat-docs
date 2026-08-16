# Phase 3 — Wave 1 FAST TRACK (single working plan)

**Mode:** Operational UX > developer convenience > architecture purity. Daily shippable. No foundations-first. Inline primitives built inside the feature that needs them. No fake UI.

**Plans v1/v2/v3 (the 12-week sequencing report):** archived as historical. **Working plan: this file only.**

**Branch:** `phase-3`. Daily commits. Each item shipped with browser proof.

---

## Wave 1 — 8 systems, ordered for shipping velocity

Order chosen by: smallest+visible first → biggest+dependent later.

| Day | System | Why this order | Browser proof |
|---|---|---|---|
| **1** | Mini Ops Dashboard MVP | 7 widgets querying existing data — operator sees value immediately, validates platform pulse | `/admin` shows the widget; counts match reality; clicks navigate |
| **2-4** | Media Library MVP | Needed by Program covers (Item #1 from operator's list) — basic version: upload, list, grid, tag, pick | Upload an image → see in library → pick from another resource |
| **5-14** | Program Management Rebuild | Largest single Wave 1 item — wizard, RTL, draft/publish, cover via Media Library, SEO, schedule | Create program from scratch → publish → see on storefront |
| **15-23** | Program Content Sections | Needs program from Item #2 — sections + 8 content types + drag-drop | Build full program curriculum |
| **24-27** | Orders Operations | Refactor existing OrderResource — line items view, customer timeline, abandoned cart tab, quick actions | Resend payment link → customer pays → captured |
| **28-30** | Coupons enhancements | Existing model rich — UI gains scope picker (not JSON), redemption analytics | Apply coupon at checkout, verify discount + tracking |
| **31-33** | Reviews + Partners (parallel small) | Both small — Reviews refactor + Partners new resource | Add review, mark featured, public page shows it; add partner, public page shows logo |
| **34-35** | Wave 1 browser proof + merge to main | Tag `phase-3.wave-1` | Full 10-step operator scenario passes |

**Realistic total: ~7 weeks = 35 working days.** Operator's "few days" goal: Day 1 ships visible UI; Day 4 ships Media Library; Day 14 ships full Program Management — that's "daily shippable" within Wave 1.

---

## Per-system specs (just enough — not over-planning)

### 1. Mini Ops Dashboard (Day 1)
**Widget rows (7 working today; 1 deferred):**
| Row | Source | Count query | Click to |
|---|---|---|---|
| Failed payments (24h) | `payments.status='failed' AND failed_at > NOW()-24h` | exists | OrderResource (manual filter) |
| Abandoned checkouts | `orders.status IN (pending,awaiting_payment) AND placed_at > NOW()-7d AND paid_at IS NULL` | exists | OrderResource |
| Draft programs | `programs.status='draft' AND deleted_at IS NULL` | exists | ProgramResource |
| Unpublished lessons | `lessons.status='draft' AND deleted_at IS NULL` | exists | LessonResource |
| Upcoming live sessions (24h) | `live_sessions.starts_at BETWEEN NOW() AND NOW()+24h` | exists | LiveSessionResource |
| Failed jobs | `failed_jobs` count | exists | FailedJobResource |
| Recent refunds (7d) | `refunds.created_at > NOW()-7d` | exists | (no refund resource yet — link to OrderResource) |
| Pending approvals | DEFERRED — needs publish workflow | — | added later |

Cached 60s in Redis. Manual refresh.

### 2. Media Library MVP (Day 2-4)
- 1 table: `media_assets` (id uuid, filename, original_filename, mime, size_bytes, url, thumbnail_url, folder_path text, tags text[], uploaded_by_user_id, created_at, updated_at, deleted_at)
- 1 table: `asset_references` (asset_id, entity_type, entity_id, field, created_at) — populated by observer
- Upload via existing Bunny pipeline (Signed URL service from Phase D)
- Filament resource: list view (grid + folder sidebar + tag filter + search box)
- AssetPicker component (used by Program Manager later)
- "Where used" inspector on each asset row
- Replace file: keeps UUID, swaps URL — all references auto-update on next render
- Image preview inline (img src), video preview (HTML5 video tag)

**Skipped for Wave 1, Phase 4:** image optimization beyond Bunny URL variants, video transcoding, advanced search, virus scanning.

### 3. Program Management Rebuild (Day 5-14)
- Refactor existing `ProgramResource.php` into a `Wizard` Filament form (5 steps)
- Steps: Basics → Category/Trainer → Pricing/Schedule → Media (uses AssetPicker) → SEO/Publish
- Draft autosave on step navigation
- "Save as draft" vs "نشر الآن" buttons
- SafeAction confirmation on publish + delete (SafeAction primitive built inline)
- Preview-as-student link (read-only storefront route)
- RTL verified
- Browser proof: create new "إدارة الوقت" program from scratch, publish, view on storefront

### 4. Program Content Sections (Day 15-23)
- Use existing `ProgramModuleModel` + `LessonModel` (rich already)
- New ContentBuilder page nested under ProgramResource edit
- Drag-drop: modules + lessons (Livewire + SortableJS, RTL-tested)
- Lesson type picker on add: video (Bunny via AssetPicker) / YouTube / Vimeo / Zoom URL / PDF (via AssetPicker) / Word/PPT (via AssetPicker) / audio / link / embed / text
- Per-lesson modal: title, description, duration, required/free-preview flags, drip rule
- Each lesson type uses the existing `LessonModel.type` enum + `media_provider` field
- Publish/unpublish per lesson
- Browser proof: build a 4-module / 12-lesson program

### 5. Orders Operations (Day 24-27)
- Refactor `OrderResource.php` — form() returns real fields now (was `[]`)
- Show line items (order_items relation)
- Show payments (multiple per order)
- Show customer timeline (signup → cart → checkout → abandoned → recovery → paid)
- Tabs in list view: All / New / Pending payment / Abandoned / Paid / Refunded / Failed
- Quick actions: resend payment link (existing CheckoutService method) / cancel with reason / refund (existing flow)
- Browser proof: customer abandons → operator clicks "resend payment link" → customer pays

### 6. Coupons enhancements (Day 28-30)
- Refactor `CouponResource.php` — replace JSON scope inputs with structured picker (multi-select of programs / categories / users)
- Add: redemption count column on list view
- Add: per-coupon analytics page (redemptions, revenue impact, conversion rate)
- Add: simulate-impact button ("this coupon would have applied to X orders last month, discounting Y SAR total")
- Existing `CouponService.apply()` validates at checkout — no model changes
- Browser proof: create 10% coupon for Program X → apply at checkout → verify discount + redemption_count++

### 7. Reviews Management (Day 31-32)
- Refactor `ReviewResource.php` — gain `is_featured` toggle, moderation queue tab (filter `moderation_status='flagged'`)
- Featured reviews show first on program pages
- Browser proof: add review → mark featured → see it sorted-first on program storefront

### 8. Partners Management (Day 33)
- New module `Modules/Partners/`
- 1 table: `partners` (id, name_ar, name_en, logo_asset_id (FK media_assets), link_url, sort_order, is_active, created_at, updated_at, deleted_at)
- Filament resource (list + form)
- Public block placeholder on homepage (just render partner logos in grid; full landing builder waits for Wave 2)
- Browser proof: add 3 partners → visible on homepage

---

## Daily reporting

```
Day N — Wave 1 / 35 days
✅ shipped: [item + commit hash]
🔨 in flight: [item]
⛔ blocker: [if any, with options]
ETA Wave 1 close: Day 35 (+/- 2)
```

---

## Decision-making in fast track

When a small architectural choice comes up that doesn't have a clear "operator preference" answer, I decide myself per these defaults:
- Prefer existing model/migration > new
- Prefer existing service > new
- Prefer Filament native > custom Livewire
- Prefer one-page form > multi-page (Principle: single-screen-per-task)
- Prefer modal > navigate-away
- Prefer Arabic-first labels everywhere
- If unsure between two approaches, pick the one shipping today

If a REAL blocker appears (something I can't decide without operator input — payment gateway choice, brand color, etc.) → I state the blocker + 2 options + my recommendation, ship something else in parallel, wait for response.

---

## What's out of Wave 1

- Landing Page Builder, Forms Builder → **Wave 2**
- Consultation rebuild, Zoom SDK, Communication center polish → **Wave 2**
- Block system / CMS / generic abstractions → **Wave 3**
- Refactoring existing working features for elegance → **never** (operational simplicity wins)
