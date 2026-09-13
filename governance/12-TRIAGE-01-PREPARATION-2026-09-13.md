# منصة ومضات التعليمية — جلسة Triage رقم 01
## FIRST POST-V1 PRODUCT TRIAGE PACK

**التاريخ المقترح:** أول Product Review بعد اعتماد V1  
**المدة:** 60–90 دقيقة  
**الحالة:** `PREPARED / NOT YET DECIDED`  
**Backlog Reference:** `11-INITIAL-PRODUCT-BACKLOG-2026-09-13.md`

---

# 1. هدف الجلسة

هذه ليست جلسة طلب Features ولا جلسة تصميم تفصيلي.

نخرج منها بـ:

1. أهم 3 Outcomes للأيام الـ90 القادمة.
2. Evidence/Business priority لأول Shortlist.
3. قرارات `REJECT / DEFER / NEEDS_EVIDENCE / NEXT_CANDIDATE`.
4. تحديد العناصر التي تستحق Scoring وFeature Brief.
5. عدم تحديد `V1.1.0` نهائيًا قبل Technical Assessment.

---

# 2. الحضور والأدوار

## Wamadat / Client Product Owner

- يحدد Business Outcomes.
- يشرح Pain/Opportunity.
- يعتمد Business priority.
- يوافق على السياسات/القواعد التجارية.

## Smart Agency / Product & Delivery Lead

- يدير الجلسة.
- يحول الطلبات إلى Backlog items.
- يمنع تحويل النقاش مباشرة إلى Coding.
- يسجل Scope/Dependencies/Risks.

## Engineering

- لا يقرر Business priority.
- يقدم Rough effort/dependency/risk بعد وضوح المشكلة.

---

# 3. قبل الاجتماع — Smart Agency

جهز نافذة بيانات واحدة بقدر المتوفر:

- Visitors / Program Views / Registration conversion.
- Checkout started / payment success / failure by gateway.
- Enrollment after payment.
- Lesson engagement / completion إن كانت البيانات موثوقة.
- Support ticket themes.
- Failed jobs / queue / webhook / email failures.
- أهم production errors.
- نسبة/حجم استخدام Mobile Web إن أمكن من GA4.

إذا Metric غير موثوقة، اكتب `NOT RELIABLE YET` ولا تخترع رقمًا.

---

# 4. الجزء الأول — Outcomes (15–20 دقيقة)

اطلب من العميل ترتيب **ثلاثة فقط** من التالي للأيام الـ90 القادمة:

- Reliability & trust.
- Commerce conversion.
- Learning completion.
- Learner engagement / return rate.
- Instructor efficiency.
- Support/operations efficiency.
- Mobile access & engagement.
- Better management visibility / analytics.

## القرار المسجل

```text
Outcome 1:
Why:
Baseline metric:
Desired direction:

Outcome 2:
Why:
Baseline metric:
Desired direction:

Outcome 3:
Why:
Baseline metric:
Desired direction:
```

---

# 5. الجزء الثاني — Shortlist Triage

## WAM-0001 — Flutter Mobile MVP Scope & Delivery Foundation

**ما نعرفه:** Mobile أولوية post-V1 معتمدة، وقرار التقنية محسوم: Flutter لـAndroid/iOS ببناء جديد من الصفر. الكود React Native/Expo القديم Retired ولا يدخل في التنفيذ.

أسئلة العميل:

1. لماذا نحتاج التطبيق وما Outcome الأساسية؟
2. ما أهم 3 رحلات يجب أن تكون في MVP؟
3. Student فقط أم Instructor أيضًا؟
4. هل Push requirement أساسي في MVP؟
5. هل Offline/download requirement أساسي؟
6. هل Checkout/payment يجب أن يكون Native داخل MVP أم Web handoff؟
7. هل التطبيق مطلوب لأسباب Store presence/brand أم لحل مشكلة استخدام؟

**قرار الجلسة:**

```text
NEXT_CANDIDATE / NEEDS_EVIDENCE / DEFER
```

> لا تتم مناقشة Framework داخل الجلسة؛ Flutter قرار معتمد. دور Smart Agency بعد وضوح الـMVP هو API gap assessment، architecture foundation، effort/risk، وDefinition of Ready.

---

## WAM-0002 — Product KPI Baseline & Health Board

أسئلة القرار:

1. ما المؤشرات التي يعتمد عليها العميل شهريًا فعلًا؟
2. ما الذي يحتاج أن يراه Executive مقابل Operations؟
3. ما المصادر التي نثق بها الآن؟

**Default recommendation:** `NOW-ENABLER` لأن بقية الـRoadmap تحتاج Evidence.

---

## WAM-0010 — Account Security Enhancements

أسئلة:

