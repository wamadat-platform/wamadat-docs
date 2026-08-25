# منصة ومضات التعليمية — النسخ الاحتياطي والاستعادة والتراجع
## 05-Backup Restore and Rollback

**نوع الوثيقة:** دليل التعافي واستعادة الخدمة
**الإصدار:** 1.2 — PASS 3 Production Delivery
**تاريخ المراجعة المعتمد:** 25 أغسطس 2026
**النطاق:** Launch V1

---

## 1. طبقات الحماية من فقدان البيانات

### 1.1 نسخة Laravel/Application

تستخدم `spatie/laravel-backup` الاتصالين معًا:

- `landlord` لقاعدة سجل المستأجرين والخطط والنظام.
- `tenant` لقاعدة المستأجرين بكل schemas من نمط `tenant_<slug>`.

يشغل Coolify Scheduled Task الأمر `php artisan backup:run`. الوجهة الرسمية
`BACKUP_DISK=s3-backup`، وهو قرص S3-compatible يشير إلى bucket خاص ومنفصل في
Cloudflare R2. يتبع `backup.destination.disks` و`monitor_backups.disks` القيمة
نفسها. القرص المحلي المؤقت ليس وجهة إنتاج نهائية.

يجب فحص `backup:list`/`backup:monitor` والتنبيهات، وتطبيق سياسة الاحتفاظ
المعتمدة. قبل كل هجرة إنتاجية يجب أن تنجح نسخة pre-deploy وأن يؤكدها المشغّل؛
عدم التأكيد يوقف release workflow.

### 1.2 PostgreSQL Disaster Recovery

WAL archiving وPITR عبر pgBackRest أو WAL-G طبقة مستقلة تُجهز على VPS-03 في
مرحلة البنية التحتية. لا تعوضها نسخة Laravel وحدها ولا تُنفذ داخل التطبيق.

### 1.3 VPS snapshots

Snapshot الخادم طبقة إضافية لتسريع تعافي العتاد، وليس استراتيجية النسخ الوحيدة
ولا بديلًا عن R2 أو WAL/PITR. وسائط R2 تحتاج versioning/retention مناسبين أيضًا.

## 2. تحقق النسخة قبل الاعتماد

1. التأكد من وجود dump للاتصالين `landlord` و`tenant` داخل الأرشيف.
2. التأكد أن bucket النسخ خاص ومنفصل عن media buckets.
3. مراجعة عمر آخر نسخة ونتيجة `backup:monitor`.
4. تنفيذ restore drill ربع سنوي في بيئة معزولة، لا في الإنتاج.
5. مقارنة سجل `tenants.database_schema` مع schemas المستعادة وفحص تسجيل الدخول
   والبرامج والطلبات والشهادات.

## 3. إجراء الاستعادة الطارئة

الاستعادة قرار حادث موثق يتطلب إيقاف الكتابة وعزل الهدف. لا تُشغّل أوامر المثال
على الإنتاج دون خطة حادث ومراجعة القيم الفعلية.

1. وضع التطبيق في maintenance mode وإيقاف Workers/Schedulers.
2. تحديد recovery point: نسخة التطبيق أو PITR بحسب طبيعة الحادث.
3. استعادة Landlord أولًا بأدوات PostgreSQL (`psql`/`pg_restore`).
4. استعادة قاعدة Tenant التي تحتوي كل schemas.
5. التحقق من تطابق schemas مع سجل المستأجرين.
6. تشغيل `php artisan ops:schema-verify` ثم فحوصات الصحة.
7. إعادة الخدمات تدريجيًا ومراقبة Sentry والسجلات والطوابير.

مثال مع placeholders لملف SQL فقط:

```text
psql -U <user> -h <host> -d <landlord-db> -f <landlord.sql>
psql -U <user> -h <host> -d <tenant-db> -f <tenant.sql>
```

للصيغة custom استخدم `pg_restore --no-owner --no-privileges`. لا تحفظ كلمات
المرور داخل الأوامر أو المستودع.

## 4. تراجع التطبيق (Application Rollback)

تراجع التطبيق يعني إعادة نشر صور GHCR السابقة ذات SHA الكامل للـBackend
والـWeb عبر `rollback-production.yml`. تُنشر صورة Backend السابقة إلى أدوار
web/queue/scheduler وصورة Web السابقة، ثم تُعاد فحوصات الصحة.

لا يستخدم مسار التراجع `git checkout` على الخادم، ولا يعيد build، ولا يشغّل
`php artisan migrate:rollback` تلقائيًا. قاعدة البيانات قد تكون متوافقة مع
الإصدار السابق أو قد تحتاج forward-fix أو استعادة/PITR؛ يحدد ذلك قائد الحادث
بعد مراجعة migrations وخطر فقدان البيانات.

## 5. مصفوفة القرار

| الحالة | الإجراء |
| --- | --- |
| عيب تطبيق والصيغة الحالية متوافقة | إعادة نشر الصور السابقة |
| عيب يمكن إصلاحه سريعًا دون مخاطرة | Forward-fix بصورة immutable جديدة |
| هجرة غير متوافقة | قرار يدوي: forward migration أو استعادة/PITR |
| فساد/فقدان بيانات | maintenance mode ثم restore/PITR وفق خطة الحادث |
| فشل خادم فقط | إعادة جدولة الحاويات؛ snapshot طبقة مساعدة |

كل تراجع أو استعادة يسجل release IDs وSHAs ووقت الحادث وrecovery point ونتائج
التحقق. لا يعد workflow ناجحًا ما لم يتم النشر والتحقق فعليًا.
