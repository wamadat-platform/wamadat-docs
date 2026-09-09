# منصة ومضات التعليمية — سجل القرارات
## PRODUCT & TECHNICAL DECISION LOG

**الإصدار:** 1.0  
**الحالة:** `LIVING REGISTER`  
**الغرض:** حفظ «لماذا قررنا» وليس فقط «ماذا فعلنا».  

---

# 1. متى نسجل قرارًا؟

يسجل القرار إذا كان:

- يغير Product scope.
- يغير priority/roadmap بصورة جوهرية.
- يفتح/يغلق Deferred feature.
- يغير Role/permission model.
- يغير payment/business rule.
- يغير architecture/release/operations contract.
- يختار مزودًا خارجيًا مهمًا.
- يقبل Risk عاليًا.
- صعب الرجوع أو مكلفًا.

لا نسجل كل UI spacing أو اسم variable.

---

# 2. Decision Record Template

```text
Decision ID: DEC-XXX
Date:
Status: PROPOSED / ACCEPTED / SUPERSEDED / REVOKED
Owner:
Participants:

Context:
Decision:
Alternatives Considered:
Rationale:
Consequences:
Risks:
Follow-up:
Related Backlog/Release/Documents:
Supersedes:
```

---

# 3. Initial Decisions inherited into Governance Baseline

## DEC-001 — Launch V1 is a focused product scope, not every source module

**Date:** before/through V1, reaffirmed 2026-09-09  
**Status:** `ACCEPTED`  

**Decision:** يعتبر V1 الرحلة التعليمية والتجارية الأساسية فقط؛ وجود كود لميزة أخرى لا يدخلها في المنتج المنشور.

**Rationale:** تقليل سطح الاختبار والمخاطر وإطلاق رحلة متكاملة.

**References:** Frozen V1 Features + Deferred Registry.

---

## DEC-002 — Single-Wamadat user experience remains the published V1 boundary

**Status:** `ACCEPTED`  

**Decision:** البنية التقنية قد تحتوي Multi-Tenancy، لكن SaaS/Multi-Academy UI ليست ضمن V1 وتظل محجوبة حتى قرار تجاري جديد.

**Consequence:** أي عودة لـSaaS تعد Strategic Initiative لا Toggle بسيط.

---

## DEC-003 — Deferred features require scope decision before re-enable

**Status:** `ACCEPTED`  

**Decision:** لا يكفي رفع `FEATURE_X`; يجب المرور عبر Change/Ready/UAT/Release وتحديث Deferred Registry.

---

## DEC-004 — V1.0.0 becomes the frozen closure baseline on 2026-09-09

**Status:** `ACCEPTED`  

**Decision:** يفصل هذا التاريخ بين Launch/Stabilization وبين managed V1.x development.

**Consequence:** New work after the boundary receives backlog/release identity.

---

## DEC-005 — Roadmap after V1 is Now/Next/Later and evidence-driven

**Status:** `ACCEPTED`  

**Decision:** لا يعتبر الترتيب الزمني في Roadmap القديمة التزامًا تلقائيًا. Candidates يعاد ترتيبهم بالبيانات، الأهداف، المخاطر، الجهد والاعتماديات.

---

## DEC-006 — P0–P3 remains the severity/priority language

**Status:** `ACCEPTED`  

**Decision:** الاحتفاظ بتصنيف P0/P1/P2/P3 مع إضافة RICE-lite للميزات غير العاجلة.

**Consequence:** P0 لا ينتظر scoring.

---

## DEC-007 — Short controlled releases over large batch releases

**Status:** `ACCEPTED`  

**Decision:** استخدام V1.0.x/V1.x releases صغيرة، مع Release Gate ومراقبة، بدل تجميع أشهر من Features في نشر واحد.

---

## DEC-008 — Code complete is not Done

**Status:** `ACCEPTED`  

**Decision:** Definition of Done تشمل permissions/validation/states/regression/UAT/config/monitoring/rollback awareness حسب Risk.

---

## DEC-009 — Reliability can override feature velocity

**Status:** `ACCEPTED`  

**Decision:** عند P0 أو تدهور Core Journey/critical reliability، يمكن تعليق non-critical feature delivery لصالح الاستقرار.

---

## DEC-010 — Deployment identity must be explicit

**Status:** `ACCEPTED`  

**Decision:** كل Production release يملك Release ID ومرجع build/SHA/images وفق Technical Operations contract؛ لا يستخدم «آخر نسخة» كهوية نشر.

---

## DEC-011 — Product metrics need explicit definitions before target commitments

**Status:** `ACCEPTED`  

**Decision:** نجمع Baseline موثوق أولًا، ثم نعتمد Targets. لا نضع نسب نجاح اعتباطية داخل Roadmap.

---

## DEC-012 — One backlog is the execution entry point

**Status:** `ACCEPTED`  

**Decision:** الطلب قد يأتي من أي قناة، لكنه لا يدخل التنفيذ دون Backlog/Incident ID.

---

# 4. Decision Index

| ID | القرار | الحالة | آخر مراجعة |
|---|---|---|---|
| DEC-001 | Focused V1 scope | ACCEPTED | 2026-09-09 |
| DEC-002 | Single-Wamadat product boundary | ACCEPTED | 2026-09-09 |
| DEC-003 | Controlled deferred re-enable | ACCEPTED | 2026-09-09 |
| DEC-004 | Freeze V1.0.0 closure baseline | ACCEPTED | 2026-09-09 |
| DEC-005 | Now/Next/Later roadmap | ACCEPTED | 2026-09-09 |
| DEC-006 | P0–P3 + RICE-lite | ACCEPTED | 2026-09-09 |
| DEC-007 | Small controlled releases | ACCEPTED | 2026-09-09 |
| DEC-008 | Full Definition of Done | ACCEPTED | 2026-09-09 |
| DEC-009 | Reliability override | ACCEPTED | 2026-09-09 |
| DEC-010 | Explicit release identity | ACCEPTED | 2026-09-09 |
| DEC-011 | Metric baseline before targets | ACCEPTED | 2026-09-09 |
| DEC-012 | Single backlog entry | ACCEPTED | 2026-09-09 |

---

# 5. How to supersede a decision

لا تحذف القرار القديم. أنشئ قرارًا جديدًا:

```text
DEC-0XX — New Decision
Status: ACCEPTED
Supersedes: DEC-00Y
```

ثم غيّر القديم إلى `SUPERSEDED` مع رابط الجديد.

---

# 6. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | إنشاء سجل القرارات وإدخال القرارات الموروثة/المعتمدة عند إغلاق V1. |
