# النسخ الاحتياطيّ والاستعادة (DR) — Runbook

> دليل تشغيليّ مختصر لنسخ قواعد البيانات واستعادتها، وإثبات أنّ النسخ **قابلة
> للاستعادة فعلًا**. آخر تمرين استعادة ناجح: **2026-06-15** (بند P12 مُغلق).

## البيئة
- PostgreSQL 16 على `127.0.0.1:5432`، مستخدم `postgres`.
- قاعدتان: **`wamadat_landlord`** (المستأجِرون/الخطط/مستخدمو النظام) +
  **`wamadat_tenants`** (بيانات المستأجِر داخل schema مثل `tenant_wamadat`).
- الأدوات: `C:\Program Files\PostgreSQL\16\bin\` (pg_dump / pg_restore / psql).

## 1) أخذ نسخة احتياطيّة (يوميًّا)
صيغة `custom` (‎-Fc‎) مضغوطة وقابلة للاستعادة الانتقائيّة:
```bash
set PGPASSWORD=<كلمة مرور postgres>
cd "C:\Program Files\PostgreSQL\16\bin"
pg_dump -h 127.0.0.1 -U postgres -Fc -d wamadat_tenants  -f D:\backups\tenants_%DATE%.dump
pg_dump -h 127.0.0.1 -U postgres -Fc -d wamadat_landlord -f D:\backups\landlord_%DATE%.dump
```
- **مهمّ (البيتا على جهاز تطويريّ بلا تكرار):** انسخ ملفّات `D:\backups` **خارج
  الجهاز** فورًا (OneDrive / قرص خارجيّ). نسخة على نفس القرص = ليست نسخة.
- جدوِلها يوميًّا عبر **Windows Task Scheduler** (أو NSSM) لتشغيل سكربت يحوي
  الأمرين أعلاه + رفعها للسحابة.

## 2) تمرين الاستعادة (شهريًّا — يثبت أنّ النسخة تعمل)
استعادة إلى قاعدة **تجريبيّة منفصلة** (لا تلمس الإنتاج إطلاقًا) ثمّ مقارنة العدّادات:
```bash
# 1. أنشئ قاعدة تجريبيّة
createdb -h 127.0.0.1 -U postgres wamadat_tenants_drill
# 2. استعد النسخة فيها
pg_restore -h 127.0.0.1 -U postgres -d wamadat_tenants_drill --no-owner --no-privileges tenants_<date>.dump
# 3. تحقّق أنّ العدّ يطابق الإنتاج
psql -h 127.0.0.1 -U postgres -d wamadat_tenants_drill -tAc "select count(*) from tenant_wamadat.users;"
psql -h 127.0.0.1 -U postgres -d wamadat_tenants      -tAc "select count(*) from tenant_wamadat.users;"
# 4. نظّف
dropdb -h 127.0.0.1 -U postgres wamadat_tenants_drill
```
**النجاح = العدّادات في القاعدة التجريبيّة تساوي الإنتاج تمامًا.**

## 3) الاستعادة الحقيقيّة عند الكارثة
```bash
# على خادم/قاعدة جديدة نظيفة:
createdb -h <host> -U postgres wamadat_tenants
pg_restore -h <host> -U postgres -d wamadat_tenants --no-owner --no-privileges tenants_<latest>.dump
# كرّر للّاندلورد. ثمّ صحّح APP_URL/DB_* في .env، وشغّل: php artisan migrate --database=landlord --force
```

## سجلّ التمارين
| التاريخ | القاعدة | الحجم | النتيجة |
|---------|---------|-------|---------|
| 2026-06-15 | tenants | 742K | ✅ استعادة في 2ث، طابقت 236 مستخدم/30 برنامج/85 تسجيل/4 طلبات/شهادة/100 جدول |
| 2026-06-15 | landlord | 37K | ✅ استعادة بلا أخطاء، طابقت 1 مستأجِر/2 مستخدم نظام |

## التمرين الشهريّ — مُؤتمَت ✅
تمرين الاستعادة يعمل **تلقائيًّا شهريًّا** عبر Windows Task Scheduler:
- **المهمّة:** `Wamadat DR Restore Drill` — يوم 16 من كل شهر، 9:07ص.
- **السكربت:** `backend/scripts/dr-drill.ps1` (غير مُتلف: نسخ → scratch → استعادة →
  تحقّق العدّ → حذف scratch). يقرأ كلمة المرور من `.env` وقت التشغيل (لا سرّ مخزَّن فيه).
- **النتيجة تُسجَّل** في `backend/storage/logs/dr-drill.log` بصيغة `PASS/FAIL` + العدّ.
- إدارة المهمّة: `schtasks /query /tn "Wamadat DR Restore Drill"` · حذف: `schtasks /delete /tn "Wamadat DR Restore Drill" /f`.
- ملاحظة: تعمل عند تسجيل دخول المستخدم (مناسب لجهاز البيتا). راجع `dr-drill.log` دوريًّا.

## النسخ اليوميّ — مُؤتمَت ✅
نسخة يوميّة تلقائيّة عبر Windows Task Scheduler:
- **المهمّة:** `Wamadat Daily DB Backup` — كل يوم 2:17ص.
- **السكربت:** `backend/scripts/daily-backup.ps1` — `pg_dump -Fc` للقاعدتين + **أرشيف الوسائط**
  مباشرةً إلى `C:\Users\U\OneDrive\wamadat-backups\<التاريخ>\` (tenants.dump + landlord.dump + **media.zip**).
- **B15 — الوسائط مشمولة:** `media.zip` يضمّ `storage/app/public` (موارد الدروس + الرفعات). الاستعادة
  الكاملة = استعادة القاعدتين **ثمّ** فكّ `media.zip` إلى `storage/app/public` (بدونها التطبيق غير قابل
  للتشغيل: شهادات/موارد/صور مكسورة). **متبقٍّ (يحتاج سرًّا — إجراؤك):** تشفير الأرشيف بكلمة مرور (الزيب
  الحاليّ غير مشفّر) + نسخة جغرافيّة ثانية غير OneDrive (S3/Azure).
- **خارج الجهاز تلقائيًّا:** المجلّد داخل OneDrive فيُزامَن للسحابة فورًا (يعالج مخاطرة
  الجهاز الوحيد). استبقاء محلّيّ 14 يومًا؛ OneDrive يحتفظ بالسحابيّ.
- **الصيغة متّسقة:** ملفّات `.dump` هذه هي **نفسها** التي يستعيدها تمرين الاستعادة والـrunbook.
- النتيجة في `storage/logs/dr-backup.log` (PASS/FAIL). إدارة: `schtasks /query /tn "Wamadat Daily DB Backup"`.
- ملاحظة: يوجد أيضًا `spatie/laravel-backup` ينتج zip أثقل (DB+ملفّات) على قرص `local` —
  منفصل وغير مُستخدَم هنا، تركته دون تغيير.

## ما يبقى (قرارك — صغير)
- **تأكّد أنّ OneDrive يُزامِن** مجلّد `wamadat-backups` للسحابة فعلًا (سجّل دخولك في OneDrive).
- راجع `dr-backup.log` و`dr-drill.log` دوريًّا (أو وجّهها لتنبيه).
- ملاحظة: المهامّ تعمل عند تسجيل دخول المستخدم (مناسب لجهاز البيتا).
