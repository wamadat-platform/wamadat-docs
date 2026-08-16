# 07 — Deployment

> **استراتيجية النشر والـ CI/CD.** يَحكم كيف ينتقل الكود من laptop المطوّر إلى production.

---

## 📑 الفهرس

1. [Environments](#1-environments)
2. [Hosting Topology](#2-hosting-topology)
3. [CI/CD Pipeline](#3-cicd-pipeline)
4. [Branching Strategy](#4-branching-strategy)
5. [Database Migrations](#5-database-migrations)
6. [Zero-Downtime Deploys](#6-zero-downtime-deploys)
7. [Rollback Strategy](#7-rollback-strategy)
8. [Monitoring & Alerts](#8-monitoring--alerts)
9. [Backups](#9-backups)
10. [Disaster Recovery](#10-disaster-recovery)

---

## 1. Environments

| Env | Purpose | Data | URL |
|---|---|---|---|
| **local** | تطوير | seeded test data | `localhost:3000`, `localhost:8000` |
| **preview** | كلّ PR (Vercel) | anonymized snapshot من staging | `pr-{n}.wamadat.dev` |
| **staging** | QA + integration testing | anonymized snapshot من prod (weekly) | `staging.wamadat.dev` |
| **production** | المستخدمون | real | `wamadat.io`, `*.wamadat.io` |

### Parity
- نفس الـ stack versions عبر كلّ environments.
- نفس الـ secrets keys (مع قيم مختلفة).
- نفس الـ Bunny config (مع zones مختلفة).

---

## 2. Hosting Topology

```
                       ┌────────────────┐
                       │  Vercel Edge   │  Frontend (Next.js)
                       │  Global CDN    │
                       └───────┬────────┘
                               │
                               ▼
            ┌──────────────────────────────────────┐
            │  Railway (KSA region preferred)       │
            │                                       │
            │  ┌────────────┐  ┌────────────┐      │
            │  │  Laravel   │  │  Laravel   │      │
            │  │  Octane    │  │  Octane    │ ...  │
            │  │  (FPM 1)   │  │  (FPM 2)   │      │
            │  └─────┬──────┘  └─────┬──────┘      │
            │        └───────┬───────┘             │
            │                │                     │
            │  ┌─────────────▼─────────────┐       │
            │  │  Horizon (queue workers)   │       │
            │  └────────────────────────────┘       │
            │                                       │
            │  ┌────────────┐  ┌────────────┐      │
            │  │  AI svc    │  │  Reverb    │      │
            │  │  FastAPI   │  │  WebSocket │      │
            │  └────────────┘  └────────────┘      │
            └───────────┬──────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   ┌────────┐     ┌──────────┐    ┌──────────────┐
   │Postgres│     │ Redis 7  │    │ Meilisearch  │
   │16+pgvec│     │ Cluster  │    │              │
   └────────┘     └──────────┘    └──────────────┘
   (Railway        (Railway        (Railway
    managed)        managed)        managed/self)
```

### Bunny.net (External CDN/Storage)
- **CDN**: PoPs في الرياض، جدّة، الدمام، دبي.
- **Storage Zones**: KSA primary، replica في الإمارات.
- **Stream**: video transcoding + delivery.

### KSA Data Residency
- Tenant flagged `data_residency='sa'` → DB row stays in KSA Railway region.
- Bunny Storage zone = KSA.
- لا cross-border transfer إلا للـ AI calls (مع DPA).

---

## 3. CI/CD Pipeline

### Tools
- **GitHub Actions** للـ CI.
- **Vercel** للـ frontend deploy (auto).
- **Railway** للـ backend deploy.

### Pipeline Stages

```
┌─────────────────────────────────────────────────────────┐
│ 1. On PR open / push to branch                          │
├─────────────────────────────────────────────────────────┤
│  ✓ Install (composer + npm)                              │
│  ✓ Lint (Pint, PHPStan level 8, ESLint, TypeScript tsc) │
│  ✓ Test (Pest unit + feature, Playwright e2e)            │
│  ✓ Security scan (composer audit, npm audit, Semgrep)   │
│  ✓ Build (Vite, Next.js build)                          │
│  ✓ Deploy Preview (Vercel) + Backend ephemeral on Railway│
├─────────────────────────────────────────────────────────┤
│ 2. On merge to main                                      │
├─────────────────────────────────────────────────────────┤
│  ✓ All above +                                           │
│  ✓ Deploy to Staging (auto)                              │
│  ✓ Run smoke tests on staging                            │
│  ✓ Wait for manual approval                              │
│  ✓ Deploy to Production (canary 10% → 50% → 100%)        │
│  ✓ Run smoke tests on production                         │
│  ✓ Notify Slack/email                                    │
└─────────────────────────────────────────────────────────┘
```

### CI Time Budget
- Lint + Tests: < 5 دقائق.
- Full pipeline: < 12 دقائق.
- إذا تجاوز → investigate (parallel jobs, caching).

---

## 4. Branching Strategy

### Trunk-based Development (modified)

```
main ──●──●──●──●──●──●──●──●──● → production
       │     │           │
       ▼     ▼           ▼
   feature  feature   hotfix
```

### قواعد
- `main` = production-deployable دائماً.
- Feature branches قصيرة العمر (< 3 أيام).
- PR لكل feature، يَجب على code review + green CI قبل merge.
- Squash merge افتراضياً (commit history نظيف).

### Releases
- Tag على main: `v1.2.3` (SemVer).
- Auto-generated CHANGELOG من Conventional Commits.

---

## 5. Database Migrations

### Tooling
- Laravel migrations + `doctrine/dbal` للـ alters.
- **stancl/tenancy** أو **spatie/laravel-multitenancy** تَدير tenant schema migrations.

### Migration Rules
- **Backward compatible** دائماً (additive changes).
- **Never drop columns** في نفس deploy كما يُضاف استخدام جديد.
- 2-phase pattern للـ breaking changes:
  1. **Deploy A**: أضِف عمود جديد، احتفظ بالقديم.
  2. **Backfill**: نسخ البيانات.
  3. **Deploy B**: استخدم العمود الجديد.
  4. **Cleanup deploy**: احذف العمود القديم بعد week.

### Tenant Migrations
- Migrations في `database/migrations/tenant/` تُطبَّق على كلّ tenant schema.
- Migrations في `database/migrations/landlord/` تُطبَّق مرّة واحدة.

### Performance
- Migrations الكبيرة (> 1M rows) تُستخدم `pg_partman` + concurrent indexes.
- لا locks طويلة في production (`SET lock_timeout`).

---

## 6. Zero-Downtime Deploys

### Backend (Laravel)
- **Laravel Octane** يبقى يَستجيب أثناء الـ deploy (rolling restart).
- New instance يَبدأ → health check → old instance يَخرج من الـ load balancer.
- Connection draining: 60 ثانية.

### Frontend (Vercel)
- Vercel atomic deploys: URL لكلّ deploy.
- promote النسخة الجديدة → traffic switch فوريّ.
- Rollback = reactivate URL سابق.

### Canary Deploys (production)
```
[New Build] → 10% traffic for 10 min →
              if error_rate < threshold →
                50% traffic for 10 min →
                  100% traffic
              else → auto-rollback
```

---

## 7. Rollback Strategy

### Triggers
- Error rate > 5% خلال 5 دقائق.
- p95 latency > 2× baseline.
- Manual: على-call engineer initiates.

### Process
1. Vercel: 1 click → revert to previous deploy.
2. Railway: rollback to previous image tag.
3. Database: forward-only migrations — لا rollback DB.
   - إذا migration كانت breaking → forward fix (لا تَعكِس).
4. Notify Slack + status page.

### Code Rollback ≠ Data Rollback
- DB schema migrations لا تُعكَس في prod.
- إذا feature flagged → toggle off بدلاً من rollback كامل.

---

## 8. Monitoring & Alerts

### Tools
- **Sentry**: errors (BE + FE) + performance.
- **Better Stack**: logs aggregation + uptime monitoring + status page.
- **Vercel Analytics**: Core Web Vitals.
- **Railway Metrics**: CPU/RAM/Network per service.

### What we monitor

| Metric | Threshold (warn) | Threshold (page) |
|---|---|---|
| API p95 latency | 500ms | 1s |
| API error rate | 1% | 5% |
| DB connection pool usage | 70% | 90% |
| Redis memory | 70% | 90% |
| Queue depth | 1k | 5k |
| Failed jobs/min | 10 | 50 |
| Payment webhook failures | 5/min | 20/min |
| Disk usage | 70% | 90% |

### Alert Channels
- Slack: #wamadat-alerts (warn).
- PagerDuty: على-call rotation (page).
- Email: digest يومي للـ status.

---

## 9. Backups

### Database
- **Frequency**: PITR (point-in-time recovery) — Railway managed، 7-day retention.
- **Snapshots**: daily، 30-day retention.
- **Long-term**: weekly snapshot → S3-compatible cold storage، 12-month retention.

### Bunny Storage
- Versioning: enabled على critical paths (certificates, ZATCA XMLs).
- Cross-zone replica: KSA → UAE.

### Redis
- AOF append-only file persisted.
- لا backup طويل المدى (الـ data ephemeral by nature).

### Restore Drills
- **Quarterly**: restore from backup إلى staging، verify integrity.
- **Yearly**: full DR drill — simulated KSA region failure.

---

## 10. Disaster Recovery

### RTO / RPO

| Service | RTO (Recovery Time) | RPO (Recovery Point) |
|---|---|---|
| Database | 4 hours | 5 minutes (PITR) |
| Application | 30 minutes | n/a (stateless) |
| File storage | 8 hours (failover to replica) | 1 hour |
| Search index | 2 hours (rebuild from DB) | acceptable to be empty momentarily |

### Failure Scenarios

| Scenario | Mitigation |
|---|---|
| KSA region down | Failover to UAE region (Phase 3) |
| Postgres primary fails | Auto-failover to replica (Railway) |
| Redis fails | Sessions lost, app degrades gracefully |
| Bunny CDN fails | Direct origin fallback |
| Tap down | Failover to alternative (manual) |
| AI service down | Fallback to "AI temporarily unavailable" UI |

### Communication
- Status page updates: real-time.
- Tenant email للـ degraded service > 30 min.
- Post-mortem يُنشَر علناً للـ Sev-1/2.

---

## ADRs الجديدة

| # | القرار |
|---|---|
| **ADR-023** | Trunk-based با feature branches قصيرة + squash merge |
| **ADR-024** | Canary deploys 10→50→100 مع auto-rollback |
| **ADR-025** | Migrations forward-only في production (no DB rollback) |
| **ADR-026** | PITR daily snapshots + weekly cold storage + quarterly restore drills |

---

<sub>**النسخة**: 1.0 · **CI tooling**: GitHub Actions · **Hosting**: Vercel (FE) + Railway (BE) + Bunny (CDN/Stream)</sub>
