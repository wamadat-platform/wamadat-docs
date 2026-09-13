# منصة ومضات التعليمية — المعمارية التقنية الحالية
## 01-Current Technical Architecture

**نوع الوثيقة:** مرجع تقني معماري  
**الإصدار:** 1.1 Baseline  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**المنتج:** منصة ومضات التعليمية (Wamadat Platform)  
**النطاق:** Launch V1  
**مصدر الحقيقة:** الكود السورس المعتمد في `wamadat-backend` و `wamadat-web`

---

## 1. التوصيف المعماري العام (System Architecture Overview)

تعتمد منصة **ومضات** على بنية معمارية متميزة تفصل بين واجهة المستخدم العصرية (Next.js App Router) والـ Backend المطور بإطار عمل (Laravel 12 RESTful API)، مع لوحة تحكم إدارية متكاملة عبر (Filament 3 Panel)، وقواعد بيانات PostgreSQL ببنية Landlord/Tenants متعددة المستأجرين.

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        User Interface Layer                            │
│  Next.js 15 (App Router) + React 19 + TypeScript + TailwindCSS v4     │
│   (Visitor / Student / Instructor / Attendance Operator surfaces)     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ HTTPS REST API
┌───────────────────────────────────▼────────────────────────────────────┐
│                        Application Layer (Backend)                     │
│        Laravel 12.x API / PHP 8.3+ (Docker: PHP 8.4 FPM) Monolith      │
│   (IdentityAccess, Catalog, Learning, Commerce, Certification, etc.)   │
└────────┬──────────────────────────┬──────────────────────────┬─────────┘
         │                          │                          │