- هل توجد شكاوى login/session/security متكررة؟
- هل يحتاج العميل رؤية Active Sessions/Devices؟
- هل 2FA requirement الآن أم لاحقًا؟

**ملاحظة:** Security risk حقيقي يمكن أن يملك Override حتى لو كان Reach منخفضًا.

---

## WAM-0011 — Direct Messaging

أسئلة:

- هل الطلاب يحتاجون التواصل الخاص مع المدرب خارج Lesson Q&A؟
- كم مرة يطلب الدعم هذا؟
- هل التواصل يجب أن يكون per-program فقط؟
- من يحق له بدء conversation؟
- هل Admin يحتاج moderation/audit؟

لا يعتمد قبل سياسة Permissions/Isolation واضحة.

---

## WAM-0012 — Advanced Notifications

أسئلة:

- ما الإشعارات التي يحتاج المستخدم التحكم بها؟
- ما الإشعارات transactional التي لا يجوز إيقافها؟
- هل المشكلة الحالية كثرة الإشعارات أم ضعفها؟
- هل هذا شرط سابق لـPush/Mobile؟

---

## WAM-0020 — Advanced Reporting & Analytics

بدل السؤال «ما التقارير التي تريدونها؟»، نسأل:

> ما القرارات التي لا يستطيع Admin/Instructor اتخاذها اليوم لأن البيانات غير ظاهرة؟

سجل كل قرار مطلوب، ثم اشتق التقرير منه.

---

## WAM-0021 — Basic Live Sessions

أسئلة:

- هل البرامج الحالية تحتاج Live فعلًا؟
- ما المزود المقبول: Zoom/Meet/Teams link أم integration؟
- هل تسجيل الحضور للجلسة requirement؟
- هل recording requirement؟
- من ينشئ الجلسة ومن يدعمها تشغيليًا؟

لا يبدأ Feature قبل Operating Model.

---

## WAM-0013 — Reviews / Ratings

أسئلة:

- هل reviews مطلوبة لرفع الثقة/التحويل؟
- من يحق له التقييم: enrolled / completed / attended؟
- هل النص mandatory؟
- هل moderation قبل النشر أم بعده؟

---

# 6. القرارات الداخلية لا تحتاج تصويت العميل

## WAM-0003 — Restore Drill

- ينفذ كـOperations/Reliability work.
- لا يحتاج RICE تجاريًا.

## WAM-0004 — Provider Health & Alerting

- Scope يركز على observability/alerts؛ لا يعيد بناء reconciliation الموجود.

## WAM-0005 — Program Interest Baseline Correction — CLOSED

تم حسمه قبل Triage: Program Interest / Waitlist جزء معتمد من V1، وتمت إزالته من Deferred Registry. لا يحتاج قرارًا من العميل في هذه الجلسة ولا يدخل Scoring.

---

# 7. قرار كل Candidate أثناء Triage

استخدم فقط:

- `REJECTED`
- `DEFERRED`
- `NEEDS_EVIDENCE`
- `NEEDS_DECISION`
- `READY_FOR_SCORING`
- `NEXT_CANDIDATE`

لا تستخدم `READY` قبل Technical Assessment + Acceptance Criteria.

---

# 8. نموذج تسجيل القرار

```yaml
id: WAM-XXXX
decision:
reason:
business_outcome:
evidence:
missing_evidence:
priority_candidate:
owner:
next_action:
due_before_next_review:
```

---

# 9. ما بعد الاجتماع — Smart Agency

خلال نفس Product Cycle:

1. تنظيف قرارات الجلسة داخل Backlog.
2. إزالة Duplicate/Rejected.
3. جمع Missing Evidence.
4. Engineering rough effort للمرشحين فقط.
5. RICE-lite للعناصر التي أصبحت قابلة للقياس.
6. Feature Brief لكل عنصر `NEXT_CANDIDATE` قوي.
7. تطبيق Definition of Ready.
8. اختيار 1–3 Initiatives كحد أقصى لـ`NEXT COMMITTED`.
9. إنشاء `V1.1.0 RELEASE SCOPE` فقط بعد ذلك.

---

# 10. Exit Criteria للجلسة

الجلسة ناجحة إذا خرجنا بـ:

- [ ] 3 Outcomes فقط.
- [ ] قرار واضح لـFlutter Mobile MVP scope.
- [ ] 3–5 candidates تستحق Scoring.
- [ ] لا يوجد Feature بدأ Coding أثناء الجلسة.
- [ ] كل Missing Evidence له Owner.
- [ ] لا يوجد Release date وهمي قبل Technical Assessment.

---

# 11. القاعدة النهائية

> العميل يختار **المشكلة والقيمة والأولوية التجارية**.  
> Smart Agency تختار **التصميم التقني، التقدير، والمخاطر**.  
> الـRelease لا يتكون إلا من عناصر Ready، وليس من قائمة رغبات.
