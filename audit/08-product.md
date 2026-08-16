# 📊 Product Audit — Reviewer 8 (Product Manager)

**Method:** End-to-end journey check for each role, plus business-logic edge-case inventory.

---

## Journey completeness by role

### Visitor → Buyer: 🟢 Functional
- Discovery → cart → register → purchase paths all exist and wire to real endpoints.
- Gaps: guest checkout (intentional?), saved-cart UX, cart preview in navbar.

### Student → Learner: 🟢 Functional
- Enrollment, lessons, quizzes, certificates, achievements, reviews — all complete.
- Gaps: "resume where you left off" surface (UX-004), explicit quiz retry from failure screen (UX-006), Q&A on lessons untested.

### Instructor → Content Creator: 🔴 Broken
- Role exists in backend, but:
  - No onboarding application flow
  - No instructor dashboard UI
  - No "create program" frontend surface
  - No grading queue UI
  - No earnings/analytics view
- **Action:** Owner decision needed — manual onboarding via support, OR build the self-service flow. This is a 2-3 week effort if greenfield.

### Admin → Academy Operator: 🟡 Partial
- Filament resources exist (Orders, Invoices, Coupons).
- Missing: User management (suspend/ban), Program moderation queue, Reviews moderation, Refunds queue, Tenant settings.

### Affiliate → Referral Partner: 🟡 Partial
- Backend module + dashboard exist; commission calculation logic exists.
- Missing: visible payout flow, real tracking of clicks/conversions UI, terms acceptance.

### Super Admin (Academy Owner): 🟡 Partial
- `/super` Filament panel for landlord operations exists.
- Missing: new-tenant onboarding wizard, plan management UI, system health dashboard.

---

## Business logic edge cases

| Scenario | Handled? | Risk |
|---|---|---|
| Payment failure → cart preserved | ✅ (cart lives in localStorage) | 🟢 |
| Network drop mid-lesson → progress saved | ⚠️ Saved per-lesson when user pauses; not continuous | 🟠 |
| User soft-deleted → enrollments survive | ✅ (soft-delete tested) | 🟢 |
| Program archived after enrollment | ❌ Untested — enrolled student access unclear | 🟠 Legal |
| Tenant suspended → student access | ❌ Not implemented | 🟠 Compliance |
| Refund within 14 days | ⚠️ Service exists, untested | 🔴 Revenue |
| Refund after 14 days | ⚠️ Logic exists, untested | 🟠 |
| Coupon stacking | ❌ Behavior undefined | 🟡 |
| Coupon expired mid-checkout | ❌ Untested | 🟠 |
| Concurrent quiz submissions | ❌ Untested | 🟡 Scoring integrity |
| 2FA disabled → old token invalidated | ❌ Not addressed | 🟠 Security |
| Free program (price = 0) | ❌ Behavior unclear | 🟡 |

## Metrics & analytics

- ❌ No tracking infra detected (Plausible / GA / Mixpanel)
- ❌ No backend analytics event recording
- ❌ Cannot measure: signup → enrollment conversion, lesson drop-off, payment success rate, affiliate conversion

**Recommendation:** At minimum, Plausible or a self-hosted lightweight analytics for the marketing surface; structured events table in the tenant DB for product analytics. This is a 1-day task and unblocks every future product decision.

---

## Feature completeness vs. MVP scope

| MVP feature | State |
|---|---|
| Browse + buy programs | ✅ Ready |
| Watch lessons + quizzes | ✅ Ready |
| Certificates with QR verify | ✅ Ready |
| Gift a program | ✅ Ready |
| Paid consultations | ✅ Ready (intake only — payment hook deferred) |
| Affiliate (referral) | 🟡 Partial |
| For-business (B2B) | 🟢 Marketing surface ready; sales flow manual |
| Live sessions | 🟡 Backend ready; UI partial |
| 2FA for students | ❌ Not exposed |
| Multi-tenant onboarding | ❌ Manual only |
| Instructor self-service | ❌ Not built |
| Email notifications | ⚠️ Events fired; listeners deferred |
| Reviews moderation | ⚠️ Backend ready; admin UI missing |
| Refund workflow | ⚠️ Service exists; admin UI missing |

---

## Recommended launch sequence

### Closed beta (private, invite-only)
- Open to: 50 students + 5 instructors (manually onboarded)
- Goal: Validate the visitor → buyer → student → certificate journey end-to-end
- Required first: QA-001 → QA-005 (checkout + webhook + refund tests)

### Soft launch (public, no marketing)
- Add: password reset, MFA for instructors, security headers (done)
- Add: instructor self-service onboarding
- Add: basic analytics

### Full launch (with PR + paid ads)
- Add: B2B sales flow polished + signed pilot
- Add: Affiliate program with public terms
- Add: Live sessions UI complete
- Add: A11y captions for all published lessons

---

## Sign-off

**❌ Hold on full launch.** Strong MVP core for individual-program purchases; weak periphery (instructor self-service, admin moderation, analytics). Closed beta is feasible NOW assuming P0 QA tests are added. Public launch needs 4–6 more weeks of focused work.

— Reviewer 8 (Product Manager)
