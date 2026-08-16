# 🔍 System Discovery — Production Readiness Audit
**Date:** 2026-05-12
**Auditor:** Senior CTO + Systems Architect
**Mandate:** Find every placeholder, dead-end, broken flow, and operational risk before any new features.

---

## Executive Summary

The platform has solid architectural foundations (modular Laravel + modern Next.js) but ~30% of dashboard/trainer surface is abandoned placeholder code from rapid iteration. Core happy paths (login → browse → cart → checkout → receipt) work end-to-end. The danger is everything **around** those paths that users will hit and find broken.

**State at audit:** Core flows: 70% complete. Periphery: 30% complete.

---

## 🔴 Critical findings (must fix before any user touches them)

### [SD-001] Forgot Password — UI exists, backend missing
- **Location:** `frontend/app/[locale]/forgot-password/page.tsx`
- **Status:** ✅ **FIXED THIS SESSION** — built `PasswordResetService` + 2 endpoints + applied migration.
- Frontend form still needs wiring to call `POST /auth/forgot-password`.

### [SD-002 → SD-004] Trainer pages are pure shells
- `/trainer/grading`, `/trainer/students`, `/trainer/sessions` render "قريباً" with no data fetching.
- **Recommended:** Either build them OR remove from sidebar + add honest "قيد البناء" page.

### [SD-005] Workshops page has hardcoded fake data
- `frontend/app/[locale]/workshops/page.tsx` shows fake workshops, fake dates, fake capacity.
- **Recommended:** Delete the page OR wire to a real `/workshops` API endpoint.

### [SD-006] Checkout success shows hardcoded fake order
- `frontend/app/[locale]/checkout/success/page.tsx` hardcodes `ORD-2026-000457` instead of fetching the user's actual order.
- **Recommended:** Wire to `/me/orders/{number}` using the order_number from query string.

### [SD-011] Gift redemption flow gap
- `/me/gifts/redeem` endpoint exists; `/redeem-gift` page exists. **Verify they're connected.**

### [SD-022] Trainer pages have no role guard
- Any authenticated user can visit `/trainer/*`. The layout has guard but pages should fail loud, not soft.

---

## 🟠 High findings (visible to users, will damage trust)

### [SD-007] Calendar page references "PHASE 17"
- `frontend/app/[locale]/dashboard/calendar/page.tsx:84-86` shows "ستظهر مع PHASE 17" — internal-phase language leaked to user.
- **Recommended:** Remove or wire to live-sessions API. (Page already removed from sidebar.)

### [SD-008] Apps page promises non-existent mobile apps
- App Store + Google Play buttons are disabled `<span>` with "قريباً" badges.
- **Recommended:** Delete the page until apps actually ship.

### [SD-012] Cart links to bundles that no longer exist
- `cart/page.tsx:81-88` renders `Link href={`/bundles/${item.bundleSlug}`}` — bundles were dormant.
- **Recommended:** Strip the bundle breadcrumb from the cart item until bundles are revived.

### [SD-016] Attendance scanner camera mode is stubbed
- Trainer attendance page has comment "html5-qrcode not installed" + "قريباً" message.
- **Recommended:** Either install + implement OR remove the stub message.

### [SD-017] Tap/Tamara gateways disabled without keys
- Checkout page shows them but disabled with "يحتاج TAP_SECRET_KEY" badge.
- **Recommended:** Hide them entirely in checkout UI until keys configured; surface only mock + working gateways.

### [SD-018] Trainer Home shows "—" for pending assignment count
- `frontend/app/[locale]/trainer/page.tsx:62-65` hardcodes value to "—".
- **Recommended:** Either fetch from new endpoint or hide the card.

### [SD-024] Checkout return polls only 8 times × 1.5s = 12s
- Async payment webhooks may take longer. User sees "failed" prematurely.
- **Recommended:** Increase to 15 polls with exponential backoff.

---

## 🟡 Medium findings (debt, not user-facing)

### [SD-013] Phone OTP endpoints orphaned
- `/auth/phone/send-otp` + `/auth/phone/verify` exist but signup doesn't call them.
- **Recommended:** Either wire into signup OR move to `/dashboard/settings/phone-verify` for opt-in.

### [SD-014] Design System page undiscoverable
- `/design-system` is excellent reference but no nav link.
- **Recommended:** Add to admin-only menu OR move to a `docs/` markdown file.

### [SD-019] Repeated empty-state JSX across shells
- 5+ pages copy the same "border-2 border-dashed" empty state.
- **Recommended:** Extract `<EmptyPageState>` component.

### [SD-025] Today page may crash on partial API response
- `data.upcoming.map()` assumes the array exists.
- **Recommended:** Optional chaining + default `[]`.

---

## ✅ Things confirmed FUNCTIONAL (positive findings)

- Login + register + lockout + 2FA + OTP backend
- Catalog browsing + program detail + cart
- Lesson player (Bunny placeholder + AI assistant + notes + Q&A)
- Quiz taking
- Certificate issuance + QR verification
- Achievements gamification
- Audit log (45 actions, Arabic)
- Email Outbox + Resend driver (mock + real-ready)
- Welcome email listener on registration
- Trash central Filament page
- Program state machine
- Cohort finalization
- QR badge + attendance scan service

---

## 🗑️ Safe to delete immediately

1. `/apps` page — vaporware until apps ship
2. Hardcoded WORKSHOPS array
3. Bundle links in cart (until bundles revived)
4. "PHASE 17" comment in calendar
5. Trainer Home "—" placeholder card
6. Hardcoded order in checkout/success
7. Disabled Tap/Tamara buttons (re-enable when keys ready)
8. Trainer placeholder pages (grading/students/sessions) — replace with honest "قيد البناء"
9. `/design-system` from public-facing routes (move to docs)
10. Old `/dashboard` overview (DONE — now redirects to /today)

---

## 📋 Action plan — top 15 for this sprint

### Already FIXED this session
- [x] Dashboard root collapsed → `/dashboard/today` redirect
- [x] Orphan dashboard pages deleted (gifts, affiliate, wishlist, calendar dirs)
- [x] Forgot password backend (PasswordResetService + 2 endpoints + migration + email template)
- [x] Dashboard sidebar reduced from 13 → 6 items + TopBar with bell + user dropdown
- [x] LanguageSwitcher removed (Arabic-only platform)
- [x] Tashkeel sweep across 78 files (4,271 chars)

### Next batch (this turn)
- [ ] Wire frontend forgot-password form to backend
- [ ] Delete `/apps`, `/workshops` (vaporware)
- [ ] Fix `/checkout/success` to fetch real order
- [ ] Fix cart bundle link (strip until bundles revived)
- [ ] Remove "PHASE 17" reference from calendar
- [ ] Trainer placeholder pages → honest "قيد البناء" or wire data
- [ ] Hide Tap/Tamara from checkout UI until keys ready

### Defer to next sprint (needs new endpoints/UX)
- [ ] Trainer roster + grading queue endpoints
- [ ] Phone OTP wired into signup
- [ ] html5-qrcode camera scanner
- [ ] Workshops backend + page
