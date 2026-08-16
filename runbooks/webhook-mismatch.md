# 🟣 Runbook — Payment Webhook Signature Mismatch

Tap (or Tamara) is hitting `/webhooks/tap` and we're rejecting it with `400
invalid_signature`. Payments may succeed in the gateway but never reconcile in
our DB → user pays but stays in `pending` state.

---

## Symptoms

- `/admin/payment-webhooks` filter `failed_only` shows rows with `error_message: "signature_mismatch"`
- `HealthStatusWidget` webhooks probe shows red OR yellow
- Users complain "I paid but my enrollment is still pending"
- Tap dashboard shows successful charges with no matching `orders` row in our DB
- Sentry: nothing fires (signature failures are intentionally NOT sent to Sentry — they're noisy)

---

## Verify (under 90 seconds)

```bash
# 1. Confirm rejections are real
# In /admin/payment-webhooks, filter:
#   - gateway = tap
#   - result = failed
#   - last 1h
# Click into a row → raw_payload + error_message visible

# 2. Compare secrets
# In production env:
echo "Backend has TAP_WEBHOOK_SECRET: ${TAP_WEBHOOK_SECRET:+SET (${#TAP_WEBHOOK_SECRET} chars)}"
# In Tap dashboard → Developer → Webhooks → reveal the configured secret
# These two MUST be identical, character-for-character (no trailing whitespace, no quotes)

# 3. Confirm Tap is hitting the right URL
# Tap dashboard webhook URL should be EXACTLY:
#   https://api.wamadat.academy/webhooks/tap
# Not /api/webhooks/tap, not /webhooks/tap/, not http://
```

---

## Common root causes (in order of likelihood)

| Cause | How to spot it | Fix |
|---|---|---|
| Secret was rotated in Tap dashboard but not in our env | Recently changed in Tap audit log; ours hasn't been updated | Copy from Tap dashboard → update `TAP_WEBHOOK_SECRET` → `config:cache` → `octane:reload` |
| Secret has accidental whitespace / quotes in `.env.production` | `grep '^TAP_WEBHOOK_SECRET=' .env.production \| cat -A` shows `$` not adjacent to the last char (Linux). On macOS, use `awk -F= '$1=="TAP_WEBHOOK_SECRET"{print length($2)}' .env.production` to print the secret's length — if it's bigger than expected, there's trailing whitespace | Re-paste, no quotes, no trailing newline |
| Proxy (Cloudflare / nginx) is rewriting / decoding the body before our app sees it | `raw_payload` in `/admin/payment-webhooks` does NOT match what Tap dashboard says it sent | Configure proxy to pass body through verbatim (no body modification, no buffering before signature check) |
| App is reading parsed JSON, signing parsed JSON, expecting it to match Tap's signature of raw body | Our code uses `request()->all()` instead of `request()->getContent()` for the HMAC input | Confirm `Tap\Webhook::verifyWebhook` uses `$request->getContent()` (the raw bytes). It does — see `app/Modules/Payments/Infrastructure/Gateways/TapGateway.php:verifyWebhook` |
| Live secret vs test secret mixed up | Tap key starts with `sk_test_` in prod env | Use `sk_live_*` keys in production only |
| Tamara case: notification_token vs api_token confusion | Mismatch between `TAMARA_NOTIFICATION_TOKEN` and `TAMARA_API_TOKEN` | The webhook uses `NOTIFICATION_TOKEN`. API calls use `API_TOKEN`. They are DIFFERENT in Tamara — don't swap |

---

## Mitigate (stop the bleeding)

1. **Identify the affected orders.** In `/admin/payment-webhooks`, every failed
   row's raw_payload contains `charge.id` / `payment.id`. Cross-reference against
   `orders` table → find the orders stuck in `pending`.

2. **Confirm payment really succeeded in the gateway** before doing anything else:
   ```bash
   curl -sS -H "Authorization: Bearer $TAP_SECRET_KEY" \
     https://api.tap.company/v2/charges/<charge-id> | jq '.status, .amount, .customer.email'
   # Expected: status=CAPTURED, amount matches, email matches the pending order
   ```
   If status is anything other than `CAPTURED`, the payment did NOT succeed —
   no reconciliation needed, the customer is correctly in `pending`.

---

## Recover

1. **Fix the secret** (most likely root cause from the table above).

2. **Replay** the rejected webhooks from `/admin/payment-webhooks`:
   - Open each failed row → click **إعادة المُعالجة**
   - Status flips: `failed` → `queued`
   - Action audits as `ops.webhook.replayed`
   - The job picks it up within seconds → order moves to `paid` → invoice issued → enrollment activated → certificate eligibility recomputed
   - All standard downstream effects fire because we use the same code path, just from a different trigger

3. **Verify** by spot-checking 2-3 affected orders end-to-end:
   - Order shows `paid`
   - Invoice row exists with ZATCA QR
   - Enrollment row exists
   - Welcome / receipt emails landed in outbox (`sent` status)

---

## Post-incident

- [ ] Audit chain: `ops.webhook.replayed` rows in `audit_logs` document who acted on what — confirm count matches the rows replayed
- [ ] Customer comms: if any user contacted support during the outage, reply with order confirmation now that it's reconciled
- [ ] If the secret was rotated externally without coordination, document the channel mismatch in `docs/audit/incidents/YYYY-MM-DD-webhook-mismatch.md` and propose a process fix
- [ ] If a proxy was modifying the body, add a regression check: a smoke test that POSTs a payload with a known signature to `/webhooks/tap` and asserts 200

---

## Pre-flight check (Stage 0)

Before flipping DNS:
- Send Tap a test webhook from their dashboard → expect 200 in our app + a row in `/admin/payment-webhooks` with `result=success`
- Do the same for Tamara (Tamara has a sandbox webhook tester)
- Rotate both secrets ONCE on launch day so dev/staging secrets can never reach prod

---

## Don'ts

- ❌ Don't manually `UPDATE orders SET status='paid'` to "fix" a stuck order. The webhook path triggers idempotent side-effects (invoice, enrollment, cert, emails) — bypassing it leaves the system inconsistent.
- ❌ Don't disable signature verification "temporarily" to unstick things. The replay button does what you want; signature verification is what stops attackers from telling us their order is paid when it isn't.
- ❌ Don't redact the raw_payload from `/admin/payment-webhooks` — operators need to see exactly what came in to diagnose this class of issue.
