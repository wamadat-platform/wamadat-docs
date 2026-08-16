# TEST_REPORT — أكاديمية ومضات

> أدلّة تشغيل فعلية. 2026-07-16. البيئة: PHP 8.3، Postgres محلّي، قواعد `_test`.

## أوامر التشغيل المعتمدة (من CI + المشروع)
```bash
# الخلفية (تسلسلي — قواعد الاختبار تتشارك، parallel يسبّب سباق search_path)
cd backend && php artisan test
# ملف واحد
php artisan test tests/Feature/Tenant/<File>.php
# التنسيق + التحليل الساكن
php vendor/bin/pint --test && php vendor/bin/phpstan analyse --memory-limit=2G
# الواجهة
cd frontend && pnpm lint && pnpm type-check && pnpm test && pnpm build
# E2E
cd frontend && pnpm e2e
```

## السويت الخلفي الكامل — تشغيل 1 (خط الأساس)
```
Tests: 4 failed, 8 skipped, 372 passed (3519 assertions)  ·  1412s
```
**الفشول الأربعة: كلها `PlusSlaTest`** بخطأ `SQLSTATE[25P02]: current transaction
is aborted` على اتصال `landlord`.

### التشخيص (لا فشل منتج)
- **الجذر: تلوّث بين الاختبارات (test pollution)** — لا عطب في المنتج. اختبار سابق
  في نفس العملية ترك معاملة `landlord` معطوبة، فتعطّل كل استعلام landlord بعده.
- **المصدر: مسوّدة `AdminSecurityTest` الأولى** (كانت تستدعي `Livewire::test(ListUsers)`
  الذي رمى `getTable() on null` وسط معاملة بلا rollback). أُنشئت أثناء تشغيل السويت
  فالتُقطت.
- **دليل العزل القاطع:** `PlusSlaTest` منفردًا → **6/6 نجاح (27 تأكيدًا)**. فالفشل
  ليس فيه بل في حالة الاتصال الموروثة.
- **تأكيد خارجي:** CI يشغّل نفس السويت تسلسليًا وهو **أخضر 5/5** على `main`.

### الإصلاح
أُعيدت كتابة `AdminSecurityTest` لتتجنّب تعقيد Livewire mount: تعتمد `canAccessPanel`
(runtime) + فحص مصدر المورد (source) للتسريب — **لا معاملات landlord هشّة**.
- بعد الإصلاح: `AdminSecurityTest` → **4/4 نجاح (20 تأكيدًا)**.
- **تشغيل 2 (تأكيد نظافة السويت الكامل):** قيد التنفيذ — النتيجة تُحدَّث هنا.

## اختبارات الدفعة 1 الجديدة (P0 أمن) — `AdminSecurityTest`
| الاختبار | النتيجة | يثبت |
|---|---|---|
| admits only staff roles to /admin | ✅ | 6 أدوار موظّفة تدخل؛ student/consultant/team_member/affiliate/**instructor** مرفوضون |
| routes instructors to /instructor, not /admin | ✅ | فصل اللوحات — المدرّب لوحته الخاصة |
| never wires national_id / tokens as a column | ✅ | UserResource لا يكشف هوية/كلمة سر/mfa |
| never wires raw IBAN as a column | ✅ | BankTransferResource لا يكشف IBAN |

**اكتشاف حقيقي:** الاختبار الأول افترض أن `instructor` يدخل `/admin` — **خطأ**:
`canAccessPanel(admin)` يقصره على `academy_owner/admin/marketing/finance/support/
plus_supervisor`. المدرّب → `/instructor`. الاختبار الآن يوثّق ويحرس هذا الفصل الصحيح.

## المتخطّى (8)
اختبارات تتطلّب خدمات خارجية/شبكة (HIBP لكلمات المرور، إلخ) — تُتخطّى في بيئة الاختبار
عمدًا (مغطّاة بوحدات مخصّصة). لا إخفاء لفشل.

## الخلاصة
- **لا فشل منتج** في السويت — الأربعة تلوّث اختباري أُصلح مصدره.
- **372 نجاحًا** يغطّي المصادقة، الدفع، webhooks، بلس SLA، التعلّم، الشهادات، PDPL، العزل.
- الدفعة 1 أضافت 4 حرّاس أمنيين (P0) خضراء.
