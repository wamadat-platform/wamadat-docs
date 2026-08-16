# Phase 3 — Dependency Graph + Categorization

**Status:** Analysis only. This is the "what must come before what, and why" of Phase 3.

## 1. The 7 categories the operator asked for

| # | Category | Subsystems |
|---|---|---|
| 1 | **Core operational systems** | A1 Admin UX Library · A2 Safety Rails · A5 Communication Center · F1 Operations Dashboard |
| 2 | **Content systems** | B1 Program Builder · B2 Content Builder · B3 Reviews + Testimonials |
| 3 | **Commerce systems** | C1 Coupons · C2 Payment Integrations · C3 Orders + Abandoned · C4 Refund Workflow · C5 Financial Operations |
| 4 | **Marketing systems** | D1 Landing Page Builder · D3 Partners + Sponsors · D4 Marketing + Tracking |
| 5 | **CMS systems** | A3 Block System Core · A4 Forms Builder · D2 Website CMS |
| 6 | **Analytics systems** | (overlaps with C5 financial + D4 marketing — no standalone analytics layer until Phase 4) |
| 7 | **Live operations systems** | E1 Zoom + Live Sessions · E2 Consultations |

**Note on Analytics:** The operator's spec category "Analytics systems" gets satisfied by reports inside C5 (financial dashboard) + D4 (conversion attribution dashboard) + F1 (ops dashboard). Building a standalone analytics warehouse (e.g., events → BigQuery → dbt) is a Phase 4 scope, not Phase 3 — it requires production traffic before it earns its cost.

---

