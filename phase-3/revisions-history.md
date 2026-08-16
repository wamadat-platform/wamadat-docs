# Phase 3 — Revisions History

Tracks every revision to Phase 3 stage plans. Each revision documents what changed, why, and what it cost in time.

---

## stage-1-implementation-plan.md — v2 (2026-05-15)

**Operator response to v1:** "approve with changes" — 9 specific modifications listed.

### Change-by-change incorporation

| # | Change | Incorporated as | Time delta |
|---|---|---|---|
| 1 | Priority: operational simplicity > clean architecture (tiebreaker) | Added as **Stage 1 Design Principle #0** at top of plan. Every conflict between "elegant abstraction" and "fewer clicks for operator" resolved toward fewer clicks, with the elegance-loss documented in the BACKLOG so we can revisit if it bites later. | 0 |
| 2 | Program Builder foundations in Stage 1 | • `pages.type` enum gains `program_landing` value from day 1.<br>• `PageModel` gains `owner_type` / `owner_id` polymorphic FK so a page can be "owned by" a Program (vs the default site-level page).<br>• Every BlockType ships a `withContext(ownerType, ownerId)` hook — a program landing's `PricingBlock` auto-pulls the program's price/currency; a generic site page's `PricingBlock` uses the operator-typed value.<br>• Test: `ProgramLandingPageBuildTest` builds a program landing using each of the 7 starter blocks. | +0.5 week |
| 3 | Stronger Block guardrails | Mandatory `BlockType` contract grows from 5 to 7 methods:<br>• `kind()`, `schema()`, `defaultPayload()`, `renderToHtml()`, `renderToMobileJson()` (new), `migrate(payload, fromVersion, toVersion)`, `previewSample()` (new).<br>• PHPStan custom rule fails CI if a class implementing `BlockType` is missing any.<br>• Contract test: every registered BlockType must produce desktop preview + mobile preview + schema-validating payload for `previewSample()` output. No exceptions. | +0.5 week |
| 4 | UX Review Gates (Week 2, Week 4, pre-Exit) | Formal review windows scheduled:<br>• **Review 1 — end of Week 2:** operator opens `/admin/dev/ux-library` and walks the demo. Marks each component "ship" / "redo". Redo items absorbed in Week 7 buffer.<br>• **Review 2 — end of Week 4:** operator builds a real page using the block builder. Marks each block + the builder flow itself.<br>• **Review 3 — end of Week 9 (pre-Exit Gate):** operator walks the full end-to-end scenario in dry-run before the official Browser Proof Checkpoint 4 ratifies Stage 1.<br>• Review windows are ASYNC (operator can do them at their own pace, no live calls needed). | +0.5 week |
| 5 | Media Library from Stage 1 | New `Modules/Media/` module:<br>• Single `media_assets` table — folder is a string path (`marketing/campaigns/2026`) not a separate table; tags are a Postgres text[] array, not a normalized tags table. Lean by design.<br>• `MediaUploadService` — uploads to Bunny, stores asset metadata.<br>• Image optimization via Bunny native URL variants (thumbnail/medium/large) — no separate processing service.<br>• Preview: thumbnail + dimensions + size shown in picker.<br>• `MediaLibraryResource` — Filament page with folder tree + grid + tag filter + search.<br>• `AssetPicker` component (was A1.3) pulls from Media Library; can also upload-and-add inline.<br>• Reusable EVERYWHERE images/videos/files appear: program covers, lesson videos, page blocks, partner logos, review videos. | +1.0 week |
| 6 | Approval Workflow flexibility (3 modes) | • `publish_policies` table per (entity_kind, role_name): `mode` enum: `direct` / `review_required` / `scheduled`.<br>• Seeded sensibly: academy_owner = direct on all; instructor = review_required on Program/Lesson; marketing = review_required on Page/Block (matches Decision 3).<br>• "Scheduled publish" UI: operator picks a date+time → record gets `scheduled_publish_at` → cron-driven `RunScheduledPublishes` job publishes when time arrives.<br>• Audit log captures schedule, schedule-cancel, schedule-fire events. | +0.5 week |
| 7 | Communication editor — simplified | **Net SIMPLIFICATION, not addition.**<br>• Drop full TipTap rich-text editor — use a minimal toolbar (bold, italic, link, ordered list, unordered list). That's it.<br>• Add: template categories enum (`transactional` / `marketing` / `notification` / `system`) with filter pills above the list.<br>• Add: variable picker as inline chip-pills, click to insert at cursor.<br>• Add: "test send to me" button — sends rendered template to current admin's email with sample data.<br>• Arabic language switcher prominent, default ar.<br>• Drop: HTML edit mode (operator-unfriendly). | **-0.5 week** |
| 8 | Mini Operations Dashboard | New widget `NeedsActionWidget` at top of `/admin` dashboard:<br>• Pending approvals count + link to filtered list<br>• Draft pages count + link<br>• Unpublished blocks-with-changes count + link<br>• Scheduled-publishes upcoming-24h count + link<br>• Failed email outbox count + link (existing data)<br>• Pending refunds count + link (existing data)<br>• Each row clickable → filtered resource list opens with the relevant scope. | +0.5 week |
| 9 | Cross-domain reusability discipline | • Every block kind passes the cross-domain test: works on `program_landing`, `cms_static`, `cms_campaign`, and (placeholder type) `consultant_profile` page types.<br>• Every form works whether embedded in any page type OR called from mobile.<br>• Every Filament resource exposes its data via `GET /api/v1/admin/<resource>` JSON endpoint (auth required, role-gated) so a future mobile admin reuses the same backend.<br>• Design checklist added to every Stage 1 PR template: "does this serve programs / pages / consultations / campaigns / mobile?" must be answered. | +0.5 week |

