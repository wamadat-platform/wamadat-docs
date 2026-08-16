# Site Control Layer — V3 Architecture Blueprint
### Site Pages + Site Identity as the Master Control Layer of WAMADAT

> **Status:** 2.0 Charter artifact — DESIGN ONLY. No code authored. Building is gated behind the
> Soft-Launch freeze, the real-user LAN test, and the Content Stream (see *Governance Gate*, §F).
> Author date: 2026-06-10. Baseline: current `pages` + `site_settings` schema.

---

## A. Council Verdict (one paragraph)

The two sections are **not** broken — the Page Builder + Site Settings shipped (2026-05-16) are a genuine
no-code win and correctly separate *engine pages* (fixed routes) from *composed pages* (`/{slug}`). But they
were designed for **a brochure site, not a control layer**. Today every connective tissue between the public
site and the platform's real entities (Programs, Trainers, Paths, Labs, Initiatives, Partners, Reviews) is
**typed by hand**: nav URLs, footer URLs, CTA targets, "featured" content. At 5 programs this is invisible;
at 100 programs / 50 trainers / hundreds of landing pages it becomes a **broken-link and stale-content
factory**. The fix is one principle applied everywhere: **Reference, never copy. Resolve, never hard-type.**

---

## B. The Ten Gaps (Final Deliverable items 1–10)

| # | Gap | Today | Risk at scale |
|---|-----|-------|---------------|
| 1 | **Current weaknesses** | `pages` is one flat type; blocks are all static; `site_settings` is one row with free-text URL arrays | No way to express landing pages, legal pages, or entity-driven content; chrome rots |
| 2 | **Missing relationships** | Pages know nothing about Programs/Trainers/Paths/Labs/Initiatives/Partners/Reviews | "Featured programs" = manually re-typed cards that drift from the catalog |
| 3 | **Missing governance** | No version history, no redirects, no scheduled publish, no preview-of-draft, no per-section permissions | A bad edit is unrecoverable; renamed slug = dead inbound links + lost SEO |
| 4 | **Missing UX** | No page templates, no block library reuse, no live preview, no mobile preview, no duplicate-page | Operator rebuilds the same hero 40 times; no confidence before publish |
| 5 | **Missing marketing** | No popups, announcement bars, campaign banners, countdowns, lead magnets, A/B | Every campaign needs a developer or a hacked Custom-HTML block |
| 6 | **Missing automation** | Nav/footer links typed manually; "featured" picked manually; no auto-collections | Linear human cost grows with catalog size; guaranteed drift |
| 7 | **Missing scalability** | Single menu set; single locale; single brand assumption baked into UI | Multi-academy / multi-language / partner-portal needs a redesign |
| 8 | **Missing analytics** | No per-page views, no CTA/funnel tracking, no campaign attribution | Cannot tell which page or campaign converts |
| 9 | **Missing SEO** | Only `seo_title` + `seo_description` per page; no global SEO, OG, canonical, sitemap policy, structured data, robots | Invisible to search; no rich results for Programs/Courses |
| 10 | **Future risks** | Custom-HTML block is an unsandboxed escape hatch; reserved-slug list is hardcoded; no content ownership/audit | Security (XSS), brittle routing, no accountability |

---

## C. Phase Findings (1, 2, 5, 6, 7 — the analytical phases)

### Phase 1 — Entity Relationship Map (current vs target)

```
CURRENT (islands):
  site_settings ──(free-text href)──>  ??? (string, may 404)
  pages.sections[] ──(operator re-types)──> copies of Program/Trainer data
  Programs / Trainers / Paths / Labs / Initiatives / Partners / Reviews  ← unreachable from Pages/Identity

TARGET (one graph):
  ┌─────────────── Site Control Layer ───────────────┐
  │  Menus (header/footer/legal/mobile)               │
  │     └─ items: TypedLink ──resolve──┐              │
  │  Pages (system|standard|landing|legal|collection) │
  │     └─ blocks: { static } | { data-bound ref } ───┤
  │  Global Config (brand, SEO, OG, tracking, contact)│
  └───────────────────────────────────────────────────┘
              │ resolve refs at render time
              ▼
   Programs · Paths · Labs · Initiatives · Trainers · Team · Partners · Reviews · Categories · Media
   (each remains the SINGLE SOURCE OF TRUTH for its own data)
```

**Missing relationships:** Page→Entity (data-bound blocks); Link→Target (typed, resolvable); Menu→Page
(auto-propose); Page→Media (already via AssetPicker — keep); Page→Locale (translations); Page→Campaign.
**Duplicate ownership:** "featured content" and entity summaries live in both the entity table *and*
re-typed in page sections. **Broken dependencies:** any free-text URL; the hardcoded `RESERVED_SLUGS`.
**Manual workflows:** building nav, footer, featured grids, legal links. **Future bottleneck:** linear
human effort per catalog item.

### Phase 2 — Single Source of Truth ledger

