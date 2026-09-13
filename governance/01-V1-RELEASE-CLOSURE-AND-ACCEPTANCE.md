# منصة ومضات التعليمية — إغلاق وقبول الإصدار الأول
## V1 RELEASE CLOSURE & ACCEPTANCE

**Release ID:** `WAMADAT-V1.0.0`  
**Baseline Date:** 9 سبتمبر 2026  
**Documentation Revision:** 13 سبتمبر 2026  
**Document Status:** `CLOSED BASELINE — APPROVED`  
**Release Family:** Launch V1  
**Product Surface:** Web Platform  

---

# 1. قرار الإغلاق

اعتبارًا من نقطة القطع هذه، يعتمد **Launch V1** كنسخة المنتج الأساسية المستقرة التي يفصل عندها العمل بين:

- ما تم تسليمه وتشغيله ضمن V1.
- الصيانة والإصلاحات المتعلقة بالتزام V1.
- التطويرات والتحسينات والميزات الجديدة بعد V1.

> **قرار الحوكمة:** أي عمل جديد لا يعالج عيبًا في نطاق V1 المعتمد أو Incident إنتاجيًا لا يعد «جزءًا ناقصًا من V1» تلقائيًا، بل يدخل عبر Product Backlog وChange Process ويُسند إلى Release لاحق.

---

# 2. أساس القبول

يستند الإغلاق إلى:

1. نطاق V1 الوظيفي المعتمد في `../qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md`.
2. معايير UAT والإطلاق في `../qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md`.
3. سجل الميزات المؤجلة في `../qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md`.
4. المعمارية والأمان والتشغيل والاستعادة في `../technical/`.
5. قبول العميل الحالي بأن المنصة وصلت إلى حالة استقرار تسمح بالانتقال من التثبيت إلى خطة التطوير.

هذه الوثيقة **لا تستبدل Evidence الاختبارات السابقة** ولا تدعي إعادة تنفيذ UAT عند توقيعها؛ بل تجمد النطاق والقرار الإداري عند نقطة الاستقرار.

---

# 3. النطاق المسلم في V1

## 3.1 الاكتشاف

- الموقع العام وهوية ومضات.
- Programs.
- Categories.
- Search.
- Program Details.
- Instructor Profiles.
- Program Interest / Waitlist capture للبرامج التي لا يتوفر فيها التسجيل المباشر، مع ربط الاهتمام بالبرنامج وبيانات التواصل.
- الصفحات التعريفية والقانونية الأساسية.

## 3.2 الحسابات والوصول

- التسجيل وتسجيل الدخول.
- استعادة الوصول للحساب.
- الملف الشخصي الأساسي.
- فصل الوصول حسب الدور.

## 3.3 التجارة والدفع

- Cart.
- Coupons.
- Checkout.
- تكامل الدفع الإلكتروني عبر **Tap Payments** و**Tabby** و**Tamara** ضمن عقد الدفع الموحد في V1.
- تفعيل كل مزود للمستخدم النهائي يعتمد على مفاتيح Production واعتماد الحساب وإعداد البيئة في وقت التشغيل.
- Bank Transfer workflow.
- Orders.
- Payments.
- Invoices.
- Webhook verification / idempotency / reconciliation ضمن العقد التقني الحالي لمنع الأثر المالي المكرر.

> **Baseline integration decision:** تكاملات Tap / Tabby / Tamara تعد جزءًا مسلمًا من V1 على مستوى المنتج والكود، بينما Availability الفعلية لكل قناة في يوم معين تبقى حالة تشغيلية مرتبطة بإعدادات Production واعتماد المزود.

## 3.4 التعلم

- Enrollment.
- My Programs.
- Units / Lessons.
- Video / PDF / Text.
- Resources.
- Lesson Notes.
- Lesson Q&A.
- Progress / Completion.

## 3.5 التقييم

- Quizzes / Attempts.
- Assignments / Submissions.
- Instructor Grading.

## 3.6 الحضور

- Student QR Card.
- Attendance Operator Portal.
- Camera QR Scanner.
- Session/enrollment/time validation.
- Present/Late وحماية duplicate scan ضمن نطاق V1 الموثق.

