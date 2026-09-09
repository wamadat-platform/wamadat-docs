# منصة ومضات التعليمية
## سجل الميزات المؤجلة وحالة الحظر — Deferred Features Registry

> **حالة ما بعد الإغلاق (9 سبتمبر 2026):** `LIVING DEFERRED FEATURE REGISTER`. أي إعادة تفعيل تتبع `../governance/05-CHANGE-REQUEST-PROCESS.md` وRelease Gate.

**نوع الوثيقة:** سجل حالة (Status Registry) — مرجع للقراءة والتدقيق، وليس خطة تنفيذ  
**الإصدار:** 2.0 Registry  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**المنتج:** منصة ومضات التعليمية — Web Platform  
**النطاق:** Launch V1 Surface Lockdown  
**مصدر الحقيقة:** الكود السورس المعتمد (`wamadat-web/lib/features.ts`, `wamadat-backend/config/features.php`, `routes/api.php`, `routes/console.php`)  
**الهدف:** توثيق حالة كل ميزة مؤجلة عبر جميع الأسطح (Frontend / API / Admin / Scheduler)، وإثبات بقاء الكود والبيانات محفوظين دون أي كشف تشغيلي في V1.

---

## 1. المبدأ الحاكم

> **كل شيء مخفي ما لم يكن ضمن Launch V1 Allowlist** — بنموذج Fail-Closed:

```text
enabled = (RELEASE_SCOPE != v1) AND (FEATURE_X = true)
```

- غياب متغير البيئة أو خطأ في ضبطه ⇒ الميزة **مغلقة** افتراضيًا (الافتراض `v1` في الطرفين).
- الكود، الهجرات، وبيانات Production للميزات المؤجلة تبقى **محفوظة بالكامل** دون حذف أو Refactor.
- الإخفاء مثبت على أربعة مستويات مستقلة: روابط الواجهة، المسارات المباشرة (404 آمن)، تسجيل مسارات الـ API، أسطح Filament، والمهام المجدولة.

---

## 2. البوابات المركزية المعتمدة (Master Gates)

| الطبقة | المتغير | قيمة الإنتاج | مصدر التحقق |
| --- | --- | --- | --- |
| Frontend | `NEXT_PUBLIC_RELEASE_SCOPE` | `v1` | `lib/features.ts` → `IS_LAUNCH_V1` |
| Backend | `RELEASE_SCOPE` | `v1` | `config/features.php` → `$isLaunchV1` |

سلوك الفشل: أي Feature Flag فردية (`FEATURE_PLUS=true` مثلًا) لا تُظهر شيئًا ما دام `RELEASE_SCOPE=v1`.

---

## 3. السجل الرئيسي — الميزات المؤجلة (22 ميزة)

**الأعمدة:** الحالة العامة في V1، وسلوك كل سطح: `Enabled` / `Hidden` (لا يظهر في أي قائمة) / `Blocked` (المسار المباشر يعيد 404 آمن) / `Disabled` (مسار API غير مسجل أو محجوب) / `Dormant` (مهمة مجدولة مشروطة ببوابة الميزة) / `N/A`.

| # | الميزة | Frontend | API | Admin Panel | Scheduler | المرحلة الهدف |
|---|--------|----------|-----|-------------|-----------|----------------|
| 1 | Direct Messaging (Student ↔ Instructor) | Blocked | Disabled | Hidden | N/A | V1.1 |
| 2 | Program Reviews / Ratings | Blocked | Disabled | Hidden | Dormant (`emails:review-requests`) | V1.1 |
| 3 | Gamification (Streak / Achievements) | Blocked (`/dashboard/achievements`) | Disabled | Hidden | N/A | V1.1 |
| 4 | Program Gifts + Redemption | Blocked (`/redeem-gift`) | Disabled | Hidden | N/A | V1.1 |
| 5 | Program Interest / Waitlist | Hidden | Disabled | Hidden | N/A | V1.1 |
| 6 | Advanced Student Profile | Hidden | Disabled | N/A | N/A | V1.1 |
| 7 | Advanced Notification Preferences | Hidden | Disabled | N/A | N/A | V1.1 |
| 8 | Calendar | Blocked (`/dashboard/calendar`) | Disabled | N/A | N/A | V1.1 |
| 9 | Wallet | Blocked (`/dashboard/wallet`) | Disabled | Hidden | N/A | V1.1 |
| 10 | Wishlist | Blocked (`/dashboard/wishlist`) | Disabled | Hidden | N/A | V1.1 |
| 11 | Live Sessions | Blocked (Student + Trainer) | Disabled | Hidden (Lesson authoring + Relations) | Dormant (`emails:live-session-reminders`, `live-sessions:sync-lifecycle`) | Phase 2 |
| 12 | Community / Forum / Discussions | Blocked | Disabled | Hidden | N/A | Phase 2 |
| 13 | Consultations | Blocked (`/consultations`) | Disabled | Hidden (Site Settings) | N/A | Phase 2 |
| 14 | Push Notifications UI | Hidden | Disabled | N/A | N/A | Phase 2 |
| 15 | SMS | Hidden | Disabled | N/A | N/A | Phase 2 |
| 16 | Marketing Automation (Cart Recovery) | Hidden | N/A | N/A | Dormant (`emails:recover-abandoned-carts`) | Phase 2 |
| 17 | Newsletter UI | Blocked | Disabled | Hidden | N/A | Phase 2 |
| 18 | Wamadat Plus / Marketplace | Blocked (`/plus`) | Disabled | Hidden (Site Settings) | Dormant (`plus:process-sla`) | Phase 3 |
| 19 | Subscriptions | Blocked (`/subscriptions`) | Disabled | Hidden | N/A | Phase 3 |
| 20 | Bundles | Blocked (`/bundles`) | Disabled | Hidden | N/A | Phase 3 |
| 21 | Affiliate | Blocked (`/affiliate`) | Disabled | Hidden | N/A | Phase 3 |
| 22 | Alumni | Blocked (`/alumni`) | Disabled | Hidden | N/A | Phase 3 |

