# منصة ومضات التعليمية
## خارطة طريق التطوير والإطلاق — Product Development Roadmap

**نوع الوثيقة:** خطة تطوير وإطلاق مرحلية  
**الإصدار:** 1.1 Baseline  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**المنتج:** منصة ومضات التعليمية  
**النطاق الحالي:** Web Platform — Launch V1 وما بعده  

---

# 1. الهدف من الوثيقة

تهدف هذه الوثيقة إلى تحويل منصة ومضات من **Launch V1 مستقر ومركز على رحلة التعليم الأساسية** إلى منصة تعليمية متقدمة قابلة للتوسع، من خلال خارطة تطوير مرتبة حسب:

- الأولوية.
- أثر الميزة على المستخدم والعمل.
- جاهزية الكود الحالي.
- تكلفة الاختبار.
- الاعتماديات.
- مخاطر الإطلاق.
- الوقت المتوقع للتنفيذ.
- الترتيب المنطقي بين المراحل.

تم بناء الخطة على مبدأ أساسي:

> **لا يتم إطلاق كل ما هو موجود في السورس في يوم واحد؛ بل يتم إطلاق ما يحقق قيمة واضحة، ويمكن اختباره وتشغيله بثقة، ثم توسيع المنتج تدريجيًا بناءً على الاستخدام الحقيقي وبيانات المنصة.**

---

# 2. نقطة الانطلاق

يمثل Launch V1 الأساس الذي تبنى عليه جميع المراحل اللاحقة.

النطاق الأساسي المعتمد في V1 يشمل:

### الاكتشاف
- Programs.
- Categories.
- Search.
- Instructor Profiles.

### التجارة
- Cart.
- Coupons.
- Checkout.
- Payments.
- Bank Transfer.
- Orders.
- Invoices.

### التعلم
- Enrollments.
- My Programs.
- Units.
- Lessons.
- Video.
- PDF.
- Text.
- Resources.
- Lesson Notes.
- Lesson Q&A.

### التقييم
- Quizzes.
- Quiz Attempts.
- Assignments.
- Submissions.
- Instructor Grading.

### الإكمال
- Progress.
- Completion.
- Attendance الأساسي.
- Certificates.
- PDF Certificates.
- QR.
- Public Verification.

### التشغيل
- Transactional Notifications.
- Support Tickets.
- Student Dashboard.
- Instructor Dashboard.
- Admin Dashboard.
- Site Settings.

ولا تدخل الوظائف التوسعية مثل الجلسات المباشرة، المجتمع العام، الاستشارات، الرسائل المباشرة، التقييمات، Gamification، التطبيقات، Marketplace والـAI ضمن Launch V1 الأول.

---

# 3. فلسفة التطوير بعد الإطلاق

تعتمد خارطة الطريق على أربع مراحل رئيسية:

| المرحلة | الهدف |
|---|---|
| **Post-Launch Stabilization** | تثبيت V1 ومعالجة ما يظهر من الاستخدام الحقيقي |
| **V1.1 — Engagement & Experience** | إضافة الوظائف القريبة من V1 والموجود جزء كبير منها في السورس |
| **Phase 2 — Learning & Operations Expansion** | توسيع التعليم والتواصل والتحليلات والخدمات |
| **Phase 3 — Product & Business Expansion** | تطبيقات الهاتف، AI، Marketplace، SaaS، B2B والتوسع التجاري |

---

# 4. نظام الأولويات

يستخدم المشروع أربع درجات أولوية:

## P0 — Critical

أي مشكلة تؤثر على:

- الدفع.
- الوصول إلى البرامج.
- فقدان البيانات.
- الصلاحيات.
- إصدار الشهادات.
- الأمن.
- توقف المنصة.
- أخطاء تمنع الرحلة الأساسية.

هذه العناصر تعالج فورًا.

---

## P1 — High

وظائف أو تحسينات ذات أثر مباشر على:

- تجربة الطالب.
- تجربة المدرب.
- عمليات الإدارة.
- Conversion.
- Retention.
- Support Load.
- Completion Rate.

---

## P2 — Medium

تطويرات مهمة ولكن يمكن تشغيل المنصة بدونها، مثل:

- Engagement.
- Community.
- Advanced analytics.
- Marketing automation.
- إضافات التشغيل.

---

## P3 — Strategic

مشاريع توسع كبيرة أو Products مستقلة، مثل:

