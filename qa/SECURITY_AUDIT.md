# تدقيق الأمن — الجولة الثانية (2026-07-17)

منهج: مراجعة عدائية للشيفرة الفعلية (لا مسح آلي فقط) عبر كل فئات الهجوم، مع محاولة استغلال عملي. المدقّق: عدسة Security Engineer مستقلّة + تحقّق ذاتي.

## الخلاصة
**لا توجد عيوب P0/P1/P2 قابلة للاستغلال في السطح المفحوص.** الخلفية مُحصَّنة بدرجة عالية مع آثار موجات إصلاح سابقة موثّقة. أدناه ما جرى التحقّق منه بالأدلّة، ثم المتبقّيات التي تحتاج تأكيدًا قبل التوقيع النهائي.

## تم التحقّق منه وسليم (بالأدلّة)

| الفئة | الدليل |
|---|---|
| عزل المستأجرين (P0) | 112/127 نموذجًا يستخدم `UsesTenantConnection`؛ الـ7 الباقية `landlord` بالتصميم (`PaymentGatewayRouteModel`, `FailedJobModel`, نماذج Tenancy). لا نموذج مستأجر يسقط لاتصال مشترك صامتًا |
| Webhook الدفع (P0) | يربط المستأجر من جدول landlord قبل أي كتابة، يتحقّق HMAC لكل بوّابة (`TapPaymentGateway::verifyWebhook` يغطّي المبلغ+الحالة)، يفرض captured==expected، dedupe بمفتاح فريد، ينهي داخل `TenantModel::execute()` |
| سلامة السعر | `CheckoutController` يقبل `program_id`/`slug` فقط؛ الأسعار تُقرأ خادميًا. لا سعر/إجمالي من العميل |
| IDOR | كل نقطة resource-by-id تُقيّد للمستخدم/المالك: `MyOrders`, `Certificate::download`, `Conversation::hasParticipant`, `PlusOrder` (abort_unless على كل تحوّل)، `Consultation::pay` (user_id!=id → 403) |
| Mass assignment | رغم `$guarded=[]` واسعًا، كل sink يملأ من `$request->validate([...])` (قائمة مفاتيح) — `ProfileController::update` لا يمكنه ضبط role/is_admin/national_id |
| المصادقة | دوران refresh نموذجيّ: sha256 at-rest، `lockForUpdate`، كشف إعادة الاستخدام يُبطل كل الجلسات؛ إعادة تعيين كلمة السر `Str::random(48)` مُجزّأ، TTL 60د، أحادي الاستخدام، لا user-enumeration |
| الحقن | كل `whereRaw` بمعاملات مربوطة؛ `DB::raw` تجميعيّ فقط. لا مدخل طلب في SQL خام |
| المهام الخلفية (P0) | كل أمر console يمسّ بيانات مُدرك-للمستأجر؛ القليل غير المدرك يعمل على landlord/cache فقط |
| SSRF/رفع | مساعد AI لا يأخذ URL ومحصور بالتسجيل؛ رفع الإيصال يتحقّق mime+size، يستنشق المحتوى (يتجاهل اسم العميل)، قرص خاصّ |
| Mock/simulate | البوّابة الوهمية مُسجّلة في `local/testing` فقط (`CommerceServiceProvider:27`)؛ الإنتاج يرفض مفاتيح `sk_test_`/sandbox — لا تجاوز دفع |

## المتبقّيات الثلاثة — أُغلقت بالتحقّق (2026-07-17)
1. **اتّساع RBAC للوحة الإدارة — ✅ سليم.** 45 من ~46 موردًا تحمل بوّابة `canViewAny` صريحة (لا تعتمد على بوّابة اللوحة فقط)، والموارد الحسّاسة مُقيَّدة بالدور الصحيح: `PlusPayout/PlusSeller → isPlus`، `Invoice/BankTransfer/Wallet → isFin`، `Order → isSystem`، `UserResource (PII) → isOwner`، `Page/LandingPage → isOwner`.
2. **`PlusPayoutService::request` — ✅ سليم.** يقفل صفّ الرصيد (`lockForUpdate`) ثم `available_halalas < amount → throw` قبل الخصم — لا صرف مزدوج.
3. **مسار الوسائط الموقّع — ✅ سليم.** دفاع 5 طبقات: تحقّق توقيع + **إعادة ربط `signed user == bearer.id` (وإلا 403)** + إعادة فحص التسجيل لحظة الطلب (يمنع رابطًا مُعاد توجيهه أو تسجيلًا مُلغى داخل النافذة).
4. **ملاحظة (لا عيب):** Tamara/Tabby `verifyWebhook` يستخدمان توكن سرّ-مشترك ثابت لا body-HMAC — يطابق تصميمهما، ومُخفَّف بفحص سلامة المبلغ + idempotency + ربط المستأجر.

## التوصية
السطح الأمني **جاهز لبيتا مغلقة، ولا مانع أمنيّ من أول عميل.** كل المتبقّيات أُغلقت بقراءة الشيفرة. يبقى فحص runtime حيّ (اختراق عمليّ ضدّ api.wmt.sa) كتأكيد نهائي مُستحسَن قبل الإطلاق العام.
