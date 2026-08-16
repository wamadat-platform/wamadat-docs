# Beta Launch Prep Runbook

**Status:** Active. Wave 1 closed `phase-3.1-wave-1`. Beta launch gated on the 5 steps below.

## Current state (verified 2026-05-15)

| Surface | State |
|---|---|
| Real tenant admin (`asseerimishal@gmail.com`) | ✅ exists in `tenant_wamadat.users`, role=academy_owner, **mfa_enabled=true** |
| Real landlord super admin | ✅ exists in `landlord.system_users`, is_super_admin=true, **mfa_enabled=false** (enrollment incomplete) |
| Seed users in tenant | ⚠️ **225** (`@wamadat.demo` + `@wamadat.local` + `@wamadat.test`) — not cleaned up yet |
| Stray landlord row `Hadid@121212` | ⚠️ still present, is_super_admin=true |
| Published programs | ✅ 5 of 5 (operator selected Lens A: voice-over / next-js / digital-marketing / leadership / mindful-meditation) |
| Network exposure | ❌ localhost only; no external URL |
| Daily backups | ✅ running (Task Scheduler, last: 2026-05-15) |
| Restore drill | ✅ last PASS in operational hardening sprint |

## The 5-step launch sequence

### Step 1 — Verify `/admin` login (browser, by operator)

**You do this:** open `http://127.0.0.1:8000/admin/login` in an incognito window.

| What you see | What to type |
|---|---|
| Email field | `asseerimishal@gmail.com` |
| Password field | the password you set earlier |
| 2FA code field (after login) | 6 digits from your Authenticator app |

**Expected:** redirected to `/admin` dashboard with the 9-widget cockpit + sidebar showing all 16 resources.

**Send back:** `"/admin works"` OR a screenshot of any error.

---

### Step 2 — Complete `/super` 2FA enrollment

Right now `mfa_enabled=false` on the super admin row. The SuperPanel's `RequireTwoFactor` middleware will redirect you to enrollment on first successful login. You haven't completed this yet.

**You do this:** open `http://127.0.0.1:8000/super/login` in incognito.

- Email: `asseerimishal@gmail.com`
- Password: the password you set
- After login: 2FA enrollment screen → scan QR with Authenticator → confirm code
- After enrollment: arrive at `/super` super-admin panel

**Send back:** `"/super works"` OR error.

---

### Step 3 — Re-approve seed cleanup (read-only dry-run first)

**Once steps 1+2 pass, the seed cleanup can run.** The cleanup is identical to what was prepared earlier:

**Will delete:**
- 225 seed users (200 `@wamadat.demo` + 17 `@wamadat.local` + 8 `@wamadat.test`)
- ~84 seed enrollments (cascade)
- ~4 seed orders + ~5 order_items + ~4 payments + ~7 refunds
- ~80 reviews + ~30 wishlists + 0 certificates by seed users
- 89 personal_access_tokens
- (Optional, separate decision) the stray `landlord.system_users` row `Hadid@121212`

**Will keep:**
- Your real admin accounts (tenant + landlord)
- All 30 programs (5 published + 25 draft)
- All 246 lessons
- All 20 categories
- All 6 payment_webhooks (forensic log)
- Phase D contract test suite intact

**Pre-cleanup automation:** a fresh backup is taken automatically.

**Rollback anchor:** the backup `.zip` in local `backups/` + OneDrive. Procedure in `docs/operations/03-db-recovery.md`.

**Operator command (when ready):**
```
scripts\operations\seed-cleanup-execute.ps1 -Confirm
```

**I will NOT run this without you typing the green light per the original collaboration rule.**

---

### Step 4 — Network exposure: Cloudflare Tunnel

Beta users need a public URL to reach the platform. No port-forwarding on home internet; Cloudflare Tunnel keeps it secure.

**One-time setup (operator does this; ~15 min):**

1. **Sign up at Cloudflare** (free tier) — only if you don't have an account
2. **Add a domain** OR use a Cloudflare-managed subdomain
3. **Install `cloudflared`** — Windows: `winget install --id Cloudflare.cloudflared`
4. **Login**: `cloudflared tunnel login` — opens browser, authorizes
5. **Create tunnel**: `cloudflared tunnel create wamadat-beta`
6. **Config file**: `~/.cloudflared/config.yml`:
   ```yaml
   tunnel: wamadat-beta
   credentials-file: C:\Users\U\.cloudflared\<uuid>.json
   ingress:
     - hostname: beta.wmt.sa     # your subdomain
       service: http://127.0.0.1:8000
     - service: http_status:404
   ```
7. **Route DNS**: `cloudflared tunnel route dns wamadat-beta beta.wmt.sa`
8. **Run**: `cloudflared tunnel run wamadat-beta` (foreground) OR install as a Windows service for auto-start

I'll write a `scripts/operations/setup-cloudflare-tunnel.ps1` helper when you say "go" on this step — it auto-generates the config file based on your subdomain + creates a Windows service via NSSM (same pattern as queue worker supervision).

---

### Step 5 — First invitations

Once Steps 1-4 pass, you can send the first invitations. **Manual at this scale** (50-user cap):

For each invitee:
1. Open `/admin/users` (if a tenant users CRUD exists) OR send them the public registration URL (`https://beta.wmt.sa/ar/register`)
2. They register through the storefront with their real email
3. They appear in `tenant_wamadat.users` with `email_verified_at` set after email confirmation
4. They show in daily summary's `Beta verified users` count

**Daily monitoring:** the existing `Wamadat-Daily-Summary` Task Scheduler entry already runs at 08:00 and produces `docs/operations/summaries/summary-YYYYMMDD.txt`. Watch:
- `Beta verified users: N / 50 (ok / !! near cap / *** AT CAP ***)`
- `Active programs: 5 / 5`
- `Failed 24h: X` (payments)
- `webhook STALE` warnings

When the cap line goes from `ok` to `!! near cap`, stop sending invitations.

---

## What I will NOT do without your explicit approval

- Run `seed-cleanup-execute.ps1` (Step 3) — needs `"cleanup approved"` from you
- Install cloudflared or modify your firewall — needs `"setup tunnel"` from you
- Add users or send invitations — your call entirely
- Delete the `Hadid@121212` landlord row — needs explicit OK

## Order of operations (recommended)

1. ✅ B done (Wave 1 closeout merged to main)
2. ⏸ **Step 1**: you log in to `/admin` — confirm
3. ⏸ **Step 2**: you complete `/super` 2FA enrollment — confirm
4. ⏸ **Step 3**: you say `"cleanup approved"` → I run with `-Confirm` flag
5. ⏸ **Step 4**: you say `"setup tunnel"` → I write the PowerShell helper + walk you through the 8 manual setup steps
6. ⏸ **Step 5**: you send first 1-3 invitations
7. (After 14 days of green daily summaries) → Wave 2 starts