- Mobile Apps.
- Marketplace.
- SaaS.
- B2B.
- AI.
- Subscription Products.

---

# 5. المرحلة صفر — Post-Launch Stabilization

**الأولوية:** P0 / P1  
**المدة المقترحة:** 1–2 أسبوع بعد الإطلاق العام  
**الهدف:** التأكد أن V1 يعمل بثبات في الاستخدام الحقيقي قبل إضافة Features جديدة.

---

## 5.1 ما يتم العمل عليه

### مراقبة الرحلة الأساسية

يتم تتبع:

```text
Registration
→ Checkout
→ Payment
→ Order
→ Enrollment
→ Learning
→ Quiz
→ Assignment
→ Completion
→ Certificate
```

### مراقبة العمليات التشغيلية

- Bank Transfers.
- Email Delivery.
- Queue.
- Scheduler.
- Webhooks.
- Failed Jobs.
- Certificate Generation.
- Storage.
- Support Tickets.

### إصلاح P0/P1 فقط

لا يفتح خلال هذه الفترة تطوير Features جديدة كبيرة.

---

## 5.2 التحسينات المسموح بها

- أخطاء UI واضحة.
- Dead Links.
- تحسين Empty/Error/Loading States.
- إصلاحات Responsive.
- تحسينات RTL.
- تحسين رسائل الخطأ.
- تحسينات UX صغيرة ذات أثر واضح.
- تحسين Logging وMonitoring.
- تحسين Observability.
- Hardening لعمليات الدفع والـWebhooks.

---

## 5.3 معايير إغلاق مرحلة الاستقرار

لا ننتقل إلى V1.1 إلا عندما:

- لا توجد مشكلة P0 مفتوحة.
- مشاكل P1 الحرجة تمت معالجتها.
- الدفع يعمل بثبات.
- Bank Transfer يعمل من البداية للنهاية.
- Enrollment لا يتكرر.
- لا توجد ازدواجية Orders/Payments/Invoices.
- Quiz وAssignment يعملان بثبات.
- Completion يعمل.
- Certificate PDF وQR وVerification يعمل.
- البريد يعمل.
- Queue/Scheduler مستقران.
- Support يعمل.
- صلاحيات Student / Instructor / Admin سليمة.

---

# 6. V1.1 — Engagement & Experience

**الأولوية:** P1  
**المدة الإجمالية المقترحة:** 4–6 أسابيع  
**التوقيت:** بعد استقرار Launch V1  
**طبيعة المرحلة:** Hardening + UI Closure + Testing لميزات موجودة كليًا أو جزئيًا في السورس.

هذه المرحلة لا تهدف إلى إعادة بناء المنصة، بل إلى الاستفادة من وظائف موجودة وإدخالها تدريجيًا بعد اختبارها بصورة مستقلة.

---

# 7. الرسائل المباشرة بين الطالب والمدرب

**الأولوية:** P1  
**التقدير:** 1–2 أسبوع  
**الحالة الحالية:** يوجد أساس وظيفي في السورس.

## الهدف

توفير قناة تواصل منظمة بين الطالب ومدرب البرنامج خارج سياق السؤال داخل الدرس.

## النطاق

- Conversations.
- Student → Instructor.
- Instructor → Student.
- Message list.
- Unread state.
- إرسال واستقبال الرسائل.
- صلاحيات تمنع التواصل خارج العلاقة التعليمية الصحيحة.
- إشعارات عند الرسائل الجديدة.

## الاختبارات المطلوبة

- Student cannot message unauthorized instructor.
- Instructor only accesses allowed students.
- Unread counts.
- Conversation persistence.
- Duplicate prevention.
- Permissions.
- Error/empty states.

## معيار الإطلاق

يتم إطلاقها فقط بعد التأكد أن الصلاحيات تمنع أي وصول إلى محادثات مستخدمين آخرين.

---

# 8. تقييمات البرامج والمراجعات

**الأولوية:** P1  
**التقدير:** 3–5 أيام عمل  
**الحالة:** جزء كبير من Student Flow موجود.

## المطلوب

- السماح للطالب المؤهل بالتقييم.
- تحديد سياسة من يحق له التقييم.
- 1–5 Stars.
- Review text.
- تعديل/حذف التقييم وفق السياسة.
- احتساب Average Rating.
- Admin Moderation.
- إخفاء التقييمات المخالفة عند الحاجة.

