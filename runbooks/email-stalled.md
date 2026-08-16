# 🟡 Runbook — Email Delivery Stalled

Resend isn't accepting our messages, OR the outbox queue isn't draining. Users
stop getting receipts, certificates, password resets, welcome emails. **The
platform's lifeline.**

---

## Symptoms

- `/admin/email-outboxes` → `queued` count climbing, `sent` not moving
- `/admin/email-outboxes` filter "فاشِل / مُرتَدّ" shows new entries
- `HealthStatusWidget` → `resend_configured` red OR scheduler red
- User support tickets: "I didn't get the email"
- Sentry: `Resend\Exception\ResendException` (or `Symfony\Component\Mailer\Exception\TransportException`)

---

## Verify (under 60 seconds)

```bash
# 1. Test the Resend key directly
curl -sS -X POST https://api.resend.com/emails \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "from": "no-reply@wamadat.academy",
    "to": ["test@yourdomain.example"],
    "subject": "ops probe",
    "text": "ping"
  }' | jq

# Expected: 200 with {"id": "..."}
# If 401 → API key bad/rotated
# If 403 → sender domain not verified in Resend dashboard
# If 422 with "domain not verified" → SPF/DKIM regression — re-verify

# 2. Is the scheduler running on this box?
ps aux | grep "schedule:run" | grep -v grep
# → should see at least one entry from the cron daemon every minute

# 3. Drain attempt
php artisan outbox:drain
# → tells you exactly which row(s) are failing and why
```

---

## Mitigate

| Diagnosed cause | Fix |
|---|---|
| API key bad/rotated | Update `RESEND_API_KEY` in env, `config:cache`, reload |
| Sender domain unverified | Resend dashboard → re-add SPF + DKIM records — wait for green; meanwhile `RESEND_DRIVER=mock` keeps the app responsive (no real send) |
| Rate-limited (429) | Slow the outbox drain: pass a smaller `--limit` to the command (`php artisan outbox:drain --limit=25`) OR change the schedule frequency in `routes/console.php` from `everyFiveMinutes` to `everyTenMinutes` |
| Scheduler not running | Re-add cron: `* * * * * cd /var/www/wamadat && php artisan schedule:run >> /var/log/wamadat-cron.log 2>&1` |
| One specific recipient bouncing (e.g. mailbox full) | Mark that outbox row as `failed_permanent` in `/admin/email-outboxes`, notify user via another channel |
| All recipients bouncing → sender reputation issue | Pause outbound, investigate domain reputation at https://postmaster.google.com etc., do NOT keep retrying |

---

## Recover

1. Fix the root cause (table above).

2. Manually drain to catch up:
   ```bash
   php artisan outbox:drain
   # Repeat until `queued` count in /admin/email-outboxes is 0 or stable
   ```

3. For permanent bounces (e.g. mailbox doesn't exist), use the UI button "تعيين
   فاشِل دائِم" so they stop retrying. The audit log captures who marked what.

4. If the Resend webhook for delivery events also stopped landing, replay missed
   events from the Resend dashboard → `/webhooks/resend` will reconcile statuses.

---

## Post-incident

- [ ] Audit row: every retry/mark-failed action writes `ops.outbox.retried` / `ops.outbox.failed_permanent`
- [ ] Open `docs/audit/incidents/YYYY-MM-DD-email-stalled.md` with: trigger, blast radius (rows affected), root cause, fix
- [ ] If delivery rate < 95% for the day, email each affected user with a follow-up
- [ ] Set a Sentry alert on `Resend*Exception` rate > 5/hour if not already set

---

## Pre-flight check (catch before users notice)

- Daily 09:00 KSA cron: run a synthetic probe — send one email to a monitoring inbox; alert if no delivery webhook within 5 min
- Weekly: spot-check delivery rate in `/admin/email-outboxes` filter (last-7d) → expect ≥ 95% `sent`

---

## Don'ts

- ❌ Don't `pnpm outbox:drain --force` past a permanently-bouncing recipient — you'll just hammer Resend and risk a reputation hit
- ❌ Don't bypass the outbox by sending Mail directly during an outage — you'll lose the audit trail and the retry guarantee
- ❌ Don't disable `RESEND_WEBHOOK_SECRET` "temporarily" — unsigned webhooks let anyone mark our emails as delivered
