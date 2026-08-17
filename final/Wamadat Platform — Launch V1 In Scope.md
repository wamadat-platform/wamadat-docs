# Wamadat Platform — Launch V1 In Scope

## 1. الغرض من الوثيقة

تحدد هذه الوثيقة بصورة صريحة ونهائية ما يدخل ضمن نطاق **الإصدار الأول التجاري Launch V1 لمنصة ومضات**.

أي عنصر غير مذكور هنا كجزء من Launch V1 لا يعتبر التزامًا للإطلاق، حتى لو كان له كود أو جداول أو Routes موجودة حاليًا في المشروع.

---

# 2. تعريف Launch V1

Launch V1 هو:

> منصة ومضات التعليمية الرسمية على الويب، مخصصة لكيان ومضات فقط، وتسمح بعرض البرامج، تسجيل الطلاب، شراء البرامج، التعلم، التقييم، إدارة الطلبات والمدفوعات، وإصدار الشهادات من خلال تجربة مستقرة واحترافية وقابلة للاستخدام التجاري الحقيقي.

الإطلاق الأساسي:

- Web.
- Responsive.
- Arabic-first.
- RTL.
- Production-ready.
- Single-Wamadat UX.

لا يعتبر Launch V1 إطلاقًا لمنصة SaaS متعددة الأكاديميات.

---

# 3. قاعدة Scope الأساسية

خلال مرحلة Launch V1 يسمح فقط بـ:

- Launch blockers.
- إصلاحات P0 وP1.
- إصلاح الأخطاء المرتبطة برحلات V1.
- UI/UX الضروري للإطلاق.
- Responsive وRTL.
- Production configuration.
- Payment integration.
- Storage configuration.
- Email integration.
- Queue/Scheduler.
- Database alignment المطلوب للإطلاق.
- Monitoring.
- Security fixes الضرورية.
- E2E.
- Soft Launch.

ولا يسمح بتوسيع النطاق بميزات جديدة.

---

# 4. Single-Wamadat Product Experience

يجب أن يرى المستخدم المنتج باعتباره:

> منصة ومضات للتدريب والتطوير

وليس SaaS.

يدخل ضمن النطاق:

- إزالة أي ظهور تجاري لمفهوم Tenant.
- إزالة أي ظهور للأكاديميات المتعددة.
- إزالة إنشاء أكاديمية.
- إزالة خطط الأكاديميات.
- إزالة اشتراكات الأكاديميات.
- إزالة SaaS pricing.
- إزالة Multi-Academy navigation.
- تثبيت هوية ومضات في الويب ولوحات التحكم.

تبقى داخليًا البنية الحالية:

```text
Tenant = wamadat
Schema = tenant_wamadat
```

ولا يتم عمل Refactor جذري للـMulti-Tenancy خلال Launch V1.

---

# 5. Feature Exposure Gate

يدخل ضمن Launch V1 تنفيذ طبقة واضحة للتحكم في ظهور الميزات.

يجب ألا يكون الإخفاء مقتصرًا على حذف رابط من Navbar فقط.

الميزة المؤجلة يجب أن تكون:

1. مخفية من Navbar.
2. مخفية من Footer.
3. مخفية من Student Dashboard.
4. مخفية من Admin Dashboard عند الحاجة.
5. غير قابلة للاستخدام عبر الوصول المباشر للواجهة.
6. العمليات الحساسة الخاصة بها محمية Server-side عند الحاجة.
7. الكود والجداول لا يتم حذفهما.

يجب أن تكون هناك Feature Flags واضحة للميزات المؤجلة الأساسية.

---

# 6. الموقع العام

يدخل ضمن النطاق:

- Homepage.
- About.
- Programs.
- Program Details.
- Categories.
- Instructors / Team.
- Contact.
- Help.
- Privacy Policy.
- Terms.
- Refund Policy.
- Security/Legal pages المطلوبة.
- Search.
- Certificate Verification.

---

# 7. Landing Page

إعادة تصميم Landing Page الحالية تدخل بالكامل ضمن Launch V1.

المطلوب:

- إعادة تصميم جذرية.
- هوية ومضات.
- Mobile-first.
- Responsive.
- RTL احترافي.
- تحسين Visual Hierarchy.
- تحسين Conversion.
- Hero احترافي.
- البرامج المميزة.
- فريق العمل/المدربين.
- المميزات.
- آلية التعلم أو التسجيل.
- Testimonials عند توفر محتوى حقيقي.
- FAQ.
- CTA.
- Footer احترافي.
- إزالة جميع عناصر SaaS وPlus والميزات المؤجلة.

