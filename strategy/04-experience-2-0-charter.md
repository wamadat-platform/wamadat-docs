# Wamadat Experience & Governance 2.0 — Charter

> **Status**: CLOSED — opens only when gate conditions are met.
> **Created**: 2026-06-08 — same day Beta preparation closed.
> **Operator decision**: deferred until first 30 days of real Beta data exist.

---

## What this file is

This file holds the strategic frame for the next major redesign of Wamadat — covering header, navigation, mobile experience, role-based UX, trust badge architecture, command-palette interaction, and accessibility upgrades to WCAG AAA on flagship flows.

It is **explicitly deferred**. The operator and the executive transformation office agreed (2026-06-08) that:

> "Header improvements, trust badge, mobile search optimisation, and role-based navigation will not ship before first real user data. We will not redesign in a vacuum."

---

## Why this file exists (and is not in `docs/executive-transformation-backlog.md`)

The standing backlog is where backlog items go. This is different: it is the **charter** for a future strategic initiative. When the charter opens, an entirely new sprint plan will be built from it — with real heatmaps, real Sentry-captured failure patterns, real user-interview quotes, real abandonment funnels.

Putting these items in the standing backlog would risk premature execution. Keeping them here, sealed, gated, ensures the initiative is opened with discipline.

---

## Gate conditions (ALL must be true to open this file)

| # | Condition | How to verify |
|---|---|---|
| 1 | First 5 paying students completed at least one program | Filament `/admin` → `users` → filter `is_active=true AND email_verified_at IS NOT NULL` AND enrollments with `completed_at IS NOT NULL` ≥ 5 |
| 2 | At least 30 days elapsed since first Beta invitation | calendar |
| 3 | Sentry has ≥ 50 frontend events captured (to identify real friction) | `https://sentry.io/issues/` filter `platform:javascript` count |
| 4 | At least 3 user interviews completed and notes filed | `docs/research/interviews/` exists with ≥ 3 dated entries |
| 5 | Tally feedback form has ≥ 20 submissions | Tally dashboard count |
| 6 | Backlog from real users has been compiled (≥ 10 documented UX pains) | `BACKLOG.md` section "Beta UX gaps" with ≥ 10 items |

If any condition is unmet on the date this file is next reviewed, **do not open this charter**. Extend the Beta. Gather more data.

---

## What this initiative will cover (when it opens)

### Track A — Header & Navigation 2.0
- Logo height optimisation (`h-9 lg:h-10` from current `h-12 lg:h-14`)
- Header height optimisation (`h-14 lg:h-16` from current `h-16 lg:h-20`)
- Search behaviour: command-palette (`Cmd+K`) replacing inline input
- Trust badge: visible icon in header linking to `/trust`
- Role-aware navigation (guest / student / instructor / enterprise variants)
- NotificationBell visibility gated on `isAuthenticated`
- Multi-role UserMenu consolidating trainer/admin/dashboard buttons
- Mobile search: icon + modal pattern

### Track B — First Impression
- Above-the-fold redesign of `/` (homepage)
- Hero section with social proof + first program tile + clear value prop
- Premium typography pass (Tajawal kerning, mixed-script handling)
- Reduce visible navigation density to 5 items max

### Track C — Trust Surface Maturity
- Status page integration (Better Stack — depends on audit Track 4.4)
- Live "verified by Wamadat" stamp on certificate render path
- ZATCA + PDPL trust marks visible in footer (and header for enterprise role)
- Public audit log summary card on `/trust` page

### Track D — Mobile-First Pass
- Comprehensive mobile audit on top 10 routes
- Tap-target sizing review
- RTL keyboard-input edge cases (Saudi phone formats, ID formats)
- PWA installability polish

### Track E — Accessibility AAA Push
- `@axe-core/playwright` integrated into CI
- WCAG AAA on flagship 4 flows: home / sign-up / catalog / checkout
- Screen-reader pass on `/learn/<slug>` lesson player
- Keyboard navigation for all interactive elements

### Track F — Information Architecture Audit
- Cross-tenant pattern analysis (when 3+ tenants exist)
- Site Settings vs Page Builder convention finalisation
- Footer link standardisation
- Sitemap.xml generation

### Track G — Visual Identity 2.0
- Brand evolution (logo refresh if data suggests need)
- Secondary palette expansion
- Iconography system
- Illustration direction

---

## Out of scope for this charter (always)

These remain refusals regardless of data:

- Becoming a course marketplace
- Generic LMS feature parity with Moodle/Talent
- Native mobile apps before H2 metric is hit (per `03-north-star.md`)
- AI tutor feature theater chasing ChatGPT
- Expansion outside Arabic-language learning
- Internationalisation beyond ar/en before H3

If any of these arise as requests during the 2.0 initiative, refer back to `01-manifesto.md` and `02-principles.md`. Each is a hard refusal.

---

## What happens when this file opens

1. The executive transformation office (or equivalent successor) reviews this charter against the data gathered in the gate conditions.
2. A working session produces a sprint plan with concrete deliverables, sized in person-days.
3. The sprint plan is committed to `docs/strategy/05-experience-2-0-sprint-plan.md`.
4. Engineering freeze is partially lifted for the scoped work.
5. After the sprint, this charter is marked CLOSED again (or REVISED if learnings reframe it).

Until then, this file stays sealed.

---

*— Wamadat board, 2026-06-08*
*Next review: no earlier than 2026-07-08 (30 days post first invitation)*
