# منصة ومضات التعليمية — عملية إدارة طلبات التغيير
## CHANGE REQUEST PROCESS

**الإصدار:** 1.0  
**الحالة:** `LIVING POLICY`  
**النطاق:** Client requests, business rules, enhancements, integrations, re-enabling deferred features  

---

# 1. الهدف

منع النمط التالي:

```text
رسالة عميل
→ تنفيذ مباشر
→ تغيير Scope ضمني
→ Regression / سوء فهم / تكلفة غير محسوبة
```

واستبداله بـ:

```text
Request
→ Classify
→ Clarify Outcome
→ Impact Assessment
→ Approve / Reject / Defer
→ Backlog + Release
→ Acceptance
```

---

# 2. قناة الطلب

يمكن أن يبدأ الطلب من WhatsApp/Email/Meeting/Support، لكن **مصدر التنفيذ الوحيد** هو Backlog item موثق.

على Delivery Owner تحويل الطلب إلى ID وعدم إجبار العميل على استخدام أداة تقنية.

مثال رد داخلي جيد:

> تم تسجيل الطلب كـ`WAM-0123` وسنراجعه ضمن Triage لتحديد هل هو Bug أم Enhancement ونطاق الإصدار المناسب.

---

# 3. التصنيف الأولي

## Bug

إذا كانت الوظيفة:

- داخل Frozen V1/Release Scope.
- ولها سلوك متوقع واضح.
- والسلوك الحالي يخالفه.

## Enhancement

الوظيفة موجودة وتعمل، لكن المطلوب تحسينها أو توسيعها.

## New Feature

Capability لم تكن ضمن Release الحالي.

## Business Rule Change

تغيير في سياسة القبول/الدفع/التقييم/الشهادة/التحضير/الصلاحيات… حتى لو التعديل الكودي صغير.

## Incident

حالة Production عاجلة؛ تدخل Incident Lane فورًا ثم توثق بعد الاحتواء.

---

# 4. الأسئلة الإلزامية قبل قبول تغيير غير عاجل

1. ما المشكلة أو النتيجة المطلوبة؟
2. من المتأثر؟
3. لماذا الآن؟
4. هل يوجد مثال/بيانات/Evidence؟
5. ما السلوك الحالي؟
6. ما السلوك المطلوب؟
7. ما الذي **لا** يدخل في الطلب؟
8. هل يغير بيانات/صلاحيات/مالية/تكامل خارجي؟
9. هل يحتاج تصميم/Content/سياسة؟
10. كيف سنقبل النتيجة؟

---

# 5. Impact Assessment

قبل الموافقة، يراجع الفريق:

| البعد | السؤال |
|---|---|
| Product | هل يغير Journey/Scope؟ |
| UX | هل يحتاج flow/design states؟ |
| Backend/Data | هل يغير schema/contracts/business rules؟ |
| Permissions | هل يضيف وصولًا جديدًا؟ |
| Finance | هل يمس price/order/payment/refund/invoice؟ |
| External | هل يعتمد على مزود/approval/key؟ |
| Operations | هل يضيف support/moderation/manual work؟ |
| Analytics | ما event/KPI المطلوب؟ |
| Migration | هل يحتاج backfill/migration؟ |
| Release | هل قابل للـrollback/flag؟ |
| Commercial | هل خارج Scope الدعم/العقد الحالي؟ |

---

# 6. Change Size

| الحجم | التعريف | المسار |
|---|---|---|
| `S` | تغيير محدود منخفض المخاطر، بلا Business/Data contract جديد | Backlog → Ready → Release |
| `M` | عدة شاشات/قواعد أو migration محدود | Feature Brief + estimate + UAT |
| `L` | Capability كاملة/مزود/داتا/عملية تشغيلية | Discovery + Scope + Technical Review + Release Plan |
| `XL` | Product track/architecture/business model | Initiative/Phase مستقلة + Business approval |

الحجم ليس مدة نهائية؛ هو مستوى الحوكمة المطلوب.

---

# 7. حالات القرار

- `APPROVED FOR DISCOVERY`
- `APPROVED FOR BACKLOG`
- `APPROVED FOR RELEASE <id>`
- `NEEDS BUSINESS DECISION`
- `NEEDS TECHNICAL ASSESSMENT`
- `DEFERRED`
- `REJECTED`

أي Reject يجب أن يحتوي سببًا؛ لا تُحذف الفكرة وكأنها لم تكن.

---

# 8. Deferred Feature Re-enable

أي Feature في `../qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md` لا تُعاد عبر:

```text
FEATURE_X=true
```

فقط.

المسار الصحيح:

1. Change Request / Initiative ID.
2. Business reason + target outcome.
3. Review source/deferred implementation freshness.
4. Scope and out-of-scope.
5. Security/permissions review.
6. Scheduler/Admin/API/Frontend surfaces review.
7. UAT plan.
8. Release Scope decision.
9. Enable in intended environment.
10. Measure after release.
11. Update Deferred Registry status.

---

# 9. Emergency Change

P0/P1 Incident يمكن أن يتجاوز بعض خطوات التخطيط، لكنه لا يتجاوز:

- تحديد Owner.
- Change/Incident ID.
- فهم blast radius بقدر الممكن.
- Rollback/forward-fix plan.
- Production verification.
- Post-incident record إذا كان الأثر جوهريًا.

بعد الاحتواء، تُستكمل الوثائق الناقصة.

---

# 10. Commercial Boundary

هذه الحوكمة لا تحدد السعر أو العقد. لكنها تلزم الفريق بتعليم أي Change يبدو خارج النطاق الحالي كالتالي:

```text
COMMERCIAL_REVIEW_REQUIRED = YES
```

قبل commitment على موعد تنفيذ.

أمثلة:

- تكامل مزود جديد.
- Mobile app.
- Marketplace.
- تغيير جذري في نموذج المنتج أو البنية التشغيلية.
- إعادة بناء Journey كبيرة.
- Operations جديدة تتطلب موظفين/Moderation.

---

# 11. نموذج Change Request

```text
CR ID:
Title:
Requested by:
Date:
Source:

Problem / Desired Outcome:
Current Behavior:
Requested Behavior:
Affected Roles:
Evidence:

Classification:
Size: S/M/L/XL
Product Impact:
Data Impact:
Security Impact:
Finance Impact:
External Dependencies:
Operational Impact:

Out of Scope:
Acceptance Criteria:
Success Metric:
Rough Effort:
Commercial Review Required: Yes/No

Decision:
Target Release:
Owner:
Decision Date:
```

---

# 12. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | اعتماد Change Control موحد لكل طلب بعد V1. |