لا يعتبر تعديل ألوان أو مسافات فقط كافيًا.

---

# 8. Authentication

يدخل ضمن Launch V1:

- Sign Up.
- Sign In.
- Email + Password.
- Logout.
- Forgot Password.
- Reset Password.
- Session handling.
- Account status validation.
- Profile.
- Password change.
- إعدادات الحساب الأساسية.

كما يدخل:

- حماية جلسات Admin.
- Email OTP المطلوب للوحات الإدارة الحالية إذا كان جزءًا من النظام الأمني الموجود.

---

# 9. Program Catalog

يدخل ضمن Launch V1:

- Programs.
- Categories.
- Program details.
- Instructor information.
- Program pricing.
- Published/unpublished state.
- Capacity إذا كانت مطلوبة من البيانات الحالية.
- Enrollment availability.
- Enrollment deadline عند استخدامها.
- Promo Badge إذا تم اعتماد migration الخاص بها.
- Program resources الأساسية.
- Program curriculum.

---

# 10. إدارة البرامج

لوحة الإدارة يجب أن تمكن الإدارة من إدارة:

- Programs.
- Categories.
- Instructors.
- Students.
- Curriculum.
- Lessons.
- Resources.
- Quiz.
- Assignments.
- Enrollments.
- Cohorts بالقدر اللازم للبرامج الحالية.
- Program availability.
- Program publishing.

---

# 11. أنواع المحتوى المعتمدة في V1

المحتوى الأساسي المعتمد:

- Video.
- PDF / Downloadable Resources.
- Quiz.
- Assignment.

يتم اختبار أي Content Type إضافي قبل السماح باستخدامه في البرامج الحقيقية.

---

# 12. Video Providers

المعتمد في Launch V1:

- YouTube.
- Bunny.

ويجب التأكد من:

- تشغيل الفيديو.
- صلاحيات المشاهدة.
- التعلم من داخل Lesson.
- عدم ظهور Provider غير مدعوم للمسؤول عند إنشاء المحتوى.

---

# 13. Learning Experience

تدخل رحلة التعلم الكاملة ضمن Launch V1:

```text
Enrollment
→ My Programs
→ Curriculum
→ Lesson / Live Sessions
→ Attendance (QR)
→ Progress
→ Lesson Completion
→ Quiz / Assignment
→ Program Completion
→ Certificate
```

ويشمل ذلك:

- Student learning dashboard.
- Curriculum navigation.
- Progress tracking.
- Lesson completion.
- Basic Student QR Attendance (تحضير الطلاب عبر QR).
- Resume learning.
- Access-control للطالب المسجل.

---

# 14. Quiz

يدخل ضمن Launch V1:

- عرض الاختبار.
- Start attempt.
- حفظ الإجابات.
- Submit.
- حساب النتيجة.
- Pass/Fail حيث ينطبق.
- منع الوصول غير المصرح.
- ربط النتيجة بالتقدم.

---

# 15. Assignments

يدخل ضمن Launch V1:

- إنشاء Assignment.
- ظهوره للطالب.
- Submission.
- عرض submissions للإدارة/المدرب.
- Grading.
- حالة الطالب.
- ربطه بالتقدم إذا كان البرنامج يعتمد عليه.

---

# 16. Progress

يدخل ضمن النطاق:

- تقدم الطالب داخل البرنامج.
- Lesson completion.
- Curriculum progress.
- Quiz/Assignment state.
- Program completion logic.

يجب ألا تصدر شهادة بسبب حالة تقدم خاطئة.

---

# 17. Certificates

تدخل رحلة الشهادة بالكامل ضمن Launch V1:

```text
Completion
→ Certificate Issue
→ PDF
→ QR
→ Public Verification
```

ويتم اختبار:

- بيانات الطالب.
- بيانات البرنامج.
- Certificate number.
- PDF.
- QR.
- Verification page.
- Final public domain.

---

# 18. Live Sessions — External Mode

الجلسات المباشرة تدخل فقط بالشكل البسيط:

- Zoom.
- Google Meet.
- Microsoft Teams.
- External meeting URL.

يشمل النطاق:

- إنشاء موعد.
- عنوان الجلسة.
- التاريخ والوقت.
- ربطها بالبرنامج.
- ظهورها للطالب.
- رابط الانضمام.
- ظهورها للمدرب عند الحاجة.