## قبل الإطلاق

يجب إكمال واجهة إدارة/Moderation التقييمات في Admin إذا لم تكن مكشوفة بالكامل.

---

# 9. Streak & Achievements

**الأولوية:** P2  
**التقدير:** 1–2 أسبوع

## الهدف

رفع الاستمرارية والتحفيز.

## Streak

- احتساب أيام النشاط.
- حماية من التكرار.
- التعامل مع Timezone.
- تعريف واضح لما يعتبر Activity.

## Achievements

أمثلة:

- أول درس مكتمل.
- أول Quiz ناجح.
- أول برنامج مكتمل.
- عدد محدد من أيام الاستمرارية.
- عدد شهادات معين.

## متطلبات الجودة

يجب ألا تؤثر Gamification على قواعد Completion الأساسية.

---

# 10. إهداء البرامج

**الأولوية:** P2  
**التقدير:** 1–2 أسبوع  
**الحالة:** يوجد أساس وظيفي في السورس.

## النطاق

- شراء برنامج كهدية.
- بيانات المستلم.
- إنشاء Gift Code.
- إرسال الكود.
- صفحة Redeem.
- Enrollment للمستفيد.

## الحالات التي يجب حسمها قبل الإطلاق

- المستفيد يمتلك البرنامج مسبقًا.
- الكود مستخدم مسبقًا.
- فشل الدفع.
- Refund.
- Expiration.
- Duplicate redemption.
- Email delivery.

---

# 11. Program Interest / Waitlist

**الحالة:** Core V1 implementation promoted / shipped for Launch V1

**الأولوية:** Launch V1 Core، والتحسينات المستقبلية P2

**التقدير:** التنفيذ الأساسي مشحون؛ تقدّر التحسينات المستقبلية بصورة مستقلة عند اعتماد نطاقها.

## ما شُحن في Launch V1

- Interest form للبرنامج المنشور عند إغلاق التسجيل أو اكتمال المقاعد.
- Contact data + Program link + PDPL consent.
- منع Spam والتكرار.
- Admin notification والقائمة المركزية.
- Program-scoped relation مع الحالة والتصفية وCSV export.
- Zapier event عند تفعيل التكامل.

## Future Enhancement

- Automated re-open notification.
- Advanced CRM follow-up.
- Lead scoring.
- Marketing automation journey.
- SMS / Push follow-up.

---

# 12. Advanced Student Profile

**الأولوية:** P2  
**التقدير:** 3–5 أيام

بعد استقرار الملف الأساسي يمكن إضافة:

- Bio.
- Skills.
- Interests.
- Social links.
- Portfolio.
- Professional information.

هذه الحقول لا يجب أن تدخل في قواعد التعليم أو الشهادات إلا إذا كان هناك سبب وظيفي واضح.

---

# 13. Advanced Notifications

**الأولوية:** P1/P2  
**التقدير:** 1–2 أسبوع

## التطوير

- Notification preferences.
- اختيار أنواع الإشعارات.
- Email preferences.
- Push readiness.
- Grouping.
- Better action links.
- Notification history.

لا يدخل Marketing Campaign Engine الكامل هنا؛ بل يبقى لمرحلة لاحقة.

---

# 14. Account Security Enhancements

**الأولوية:** P1  
**التقدير:** 3–7 أيام

تشمل:

- Active sessions.
- الأجهزة المستخدمة.
- إلغاء Session محددة.
- Logout from other devices.
- تحسين Security events.
- 2FA إذا تم اعتماده.
- Audit مبسط للأحداث المهمة.

---

# 15. Student Calendar & Personal Organization

**الأولوية:** P2  
**التقدير:** 3–7 أيام

يمكن أن يشمل:

- Quiz deadlines.
- Assignment deadlines.
- Program milestones.
- Calendar view.
- ICS export.
- Upcoming activities.

ويتم تأجيل التكاملات الخارجية المعقدة إلى مرحلة مستقلة.

---

# 16. نتيجة V1.1 المتوقعة

بعد V1.1 تصبح المنصة:

- أكثر تفاعلاً.
- أقوى في التواصل.
- أفضل في Retention.
- أفضل في Engagement.
- أكثر جاهزية للتوسع.
- مع الحفاظ على استقرار رحلة التعليم الأساسية.

---

# 17. Phase 2 — Learning & Operations Expansion

