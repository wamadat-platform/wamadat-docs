# منصة ومضات التعليمية — دليل التشغيل والنشر الإنتاجي
## 04-Production Deployment and Operations

**نوع الوثيقة:** دليل تشغيلي ونشر إنتاجي
**الإصدار:** 1.2 — PASS 3 Production Delivery
**تاريخ المراجعة المعتمد:** 25 أغسطس 2026
**النطاق:** Launch V1 Production Topology

---

## 1. المعمارية الإنتاجية المعتمدة

تعمل المنصة لاحقًا على ثلاثة خوادم خاصة، دون تثبيت قاعدة البيانات أو Redis
داخل خوادم التطبيق:

```text
Cloudflare (DNS / TLS / WAF / Load Balancing)
                    │
        ┌───────────┴───────────┐
        │                       │
VPS-01 — Cloud VPS 4    VPS-02 — Cloud VPS 4
APP-A                   APP-B
Laravel Web/API         Laravel Web/API
Next.js                 Next.js
Queue Worker            Queue Worker
Scheduler               Scheduler
        │                       │
        └───────────┬───────────┘
                    │ private network
          VPS-03 — Cloud VPS Plus 6 / NVMe
          PostgreSQL + Redis + backup tooling
```

الخدمات الخارجية المعتمدة: Cloudflare R2 للوسائط ونسخ التطبيق، Sentry
للمراقبة، Resend للبريد، Tap/Tabby للمدفوعات، GitHub Actions وGHCR للصور،
وCoolify لتنسيق النشر. لا تُحفظ عناوين IP أو UUIDs أو أسرار في Git.

## 2. ملكية البناء والإصدار

- `wamadat-backend`: CI وصورة Laravel وأوامر الجاهزية والهجرات والنسخ.
- `wamadat-web`: CI وصورة Next.js وقيم `NEXT_PUBLIC_*` وقت البناء ورفع
  sourcemaps اختياريًا إلى Sentry.
- `wamadat-platform`: مانيفست الإصدار والترقية والنشر والتحقق والتراجع.

لا يُسحب السورس ولا تُثبت Composer أو pnpm ولا يُنفذ build على VPS. خط التسليم
الوحيد هو:

```text
GitHub Actions → GHCR images tagged with full Git SHA → Coolify
```

كل صورة غير قابلة للتبديل. لا يجوز استخدام `latest` أو `main` أو `production`
أو `stable` كمرجع نشر. SHA الـBackend وSHA الـWeb مستقلان، ويجمعهما معرف موحد:

```text
wamadat@<release-id>
├── backend_git_sha + backend_image
└── web_git_sha + web_image
```

## 3. عقد بناء الواجهة

قيم `NEXT_PUBLIC_*` عامة وتُدمج داخل حزمة المتصفح أثناء `next build`؛ تغييرها
يتطلب بناء صورة جديدة. `SENTRY_AUTH_TOKEN` سر بناء مؤقت: يستخدمه GitHub
Actions لرفع sourcemaps، ولا يُمرر كـDocker build argument ولا يدخل الصورة.
يمكن إنشاء artifacts الخاصة بإصدار Sentry أثناء البناء، لكن لا يوضع production
deploy marker إلا بعد نجاح النشر والتحقق.

## 4. ترتيب النشر الرسمي

1. اعتماد `release_id` وSHA كامل لكل مكوّن.
2. التحقق من المانيفست ومن وجود الصورتين في GHCR.
3. تشغيل `php artisan ops:release-preflight` على Backend release runner.
4. تأكيد نجاح نسخة احتياطية قبل النشر.
5. تشغيل الهجرات مرة واحدة فقط:
   `php artisan ops:release-migrate --backup-confirmed --confirm=migrate-production`.
6. ينفذ الـBackend داخليًا landlord migrations ثم tenant migrations ثم
   `ops:schema-verify`؛ لا يحتوي Platform على SQL أو منطق Tenancy.
7. نشر صورة Backend نفسها إلى `APP_MODE=web` ثم `queue` ثم `scheduler`.
8. نشر صورة Web المعتمدة.
9. التحقق من Backend `/up` و`/api/v1/health` ومن استجابة الواجهة وعدم وجود 5xx.
10. تسجيل Sentry production deploy marker وحفظ metadata الإصدار بعد النجاح فقط.

لا تعمل migrations أو seeders في `docker/entrypoint.sh`. بدء APP-A وAPP-B أو
Workers/Schedulers لا يغيّر قاعدة البيانات.

## 5. أدوار صورة Backend وRedis

| `APP_MODE` | العملية | الدور |
| --- | --- | --- |
| `web` | Nginx + PHP-FPM | API ولوحة Filament |
| `queue` | `php artisan queue:work` | الطوابير |
| `scheduler` | `php artisan schedule:work` | المهام الدورية |

المسارات الرسمية للطوابير مرتبة: `critical,payments,notifications,default`.
تستخدم العقدة المشتركة Redis DB0 للعمليات/default، DB1 للكاش، DB2 للجلسات،
وDB3 للطوابير. تعيد Coolify تشغيل الحاويات عند خروج العملية؛ لا يوجد Supervisor
داخل الصورة.

## 6. السجلات والمراقبة

- Backend يرسل سجلات الحاوية إلى `stderr` (`LOG_CHANNEL=stderr`) لتجميعها
  بواسطة Coolify/منصة السجلات؛ ملف `laravel.log` ليس مسار الإنتاج الوحيد.
- `/up` liveness عام، و`/api/v1/health` يفحص التبعيات وفق صلاحية العرض المتاحة.
- Sentry release لكل الإصدار هو `wamadat@<release-id>` للمكوّنين.
- أي workflow يفتقد إعداد Coolify أو عناوين الإنتاج يفشل؛ لا توجد نتيجة خضراء
  تدعي أن النشر تم بينما لم يحدث.

## 7. بوابات التشغيل الأول

قبل أول نشر فعلي: تجهيز الخوادم والشبكة والجدار الناري، PostgreSQL وRedis،
موارد Coolify وGHCR، Cloudflare وR2، جميع الأسرار، WAL/PITR، نقل البيانات، ثم
Production E2E وSoft Launch. هذه أعمال بنية تحتية ولا يزعم PASS 3 تنفيذها.
