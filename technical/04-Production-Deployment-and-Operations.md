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

تتكون بيئة تشغيل منصة ومضات الإنتاجية من الخدمات التالية:

```text
               ┌─────────────────────────────────────────┐
               │    Reverse Proxy (Caddy / Nginx)        │
               └───────────┬─────────────────┬───────────┘
                           │                 │
              ┌────────────▼─────┐     ┌─────▼────────────┐
              │ Web Frontend Container││ API Backend Container │
              │ Next.js (Node.js)│     │ Laravel (PHP-FPM)│
              └──────────────────┘     └─────────┬────────┘
                                                 │
                  ┌──────────────────────────────┼──────────────────────────────┐
                  │                              │                              │
         ┌────────▼────────┐            ┌────────▼────────┐            ┌────────▼────────┐
         │ Database Service│            │ Cache & Redis   │            │ Worker Service  │
         │ MySQL 8.0+      │            │ Redis Instance  │            │ Queue & Scheduler│
         └─────────────────┘            └─────────────────┘            └─────────────────┘
```

---

## 2. ترتيب خطوات النشر (Deployment Order)

عند نشر تحديث جديد للبيئة الإنتاجية، يجب اتباع التسلسل التالي لتجنب تعارض الحالة:

1. **النسخ الاحتياطي السريع (Pre-Deploy Backup):** أخذ Snapshot لقاعدة البيانات قبل البدء.
2. **سحب الشفرة المصدرية (Git Pull):** التحديث إلى أحدث Commit معتمد في branch الإنتاج.
3. **تحديث الحزم وتوليد الكاش (Backend Build & Cache):**
   ```bash
   composer install --no-dev --optimize-autoloader
   php artisan migrate --force
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   ```
4. **تحديث طابور العمليات (Queue Worker Reload):**
   ```bash
   php artisan queue:restart
   ```
5. **بناء وإعادة تشغيل واجهة المستخدم (Frontend Build):**
   ```bash
   npm install --frozen-lockfile
   npm run build
   # Restart Node container / PM2 / Docker process
   ```
6. **فحوصات السلامة السريعة (Smoke Checks):** التثبت من استجابة `/health` والصفحة الرئيسية وبوابة الإدارة.

---

## 3. إدارة المهام الخلفية والجدولة (Worker Supervision & Scheduler)

- **طابور العمليات (Queue Worker):** تشغيل `php artisan queue:work --queue=default,emails,certificates --tries=3` تحت إشراف Supervisor أو Docker Restart Policy.
- **المجدل الدوري (Artisan Scheduler):** إيقاد `php artisan schedule:run` كل دقيقة عبر Cron Job لإدارة تنظيف الأسرار المؤقتة، جدولة البريد، والمهام الدورية.

---

## 4. المراقبة واستكشاف الأخطاء (Logs & Observability)

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
