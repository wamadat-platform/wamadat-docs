# ADR — Queues, Outbox, and Async Dispatch

**Status:** Accepted (2026-05-13). Captures the current state of the platform's async surface so the next session doesn't reinvent the model.

**Context window:** Stage 0 / soft-launch sprint. The platform must reliably deliver transactional email + in-app notifications + reconcile lost webhooks, without a Redis cluster being provisioned yet.

---

## TL;DR

There are **three** independent asynchronous mechanisms in the codebase. They look similar but solve different problems and have different reliability guarantees. Don't merge them; do plumb them through known seams.

| Mechanism | What it carries | Worker | Idempotency | Retry | Today's state |
|---|---|---|---|---|---|
| **Email outbox** | Marketing + transactional bulk emails (template-driven) | `outbox:drain` cron every 5 min | Caller-provided (`OutboxService::enqueue`) | Yes — `attempts` column, 5-attempt max, dead-letter status | Solid. Underused for transactional flows. |
| **Laravel queue** | `ShouldQueue` mailables (NOT actually queued today) | None — `QUEUE_CONNECTION=sync` | None at queue layer | None — sync dispatch means failure surfaces immediately to caller | **Inactive**. `Mail::send()` is sync. Tracked as R-OPEN-1. |
| **Webhook replay** | `payment_webhooks` rows with `result='queued'` | `webhooks:replay-queued` cron every 2 min | DB UNIQUE on `(gateway, external_id, event_type)` + downstream handler idempotency | Implicit — the row stays `queued` until either OK or admin deletes it | Live as of Phase C part 1 (`4ff5efb`). |

The cron-driven mechanisms (outbox drain + webhook replay) **work without Redis** because they poll Postgres. This is intentional for Stage 0 — when Redis lands, the Laravel queue becomes the third leg without changing the other two.

---

## 1. Email outbox

### Schema (tenant connection)
- `email_outbox` table, UUID PK.
- Columns: `recipient_email`, `recipient_user_id`, `subject`, `body_html`, `body_text`, `template_slug`, `status` enum (`queued|sending|sent|failed|bounced|complained`), `attempts` (default 0), `gateway`, `gateway_message_id`, `error_message`, `scheduled_for`, `sent_at`, `failed_at`.
- Indexes: `(status, scheduled_for)`, `gateway_message_id`, `(recipient_user_id, created_at)`.

### Drain semantics
- `DrainEmailOutbox` console command:
  - Takes `--limit=100` per tick.
  - Filters `status IN (queued)` AND `scheduled_for <= now()` AND `attempts < MAX_ATTEMPTS (5)`.
  - Per row: flip to `sending`, call `ResendDriver::send`, on success flip to `sent` + capture `gateway_message_id`. On failure: increment `attempts`, flip to `failed` if cap hit.
- Resend driver has a `mock` mode for dev (no real API calls).

### Scheduling
- `Schedule::command('outbox:drain --limit=100')->everyFiveMinutes()->withoutOverlapping()->runInBackground();`

### What it's used for today
- Bulk marketing emails (admin sends a template to a segment).
- The `UserRegistered` event → `SendWelcomeEmail` listener enqueues into the outbox.

### What it's **not** used for today
- `OrderPaidMail`, `RefundIssuedMail`, `CertificateIssuedMail` — these still go via `Mail::to($email)->send()` synchronously. See R-OPEN-1.

### Migration path (Phase C part 2)
- Add an `idempotency_key` column with partial UNIQUE index (`WHERE idempotency_key IS NOT NULL`).
- Add a `OutboxService::enqueueFromMailable(Mailable $m, ?string $idempotencyKey)` adapter that renders the mailable via `$m->render()` and inserts an outbox row.
- Update `CheckoutService::finalizePaidOrder` and `RefundService::executeGatewayRefund` to call the adapter instead of `Mail::send()`.

---

## 2. Laravel queue (`QUEUE_CONNECTION=sync`)

### Today
- `.env` sets `QUEUE_CONNECTION=sync`. Every `dispatch()` / `Mail::queue()` runs synchronously in the request thread.
- No `app/Jobs/` directory; no `Job` classes; no `failed_jobs` rows.
- `failed_jobs` migration exists (`landlord/2026_05_13_110000_create_failed_jobs_table.php`) and the table is empty.
- `FailedJobResource` Filament UI is wired but lists 0 rows because no worker exists.

### When Redis lands (Stage 0 ops sprint)
- `QUEUE_CONNECTION=redis` in production `.env`.
- Worker process: `php artisan queue:work --queue=high,default,low --tries=3 --backoff=30,60,120`.
- Mailables transition from sync `Mail::send()` to `Mail::queue()` OR through the outbox adapter (see above). Pick ONE; don't run both.
- Failed jobs land in `failed_jobs` and become visible in `FailedJobResource`.
- Retention: `ops:failed-jobs:purge --older-than=30d` weekly (R-OPEN-7).

