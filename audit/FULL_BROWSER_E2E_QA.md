# 🧪 FULL BROWSER E2E QA — Pre Soft-Launch

**Date:** 2026-05-13
**Method:** Real browser automation (Playwright + Chromium) running against a live local stack (Laravel `php artisan serve` + Next.js `next dev --turbo` + Postgres + file-driver fallbacks for Redis-less local). Backed up by parallel static analysis from six specialised agents (security/checkout/learn/dashboard/trainer-admin/public-pages/auth).
**Artifacts:**
- `qa-results/public-report.json` — machine-readable findings, 34 page-runs
- `qa-results/auth-report.json` — 56 page-runs across student/trainer/admin × desktop/mobile
- `qa-results/screenshots/` — 90 screenshots

## 0. The honest one-line verdict

**Closed-beta-blocked.** Public pages are healthy and the student journey loads correctly after the fixes in §2, but several real bugs sit on the path between "user signs up" and "user completes a program." None are architectural — all are surgical fixes (minutes to hours each).

Total found: **2 CRITICAL** (already fixed), **6 HIGH**, **11 MED**, **8 LOW**.
Score by area: Public 8 / 10 — Auth 7 — Student 8 — Learn 7 — Checkout 5 — Trainer 6 — Admin (Filament) 7. Weighted average **6.9 / 10**.

---

## 1. What was actually tested (transparency)

### Browser-tested (Playwright + headless Chromium, also verified in headed mode)
- 17 public/unauthenticated pages × {desktop 1366×800, mobile 390×844}
- 13 student dashboard pages × 2 viewports
- 3 trainer pages × 2 viewports
- 12 admin URL paths (but see §3 — Filament lives on backend port, not frontend)
- Sign-in flow (POST /auth/login + cookie set + middleware gate)
- Page-by-page: HTTP status, console errors, page errors, requestfailed events, response status codes 4xx/5xx, raw translation-key leakage, on-page text scanning

### Static-analysed but NOT browser-clicked
- Form submission flows beyond sign-in (sign-up, password reset, OTP, contact, redeem-gift)
- Quiz flow (autosave, submit, resume)
- Real payment chaos (refund webhook delivery, lost webhook, etc. — already covered by `CRITICAL_FIX_SPRINT_REPORT.md`)
- Drag-drop, file uploads, image rendering quality

### Not tested at all (call-out)
- Real Tap / Tamara API interactions (no live keys)
- Live email delivery via Resend (no DSN)
- Real DB load (a few rows of seeded data)
- Visual regression / pixel-perfect rendering (no baseline)

---

## 2. Bugs fixed during this run

### 🔴 BUG-1 — `nav.consultations` showed as raw translation key
**Severity:** CRITICAL · **Status:** ✅ FIXED in this session
**Where:** Home page navbar, plus anywhere the navbar appears.
**Root cause:** `components/layouts/navbar.tsx:21` references `t('consultations')`, but `messages/ar.json:nav` and `messages/en.json:nav` were missing the `consultations` key. The fallback `'الاستشارات'` only fires when `t()` throws, which it didn't — it silently returned the key string. Result: the menu showed literal text `nav.consultations`.
**Fix:** Added `"consultations": "الاستشارات"` to `ar.json` and `"Consultations"` to `en.json`. Also normalized `"sign_up"` to `"ابدأ مجاناً"` (was `"ابدأ مجانا"`, missing final hamza-on-alef diacritic).

