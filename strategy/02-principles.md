# Wamadat — The Three Principles

> *Three. Not ten. A board that needs ten principles has no principles.*

---

## I. Arabic-first, not Arabic-translated

### Definition
Every decision is tested by one question: *does the Arabic user's experience equal or exceed the English-language equivalent?*

If the answer is "almost" — the decision is wrong.

### The test
Open any page on Wamadat side-by-side with Coursera Arabic, Edraak, and a comparable English product. The Arabic version of Wamadat must look *more* deliberate, *more* readable, *more* finished than each.

### What this means in practice
- RTL is the **default**, not a flag.
- Tajawal kerning + line-height calibrated for the Arabic letter, not Latin.
- All UX copy is written in Arabic first, English is the translation.
- Saudi names (Mohammad ≠ Muḥammad ≠ محمد), Saudi dates (Hijri available, Gregorian fallback), Saudi numbers (Arabic-Indic digits where the user prefers), Saudi currency (ر.س with proper RTL placement).
- Error messages are written in the linguistic register that matches who reads them: students get warm, instructors get crisp, admins get terse.

### Anti-pattern
"We support Arabic." If you ever hear yourself say that about Wamadat, you have failed this principle.

---

## II. Operator-owned, not marketplace-controlled

### Definition
Every academy on Wamadat **owns**: their brand, their content, their pricing, their customer relationship, their data. Wamadat does not sit in the middle.

### The test
Can an academy leave Wamadat tomorrow morning with everything intact (content, students, certificates, revenue history, branding assets)? If yes, principle holds. If no, principle is broken.

### What this means in practice
- The Page Builder + Site Settings (already shipped 2026-05-16) is not a feature — it is the embodiment of this principle.
- Wamadat **never** emails the academy's students directly except for transactional auth (password reset, OTP, receipt). The academy owns marketing.
- Wamadat **never** shows other academies' content to a student. No "students also enrolled in" cross-tenant suggestions. Ever.
- Pricing is set by the academy. Wamadat takes a published, fixed transaction fee — no opaque revenue share, no "promotional discounting" we impose.
- PDPL data export endpoint exists and works (verified 2026-06-08).

### Anti-pattern
"We give the academy data, but the relationship is ours." This is how every marketplace dies — vendor mistrust accumulates. Wamadat sells the rail, never the customer.

---

## III. Governance is the product, not a feature

### Definition
In Saudi Arabia and the Gulf, trust is not earned by marketing — it is earned by **demonstrable governance**. Trust is the moat.

### The test
A skeptical compliance officer at a 200-employee Saudi corporation, evaluating Wamadat for internal training procurement, should be able to find, in under 5 minutes on the public site, evidence of: ZATCA compliance, PDPL data-export, audit log immutability, multi-tenant isolation, breach notification policy, and the operator's actual contact.

### What this means in practice
- The technical work is largely done (audit_logs immutable trigger, PDPL endpoints, ZATCA Phase 2 lib, schema-per-tenant isolation, 2FA mandatory on admin panels). The remaining work is **showing it**.
- `/trust` page (this initiative, today) makes governance verifiable, not just claimed.
- `/security.txt` (RFC 9116) makes us reachable to security researchers — a marker of seriousness.
- Every certificate carries a "verified by Wamadat" mark with a public verification URL (already implemented at `/verify/{code}`, needs visual prominence).
- Status page (next initiative): live signals that we are honest about uptime.

### Anti-pattern
"We have audit logs but we don't show them" — invisible governance is no governance. The compliance officer must *see* what we do, not trust us that we do it.

---

## How to use these three

When facing any product decision:

1. Does this serve principle I, II, or III? If none, do not ship.
2. Does this **violate** principle I, II, or III? If yes, do not ship even if it serves the others.
3. If it is principle-neutral, deprioritise. Time is short.

---

*A principle that is contradicted on day 30 was never a principle — it was a slogan.*

— *board, 2026-06-08*
