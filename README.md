# منصة ومضات التعليمية — Documentation Baseline

**تاريخ آخر مراجعة:** 24 أغسطس 2026  
**نطاق المنتج:** Launch V1  
**حالة المستودع:** Current Authoritative Documentation Baseline

---

## 1. التوثيق المعتمد والنطاق الحالي (Authoritative Current Documents)

هذا المستودع يمثل **مصدر الحقيقة المعتمد** لمنصة ومضات التعليمية. جميع الوثائق الظاهرة في الـ `HEAD` تم تحديثها وإعادة مطابقتها مع السورس كود الحالي في `wamadat-backend` و `wamadat-web`.

### 📋 وثائق إطلاق V1 والاختبارات (QA & Scope Baseline)
1. [qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md) — النطاق الوظيفي والعمليات التشغيلية الرسمية لإطلاق V1 (يشمل البرامج الإلكترونية والبرامج حضورية ونظام موظف الحضور).
2. [qa-final/02-Wamadat-Product-Development-Roadmap.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/qa-final/02-Wamadat-Product-Development-Roadmap.md) — خارطة طريق التطوير المستقبلية (تستبعد ميزات V1 المنجزة).
3. [qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md) — قائمة اختبارات قبول المستخدم اليدوية الشاملة (تشمل اختبارات Attendance Operator، Student QR، البرامج الحضورية، و الـ Golden Journeys).
4. [qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md) — سجل الميزات المؤجلة وحالتها في واجهات المستخدم والـ Backend.
5. [qa-final/05-Wamadat-Launch-V1-UAT-Links-and-Test-Payment-Data.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/qa-final/05-Wamadat-Launch-V1-UAT-Links-and-Test-Payment-Data.md) — روابط بيئة الاختبار وبيانات بطاقات الدفع التجريبية المصرح بها.

### ⚙️ الوثائق التقنية والتشغيلية (Technical Architecture & Operations)
1. [technical/01-Current-Technical-Architecture.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/01-Current-Technical-Architecture.md) — المعمارية التقنية الحالية (Laravel 11 API / Next.js / MySQL / Redis / Attendance Portal).
2. [technical/02-Security-and-Access-Control.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/02-Security-and-Access-Control.md) — سياسات الأمان والأدوار الخمسة المعتمدة (Visitor, Student, Instructor, Attendance Operator, Admin).
3. [technical/03-Payments-and-External-Integrations.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/03-Payments-and-External-Integrations.md) — معمارية المدفوعات وبوابات الدفع الإلكتروني والتحويل البنكي والتكاملات الخارجية.
4. [technical/04-Production-Deployment-and-Operations.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/04-Production-Deployment-and-Operations.md) — دليل التشغيل والمراقبة والنشر الإنتاجي (Docker / Coolify / Queue Workers / Migrations).
5. [technical/05-Backup-Restore-and-Rollback.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/05-Backup-Restore-and-Rollback.md) — خطة النسخ الاحتياطي واستعادة البيانات والـ Rollback.
6. [technical/COURSE_BUILDER_CONTRACT.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/technical/COURSE_BUILDER_CONTRACT.md) — عقد واجهة بناء المناهج والبرامج التعليمية.

### 🎨 الهوية والتصميم والتكوين الهندسي (Brand System & Engineering Standards)
1. [engineering/coding-standards.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/engineering/coding-standards.md) — معايير وكتابة الكود والتنسيق الهندسي.
2. [brand-system/README.md](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/brand-system/README.md) — دليل نظام الهوية البصرية والتصميم (Design System).

### 📜 السجلات التاريخية للإنقاذ والحوادث (Incident History)
- [incidents/](file:///c:/Users/Smart%20Academy/Desktop/Projects/Wamadat/wamadat-docs/incidents/) — تقارير Postmortem للحوادث التاريخية. (هذه التقارير هي سجلات تاريخية وثابتة ولا تمثل بلاغات مفتوحة حالية).

---

## 2. سياسة الوثائق التاريخية (Historical Documentation Policy)

تمت تصفية وإزالة جميع ملفات التدقيق القديمة (Audit)، ومراحل التطوير السابقة (Beta, Phase 3, Wave Closeouts)، والتقريرات المؤقتة من الـ `HEAD` الخاص بالمستودع لمنع التشتت وتعارض المعلومات.

> **ملاحظة:** التاريخ الكامل للمستودع والتقارير التاريخية السابقة محفوظ بشكل دائم ومتاح عبر سجل **Git History**.
