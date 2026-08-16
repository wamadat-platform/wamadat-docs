# 🛠️ P4.3 — Operational Filament Resources

**Date:** 2026-05-13
**Scope:** A small operational control room inside the existing `/admin` Filament panel. Five surfaces — failed jobs, outbox, webhooks, queue depth, health — each scoped to the current tenant. Every state-changing action audits to `audit_logs`.

> Not a dashboard. A control room. Read-mostly, surgical retry/replay actions, no creation forms.

---

## 1️⃣ What shipped

### Resources (CRUD-mostly read, with surgical actions)

| Resource | Route | Source | Actions |
|---|---|---|---|
| **Failed Jobs** | `/admin/failed-jobs` | `landlord.failed_jobs` (new table) | view (full payload + exception) · retry one · retry many (bulk) · delete |
| **Email Outbox** | `/admin/email-outboxes` | `tenant.email_outbox` | view (recipient + subject + body + error) · retry (re-queue with attempts=0) |
| **Payment Webhooks** | `/admin/payment-webhooks` | `tenant.payment_webhooks` | view (raw_payload + signature) · replay (set result=queued) |

### Widgets (admin home stats)

| Widget | Probes | Refresh |
|---|---|---|
| **HealthStatusWidget** | landlord_db · tenant_db · cache · resend_configured · scheduler · webhooks | 60s |
| **QueueDepthWidget** | pending queue depth (Redis-aware, multi-queue) · failed jobs total · outbox queued/failed · last-sent timestamp | 30s |

### Audit linkage

Every retry / delete / replay action records to the per-tenant `audit_logs` table via the new `Operations\Application\Services\AdminAuditWriter`:

| Action | Audit entry |
|---|---|
| Retry failed job (single) | `ops.failed_job.retried` |
| Retry failed jobs (bulk) | `ops.failed_job.retried_bulk` |
| Delete failed job | `ops.failed_job.deleted` |
| Retry outbox email | `ops.outbox.retried` |
| Replay webhook | `ops.webhook.replayed` |

Each entry captures `actor_user_id` (current admin), `subject_type` + `subject_id` (the record acted on), `before_state` / `after_state` (for retry/replay where status flips), and `context` (gateway, queue, recipient, etc.). The `audit_logs` table has an append-only DB trigger, so even an admin with full Filament access can't erase the trail.

---

## 2️⃣ Tenant scoping (safety)

The Filament admin panel authenticates against tenant users (per `AdminPanelProvider::authGuard('web')`), but the queue and `failed_jobs` table are **shared across tenants** (workers don't have a tenant context at failure time).

**Scoping strategy:**
- `FailedJobResource::getEloquentQuery()` filters by `payload LIKE '%"tenantId":"<current-tenant-id>"%'` — leveraging the `tenantId` field Spatie Multitenancy stamps onto every queued job. A tenant admin sees only their own failures.
- `EmailOutboxResource` / `PaymentWebhookResource` live in tenant-scoped tables already (via `UsesTenantConnection`).

---

## 3️⃣ Files created

```
database/migrations/landlord/2026_05_13_110000_create_failed_jobs_table.php

app/Modules/Operations/
├── Application/
│   └── Services/
│       └── AdminAuditWriter.php
└── Infrastructure/
    ├── Eloquent/
    │   └── FailedJobModel.php
    └── Filament/
        ├── Resources/
        │   ├── FailedJobResource.php
        │   ├── FailedJobResource/Pages/{ListFailedJobs,ViewFailedJob}.php
        │   ├── EmailOutboxResource.php
        │   ├── EmailOutboxResource/Pages/{ListEmailOutbox,ViewEmailOutbox}.php
        │   ├── PaymentWebhookResource.php
        │   └── PaymentWebhookResource/Pages/{ListPaymentWebhooks,ViewPaymentWebhook}.php
        └── Widgets/
            ├── QueueDepthWidget.php
            └── HealthStatusWidget.php
```

## 4️⃣ Files updated

`app/Providers/Filament/AdminPanelProvider.php` — registered 3 new resources + 2 new widgets under the existing tenant admin panel.

---

## 5️⃣ Verification

| Step | Result |
|---|---|
| PHP lint on 15 changed/new files | **CLEAN** |
| Landlord migration `create_failed_jobs_table` | **DONE** in 22ms |
| Filament route registration | **6 new routes** registered (3 list + 3 view) |
| Full backend test suite | **125 passed, 1 risky, 8 skipped** in standard `tests/` directory. Same 14 module-tests under `app/Modules/Tenancy/Tests/*` still failing — separate harness issue tracked as P4.2.A |

---

## 6️⃣ Operator workflow examples

**"A job died at 02:37 — what was it?"**
1. Open `/admin` → see the **Failed Jobs** nav badge showing `1` in red
2. Click → list shows job class, queue, one-line exception
3. Click into the row → full exception stack + pretty-printed payload
4. If fixable → click **إعادة** → confirmation modal → `Artisan::call('queue:retry', ...)` → audit logged → notification

**"Did Mishal get his welcome email?"**
1. Open `/admin` → **صُندوق البَريد الصادر**
2. Filter "فاشِل / مُرتَدّ فقط" → search his email
3. View row → see error_message (e.g., "550 mailbox full")
4. Click **إعادة الإرسال** → status flips to `queued`, attempts=0 → next `outbox:drain` picks it up → audit logged

**"Was our Tap webhook actually rejected, or did we lose it?"**
1. Open `/admin` → **سِجِلّ الـ webhooks** → filter by gateway=tap, last 24h
2. Find the event → status badge tells you (`failed` / `invalid_signature` / `duplicate`)
3. View row → raw_payload is fully visible
4. If signature was a transient issue → click **إعادة المُعالجة** → audit logged

**"Is the platform okay right now?"**
1. Open `/admin` home → 4 widgets visible
2. **HealthStatusWidget** — green/red dots for landlord_db, tenant_db, cache, resend, scheduler, webhooks
3. **QueueDepthWidget** — pending count + failed count + last-sent age

---

## 7️⃣ Intentional NON-goals

- **No charts.** The widgets are stat-strips, not time-series graphs. A chart-heavy ops dashboard is Phase 5 (Pulse / Telescope integration).
- **No Horizon.** The user explicitly asked to defer Horizon until we understand query patterns (delivered in P4.1).
- **No log explorer.** Sentry already aggregates exceptions; that integration is verified in **P4.4** next.
- **No "drop everything" / "purge queue" buttons.** Destructive bulk operations deliberately omitted — anyone needing those uses `php artisan queue:flush` from the CLI with operator-level access.

---

## ✅ P4.3 closure

- 3 resources, 2 widgets, 1 helper service — wired into the existing admin panel
- Tenant-scoped (no data leak across tenants)
- Audit-linked (append-only, signed by `actor_user_id`)
- 30s polling on action surfaces (failed jobs / outbox / webhooks) for live ops sessions
- 60s polling on the health widget (lower-frequency, more expensive probes)

Ready for **P4.4 — Sentry Verify** (test exception → context → user_id + tenant_id → release → environment).
