# Phase 3 — Executive Summary + Decisions for the Operator

**Read this first.** Other files are deep references. This is the 1-page version.

## What you asked for

A full operational rebuild of the admin: 19 subsystems covering Program Builder, Content Builder, Landing Pages, Website CMS, Forms, Commerce, Marketing, Live Ops, Communication, and the Admin Dashboard. Arabic-first, RTL-clean, block-based, no Filament CRUD mentality, scalable without rebuild.

## The honest reality

| Question | Answer |
|---|---|
| Total realistic scope (serial) | ~11 months |
| With disciplined parallelism (2 tracks at once) | ~8 months |
| With scope discipline (defer 5 subsystems to Phase 4) | ~5-6 months |
| Beta launch during Phase 3? | Possible from week ~15 (after Content layer) if you accept rough commerce + marketing UX in the interim |
| Largest single piece | Landing Page Builder (~7 weeks) |
| Highest-risk piece if rushed | Block System Core — everything else depends on it |
| Biggest existing strength | Phase D commerce + audit log + role system. ~9 domains we KEEP unchanged. |
| Biggest existing gap | CMS folder is empty. Marketing module is empty. No Forms builder. No Consultation module. |

## The recommendation

**Do Phase 3 in 6 stages, ~5-6 months trimmed scope:**

1. **Stage 0 (1 week)** — you sign off on this plan; we lock architecture rules.
2. **Stage 1 (6 weeks)** — Foundations: admin UX library, safety rails, block system, forms builder, communication center.
3. **Stage 2 (8 weeks)** — Program Builder + Content Builder + Reviews extension. After this, **you can author programs end-to-end in <2 hours instead of all day**. This is the high-leverage moment.
4. **Stage 3 (8 weeks)** — Commerce: coupons + Moyasar/HyperPay + Orders/Abandoned + Refund + Financial dashboard.
5. **Stage 4 (8 weeks)** — Landing Pages + Website CMS + Partners. (Trim D4 Marketing pixels to Phase 4.)
6. **Stage 5 (6 weeks)** — Zoom integration + Operations dashboard + browser-proof closeout. (Trim Consultations + SMS/WhatsApp to Phase 4.)

Trimmed Phase 3 = **16 of 21 subsystems**, ~5-6 months. Deferred to Phase 4: Marketing pixels, Consultations rebuild, SMS, WhatsApp, optional landing-page blocks (Countdown / Map).

## What's in this report

| File | Purpose |
|---|---|
| `00-ux-audit.md` | What's in the admin today, pain points, what works | 
| `01-subsystem-matrix.md` | Per-subsystem profile (goal, user, complexity, blockers, risks, DB impact, API impact, admin UX impact, mobile-later) |
| `02-dependency-graph.md` | Which subsystem depends on which; categorization into 7 groups |
| `03-sequencing-plan.md` | Week-by-week execution stages with rationale per stage |
| `04-architecture-rules.md` | Reusable / generic / coupled / config-driven / block-based rules every subsystem must obey |
| `05-rebuild-vs-refactor.md` | Per-domain: keep / extend / refactor / rebuild / new-build action |
| `06-summary-and-decisions.md` | This file |

## What you need to decide before any code ships

Five blocking decisions:

### Decision 1 — Scope
- ☐ Full Phase 3 (21 subsystems, ~8-11 months)
- ☐ Trimmed Phase 3 (16 subsystems, ~5-6 months) — **recommended**
- ☐ Custom (you tell me which 5 to drop and we re-cost)

### Decision 2 — Beta during Phase 3
- ☐ Pause Closed Beta entirely until Phase 3 ships
- ☐ Open Closed Beta after Stage 2 (Content layer ready, week ~15) on the existing Filament for commerce — **recommended** because operator + 3-5 invited users will surface UX bugs Phase 3 hasn't seen yet
- ☐ Open Closed Beta NOW on current Filament — Phase 3 runs alongside, user feedback shapes priorities

### Decision 3 — Approval workflow on programs
- ☐ Instructor proposes → owner publishes (matches code comments + adds safety)
- ☐ Instructor publishes directly (matches today's behavior, faster, less safe)

### Decision 4 — Block storage shape (A3 architecture)
- ☐ Polymorphic table (each block is a row) — flexible, slower reads, easier migrations
- ☐ Versioned JSONB document per page — fast reads, stricter migrations
- (I recommend polymorphic — Wamadat is read-heavy but cache layer can absorb. Operator can defer this to the dev choosing Stage 1 implementation.)

### Decision 5 — Branch strategy
- ☐ Continue on `main` (every commit pushed, same habit as Phase D) — simpler, less rollback safety
- ☐ Long-lived `phase-3` branch merging to `main` at each Stage boundary — cleaner rollback, more git complexity

## Cost of "no decision"

If you don't pick a scope, I'll default to the trimmed Phase 3 (Decision 1 = trimmed). The other decisions can be made later — they don't block Stage 0/1 from starting.

But I will **not** start coding any subsystem without:
1. Your explicit "go" on the overall scope.
2. Your sign-off on Decision 3 (approval workflow) — this changes B1's data model.
3. Your sign-off on Decision 5 (branch strategy) — affects every commit.

Decisions 2, 4 can land later.

## What I won't do without asking

- Drop any current admin feature (the 16 resources stay until their rebuild ships).
- Modify Phase D-frozen API surface (`/api/v1/*`) without a new versioned endpoint.
- Touch the 25-test contract safety net unless adding tests.
- Run any destructive DB op (DROP, TRUNCATE, ALTER COLUMN DROP NOT NULL on populated columns) without your explicit per-op approval.
- Open Closed Beta to external users without your "go".
- Add a new payment-gateway integration that requires shared API keys without you handing over the keys through a safe channel (not chat).

## What happens after you decide

Stage 0 closes. I produce a short "Stage 1 implementation plan" — a ~3-page doc covering: exact files to create, exact migrations, exact UX library components, the Browser Proof checklist for Stage 1's exit gate. You sign off again. Stage 1 begins.

This is the only "big plan" I'll ask you to digest. Stages 2-6 produce smaller implementation plans (1-2 pages each) at their start — much easier reads.

## Two questions you might be wondering

**"Can we just do Program Builder first to feel progress?"**
Yes, but only after the 6 weeks of Foundations land. Program Builder built on top of unstable foundations will be rebuilt later. Foundations first is the only path that doesn't waste 4-6 weeks of rework.

**"Can you give me a Gantt chart?"**
The `03-sequencing-plan.md` is the Gantt in text form. If you want a visual chart, I'll generate one in Stage 0 — but the text version is faster to update as decisions land.

---

**Next step on you:** read the files above (start with `00-ux-audit.md` and `06-summary-and-decisions.md` — those two give you 80% of the picture). Then answer the 5 decisions. After that, I produce the Stage 1 implementation plan.
