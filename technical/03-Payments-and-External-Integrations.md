# منصة ومضات التعليمية — المدفوعات والتكاملات الخارجية
## 03-Payments and External Integrations

**نوع الوثيقة:** مرجع تشغيلي للمدفعات والتكاملات الخارجية  
**الإصدار:** 1.2 Baseline  
**Baseline الفعلي:** 9 سبتمبر 2026  
**آخر تحديث توثيقي:** 13 سبتمبر 2026  
**المنتج:** منصة ومضات التعليمية (Wamadat Platform)  
**النطاق:** Launch V1  
**مصدر الحقيقة:** `wamadat-backend` (`Commerce`, `CheckoutService.php`)

---

## 1. تجريد خدمة الشراء (Checkout & Payment Abstraction)

تعتمد منصة ومضات خدمة شراء ممرورة عبر `CheckoutService` تنظم الدورة التجاريّة بالكامل وفق التسلسل التالي:

```text
Cart / Program 
   └──> Apply Coupon (Validation & Discount Calculation)
         └──> Create Order (Pending Payment)
               ├──> Electronic Gateway (Tap / Tabby / Tamara / Mock) ──> Webhook ──> Approve
               └──> Bank Transfer ──> Receipt Upload ──> Admin Audit ──> Approve
                     └──> Mark Order & Payment Paid
                           └──> Create Enrollment & Issue Invoice
```

---

## 2. بوابات الدفع المدعومة وحالاتها (Payment Gateways)

تميز الوثيقة بين البوابات المدعومة كودياً والبوابات المفعّلة بيئياً:

| بوابة الدفع | حالة دعم الكود | حالة تفعيل البيئة | النطاق والاستخدام |
|---|---|---|---|
| **Tap Payments** | مدعوم ومكتمل | يعتمد التفعيل الفعلي على Production credentials/config | مدفوعات البطاقات البنكية (Visa / Mastercard / Mada) |
| **Tabby** | مدعوم ومكتمل | يعتمد التفعيل الفعلي على اعتماد الحساب وProduction credentials/config | الشراء الآجل / التقسيط (BNPL - KSA SAR) |
| **Tamara** | مدعوم ومكتمل | يعتمد التفعيل الفعلي على اعتماد الحساب وProduction credentials/config | الشراء الآجل / التقسيط (BNPL) |
| **التحويل البنكي (Bank Transfer)** | مدعوم ومكتمل | مفعّل في جميع البيئات | تحويل يدوي + رفع إيصال مراجع إدارياً |

---

## 3. حماية Webhooks ومنع الازدواجية المالية (Idempotency & Webhooks)

1. **التحقق من التوقيع (Webhook Signature Verification):** يتم تأمين جميع إشعارات الدفع القادمة من مزودي الخدمات للتحقق من مصدرها الرسمي.
2. **معالجة التكرار (Idempotency Handling):** يمنع النظام معالجة نفس الـ Webhook أكثر من مرة؛ عند تكرار الإشعار يتم التثبت من حالة الطلب والـ Payment المعالج مسبقاً وإرجاع رد `200 OK` دون إنشاء Order أو Enrollment أو Invoice مكررة.

---

## 4. نظام الفواتير وخصومات الكوبونات والـ ZATCA QR

- **الخصومات والكوبونات:** تحسب قيمة الخصم وتخزن مبالغ الخصم الأصلية بالطلب والدفع.
- **الفواتير والضريبة (Invoices & VAT):** تصدر الفاتورة فور اعتماد العملية المالية وتتضمن بيانات الضريبة والمنشأة.
- **رمز الـ QR الخاص بالفاتورة (ZATCA QR):** ينشأ رمز الـ QR المعتمد للفاتورة وفق متطلبات هيئة الزكاة والضريبة والجمارك عند تفعيل إعدادات الضريبة بالمنصة.

---

## 5. خدمات البريد والتخزين السحابي (External Services)

- **خدمة البريد الإلكتروني (Transactional Email):** تستخدم المنصة محرك البريد لإرسال إشعارات التسجيل، إشعار اعتماد التحويل، وتنبيهات تذاكر الدعم عبر الخدمة المعتمدة بيئيًا (Resend / SMTP).
- **التخزين السحابي (S3 Storage):** تخزين إيصالات التحويل البنكي وصور غلاف البرامج والشهادات الصادرة عبر المزود المعتمد في `.env`.
---

## 6. تكاملات التسويق والتحليلات ضمن V1

تتضمن طبقة التكاملات الحالية دعمًا مباشرًا وقابلًا للإدارة من إعدادات المنصة لـ:

| التكامل | حالة القدرة في V1 | الاستخدام |
|---|---|---|
| Google Tag Manager (GTM) | مدعوم | إدارة Google tags وdataLayer |
| Google Analytics 4 (GA4) | مدعوم | Analytics + ecommerce measurement |
| Meta Pixel | مدعوم | PageView / Purchase وغيرها وفق Consent والحدث |
| TikTok Pixel | مدعوم | ViewContent / AddToCart / InitiateCheckout / AddPaymentInfo / CompletePayment وفق الرحلة |
| Snapchat Pixel | مدعوم | Page View / Purchase وقياس التحويل |

تطبق الواجهة تحميل التكاملات وفق فئة Consent المناسبة، وتُدار المعرّفات العامة من Site/Integrations Settings.

كما تتضمن رحلة Checkout حفظ بيانات Attribution التي تدعمها النسخة الحالية مثل UTM و`gclid` / `gbraid` / `wbraid`.

> القاعدة: **Integration capability جزء من V1، أما ID/account activation فهو Runtime configuration.**

