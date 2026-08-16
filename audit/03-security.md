# 🛡️ Security Audit — Reviewer 3 (OWASP + PDPL)

**Scope:** Auth, RBAC, sessions, headers, encryption, input validation, tenant isolation, payment, PDPL.

---

## Critical findings (🔴)

### [S-001] No password-reset / forgot-password endpoint — **OPEN**
- **OWASP:** A07 Identification & Authentication Failures
- **Issue:** No `/auth/forgot-password` or `/auth/reset-password` route exists. The frontend `/forgot-password` page has nowhere to submit to. Users with compromised credentials cannot recover their accounts.
- **Required fix:** Implement Laravel built-in password reset (or custom with rate-limited token email). Block from production launch.
- **Status:** Open — not in scope for this audit pass to implement end-to-end (needs email infra).

### [S-002] No account lockout after failed login attempts — **PARTIALLY MITIGATED**
- **OWASP:** A07
- **Issue:** Throttle `5/min per email+IP` exists, but a distributed attack from 50 IPs hits 250 tries/min without ever locking the account.
- **Suggested fix:** After 5 failures over any time window, lock account for 15 min. Track in `users.locked_until`.
- **Status:** Open — needs DB column + AuthService logic.

### [S-003] Missing security response headers — **FIXED**
- **OWASP:** A05 Security Misconfiguration
- **Was:** No CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy on responses.
- **Fix:** Added `App\Http\Middleware\SecurityHeaders` middleware, registered globally in `bootstrap/app.php`. Verified live: all 5 headers present on response.
- **Status:** ✅ Fixed in this audit pass.

### [S-004] Review deletion ownership check was service-only — **HARDENED**
- **OWASP:** A01 Broken Access Control (IDOR)
- **Issue:** Controller fetched review by ID alone; service then checked ownership. While not exploitable in practice, the controller leaked existence info via 404 vs 422 timing.
- **Fix:** Controller now scopes the query by `user_id` so the route returns the same 404 whether the review doesn't exist OR belongs to someone else. Service ownership check stays as defense-in-depth.
- **Status:** ✅ Fixed in this audit pass.

## High findings (🟠)

### [S-005] Session lifetime is 7 days (10080 minutes)
- **OWASP:** A07
- **File:** `.env.example:51`, cookie max-age in `AuthController.php:115`
- **Suggested:** Drop to 24h with refresh-on-activity. Add explicit logout on password change.
- **Status:** Documented for production config.

### [S-006] MFA only available for landlord super-admins
- **OWASP:** A07
- **Files:** `TwoFactorService` exists, but `users.mfa_enabled` column never wired to a tenant-facing enrollment flow.
- **Suggested:** Add `/me/2fa/enroll` + `/me/2fa/verify` API + dashboard UI. Force MFA for `admin`, `finance`, `instructor` roles.
- **Status:** Open — sized as 2-day task post-launch.

### [S-007] National ID hash uses only APP_KEY salt
- **OWASP:** A02 Cryptographic Failures
- **File:** `UserModel.php:81-88`
- **Issue:** HMAC-SHA256 keyed on `APP_KEY` — single secret, no per-user salt.
- **Suggested:** Either per-user salt (loses lookup ability) OR rotate APP_KEY using a versioned-keys design. Document trade-off explicitly in PDPL register.
- **Status:** Documented. PDPL register entry needs updating.

### [S-008] Assignment file URLs not validated for malicious targets
- **OWASP:** A04 Insecure Deserialization / SSRF
- **File:** `AssignmentController.php:90-96`
- **Issue:** `files.*.url` accepted as bare string. No domain allow-list, no scan, no signed URL requirement.
- **Suggested:** Restrict to `https://` + allow-listed CDN domains. Reject internal/private IP ranges.
- **Status:** Open.

### [S-009] LOG_LEVEL=debug leaks sensitive data in logs
- **OWASP:** A09
- **File:** `.env.example:71`
- **Suggested:** Production `.env` must override to `LOG_LEVEL=warning`. Add a deploy check.
- **Status:** Document in ops runbook.

### [S-010] LoginRequest password validation was `min:1` — **FIXED**
- See B-001 in backend report. Now `min:8, max:160`.
- **Status:** ✅ Fixed.

## Medium findings (🟡)

### [S-011] Session timeout never invalidated on password change
- **File:** `ProfileController.php:77-83`
- **Issue:** Code revokes "other" tokens but current request stays authenticated. A leaked active session survives a password rotation.
- **Suggested:** Revoke ALL tokens on password change; require explicit re-auth.

### [S-012] No CAPTCHA on contact / newsletter / consultations
- Throttle exists but motivated bot via 50 IPs bypasses.
- **Suggested:** Cloudflare Turnstile for production.

### [S-013] Reviews + lesson notes accept raw text up to 5000 chars
- **Issue:** XSS risk if anyone renders this with `dangerouslySetInnerHTML` later.
- **Suggested:** Strip HTML server-side; document the "store as plaintext, render as plaintext" contract.

### [S-014] CORS pattern allows any `localhost:*` port in dev
- **File:** `config/cors.php:24`
- **Status:** Acceptable for dev only — make sure production `FRONTEND_URL` env is the explicit allow-list, not the regex.

### [S-015] Webhook signature verification missing length check
- **File:** `PaymentWebhookController.php:64`
- **Issue:** Signature truncated to 500 chars but no constant-time comparison or length validation.

### [S-016] Data export endpoint has no rate limit
- **File:** `routes/api.php:151` — `/me/data-export`
- **Suggested:** `->middleware('throttle:5,60')` (5 per hour).

## Low findings (🟢)

- [S-017] Bcrypt cost factor not pinned — default 10. Consider 12 in production.
- [S-018] No formal PDPL data register published.
- [S-019] Audit trail (who-changed-what) not implemented at admin layer.
- [S-020] Stripe / Tap secrets verification — confirm not committed to git (sampled `.env.example` looks clean).

---

## OWASP Top 10 mapping

| OWASP | # of findings |
|---|---|
| A01 Broken Access Control | 2 |
| A02 Cryptographic Failures | 1 |
| A03 Injection | 0 |
| A04 Insecure Design | 1 |
| A05 Security Misconfiguration | 2 |
| A06 Vulnerable Components | 1 |
| A07 Identification & Auth Failures | 5 |
| A08 Software & Data Integrity | 0 |
| A09 Logging Failures | 1 |
| A10 SSRF | 1 |

## PDPL (Saudi) status

- ✅ Right to delete: implemented (soft-delete + 30-day purge)
- ✅ Right to access: data-export endpoint exists
- ⚠️ Data minimization in logs: pending (LOG_LEVEL fix)
- ⚠️ National ID encryption: single-key design (acceptable, document)
- ❌ Breach notification process: documented? No PDPL register file exists.

---

## Sign-off

**❌ Block production launch until:**

1. **S-001** — Password reset flow implemented end-to-end (with email)
2. **S-002** — Account lockout after N failed attempts
3. **S-006** — MFA available for tenant admins / instructors / finance

**✅ Fixed in this audit pass:**
- S-003 (Security headers)
- S-004 (Review IDOR controller hardening)
- S-010 (Login password minimum)

The other 14 findings can ship to a closed-beta + production-soft-launch if documented in the security register. Critical 3 above are not optional.

— Reviewer 3 (Security Engineer)