### 🔴 BUG-2 — `/pricing` redirected to `/bundles` which is a 404
**Severity:** CRITICAL · **Status:** ✅ FIXED
**Where:** `app/[locale]/pricing/page.tsx`.
**Root cause:** `/pricing` was kept as a route for SEO continuity but it redirected to `/bundles`. The bundles feature was deleted in P6 cleanup (it's dormant per project memory). So `/pricing` → `/bundles` → 404.
**Fix:** Redirected `/pricing` to `/programs` instead (where actual pricing per program lives).

### 🟠 BUG-3 — PWA service worker served a stale offline page with robotic over-diacriticization
**Severity:** HIGH · **Status:** ✅ FIXED
**Where:** `public/sw.js` + browser cache.
**Root cause:** The cleaner Arabic text in `app/[locale]/offline/page.tsx` was already there, but the service worker had cached the OLD over-diacriticized version (`لا اتِّصال` / `تَحَقَّق` / `حاوِل`) from before the email-template cleanup sprint. The SW version was `wmt-v1`, and old caches were never invalidated.
**Fix:** Bumped `VERSION` from `wmt-v1` to `wmt-v2` in `public/sw.js`. The `activate` event handler already deletes caches not starting with the current version — so on next SW activation, the stale offline page is purged.
**For the user:** to see immediately, in DevTools → Application → Service Workers → click "Unregister" + reload. Otherwise the new SW will activate naturally on next visit.

---

## 3. The big findings (verified in browser unless noted)

### 🟠 H-1 — Mock payment simulator clears cart BEFORE webhook is processed
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `app/[locale]/checkout/simulate/page.tsx:38`
**Behavior:** On success-button click, `clearCart()` runs synchronously. The webhook POST to `/webhooks/mock` then fires. If the webhook fails (e.g., gateway adapter rejects payload, signature mismatch, network blip), the cart is permanently lost AND the order never reaches `paid`. User sees a success page for an order that doesn't exist.
**Fix:** Sequence must be: POST webhook → await response → if `result.ok` then `clearCart()`. If webhook fails, keep the cart and surface a retry path.

### 🟠 H-2 — Coupon UI on `/cart` is dead code
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `app/[locale]/cart/page.tsx:119-127`
**Behavior:** Input + "Apply" button render but have no `useState`, no `onChange`, no submit handler. Clicking the button does nothing. Users will conclude their valid coupon code is broken.
**Fix:** Either wire it to the same `/coupons/validate` endpoint the checkout page uses, OR remove the UI (coupons only work at checkout per the cart-vs-checkout split). Removing is cleaner — coupon stays at checkout, cart is just inventory.

### 🟠 H-3 — Sign-in vs Sign-up redirect inconsistency
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `app/[locale]/cart/page.tsx:29` (sends to /sign-up?next=/checkout) vs `app/[locale]/checkout/page.tsx:81` (sends to /sign-in)
**Behavior:** If the user clicks "Checkout" from cart while logged out → sign-up. If they directly visit `/checkout` while logged out → sign-in. Two different onboarding flows depending on entry point.
**Fix:** Pick one. Recommend sign-in by default (with a "first time? sign up" link in the form) since most checkouts are repeat customers, and the sign-up page already accepts a `next=` param.

### 🟠 H-4 — Refund admin action defaults to `requested_by_customer`
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `app/Modules/Commerce/Infrastructure/Filament/Resources/OrderResource.php:123`
**Behavior:** Filament refund action has `->default('requested_by_customer')`. If an operator refunds for any other reason (admin error, dispute, fraud) and forgets to change the dropdown, the audit log says "customer requested" — misleading downstream analysis and customer-trust math.
**Fix:** Change default to `'admin_initiated'` so operators must consciously pick. Or require the field (`->required()`) with no default at all.

### 🟠 H-5 — `/trainer` and `/trainer/attendance` are not role-gated on the FRONTEND
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `app/[locale]/trainer/page.tsx`, `app/[locale]/trainer/attendance/page.tsx`
**Behavior:** Backend properly checks instructor / cohort_lead / admin roles before serving trainer data. But the frontend pages don't wrap themselves in `<DashboardGuard requiredRole={['instructor', 'cohort_lead', 'admin', 'academy_owner']}>` — so a regular student can land on `/trainer/attendance`, see the form UI (which then fails on submit with 403). This leaks the existence of trainer pages and confuses non-trainer users.
**Fix:** Add `<DashboardGuard requiredRole={['instructor', 'cohort_lead', 'admin', 'academy_owner']}>` to both pages' tree (or to a `trainer/layout.tsx` if one doesn't exist yet).

