# ربط بوّابات الدفع (Tap / Tamara / Tabby)

مرجع تشغيلي: ماذا يلزم لتشغيل كل بوّابة دفع. الكود يقرأ المفاتيح من `.env`؛
الحصول على المفاتيح خطوة تجارية من حساب التاجر لدى كل مزوّد.

> رابط الويبهوك الذي تسجّله في لوحة كل مزوّد:
> `https://<نطاقك>/api/v1/webhooks/{gateway}` — مثل `/api/v1/webhooks/tap`.
> نقطة الدفع للاستشارة: `POST /api/v1/consultations/{tracking}/pay` (تقبل `gateway`، الافتراضي `tap`).

---

## ✅ Tap — جاهزة في الكود (إنتاج)

أضف إلى `.env`:

```
TAP_SECRET_KEY=sk_live_xxx
TAP_PUBLIC_KEY=pk_live_xxx
TAP_WEBHOOK_SECRET=xxx
TAP_BASE_URL=https://api.tap.company/v2
```

ثم في لوحة Tap سجّل الويبهوك: `https://<نطاقك>/api/v1/webhooks/tap`.
بعدها تعمل مباشرة (شراء البرامج + دفع الاستشارات).

## 🟡 Tamara — موجودة لكن على Sandbox

أضف إلى `.env` (وغيّر `BASE_URL` للإنتاج عند الجاهزية):

```
TAMARA_API_TOKEN=xxx
TAMARA_WEBHOOK_SECRET=xxx
TAMARA_BASE_URL=https://api.tamara.co        # الإنتاج (الافتراضي حالياً sandbox)
```

الويبهوك: `https://<نطاقك>/api/v1/webhooks/tamara`. تحتاج تحقّقاً نهائياً
على الإنتاج (السكافولد جاهز لكن غير مُختبَر حيًّا).

## ❌ Tabby — غير مبنيّة (تحتاج تطوير)

لإضافتها (عند توفّر حساب Tabby):
1. كلاس `TabbyPaymentGateway` ينفّذ عقد `PaymentGateway` (charge + verifyWebhook + handleWebhook).
2. إعداد في `config/services.php` (`tabby.api_key`, `tabby.webhook_secret`).
3. تسجيلها مرّة في `CommerceServiceProvider` (نفس مكان تسجيل Tap/Tamara).
4. مفاتيح `.env` + رابط الويبهوك `/api/v1/webhooks/tabby`.

مسار دفع الاستشارة يقبل `tabby` تلقائيًّا متى سُجّلت البوّابة (يتحقّق من
المُسجَّل، ويتجاهل غير المُسجَّل بأمان).

---

## ملاحظات

- جميع المبالغ بالهللات (1 ريال = 100 هللة)، العملة `SAR`، والضريبة 15%.
- عند نجاح الدفع: الويبهوك الموجود يؤكّد الطلب، ويُصدر فاتورة ZATCA، ويُرقّي
  الاستشارة إلى «تم السداد» تلقائيًّا، ويُشعر العميل.
- لاختيار البوّابة من زر الدفع (تقسيط Tamara/Tabby مقابل بطاقة Tap) يلزم منتقي
  بوّابة في واجهة الدفع — يُضاف عند تفعيل أكثر من بوّابة.