---

# 19. QR Attendance (تحضير الطلاب عبر QR)

يدخل نظام **QR Attendance الأساسي لتحضير الطلاب** صراحة ضمن نطاق Launch V1 ✅.

يشمل النطاق الأساسي:

- توليد وعرض رمز QR الخاص بالطالب / التحضير.
- مسح QR وتأكيد حضور الطلاب في الجلسات والفعاليات واللقاءات الأساسية.
- تسجيل وتحديث حالة التحضير بالاعتماد على QR في رحلة الطالب.

> **قاعدة Scope المعتمدة للمشروع:**
> - **QR Attendance الأساسي لتحضير الطلاب = In Scope ✅**
> - **ميزات QR المستقبلية والأجهزة المتقدمة غير المتعلقة برحلة التحضير الأساسية = يمكن تأجيلها لما بعد Launch V1.**

---

# 20. Cart

يدخل ضمن Launch V1:

- Add to Cart.
- Remove.
- Cart persistence بالقدر الحالي المطلوب.
- Program pricing.
- Coupon application.
- انتقال صحيح إلى Checkout.

---

# 21. Checkout

يجب دعم الرحلة:

```text
Program
→ Cart
→ Checkout
→ Payment Method
→ Order
→ Payment
→ Enrollment
```

ويجب منع:

- Duplicate orders غير المقصودة.
- Duplicate payment processing.
- Double enrollment.

---

# 22. Tap Payment

Tap يدخل بالكامل ضمن Launch V1.

المطلوب:

- Live credentials.
- Checkout.
- Payment creation.
- Return URL.
- Webhook.
- Signature/security verification.
- Successful payment.
- Failed payment.
- Duplicate webhook.
- Retry/idempotency.
- Refresh return page.
- Order reconciliation.

النتيجة المطلوبة:

```text
Paid Payment
→ Paid Order
→ Enrollment
→ Invoice
```

---

# 23. Bank Transfer

التحويل البنكي يدخل بالكامل ضمن Launch V1.

رحلة الطالب:

```text
Checkout
→ Bank Transfer
→ Bank Details
→ Order awaiting payment
→ Upload Receipt
```

رحلة الإدارة:

```text
Receipt
→ Review
→ Approve / Reject
```

وعند Approve:

```text
Order Paid
→ Enrollment
→ Invoice
```

يجب أن تكون الإيصالات داخل Private Storage.

---

# 24. Orders

يدخل ضمن Launch V1:

- إنشاء الطلب.
- Order number.
- Status.
- Student orders.
- Admin orders.
- Payment linkage.
- Enrollment linkage.
- Bank transfer linkage.
- منع المعالجة المكررة.

---

# 25. Invoices

يدخل ضمن Launch V1:

- إنشاء الفاتورة.
- بيانات العميل.
- بيانات البرنامج.
- السعر.
- الخصم.
- الضريبة عند انطباقها.
- Invoice number.
- PDF.
- Admin access.
- Student access عند توفر الواجهة الحالية.

---

# 26. Coupons

يدخل ضمن Launch V1:

- إنشاء الكوبون.
- صلاحية الكوبون.
- تطبيقه في Checkout.
- حساب الخصم الصحيح.
- منع استخدام كوبون غير صالح.

ولا يدخل تطوير Campaign Engine جديد.

---

# 27. VAT / ZATCA — Phase 1

إذا كانت ومضات خاضعة لضريبة القيمة المضافة:

يدخل ضمن Launch V1:

- VAT number.
- Tax configuration.
- Invoice tax fields.
- ZATCA Phase-1 QR الموجود حاليًا.

ولا يدخل ZATCA Phase 2 onboarding.

---

# 28. Email

Resend يدخل ضمن Launch V1.

يجب اختبار رسائل حقيقية على Production أو بيئة Production-like لـ:

- Welcome.
- Password Reset.
- Payment.
- Bank Transfer.
- Enrollment.
- Certificate.

لا يقبل Log أو Mock كدليل نجاح.

---

# 29. Outbox / Queue

الـQueue جزء أساسي من Launch V1.

يجب تشغيل Worker دائم.

والتحقق من:

- Email jobs.
- Notification jobs الضرورية.
- Payment-related jobs.
- Outbox.

لا يعتبر وجود الكود وحده كافيًا.

---

# 30. Scheduler