### 🟠 H-6 — Reset-password backend min:8 vs sign-up min:10 mismatch
**Severity:** HIGH · `[CODE-LEVEL]`
**Where:** `PasswordResetController.php:37` (min:8) vs `RegisterRequest.php:145` (min:10)
**Behavior:** A user resets their password to an 8-char value — succeeds. But that password would NEVER pass sign-up validation. Inconsistent security floor. Sets up confusion in support tickets and audit reviews.
**Fix:** Align reset-password to `min:10` (same `Password::min(10)->letters()->mixedCase()->numbers()` rule as register). Reset is a higher-risk operation than register, not lower-risk.

### 🟡 M-1 — Help topic links go to non-existent routes
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `app/[locale]/help/page.tsx:68, 93`
**Behavior:** Links use template literal `/help/${topic.title}` which builds URLs like `/help/الحساب والتسجيل`. There's no `/help/[topic]` route. Clicking these gets 404.
**Fix:** Either change the help page to accordion (in-page expand) or create the dynamic route. Accordion is faster — no new pages.

### 🟡 M-2 — Login error format mismatch
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `AuthController.php:50-57` (returns `{errors: [{code, title, detail}]}`), frontend expects `{errors: Record<string, string[]>, message?: string}`
**Behavior:** When login fails (bad password), backend's structured-array errors don't match frontend's flat map. Frontend's `parseApiError` falls back to generic "حدث خطأ. حاول مرة أخرى." (something went wrong). User can't tell if their email is wrong or password is wrong.
**Fix:** Add `message: 'بيانات الدخول غير صحيحة'` to the login 401 response so frontend can surface it. Keep the array form for advanced debugging.

### 🟡 M-3 — `Dashboard /orders` shows numbers in `en-US` not `ar-SA` locale
**Severity:** MED · `[CODE-LEVEL]`
**Where:** Various dashboard pages — `orders/page.tsx:89, 99`, `certificates/page.tsx:61`, `my-programs/page.tsx:141`.
**Behavior:** Prices and counts use `.toLocaleString('en-US')` so a Saudi student sees Western numerals `1,234.50` instead of the more locally familiar formatting. Either commit to Western (decimal point, comma thousands) or to Arabic-Indic — currently mixed across the app.
**Fix:** Pick one. `ar-SA` is more locally familiar for prices in SAR but `en-US` is more readable when scanning a table of orders. Recommendation: stay with Western numerals for prices in SAR (clarity) but ALWAYS append `ر.س` after, and use `ar-SA` for COUNTS (e.g., "5 شهادات" should accept Arabic-Indic).

### 🟡 M-4 — `Settings /dark mode` is in the form but non-functional
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `app/[locale]/dashboard/settings/page.tsx:128-130`
**Behavior:** Select renders "Light / Dark / System" options. Submitting Dark saves the preference. But UI never switches to dark. Hint text says "PHASE 22 polish" but the option is still visible.
**Fix:** Either implement dark mode now OR hide the option entirely until P22. Visible-but-broken is worse than absent.

### 🟡 M-5 — Profile avatar URL field with deferred message
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `dashboard/profile/page.tsx:121`
**Behavior:** Same pattern as M-4. Form accepts a URL string for avatar, but hint says "PHASE 22 — direct upload." Either disable the input or let URL work (it might already work for any image host).
**Fix:** Verify the URL path actually renders the avatar today. If yes, keep but remove the misleading "PHASE 22" hint. If no, disable the field.

### 🟡 M-6 — Order summary on `/checkout/return` doesn't show applied coupon
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `app/[locale]/checkout/return/page.tsx`, plus `MyOrdersController::shape()` line 75-81 doesn't include `applied_coupon_id`
**Behavior:** User paid 110 SAR after a 10 SAR coupon discount. Return page shows subtotal 100 + tax 15 = total 115, total paid 110, but no "coupon" row explaining the gap. Looks like a billing error.
**Fix:** Add `applied_coupon` to `MyOrdersController::shape()` output and render the discount line on the return page.

