# منصة ومضات التعليمية — خارطة طريق المنتج الحية
## PRODUCT ROADMAP — NOW / NEXT / LATER

**الإصدار:** 1.2  
**تاريخ الأساس:** 9 سبتمبر 2026  
**آخر مراجعة:** 13 سبتمبر 2026  
**Document Status:** `LIVING`  
**Owner:** Product/Business Owner + Delivery Owner  
**Review Cadence:** Monthly + Quarterly Strategy Review  

---

# 1. ماذا تستبدل هذه الوثيقة؟

`../qa-final/02-Wamadat-Product-Development-Roadmap.md` وثيقة قوية بنت Candidate Roadmap قبل إغلاق الاستقرار، وتظل مصدرًا مهمًا للمرشحين والاعتماديات. بعد V1 لا نعتمد ترتيبها الزمني تلقائيًا.

هذه الوثيقة هي **الترتيب التنفيذي الحي** منذ 9 سبتمبر 2026.

---

# 2. فلسفة Roadmap الجديدة

الخارطة لا تسأل «ما Feature التالية؟» بل:

1. ما Outcome الذي نريد تحسينه؟
2. ما Evidence الذي يثبت وجود المشكلة أو الفرصة؟
3. ما المبادرة الأقل مخاطرة والأعلى قيمة؟
4. كيف سنعرف أنها نجحت؟

لذلك تستخدم ثلاث نوافذ:

- **NOW:** ما نعمل على اكتشافه/تنفيذه حاليًا.
- **NEXT:** المرشح الأقرب بعد تحقق الأدلة والجاهزية.
- **LATER:** اتجاهات وفرص مهمة بلا التزام زمني.

ولا تستخدم تواريخ بعيدة كالتزام تعاقدي إلا داخل Release Plan منفصل.

---

# 3. Product Outcomes المعتمدة كبداية

يختار Product Owner من هذه Outcomes ويحدد Target قابلًا للقياس بعد جمع Baseline الفعلي:

| Outcome | أمثلة KPI | لماذا مهم |
|---|---|---|
| Reliability | P0/P1, payment/webhook/queue failures | حماية الإيراد والثقة |
| Commerce Conversion | checkout→payment, payment success | زيادة التحويل من الاهتمام إلى تسجيل مدفوع |
| Learning Completion | lesson engagement, completion | تحقيق القيمة التعليمية |
| Learner Engagement | returning learners, quiz/assignment engagement | زيادة الاستمرارية |
| Support Efficiency | ticket volume, response/resolution | تقليل الضغط التشغيلي |
| Instructor Efficiency | grading/support workflow time | تحسين التشغيل التعليمي |
| Security & Trust | access/session/security events | حماية الحسابات والبيانات |

الأرقام المستهدفة لا تُخترع داخل الوثيقة؛ تُثبت بعد Baseline قياس حقيقي.

---

# 4. NOW — مرحلة الانتقال بعد V1

**Theme:** `V1 Closure → Evidence-Driven V1.1 Planning`

الأعمال الحالية ذات الأولوية:

1. اعتماد V1 Closure وProduct Baseline.
2. تجميع Backlog واحد بدل الطلبات المتفرقة.
3. تفعيل/تأكيد Measurement للـKPIs الضرورية لاتخاذ القرار.
4. مراجعة Production feedback والدعم وأخطاء التشغيل.
5. Score مرشحي V1.1 وفق `04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md`.
6. اختيار **1–3 مبادرات فقط** لأول Release بعد V1.
7. كتابة Feature Brief/Acceptance Criteria لكل مبادرة مختارة.

> **لا يبدأ كل V1.1 دفعة واحدة.** أول Release بعد الاستقرار يجب أن يكون صغيرًا وقابلًا للقياس والتراجع.

---

# 5. NEXT — Candidate Pool لـV1.1

هذه قائمة مرشحين، وليست Commitment order:

| Candidate | المصدر السابق | Theme | ملاحظة قرار |
|---|---|---|---|
| **Mobile App (Flutter — Android/iOS)** | Business + technical decision post-V1 | Mobile Access / Engagement | **PRIORITY CANDIDATE**؛ Flutter محسوم نهائيًا، والبناء سيكون Clean Flutter implementation دون إعادة استخدام React Native/Expo القديم. المتبقي هو MVP/API readiness/effort/release scoping فقط، ويمكن أن يسير كمسار مستقل عن Web V1.1 |
| Direct Messaging | V1.1 / P1 | Communication | يحتاج permission/isolation review صارم |
| Program Reviews / Ratings | V1.1 | Trust / Conversion | يحتاج eligibility + moderation policy |
| Account Security Enhancements | V1.1 / P1 | Security | بعض العناصر قد تصعد فورًا إن كانت Risk لا Feature |
| Advanced Notifications | V1.1 P1/P2 | Engagement | يفضل فصل transactional preferences عن marketing |
| Gamification | V1.1 P2 | Engagement | لا يسبق مشاكل completion/reliability |
| Gifts | V1.1 P2 | Commerce | يحتاج refund/expiry/redeem rules |
| Calendar | V1.1 P2 | Organization | يقرر حسب deadlines/support feedback |
| Advanced Student Profile | V1.1 P2 | Profile | لا يربط بالـcertificate دون سبب واضح |
| Wallet | Deferred V1.1 | Commerce | يحتاج Business Case وfinancial contract واضح |
| Wishlist | Deferred V1.1 | Discovery/Retention | تعتمد على Evidence الاستخدام |