Scheduler يدخل ضمن Launch V1.

ويجب تشغيله بصورة دائمة.

يتم التحقق من Background Tasks الحالية التي يعتمد عليها V1، بما فيها:

- Reconciliation.
- Outbox.
- Cleanup.
- Scheduled processing.

---

# 31. Student Dashboard

الحد الأدنى المعتمد:

- Today أو الصفحة الرئيسية للطالب.
- My Programs.
- Assignments.
- Certificates.
- Orders.
- Profile.
- Settings.
- Support.
- Live Sessions الخارجية إذا كانت مستخدمة في البرنامج.

كل عنصر آخر يجب ألا يعتبر متطلبًا للإطلاق إلا إذا تم إضافته رسميًا للـScope.

---

# 32. Instructor Dashboard

يدخل Instructor Panel الحالي ضمن V1 ضمن نطاق تدريسي محدود.

المطلوب:

- Programs التي يدرسها.
- Lessons.
- Assignments.
- Quizzes.
- Live Sessions الخارجية.

لا يحتاج المدرب إلى وظائف مالية أو SaaS أو Marketplace.

---

# 33. Admin Dashboard

تدخل لوحة الإدارة ضمن Launch V1، لكن يجب تبسيطها.

المستخدم الإداري التجاري يجب أن يرى بصورة أساسية:

- Dashboard.
- Programs.
- Categories.
- Lessons.
- Students / Users.
- Instructors.
- Enrollments.
- Quizzes.
- Assignments.
- Orders.
- Payments.
- Bank Transfers.
- Invoices.
- Coupons.
- Certificates.
- Support.
- Site Settings.
- Landing/Home Content.

لا ينبغي أن تتحول لوحة V1 إلى لوحة تعرض كل Module موجود في السورس.

---

# 34. Site Settings

يدخل ضمن Launch V1:

- اسم المنصة.
- Logo.
- Site identity.
- Contact information.
- Social links عند الحاجة.
- Payment notification email.
- Footer.
- Basic integrations الضرورية للإطلاق.

---

# 35. UI/UX

إعادة تحسين UI/UX تدخل ضمن النطاق لـ:

- Public website.
- Landing page.
- Student Dashboard.
- Instructor Dashboard.
- Admin Dashboard.

الأولوية:

1. وضوح.
2. سهولة الاستخدام.
3. Mobile responsiveness.
4. RTL.
5. Consistency.
6. Accessibility الأساسية.
7. Empty/Error/Loading states.
8. عدم ظهور عناصر تقنية للمستخدم.

---

# 36. Responsive

الإطلاق يجب أن يكون صالحًا على الأقل لـ:

- Desktop.
- Laptop.
- Tablet.
- Mobile browser.

ويتم اختبار أهم الرحلات على أحجام شاشة حقيقية.

---

# 37. Production Database Alignment

قاعدة البيانات الحالية تحتوي بيانات حقيقية، لذلك تدخل ضمن النطاق:

- Backup قبل أي تعديل.
- مراجعة جدول migrations الفعلي في Production.
- مقارنة Production schema مع متطلبات Release Candidate.
- تطبيق migrations اللازمة لـV1 فقط.

ممنوع:

```text
migrate:fresh
db:wipe
reset
random seed
```

---

# 38. Migration Allowlist

يجب إنشاء قائمة صريحة للمigrations المسموح بتطبيقها على Production.

لا يجوز تشغيل جميع Pending Migrations بصورة عمياء.

قبل التنفيذ:

```text
Production migrations table
vs
Repository migrations
```

ثم تصنيف كل migration:

```text
Required for V1
Already Applied
Deferred
Unsafe / Needs Review
```

ثم تطبيق Required فقط.

---

# 39. Storage

الإعداد المعتمد للإنتاج:

```text
Public media  → Persistent object storage
Private files → Private object storage
```

في حالة Supabase Storage:

```text
assets          → PUBLIC
private-assets  → PRIVATE
```

ويجب ضبط Laravel فعليًا لاستخدام S3-compatible Supabase Storage في Production.

خصوصًا:

- `MEDIA_DISK`
- `MEDIA_PRIVATE_DISK`
- S3 endpoint.
- Public bucket.
- Private bucket.
- Credentials.

Bank Transfer receipts يجب ألا تدخل Public bucket.

---

# 40. Production Environment

يجب تدقيق:

