# Wamadat Governance Revision — 13 September 2026

**Baseline release:** `WAMADAT-V1.0.0`  
**Baseline effective date:** `2026-09-09` — unchanged  
**Documentation revision date:** `2026-09-13`  
**Purpose:** incorporate final pre-sign-off business decisions without moving the V1 closure boundary.

## Decisions incorporated

1. **Permanent Single-Academy product boundary**
   - Wamadat is a product for Wamadat Academy only.
   - There is no future multi-academy product roadmap.
   - Existing tenant-aware/schema-per-tenant infrastructure is documented only as a legacy technical implementation detail.
   - Any future simplification/removal of that layer is Architecture/Technical Debt work, not Product Expansion.

2. **Mobile App becomes an early post-V1 priority with Flutter fixed**
   - Flutter Android/iOS is outside Launch V1 but is no longer treated as a late strategic item.
   - Flutter is the final implementation technology.
   - The old React Native/Expo codebase is retired and will not be reused.
   - MVP scope, API readiness, capacity and release assignment still pass through the governance gates.

3. **Payment integrations are explicitly part of V1 delivered scope**
   - Tap Payments.
   - Tabby.
   - Tamara.
   - Bank Transfer remains part of V1.
   - Provider availability at any given time remains dependent on production credentials/configuration/provider approval.

4. **Marketing and analytics integrations are explicitly part of V1 delivered scope**
   - Google Tag Manager (GTM).
   - Google Analytics 4 (GA4).
   - Meta Pixel.
   - TikTok Pixel.
   - Snapchat Pixel.
   - Commerce/conversion events and current attribution metadata are documented as part of the baseline capability.

5. **Program Interest / Waitlist is part of V1 delivered scope**
   - Public interest capture is approved in V1.
   - Admin lead visibility/export is approved in V1.
   - It is removed from the Deferred Registry and future Candidate Roadmap.

6. **Flutter clean implementation is authoritative**
   - No React Native/Expo continuation path remains.
   - Mobile planning now focuses on MVP/API/store readiness, not framework selection.

## Governance decisions added

- `DEC-013` — Permanent Single-Academy product.
- `DEC-014` — Mobile App early post-V1 priority.
- `DEC-015` — Payment and marketing analytics integrations are part of V1 delivered baseline.
- `DEC-016` — Flutter clean mobile implementation; legacy React Native/Expo retired.
- `DEC-017` — Program Interest / Waitlist is part of V1 delivered baseline.

`DEC-002` is retained only as a superseded historical decision for auditability.

## Source verification

The current uploaded backend/web source was checked for the documented integrations:

- Backend exposes/configures `gtm_id`, `ga4_id`, `meta_pixel_id`, `snapchat_pixel_id`, and `tiktok_pixel_id` through the integration settings/API.
- Web integration loaders include GTM, GA4, Meta, Snapchat and TikTok.
- Commerce analytics implement ecommerce/conversion events through the purchase funnel.
- Checkout/payment code registers and handles Tap, Tabby and Tamara gateway flows.

## Files materially revised

- `governance/00-PRODUCT-GOVERNANCE-README.md`
- `governance/01-V1-RELEASE-CLOSURE-AND-ACCEPTANCE.md`
- `governance/02-CURRENT-PRODUCT-BASELINE.md`
- `governance/03-PRODUCT-ROADMAP.md`
- `governance/05-CHANGE-REQUEST-PROCESS.md`
- `governance/09-TECHNICAL-DEBT-AND-RISK-REGISTER.md`
- `governance/10-DECISION-LOG.md`
- `governance/11-INITIAL-PRODUCT-BACKLOG-2026-09-13.md`
- `governance/12-TRIAGE-01-PREPARATION-2026-09-13.md`
- `governance/13-MOBILE-FLUTTER-FOUNDATION-BRIEF.md`
- `qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md`
- `qa-final/02-Wamadat-Product-Development-Roadmap.md`
- `qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md`
- `qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md`
- `technical/01-Current-Technical-Architecture.md`
- `technical/03-Payments-and-External-Integrations.md`
- two Brand System references were aligned so they no longer describe Wamadat as a multi-academy product.

## Sign-off status

The documentation is **ready for final sign-off**. The only remaining manual action is to complete the Acceptance Record in:

`governance/01-V1-RELEASE-CLOSURE-AND-ACCEPTANCE.md`

No signature, representative name, or client acceptance value has been fabricated by this revision.