### Net time delta

| | Weeks |
|---|---:|
| v1 estimate | 7 (6 dev + 1 buffer) |
| Changes 2,3,4,5,6,8,9 | +4.0 |
| Change 7 (simplification) | -0.5 |
| **v2 estimate** | **10.5 → round to 10 weeks** |

**Stage 1 v2 timeline: 10 weeks** (8.5 weeks dev + 1.5 weeks buffer for the 3 review-gate iterations).

### What this delta means for downstream stages

Stage 2 estimate (Content layer — Program Builder + Content Builder + Reviews) was 8 weeks. With Stage 1 v2 doing more foundational work, Stage 2 may shrink by ~1-2 weeks (Program Builder's data + block underpinnings already in place). **Revised Stage 2 estimate: 6-7 weeks** (down from 8).

So overall Phase 3 trimmed scope:
- v1 estimate (Stage 1 + 2 + 3 + 4 + 5 + 6): ~5-6 months
- v2 estimate (with Stage 1 expanded, Stage 2 shrunk): ~5-6 months

**No net Phase 3 extension** — the Stage 1 expansion is absorbed by Stage 2 simplification.

### Risk register updates

New risks introduced by the v2 changes:

| Risk | Severity | Mitigation |
|---|---|---|
| Media Library scope-creep into "image editor" features | Medium | Lean schema (no separate folders table, no separate tags table) — sticks to "find / upload / preview". No crops, no filters in Stage 1. |
| Scheduled publish job missed firing | Medium | Cron heartbeat + alert (uses existing failed-jobs alerting); idempotent job design. |
| Cross-domain abstraction over-generalization | Medium | Hard rule: contracts are minimum-viable; only add a generic interface when ≥2 consumers exist. No "future-proof" abstractions for hypothetical consumers. |
| Approval workflow becomes confusing UI | Medium | Per-content-type "publish mode" picker is HIDDEN from non-owner roles. Default behavior is the seeded policy. Operator only sees the mode switch in settings, not every edit. |
| BlockType contract growth blocks shipping | Low | Mobile JSON method default implementation is `array_merge(['kind' => $this->kind()], $payload)` — minimum-viable mobile representation that's good enough for v1. Only override if the mobile renderer needs something specific. |

### Operator-facing summary (1 sentence per change)

1. ✅ Architecture rule baked in: operational simplicity wins ties.
2. ✅ Program landing pages are part of Stage 1, not a Stage 4 surprise.
3. ✅ Every block has enforced contracts; CI fails on contract violation.
4. ✅ Three formal review windows scheduled (weeks 2, 4, 9).
5. ✅ Media Library in Stage 1 — folders, tags, reusable picker, image optimization via Bunny.
6. ✅ Three publish modes (direct / review / scheduled) configurable per role per content type.
7. ✅ Communication editor simplified — minimal toolbar, categories, variable chips, test-send.
8. ✅ Mini ops dashboard widget shows pending operator tasks at top of /admin.
9. ✅ Every Stage 1 contract works across programs / pages / consultations / campaigns / future mobile.

Timeline: **10 weeks (was 7)**. Overall Phase 3 unchanged.

---

## stage-1-implementation-plan.md — v3 (2026-05-15)

**Operator response to v2:** "go with changes" — 9 further refinements emphasizing real operational proof over component showcase.

### Change-by-change incorporation (v3)

| # | Change | Incorporated as | Time delta |
|---|---|---|---:|
| 1 | Each Gate must prove operational pain relief, not internal architecture | Every Review Gate scenario rewritten — operator interacts with REAL flows (create/edit a program, upload an image, search the library) not "open the demo URL". Component showcase removed. | 0 (process) |
| 2 | Week 2 Gate = real operational proof | The Gate 1 scenario now demands: edit an existing Program through the refactored UX (A1 wizard + A2 SafeAction + AssetPicker from Media Library); upload an image to Media Library; preview; publish via SafeAction; verify RTL is clean; verify usable on mobile viewport. Forces foundations-applied-to-Program-resource by Week 2. | +0.5 week |
| 3 | Media Library — expanded scope | Add: `asset_references` table tracking every entity using each asset; replace-without-break (entities reference assets by UUID, not URL; render-time URL lookup); "where used" inspector on every asset; copy-URL action; image + video preview in grid; folder search + tag search + filename search combined; replace-file action keeps UUID stable. | +0.5 week |
| 4 | Block System guardrails confirmed | Already enforced in v2 (7-method contract + PHPStan rule + previewSample test). No additional change. | 0 |
| 5 | Forms Builder — expanded scope | Add: `submission_audit_log` (every submission emits an audit event with IP, user-agent, field-response diff); `SpamCheck` interface (abstract trait so honeypot today + future ReCaptcha plugs in); export filtering UI (date range, status, field-value filters before export); rate-limit settings per-form (not just global). | +0.3 week |
| 6 | Publish Policies — expanded scope | Add: `scheduled_unpublish_at` column (entity can be auto-unpublished at a future date — promotional pages, time-limited offers); `publish_history` table (every publish, unpublish, schedule, schedule-cancel, schedule-fire emits a row); per-entity "Publish History" timeline view in admin. | +0.3 week |
| 7 | Mini Ops Dashboard — fully actionable 8-widget | Grow from 6 to 8 click-through widgets: pending approvals · failed email sends · **abandoned checkouts** (new) · draft pages · **unpublished programs** (new — separate from draft pages) · **upcoming live sessions next 24h** (new) · failed jobs · recent refunds (last 7 days). Each widget shows count + 1-line actionable sub-text (e.g., "12 pending — oldest 3 days") + click → filtered list. | +0.5 week |
| 8 | Single-screen-per-task UX rule | Codified as Stage 1 Design Principle #6: if a logical task forces the operator across more than one screen, the design is wrong. Forces modals-not-pages for child operations (e.g., editing a lesson is a modal on the program edit page, not a separate page). | 0 (discipline) |
| 9 | Operational UX > developer convenience (Phase 3 wide) | Re-affirmed; integrated with Principle #0. | 0 (discipline) |

### Net time delta (v2 → v3)

| | Weeks |
|---|---:|
| v2 estimate | 10 |
| Changes 2, 3, 5, 6, 7 | +2.1 |
| Discipline rules 1, 4, 8, 9 | 0 |
| **v3 estimate** | **~12 weeks** |

### What this delta means for downstream stages

The Week 2 Gate change (Change #2) pulls Program-edit UX into Stage 1's scope. This was originally Stage 2's first deliverable. So Stage 2 shrinks further.

| Stage | v1 | v2 | v3 |
|---|---:|---:|---:|
| 1 — Foundations + operationally-proven gates | 7w | 10w | **12w** |
| 2 — Content (Program Builder + Content Builder + Reviews) | 8w | 6-7w | **5w** (Program-edit done; only full wizard + Content Builder + Reviews left) |
| 3 — Commerce | 8w | 8w | 8w |
| 4 — Marketing + CMS | 12w | 12w | 12w |
| 5 — Live ops + aggregation | 8w | 8w | 8w |
| 6 — Browser proof closeout | 4w | 4w | 4w |
| **Phase 3 total (trimmed scope)** | **~47w** | **~48w** | **~49w** |

Overall: ~1 week longer than the original Phase 3 estimate. Within acceptable variance for the scope additions.

### Risk register updates (v3)

| Risk | Severity | Mitigation |
|---|---|---|
| Week 2 Gate too aggressive | High | If by end of Week 1 we can't demo a clean RTL ProgramResource refactor, the Gate scope shrinks to "demo on a simpler resource (Category)" and the Program refactor moves to Week 3. Stage 1 timeline holds; Week 2 Gate doesn't slip. |
| `asset_references` table makes Media Library writes slow | Medium | Update via async job, not synchronous trigger. Reads (where-used inspector) are eventually-consistent (~60s stale acceptable). |
| Scheduled unpublish race conditions | Low | Idempotent unpublish job. If a manual publish happens between schedule-fire and unpublish-fire, audit log captures both — operator sees the conflict. |
| 8-widget dashboard slows admin loading | Low | All widget queries cached in Redis 60s. Dashboard renders skeleton first, then fills in. |
| Single-screen rule conflicts with deep workflows | Medium | Modals stack up to 2 levels (page → lesson modal → quiz inside lesson — that's the limit). Deeper edits force a separate page. |

### Operator-facing 1-sentence-per-change

1. ✅ Every Review Gate is a real operational scenario, not a component demo.
2. ✅ Week 2 Gate edits a real Program via refactored UX + Media Library + SafeAction.
3. ✅ Media Library tracks asset usage and supports replace-without-breaking-links.
4. ✅ Block System guardrails already enforced in v2; no change.
5. ✅ Forms get per-submission audit + spam abstraction + export filtering.
6. ✅ Publish policies include scheduled unpublish + per-entity history timeline.
7. ✅ Ops Dashboard has 8 actionable widgets, each click-through to filtered work.
8. ✅ Single-screen-per-task UX rule codified.
9. ✅ Operational UX > developer convenience confirmed Phase 3-wide.

Timeline: **12 weeks (was 10)**. Phase 3 total: ~49 weeks vs original ~47w — small overall growth.
