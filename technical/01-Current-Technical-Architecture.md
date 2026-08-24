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

تعتمد منصة **ومضات** على بنية معمارية متميزة تفصل بين واجهة المستخدم العصرية (Next.js App Router) والـ Backend المطور بإطار عمل (Laravel 11 RESTful API)، مع لوحة تحكم إدارية متكاملة عبر (Filament 3 Panel).

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        User Interface Layer                            │
│   Next.js 14 (App Router) + TypeScript + TailwindCSS/CSS + i18n RTL   │
│   (Visitor / Student / Instructor / Attendance Operator surfaces)     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ HTTPS REST API
┌───────────────────────────────────▼────────────────────────────────────┐
│                        Application Layer (Backend)                     │
│               Laravel 11.x API / PHP 8.2+ / Modular Monolith           │
│   (IdentityAccess, Catalog, Learning, Commerce, Certification, etc.)   │
└────────┬──────────────────────────┬──────────────────────────┬─────────┘
         │                          │                          │
┌────────▼─────────┐       ┌────────▼─────────┐       ┌────────▼─────────┐
│ Database Layer   │       │ Cache & Queues   │       │ Storage Layer    │
│ MySQL 8.0+       │       │ Redis            │       │ S3 Object        │
│ (Primary Store)  │       │ (Queue Worker)   │       │ Storage          │
└──────────────────┘       └──────────────────┘       └──────────────────┘
```

---

## 2. المكونات التقنية الرئيسية (Technology Stack)

### 2.1 Backend Framework
- **الإطار:** Laravel 11.x (PHP 8.2+).
- **لوحة الإدارة:** Filament 3.x.
- **التوثيق وتصميمه:** Modular Monolith Architecture يضم الموديولات الحالية:
  - `IdentityAccess`: إدارة الهوية، الأسرار، والأدوار الخمسة.
  - `Catalog`: إدارة الكتالوج، البرامج، الوحدات، والدروس، ونمط التسليم (`ProgramMode`: Online vs In-Person).
  - `Learning`: إدارة التسجيلات (Enrollments)، تقدم الدروس، الملاحظات، والأسئلة (Lesson Q&A)، وخدمة Student QR.
  - `Commerce`: السلة، الكوبونات، الخصومات، الطلبات، المدفوعات، التحويل البنكي، والفواتير (ZATCA QR).
  - `Certification`: إدارة شروط الإكمال، طلب الشهادة، توليد PDF، والتثبت عبر QR.
  - `Attendance`: بوابة وموظف الحضور، ماسح الكاميرا، والتحقق من الجلسات الميدانية.

### 2.2 Frontend Framework
- **الإطار:** Next.js 14+ (App Router).
- **اللغة:** TypeScript.
- **التنسيق لدعم RTL:** Vanilla CSS / TailwindCSS / Lucide Icons / RTL Internationalization.
- **إدارة الحالة والـ API:** Fetch API / React Server Components / Client Action State Managers.

### 2.3 قواعد البيانات والتخزين والوظائف الخلفية (Data & Infrastructure)
- **قاعدة البيانات الأساسية:** MySQL 8.0+.
- **التخزين المؤقت والزمام الزمني (Cache & Lock):** Redis.
- **طابور المهام (Queue Workers):** Laravel Queue Worker / Horizon لخدمة إرسال البريد، معالجة إشعارات الدفع (Webhooks)، وتوليد شهادات PDF.
- **التخزين السحابي للملفات (Object Storage):** S3-compatible Object Storage لحفظ غلاف البرامج، المراجع التعليمية، إيصالات التحويل البنكي المراجعة، وشهادات PDF الصادرة.

---

## 3. توضيح بنية Multi-Tenancy مقابل نطاق V1

> **تنبيه معماري:** يحتفظ الـ Backend تقنيًا بالبنية التحتية الداخلية لدعم Tenancy architecture (`IdentityAccess`, `TenantScope`), لكن واجهة المستخدم والنموذج التشغيلي المعتمد لإطلاق Launch V1 يعرضان منصة ومضات التعليمية **لمستخدم ينتمي لمنصة واحدة فقط (Single-Wamadat Platform)** دون إظهار واجهات SaaS أو إنشاء أكاديميات متعددة للمستخدم النهائي.

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
