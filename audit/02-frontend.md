# 🎨 Frontend Audit — Reviewer 2 (Senior Next.js Engineer, 12 yrs)

**Scope:** 94 TSX files, 48 app routes, 40 components, 20 API client modules, 2 Zustand stores.
**Method:** Spot-audit of high-traffic components + structural pass on stores, routing, types.

---

## What's strong ✅

- App Router used correctly with locale prefix (`[locale]`)
- next-intl integration consistent
- httpOnly cookie auth design respected — no token in localStorage
- Logical RTL properties used in most places (`start-*` / `end-*` / `ms-*` / `me-*`)
- TypeScript strict mode appears active (no implicit-any errors compile-clean)
- Tailwind tokens used over hard-coded hex in nearly every component
- Suspense boundaries around search params in `sign-in/page.tsx`

---

## Critical findings (🔴)

_None._

## High findings (🟠)

### [F-001] 40+ pages missing `metadata` / `generateMetadata` exports
- **Impact:** No SEO `<title>` / `<meta description>` / OG image. Search visibility weakened, social sharing looks broken.
- **Examples:**
  - `app/[locale]/learn/quizzes/[id]/page.tsx`
  - `app/[locale]/dashboard/*/page.tsx` (probably intentional for auth-gated)
  - `app/[locale]/cart/page.tsx`
- **Suggested:** Public pages MUST have metadata. Dashboard pages can skip but should set `robots: 'noindex'`. Add a `lib/seo.ts` helper for consistent meta generation.
- **Status:** Documented. Public-facing fix is medium effort.

### [F-002] Unsafe `as never` casts in routing
- **Files:**
  - `app/[locale]/programs/[slug]/page.tsx:136` — verification link cast
  - `app/[locale]/sign-up/page.tsx:37` — redirect URL cast
  - `components/programs/program-detail-cta.tsx` — similar pattern
- **Issue:** `as never` silences TS but loses runtime safety. Suggests we're fighting next-intl Link typing rather than typing properly.
- **Suggested:** Define a `LocalizedHref` union and a `localized()` helper instead of per-call casts.
- **Status:** Documented.

### [F-003] `visibilitychange` listener leak in notification bell
- **File:** `components/layouts/notification-bell.tsx:60`
- **Issue:** Adds `document.addEventListener('visibilitychange', ...)` but never `removeEventListener` on unmount.
- **Impact:** Memory leak + duplicated callbacks if the component mounts/unmounts repeatedly (e.g., layout swaps).
- **Suggested:** Standard `useEffect(() => { ...; return () => doc.removeEventListener(...); }, [])` cleanup.
- **Status:** Documented.

## Medium findings (🟡)

### [F-004] Tailwind `<img>` instead of `next/image`
- **Impact:** No automatic AVIF/WebP, no priority hint, no responsive `srcset`.
- **Examples:** `program-detail-cta.tsx`, `hero.tsx` (watermark), various dashboard cards.
- **Status:** Acceptable for SVG brand assets (`wmt-mark-dark.svg`) but should be Image for any raster.

### [F-005] Implicit `any` in form `register` callback
- **File:** `app/[locale]/dashboard/settings/page.tsx:204`
- **Suggested:** Type the field path explicitly.

### [F-006] `useEffect` dep-arrays partially silenced
- **Files:** `components/learn/video-player.tsx:48-49,59-60`
- **Issue:** ESLint `react-hooks/exhaustive-deps` disabled inline. `lessonDuration` not in deps but used inside.
- **Status:** Risk of stale value after props change — worth a careful re-review.

### [F-007] Custom dialog (gift program) has no focus trap or ESC handler
- **File:** `components/programs/gift-program-button.tsx`
- **Impact:** Keyboard users can't easily dismiss the modal. Surfaced again in A11y report.

## Low findings (🟢)

- [F-008] Some Hero / Partnerships components could lazy-load (no SSR critical content).
- [F-009] Bundle analyzer not configured — no visibility on bundle size.
- [F-010] Some marketing components have very long static arrays inline (testimonials 50 entries) — fine for now but consider extracting to a JSON for content team edits.

---

## Sign-off

**⚠️ Approved with documented work items.** No critical issues found, but the metadata gap is a real SEO miss for marketing pages and should be closed before public launch. Code quality is high overall — type safety, RTL, and design tokens are all respected.

— Reviewer 2 (Frontend Engineer)
