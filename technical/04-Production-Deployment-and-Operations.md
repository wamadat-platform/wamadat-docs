# منصة ومضات التعليمية — دليل التشغيل والنشر الإنتاجي
## 04-Production Deployment and Operations

**نوع الوثيقة:** دليل تشغيلي ونشر إنتاجي  
**الإصدار:** 1.1 Baseline  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**المنتج:** منصة ومضات التعليمية (Wamadat Platform)  
**النطاق:** Launch V1 Production Topology  
**بيئة التشغيل:** Coolify / Docker Containers

---

## 1. المعمارية التشغيلية للبيئة الإنتاجية (Production Topology)

تتكون بيئة تشغيل منصة ومضات الإنتاجية (Coolify / Docker) من الخدمات التالية. صورة الـ Backend الواحدة (`wamadat-backend`) مُصممة خصيصًا للعمل بثلاثة أنماط عبر متغير `APP_MODE` (`docker/entrypoint.sh`)، فتُنشأ منها حاويات مستقلة: `web` و `queue` و `scheduler`:

```text
                ┌─────────────────────────────────────────┐
                │      Reverse Proxy (Caddy / Nginx)      │
                └───────────┬─────────────────┬───────────┘
                            │                 │
               ┌────────────▼─────┐     ┌─────▼─────────────┐
               │ Web Frontend     │     │ API Backend       │
               │ Next.js (Node 22)│     │ APP_MODE=web      │
               └──────────────────┘     │ PHP-FPM + Nginx   │
                                        └─────────┬─────────┘
                                                  │
                    ┌─────────────────┬───────────┴──────────────┐
                    │                 │                        │
         ┌──────────▼───────┐ ┌───────▼────────┐ ┌─────────────▼──────────┐
         │ Database Service │ │ Cache & Redis  │ │ Worker Containers      │
         │ PostgreSQL       │ │ Redis Instance │ │ نفس صورة Laravel       │
         │ landlord+tenants │ └────────────────┘ │ APP_MODE=queue         │
         └──────────────────┘                    │ APP_MODE=scheduler     │
                                                 └────────────────────────┘
```

---

## 2. ترتيب خطوات النشر (Deployment Order)

عند نشر تحديث جديد للبيئة الإنتاجية، يجب اتباع التسلسل التالي لتجنب تعارض الحالة:

1. **النسخ الاحتياطي السريع (Pre-Deploy Backup):** أخذ Snapshot لقاعدتي `wamadat_landlord` و `wamadat_tenants` قبل البدء (راجع وثيقة 05).
2. **بناء الصور ونشرها (Image Build & Deploy):** الـ Dockerfile يتولى تثبيت الحزم تلقائيًا:
   - Backend: `composer install --no-dev` + `dump-autoload --optimize --classmap-authoritative` داخل الصورة.
   - Frontend: `pnpm install --frozen-lockfile` + `pnpm build` (مخرجات standalone) داخل الصورة.
3. **تنفيذ الهجرات صراحةً (Explicit Migrations):** الـ entrypoint **لا يُشغّل** الهجرات أو الـ Seeding تلقائيًا عمدًا — دورة حياة قاعدة البيانات عملية إصدار صريحة:
   ```bash
   php artisan migrate --force
   ```
4. **إعادة تشغيل طابور العمليات (Queue Worker Reload):** إعادة نشر/إعادة تشغيل حاوية `APP_MODE=queue` لتلتقط العاملات الكود الجديد، أو إصدار إشارة إعادة التشغيل اللينة:
   ```bash
   php artisan queue:restart
   ```
5. **إعادة نشر بقية الحاويات:** حاويات `APP_MODE=scheduler` و `APP_MODE=web` وحاوية الـ Frontend.
6. **فحوصات السلامة السريعة (Smoke Checks):** التثبت من استجابة `/api/v1/health` (200) والصفحة الرئيسية وبوابة الإدارة.

---

## 3. إدارة المهام الخلفية والجدولة (Worker Supervision & Scheduler)

نفس صورة الـ Backend تعمل بثلاثة أنماط عبر `APP_MODE` وفق `docker/entrypoint.sh`. أي قيمة غير مدعومة تُنهي الحاوية فورًا برمز خطأ (Fail-Fast):

| APP_MODE | العملية (PID 1) | الدور |
| --- | --- | --- |
| `web` | Nginx (مع PHP-FPM في الخلفية على المنفذ 8080) | خدمة الـ API ولوحة Filament |
| `queue` | `php artisan queue:work` | معالجة الطوابير |
| `scheduler` | `php artisan schedule:work` | المجدل الدوري |

- **طابور العمليات (Queue Worker):** يقرأ أسماء الطوابير والإعدادات من متغيرات البيئة، وقيمها الافتراضية:
  ```bash
  php artisan queue:work \
      --queue="${QUEUE_NAMES:-critical,payments,notifications,default}" \
      --sleep="${QUEUE_SLEEP:-3}" \
      --tries="${QUEUE_TRIES:-3}" \
      --timeout="${QUEUE_TIMEOUT:-120}"
  ```
- **المجدل الدوري (Scheduler):** يعمل كعملية طويلة الأمد عبر `php artisan schedule:work` — **لا يحتاج Cron Job خارجيًا** ولا نداء `schedule:run` كل دقيقة؛ الحاوية نفسها تطلق المهام الدورية في توقيتها.
- **الإشراف:** إعادة التشغيل تحت سياسة Docker/Coolify Restart Policy؛ لا حاجة لـ Supervisor داخل الحاوية.

---

## 4. المراقبة واستكشاف الأخطاء (Logs & Observability)

- **فحوصات السلامة (Healthchecks):** مدمجة في الصورة عبر `docker/healthcheck.sh` وتتغير حسب الـ Mode:
  - `web`: طلب `curl` إلى `http://127.0.0.1:8080/api/v1/health` — فشل حرج في قاعدة البيانات/الكاش يعيد 503، والتدهور المعلوماتي في الطوابير/المجدل يعيد 200.
  - `queue` / `scheduler`: فحص عملية PID 1 (`artisan queue:work` / `artisan schedule:work`) — أي خروج للعملية يُعتبر فشلًا.
- **سجلات الـ Backend:** تتواجد السجلات في `storage/logs/laravel.log`.
- **المهام الفاشلة (Failed Jobs):** يمكن متابعة المهام التي فشلت عبر لوحة Filament أو الأمر:
  ```bash
  php artisan queue:failed
  ```
- **تفريغ وإعادة تشغيل الكاش (Emergency Cache Clear):**
  ```bash
  php artisan cache:clear
  php artisan config:clear
  ```
