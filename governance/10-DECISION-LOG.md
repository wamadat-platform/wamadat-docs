# منصة ومضات التعليمية — سجل القرارات
## PRODUCT & TECHNICAL DECISION LOG

**الإصدار:** 1.1  
**آخر مراجعة:** 13 سبتمبر 2026  
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

**Status:** `SUPERSEDED`  
**Superseded by:** `DEC-013`

**Historical Decision:** كان القرار عند الإطلاق هو نشر تجربة Wamadat واحدة مع إبقاء احتمال التوسع المتعدد نظريًا. تم حسم هذا الاحتمال نهائيًا في DEC-013.

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


## DEC-013 — Wamadat is permanently a Single-Academy product

**Date:** 2026-09-13  
**Status:** `ACCEPTED`  
**Supersedes:** `DEC-002`

**Decision:** منتج ومضات مخصص لأكاديمية ومضات فقط. لا توجد خارطة منتج لإدارة أكاديميات متعددة، ولا Tenant Selector أو Academy creation/plans كاتجاه تجاري مستقبلي.

**Consequence:** أي tenant-aware infrastructure موجودة حاليًا تعامل كتفصيل تقني موروث. تبسيطها أو إزالتها مستقبلًا — إن تقرر — يكون Architecture/Technical-Debt initiative مستقلة، وليس Product Expansion.

---

## DEC-014 — Mobile App is an early post-V1 priority

**Date:** 2026-09-13  
**Status:** `ACCEPTED`

**Decision:** تطبيق Flutter لـAndroid/iOS ينتقل من Strategic/Late candidate إلى **Priority post-V1 initiative**.

**Consequence:** يبدأ Discovery وMVP scoping مبكرًا بعد V1 Closure، مع بقاء Definition of Ready وAPI readiness وcapacity/release decision إلزامية قبل التنفيذ.

---

## DEC-015 — Payment and marketing analytics integrations are part of V1 delivered baseline

**Date:** 2026-09-13  
**Status:** `ACCEPTED`

**Decision:** يعتبر V1 متضمنًا تكاملات Tap / Tabby / Tamara ضمن عقد الدفع، إضافة إلى GTM / GA4 / Meta Pixel / TikTok Pixel / Snapchat Pixel ضمن طبقة القياس والتسويق.

**Consequence:** تفعيل الحساب/المعرّف في Production حالة تشغيلية قابلة للتغيير، لكن التكامل نفسه ليس Feature مستقبلية أو خارج Baseline V1.

---

## DEC-016 — New Mobile App will be Flutter; legacy React Native/Expo is retired

**Date:** 2026-09-13  
**Status:** `ACCEPTED`

**Decision:** تطبيق ومضات الجديد لـAndroid/iOS يبنى بـFlutter من مشروع نظيف. مستودع React Native/Expo القديم لا يعاد استخدامه ولا يستمر كمسار تطوير.

**Consequence:** لا توجد Framework comparison ضمن WAM-0001. العمل المتبقي هو MVP/API readiness/architecture/effort/release planning. أي رجوع للكود القديم يكون مرجعًا سلوكيًا غير ملزم فقط.

---

## DEC-017 — Program Interest / Waitlist is part of V1 delivered baseline

**Date:** 2026-09-13  
**Status:** `ACCEPTED`

**Decision:** Program Interest / Waitlist قدرة مسلمة ومعتمدة ضمن `WAMADAT-V1.0.0` وليست V1.1 Candidate أو Deferred Feature.

**Consequence:** تزال من Deferred Registry وFuture Backlog، وتضاف إلى Frozen Functional Scope وProduct Baseline وUAT. أي kill-switch تقني لها يعد Operational Control لقدرة V1 وليس Scope Gate مستقبلية.

---

# 4. Decision Index

| ID | القرار | الحالة | آخر مراجعة |
|---|---|---|---|
| DEC-001 | Focused V1 scope | ACCEPTED | 2026-09-09 |
| DEC-002 | Single-Wamadat product boundary (historical) | SUPERSEDED | 2026-09-13 |
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
| DEC-013 | Permanent Single-Academy product | ACCEPTED | 2026-09-13 |
| DEC-014 | Mobile App early post-V1 priority | ACCEPTED | 2026-09-13 |
| DEC-015 | V1 payment + marketing integration baseline | ACCEPTED | 2026-09-13 |
| DEC-016 | Flutter clean mobile implementation; legacy RN/Expo retired | ACCEPTED | 2026-09-13 |
| DEC-017 | Program Interest / Waitlist is V1 delivered | ACCEPTED | 2026-09-13 |

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
| 2026-09-13 | 1.1 | حسم Single-Academy نهائيًا، رفع Mobile App كأولوية مبكرة، وتثبيت Payment/Marketing integrations ضمن V1. |
| 2026-09-13 | 1.2 | تثبيت Flutter كتقنية التطبيق الجديد وإيقاف React Native/Expo القديم، واعتماد Program Interest / Waitlist ضمن V1. |
