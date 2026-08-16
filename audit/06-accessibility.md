# ♿ Accessibility Audit — Reviewer 6 (WCAG 2.1 AA Specialist)

**Method:** Manual review of components against WCAG 2.1 AA criteria. Sample-based, focused on high-traffic surfaces.

---

## High findings (🟠)

### [A11Y-001] Notification bell touch target under 44×44
- **File:** `components/layouts/notification-bell.tsx:77-89`
- **Issue:** `p-2` (8px) wraps a 20px icon — total ~36px square. WCAG 2.5.5 (target size) requires ≥ 44px.
- **Suggested:** `p-2.5` or `p-3` to reach 44px.

### [A11Y-002] Modal dialog (gift program) missing focus trap + ESC
- **File:** `components/programs/gift-program-button.tsx`
- **Issue:** Custom modal — no `role="dialog"`, no `aria-modal`, no focus return on close, no ESC to dismiss.
- **Required for WCAG 2.1.1 (keyboard) and 2.4.3 (focus order).**
- **Suggested:** Use Radix Dialog or a headless-UI primitive.

### [A11Y-003] Icon-only buttons missing `aria-label`
- **Files:**
  - `components/layouts/navbar.tsx` — mobile menu toggle
  - `components/layouts/notification-bell.tsx` — backdrop close button
  - Various dashboard close-X buttons
- **Issue:** Screen readers announce "button" with no purpose.
- **Suggested:** `<button aria-label="فتح القائمة">` etc.

### [A11Y-004] Custom forms — focus indicators
- **Issue:** Some custom buttons (package selector in consultation form) rely on color alone for "selected" state.
- **WCAG 1.4.1 (use of color):** never rely on color alone — also use shape, weight, or icon.
- **Suggested:** Add ✓ checkmark on selected package.

## Medium findings (🟡)

### [A11Y-005] Headings hierarchy in dashboard
- Some dashboard pages start with H2 not H1 (e.g., `/dashboard/gifts`).
- **Suggested:** One H1 per page, then H2 / H3 in order.

### [A11Y-006] `<html lang>` set to `ar` on Arabic locale — confirmed ✅
### [A11Y-007] `dir="rtl"` applied at root — confirmed ✅

### [A11Y-008] Color contrast risks
- `text-jet-300` (#a3a3a3) on white — contrast ~2.6:1 (FAILS AA for body text at <18pt).
- `text-jet-400` (#7a7a7a) on white — ~4.3:1 (borderline; AA pass for body, AA fail for UI).
- **Suggested:** Restrict `text-jet-300` to decorative/disabled states only. Body copy ≥ `text-jet-500`.

### [A11Y-009] No skip-to-main-content link
- Keyboard users tab through navbar on every page load.
- **Suggested:** Add `<a class="sr-only focus:not-sr-only" href="#main">انتقل إلى المحتوى</a>` before navbar.

### [A11Y-010] Form inputs without `id` + `<label htmlFor>` association
- Spot-check shows most forms use the Input component which handles this. Spot-check `consultation-form.tsx` — labels are siblings with `htmlFor`. Good.
- Risk areas: search inputs in navbar / programs page — verify they have labels (even visually-hidden).

### [A11Y-011] Images without `alt`
- Hero watermark uses empty `alt=""` correctly (decorative).
- Logo `<img>` in navbar: `alt="أكاديميّة ومضات"` — good, but should match the brand-plain rule (remove tashkeel).

### [A11Y-012] Marquee animations don't pause on `prefers-reduced-motion`
- **Status:** Mitigated — `globals.css` has a `@media (prefers-reduced-motion: reduce)` rule that disables `wmt-*` keyframes. Verify it applies to the dynamic inline-style animations.

### [A11Y-013] Captions / transcripts on video lessons
- **File:** `components/learn/video-player.tsx`
- **Issue:** Captions track not enforced or surfaced. Required for AA 1.2.2 (captions for prerecorded video).
- **Status:** Backend/content team task — every published lesson should have at least an Arabic transcript.

## Low findings (🟢)

- [A11Y-014] Touch-target spacing OK in main CTAs.
- [A11Y-015] Form errors are announced via the regular paragraph — could be `aria-live="polite"` for clarity.
- [A11Y-016] Custom marquee in partnerships isn't keyboard-focusable but content is decorative placeholder — OK.

---

## Tools to run pre-launch

- `npx axe-cli http://localhost:3000` against every public route
- Lighthouse a11y score per page (target ≥ 95)
- NVDA / VoiceOver manual sweep on the 5 critical journeys

---

## Sign-off

**⚠️ Conditional pass.** No legal-risk blockers (alts, lang, dir all OK), but A11Y-001, A11Y-002, and A11Y-013 (video captions) are real WCAG failures. Modal focus trap is non-trivial — recommend swapping the custom dialog for Radix Dialog as a single fix that closes A11Y-002 + improves keyboard/screen-reader UX in one stroke.

— Reviewer 6 (Accessibility Specialist)
