# منصة ومضات التعليمية — مرجع حوكمة المنتج والإصدارات
## WAMADAT V1 CLOSURE & PRODUCT GOVERNANCE BASELINE

**الإصدار:** 1.1  
**تاريخ الاعتماد الأساسي:** 9 سبتمبر 2026  
**آخر مراجعة حوكمية:** 13 سبتمبر 2026  
**حالة الوثيقة:** `LIVING — AUTHORITATIVE GOVERNANCE INDEX`  
**نطاق التطبيق:** ما بعد إغلاق Launch V1 لمنتج ومضات؛ Web هو Baseline V1، وMobile مسار Flutter جديد مستقل في التنفيذ ومحكوم بنفس العملية  
**الجهات المعنية:** Wamadat Product/Business Owner + Smart Agency Delivery/Engineering  

---

# 1. الغرض من هذه الحزمة

هذه الحزمة تنقل منصة ومضات من نموذج **مشروع إطلاق** إلى نموذج **منتج Production مُدار بإصدارات وقرارات قابلة للتتبع**.

لا تعيد هذه الحزمة توثيق كل Feature أو Endpoint أو تفصيل تقني موجود مسبقًا، بل تضيف طبقة الحوكمة التي تحدد:

- ما الذي تم إغلاقه كـV1.
- ما الذي يعد Baseline ثابتًا وما الذي يظل Living Document.
- كيف تدخل طلبات العميل والمستخدمين والفريق.
- كيف تُصنّف Bugs وEnhancements وNew Features وIncidents.
- كيف تُرتب الأولويات بناءً على الأثر والبيانات لا على ترتيب الرسائل.
- كيف تتحول المبادرة إلى Feature جاهزة للتطوير.
- كيف يتم الإصدار والنشر والتراجع.
- كيف تقاس صحة المنتج والاستقرار بعد كل Release.
- كيف يسجل الدين التقني والمخاطر والقرارات.

> **المبدأ الحاكم:** بعد إغلاق V1 لا تعتبر أي فكرة أو طلب أو كود موجود مسبقًا جزءًا من المنتج المنشور ما لم يدخل عبر Backlog، ويُعتمد Scope، ويجتاز Ready/Done/Release Gates، ويُربط بإصدار محدد.

---

# 2. حالة المنتج عند نقطة القطع

تعتمد هذه الحزمة نقطة القطع التالية:

```text
Product: Wamadat Educational Platform
Baseline Release: WAMADAT-V1.0.0
Baseline Date: 2026-09-09
Business State: Production Stable / V1 Closure
Next Operating Mode: Managed Product Development
```

إغلاق V1 هنا يعني **تجميد نطاق النسخة المسلمة** وليس إيقاف الصيانة. الإصلاحات الأمنية، أعطال Production، أخطاء البيانات، وأعمال الاستقرار تظل مسارًا مستمرًا.

---

# 3. هرم مصادر الحقيقة (Source of Truth Hierarchy)

عند التعارض، يطبق الترتيب التالي:

1. **قرارات Governance المعتمدة الأحدث** داخل هذا المجلد.
2. **Current Product Baseline** في `02-CURRENT-PRODUCT-BASELINE.md`.
3. **V1 Frozen Functional Scope** في `../qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md`.
4. **Deferred Features Registry** في `../qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md`.
5. الوثائق التقنية الحالية داخل `../technical/`.
6. UAT/QA Evidence الخاصة بالإطلاق.
7. Git history والتقارير التاريخية عند الحاجة للتحقيق فقط.

الـRoadmap القديمة في `../qa-final/02-Wamadat-Product-Development-Roadmap.md` تبقى **مرجع Candidate Vision وخلفية تاريخية**، لكن ترتيبها الزمني لا يعد التزامًا تنفيذيًا بعد 9 سبتمبر 2026. المرجع التنفيذي الحي هو `03-PRODUCT-ROADMAP.md`.

---

# 4. تصنيف الوثائق

