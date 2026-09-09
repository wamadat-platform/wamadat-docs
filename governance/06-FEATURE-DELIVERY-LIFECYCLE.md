# منصة ومضات التعليمية — دورة حياة تسليم الميزة
## FEATURE DELIVERY LIFECYCLE

**الإصدار:** 1.0  
**الحالة:** `LIVING DELIVERY STANDARD`  

---

# 1. الهدف

تحديد متى تكون Feature مجرد فكرة، ومتى تصبح جاهزة للتطوير، ومتى تعتبر منتهية فعليًا.

```text
Idea
→ Discovery
→ Feature Brief
→ Ready Gate
→ Design/Technical Review
→ Implementation
→ Verification
→ UAT
→ Release Gate
→ Production
→ Measure
```

---

# 2. Stage 0 — Discovery

تستخدم عندما يكون الحل أو القيمة غير واضحة.

الناتج ليس كودًا بالضرورة، بل إجابة عن:

- المشكلة.
- المستخدم.
- Evidence.
- الخيارات.
- المخاطر.
- التوصية.
- هل نبني أم لا؟

يمكن إغلاق Discovery بـ`DO NOT BUILD` وهذا يعد نتيجة صحيحة.

---

# 3. Stage 1 — Feature Brief / PRD-lite

لفريق صغير لا نحتاج PRD من 30 صفحة. يكفي:

```text
Feature ID / Initiative:
Problem:
Outcome:
Target Users/Roles:
In Scope:
Out of Scope:
User Flow:
Business Rules:
Permissions:
Edge Cases:
Acceptance Criteria:
Success Metric:
Dependencies:
Release Strategy:
```

للـL/XL يضاف Technical Design/RFC حسب الحاجة.

---

# 4. Definition of Ready

لا يبدأ Coding حتى تتحقق العناصر المناسبة لحجم Feature:

- [ ] Backlog ID موجود.
- [ ] Problem Statement واضح.
- [ ] Owner محدد.
- [ ] Target users/roles محددة.
- [ ] Scope محدد.
- [ ] Out-of-scope مكتوب.
- [ ] Business rules محسومة.
- [ ] Acceptance Criteria قابلة للاختبار.
- [ ] UX flow/states محسومة عند الحاجة.
- [ ] API/data impact معروف.
- [ ] Permissions/security impact مراجع.
- [ ] Finance/payment impact مراجع إن وجد.
- [ ] External dependency مذكور.
- [ ] Migration/backfill impact مذكور.
- [ ] Success Metric معروف.
- [ ] Rough effort مقبول.
- [ ] Target release/cycle معروف.
- [ ] Deferred feature gate plan محدد إن كانت الميزة مؤجلة.

إذا كان عنصر جوهري غير محسوم: `NOT READY`.

---

# 5. UX/Design Readiness

أي Feature لها UI يجب أن تغطي قبل/أثناء التنفيذ:

- Default state.
- Loading.
- Empty.
- Error.
- Validation.
- Success feedback.
- Permission-denied behavior.
- Responsive.
- RTL/Arabic behavior.
- Accessibility الأساسية ذات الصلة.

لا تقبل شاشة Happy Path فقط.

---

# 6. Technical Review

المراجعة تناسب المخاطر ولا تتحول إلى بيروقراطية.

أسئلة أساسية:

- هل نعيد استخدام Contract قائم أم نخلق واحدًا جديدًا؟
- هل يوجد Data migration؟
- هل العملية idempotent حيث يلزم؟
- هل نحتاج queue/scheduler/webhook؟
- هل permissions server-side؟
- هل يوجد PII/financial data؟
- هل نحتاج feature flag؟
- هل التغيير backward compatible؟
- كيف نرصد الخطأ؟
- كيف نتراجع؟

يحتاج ADR/RFC عندما يكون القرار صعب الرجوع أو يؤثر على عدة مكونات.

---

# 7. Implementation Standard

أثناء التنفيذ:

1. التزام Coding Standards الموجودة.
2. لا توسيع Scope أثناء البرمجة دون Change Note.
3. حفظ contracts بين Backend/Web متزامنة.
4. عدم تفعيل Deferred surface غير مقصودة.
5. عدم وضع أسرار أو environment-specific values في Git.
6. إضافة logging/observability عندما يكون الفشل غير مرئي للمستخدم أو عالي الأثر.

---

# 8. Verification

يطبق حسب Feature:

- Functional verification.
- Negative cases.
- Permissions/isolation.
- Validation.
- Regression على الرحلات الأساسية.
- Responsive/RTL.
- Build/lint/static checks المناسبة.
- Migration/schema verification إذا وجدت.
- External integration sandbox/staging verification إذا وجدت.

الـUAT اليدوي يستفيد من `../qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md` للرحلات التي قد تتأثر.

---

# 9. Definition of Done

Feature لا تعتبر Done إلا عند:

- [ ] تحقق الهدف/السلوك المعتمد.
- [ ] تعمل فقط للأدوار الصحيحة.
- [ ] الحالات الأساسية والسلبية المهمة مغطاة.
- [ ] لا Regression معروف على core journeys.
- [ ] UI/UX states مكتملة.
- [ ] Production configuration موثقة.
- [ ] Monitoring/logging موجود حيث يلزم.
- [ ] Migration/backfill مكتمل أو له plan واضح.
- [ ] Release notes entry جاهز.
- [ ] Rollback/forward-fix awareness موجود.
- [ ] UAT/acceptance completed حسب الحجم.
- [ ] Release ID محدد.

`CODE COMPLETE` حالة تسبق `DONE`.

---

# 10. Release Gate

يحتفظ المشروع بالـ12 عناصر التي عرّفتها Roadmap السابقة:

1. Functional implementation.
2. Permissions review.
3. Validation.
4. Error handling.
5. Empty/loading states.
6. Responsive/RTL.
7. API contract check.
8. Regression tests.
9. Manual UAT.
10. Production configuration.
11. Monitoring/logging where required.
12. Rollback awareness.

وتُطبق بعمق يتناسب مع Risk.

---

# 11. Post-Release

خلال نافذة المراقبة المناسبة:

- راقب Errors/Jobs/Webhooks ذات الصلة.
- راقب KPI المستهدف.
- راقب Support feedback.
- تأكد أن Feature flag/scope صحيح.
- قرر `KEEP / ITERATE / ROLLBACK / DISABLE`.

Feature بلا قياس ليست نهاية دورة المنتج.

---

# 12. Feature Brief Template

```text
# <Feature Name>
ID:
Owner:
Target Release:

## Problem

## Outcome / Success Metric

## Users / Roles

## In Scope

## Out of Scope

## User Flow

## Business Rules

## Permissions

## Edge Cases

## Acceptance Criteria
- Given ... When ... Then ...

## Data / API Impact

## External Dependencies

## Analytics / Events

## Rollout / Flag

## Rollback / Disable
```

---

# 13. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | اعتماد Ready/Done/Release lifecycle موحد لما بعد V1. |