ميزات إضافية مغطاة بنفس البوابات دون مسار عام مخصص:

| الميزة | Frontend | API | المرحلة الهدف |
|--------|----------|-----|----------------|
| B2B / For Business | Blocked (`/for-business`) | Disabled | Phase 3 |
| AI Assistant | Blocked (`/ai-assistant`) | Disabled | Phase 3 |

---

## 4. نطاق V1 المسموح (Allowlist ملخص)

| الشخصية | الأسطح المفعّلة في V1 |
|---------|------------------------|
| Public / Visitor | Home، Programs، Program Details، Categories، Search، Instructors، About، Contact، Help، Legal، Sign Up/In، Password Reset، Certificate Verification |
| Commerce | Cart، Coupons، Checkout، بوابات الدفع المعتمدة، Bank Transfer، Orders، Payments، Invoices |
| Student | Today (منقّح)، My Programs، Learning كامل (Lessons/Notes/Q&A/Quiz/Assignments/Progress/Completion)، Attendance V1، Certificates + PDF + Verification، Orders & Payments، Support، Notifications الأساسية، Profile/Settings الأساسية والحقوق النظامية (PDPL) |
| Instructor | Instructor Home، Programs، Lessons، Students، Quiz، Assignments، Grading، Lesson Questions، Attendance V1، Certificates ضمن الصلاحيات |
| Admin | Dashboard، Programs، Lessons، Students، Enrollments، Instructors، Quizzes، Assignments، Orders، Payments، Bank Transfers، Invoices، Coupons، Issued Certificates، Support Tickets، Site Settings |

---

## 5. سجل المهام المجدولة (Scheduler Registry)

الحالة الفعلية في `routes/console.php` بعد تطبيق بوابات الميزات:

### 5.1 نشطة في V1 (غير مشروطة)

| المهمة | الإيقاع | التصنيف |
|--------|---------|---------|
| `ops:scheduler-heartbeat` | كل دقيقة | مراقبة صحة المجدل (Health) |
| `outbox:drain` | كل 5 دقائق | بريد معاملات أساسي |
| `orders:reconcile` | كل 5 دقائق | مطابعة مدفوعات (تعافي Webhooks مفقودة) |
| `webhooks:replay-queued` | كل دقيقتين | إعادة تشغيل Webhooks معلّمة يدويًا |
| `cohorts:sync-statuses` | كل ساعة | مزامنة حالة الدفعات (Idempotent) |
| `emails:cohort-start-reminders` | يوميًا 06:00 | تنبيه تعليمي معاملاتي — ضمن V1 |
| `emails:attendance-cards` | يوميًا 09:00 | بطاقات حضور البرامج الحضورية — ضمن V1 |
| `accounts:purge-deleted` | يوميًا 03:30 | حق النظامي PDPL/GDPR — لا يُخفى أبدًا |
| `ops:failed-jobs:purge` | أسبوعيًا الأحد 03:00 | نظافة تشغيلية |

### 5.2 خاملة (Dormant — مشروطة ببوابة الميزة)