**قاعدة الاختيار:** Candidate لا يدخل `NEXT COMMITTED` حتى يمتلك Problem, Evidence, Owner, KPI, Rough Effort, Risk, Dependencies.

---

# 6. LATER — Phase 2 Opportunity Pool

- Advanced Analytics.
- Live Sessions.
- Advanced Attendance.
- Push Notifications.
- Community / Forum.
- Consultations.
- SMS.
- Marketing Automation.
- Newsletter.
- CRM / Leads.

هذه ليست Release واحدة. كل عنصر يحتاج Business/Operational readiness خاصًا به، خصوصًا Community/Consultations/Live لأنها تضيف Moderation/Support/Provider dependencies.

---

# 7. LATER — Strategic Expansion Pool

- AI Assistant / AI learning features.
- Wamadat Plus / Marketplace.
- Subscriptions.
- Bundles.
- Affiliate.
- Alumni.
- B2B.

القاعدة الاستراتيجية: **لا يفتح أكثر من مسار توسع Product كبير واحد أو اثنين في نفس الدورة الاستراتيجية** ما لم تتوفر قدرة مستقلة واضحة. تطبيق الهاتف هو الاستثناء الوحيد الذي تم رفعه مسبقًا إلى Priority Candidate، لكنه يظل خاضعًا لـScope/Ready/Capacity gates.

---

# 8. V1.x Release Themes المقترحة

هذه Naming/Packaging strategy وليست التزامًا ثابتًا:

```text
V1.0.x — Stability / Hotfix / small operational corrections
V1.1.x — First validated post-V1 product improvements
V1.2.x — Second outcome-driven package
V1.3.x — Larger capability only if evidence justifies it
V2.0.0 — Major product/business boundary change
```

لا نسمي `V1.1.0` «Messaging Release» قبل اعتماد Messaging فعليًا.

---

# 9. كيفية نقل عنصر على الخارطة

## LATER → NEXT CANDIDATE

يتطلب:

- Problem واضح.
- Evidence أولي.
- Strategic fit.
- Owner.
- Rough effort/dependency view.

## NEXT CANDIDATE → NEXT COMMITTED

يتطلب:

- Priority score.
- Scope draft.
- Success Metric.
- Capacity fit.
- لا يوجد P0/P1 stability conflict.

## NEXT COMMITTED → NOW

يتطلب Definition of Ready كاملة.

## NOW → RELEASED

يتطلب Definition of Done + Release Gate.

---

# 10. Roadmap Guardrails

1. P0 يوقف ترتيب Roadmap الطبيعي.
2. ارتفاع واضح في reliability failures يسمح بتجميد Feature delivery مؤقتًا.
3. Feature مطلوبة من شخص واحد دون Evidence لا تصبح تلقائيًا P1.
4. Feature منخفضة Effort وعالية الأثر يمكن أن تتقدم على مشروع أكبر.
5. أي ميزة ذات Financial/Security/Data risk تحتاج Review إضافي حتى لو Score مرتفع.
6. الاعتماد على مزود خارجي يجب أن يظهر Dependency قبل الالتزام بالتاريخ.
7. لا ننقل Feature مؤجلة إلى Production بالـFlag فقط.

---

# 11. Product Review الشهري

في كل اجتماع شهري تعرض صفحة واحدة:

1. What changed since last review?
2. Top product KPIs.
3. Platform health summary.
4. Top customer/support themes.
5. Roadmap NOW status.
6. Decisions needed.
7. Candidates moving NEXT/LATER.
8. Scope/Commercial impacts.

الهدف قرار، لا تقرير طويل.

---

# 12. Quarterly Strategy Questions

- ما أهم Outcome حققناه؟
- ما الذي لم يتحسن رغم Features؟
- هل تغير segment/market need؟
- هل لدينا قدرة تشغيلية للمبادرة القادمة؟
- ما هو MVP الصحيح لتطبيق Flutter ومتى يبدأ Track التنفيذ دون الإضرار باستقرار Core Web؟
- ما أولوية B2B/Live/AI وغيرها بعد قياس نتائج الـCore؟
- ما المخاطر/الدين التقني الذي يجب تمويله قبل التوسع؟

---

# 13. Roadmap Snapshot Template

```text
NOW
- [Initiative] — Outcome — Owner — Target Release — Health

NEXT COMMITTED
- [Initiative] — Evidence — Success Metric — Rough Effort

NEXT CANDIDATE
- [Initiative] — Missing decision/evidence

LATER
- Theme / Opportunity only

BLOCKED
- [Initiative] — Dependency / decision / external provider
```

---

# 14. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | استبدال الترتيب الزمني الثابت بخارطة Now/Next/Later مبنية على Outcomes وEvidence. |
| 2026-09-13 | 1.1 | رفع Mobile App إلى Priority Candidate مبكر، وإزالة تعدد الأكاديميات من Strategic Expansion Pool. |
| 2026-09-13 | 1.2 | تثبيت Flutter كتقنية التطبيق الجديد وإخراج Program Interest / Waitlist من Candidate Pool لأنه جزء معتمد من V1. |