```text
APP_ENV
APP_DEBUG
APP_URL
FRONTEND_URL

Database
Tenant resolver

CORS
Cookies
Sanctum

Storage

Resend

Tap

Queue
Scheduler

Sentry
```

ويجب التأكد من الاتصال بمشروع Supabase Production الصحيح.

---

# 41. Security Launch Hardening

يدخل ضمن V1:

- Production debug off.
- HTTPS.
- Secure cookies.
- Correct CORS.
- Correct Sanctum domains.
- Authentication/authorization validation.
- Bank receipt private storage.
- Webhook validation.
- Rate limiting الموجود.
- Prevention of obvious IDOR/access-control issues.
- عدم تعريض Out-of-Scope mutations للعامة.

---

# 42. Monitoring

يدخل ضمن Launch V1:

- Health endpoint.
- Sentry.
- Application errors.
- Queue health.
- Failed jobs monitoring.
- Payment webhook monitoring.
- Basic uptime check.

الواجهات التقنية لهذه الأدوات ليست بالضرورة ظاهرة للعميل.

---

# 43. CI / Release Candidate

قبل Production Launch يجب أن تكون Release Candidate معروفة ومثبتة.

المطلوب:

- Backend checks.
- Frontend checks.
- Essential tests.
- E2E.
- Release SHA.
- Clean deploy artifact.

ولا يتم إدخال Dependency أو Framework upgrades كبيرة.

إذا كانت GitHub Actions غير موجودة أو غير مفعلة في الريبو الفعلي، يجب تحديد Release Gate بديل صريح قبل الإطلاق.

---

# 44. E2E — Student

يجب اختبار مستخدم حقيقي:

```text
Homepage
→ Program
→ Sign Up / Sign In
→ Cart
→ Coupon if applicable
→ Checkout
→ Payment
→ Order
→ Enrollment
→ Learning
→ Quiz
→ Assignment
→ Progress
→ Completion
→ Certificate
→ Verification
```

---

# 45. E2E — Admin

يجب اختبار:

```text
Admin Login
→ Program
→ Student
→ Order
→ Payment
→ Bank Transfer
→ Enrollment
→ Assignment
→ Certificate
```

---

# 46. E2E — Instructor

يتم اختبار:

```text
Instructor Login
→ Assigned Programs
→ Lessons
→ Quiz / Assignment
→ Student submission
→ Live Session
```

---

# 47. Failure Scenarios

يجب اختبار الحالات الحرجة:

- Failed card payment.
- Payment canceled.
- Duplicate webhook.
- Refresh return page.
- Bank transfer rejected.
- Unauthorized lesson access.
- Invalid coupon.
- Expired session.
- Email failure.
- Worker stopped.
- Missing media.
- Mobile responsive checkout.

---

# 48. Soft Launch

قبل Public Launch:

- 1 Admin.
- 1 Instructor.
- 3–5 Students.

يستخدمون النسخة الفعلية.

ولا يتم فتح Public Launch مع P0 أو P1 معروف.

---

# 49. Backup / Rollback

قبل Release:

- Database backup.
- Storage validation.
- Release SHA.
- Previous release identifier.
- Rollback procedure.
- Environment snapshot/record.
- Migration record.

---

# 50. Launch Acceptance Criteria

Launch V1 يعتبر ناجحًا فقط عندما تكون:

```text
Single-Wamadat UX          PASS
Deferred Features Hidden   PASS
Landing/UI                 PASS
Responsive / RTL           PASS

Auth                       PASS
Programs                   PASS
Learning                   PASS
Quiz                       PASS
Assignments                PASS
QR Attendance              PASS
Progress                   PASS
Certificates               PASS

Cart                       PASS
Tap                        PASS
Bank Transfer              PASS
Orders                     PASS
Invoices                   PASS

Email                      PASS
Queue                      PASS
Scheduler                  PASS

DB Alignment               PASS
Storage                    PASS
Security                   PASS
Monitoring                 PASS

Production E2E             PASS
Soft Launch                PASS
Backup / Rollback          READY
```

---

# 51. Definition of Done

Launch V1 لا يعني:

> الانتهاء من كل ما في repository.

بل يعني:

> وجود نسخة مستقرة وآمنة واحترافية ومخصصة بالكامل لمنصة ومضات، يستطيع الطالب شراء برنامج والتعلم والحصول على شهادته، وتستطيع الإدارة تشغيل النشاط التعليمي والمالي اليومي دون الاعتماد على Feature غير مكتمل.