**الأولوية العامة:** P1 / P2  
**المدة المقترحة:** 8–12 أسبوع  
**التنفيذ:** يمكن تقسيمها إلى Releases صغيرة بدل إصدار ضخم واحد.

---

# 18. الجلسات المباشرة Live Sessions

**الأولوية:** P1  
**التقدير:** 2–4 أسابيع  
**الحالة:** مؤجلة بالكامل من V1.

## Phase 2A — Basic Live Sessions

- إنشاء الجلسة.
- الموعد.
- المدة.
- رابط Zoom / Meet / Teams.
- ظهور الجلسة للطالب.
- Reminder.
- الدخول للجلسة.
- ربطها بالبرنامج أو الدرس.
- حضور مرتبط بالجلسة.

## Phase 2B — Advanced Live Learning

يمكن لاحقًا إضافة:

- Internal provider integrations.
- Recording.
- Recording playback.
- Attendance automation.
- Polls.
- Breakout Rooms.
- Session analytics.

لا يتم تنفيذ هذه العناصر إلا عند وجود احتياج حقيقي لها.

---

# 19. المنتدى والمجتمع Community

**الأولوية:** P2  
**التقدير:** 3–5 أسابيع

## الهدف

تحويل النقاش من Lesson Q&A فقط إلى مساحة مجتمع تعليمي منظمة.

## النطاق المقترح

- Program discussions.
- Community posts.
- Replies.
- Pinning.
- Closing discussions.
- Announcements.
- Instructor/Admin moderation.
- Reporting abuse.
- Notifications.
- Search.
- Permissions.
- Community guidelines.

## شرط أساسي

لا يتم إطلاق Community بدون أدوات Moderation كافية.

---

# 20. الاستشارات

**الأولوية:** P2  
**التقدير:** 2–4 أسابيع  
**الحالة:** يوجد أساس متقدم في السورس لكنه خارج V1.

## الرحلة

```text
Consultation Request
→ Attachments
→ Admin Review
→ Approval / Rejection
→ Pricing
→ Payment
→ Tracking
→ Service Delivery
→ Closure
```

## قبل الإطلاق

يجب تحديد:

- أنواع الاستشارات.
- الأسعار.
- SLA.
- الفريق المسؤول.
- طريقة التسليم.
- سياسة الإلغاء والاسترداد.
- Notifications.

---

# 21. Advanced Reporting & Analytics

**الأولوية:** P1  
**التقدير:** 3–5 أسابيع

## الإدارة

Dashboard متقدم يعرض:

- Registrations.
- Revenue.
- Conversion.
- Enrollment.
- Completion rate.
- Quiz performance.
- Assignment performance.
- Attendance.
- Certificates.
- Support metrics.

## المدرب

- Students progress.
- At-risk students.
- Quiz averages.
- Assignment grading backlog.
- Completion.
- Attendance.

## الطالب

- Learning history.
- Progress breakdown.
- Completed activities.
- Performance trends.

---

# 22. Push Notifications

**الأولوية:** P2  
**التقدير:** 1–2 أسبوع للويب، وقد يتوسع مع التطبيق لاحقًا.

## الاستخدامات

- Assignment graded.
- New announcement.
- Certificate issued.
- Important learning reminder.
- Live session reminder لاحقًا.

يجب أن تحترم Push Notifications تفضيلات المستخدم.

---

# 23. SMS Integration

**الأولوية:** P2  
**التقدير:** 1–2 أسبوع بعد اختيار المزود.

## الاستخدامات المحتملة

- OTP.
- Critical reminders.
- Payment notifications.
- Important operational messages.

## الاعتماديات

- SMS Provider.
- Sender ID.
- Country regulations.
- Costs.
- Templates.
- Delivery reports.

لا يبدأ التنفيذ قبل اعتماد المزود والتكلفة.

---

# 24. Marketing Automation

**الأولوية:** P2  
**التقدير:** 2–4 أسابيع

## نطاق مقترح

- Newsletter management.
- Segmentation.
- Campaigns.
- Abandoned cart recovery.
- Enrollment follow-up.
- Program launch campaigns.
- Re-engagement.

ويجب الفصل بوضوح بين:

**Transactional messages** و**Marketing messages**.

---

# 25. CRM / Leads

**الأولوية:** P2  
**التقدير:** 2–3 أسابيع

يمكن توحيد:

