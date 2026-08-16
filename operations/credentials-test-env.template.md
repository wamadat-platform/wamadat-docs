# Credentials — Local Test Environment Only

> **THIS FILE IS A TEMPLATE.** Copy it to `credentials-test-env.txt` (gitignored)
> and fill in real values there. Never commit real credentials.

## Why this exists

During Closed Beta on the dev machine, the operator needs a record of the
real (non-seed) admin accounts created via `scripts/operations/create-real-admins.bat`.
This file is the operator's local-only reference.

## How to use

1. Copy this file: `cp credentials-test-env.template.md credentials-test-env.txt`
2. Fill in the values below in `credentials-test-env.txt`.
3. `credentials-test-env.txt` is gitignored via `/docs/operations/credentials-test-env.txt` — it never reaches git/GitHub.
4. Verify `git status` shows no trace of it before committing anything.

## Template — copy into credentials-test-env.txt

```
=========================================
Wamadat Beta — Real Admin Accounts
Created: <YYYY-MM-DD>
=========================================

[Tenant academy_owner — /admin]
  URL:          http://127.0.0.1:8000/admin/login   (or production URL)
  Email:        <fill-in>
  Display name: <fill-in>
  Password:     <fill-in — long random; you typed it interactively>
  2FA secret:   <after enrolling at /admin/mfa-setup, copy the QR-derived secret here as backup>
  Recovery codes:
    1.  <fill-in>
    2.  <fill-in>
    ...

[Landlord super admin — /super]
  URL:          http://127.0.0.1:8000/super/login
  Email:        <fill-in>
  Display name: <fill-in>
  Password:     <fill-in>
  2FA secret:   <fill-in after enrollment>
  Recovery codes:
    1.  <fill-in>
    2.  <fill-in>
    ...

[Notes]
  - Both accounts MUST have 2FA enrolled before production traffic.
  - If the dev machine is lost, recovery requires:
      a) restore DB from OneDrive backup (03-db-recovery.md),
      b) re-create credentials via wamadat:create-tenant-owner /
         wamadat:create-super-admin (interactive password prompt).
  - Recovery codes above let you log in once each if the 2FA app is lost.
  - Rotate these passwords if the dev machine is ever shared, repaired,
    or recovered by a third party.
```

## Defensive checklist

- [ ] `credentials-test-env.txt` exists locally
- [ ] `git status` does NOT list `credentials-test-env.txt`
- [ ] Both 2FA enrollments completed
- [ ] Recovery codes copied
- [ ] Test login worked at /admin AND /super
- [ ] Existing `super@wamadat.test` / `admin@wamadat.test` are NOT deleted yet (still need them as fallback until real accounts proven)
