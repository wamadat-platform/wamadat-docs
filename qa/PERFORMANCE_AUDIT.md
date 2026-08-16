# تدقيق الأداء + المقياس (2026-07-17)

فرضية الحمل: 100 أكاديمية، 1000 مدرّب، 10000 طالب، 100000 برنامج/مستأجر، مليون مشاهدة. كل المسارات على DB المستأجر (لكل أكاديمية) — فـ«100k برنامج» أسوأ حالة لكل مستأجر.

**حكم جاهزية البيتا:** لا شيء من هذه حاجب لبيتا (أول عميل، بيانات صغيرة). كلّها منحدرات **قبل التوسّع** — موثّقة هنا بأدلّة + عتبة تفعيل، تُصلَح قبل بلوغ الحجم لا قبل الإطلاق. مسارات الطالب/API مُحمّلة مسبقًا ومفهرسة بعناية (تحقّق أدناه).

## النتائج (مرتّبة بنصف قطر الأثر)

### P1 (قبل التوسّع) — بحث الكتالوج العام: مسح كامل + regex لكل صفّ، فهرس GIN غير مُستخدَم
`ProgramCatalogService::applySearch` (~:123-137) يلفّ كل عمود في `translate(regexp_replace(...))` ويطابق بـ`ILIKE '%needle%'` بحرف بدل بادئ → **لا فهرس يُستخدم**؛ Postgres يمسح كل برنامج منشور ويشغّل regex مرّتين على 3 أعمدة نصّية (منها `description_ar` الكبير) **لكل صفّ**. نقطة عامّة غير مُصادَقة (`GET /catalog/programs?q=`). فهرس `fullText` المُنشأ في الهجرة ميّت (لا `@@ to_tsquery`).
- **العتبة:** يبدأ الألم الملموس ~5k-10k برنامج/مستأجر.
- **الإصلاح:** عمود مطبَّع مُولَّد + فهرس `pg_trgm gin_trgm_ops`، أو استخدام tsvector عبر `whereFullText()`.

### HIGH (قبل التوسّع) — N+1 في جداول Filament (~15 موردًا)
Filament 3 لا يُحمّل أعمدة العلاقات تلقائيًا: `UserResource` (`roles.name`, pivot)، `OrderResource` (`user`)، `ProgramResource` (`category`, و`getEloquentQuery` يزيل soft-delete scope بلا `with()`)، وكذلك Review/Invoice/Wallet/Gift/ProgramInterest/AffiliateConversion/SupportTicket/CohortBatch/AuditLog/TenantAuditLog/Plus*. بعض الموارد تُعوّض صحيحًا (`AbandonedCart`, `Tenant`, `Instructor`, `IssuedCertificate`, `BankTransfer`).
- **العتبة:** يظهر عند صفحة 25-100 صفًّا مع آلاف السجلّات.
- **الإصلاح:** سطر واحد لكل مورد: `getEloquentQuery(): Builder { return parent::getEloquentQuery()->with([...]); }`.

### HIGH (قبل التوسّع) — ترتيبات الكتالوج (popular/rating/price) بلا فهرس داعم
`ProgramCatalogService` (~:155-164) يرتّب على `students_count`/`average_rating`/`price_halalas`؛ جدول programs يفهرس `(status, published_at)`، `category_id`، `primary_instructor_id`، `type` فقط. `newest` مُغطّى؛ البقية لا.
- **الإصلاح:** فهارس مركّبة تطابق نمط القراءة: `(type, status, students_count DESC)`، `(type, status, average_rating DESC)`، `(type, status, price_halalas)`.

### MEDIUM — شهادة PDF تُبنى تزامنيًا (DomPDF) كل تنزيل، بلا كاش
`CertificateRenderer` (~:19-37) يبني QR SVG + DomPDF داخل الطلب (~300-800ms CPU يشغل عامل PHP-FPM كاملًا). المحتوى ثابت بعد الإصدار.
- **الإصلاح:** ابنِ مرّة عند الإصدار (أو أوّل تنزيل)، خزّن blob على قرص/S3 بمفتاح `certificate_id`، ثم بثّ المخزَّن.

## تم فحصه ومقبول (بالأدلّة)
- قائمة الكتالوج العامّة: pagination + eager-load (`primaryInstructor`, `instructorProfile.user`, `category`) + `whenLoaded`. لا N+1.
- لوحة الطالب (`MyEnrollmentsController`): eager-load للبرنامج + مدرّبه، محدود بالمستخدم.
- المراجعات/الرسائل: eager-load + `limit`، مفهرسة `(program_id, is_visible, rating)`.
- **نبض تقدّم المشاهدة (مسار المليون-مشاهدة):** مُقيَّد على العميل (10s + بوابة حركة 5s)، كتابة خادمية واحدة مفهرسة بقفل نادر عند الإكمال فقط. صحيح.
- فهرسة enrollments/orders: مغطّاة جيّدًا (`unique(user_id,program_id)`، `(program_id,completed_at)`، `order_number` فريد...).
- البريد/الإشعارات: مستمعون `ShouldQueue`؛ إصدار الشهادة عند الإكمال DB-only بلا PDF.
- الواجهة: الصفحات العامّة الساخنة server components بـISR (`revalidate=30-60`)؛ `'use client'` محصور بالصفحات التفاعلية فعلًا.

## التوصية
لا إصلاح أداء مطلوب قبل البيتا. أدرِج الأربعة أعلاه في «تحسينات قبل التوسّع» وراقب أحجام الجداول؛ نفّذ فهرس البحث أوّلًا عند اقتراب أي مستأجر من ~5k برنامج.
