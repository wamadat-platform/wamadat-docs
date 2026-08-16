# Phase 3 — Stage 0 Closeout (Decisions Locked)

**Date:** 2026-05-15. **Operator sign-off:** received in chat. **Status:** locked — these decisions drive every subsequent stage and cannot change without a written re-decision.

## The 5 locked decisions

### Decision 1 — Scope: TRIMMED Phase 3 (16 of 21 subsystems)

**Trimmed in (Phase 3, ~5-6 months):**

| # | Subsystem |
|---|---|
| A1 | Admin UX Library |
| A2 | Safety Rails |
| A3 | Block System Core |
| A4 | Forms Builder |
| A5 | Communication Center (email + in-app only) |
| B1 | Program Builder |
| B2 | Program Content Builder |
| B3 | Reviews + Testimonials |
| C1 | Coupons + Promotion Codes |
| C2 | Payment Integrations (Moyasar + HyperPay) |
| C3 | Orders + Abandoned Checkout |
| C4 | Refund Workflow Hardening |
| C5 | Financial Operations Dashboard |
| D1 | Landing Page Builder |
| D2 | Website CMS |
| D3 | Partners + Sponsors |
| E1 | Zoom + Live Sessions |
| F1 | Operations Dashboard |
| G1 | Browser Proof Suite |

(19 entries — Refund Workflow and Operations Dashboard counted as separate items though they were sub-divisions earlier.)

**Deferred to Phase 4:**

| Deferred | Why |
|---|---|
| SMS channel (A5) | 50-user Beta is small enough that email + in-app cover all notification needs |
| WhatsApp channel (A5) | Same as SMS + Meta template approval adds 2-4 weeks |
| Marketing pixels / UTM / tracking (D4) | Pixels need traffic; invitation-only Beta has invitation-only traffic |
| Consultations Rebuild (E2) | Existing form-based consultations work for Beta; rebuild becomes Phase 4 product |
| Countdown / Map / SCORM blocks | Marginal value vs core 17 block types |

These deferrals are written, not implicit. If we miss something Phase 3 needs that's currently deferred, the Stage closeout for the affected stage must call it out and re-open a scope conversation BEFORE proceeding.

### Decision 2 — Beta timing: open AFTER Stage 2 ships

Closed Beta launches at end of Stage 2 (target week ~14-15 from Phase 3 start), once Program Builder + Content Builder + Reviews extension are merged + browser-proven.

**Beta operational invariants (carried from `06-closed-beta.md`):**
- Max 50 verified users
- Max 5 published programs
- No public marketing
- No SLA
- Daily summary monitored