- Contact leads.
- Program interest.
- Consultation leads.
- Corporate inquiries.
- Campaign source.

مع حالات مثل:

```text
New
Contacted
Qualified
Converted
Lost
```

---

# 26. التحسينات المتقدمة بنظام الحضور (Advanced Attendance)

**الأولوية:** P2  
**التقدير:** 2–4 أسابيع  
**ملاحظة التوحيد:** تم إدماج ماسح كاميرا الـ QR وتوليد الرمز وبوابة موظف الحضور (Attendance Operator Portal) رسمياً ضمن إطلاق V1؛ ولا تبقى ضمن خارطة الطريق المستقبلية سوى التحسينات المتقدمة التالية:

- دورات وساعات السماح والـ Attendance windows.
- حالات التأخير والأعذار الرسمية (Late / Excused workflows).
- التحضير الجماعي للدفعة (Bulk attendance operations).
- التقارير المتقدمة وتحليلات الحضور التفصيلية للبرامج والحضور التاريخي.
- تحليلات الحضور للمدرب والإدارة العليا.
- أنظمة منع الاحتيال والتأمين المتقدمة (Advanced Anti-abuse controls).
- سجلات التدقيق والتغيير في سجلات الحضور (Attendance Audit logs).

---


# 27. Phase 2 — ترتيب التنفيذ المقترح

الترتيب المفضل:

### Phase 2A

1. Advanced Analytics.
2. Live Sessions الأساسية.
3. Push Notifications.
4. Advanced Attendance.

### Phase 2B

5. Community / Forum.
6. Consultations.
7. CRM / Leads.
8. Marketing Automation.
9. SMS.

السبب:

التحليلات والجلسات والحضور ترتبط مباشرة بالتعليم، بينما Community والاستشارات والتسويق توسعات إضافية يمكن تنفيذها بعد استقرار النظام التعليمي.

---

# 28. Phase 3 — Product & Business Expansion

**الأولوية:** P2 / P3  
**الفترة:** بعد إثبات نجاح Web Platform والحصول على بيانات استخدام حقيقية  
**المدة:** تعتمد على المنتجات المختارة، وغالبًا 3–6 أشهر أو أكثر إذا تم تنفيذ عدة مسارات.

---

# 29. تطبيق الهاتف Flutter

**الأولوية:** P1/P2 بحسب استخدام العملاء  
**التقدير:** 8–12 أسبوع لـMVP جيد، و12–16 أسبوع لنسخة Production أوسع.

## المنصات

- Android.
- iOS.

## MVP المقترح

- Authentication.
- Home / Today.
- Programs.
- My Programs.
- Learning.
- Video/PDF/Resources.
- Quiz.
- Assignment.
- Progress.
- Notifications.
- Certificates.
- Support.

## المرحلة التالية للتطبيق

- Offline capabilities.
- Push.
- Biometrics.
- Downloads.
- Advanced media.
- Live Sessions.
- Community.

> لا ينصح ببدء التطبيق قبل استقرار APIs ورحلة Web الأساسية.

---

# 30. AI Assistant & AI Learning Features

**الأولوية:** P3  
**التقدير الأولي:** 3–6 أسابيع لأول Feature مفيدة.

بدل إطلاق AI عام، يفضل اختيار حالات استخدام محددة، مثل:

- تلخيص درس.
- اقتراح أسئلة مراجعة.
- Q&A على محتوى البرنامج.
- مساعد تعلم.
- Instructor content assistant.
- Quiz generation مع Review بشري.

## المتطلبات

- Provider.
- Cost control.
- Privacy.
- Content boundaries.
- Hallucination controls.
- Usage limits.
- Audit.
- Human review where needed.

---

# 31. Wamadat Plus Marketplace

**الأولوية:** P3  
**التقدير:** 8–12 أسبوع على الأقل لنسخة تشغيلية قوية.

هذه ليست Feature صغيرة؛ بل Product مستقل داخل المنصة.

قد تشمل:

- Sellers.
- Services.
- Marketplace orders.
- Commission.
- Payouts.
- Disputes.
- Messaging.
- Ratings.
- Refund policies.
- Finance reconciliation.

يجب التعامل معها كمشروع مستقل وليس كتحديث بسيط.

---

# 32. Subscriptions

**الأولوية:** P3  
**التقدير:** 3–5 أسابيع بحسب نموذج الاشتراك.