### 🟡 M-7 — Success page has schema mismatch: `unit_price_sar` vs `price_sar`
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `app/[locale]/checkout/success/page.tsx:13-27` expects `price_sar`, backend `MyOrdersController::shape()` returns `unit_price_sar`
**Behavior:** Success page renders `undefined ر.س` for each line item. Cosmetic — user already paid — but bad first impression.
**Fix:** Rename either side. Backend's `unit_price_sar` is more accurate; update the frontend type.

### 🟡 M-8 — Long-quiz progress autosave fires every keystroke without backoff
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `components/learn/video-player.tsx:44-59` reports progress every 10s with no backoff on failure
**Behavior:** If `/progress` API returns 500 repeatedly, the 10s interval keeps hammering the backend. No exponential backoff or max-retry. Bad on a metered mobile plan + bad on backend during incidents.
**Fix:** Wrap the report call in an exponential backoff: 10s → 30s → 60s after each failure, reset to 10s on success. Cap at 5 min.

### 🟡 M-9 — Q&A reloads ENTIRE question list on each new reply
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `components/learn/lesson-qa.tsx:127` calls `onReload()` after each reply
**Behavior:** User asks a question, instructor replies within 10s, user sees the whole list flash + reload. Optimistic append would be smoother + less server load.
**Fix:** Optimistic append: `setQuestions(prev => prev?.map(q => q.id === qid ? {...q, answers: [...q.answers, newAnswer]} : q))`. Refetch only on stale.

### 🟡 M-10 — Refresh-during-quiz: resume picks first incomplete by array index
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `components/learn/learn-shell.tsx:576-593` `pickResumeLesson()`
**Behavior:** If instructor deletes a lesson between student visits, resume logic re-iterates the curriculum and lands on a different lesson than intended. The student loses context.
**Fix:** Persist `last_lesson_id` server-side (already in `lesson_progress`). On resume, prefer that ID; only fall back to "first incomplete" if the ID doesn't resolve.

### 🟡 M-11 — Hardcoded contact phone, email, address in `/contact`
**Severity:** MED · `[CODE-LEVEL]`
**Where:** `app/[locale]/contact/page.tsx:54-77`
**Behavior:** Phone `0506984434`, email `hello@wamadat.academy`, address "الرياض، المملكة العربية السعودية" all inline. Changing them requires a code change + deploy. They should be in `messages/ar.json:contact_page` so they can change without a release.
**Fix:** Move to translations file, use `t()`.

### 🔵 LOW findings (8 items — non-blocking, captured for backlog)