## 3.7 الشهادات

- Eligibility وفق قواعد البرنامج.
- Certificate issuance.
- PDF Certificate.
- QR.
- Public Verification.

## 3.8 التشغيل

- Transactional Notifications.
- Support Tickets.
- Student Dashboard.
- Instructor Dashboard.
- Admin Dashboard.
- Site Settings.
- إدارة Program Interest / Waitlist leads ومراجعتها/تصديرها ضمن العمليات الإدارية المعتمدة.
- Queue / Scheduler operational flows المسموح بها في V1.

## 3.9 التسويق والتحليلات

تعد التكاملات التالية جزءًا مسلمًا من V1:

- **Google Tag Manager (GTM)**.
- **Google Analytics 4 (GA4)**.
- **Meta Pixel**.
- **TikTok Pixel**.
- **Snapchat Pixel**.
- Ecommerce/conversion events لرحلة الشراء، بما فيها `view_item`, `select_item`, `add_to_cart`, `begin_checkout`, `add_payment_info`, و`purchase` حيث ينطبق.
- Attribution capture للمعرّفات والحملات التي تدعمها الرحلة الحالية، مثل UTM و`gclid` / `gbraid` / `wbraid`.

> معرّفات الحسابات والبكسلات وإعدادات Consent/Production تشغيلية وقابلة للتغيير؛ أما **قدرة التكامل نفسها** فهي ضمن Baseline V1.

---

# 4. المستخدمون والأدوار المعتمدة

النطاق المغلق يستخدم الأدوار التالية كما هي موثقة:

1. Visitor.
2. Student.
3. Instructor.
4. Attendance Operator.
5. Admin.

وأي Role أو صلاحية جديدة بعد هذا Baseline تعامل كتغيير Product/Security وتخضع لمراجعة مستقلة.

---

# 5. ما هو خارج V1 رسميًا

المرجع التفصيلي الوحيد لحالة الحظر هو `../qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md`.

ملخص الفئات المؤجلة:

### V1.1 Candidates

- Direct Messaging.
- Reviews / Ratings.
- Gamification.
- Gifts.
- Advanced Student Profile.
- Advanced Notification Preferences.
- Calendar.
- Wallet.
- Wishlist.
- Account Security enhancements كتحسين Product مرشح حيث لا تكون إصلاحًا أمنيًا عاجلًا.

### Phase 2 Candidates

- Live Sessions.
- Community / Forum.
- Consultations.
- Push UI/advanced notifications.
- SMS.
- Marketing Automation.
- Newsletter.
- Advanced Analytics / Attendance / CRM حسب Scope الجديد.

### أولوية تطوير مبكرة بعد V1

- **Mobile App (Flutter — Android / iOS):** قرار التقنية معتمد نهائيًا: التطبيق الجديد يبنى بـFlutter من قاعدة نظيفة، والكود القديم React Native/Expo لا يُعاد استخدامه ولا يدخل كأصل تنفيذي للمسار الجديد. يبقى تحديد MVP والـAPI gaps والموعد والقدرة الاستيعابية خاضعًا للـBacklog/Definition of Ready، لكن اختيار التقنية لم يعد موضوع Discovery.

### Strategic / Later Candidates

- Wamadat Plus / Marketplace.
- Subscriptions.
- Bundles.
- Affiliate.
- Alumni.
- B2B.
- AI.

> **حد المنتج النهائي:** ومضات منتج مخصص لأكاديمية ومضات فقط. لا توجد خارطة منتج متعددة الأكاديميات. أي بنية Tenancy قديمة داخل الكود تُعامل كتفصيل تقني موروث، وليست Capability أو اتجاهًا تجاريًا مؤجلًا.

وجود أساس كودي لأي عنصر مؤجل **لا يغيّر حالة Scope**.

---

# 6. الفرق بين Warranty/Maintenance وNew Development

