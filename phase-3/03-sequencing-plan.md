# Phase 3 — Sequencing Plan (week-by-week)

**Status:** Analysis only. This is the recommended execution order with rationale per slot.

## How to read this document

- Each "stage" is a coherent shippable milestone. Operator should see progress at each stage boundary.
- Durations are realistic (include integration + browser-proof per subsystem), not "happy path" estimates.
- The plan is **serial-by-default** with explicit parallel-tracks called out.
- Phase 3 total: **5-7 months focused work** (depends on parallelism + scope discipline).

---

## Stage 0 — Decision & Setup (week 0)

**Goal:** Get architectural decisions on paper BEFORE coding starts.

| Item | Output |
|---|---|
| Operator approves the sequencing plan | Sign-off on this document |
| Operator approves the architecture rules (`04-architecture-rules.md`) | Locked design constraints |
| Operator approves the rebuild-vs-refactor matrix (`05-rebuild-vs-refactor.md`) | List of what's safe to extend vs needs replacement |
| Phase 3 branch strategy | All work on `main` (continuing current habit) OR `phase-3` long-lived branch (cleaner rollback) |
| Beta launch decision | Pause Beta until Phase 3.1 ships OR open small Beta on current Filament + iterate |

**Deliverable:** Locked plan document + operator sign-off.

---

## Stage 1 — Foundations (weeks 1-6) — BLOCKING

Nothing else can ship until these land. No parallelism here; foundations must be sequential.

| Week | Subsystem | Justification |
|---|---|---|
| 1 | A1 Admin UX Library (start) | Every later subsystem uses these primitives. Build first, refine while consuming. |
| 1-2 | A2 Safety Rails | Wraps every destructive action. Wire into existing refund + role change + delete actions before any new destructive feature ships. |
| 2-4 | A3 Block System Core | Largest foundation. Block contract + 5 sample block types (Hero, Text, Image, FAQ, CTA) shipped before D1/D2 start. |
| 3-5 | A4 Forms Builder | Generic form model + builder UI + public submission endpoint + Excel export. Used by D1, D2, E2. |
| 4-6 | A5 Communication Center extension | Existing email infra stays. Add: SMS adapter shell (Unifonic-ready), WhatsApp adapter shell, template editor with variable picker, delivery dashboard. |

**Stage 1 deliverable:** A polished foundation layer that the operator can see (UX library demo page, sample form built and submitted, sample block-page composed, sample template edited). Browser-proof: build a Form → embed in a placeholder page → submit → see entry → trigger email → see delivery log. This is the operator's first "the platform is becoming a real product" moment.

---

## Stage 2 — Content & Catalog (weeks 7-14) — operator's daily work

Highest-frequency operator activity. Ship this before commerce/marketing so the operator can actually USE the platform daily during Phase 3 build.

| Week | Subsystem | Notes |
|---|---|---|
| 7-9 | B1 Program Builder | Wizard with Draft → Preview → Publish. Approval gate (instructor proposes → owner publishes). |
| 10-13 | B2 Content Builder | The big one. Drag-drop modules + lessons, 15 lesson types, modal editing. |
| 13-14 | B3 Reviews extension | Video reviews + moderation queue + post-completion auto-request. Light parallel work to B2 finale. |

