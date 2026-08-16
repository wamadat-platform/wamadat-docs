# Wamadat — Three-Horizon North Star

> *Strategic horizons matter only if the metrics are real. Each horizon has a single primary metric. Hit it, advance. Miss it, re-plan.*

---

## H1 — 0 to 90 days: **Closed Beta with first 10 paying users**

### The primary metric
**5 students who complete at least one program AND would recommend Wamadat unprompted to a peer.**

Not 100 sign-ups. Not 1,000 page views. *Five completions + word-of-mouth signal*. Anything else is vanity at this stage.

### The single bet
The flagship tenant (`wamadat`, your own academy) is the proof of concept. If your own 5 programs cannot generate 10 paying-and-completing students by 2026-09-08, the thesis needs revision.

### What we measure weekly
- Verified users (cap: 50)
- Paying enrolments (target: 1+ per week from week 2)
- Program completions (target: 5 by day 90)
- Net Promoter signal: from Tally form question "would you recommend Wamadat to a colleague?" (1-5)
- Critical incidents (target: 0 P0, ≤2 P1)

### What we do NOT do in H1
- No second tenant onboarding.
- No mobile app.
- No new product features outside ops/conversion/reliability.
- No marketing site overhaul.
- No SEO investment.
- No paid acquisition.

---

## H2 — 90 days to 1 year: **Multi-tenant validation with 5 academies**

### The primary metric
**3 academies (besides the flagship) running ≥1 program with ≥10 paying students each, on Wamadat for ≥60 days, who renew their subscription unprompted.**

Not "100 trial signups". *Three serious academies + retention*. Retention is the only signal that the rail-not-marketplace thesis works.

### The single bet
The thesis is: serious Saudi/Gulf instructors will pay a meaningful platform fee to run their academy on Wamadat instead of: (a) building their own site, (b) using Teachable/Thinkific, (c) using Coursera-for-Business. If that bet loses, the entire infrastructure thesis is wrong.

### What we measure monthly
- Active tenants (paying)
- Per-tenant programs published
- Per-tenant active students
- Per-tenant 90-day retention
- Wamadat take-rate revenue (transparent published fee × volume)
- Operator support load (hours/week per tenant)

### What we invest in during H2 (in priority order)
1. **Tenant onboarding command** (`php artisan tenant:onboard {slug} {owner-email}`) — repeatable, idempotent, 30-second tenant creation.
2. **Subdomain DNS + wildcard cert provisioning** — `<academy>.wamadat.academy` resolves cleanly.
3. **Cross-tenant cache fix verification** at runtime (done — C-001 closed today).
4. **Public trust page maturation** — case studies of the 3 academies.
5. **Selective accessibility upgrades** to WCAG AAA on flagship flows.

### What we do NOT do in H2
- No native mobile app (PWA is enough at this scale).
- No "AI tutor" feature theater. AI assistant exists, ship UX polish, do not chase parity with ChatGPT.
- No expansion outside KSA + GCC.
- No B2B enterprise sales pitch. Self-serve only.

---

## H3 — 1 to 3 years: **Regional rail for Arabic-language learning**

### The primary metric
**100+ academies on Wamadat, of which 20 generate >50,000 SAR/year in transaction volume each.**

This is when "infrastructure rail" stops being a thesis and becomes a market position. Below this scale, we are a tool. Above it, we are a platform with its own gravity.

### The single bet
The bet at H3: Arabic-language learning is large enough, and underserved enough, that a Saudi-headquartered platform with Saudi-grade compliance can become the *default* place where serious Arabic education happens online — without Anglo-translation, without foreign payment rails, without American data residency.

### What we measure quarterly
- Total platform GMV in Arabic-language education
- % of paying students from KSA / GCC / wider MENA / outside MENA
- Academy NPS
- Student verification requests against `/verify/{code}` (a proxy for cert credibility)
- Time-to-first-student for new academies onboarding
- Wamadat margin as a function of GMV (we should be profitable on infrastructure margins, not subsidised by VC)

### What becomes possible in H3 (but only after H2 proves out)
- B2B enterprise tier (compliance dashboards, SSO, dedicated tenant DB option) — see audit Track 12.
- Native mobile apps (when student count justifies the support burden).
- API for third-party integrations (HR systems, government training programs).
- A second-region deployment (probably UAE) for data residency / DR.
- A formal accreditation partnership with at least one Saudi or Gulf educational authority.

### What we do NOT do in H3
- Do not become a marketplace, even when pressured. The principle holds.
- Do not internationalise into non-Arabic languages. The principle holds.
- Do not raise venture capital that would require us to chase scale over discipline. If we raise, it is a small operator-aligned round.

---

## The exit conditions for each horizon

- **H1 fails to hit primary metric by 2026-09-08** → revise thesis (maybe Wamadat is actually a single-tenant tool for the flagship operator, not infrastructure-for-many).
- **H2 fails to hit primary metric by 2027-06-08** → revise thesis (maybe Arabic-language entrepreneur market is smaller than we modeled).
- **H3 fails to hit primary metric by 2029-06-08** → revise the company (maybe the format has changed, AI-native learning has eaten LMS, time to pivot).

Each horizon has a clear failure condition. A strategy without exit conditions is a religion.

---

*Strategy lives in two columns: what you will do, and what you will refuse to do even when it gets hard. The second column is twice as important.*

— *board, 2026-06-08*
