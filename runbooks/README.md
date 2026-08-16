# Operational Runbooks

One-page playbooks for the failure modes that an operator can hit during the
first 30 days post-launch. Each runbook is structured the same way:

1. **Symptoms** — how you notice the problem
2. **Verify** — fast commands to confirm the diagnosis
3. **Mitigate** — what to do right now to stop bleeding
4. **Recover** — bring the system back to green
5. **Post-incident** — what to write in the audit log + Sentry

| Failure mode | Runbook |
|---|---|
| Postgres unreachable / slow | [db-down.md](./db-down.md) |
| Redis unreachable | [redis-down.md](./redis-down.md) |
| Email delivery stalled | [email-stalled.md](./email-stalled.md) |
| Payment webhook signature mismatch | [webhook-mismatch.md](./webhook-mismatch.md) |

These map directly to the 6 health probes in `HealthStatusWidget` and the
ops Filament resources added in P4.3.