قبل التنفيذ يجب تحديد:

- Student subscription?
- Program subscription?
- Academy membership?
- Monthly / yearly?
- Auto renewal?
- Grace period?
- Cancellation?
- Refund?
- Entitlements?

لا يبدأ التنفيذ قبل اعتماد Business Model.

---

# 33. Bundles

**الأولوية:** P3  
**التقدير:** 2–4 أسابيع

تشمل:

- Bundle of programs.
- Bundle pricing.
- Discount calculation.
- Bundle enrollment.
- Individual completion.
- Order/invoice representation.

---

# 34. Affiliate

**الأولوية:** P3  
**التقدير:** 3–5 أسابيع

يشمل:

- Affiliate accounts.
- Links/codes.
- Attribution.
- Conversion tracking.
- Commission.
- Approval.
- Payout.
- Fraud controls.
- Reports.

---

# 35. Alumni

**الأولوية:** P3  
**التقدير:** 3–5 أسابيع لأول إصدار.

يمكن أن تشمل:

- Graduate status.
- Alumni directory.
- Alumni benefits.
- Networking.
- Events.
- Opportunities.

ولا تنفذ قبل وجود قاعدة خريجين واستخدام واضح.

---

# 36. B2B Portal

**الأولوية:** P3  
**التقدير:** 6–10 أسابيع حسب النطاق.

يمكن أن يشمل:

- Corporate accounts.
- Team enrollment.
- Seat management.
- Company admin.
- Employee progress.
- Corporate invoices.
- Reports.
- Bulk purchase.
- Training plans.

هذا Product مختلف عن شراء الفرد لبرنامج.

---

# 37. Multi-Academy / SaaS

**الأولوية:** P3 — Strategic  
**التقدير:** 8–16 أسبوع أو أكثر حسب مدى إعادة التفعيل المطلوبة.

البنية الداخلية الحالية يمكن أن تحتوي جذور Multi-Tenancy، لكن واجهة V1 مصممة لتظهر كمنصة ومضات واحدة.

إذا تم اتخاذ قرار تجاري بإعادة المنصة إلى SaaS، يجب تنفيذ مشروع مستقل يشمل:

- Tenant onboarding.
- Tenant isolation.
- Plans.
- Subscriptions.
- Tenant branding.
- Tenant domains.
- Tenant admin.
- Billing.
- Limits.
- Storage.
- Support model.
- Security review.
- Operational tooling.

ولا ينصح بخلط هذا المسار مع تطوير منصة ومضات الحالية قبل ثبوت الحاجة التجارية.

---

# 38. خارطة الطريق الزمنية المقترحة

## الشهر الأول

### الأسبوع 1–2
**Post-Launch Stabilization**

- Monitoring.
- P0/P1 fixes.
- Payments.
- Enrollment.
- Learning.
- Certificates.
- Support.

### الأسبوع 3–4
**بداية V1.1**

- Direct Messaging.
- Reviews.
- Account Security enhancements.

---

## الشهر الثاني

**استكمال V1.1**

- Streak.
- Achievements.
- Gifts.
- Program Interest follow-up enhancements (automation/CRM only).
- Advanced Profile.
- Advanced Notifications.
- Calendar enhancements.

ثم:

**V1.1 Release**

---

## الشهر الثالث والرابع

**Phase 2A**

- Advanced Analytics.
- Live Sessions.
- Push Notifications.
- Advanced Attendance.

إطلاق مرحلي، وليس دفعة واحدة.

---

## الشهر الخامس والسادس

**Phase 2B**

- Community.
- Consultations.
- CRM.
- Marketing Automation.
- SMS.

---

## ما بعد الشهر السادس

اختيار مسار أو أكثر بناءً على نتائج الأعمال:

- Mobile App.
- AI.
- Marketplace.
- B2B.
- Subscriptions/Bundles.
- SaaS.

ولا ينصح ببدء جميع هذه المشاريع بالتوازي.

---

# 39. ما يمكن تنفيذه بالتوازي

عند توفر فريق متعدد التخصصات يمكن العمل بالتوازي على:

### Track A — Core Platform
- Backend.
- Security.
- Payments.
- Data.

### Track B — Product UX
- Web UI.
- Student experience.
- Instructor experience.
- Admin experience.

### Track C — Engagement
- Messaging.
- Reviews.
- Notifications.
- Gamification.

