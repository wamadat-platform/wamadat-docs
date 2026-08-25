# منصة ومضات التعليمية — النسخ الاحتياطي واستعادة البيانات والـ Rollback
## 05-Backup Restore and Rollback

**نوع الوثيقة:** دليل التعافي والنسخ الاحتياطي واستعادة الخدمة  
**الإصدار:** 1.1 Baseline  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**المنتج:** منصة ومضات التعليمية (Wamadat Platform)  
**النطاق:** Launch V1  
**مصدر الحقيقة:** إعدادات السيرفر والبيئة المعتمدة

---

## 1. استراتيجية النسخ الاحتياطي (Backup Strategy)

تعتمد المنصة سياسة النسخ الاحتياطي المنتظم لضمان سلامة بيانات الطلاب والعمليات التجارية. الأداة الرسمية هي حزمة **`spatie/laravel-backup`** المثبتة في الـ Backend والمهيأة في `config/backup.php`.

### 1.1 قواعد البيانات (PostgreSQL Database Backups)
- **التغطية الإجبارية للقاعدتين (FOAT-B4):** الإعداد المعتمد `source.databases = [landlord, tenant]` — أي نسخة احتياطية كاملة يجب أن تُفرّغ القاعدتين الفعليتين معًا:
  - اتصال `landlord` → `pg_dump` لقاعدة `wamadat_landlord` (المستأجرون، الخطط، الاشتراكات، مستخدمو النظام).
  - اتصال `tenant` → `pg_dump` **واحد** لقاعدة `wamadat_tenants` يلتقط كل الـ Schemas التابعة (`tenant_<slug>`) دفعة واحدة.
  - نسخة تغطي قاعدة واحدة فقط تُعد غير صالحة للاستعادة.
- **أمر التشغيل:** `php artisan backup:run` — ينتج أرشيف Zip مضغوطًا (مستوى ضغط 9) يضم ملفات الـ Dump المسماة حسب الاتصال داخل مجلد `db-dumps`.
- **التشفير:** اختياري عبر متغير البيئة `BACKUP_ARCHIVE_PASSWORD`.
- **الجدولة والتخزين المعزول:** أمر النسخ **غير مجدول** ضمن scheduler المنصة (`routes/console.php`)؛ لذا يجب جدولته خارجيًا (Cron خارجي أو Coolify Scheduled Task) يوميًا. الأرشيف يُكتب أولًا على القرص المحلي (`destination.disks = [local]`) ثم **يجب** مزامنته إلى تخزين سحابي معزول (Off-site S3) — الاكتفاء بالقرص المحلي لا يحقق متطلبات التعافي.
- **النسخ التكتيكي قبل النشر (Pre-deploy Snapshot):** نسخة إجبارية (`php artisan backup:run` أو `pg_dump` يدوي لكلا القاعدتين) قبل أي عملية `php artisan migrate` في البيئة الإنتاجية.
- **التنظيف والمراقبة:** سياسة الاحتفاظ الافتراضية (7 أيام كاملة، 16 يومي، 8 أسبوعي، 4 شهري، 2 سنوي بسقف 5000MB)، وفحص صحة يومي (MaximumAgeInDays = 1) مع تنبيهات بريدية عند فشل النسخ أو وجود نسخة غير صحية.

### 1.2 الملفات والمستندات (Media & Storage Backups)
- **تخزين S3 المرفق:** تفعيل خاصية الـ Versioning والمزامنة اليومية لملفات إيصالات التحويل البنكي، صور البرامج، وشهادات PDF الصادرة.

---

## 2. إجراءات استعادة البيانات (Database Restore Procedure)

عند حدوث طارئ يقتضي استعادة نقطة زمنية سابقة لقاعدة البيانات:

1. **إيقاف تطبيق الـ Backend مؤقتاً (Maintenance Mode):**
   ```bash
   php artisan down --secret="wamadat-restore-drill"
   ```
2. **فك الأرشيف واستخراج ملفات الـ Dump:**
   ```bash
   unzip /path/to/backup.zip -d /tmp/restore
   # الملفات داخل مجلد db-dumps وأسماؤها حسب الاتصال: landlord.sql + tenant.sql
   ```
3. **استعادة قاعدة السجل المركزي أولًا (Landlord):** الترتيب مهم — `wamadat_landlord` هو سجل المستأجرين المرجعي:
   ```bash
   psql -U [user] -h [host] -d wamadat_landlord -f /tmp/restore/db-dumps/landlord.sql
   ```
4. **استعادة قاعدة المستأجرين (Tenants):** ملف الـ dump الواحد يعيد إنشاء كل الـ Schemas (`tenant_<slug>`) تلقائيًا:
   ```bash
   psql -U [user] -h [host] -d wamadat_tenants -f /tmp/restore/db-dumps/tenant.sql
   ```
   > **ملاحظة:** إذا كانت النسخة بصيغة Custom (`pg_dump -Fc`) فاستخدم `pg_restore` بدلًا من ذلك:
   > ```bash
   > pg_restore --no-owner --no-privileges -U [user] -h [host] -d wamadat_tenants /path/to/tenant.dump
   > ```
5. **التحقق من تطابق المستأجرين بعد الاستعادة:** عدد الـ Schemas المستعادة يجب أن يطابق سجل المستأجرين في الـ Landlord (حماية من تكرار سيناريو الانفصال الموثق في `incidents/2026-05-15-landlord-tenant-orphan.md`):
   ```sql
   SELECT nspname FROM pg_namespace WHERE nspname LIKE 'tenant_%' ORDER BY 1;
   SELECT slug, database_schema FROM tenants ORDER BY slug;
   ```
6. **تحديث الكاش والروابط:**
   ```bash
   php artisan cache:clear
   php artisan config:cache
   php artisan route:cache
   ```
7. **إعادة فتح الخدمة:**
   ```bash
   php artisan up
   ```

---

## 3. خطة التراجع والتراجع التكتيكي (Application Rollback Policy)

إذا ظهرت مشكلة عالية الخطورة (P0) فور النشر:

1. **الرجوع لنسخة الـ Git السابقة (Git Rollback):**
   ```bash
   git checkout [previous_stable_sha]
   ```
2. **التعامل مع التعديلات في قاعدة البيانات (Migrations Policy):**
   - إذا كانت التعديلات قابلة للتراجع:
     ```bash
     php artisan migrate:rollback --step=1
     ```
   - إذا كانت التغيرات غير قابلة للتراجع دون فقدان بيانات حيوية: يُعتمد مبدأ **Forward-Fix** بإصدار Patch سريع يعالج الخلل فوراً دون المساس بالهيكل التجاري السليم.
3. **إعادة بناء الـ Frontend والـ Backend:** إعادة بناء الواجهات والحاويات على الـ Commit المستقر السابق.

---

## 4. تمارين اختبار الاستعادة (Restore Drill Procedure)

يُنفذ التمارين الدورية للتحقق من سلامة النسخ الاحتياطية كل ثلاثة أشهر، ويتم فيه:
- تنزيل أحدث نسخة احتياطية من التخزين المعزول.
- استعادتها على بيئة اختبار معزولة (Staging/Local) وفق إجراء القسم 2 كاملًا.
- التحقق من أن القاعدتين استُعيدتا: `wamadat_landlord` + `wamadat_tenants` مع تطابق عدد الـ Schemas (`tenant_<slug>`) مع سجل المستأجرين.
- التأكد من إمكانية تسجيل دخول حسابات الإدارة والطالب وقراءة بيانات البرامج والطلبات والشهادات بدون أخطاء.