| الحالة | المعنى | هل تعدل؟ |
|---|---|---|
| `FROZEN` | Baseline لإصدار مغلق أو Evidence تاريخي | لا؛ أي تصحيح يكون بإصدار/ملحق جديد |
| `LIVING` | مرجع تشغيلي أو Product Governance مستمر | نعم مع Changelog |
| `ARCHIVED` | دليل تاريخي لا يقود قرارات حالية | لا إلا لتصحيح وصفي واضح |
| `REGISTER` | سجل حي للطلبات/المخاطر/القرارات | يحدّث باستمرار |
| `TEMPLATE` | نموذج يعاد استخدامه | ينسخ لكل حالة ولا يكتب فوق الأصل |

---

# 5. خريطة الوثائق المعتمدة بعد الإغلاق

| الوثيقة | الحالة بعد 9 سبتمبر 2026 | الدور |
|---|---|---|
| `qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md` | `FROZEN` | النطاق الوظيفي المسلم في V1 |
| `qa-final/02-Wamadat-Product-Development-Roadmap.md` | `ARCHIVED CANDIDATE ROADMAP` | الرؤية السابقة ومخزون المرشحين فقط |
| `qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md` | `FROZEN EVIDENCE + REUSABLE REGRESSION SOURCE` | مرجع قبول V1 ومصدر حالات regression |
| `qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md` | `LIVING REGISTER` | المصدر الرسمي للميزات المغلقة/المؤجلة |
| `qa-final/05-Wamadat-Launch-V1-UAT-Links-and-Test-Payment-Data.md` | `ARCHIVED / TEST REFERENCE` | بيانات وروابط اختبار؛ ليست مرجع إعداد Production |
| `technical/01..05` | `LIVING TECHNICAL BASELINE` | المعمارية، الأمان، التكاملات، التشغيل، الاستعادة |
| `governance/01` | `FROZEN AFTER SIGN-OFF` | إغلاق وقبول V1 |
| `governance/02` | `FROZEN PER MAJOR BASELINE` | صورة المنتج المنشور عند الإغلاق |
| `governance/03..10` | `LIVING` | إدارة التطوير بعد V1 |

---

# 6. الحزمة الجديدة

1. `01-V1-RELEASE-CLOSURE-AND-ACCEPTANCE.md` — إغلاق V1 وفصل ما بعده.
2. `02-CURRENT-PRODUCT-BASELINE.md` — صورة واضحة لما يقدمه المنتج اليوم.
3. `03-PRODUCT-ROADMAP.md` — Now / Next / Later وإدارة المبادرات.
4. `04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md` — Backlog + Triage + RICE-lite.
5. `05-CHANGE-REQUEST-PROCESS.md` — ضبط طلبات العميل والتغييرات.
6. `06-FEATURE-DELIVERY-LIFECYCLE.md` — Definition of Ready/Done ومسار Feature.
7. `07-RELEASE-MANAGEMENT.md` — الإصدارات، Gate، Hotfix، Rollback، Release Notes.
8. `08-PRODUCT-KPI-AND-PLATFORM-HEALTH.md` — Product/Business/Platform KPIs وصحة التشغيل.
9. `09-TECHNICAL-DEBT-AND-RISK-REGISTER.md` — الدين التقني والمخاطر.
10. `10-DECISION-LOG.md` — سجل القرارات الجوهرية.

---

# 7. الأدوار والمسؤوليات

لا يشترط وجود موظف مستقل لكل دور؛ يمكن لشخص واحد حمل أكثر من قبعة، لكن يجب أن يكون **صاحب القرار واضحًا**.