| Data | Canonical owner | How Pages/Identity must use it |
|------|-----------------|-------------------------------|
| Program / Path / Lab / Initiative details | their own tables | **data-bound block** queries them; never copies title/price/image |
| Trainer card | `instructor_profiles` (+ optional user) | `trainer_grid` block by tag/manual-pick |
| Partner logos | partners table | `partner_logos` block |
| Testimonials | reviews table | `testimonials` block (filter by rating/program) |
| Live stats (#programs, #learners) | computed | `stats` block reads live counts, not typed numbers |
| **Any URL** (nav/footer/CTA/button) | the **route/page/entity itself** | `TypedLink{type,ref}` resolved at render → no manual URL |
| Page canonical URL | `pages.slug` (+ redirects table) | renamed slug auto-creates a 301; links by ref never break |

### Phase 5 — Broken-experience analysis

- **Enter data twice:** trainer bio (profile) vs re-typed on a "meet the team" page; program price (catalog)
  vs typed in a hero. → Eliminated by data-bound blocks.
- **Broken links:** any nav/footer item pointing at a slug that is later renamed/unpublished. → Eliminated
  by TypedLink resolution + auto-hide of unpublished targets + redirects table.
- **Outdated pages:** "featured programs" hero pointing at a retired program. → Data-bound blocks auto-drop
  unpublished entities; a governance widget flags pages referencing archived entities.
- **Inconsistent branding:** Custom-HTML blocks with inline colors/fonts bypassing the design tokens. →
  Identity exposes design tokens (CSS vars); blocks consume tokens; Custom-HTML is sandboxed + flagged.

### Phase 6 — Marketing ecosystem placement (decision)

**Do NOT cram popups/bars/campaigns into either section.** Create a **third sibling: "Marketing & Engagement
Center."** Rationale: campaigns are *time-boxed, cross-page, audience-targeted, and A/B-tested* — a different
lifecycle from durable pages (content) and durable identity (config).

| Capability | Home |
|-----------|------|
| Announcement bar, popups, campaign banners, countdown, lead magnets, A/B tests, exit-intent | **Marketing Center** (new) |
| Conversion/funnel tracking config (GA4/Meta/TikTok/GTM IDs) | **Site Identity → Tracking** (global config) |
| Per-page SEO/OG, page-level CTA blocks | **Site Pages** (content) |
| Brand tokens, nav, footer, contact, legal-link policy | **Site Identity** (config) |

### Phase 7 — Future expansion (must survive without redesign)

- **Multiple academies / brands:** already isolated — `site_settings` is one row *per tenant* (Spatie). New
  brand = new tenant; zero schema change. For multi-brand *within* one tenant, promote `site_settings` from
  single-row to keyed-by-brand (the `current()` accessor is the only seam to change).
- **Multiple languages:** add `locale` + `translation_key` to `pages`; nav/footer labels become
  `{locale: label}` maps; TypedLink resolves to the locale-correct URL. Blocks are locale-agnostic (they
  reference entities, which carry their own translations).
- **Mobile app / partner portals:** because pages are **structured block trees referencing resolvable
  entities**, the same content graph serves a JSON content-API for mobile and a scoped partner portal with
  no re-authoring. (Custom-HTML is the only non-portable block — another reason to sandbox/limit it.)

---

## D. THE DEFINITIVE V3 ARCHITECTURE

Rename the mental model. The "Site Control Layer" has **three** screens, not two:

```
Site Control Layer
├── 1. Pages & Content Studio   (was: Site Pages)        — the CONTENT layer
├── 2. Brand & Global Settings  (was: Site Identity)     — the CONFIG layer
└── 3. Marketing & Engagement   (NEW, sibling)           — the CAMPAIGN layer
```

### 1) PAGES & CONTENT STUDIO

**Page taxonomy (the single biggest schema change):** add `pages.type`:

| type | URL | Editable structure | In nav? | Notes |
|------|-----|--------------------|---------|-------|
| `system` | fixed (`/programs`…) | SEO + intro/outro slots only | always available as link target | engine renders the body; operator owns chrome + SEO only |
| `standard` | `/{slug}` | full block builder | optional | general content/marketing page |
| `landing` | `/{slug}` or `/lp/{slug}` | full builder, no global nav/footer | no | campaign page, A/B-able, conversion-tuned |
| `legal` | `/{slug}` | rich text + version-lock | footer "legal" menu | privacy/terms; effective-date + version history mandatory |
| `collection` | `/{slug}` | auto-list of an entity set | optional | e.g. "All Labs" = generated, not hand-built |

**Tabs / fields per page:**

- **Content** — block builder. **Two block families:**
  - *Static blocks* (today's set): Hero, Rich Text, Image Banner, Image+Text, Video, FAQ, CTA, Stats, Custom HTML *(sandboxed + flagged)*.
  - *Data-bound blocks (NEW):* `program_grid`, `path_showcase`, `lab_strip`, `initiative_strip`,
    `trainer_grid`, `testimonials`, `partner_logos`, `live_stats`. Each stores only **{ source, filter
    (category/path/tag/featured/manual-ids), sort, limit, layout }** — never entity content.
- **SEO & Social** — SEO title/description (inherits global defaults), canonical, robots (index/noindex),
  Open Graph image/title/desc, auto JSON-LD (`Course`/`Organization` for system/program pages).
- **Publishing** — status `draft → scheduled → published → archived`; `publish_at`/`unpublish_at`;
  **preview token** (share a draft); **version history** (snapshot of blocks on each publish; one-click
  restore); **redirects** (auto-301 on slug change).
- **Settings** — template, locale + translation group, show-in-nav proposal, owner, last-editor (audit).
- **Analytics** — page views, CTA clicks, scroll depth, conversion (read from Marketing/analytics).

### 2) BRAND & GLOBAL SETTINGS

| Tab | Fields | Note |
|-----|--------|------|
| **Brand** | logo (light/dark/favicon/OG-default), primary/secondary/heading/text colors, fonts, base size | exposed as **design tokens / CSS vars** that blocks consume |
| **Menus** | header menu, footer columns (n), mobile menu, **legal menu** — each = ordered list of **TypedLink** | replaces free-text `nav_items`/`footer_links`; pages auto-propose |
| **Footer** | tagline, columns (menus), social links (typed), copyright (auto-year) | |
| **Contact & Org** | name, email, phone, address, map, hours, CR/VAT (for invoices + JSON-LD) | single source for contact shown anywhere |
| **Global SEO** | default title pattern (`%page% — %site%`), default description, default OG image, sitemap policy, robots | per-page overrides inherit from here |
| **Tracking** | GA4 / Meta Pixel / TikTok / GTM IDs, consent mode | no more hardcoded scripts |
| **Localization** | enabled locales, default locale, RTL flag | feeds multi-language |

**TypedLink (the keystone value object):**
```
{ type: route | page | program | path | lab | initiative | category | external | anchor,
  ref:  <route-key | page-id | entity-id | url | #anchor>,
  label: { ar: "...", en: "..." },
  new_tab: bool }
→ resolved to the canonical URL at render time.
→ if target unpublished/deleted: auto-hide on site + warning badge in admin.
```

### 3) MARKETING & ENGAGEMENT CENTER (new sibling)

Campaigns (announcement bar / popup / banner / countdown / lead-magnet), each with: audience rules
(page match, new vs returning, locale), schedule, A/B variants, and a TypedLink CTA. Conversions report
back into page analytics. Lives outside Pages/Identity by design (different lifecycle).

### Relationships · Automations · Permissions · Workflows

- **Automations:** menus auto-offer published pages; data-bound blocks auto-refresh from entities;
  slug-rename auto-creates redirect; unpublishing an entity auto-hides links/cards referencing it;
  copyright year auto-updates; sitemap auto-regenerates.
- **Permissions (RBAC):** `academy_owner/admin` = full; `marketing` = pages + marketing center, NOT brand
  tokens or legal; `finance/instructor/support` = none. Custom-HTML block = owner/admin only. Legal pages =
  owner only. (Aligns with existing `AdminRoleGate`.)
- **Golden workflow:** create page → pick template → drag blocks (static + data-bound) → preview (desktop +
  mobile, draft token) → set SEO (inherits globals) → schedule/publish → page auto-proposes to nav →
  versioned + redirect-safe forever.

---

## E. Migration path (when unfrozen) — additive, no big-bang

1. `pages.type` (default `standard`) + backfill; keep existing pages working.
2. `TypedLink` renderer + a compatibility shim that wraps existing free-text URLs as `{type:external|route}`.
3. Data-bound blocks (read-only consumers of existing entity APIs) — highest ROI, lowest risk.
4. Publishing pipeline (scheduling, versions, redirects).
5. Global SEO + tracking in Identity.
6. Marketing Center as a new module.
7. Localization fields (only when a 2nd locale is actually needed).

Each step ships independently and is reversible. Nothing requires touching the engine pages' rendering.

---

## F. Governance Gate (READ THIS BEFORE BUILDING)

- This blueprint is a **2.0 charter artifact**. It is consistent with [[wamadat-operating-mode]]
  (no new features during Soft Launch unless they serve ops/retention/conversion/reliability/usability/perf)
  and [[beta-prep-complete-2026-06-08]] (redesign sealed behind hard gates).
- **Do not implement now.** The locked next actions are the **real-user LAN test** ([[next-session-lan-test]])
  and the **Content Stream** (operator fills 5 programs / 5 trainers / paths / labs / initiatives). Those
  prove the platform with real users — which is exactly what this blueprint cannot substitute for.
- **Highest-conversion-value slice** (if/when a slice is justified pre-2.0 under the operating-mode test):
  data-bound `program_grid`/`trainer_grid` blocks + TypedLink for nav/footer. They directly serve
  *conversion* and *reliability* (no broken links), and are additive/reversible.
- Recommended trigger to open this: after the LAN test yields real feedback AND content is entered, fold
  the relevant slices into the 2.0 charter's gated scope — not before.