| المهمة | بوابة الإيقاف | الميزة |
|--------|----------------|--------|
| `plus:process-sla` | `features.plus` | Wamadat Plus (#18) |
| `emails:review-requests` | `features.reviews` | Reviews (#2) |
| `emails:live-session-reminders` | `features.live_sessions` | Live Sessions (#11) |
| `live-sessions:sync-lifecycle` | `features.live_sessions` | Live Sessions (#11) |
| `emails:recover-abandoned-carts` | `features.marketing_automation` | Marketing Automation (#16) |

لا توجد مهمة مجدولة تعمل لميزة مؤجلة أثناء `RELEASE_SCOPE=v1`؛ ولا ترسل المنصة أي بريد ترويجي لميزة خاملة.

---

## 6. المسارات المحجوبة في V1 (Direct Access → 404 آمن)

الفحص المطبق: `isFeaturePathEnabled()` في `lib/features.ts` يفلتر الروابط الثابتة وروابط CMS والمسارات المباشرة معًا.

```text
# Student
/dashboard/live-sessions/*   /dashboard/messages   /dashboard/achievements
/dashboard/wallet            /dashboard/wishlist   /dashboard/calendar
/dashboard/gifts             /dashboard/reviews    /dashboard/plus

# Instructor
/trainer/messages            /trainer/live-sessions   /trainer/community

# Public
/consultations/*   /plus/*          /redeem-gift     /for-business
/subscriptions/*   /bundles/*       /affiliate/*     /alumni/*
/ai-assistant/*    /community       /forum           /sms
```

قيود استعلامية إضافية: `?mode=live` محجوب على `/programs` و `/search`، و `?type=forum` محجوب على `/programs`.

### مسارات SaaS مخفية نهائيًا (بكل السيناريوهات، لا علاقة لها بمراحل قادمة)

```text
/pricing   /plans   /academy   /academies   /create-academy   /tenants
```

---

## 7. مرجع متغيرات البيئة المعتمد للإنتاج

جميع القيم الافتراضية في Dockerfile/بيئة الإنتاج = `false`، مع `RELEASE_SCOPE=v1`:

```env
# Master Gates
RELEASE_SCOPE=v1
NEXT_PUBLIC_RELEASE_SCOPE=v1

# Deferred features (Backend) — القيم الافتراضية false لكل منها
FEATURE_SUBSCRIPTIONS FEATURE_BUNDLES FEATURE_PLUS FEATURE_CONSULTATIONS
FEATURE_BUSINESS FEATURE_AI_ASSISTANT FEATURE_SMS FEATURE_AFFILIATES
FEATURE_ALUMNI FEATURE_DIRECT_MESSAGING FEATURE_REVIEWS FEATURE_GAMIFICATION
FEATURE_GIFTS FEATURE_PROGRAM_INTEREST FEATURE_LIVE_SESSIONS FEATURE_COMMUNITY
FEATURE_NEWSLETTER FEATURE_MARKETING_AUTOMATION FEATURE_PUSH FEATURE_CALENDAR
FEATURE_WALLET FEATURE_WISHLIST

# Deferred features (Frontend) — البادئة NEXT_PUBLIC_ بنفس الأسماء
# استثناءان تسميان فقط في Frontend:
NEXT_PUBLIC_FEATURE_ADVANCED_PROFILE=false
NEXT_PUBLIC_FEATURE_ADVANCED_NOTIFICATION_SETTINGS=false
```

ملاحظات حماية:
- فلتر المحتوى `isReleaseContentVisible()` يمنع أقسام CMS/العروض المشغَّلة يدويًا من ذكر أي ميزة مؤجلة (قائمة Markers عربية وإنجليزية معتمدة).
- موارد الـ API تُصفّي الحقول نفسها (مثل `average_rating` و `reviews_count` تصفر عندما Reviews مغلقة).
- لوحة Filament تزيل أنواع الدروس Live وخيارات Site Settings المرتبطة بالميزات المغلقة.

---

## 8. إجراء إعادة تفعيل ميزة مستقبلًا (Re-Enable Procedure)

1. رفع `RELEASE_SCOPE` إلى قيمة غير `v1` (أو إصدار Scope جديد معتمد).
2. رفع Flag الميزة المستهدفة فقط (`FEATURE_X=true` + نظيرتها `NEXT_PUBLIC_FEATURE_X=true`).
3. مراجعة صف الميزة في الجدول أعلاه: تأكيد أسطح Admin و Scheduler المرتبطة بها.
4. تشغيل قائمة UAT ذات الصلة من `03-Wamadat-Launch-V1-Manual-UAT-Checklist.md`.
5. تحديث هذا السجل (الحالة + عمود المرحلة الهدف + سجل التغيير أدناه).

> لا يجوز تفعيل ميزة بمجرد رفع الـ Flag الفردي دون قرار Scope موثق.

---

## 9. الربط بالتحقق

- فحوصات القبول اليدوية: `qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md`.
- اختبارات آلية موجودة في السورس: Route E2E للـ Frontend، واختبارات API/Admin/Scheduler في Pest تغطي بوابات `config('features.*')`.
- نطاق الإصدار: `NEXT_PUBLIC_RELEASE_SCOPE=v1` + `RELEASE_SCOPE=v1` (راجع `technical/01` قسم 5).

---

## 10. سجل التغيير

| التاريخ | الإصدار | التغيير |
|---------|---------|---------|
| 24 أغسطس 2026 | 2.0 | تحويل الوثيقة من خطة تنفيذ (Implementation Plan) إلى سجل حالة (Status Registry) مطابق للكود المعتمد: 22 ميزة مؤجلة، 9 مهام نشطة، 5 مهام خاملة، ومسارات محجوبة موثقة من `lib/features.ts`. |
