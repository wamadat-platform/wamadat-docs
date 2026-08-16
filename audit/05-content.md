# ✍️ Content Audit — Reviewer 5 (Content Strategist, Arabic-native)

**Brand reminder:** Wamadat voice — practical, modern, no philosophical flourish. Plain Arabic without tashkeel (owner directive 2026-05-12).

---

## Critical findings (🔴)

### [C-001] Heavy use of tashkeel across user-visible files
- **Severity:** 🔴 Critical (brand-violating)
- **Scope:** Almost every Arabic string written in the most recent sessions uses extensive diacritics — fatha, kasra, damma, tanween — where modern Arabic UI should be plain.
- **Examples (representative, not exhaustive):**
  - `components/marketing/hero.tsx` — multiple
  - `components/marketing/testimonials.tsx` — all 50 entries
  - `components/marketing/partnerships.tsx` — section titles
  - `components/marketing/banner-placeholder.tsx`
  - `app/[locale]/consultations/page.tsx` — package titles + features
  - `components/consultations/consultation-form.tsx` — form labels + helper text
  - `app/[locale]/instructors/page.tsx`
  - `components/team/team-section.tsx`
  - `components/team/join-us-cta.tsx`
  - `app/[locale]/cart/page.tsx` (some lines)
  - `app/[locale]/redeem-gift/page.tsx`
  - `app/[locale]/dashboard/gifts/page.tsx`
- **Action:** A dedicated cleanup pass is required. The sweep is mechanical (regex strip of `[ً-ْ]` — fatha/kasra/damma/tanween/sukun/shadda — with selective preservation of shadda where it disambiguates).
- **Status:** Owner directive captured in memory (`feedback_arabic_plain.md`). Sweep in progress; not 100% complete this turn.

## High findings (🟠)

### [C-002] Mixed conversational vs formal tone in consultations
- **File:** `app/[locale]/consultations/page.tsx` (FAQ)
- **Issue:** "فيه ضَمان استرداد" — colloquial "فيه" mixed with formal context.
- **Fix:** "يوجد ضمان استرداد"

### [C-003] Button labels inconsistent across CTAs
- **Examples:** "احجز الباقة" vs "احجز الآن" vs "ادفع الآن" vs "اشتر الآن"
- **Suggested rule:** Pick ONE pattern per action class and stick with it.
  - Purchase: "اشتر الآن"
  - Book a slot: "احجز الجلسة"
  - Open form: "أرسل طلبك"

### [C-004] "Trusted by leading organizations" eyebrow in English on Arabic-default page
- **File:** Was in `hero.tsx` (now removed in latest pass — confirmed removed).
- **Status:** ✅ Fixed in earlier turn.

### [C-005] Generic empty-state copy
- Multiple "لا توجد بيانات" placeholders. Per UX-012, these should be specific + actionable.

## Medium findings (🟡)

### [C-006] Some headings exceed natural line breaks on mobile
- e.g., hero headline runs awkwardly on small screens. Suggested: shorter alternative in mobile breakpoint or `text-wrap: balance`.

### [C-007] Footer links not all final
- Privacy, Terms exist — but a few footer entries currently link to anchors that don't exist (`#help`, etc. — sample check).

### [C-008] Date formatting inconsistent
- Some places use `toLocaleDateString('ar-SA')` (Hijri context), others use `en-US`. Pick one approach. Recommendation: Gregorian dates with Western numerals everywhere in operational UI (orders, certificates); Hijri only where culturally expected.

### [C-009] Error messages too generic in some auth flows
- Sign-in error "حدث خطأ. حاول مرة أخرى" gives no clue why. Sometimes that's intentional (security), but for non-credential errors (network) it should say so.

## Low findings (🟢)

- [C-010] Some marketing components use 50-character labels where 25 would work harder. Tighten copy.
- [C-011] Tooltip text often missing from icon-only badges.

---

## Recommended copy patterns (style guide essentials)

| Surface | Pattern |
|---|---|
| Page title (H1) | Statement, no question mark, no exclamation |
| CTA button | Verb-first: "ابدأ الآن", "اشترك", "احجز" |
| Empty state | One short sentence + action button — never just "لا توجد بيانات" |
| Error | What happened + what to do next |
| Success | Acknowledge + next step |

---

## Sign-off

**⚠️ Approved with mandatory Arabic-plain sweep.**

The tashkeel cleanup is the single biggest content win. Tone is largely on-brand. Button-label consistency needs a one-time pass to canonicalize.

— Reviewer 5 (Content Strategist)
