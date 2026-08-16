# FINANCIAL_FLOW_AUDIT — أكاديمية ومضات

> فحص كود فعلي + خريطة تغطية الاختبارات. 2026-07-16. الدفعة 2 (P0 مالي).

## مسار الدفع (من الكود)
```
سلّة (عميل) → POST /api/v1/checkout [Idempotency-Key]
  → CheckoutService::placeOrder:
      • تحقّق توفّر البوّابة (fail-closed إنتاجيًا)
      • فحص idempotency: (user_id, idempotency_key) موجود؟ → أرجِع الطلب الأصلي
      • حظر إعادة الشراء فقط على تسجيل usable
      • كوبون: validateAndCompute + تسجيل redemption
      • إجماليات VAT-مشمولة (هللات صحيحة) داخل معاملة tenant
      • Order(status=awaiting_payment) → البوّابة تُرجع redirect_url
→ webhook: POST /api/v1/webhooks/{gateway}
      • بوّابة مجهولة → رفض
      • verifyWebhook() (HMAC/توقيع) إلزامي
      • dedup: فهرس فريد (gateway, external_id, event_type)
      • ربط المستأجر عبر جدول توجيه landlord (CR-1)
      • refund → RefundWebhookHandler
→ finalizePaidOrder (idempotent على الحالات الطرفية):
      • تحقّق تطابق المبلغ → عدم تطابق = on_hold (لا إكمال)
      • flip paid → تسجيل تلقائي (Learning) → فاتورة ZATCA + QR
→ شبكة أمان: orders:reconcile (5د) · webhooks:replay-queued (2د)
```

## المبادئ المُتحقَّقة (ثوابت)
| المبدأ | التنفيذ | الدليل |
|---|---|---|
| **هللات صحيحة** | كل المبالغ `*_halalas` int | CheckoutTest |
| **VAT 15% مشمولة** | `tax = round(gross×15/115)` لا تُضاف فوق | CheckoutTest (happy) |
| **لا شحن مزدوج (idempotency)** | مفتاح checkout + فهرس فريد | ← **الفجوة، تُملأ بـCheckoutIdempotencyTest** |
| **عدم تطابق المبلغ = حجز** | on_hold لا finalize | PreLaunchMoneyFixes «holds…mismatch» |
| **webhook مكرّر = إكمال واحد** | finalize idempotent على paid | CertProgressEnrollEvent:210 «no double-dispatch on replay» |
| **توقيع webhook** | verifyWebhook HMAC | TapWebhookSignatureTest · TabbyWebhookTest (Unit) |
| **ربط المستأجر** | جدول توجيه landlord قبل الكتابة | WebhookTenantBinding «no tenant header…CR-1» |
| **تزامن webhook+تسوية** | UPDATE مشروط، قيد مرّة | PlusLaunchGate:165 |
| **استرداد durable** | dedup 60s + جزئي متعدّد | RefundDurability (5 اختبارات) |
| **كوبون منتهي/مستخدم** | يُرفض | CheckoutTest |
| **هدية غير قابلة للاسترداد قبل الدفع** | كود لا يُصدَر حتى paid | PreLaunchMoneyFixes |

## خريطة التغطية مقابل قائمة الحالات المطلوبة
✅ **مغطّى**: سلّة فارغة · برنامج غير منشور · مسجّل مسبقًا · كوبون منتهي · عدم تطابق
المبلغ (حجز) · webhook مكرّر (لا تسجيل مزدوج) · توقيع خاطئ (Unit) · webhook بلا ربط
مستأجر · استرداد/استرداد جزئي/dedup · تزامن webhook+تسوية · فاتورة VAT متّسقة.

🟠 **الفجوة (تُملأ هذه الدفعة)**: **idempotency-key للـcheckout عبر HTTP** — لم يكن
هناك اختبار يثبت أن إعادة إرسال checkout بنفس المفتاح تُرجع **نفس الطلب** (لا طلبًا
ثانيًا). العمل (B1) منفّذ في الخدمة لكن غير محروس باختبار.

🔵 **حالات يُنصح بتغطيتها لاحقًا** (P2، ليست عيوبًا مؤكّدة):
- webhook بعملة مختلفة (currency mismatch) — التحقّق الحالي على المبلغ؛ يُستحسن تأكيد العملة صراحةً.
- webhook لطلب مدفوع بالفعل → no-op صريح (مغطّى ضمنيًا عبر idempotency الحالة، يُستحسن اختبار مباشر).
- كوبون من مستأجر آخر (عزل) — يُغطّى في دفعة العزل.