**Stage 2 deliverable:** Operator creates a brand-new program end-to-end from the admin: 4 modules, 20 lessons across video / PDF / quiz / Zoom / embed types, all in <2 hours (vs today's "all day"). Browser-proof captures this flow.

---

## Stage 3 — Commerce (weeks 15-22) — revenue path

Now the operator can author. Next: monetize and recover lost revenue.

| Week | Subsystem | Track |
|---|---|---|
| 15-17 | C1 Coupons UX + scope expansion | Serial |
| 16-19 | C2 Payment Integrations (Moyasar + HyperPay) | **Parallel** to C1 if 2nd dev exists |
| 18-21 | C3 Orders + Abandoned Cart UX | Serial — depends on C1 |
| 21-22 | C4 Refund Workflow Hardening | Serial — depends on A2 |
| 22 | C5 Financial Operations dashboard (start; finishes Stage 5) | Aggregates C1-C4 data |

**Stage 3 deliverable:** Operator runs a "10% off May" coupon campaign end-to-end: creates coupon, scopes to 3 programs, auto-applies, runs 1 week, sees redemption dashboard, identifies the abandoned-cart funnel and recovers 2 lost orders manually + watches 1 recovery email convert.

---

## Stage 4 — Marketing & Public Surface (weeks 23-34) — acquisition

The 4 marketing subsystems together let the academy be marketed without a developer.

| Week | Subsystem | Notes |
|---|---|---|
| 23-29 | D1 Landing Page Builder | XL — biggest single subsystem. 20 block types, drag-drop, live mobile preview, version history. |
| 27-32 | D2 Website CMS | **Parallel** to D1's tail — reuses D1's block library. |
| 30-31 | D3 Partners + Sponsors | Small, parallel to D2 |
| 32-34 | D4 Marketing + Tracking | Last — depends on D1/D2 pages to attach pixels to. |

**Stage 4 deliverable:** Operator publishes a campaign landing page from admin, adds Meta Pixel ID in settings, runs a Meta ad → click → land on page → register form → cart → checkout. Conversion event reaches Meta Events Manager. Marketing dashboard shows pixel-attributed revenue.

---

## Stage 5 — Live Operations + Final Aggregation (weeks 35-42)

The remaining pieces.

| Week | Subsystem | Notes |
|---|---|---|
| 35-39 | E1 Zoom integration | OAuth server-to-server + webhook + meeting auto-create + reminder pipeline. |
| 38-42 | E2 Consultations | **Parallel** to E1 tail — uses E1 rooms. |
| 41-43 | F1 Operations Dashboard | The owner's daily home page. Aggregates everything. |
| 41-43 | C5 Financial Operations (finish) | **Parallel** to F1. |

**Stage 5 deliverable:** Operator schedules a live session for an existing program → invited students receive 24h + 1h reminders → attendance auto-captured from Zoom webhook → recording link auto-attached to the lesson. Consultant publishes profile + 5 weekly slots → student books → pays → joins Zoom → leaves review. Operator's dashboard shows all of this on the day-after.

---

## Stage 6 — Browser Proof + Phase 3 Closeout (weeks 43-46)

| Week | Activity |
|---|---|
| 43-45 | G1 End-to-end Playwright suite covering the full operator flow + visitor flow + edge cases (failed payment, abandoned cart, refund, video upload, etc.) |
| 45-46 | Tune any regressions surfaced; final operator browser walkthrough |
| 46 | Phase 3 closeout doc + memory updates + retrospective |

**Phase 3 closeout gate:** every item on the operator's "Proof Required" list (19 items in the original spec) has a Playwright test that passes + a screen recording attached to the closeout doc.

---

## Total calendar — realistic estimate

| Stage | Weeks | Cumulative |
|---|---:|---:|
| 0 Decision & Setup | 1 | 1 |
| 1 Foundations | 6 | 7 |
| 2 Content & Catalog | 8 | 15 |
| 3 Commerce | 8 | 23 |
| 4 Marketing & Public Surface | 12 | 35 |
| 5 Live Ops + Aggregation | 8 | 43 |
| 6 Proof & Closeout | 4 | 47 |
| **TOTAL serial** | **47** | **~11 months** |

With 1 developer (or Claude alone), this is **~11 months**.

With **disciplined parallelism** on Stage 3 commerce track and Stage 4 marketing track (when 2 work streams can run independently), this compresses to **~8 months**.

With **scope discipline** (defer D4 marketing pixels to Phase 4, defer E2 Consultations to Phase 4, defer SMS/WhatsApp in A5 to Phase 4), this compresses to **~5-6 months**.

---

## Recommended scope-discipline cuts for a 5-6 month Phase 3

If the operator wants Phase 3 in ~6 months, defer these to Phase 4:

| Cut | Saved | Why safe to defer |
|---|---:|---|
| SMS + WhatsApp channels in A5 | 2-3w | Beta cohort is tiny; email + in-app is enough for first 50 users. |
| D4 Marketing pixels (defer to Phase 4) | 5-7w | Pixels need traffic to be useful. Beta has invitation-only traffic. |
| E2 Consultations (defer to Phase 4) | 5-7w | Today consultations are a Form (rough but functional). Defer rebuild until program side is mature. |
| Animated transitions / advanced block types (Countdown, Map) in D1 | 1-2w | 15 blocks instead of 20 ships first. |

**Trimmed Phase 3 = 5-6 months, ships ~16 of 21 subsystems.** Remaining 5 become Phase 4.

---

## What to do if a stage runs late

Rule: **never ship a half-built foundation.** If Stage 1 takes 8 weeks instead of 6, push everything else 2 weeks. Foundations being right is what saves rebuilds later. Don't ship D1 on top of an unstable A3 — that's the bug factory.

Rule: **content layer (Stage 2) is the must-have.** If anything must be cut, cut from marketing or live-ops, never from content. The operator must be able to author programs.

Rule: **commerce stays serial.** Don't parallelize within commerce (C1 → C2 → C3 → C4 must go in order because they layer financial guarantees on each other).

---

See `04-architecture-rules.md` for the design constraints these subsystems must obey + `05-rebuild-vs-refactor.md` for what's keep / extend / rebuild.