**Additional Phase-3-specific Beta invariants:**
- Beta launches on the `phase-3` branch (NOT a separate beta branch). Stage 2's exit-gate commit gets merged to `main` and tagged `phase-3.2-content`.
- Production deploys from `main` only.
- Rollback procedure: revert to the previous `main` tag (which is `phase-3.1-foundations`'s merge tip) AND restore the pre-deploy DB backup. Documented in `docs/operations/02-rollback.md` already.
- During Stages 3-6 (commerce + marketing + live ops + final), Beta runs on Stage 2's tagged main + bug fixes only. New feature work continues on `phase-3` branch until next stage merge.

### Decision 3 — Approval workflow: instructor proposes → owner publishes

**Governance rule:** any new or modified content that's visible to learners or customers requires:
- An **author** (any role with content-write permission) submits the change as a **proposal**.
- An **approver** (academy_owner or admin role only) reviews and publishes OR rejects with a note.

**Scope of governed content:**

| Entity | Author roles | Approver roles |
|---|---|---|
| Program (create/edit) | academy_owner, admin, instructor | academy_owner, admin |
| Lesson (create/edit) | academy_owner, admin, instructor (own programs) | academy_owner, admin, instructor (own programs) |
| Block / Page (landing or CMS) | academy_owner, admin, marketing | academy_owner, admin |
| Review (publish/hide moderation decision) | academy_owner, admin, support | (single-step; no second approver) |
| Coupon (create/edit if discount > 25%) | academy_owner, admin, marketing | academy_owner, admin |
| Coupon (create/edit if discount ≤ 25%) | academy_owner, admin, marketing | (single-step; no second approver) |
| Refund (any) | academy_owner, admin, finance | different user in same role set, separation-of-duty enforced |
| Form (create/edit) | academy_owner, admin, marketing | (single-step) |
| Partner (create/edit) | academy_owner, admin, marketing | (single-step) |

**Implementation pattern (used by every governed entity):**
- A `proposed_changes` JSONB column on the model (or a sibling `*_drafts` table for entities where the draft itself is a versioned snapshot — programs and pages).
- Status transitions: `draft` → `in_review` → `published` (or `rejected` back to `draft`).
- Audit log captures: proposer, reviewer, before-state, after-state, reason for reject (if any).
- A "My Pending Approvals" admin widget shows owner/admin queues.

**Permission keys** added to Spatie role permission seeds in Stage 1:
- `*.propose` (author capability)
- `*.publish` (approver capability)
- Mapped per role per entity above.

### Decision 4 — Block storage: versioned JSONB with strict schema discipline

**Pattern locked:**
- Each page (landing or CMS) stores its current blocks as a single `pages.blocks` JSONB column (an ordered array of block objects).
- Every time a page is published, a `page_versions` row is created storing the full blocks snapshot at that version. Operator can rollback to any version.
- The blocks JSONB is **NOT free-form**. Each block object MUST conform to a schema validated server-side at every save.

**Schema discipline rules:**
1. Each block kind (`hero`, `text`, `image`, `cta`, ...) has a dedicated PHP class implementing `BlockType` interface. Single source of truth for the kind.
2. The PHP class declares: `kind` slug, `schema()` (Laravel validation rules), `defaultPayload()`, `renderToHtml()` (Blade), and `migrate(old, fromVersion, toVersion)` for schema evolution.
3. Every block carries a `schemaVersion` integer. Reads route through the latest registered `migrate` chain to bring old payloads forward without DB writes.
4. Adding a new block kind = new PHP class + registration line. No migration.
5. Evolving an existing block kind = bump its `schemaVersion` + add a `migrate` step. No DB write; old pages keep their old version stamps and lazy-upgrade on next save.
6. Forbidden in payloads: arbitrary fields not declared in `schema()`. Validator rejects extras (strict mode).

**Tables created (Stage 1 migration):**

```
pages
  id (uuid)
  slug (unique per tenant)
  type (enum: landing | cms_static | cms_campaign | consultant_profile)
  title_ar, title_en
  status (enum: draft | in_review | published | archived)
  published_version_id (FK page_versions.id, nullable)
  draft_blocks (jsonb)              -- current edit state
  seo (jsonb)
  metadata (jsonb)
  created_at, updated_at, deleted_at
  audit fields (created_by, last_edited_by, published_by)

page_versions
  id (uuid)
  page_id (FK pages.id)
  version_number (int, increments per page)
  blocks (jsonb)                    -- frozen snapshot
  seo_snapshot (jsonb)
  created_at
  published_by (FK users.id)
  reason (text, optional)
```

**Why this beats the polymorphic-table alternative:**
- Render is O(1) DB read (one row → JSONB tree).
- Version history is O(1) snapshot insert at publish, not N row writes per block.
- Rollback is "set published_version_id = X; copy blocks from version X to draft_blocks".
- Schema discipline lives in PHP code (testable, auditable in git), not DB constraints.

**The cost of this choice (acknowledged):**
- Searches across blocks ("find all pages containing a `Hero` block with headline X") need JSONB ops. Postgres handles this with GIN indexes — added in the migration.
- Schema evolution requires a migration STEP in each affected BlockType class, which is more code than a DB ALTER. Trade-off accepted.

### Decision 5 — Branch strategy: `phase-3` long-lived branch

**Branch model:**

```
main ──●──●──●──●──── (production tags only)
        \         ↑
         \        └─ merge from phase-3 at each Stage boundary
          \
phase-3 ──●──●──●──●──●── (active dev; rebased on main as needed)
            ↑
            └─ daily work commits here
```

**Tagging schema** (production milestones):
- `phase-3.1-foundations` — Stage 1 closeout merge
- `phase-3.2-content` — Stage 2 closeout merge (Beta opens at this tag)
- `phase-3.3-commerce` — Stage 3 closeout merge
- `phase-3.4-marketing` — Stage 4 closeout merge
- `phase-3.5-live-ops` — Stage 5 closeout merge
- `phase-3-closed` — final merge after Stage 6 browser proof

**Merge gate per stage:**
- All listed deliverables produced
- Browser proof checkpoints passing
- Operator sign-off on the stage closeout doc
- Pre-merge DB backup taken (use existing `backup-now.bat`)
- Migration plan written for any DB changes in the stage
- Rollback steps documented in the closeout

**Hotfixes during a stage:**
- Production bugs found mid-stage (not Phase 3 features, real prod issues) go to `main` directly OR a short-lived `hotfix-<date>` branch off main. Phase 3 branch rebases on main after the hotfix to absorb it.

**Operator's mental model:** `main` is what's live. `phase-3` is what's coming. Every stage tag is a "we've shipped this much" snapshot.

---

## Implications of the locked decisions

### Scope discipline (Decision 1)

I will refuse to add SMS, WhatsApp, marketing pixels, or full consultations rebuild work during Phase 3 even if it seems easy or "while we're in there." If you change your mind, that's a written re-decision — not a slip.

### Beta after Stage 2 (Decision 2)

Stages 3, 4, 5, 6 are running while Beta is LIVE. This raises the bar on every merge to `main` after Stage 2 — each one affects real users. Mitigation:
- Stage 3+ merges happen during low-traffic windows (overnight Saudi time).
- Each Stage 3+ merge has a pre-merge browser proof + a 24h smoke-watch period before considering it "settled".
- Rollback decision is the operator's, but the runbook is pre-baked.

### Approval workflow (Decision 3)

Schema additions required NOW in Stage 1:
- `proposed_changes` JSONB column added to programs, lessons, pages, blocks, coupons, partners, forms — but actually we'll prefer the `*_drafts` sibling table for programs and pages (snapshot pattern). Single column for entities where drafts are simpler (coupons, partners, forms).
- New Spatie permissions seeded (`*.propose`, `*.publish` per entity).
- Filament resources gain "Submit for review" vs "Publish" buttons gated by permission.

This is foundation work, not Stage 1 feature work — but it DOES change Stage 1 migrations.

### Versioned JSONB (Decision 4)

The `pages` + `page_versions` tables ship in Stage 1 even though they're consumed mainly by Stage 4 (D1 Landing + D2 CMS). Reason: Form-as-Block (Stage 1's A4) renders inside a page's block tree, so the page model needs to exist before forms can be embedded.

