# منصة ومضات التعليمية — خط الأساس الحالي للمنتج
## CURRENT PRODUCT BASELINE — WAMADAT V1.0.0

**Baseline Date:** 9 سبتمبر 2026  
**Document Status:** `FROZEN PRODUCT CAPABILITY BASELINE`  
**الغرض:** تقديم صورة واحدة مختصرة وواضحة لما يعتبر جزءًا من المنتج المنشور عند إغلاق V1.  

---

# 1. قاعدة القراءة

هذه الوثيقة **Capability Baseline** وليست قائمة جداول أو Routes أو Models. التفاصيل الأدق تبقى في الوثائق الأصلية:

- Functional: `../qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md`
- Deferred: `../qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md`
- Architecture: `../technical/01-Current-Technical-Architecture.md`
- Security: `../technical/02-Security-and-Access-Control.md`
- Payments: `../technical/03-Payments-and-External-Integrations.md`
- Operations: `../technical/04-Production-Deployment-and-Operations.md`
- Recovery: `../technical/05-Backup-Restore-and-Rollback.md`

إذا لم تذكر Capability هنا ولكنها موثقة بوضوح داخل Frozen V1 Scope، يبقى Frozen V1 Scope المرجع التفصيلي.

---

# 2. تعريف المنتج عند الإغلاق

منصة ومضات V1 هي منصة Web تعليمية وتشغيلية تغطي الرحلة التالية:

```text
Discover Program
→ Create/Access Account
→ Cart / Checkout / Payment or Bank Transfer
→ Order / Payment / Invoice
→ Enrollment
→ Learning Content
→ Quiz / Assignment / Attendance where applicable
→ Progress / Completion
→ Certificate
→ Verification
→ Support / Operations
```

الهدف الأساسي ليس تشغيل كل Module موجود في الشفرة، بل تشغيل رحلة تعليمية وتجارية حقيقية من الاكتشاف حتى الشهادة.

---

# 3. Product Surfaces

## 3.1 Public / Visitor

- Home/public pages.
- Program catalog and details.
- Categories and search.
- Instructor information.
- Authentication entry points.
- Legal/help/contact surfaces الأساسية.
- Public certificate verification.

## 3.2 Student

- Account/profile basics.
- Cart/checkout/order/payment/invoice visibility حسب التدفق.
- My Programs.
- Learning player and supported content types.
- Lesson Notes and Q&A.
- Quizzes.
- Assignments.
- Progress/Completion.
- Attendance QR where applicable.
- Certificates.
- Notifications الأساسية.
- Support Tickets.

## 3.3 Instructor

- Instructor home.
- Assigned programs/content visibility.
- Assigned students.
- Quiz/assignment related flows.
- Grading/submission review.
- Lesson Questions.
- Attendance within granted scope.
- Certificate-related access within permissions.

## 3.4 Attendance Operator

- Dedicated attendance sign-in/portal.
- Session selection.
- QR scan.
- Attendance result.
- No access to general Admin/Student/Instructor financial or learning administration surfaces.

## 3.5 Admin

- Programs/content administration.
- Students/enrollments/instructors.
- Quizzes/assignments.
- Orders/payments/bank transfer review/invoices/coupons.
- Certificates.
- Support.
- Site settings.
- Attendance-related administration within V1 scope.

---

# 4. Capability Map

| Domain | V1 Capability | Baseline State |
|---|---|---|
| Discovery | Programs, categories, search, instructors | IN |
| Identity | Registration/login/password recovery/basic profile | IN |
| Commerce | Cart, coupons, checkout | IN |
| Payments | Electronic payment abstraction + environment-enabled providers | IN |
| Manual Payment | Bank Transfer + receipt review | IN |
| Finance Ops | Orders, payments, invoices | IN |
| Enrollment | Access activation after valid conditions | IN |
| Learning | Units, lessons, video, PDF, text, resources | IN |
| Engagement in lesson | Notes + Lesson Q&A | IN |
| Assessment | Quiz + attempts | IN |
| Assessment | Assignments + submissions + grading | IN |
| Progress | Progress + Completion | IN |
| Attendance | Student QR + operator portal + scanner | IN |
| Certification | Issue + PDF + QR + public verify | IN |
| Notifications | Transactional/basic operational notifications | IN |
| Support | Student/Admin ticket lifecycle | IN |
| Administration | Core Admin Panel | IN |
| Advanced messaging/reviews/gamification/etc. | Deferred register | OUT |
| Live/community/consultations/etc. | Deferred register | OUT |
| Marketplace/subscriptions/B2B/AI/SaaS expansion | Strategic deferred | OUT |

