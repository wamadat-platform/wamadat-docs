# منصة ومضات التعليمية — سجل الدين التقني والمخاطر
## TECHNICAL DEBT & RISK REGISTER

**الإصدار:** 1.0  
**الحالة:** `LIVING REGISTER`  
**Review:** Monthly + before major phase  

---

# 1. الهدف

منع أن تستهلك Features كل القدرة بينما تتراكم مخاطر غير مرئية في:

- Architecture.
- Security.
- Data.
- Payments.
- Operations.
- Dependencies.
- Observability.
- Deferred code.

الدين التقني ليس «تنظيف كود» فقط؛ هو أي قرار مؤجل يرفع تكلفة/خطر التغيير مستقبلاً.

---

# 2. الفرق بين Debt وRisk وIssue

| النوع | المعنى |
|---|---|
| `TECH_DEBT` | تكلفة مستقبلية بسبب تصميم/تعقيد/تأجيل |
| `RISK` | حدث محتمل قد يسبب أثرًا |
| `ISSUE` | مشكلة حدثت بالفعل وتحتاج معالجة |
| `INCIDENT` | Issue إنتاجية ذات أثر يحتاج استجابة منظمة |

---

# 3. Risk Scoring

## Likelihood

1 = Rare  
2 = Unlikely  
3 = Possible  
4 = Likely  
5 = Almost certain

## Impact

1 = Minor  
2 = Moderate  
3 = Major  
4 = Severe  
5 = Critical

```text
Risk Score = Likelihood × Impact
```

التصنيف الإرشادي:

- 1–4 Low
- 5–9 Medium
- 10–16 High
- 17–25 Critical

Security/financial/data risks قد تصعد يدويًا بغض النظر عن الحساب.

---

# 4. Risk Register Contract

```yaml
id: RISK-XXX
title:
category:
description:
trigger:
likelihood:
impact:
score:
owner:
mitigation:
contingency:
status:
target_review:
links:
```

---

# 5. Technical Debt Contract

```yaml
id: DEBT-XXX
title:
area:
why_it_exists:
current_cost_or_risk:
what_breaks_if_ignored:
recommended_action:
rough_effort:
priority:
owner:
target_window:
status:
```

---

# 6. Initial Governance Risk Register

العناصر التالية مشتقة من حدود ومخاطر موثقة في Baseline الحالية؛ الأرقام الأولية تحتاج مراجعة Owner ولا تعد incident assertion.

| ID | Risk | Category | Initial Level | Mitigation |
|---|---|---|---|---|
| RISK-001 | Scope creep بعد الاستقرار | Product | High | Backlog/Change Process + WIP limits |
| RISK-002 | تفعيل Feature مؤجلة دون full surface review | Release/Security | High | Deferred re-enable procedure + Release Gate |
| RISK-003 | اعتماد زائد على بوابات/بريد/مزودين خارجيين | External | High | Provider health, fallbacks where valid, config/contract tests |
| RISK-004 | خلل payment/webhook يسبب أثرًا ماليًا أو enrollment خاطئًا | Finance | Critical | idempotency, reconcile, monitoring, focused regression |
| RISK-005 | توسع Product كبير متوازٍ يفوق قدرة الفريق | Delivery | High | One/two strategic tracks max |
| RISK-006 | تضخم operations بسبب Community/Marketplace/Consultations | Operations | High | operational owner/model before release |
| RISK-007 | التباس منتج Wamadat أحادي الأكاديمية مع tenant-aware technical foundation الموروثة | Architecture/Product | Medium | تثبيت Single-Academy boundary؛ منع ظهور أي Tenant/Admin-academy surfaces؛ تقييم تبسيط البنية لاحقًا كTechnical Debt مستقل |
| RISK-008 | Backup موجود نظريًا دون restore confidence كافٍ | DR | High | scheduled restore drill + backup monitoring |
| RISK-009 | KPI غير موحدة تؤدي لقرارات خاطئة | Product/Data | Medium | metric contracts + single dashboard definitions |
| RISK-010 | Feature velocity تسبق reliability | Operations/Product | High | Error-budget-lite guardrail |

---

# 7. Areas التي تراجع دوريًا

- Authentication/session/access control.
- Financial workflows.
- Queue/scheduler/webhooks.
- Database migrations/indexes/query performance.
- Storage permissions/retention.
- Email deliverability.
- External API versions/deprecations.
- Frontend/backend contract drift.
- Deferred feature code freshness.
- Dependencies/security updates.
- Backup/restore/PITR readiness.
- Observability gaps.

---

# 8. Debt Budget

لا يحدد رقمًا ثابتًا عالميًا. يقرر كل Cycle نسبة Capacity بناءً على Health.

قاعدة مقترحة:

- إذا Health طبيعي: احجز Capacity منتظمة للدين ذي المخاطر.
- إذا debt يسبب P1/P0 أو يبطئ كل Feature: يصعد إلى NOW.
- لا تؤجل Security/financial/data integrity debt لأنه «ليس Feature».

---

# 9. دخول Debt إلى Roadmap

Debt يدخل Product/Delivery Roadmap إذا:

- يخفض reliability.
- يمنع Initiative مهمة.
- يرفع تكلفة التطوير بشكل متكرر.
- له deadline/deprecation.
- يحمل Security/Data/Financial risk.

ويجب شرح أثره بلغة Business، لا فقط «refactor code».

مثال:

> «تقليل احتمال ازدواجية/تعطل المعالجة المالية عند retry» أفضل من «تنظيف PaymentService».

---

# 10. Review Process

شهريًا:

1. New risks/debt.
2. Changed score.
3. Risks بدون Owner.
4. Mitigations overdue.
5. Incidents التي كشفت risk جديدًا.
6. Debt الذي ظهر في أكثر من Release.
7. ما يحتاج ADR/roadmap capacity.

قبل Phase كبيرة:

- Security.
- DR.
- external dependencies.
- data model.
- capacity.
- operational ownership.

---

# 11. Closure Rules

Risk يغلق فقط إذا:

- السبب أزيل، أو
- mitigation خفضه لمستوى مقبول موثق، أو
- Business Owner قبل الخطر صراحة مع review date.

Debt يغلق بعد تنفيذ العمل والتحقق من الأثر، لا بمجرد فتح PR.

---

# 12. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | إنشاء سجل مخاطر ودين تقني وربطه بالتطوير والإصدارات. |
