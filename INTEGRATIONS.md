# دليل التكاملات — المتطلبات التقنية لكل خدمة

كل تكامل في لوحة **«الموقع ← التكاملات»** يطلب **معلومة الربط** الصحيحة فقط.
أدناه ما تلصقه بالضبط، ومن أين تأخذه، وكيف تتأكّد أنه يعمل. الأكواد المحقونة
هي الأكواد **الرسمية** لكل خدمة.

| الخدمة | تلصق | من أين | كيف تتأكّد |
|---|---|---|---|
| **Google Tag Manager** | `GTM-XXXXXXX` | tagmanager.google.com ← Admin ← Container ID | GTM Preview / Tag Assistant |
| **Google Analytics 4** | `G-XXXXXXXXXX` | analytics.google.com ← Admin ← Data Streams ← Measurement ID | GA4 ← Realtime / DebugView |
| **Meta Pixel** | معرّف رقمي | Meta Events Manager ← Data Sources ← Pixel ID | إضافة Meta Pixel Helper |
| **Snapchat Pixel** | معرّف (UUID) | Snapchat Ads ← Events Manager ← Pixel ID | Snap Pixel Helper |
| **TikTok Pixel** | `CXXXX…` | TikTok Ads ← Assets ← Events ← Pixel ID | TikTok Pixel Helper |
| **Microsoft Clarity** | معرّف المشروع | clarity.microsoft.com ← Settings ← Project ID | لوحة Clarity (تسجيلات/خرائط) |
| **X (Twitter) Pixel** | `oXXXX` | X Ads ← Tools ← Conversion tracking ← Pixel ID | X Pixel Helper |
| **Calendly** | رابط صفحتك `https://calendly.com/...` | حساب Calendly ← Share | يظهر تضمين الحجز في صفحة الاستشارات |
| **Tawk.to** | `propertyId/widgetId` | Tawk.to ← Admin ← Channels ← Chat Widget (الجزء بعد embed.tawk.to/) | تظهر نافذة الدردشة بالموقع |
| **Search Console** | قيمة `content` لوسم HTML | Search Console ← Settings ← Ownership verification ← HTML tag | زر «التحقّق» في Search Console |
| **زر واتساب** | رقم دولي `9665XXXXXXXX` | رقمك | يظهر زر أخضر عائم بكل الصفحات |
| **Zapier** | رابط Webhook | Zapier ← Zap ← «Webhooks by Zapier ← Catch Hook» | اختبر الـZap بعد أوّل حدث |

## ملاحظات مهمّة (وقت التطبيق)

- **التحقّق النهائي يحتاج معرّفك الحقيقي + أداة كل خدمة** (Pixel Helper / DebugView…)
  — لا يمكن تأكيد «الإطلاق الفعلي» إلا بحسابك الحقيقي.
- **البكسلات تتبّع `PageView` تلقائياً.** تتبّع أحداث التحويل (Purchase) داخل كل بكسل
  هو خطوة لاحقة أعمق (إطلاق حدث شراء عند إتمام الطلب) — نضيفه عند الحاجة.
- **Zapier** يستقبل أحداث: `consultation.created`، `order.paid`، `newsletter.subscribed`
  (أفضل جهد، رابط الويبهوك سرّي على الخادم ولا يُكشف للواجهة).
- **GTM**: نحقن سكربت الرأس الرسمي؛ وسم `noscript` الاحتياطي (للمتصفّحات بلا JS)
  اختياري ويمكن إضافته لاحقاً.