### Track D — Future Products
- Mobile.
- AI.
- Marketplace.

لكن يجب الحفاظ على Release Gate موحد وعدم دمج تغييرات غير مستقرة في Production لمجرد اكتمالها داخل Track منفصل.

---

# 40. نموذج الإصدارات المقترح

بدل Deployment ضخم كل عدة أشهر، يوصى باستخدام Releases قصيرة:

```text
V1.0.0  → Public Launch
V1.0.1  → Hotfix / Stability
V1.0.2  → UX/Operational fixes

V1.1.0  → Engagement Release
V1.1.1  → V1.1 Hardening

V1.2.0  → Analytics / Attendance improvements
V1.3.0  → Live Learning

V2.0.0  → Major platform expansion when justified
```

---

# 41. Release Gate لكل ميزة

لا تعتبر أي Feature جاهزة للإنتاج إلا بعد المرور على:

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

---

# 42. تعريف Done

أي Feature تعتبر **Done** عندما:

- تحقق الهدف التجاري أو التعليمي المحدد.
- تعمل للRole الصحيح فقط.
- جميع الحالات الأساسية تعمل.
- الحالات السلبية الأساسية مختبرة.
- لا تسبب Regression للرحلات الحالية.
- لا تكسر Checkout أو Enrollment أو Learning.
- تمت مراجعة UI/UX.
- تمت مراجعة Production configuration.
- يوجد سجل واضح لقرار إطلاقها.

---

# 43. مؤشرات الأداء التي يجب مراقبتها

بدءًا من V1 يفضل قياس:

## Acquisition

- Visitors.
- Program views.
- Registration conversion.

## Commerce

- Cart → Checkout conversion.
- Checkout → Payment success.
- Payment failure rate.
- Coupon usage.
- Bank transfer approval rate.

## Learning

- Enrollment activation.
- Lesson engagement.
- Quiz participation.
- Quiz pass rate.
- Assignment submission rate.
- Program completion rate.

## Certification

- Certificates issued.
- PDF downloads.
- Verification usage.

## Support

- Tickets created.
- Average response time.
- Resolution time.
- Most common issues.

## Platform Health

- Error rate.
- Failed jobs.
- Queue delay.
- Webhook failures.
- Email delivery failures.

هذه البيانات يجب أن تؤثر على ترتيب Roadmap، لا أن تكون الخطة ثابتة مهما حدث.

---

# 44. قواعد تعديل الأولويات

يمكن تغيير ترتيب أي Feature إذا أثبت الاستخدام الحقيقي أن هناك حاجة أكبر.

مثال:

إذا تبين أن:

- الطلاب يحتاجون التواصل مع المدربين بشدة → Messaging يصعد.
- الجلسات المباشرة أصبحت مطلبًا أساسيًا → Live Sessions تصعد.
- معظم المبيعات تأتي من الشركات → B2B يصعد.
- استخدام الويب من الهاتف مرتفع جدًا → Mobile App يصعد.
- الدعم مثقل بأسئلة متكررة → Knowledge Base / AI Assistance قد يصعد.

خارطة الطريق هي **خطة قرار** وليست قيدًا جامدًا.

---

# 45. المخاطر الرئيسية

## توسع النطاق Scope Creep

إدخال Features كثيرة في Release واحد يزيد:

- الأخطاء.
- وقت الاختبار.
- Regression.
- تأخير الإطلاق.

### الإجراء

كل Feature جديدة يجب أن تدخل Release محدد.

---

## الخدمات الخارجية

مثل:

- Payment gateways.
- SMS.
- Email.
- Push.
- AI.

تعتمد على مزودين خارجيين.

### الإجراء

عدم اعتبار Feature جاهزة حتى يتم اختبار Production integration.

---

## بناء عدة Products في نفس الوقت

Mobile + Marketplace + B2B + SaaS + AI بالتوازي قد يشتت الفريق.

### الإجراء

اختيار Product Expansion واحد أو اثنين فقط في كل دورة استراتيجية.

---

## تضخم العمليات التشغيلية

Features مثل Marketplace وCommunity وConsultations تحتاج:

- Moderation.
- Support.
- Finance.
- Policies.
- Operations.

لذلك يجب أن يسبق إطلاقها تحديد Owner تشغيلي واضح.

---

# 46. القرارات التي يجب أن تعتمد قبل كل مرحلة

## قبل V1.1