## 2. Dependency diagram (ASCII)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          FOUNDATIONS LAYER                              │
│                                                                         │
│   A1 Admin UX Library ──────┬─────────────┬───────────────┐             │
│        (wizards, RTL,       │             │               │             │
│         danger modals,      │             │               │             │
│         drag-drop)          │             │               │             │
│                             ▼             ▼               ▼             │
│   A2 Safety Rails ──→ everything destructive (refunds, role chg, ban)   │
│                                                                         │
│   A3 Block System Core ──────┬────────────┐                             │
│        (pages, blocks,       │            │                             │
│         versioning)          │            │                             │
│                              ▼            ▼                             │
│                          D1 Landing   D2 Website                        │
│                                                                         │
│   A4 Forms Builder ──────┬───────────┬────────────┐                     │
│        (generic forms,   │           │            │                     │
│         submissions)     │           │            │                     │
│                          ▼           ▼            ▼                     │
│                       D1 Landing   D2 Website   E2 Consult              │
│                       (Reg block)  (Contact)    (Booking)               │
│                                                                         │
│   A5 Communication ──────┬───────────┬────────────┬──────────┐          │
│        (email/in-app/    │           │            │          │          │
│         SMS/WhatsApp)    │           │            │          │          │
│                          ▼           ▼            ▼          ▼          │
│                      C3 Abandoned  C4 Refund   E1 Zoom   E2 Consult     │
│                      (recovery)    (notify)    (remind)  (remind)       │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          CONTENT LAYER                                  │
│                                                                         │
│   B1 Program Builder ──────┬─────────────┐                              │
│        (wizard, draft/     │             │                              │
│         preview/publish)   │             │                              │
│                            ▼             ▼                              │
│                       B2 Content    D1 Landing                          │
│                       (depends on   (lists                              │
│                        program ID)   programs)                          │
│                                                                         │
│   B3 Reviews ─────────────────────────────┐                             │
│        (existing model +                  │                             │
│         video + queue)                    ▼                             │
│                                       D1 Landing                        │
│                                       (testimonials                     │
│                                        block)                           │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          COMMERCE LAYER                                 │
│                                                                         │
│   C2 Payment Integrations ──→ C3 Orders ──→ C5 Financial Ops            │
│        (Moyasar, HyperPay     (UX over     (reports on                  │
│         + existing Tap,        existing     existing                    │
│         Tamara, Mock)          service)     payments)                   │
│                                                                         │
│   C1 Coupons ────────────────→ C3 Orders                                │
│        (scope picker + UX)     (apply at checkout)                      │
│                                                                         │
│   C4 Refund ─────────────────→ uses A2 Safety + A5 Communication        │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     PUBLIC SURFACE LAYER                                │
│                                                                         │
│   D1 Landing Page Builder ────┬──────────┬─────────────┐                │
│        (uses A3, A4, B3)      │          │             │                │
│                               ▼          ▼             ▼                │
│                           D3 Partners  D4 Marketing  C3 Orders          │
│                                        (Pixels on    (Reg→Cart→         │
│                                         every page    Checkout)         │
│                                                                         │
│   D2 Website CMS ─────────── reuses A3 blocks; deps similar to D1       │
│                                                                         │
│   D4 Marketing + Tracking ────→ fires on Forms (A4), Orders (C3),       │
│                                  PageView (D1/D2), Consultation (E2)   │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       LIVE OPS LAYER                                    │
│                                                                         │
│   E1 Zoom ──────→ E2 Consultations (uses Zoom rooms)                    │
│        (uses A5,         (uses A4, A5, E1, C2)                          │
│         server-to-                                                      │
│         server OAuth)                                                   │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       AGGREGATION LAYER                                 │
│                                                                         │
│   F1 Operations Dashboard ── reads from every subsystem above           │
│   C5 Financial Operations ── reads C1, C2, C3, C4                       │
│   G1 Browser Proof Suite ── exercises end-to-end across all             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Direct dependency map (operator's requested form)

The operator asked for an example like: `Program Builder → Content Builder → Landing Pages → Orders → Coupons → Analytics`. Below is the actual map for Phase 3:

```
A1 Admin UX Library ─┬─→ B1 Program Builder ─┬─→ B2 Content Builder ─┐
                     │                       │                       │
                     ├─→ B3 Reviews          │                       │
                     │                       │                       │
A3 Block System Core ┴─→ A4 Forms Builder ───┼─→ D1 Landing Pages ───┤
                                             │                       │
A5 Communication Center ─────────────────────┼─→ C3 Orders ──────────┤
                                             │       ↑               │
                                             │   C2 Payments         │
                                             │       ↑               │
                                             │   C1 Coupons          │
                                             │       ↑               │
A2 Safety Rails ─────────────────────────────┴─→ C4 Refund          │
                                                                    │
D4 Marketing+Tracking ──────────────────────────────────────────────┤
                                                                    │
E1 Zoom ─────→ E2 Consultations ────────────────────────────────────┤
                                                                    │
D2 Website CMS ─────────────────────────────────────────────────────┤
                                                                    │
D3 Partners ────────────────────────────────────────────────────────┤
                                                                    │
                                                                    ▼
                                              F1 Operations Dashboard
                                              C5 Financial Operations
                                              G1 Browser Proof Suite
```

**Key insight:** the dependency tree has 4 layers (foundations → content/commerce → public surface → aggregation). Skipping foundations or commerce to jump to landing pages would force rework — landing pages need block primitives, form primitives, communication primitives, and program data.

---

## 4. What MUST come first (and why)

Sorted by "if delayed, forces rebuild of everything downstream":

| Rank | Subsystem | If delayed | Cost of delay |
|---|---|---|---|
| 1 | **A3 Block System Core** | Hardcoded blocks in D1 and D2 | Rewrite both builders later |
| 2 | **A1 Admin UX Library** | Each subsystem invents its own wizard/modal patterns | Long-term inconsistency + 2x dev effort across 19 subsystems |
| 3 | **A4 Forms Builder** | Hardcoded forms in registration, contact, consultation booking | 6+ feature-specific form implementations to delete and re-build |
| 4 | **A5 Communication Center** | Each subsystem writes its own email template plumbing | N parallel template stores; nightmare to migrate |
| 5 | **A2 Safety Rails** | Destructive ops added without audit / sep-of-duty | Retrofit audit on existing destructive actions across the codebase |
| 6 | **B1 Program Builder** | B2/D1/C3 all reference program shape | Schema drift if program changes after they ship |

These six are the foundations layer. Phase 3 should NOT ship D1 (Landing Pages) before A3 + A4 are stable, or any consumer-facing surface before A2 + A5.

---

## 5. What can parallelize once foundations land

After A1-A5 + B1 are in place (~week 14-16), several tracks become independent:

- **Commerce track**: C1 → C2 → C3 → C4 → C5 (mostly serial within the track, but independent of marketing track)
- **Marketing track**: D1 → D2 → D3 → D4 (serial within track, independent of commerce)
- **Live-ops track**: E1 → E2 (serial)
- **Content polish**: B3 (Reviews extensions) can run anytime after A1

Two developers (or one dev + Claude) can run Commerce + Marketing in parallel from week 14 onward.

---

## 6. The "if I skip this, what breaks" matrix

| Subsystem skipped | What breaks |
|---|---|
| A1 Admin UX | Every subsystem reinvents primitives. Inconsistent UX. 30% longer total dev. |
| A2 Safety | Operator accidentally refunds full amount; one role unilaterally bans a user. Real money/legal risk. |
| A3 Blocks | Landing + CMS hardcode templates. Operator can't add a new block type without dev. |
| A4 Forms | Every page-with-a-form is bespoke code. Adding a new field on registration form = code change. |
| A5 Communication | Cannot recover abandoned carts. Cannot remind students about live sessions. Lose 20%+ revenue from missed comms. |
| B1 Program | Operator stuck with current 5-tab CRUD. Daily friction. Won't onboard new programs at scale. |
| B2 Content | Cannot add a video lesson with attachments without 4 form saves. Limits content velocity. |
| B3 Reviews | Cannot moderate at scale; video testimonials never ship. |
| C1 Coupons | Cannot run marketing campaigns. Lose acquisition lever. |
| C2 Payments | Stuck on Tap/Tamara only. Cannot offer Moyasar (cheaper) or HyperPay (corporate). |
| C3 Orders | Lose revenue to abandoned carts; cannot help customers complete payment. |
| C4 Refund | Today's refund is "amount default = full, one click." High-risk financial action. |
| C5 Financial | Operator runs blind on revenue trends. Cannot identify which programs make money. |
| D1 Landing | Marketing depends on developer for every program page. Acquisition velocity capped. |
| D2 CMS | Marketing depends on developer for every site copy change. |
| D3 Partners | Cannot add sponsor logos without code. Low-stakes. |
| D4 Marketing+Tracking | Cannot measure paid-ad ROI. Cannot retarget. Cannot optimize. |
| E1 Zoom | Live sessions are "paste a URL" — no calendar sync, no attendance, no reminders. |
| E2 Consultations | Consultations live as a Form (current state). Cannot scale consultant business. |
| F1 Ops Dashboard | Operator opens admin and sees 8 widgets, scattered. No "what needs my attention" signal. |
| G1 Browser Proof | Phase 3 ships with bugs nobody caught — visible to first 5 Beta users. |

---

See `03-sequencing-plan.md` for the calendar-week execution sequence.