┌────────▼─────────┐       ┌────────▼─────────┐       ┌────────▼─────────┐
│ Database Layer   │       │ Cache & Queues   │       │ Storage Layer    │
│ PostgreSQL       │       │ Redis            │       │ S3 Object        │
│ wamadat_landlord │       │ (Queue Worker)   │       │ Storage          │
│ + tenant schemas │       └──────────────────┘       └──────────────────┘
└──────────────────┘
```

---

## 2. المكونات التقنية الرئيسية (Technology Stack)

### 2.1 Backend Framework
- **الإطار:** Laravel 12.x (`laravel/framework ^12.61.1`) على PHP `^8.3`.
- **بيئة التشغيل الإنتاجية:** حاوية Docker بصورة `php:8.4-fpm-alpine` (PHP-FPM 8.4) مع Nginx داخلي؛ تعمل نفس الصورة لخدمات الـ Web والـ Queue والـ Scheduler عبر متغير `APP_MODE`.
- **لوحة الإدارة:** Filament 3.x (`filament/filament ^3.3.54`).
- **التوثيق وتصميمه:** Modular Monolith Architecture يضم الموديولات الحالية:
  - `IdentityAccess`: إدارة الهوية، الأسرار، والأدوار الخمسة.
  - `Catalog`: إدارة الكتالوج، البرامج، الوحدات، والدروس، ونمط التسليم (`ProgramMode`: Online vs In-Person).
  - `Learning`: إدارة التسجيلات (Enrollments)، تقدم الدروس، الملاحظات، والأسئلة (Lesson Q&A)، وخدمة Student QR.
  - `Commerce`: السلة، الكوبونات، الخصومات، الطلبات، المدفوعات، التحويل البنكي، والفواتير (ZATCA QR).
  - `Certification`: إدارة شروط الإكمال، طلب الشهادة، توليد PDF، والتثبت عبر QR.
  - `Attendance`: بوابة وموظف الحضور، ماسح الكاميرا، والتحقق من الجلسات الميدانية.

### 2.2 Frontend Framework
- **الإطار:** Next.js 15 (`next ^15.5.23` — App Router) مع React 19 (`react ^19.2.8`).
- **اللغة:** TypeScript.
- **بيئة التشغيل:** Node.js 22 (صورة `node:22-bookworm-slim`) بمخرجات بناء `standalone`.
- **مدير الحزم:** pnpm (مثبَّت بإصدار محدد داخل الـ Dockerfile مع `pnpm-lock.yaml`).
- **التنسيق لدعم RTL:** TailwindCSS v4 / Radix UI / Lucide Icons / RTL Internationalization (`next-intl`).
- **إدارة الحالة والـ API:** Fetch API (ky) / React Server Components / TanStack Query + Zustand.

### 2.3 قواعد البيانات والتخزين والوظائف الخلفية (Data & Infrastructure)
- **محرك قاعدة البيانات:** PostgreSQL عبر driver `pgsql` في جميع الاتصالات (`config/database.php`).
- **بنية القواعد:** ليست Database واحدة، بل قاعدتان فعليتان على نفس الـ Cluster:
  - `wamadat_landlord` (connection: `landlord`) — السجل المركزي: المستأجرون (`tenants`)، الخطط، الاشتراكات (`tenant_subscriptions`)، مستخدمو النظام (`system_users`)، وسجل التدقيق المركزي (`audit_logs_central`). الاتصال `pgsql` هو Alias لهذا الاتصال حتى تعمل تدفقات Laravel الافتراضية (auth, queue, cache fallbacks) على قاعدة الـ Landlord.
  - `wamadat_tenants` (connection: `tenant`) — يستوعب Schema مستقلًا لكل مستأجر باسم `tenant_<slug>` (مثل `tenant_wamadat`)، ويُبدَّل `search_path` ديناميكيًا وقت التشغيل (`SwitchTenantDatabaseTask` / `SubdomainTenantFinder`) قبل أي وصول لبيانات المستأجر.
- **التخزين المؤقت والزمام الزمني (Cache & Lock):** Redis مشترك بقواعد مرقمة مثبتة: DB 0 للعمليات/default، DB 1 للـCache، DB 2 للـSessions، وDB 3 للـQueues.
- **طابور المهام (Queue Workers):** Laravel Queue Worker / Horizon لخدمة إرسال البريد، معالجة إشعارات الدفع (Webhooks)، وتوليد شهادات PDF.
- **التخزين السحابي للملفات (Object Storage):** Cloudflare R2 بعقد S3-compatible: bucket عام للوسائط، bucket خاص للمستندات الحساسة، وbucket خاص ومنفصل لنسخ التطبيق الاحتياطية.

### 2.4 بنية التسليم والإنتاج

- خادما تطبيق `Cloud VPS 4` يشغلان Laravel Web/API وNext.js وQueue Worker وScheduler عبر Coolify.
- خادم بيانات `Cloud VPS Plus 6` مع NVMe يشغل PostgreSQL وRedis وأدوات النسخ والمراقبة.
- تبني GitHub Actions صورتين مستقلتين في GHCR؛ وسم كل صورة هو Git SHA كامل وغير قابل للتبديل.
- صورة Backend واحدة تخدم `APP_MODE=web|queue|scheduler`، ولا تشغل migrations عند بدء الحاوية.
- يجمع `wamadat-platform` SHA مستقلًا للـBackend وSHA مستقلًا للـWeb تحت `wamadat@<release-id>`، ثم ينسق Coolify والتحقق وSentry.
- Cloudflare هي طبقة edge، وSentry للمراقبة، وResend للبريد. لا يتم build من السورس أو `git pull` على خوادم الإنتاج.

---

## 3. توضيح بنية Tenancy الموروثة مقابل حدود المنتج

> **تنبيه معماري:** الكود الحالي ما يزال يستخدم بنية **Schema-per-Tenant** فوق PostgreSQL عبر `spatie/laravel-multitenancy` مع `tenant_<slug>` وعزل للكاش والطوابير. هذه بنية تقنية موروثة وليست نموذج المنتج المستقبلي. القرار المنتجّي المعتمد هو **Single-Academy Wamadat** فقط؛ لا توجد واجهات أو خطة منتج لإنشاء/إدارة أكاديميات متعددة. أي تبسيط لهذه الطبقة لاحقًا يجب أن يُدار كمبادرة Architecture/Technical Debt مستقلة وبخطة Migration آمنة.

---

## 4. الموديولات التشغيلية الخاصة بإطلاق V1

### 4.1 مسارات التعلم التفاعلي والبرامج الحضورية
1. **البرنامج الإلكتروني (Online Program):** تسلسل الدروس إلكترونيًا عبر الفيديو / PDF / النصوص، مع اختبارات وواجبات.
2. **البرنامج الحضوري (In-Person Program):** واجهة مخصصة للمعلومات الميدانية، المراجع، التكليفات الميدانية، وتوثيق الحضور الميداني بملف الطالب (`training-program-view`).

### 4.2 نظام حضور الطلاب وبوابة موظف الحضور (Attendance System)
- **Student QR Code:** توليد رمز QR فريد لكل طالب عبر `StudentQrController` وتأمينه ضد التزييف.
- **Attendance Operator Portal:** بوابة منفصلة ومحمية (`/attendance/sign-in`) تستخدم Middleware مخصص (`RestrictAttendanceOperatorApi`) يقيد الصلاحيات فقط لمسح بطاقة الطالب بالكاميرا والتأكد من الجلسة المفتوحة ومسح الـ QR لحظيًا.

### 4.3 نظام الشهادات والتحقق
- تقييم شروط الإكمال عبر `ProgramCompletionChecker`.
- توليد ملف PDF وتخزينه سحابيًا وتوليد كود التثبت (Verification Code).
- صفحة تثبت عامة تتيح لأي جهة خارجية قراءة صحة الشهادة عند مسح رمز الـ QR.

---

## 5. ضوابط إخفاء الميزات المستقبلية (Release Scope & Gating)

تعتمد المنصة على Master Release Gate موحد على مستوى الـ Frontend والـ Backend:

```env
# Frontend (.env)
NEXT_PUBLIC_RELEASE_SCOPE=v1

# Backend (.env)
RELEASE_SCOPE=v1
```

الوظائف البرمجية ذات الشفرة الموجودة تاريخيًا في السورس والتي تم حجبها في Launch V1 تشمل:
- البث المباشر المتقدم والجلسات المباشرة التفاعلية (Live Sessions Product).
- المراسلات المباشرة الخاصة بين الطالب والمدرب (Direct Messaging).
- تقييمات البرامج العامة (Program Reviews).
- واجهات Wamadat Plus, Marketplace, Consultations, Subscriptions, Bundles, AI Assistant.

تظل الشفرة المصدرية وقواعد البيانات لهذه الميزات محفوظة ومحمية بـ Gating دون إظهارها في أي واجهة إدارية أو عامة.