- الأولوية بين Messaging وGamification.
- سياسة Reviews.
- سياسة Gifts.
- Notification preferences.

## قبل Phase 2

- Live provider strategy.
- Community moderation model.
- Consultation business model.
- SMS provider.
- Analytics requirements.

## قبل Phase 3

- هل الأولوية Mobile أم B2B أم Marketplace؟
- هل المنصة ستظل Single-Wamadat أم تعود إلى SaaS؟
- هل الاشتراكات جزء من نموذج الدخل؟
- ما هي حالات AI ذات القيمة الفعلية؟

---

# 47. التقديرات الزمنية — ملاحظة مهمة

المدد الواردة في هذه الوثيقة **تقديرات تنفيذية أولية وليست التزامًا تعاقديًا نهائيًا**.

تعتمد المدة الفعلية على:

- عدد أعضاء الفريق.
- العمل المتوازي.
- التغييرات المطلوبة على التصميم.
- جاهزية الخدمات الخارجية.
- بيانات Production.
- نتائج UAT.
- حجم التعديلات المطلوبة على الكود الموجود.
- وقت مراجعة العميل واعتماد القرارات.

قبل بدء أي Phase كبيرة يفضل تحويلها إلى:

- Scope تفصيلي.
- Backlog.
- Acceptance Criteria.
- Sprint Plan.
- Release Plan.

---

# 48. ملخص الأولويات

## P0 — الاستقرار

```text
Production stability
Payments
Enrollment
Learning
Certificates
Security
Data integrity
Queue / Scheduler
Critical support
```

## P1 — التجربة والتعليم

```text
Direct Messaging
Advanced Analytics
Live Sessions
Account Security
Advanced Notifications
```

## P2 — التفاعل والتشغيل

```text
Reviews
Gamification
Gifts
Program Interest automation / CRM enhancements
Community
Consultations
Push
SMS
Marketing Automation
CRM
Advanced Attendance
```

## P3 — التوسع الاستراتيجي

```text
Mobile App
AI
Marketplace
Subscriptions
Bundles
Affiliate
Alumni
B2B
Multi-Academy SaaS
```

---

# 49. الخطة التنفيذية الموصى بها

الخيار الأكثر اتزانًا هو:

```text
Launch V1
↓
1–2 Weeks Stabilization
↓
V1.1 — 4–6 Weeks
↓
Phase 2A — 4–6 Weeks
↓
Phase 2B — 4–6 Weeks
↓
Business Review
↓
Select Phase 3 Strategic Track
```

وبذلك لا يصبح التطوير سلسلة Features غير مترابطة؛ بل **مراحل واضحة، لكل منها هدف ونتيجة قابلة للقياس.**

---

# 50. النتيجة المستهدفة

## بعد V1

منصة تعليمية وتجارية أساسية تعمل من البداية إلى الشهادة.

## بعد V1.1

منصة أكثر تفاعلًا واحتفاظًا بالمستخدمين.

## بعد Phase 2

منصة تعليمية متقدمة تشمل التواصل والمجتمع والتحليلات والخدمات الموسعة.

## بعد Phase 3

منظومة تعليمية قابلة للتوسع إلى:

- تطبيقات هاتف.
- منتجات AI.
- Marketplace.
- B2B.
- SaaS.
- نماذج إيرادات إضافية.

---

# 51. الخلاصة

خارطة طريق ومضات لا تعتمد على مبدأ:

> "ما دام الكود موجودًا فلنطلقه."

بل على المبدأ التالي:

> **نطلق أولًا ما يحقق رحلة تعليمية وتجارية موثوقة، ثم نضيف ما يزيد التفاعل والقيمة، ثم نتوسع إلى Products جديدة عندما يثبت الاستخدام الحقيقي الحاجة إليها.**

هذا النهج يحقق:

- إطلاقًا أسرع وأكثر أمانًا.
- مساحة اختبار أصغر.
- أخطاء أقل.
- قدرة أفضل على قياس الاستخدام.
- Roadmap واضحة للعميل والفريق.
- تطويرًا مستدامًا بدل التوسع غير المنضبط.
- قدرة على ترتيب الاستثمار بناءً على بيانات حقيقية.

وتصبح الخطة العامة:

**Stability → Engagement → Learning Expansion → Product Expansion**

وهي المسار الموصى به لبناء منصة ومضات بصورة متزنة وقابلة للنمو.