- **L-1** Tap webhook `raw_payload` stores parsed JSON rather than raw bytes (still works for signature verify; concerns only post-incident replay).
- **L-2** Quiz `sessionStorage` not cleared on browser back button after submit — old answers may carryover to a new attempt.
- **L-3** Cart page `aside` lacks `pb-safe` (only checkout aside got it during the critical-fix sprint).
- **L-4** AI assistant rate-limit message doesn't include retry-after timer.
- **L-5** Notes list lacks search and sort controls — becomes unusable past 20 notes.
- **L-6** `cohort_lead` role is not in `InstructorStatsWidget`'s visibility predicate — cohort leads can't see stats.
- **L-7** Mobile sidebar drawer occludes the lesson title while open (covers full height).
- **L-8** ListView pagination is missing on `/admin/audit-logs` for very large rolls (won't bite until 5k+ rows accumulate).

---

## 4. What worked correctly (give credit where due)

1. **Sign-in flow under normal browser usage** — POST /auth/login returns 200, sets `wamadat_session` HTTP-only cookie, client store sets the `wamadat_auth_hint` cookie + persists user to localStorage. Subsequent navigation to /dashboard/today renders correctly. Verified in both headless and headed Chromium.
2. **All 17 public pages return HTTP 200** on both desktop and mobile viewports. No 500s, no crashes, no PageErrors.
3. **Student dashboard, all 11 sub-pages, render correctly** for an authenticated student. Each shows real data from the seeded `student@wamadat.test` account.
4. **Trainer dashboard renders** for the seeded trainer account.
5. **R6 mobile-safe-area fix on `/checkout`** is in place (verified `pb-safe` on the aside in `app/[locale]/checkout/page.tsx`).
6. **RTL discipline is excellent** — no `text-left`, `text-right`, `ml-*`, `mr-*` snuck back in.
7. **No console errors on any of the 17 public pages × 2 viewports = 34 page-runs** except the now-fixed `/pricing` → `/bundles` 404.
8. **Translation key coverage is solid** for the navbar after the one missing `consultations` was added — the crawler's raw-key scanner found zero leaks across all 56 authenticated page-runs and 34 public page-runs.

---

## 5. Operator runbook for repeating this E2E QA

Whenever the codebase changes meaningfully, run this loop locally before deploying:

```powershell
# Terminal A — backend
cd C:\Users\U\wamadat-platform\backend
C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe artisan serve --host=127.0.0.1 --port=8000

# Terminal B — frontend (skip pnpm preinstall hook which prompts for approval)
cd C:\Users\U\wamadat-platform\frontend
.\node_modules\.bin\next dev --turbo -p 3000

# Terminal C — run the QA
cd C:\Users\U\wamadat-platform
node .\scripts\qa\crawl-public.mjs    # 17 pages × 2 viewports
node .\scripts\qa\crawl-auth.mjs      # student + trainer + admin × 2 viewports

# Inspect findings
cat qa-results\public-report.json | jq '[.[] | select(.verdict != "OK")]'
cat qa-results\auth-report.json   | jq '[.[] | select(.verdict != "OK")]'
```

Both scripts capture:
- HTTP status per page
- Console errors (`error` level)
- Failed network requests (4xx / 5xx / aborted)
- Raw translation-key leakage (e.g. `nav.something` text in the rendered body)
- Screenshots in `qa-results/screenshots/<role>-<page>-<viewport>.png`

---

## 6. What still NEEDS a real human in a real browser

Marked `[NEEDS-HUMAN-EYES]` because Playwright can't judge:

- **Visual hierarchy on the home page hero** — is "نطلق الإمكانات ونشعل ومضات التميز" legible? Does the brand orange contrast enough against the dark background?
- **Quiz mobile layout** — choice buttons readable on 5" screen without zooming?
- **Video playback on /learn** — YouTube embed actually plays, controls don't look broken in RTL, fullscreen works on mobile.
- **Form keyboard overlap on iPhone Safari** — the sign-up form has 8 fields; on iPhone SE the keyboard hides the lower fields. Playwright reports 200, but the human experience is awful.
- **Checkout flow on real iPhone with iOS Safari home indicator** — the `pb-safe` fix is structurally correct, but only a real device test confirms the button is comfortable to tap.
- **Real Tap charge** — once live keys arrive, a $1 SAR charge to a real test card + verify webhook lands + order moves to paid.
- **Email delivery** — once Resend domain is verified, send a welcome email, check SPF/DKIM/DMARC pass at the recipient.
- **AI assistant** — does it answer in good Arabic? Does it understand lesson context? Rate-limit feedback when 20/min hit?

---

## 7. Pre-launch decision

**Closed Beta readiness:** ✅ unblocked AFTER the 3 fixed bugs (BUG-1/2/3) ship to staging. The 6 HIGH bugs in §3 should also be addressed because they're each <2 hours of work and they all sit on the "first-time user" path.

**Paid acquisition / public marketing:** ⏸ wait until §3 HIGH items are fixed + the Launch-Engineering items (Tap KYC, Resend domain, Postgres prod, Redis prod, DNS) land. Those are infra, not code — see `docs/launch/00-LAUNCH_ENGINEERING.md`.

**Next concrete step:** burn through §3 HIGH (about 8-10 hours combined). Then re-run the crawlers. If green: closed beta is greenlit.

---

**End of FULL BROWSER E2E QA report.**

The hard truth: the platform LOOKS finished but has 6 surgical bugs in the user-facing flows that would each cost a customer or a support ticket. Fix the 6 HIGHs and you're confidently in closed beta. The architecture is solid; these are last-mile polish items.