## المخاطر
- **لا مخاطر P0 مفتوحة مرصودة في المسار المالي** — الثوابت الحرجة (لا شحن/تسجيل/فاتورة
  مزدوجة، عدم تطابق = حجز، توقيع، ربط مستأجر) كلها منفّذة ومغطّاة، والفجوة الوحيدة
  (idempotency HTTP) تُغلَق هذه الدفعة.
- **تذكير المالك**: البوّابات على mock محلّيًا؛ الاختبار الحقيقي بمفاتيح Tap الحيّة =
  OWNER_ACTIONS.

## نتائج الاختبارات (الدفعة 2)
`CheckoutIdempotencyTest` → **2/2 نجاح (9 تأكيدات)**:
- «same order on retry with same key» → نفس `order_id`/`order_number`، **طلب واحد فقط** للمشتري.
- «distinct order on different key» → طلب متمايز، طلبان.

يثبت أن idempotency-key للـcheckout (B1) يمنع الطلب/الشحن المزدوج عبر HTTP — الفجوة مُغلقة.
(شُغِّل على قواعد اختبار معزولة لتفادي تعارض السويت الكامل — النتيجة مطابقة لبيئة CI.)

---

## مُلحق الجولة الثانية (2026-07-17) — تتبّع كل مسارات المال

تدقيق عدائي كامل لوحدتَي Commerce/Marketplace. **لا P0 في مسار شراء البرامج الأساسي.** العيوب كانت في مسارات الشحن الثانوية (استشارة، هدية) التي لم تُعِد استخدام آلة idempotency الرئيسية.

**أُصلح + اختبار ارتداد:**
- **FIN-1 (P1) — شحن الاستشارة المزدوج:** `ConsultationCheckoutService::startPayment` كان يُنشئ payment جديدًا ويشحن بمفتاح متغيّر كل نقرة → نقرتان قبل الـwebhook = شحنتان في Tap، طلب واحد يُنفَّذ، الأسر اليتيم غير مكشوف (reconciliation يمسح awaiting_payment فقط). الإصلاح: قصر-دائرة الرابط المخزَّن + مفتاح ثابت `order_number` + إعادة استخدام صفّ الدفع. اختبار: نقرتان = دفعة واحدة + نفس الرابط.
- **FIN-2 (P2) — هدية checkout داخل معاملة:** `GiftCheckoutService::place` كان ينادي `->charge()` داخل معاملة DB (يخالف الثابت الموثّق في CheckoutService/PlusCheckoutService). الإصلاح: الالتزام أوّلًا ثم الشحن بعده + عكس عبر `failOrderAfterGatewayError` المشترك. اختبار: أوّل تغطية للهدية (نجاح + فشل يُفشِل الطلب المُلتزَم).

**موثّق للمالك (يحتاج بيانات إنتاج/قرار):**
- **FIN-3 (P2 مشروط) — استرداد جزئي قد يُسجَّل كاملًا:** يحتاج شكل payload استرداد Tap الجزئي الحيّ. انظر OWNER_ACTIONS #7. لم تُلمَس شيفرة الاسترداد (خطر كسر مسار الاسترداد الكامل العامل).
- **FIN-4 (P3 تصلّب) — clamp دفتر بلس يقرأ رصيدًا قديمًا:** no-op تحت دفتر متّسق (يبيت فقط على دلو مُستنزَف)؛ التصلّب الأمثل `lockForUpdate` قبل الـclamp. لا كسر حاليًا.
- **ملاحظة:** لا قيد فريد على `invoices.order_id` — يمنعه حارس `$finalized` أحادي-الفائز حاليًا؛ فهرس فريد يجعله بنيويًا آمنًا.

**ثوابت تحقّقت سليمة:** idempotency البرامج (مفتاح + فهرس فريد جزئي)، webhook dedup/سلامة-مبلغ/توقيع (Tap HMAC، Tamara/Tabby سرّ-مشترك fail-closed)، دفتر المحفظة (مقفول، حارس عدم-سالب، عكس أحادي)، الاسترداد (قفل + dedupe + موافق-ثانٍ بلا موافقة-ذاتية + شحن خارج المعاملة)، آلة الحالة (لا رجوع لـpaid من حالة نهائية)، ضريبة 15% شاملة (متطابقة عبر كل المسارات)، ترقيم فواتير ذرّي بلا فجوات، كوبونات (سقف + قفل)، صرف بلس (مبلغ محجوز تحت قفل).