| الدور | المسؤولية الأساسية | لا يملك منفردًا |
|---|---|---|
| **Product / Business Owner — Wamadat** | الأهداف التجارية، الأولويات، قبول Scope، UAT التجاري | فرض تنفيذ فوري خارج Change Process |
| **Delivery Owner — Smart Agency** | تنظيم Backlog، التخطيط، التنسيق، إصدار التقديرات، توثيق القرارات | تغيير هدف Business دون اعتماد العميل |
| **Engineering Owner** | التصميم التقني، المخاطر، جودة التنفيذ، Migration/Release readiness | تحويل Candidate إلى التزام Product منفردًا |
| **QA / Acceptance Owner** | معايير القبول، Regression، UAT evidence | تغيير Scope لإغلاق الاختبار |
| **Operations Owner** | Production health، incidents، backup، deploy/rollback readiness | تجاهل Release Gate لأجل السرعة |

---

# 8. Cadence التشغيلي المعتمد

لمنع البيروقراطية مع الحفاظ على الانضباط:

| التكرار | النشاط | الناتج |
|---|---|---|
| مستمر | Intake | Backlog item موحد |
| أسبوعيًا | Product Triage — 30 إلى 45 دقيقة | تصنيف، merge، reject، clarify، score |
| كل أسبوعين | Development Cycle Planning | مجموعة صغيرة جاهزة للتنفيذ |
| عند الحاجة | Discovery / Feature Brief | Scope + Acceptance Criteria |
| لكل Release | Release Gate | Go / No-Go موثق |
| أسبوعيًا | Platform Health Review مختصر | P0/P1 + Health signals |
| شهريًا | Product & Roadmap Review | KPI + Priorities + Decisions |
| ربع سنوي | Strategy / Risk Review | تغيير Themes أو Phase عند الحاجة |

لا يشترط Sprint إذا كان تدفق العمل Continuous، لكن **لا يبدأ عنصر غير Ready**.

---

# 9. القواعد العشر غير القابلة للتجاوز

1. Production هو منتج حي؛ الاستقرار يسبق سرعة Features.
2. لا طلب تنفيذ مباشر من WhatsApp/Email دون Backlog ID.
3. P0 Incident يتجاوز الترتيب العادي لكنه يدخل Incident Record.
4. Bug لوظيفة V1 الملتزم بها يختلف عن Enhancement أو New Feature.
5. لا Feature بلا Problem Statement وAcceptance Criteria.
6. لا Feature مؤجلة تفعّل بمجرد رفع Flag؛ يلزم Scope/Release decision موثق.
7. لا Release دون Backup/rollback awareness والتحقق المناسب لحجمه.
8. لا تعتبر Feature Done بمجرد انتهاء Coding.
9. Roadmap ليست وعد تواريخ بعيدة؛ هي ترتيب قرارات مبني على Evidence.
10. كل تغيير جوهري قابل للتتبع من Request → Decision → Release → Result.

---

# 10. Workflow المختصر

```text
Feedback / Request / Data / Incident
                ↓
            Intake ID
                ↓
     Classify + Triage + Evidence
                ↓
        Backlog / Incident Lane
                ↓
         Priority / Roadmap
                ↓
       Feature Brief / Scope
                ↓
        Definition of Ready
                ↓
    Technical Review / Delivery
                ↓
      Verification + UAT Gate
                ↓
             Release
                ↓
       Monitor + Measure Result
                ↓
     Keep / Iterate / Rollback
```

---

# 11. سياسة التغيير على هذه الحزمة

أي تعديل حوكمي جوهري يجب أن يسجل في `10-DECISION-LOG.md` ويحدث تاريخ الوثيقة ذات العلاقة. لا تعدل وثائق `FROZEN` لتبدو كما لو أن القرار كان موجودًا وقت الإغلاق.

---

# 12. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | إنشاء Baseline رسمي لإغلاق V1 والانتقال إلى Managed Product Development. |
| 2026-09-13 | 1.1 | اعتماد Single-Academy product boundary، Mobile priority، وتثبيت Integration baseline ضمن V1. |
| 2026-09-13 | 1.2 | تثبيت Flutter كتقنية Mobile الجديدة، واعتماد Program Interest / Waitlist ضمن V1. |