| الحالة | التصنيف بعد V1 |
|---|---|
| وظيفة موثقة في V1 لا تعمل وفق Acceptance Criteria | Bug / Maintenance |
| انهيار خدمة، دفع، أمن، فقد بيانات، صلاحيات | Incident P0/P1 |
| تحسين تجربة وظيفة تعمل أصلًا | Enhancement |
| توسيع Rule أو Role أو Data contract | Change Request |
| وظيفة غير موجودة ضمن Baseline | New Feature |
| ميزة Deferred يراد فتحها | New Product Scope / Re-enable Project |
| Refactor لا يغير السلوك | Technical Debt / Engineering |

التصنيف التجاري/التعاقدي النهائي يتبع العقد بين الأطراف؛ هذه الحزمة تضبط التصنيف المنتجـي والتقني.

---

# 7. حالات معروفة وحدود الإغلاق

هذه الوثيقة لا تسجل عيبًا مفتوحًا بعينه ما لم يضاف إلى السجل الرسمي. القاعدة هي:

- أي P0 مفتوح يلغي أهلية `CLOSED/STABLE` إلى أن يُعالج.
- P1 غير مانع يمكن قبوله فقط إذا كان Owner/mitigation/target release موثقًا.
- P2/P3 لا تمنع الإغلاق ما لم يقرر Product Owner عكس ذلك.
- اعتماد V1 لا يعني خلو النظام من العيوب المستقبلية.

---

# 8. Baseline Freeze Rules

بعد اعتماد هذه الوثيقة:

1. لا يعدل V1 Feature Scope لتضمين ميزة أطلقت لاحقًا.
2. أي إضافة جديدة تحصل على Release ID جديد.
3. أي Incident بعد الإغلاق يسجل بتاريخ حدوثه، ولا يعاد كتابة تاريخ V1.
4. أي Feature مؤجلة تبقى Disabled/Hidden/Blocked حتى قرار Re-enable موثق.
5. المرجع التقني يمكن تحديثه إذا تغير Production topology، لكن يجب حفظ أثر القرار.

---

# 9. انتقال الملكية إلى Product Development

الإصدار التالي لا يحدد بمجرد اتباع ترتيب Roadmap السابقة. يبدأ العمل من:

```text
Production data + Client priorities + User feedback + Risks
                         ↓
                  Product Triage
                         ↓
                Approved Initiative
                         ↓
                     V1.x
```

المرجع: `03-PRODUCT-ROADMAP.md` و`04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md`.

---

# 10. Acceptance Record

## فريق التنفيذ — Smart Agency

**الممثل:** ______________________________  
**التاريخ:** ______________________________  
**Release/Manifest Reference:** ______________________________  
**القرار:** `CLOSED / CLOSED WITH NOTES / REOPENED`  
**الملاحظات:**  

__________________________________________________________________

**الاعتماد:** ______________________________

## ممثل العميل — Wamadat

**الممثل:** ______________________________  
**التاريخ:** ______________________________  
**القرار:** `ACCEPTED / ACCEPTED WITH NOTES / REJECTED`  
**الملاحظات:**  

__________________________________________________________________

**الاعتماد:** ______________________________

---

# 11. Closure Declaration

عند اعتماد الطرفين أو اعتماد الجهة صاحبة الصلاحية وفق العلاقة التعاقدية، تصبح الحالة:

```text
WAMADAT-V1.0.0
STATUS: CLOSED
BASELINE: FROZEN
OPERATIONS: ACTIVE
MAINTENANCE: ACTIVE
PRODUCT DEVELOPMENT: OPEN FOR V1.x
```

---

# 12. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | إنشاء سجل الإغلاق الرسمي لـLaunch V1 والانتقال إلى Product Development. |
| 2026-09-13 | 1.1 | مراجعة ما قبل التوقيع: تثبيت Single-Academy، رفع Mobile App إلى أولوية تطوير مبكرة، وتثبيت تكاملات الدفع والتسويق/التحليلات ضمن Baseline V1. |
| 2026-09-13 | 1.2 | اعتماد Program Interest / Waitlist ضمن V1 Delivered Baseline، وحسم Flutter كتقنية التطبيق الجديد مع إيقاف مسار React Native/Expo القديم. |

---