### Branch strategy (Decision 5)

First Stage 1 action (before any code): create `phase-3` branch off `main` (currently at `d60d8d8`). All Stage 1 commits land on `phase-3`. No work on `main` except hotfixes for production issues.

---

## Trimmed subsystem dependency re-check

Removing E2 (Consultations) and D4 (Marketing pixels) from the dependency graph:
- D1 Landing Page Builder no longer needs to render consultation booking widgets — only program landing. Simpler.
- D2 Website CMS no longer needs to render the Consultations index page — keeps a placeholder "Coming Soon" page or links to the legacy form-based consultations.
- F1 Operations Dashboard no longer surfaces consultation booking metrics — one less widget.
- C5 Financial Operations dashboards no longer split revenue by consultations vs programs — programs only.

No subsystem outside the deferred set loses functionality. The graph holds.

---

## Risk register at Stage 0

These are the risks I see entering Stage 1. Each gets a row in stage closeouts as it evolves.

| Risk | Severity | Stage | Mitigation |
|---|---|---|---|
| Block schema evolution breaks live pages | High | 1, 4 | Strict schema versioning + `migrate` chain + tests per block kind |
| Filament 3 + RTL drag-drop edge cases | Medium | 1, 2 | POC the drag-drop in week 1; fall back to up/down buttons if Filament drag-drop fails on RTL |
| Forms file-upload security (no virus scan in Beta) | Medium | 1 | File type whitelist + size cap + signed URLs + admin manual review queue for first 30 days |
| Approval workflow makes simple edits feel slow | Medium | 2 | "Auto-approve own changes if role = academy_owner" optimization |
| Beta users hit a Phase 3 bug not covered by tests | High | 3+ | Daily summary + 3-channel failed-jobs alert + small cohort + direct support |
| Branch divergence between phase-3 and main | Medium | All | Weekly rebase discipline; CI runs on phase-3 same as main |
| Hidden tenancy bugs in new code | High | All | Every Phase 3 migration goes in both `migrations/landlord/` and `migrations/tenant/` correctly; new models declare `UsesTenantConnection` if tenant-scoped |
| ZATCA Phase 2 e-invoicing not implemented | Medium | 3 (C5) | C5 ships VAT summary; ZATCA Phase 2 invoicing flagged as a P4 task (Saudi-specific compliance gap) |
| Operator overload reviewing each stage | Low | All | Each stage closeout is ≤ 1 page; bulk approval at end of stage, not per-feature |
| Phase 3 takes longer than 6 months | Medium | 3+ | Each stage has explicit "if running late by 2 weeks, cut these items" sub-plans (added in stage plans) |

---

## What I commit to before Stage 1 begins

1. **No code change** until you approve `stage-1-implementation-plan.md`.
2. **No branch creation** until you approve.
3. The Stage 1 plan will list exact files, exact migrations, exact tests, exact browser-proof checkpoints, and the realistic estimate (with buffer).
4. Stage 1's plan also defines the **stage-closeout report shape** — a 1-page operator-facing doc you'll see at the end of Stage 1.

---

## Reporting cadence (operator-facing)

| Cadence | Output | Audience |
|---|---|---|
| Daily (during stages) | A single line in chat: "today shipped X, blocked on Y, ETA unchanged / +Nd" | Operator |
| Stage boundary | 1-page closeout doc: shipped, deferred, risks updated, next stage estimate | Operator |
| Inter-stage | None (gap between stages is operator-only review time) | — |

No standups, no Jira, no kanban. The daily line + closeout doc is the entire reporting surface.

---

**Next deliverable in this directory:** `stage-1-implementation-plan.md` (already produced — read alongside this).