### Why not redis in dev yet
- Laragon ships Redis but the project hasn't pinned a host/port/auth pattern, and the soft-launch sprint hasn't budgeted a worker process supervisor (Supervisor / systemd unit / pm2). Stage 0 ops sprint owns this.

---

## 3. Webhook replay loop

### Why it exists
- Gateways retry POSTs with exponential backoff on non-2xx. The HTTP path (`PaymentWebhookController::handle`) records the webhook BEFORE processing, then dispatches. On unexpected exception it logs + returns 200 (so the gateway stops retrying) with `result='failed'`.
- The admin "Replay" action on `PaymentWebhookResource` flips `result` to `queued` — that's the operator's "try again" signal.
- Phase C part 1 added the missing piece: a drain command that actually re-runs the handler on every `result='queued'` row.

### Schema (tenant connection)
- `payment_webhooks` table, UUID PK.
- Columns: `gateway`, `external_id`, `event_type`, `signature`, `raw_payload` (jsonb), `result` enum (`queued|ok|duplicate|invalid_signature|failed`), `error_message`, `received_at`, `processed_at`.
- **UNIQUE INDEX:** `(gateway, external_id, event_type)`. Dedupes gateway retries at INSERT time.

### Drain semantics
- `webhooks:replay-queued` console command:
  - Iterates every tenant via `TenantModel::query()->get()` + `makeCurrent()`.
  - Fetches `result='queued'` rows ordered by `received_at`, limit configurable (`--limit`, default 50).
  - For each row: rebuilds the dispatch path from `raw_payload`, calls `RefundWebhookHandler` (for refund events) OR `CheckoutService::finalizePaidOrder` (for payment-success events).
  - Updates `result` to `ok` + `processed_at` on success, `failed` + `error_message` on exception.
- **Signature is NOT re-verified.** The original HTTP path already passed verification; replay trusts the persisted payload. This is intentional — without it, replaying a webhook days later would fail because timestamp-based signatures expire.

### Idempotency
- `RefundWebhookHandler::upsertRefundRow` returns null if a refund with the same `gateway_refund_id` is already `completed`. The handler then returns early — no duplicate notification, no duplicate enrollment drop, no duplicate audit row.
- `CheckoutService::finalizePaidOrder` refuses to regress from terminal non-paid states (`paid`/`refunded`/`partially_refunded`) — same effect.

### Scheduling
- `Schedule::command('webhooks:replay-queued --limit=50')->everyTwoMinutes()->withoutOverlapping()->runInBackground();`

---

## Cross-cutting decisions

### Why three mechanisms instead of one
- Outbox = persistent, queryable, admin-reviewable from `EmailOutboxResource`. Marketing wants visibility into "what got sent to whom" without a Redis console.
- Webhook replay = an admin-driven re-try of a specific event, not a generic queue. Conflating it with general async would lose the per-event audit trail in `payment_webhooks`.
- Laravel queue = generic async for things that aren't email and aren't webhooks (PDF generation, certificate issuance side-effects, etc.). Currently zero use.

### Where new async work goes
- **Email** (transactional or marketing) → outbox.
- **Webhook re-dispatch** → `result='queued'` + drain.
- **Anything else** → wait for Redis worker. Don't introduce a fourth mechanism.

### What to never do
- Don't add a fourth poll-based table for "background things in general." Either it's email (outbox), webhook (payment_webhooks), or it can wait for Redis.
- Don't make the outbox drain dispatch directly via `Mail::send()` from inside the drain — that's circular. The drain talks to the Resend driver directly.
- Don't replay a `result='failed'` row by changing it back to `queued` from code. The admin Replay action is the only sanctioned path so the audit-log row exists.

---

## Decision references

- Phase A commit `7d94788` — admin panel CRUD + dashboard widgets.
- Phase B commit `3d92127` — refund + invoice + 2FA + audit + rate limit.
- B-Hardening commit `317125f` — replay idempotency, race-test harness, RBAC, TRUNCATE block, ZATCA doc, MFA setup page.
- Phase C part 1 commit `4ff5efb` — `webhooks:replay-queued` drain loop.

---

## Open questions for Phase C part 2

1. Should `idempotency_key` on email_outbox be derived from `(template_slug, recipient_email, key_namespace)` or always caller-supplied? — caller-supplied gives the most control but is easy to forget.
2. Should the queue worker share a process pool with cron, or be separate? — depends on hosting (PaaS vs. VM).
3. Should `webhooks:replay-queued` cap retries (e.g. mark `permanently_failed` after N drain attempts)? — currently a `failed` row would be re-queued by the admin again; no auto-escalation.

Answer these before writing Phase C part 2 code.