---

# 5. Release Scope Guarding

وثيقة Deferred Registry تثبت أن نطاق V1 يعتمد Master Gates:

```text
Backend:  RELEASE_SCOPE=v1
Frontend: NEXT_PUBLIC_RELEASE_SCOPE=v1
```

والميزات المؤجلة تُحجب على أكثر من سطح: الواجهة، direct routes، API registration، Admin surfaces، Scheduler وفق كل Feature.

هذه الآلية جزء من **سلامة Scope** وليست مجرد إخفاء UI.

---

# 6. Data & Architecture Baseline

بحسب الوثائق التقنية الحالية:

- Backend Laravel API/Admin.
- Frontend Next.js.
- PostgreSQL مع بنية Multi-Tenancy موجودة تقنيًا، بينما نطاق المنتج المنشور Single-Wamadat.
- Redis لاستخدامات التشغيل مثل sessions/cache/queues وفق بيئة Production.
- Queue/Scheduler كعمليات مستقلة.
- Storage خارجي S3-compatible للوسائط/المستندات حسب الإعداد.
- Monitoring/logging and release operations موثقة بشكل منفصل.

> وجود Multi-Tenancy تقنيًا لا يعني أن Multi-Academy/SaaS UI داخل V1. نطاق SaaS مؤجل/محجوب.

---

# 7. Security Baseline

الحد الأدنى المغلق يشمل:

- Role-based boundaries للأدوار الخمسة.
- عدم تمكين مستخدم من الوصول إلى بيانات/وظائف غير مصرح بها.
- حماية المحتوى/المستندات الحساسة وفق نوعها.
- عدم حفظ الأسرار داخل Git.
- مراجعة permissions كجزء من Release Gate لأي Feature لاحقة.

أي توسع في Role/Permission/Data visibility يحتاج Security Review.

---

# 8. Payment & Financial Integrity Baseline

العقد الحالي يفصل بين Order / Payment / Enrollment / Invoice ويراعي:

- Webhook signature verification حسب المزود.
- Idempotency وعدم مضاعفة الأثر المالي عند تكرار Webhook.
- Bank Transfer كمسار إداري منفصل.
- إصدار الفاتورة بعد تحقق الحالة المالية المعتمدة.

حالة تمكين Tap/Tabby/Tamara أو غيرها في Production **بيئية وتشغيلية** وتتغير دون أن نعيد تعريف Product Capability نفسها.

---

# 9. Operations Baseline

الوثائق الحالية تعرف مسار نشر منضبط قائمًا على:

- Build images خارج VPS.
- Immutable image references / Git SHA.
- Release manifest يربط Backend/Web.
- Preflight + backup confirmation before migrations.
- Migrations controlled once per release.
- Health verification after deploy.
- Rollback/restore paths موثقة.

قد تتغير Topology المادية لاحقًا؛ أي تغيير فعلي يجب أن يحدث `technical/04` وأثر القرار في `10-DECISION-LOG.md`.

---

# 10. Deferred Capability Boundary

لا يحق لأي فريق اعتبار الميزة «موجودة» للمستخدم فقط لأن:

- Model موجود.
- API جزئي موجود.
- UI قديم موجود.
- Migration موجودة.
- Feature flag يمكن رفعها.

الميزة تصبح Product Capability فقط بعد:

```text
Approved Scope
→ Ready Gate
→ Delivery
→ Verification/UAT
→ Release Decision
→ Production Enablement
```

---

# 11. Baseline Change Policy

`WAMADAT-V1.0.0` لا يُعاد كتابته. بعد إضافة Capability جديدة:

- يسجل Release جديد مثل `V1.1.0`.
- تحدث Living Roadmap/Backlog.
- يحدث Current Product Baseline بنسخة جديدة إذا كان التغيير جوهريًا.
- يبقى هذا الملف شاهدًا على V1.0.0.

---

# 12. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | تجميد Product Capability Baseline للإصدار V1.0.0. |
