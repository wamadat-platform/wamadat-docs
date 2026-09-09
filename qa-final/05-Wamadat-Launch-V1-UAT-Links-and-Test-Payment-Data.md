# منصة ومضات التعليمية
## مرجع روابط Launch V1 وبيانات اختبار الدفع

> **حالة ما بعد الإغلاق (9 سبتمبر 2026):** `ARCHIVED TEST REFERENCE`. بيانات Sandbox/UAT لا تحدد حالة أو مفاتيح Production الحالية.

**نوع الوثيقة:** QA / UAT Handover Reference  
**المنتج:** منصة ومضات التعليمية — Launch V1  
**الإصدار:** 1.1 Baseline  
**تاريخ المراجعة المعتمد:** 24 أغسطس 2026  
**الغرض:** تجميع الروابط الأساسية وبيانات الدفع التجريبية المطلوبة لتنفيذ اختبار القبول اليدوي للنسخة التجريبية.


---

# 1. قائمة اختبار القبول الكلية

قائمة اختبار القبول اليدوي الشاملة لـ Launch V1:

https://github.com/wamadat-platform/wamadat-docs/blob/main/qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md

هذه الوثيقة هي المرجع الأساسي لتنفيذ الـ UAT، وتغطي الرحلة من:

```text
Discovery
→ Registration
→ Cart
→ Coupon
→ Checkout
→ Payment
→ Order
→ Enrollment
→ Learning
→ Quiz
→ Assignment
→ Completion
→ Certificate
→ Verification
```

إضافة إلى اختبارات:

- Student Dashboard.
- Instructor Dashboard.
- Admin Dashboard.
- Bank Transfer.
- Orders / Payments / Invoices.
- Notifications.
- Support.
- Attendance.
- Permissions / Data Isolation.
- Responsive / RTL.
- Launch V1 Scope Gate.

---

# 2. خارطة تطوير المنصة

خارطة تطوير وإطلاق منصة ومضات بعد Launch V1:

https://github.com/wamadat-platform/wamadat-docs/blob/main/qa-final/02-Wamadat-Product-Development-Roadmap.md

توضح الخارطة الترتيب المعتمد:

```text
Launch V1
↓
Post-Launch Stabilization
↓
V1.1 — Engagement & Experience
↓
Phase 2 — Learning & Operations Expansion
↓
Phase 3 — Product & Business Expansion
```

---

# 3. المميزات والعمليات المعتمدة للمرحلة V1

وثيقة النطاق الوظيفي الرسمي للمرحلة الأولى:

https://github.com/wamadat-platform/wamadat-docs/blob/main/qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md

تمثل هذه الوثيقة المرجع لتحديد:

- ما يدخل ضمن Launch V1.
- ما يجب أن يعمل عند التسليم.
- ما يستطيع الطالب تنفيذه.
- ما يستطيع المدرب تنفيذه.
- ما تستطيع الإدارة تنفيذه.
- ما تم تأجيله إلى V1.1 أو Phase 2 أو مراحل لاحقة.

> وجود Feature في السورس لا يعني أنها داخلة ضمن Launch V1. المرجع النهائي للنطاق هو وثيقة المميزات والعمليات أعلاه.

---

# 4. رابط المنصة التجريبية

**Frontend / Web Platform:**

https://wamadat.smartagency-ye.com/

يستخدم لاختبار:

- الموقع العام.
- إنشاء الحساب وتسجيل الدخول.
- البرامج.
- السلة.
- الكوبونات.
- Checkout.
- الدفع.
- لوحة الطالب.
- رحلة التعلم.
- الاختبارات.
- الواجبات.
- الشهادات.
- الإشعارات.
- الدعم.

---

# 5. رابط لوحة التحكم

**Admin Panel:**

https://api-wamadat.smartagency-ye.com/admin/login

يستخدم لاختبار عمليات الإدارة ضمن نطاق Launch V1، مثل:

- Programs.
- Students.
- Enrollments.
- Instructors.
- Orders.
- Payments.
- Bank Transfers.
- Invoices.
- Coupons.
- Certificates.
- Support Tickets.
- Site Settings.

> بيانات حساب الإدارة نفسها يجب مشاركتها عبر قناة خاصة، ولا يوصى بوضع كلمة مرور Admin داخل هذه الوثيقة أو داخل مستودع عام.

---

# 6. بيانات الدفع التجريبي

## 6.1 Tap Payments — Sandbox

يجب استخدام هذه البيانات فقط عندما تكون بوابة Tap مضبوطة بمفاتيح **Test / Sandbox**.

### اختبار دفع ناجح — Visa

```text
Card Number: 4508750015741019
Expiry: 01/39
CVV: 100
3D Secure: Yes
Expected Result: APPROVED / CAPTURED
```

### اختبار دفع ناجح — Mastercard

```text
Card Number: 5123450000000008
Expiry: 01/39
CVV: 100
3D Secure: Yes
Expected Result: APPROVED / CAPTURED
```

### اختبار دفع ناجح — Mada

```text
Card Number: 4464040000000007
Expiry: 01/39
CVV: 100
3D Secure: Yes
Expected Result: APPROVED / CAPTURED
```

---

## 6.2 Tap — محاكاة حالات مختلفة

في بطاقات Visa / Mastercard التجريبية يمكن استخدام تاريخ الانتهاء لمحاكاة النتيجة.

### نجاح

```text
Expiry: 01/39
Expected: APPROVED
```

### رفض

```text
Expiry: 05/22
Expected: DECLINED
```

### بطاقة منتهية

```text
Expiry: 04/27
Expected: EXPIRED_CARD
```

### Timeout

```text
Expiry: 08/28
Expected: TIMED_OUT
```

### CVV مطابق

```text
CVV: 100
Expected CVV Result: MATCH
```

### CVV غير مطابق

```text
CVV: 102
Expected CVV Result: NO_MATCH
```

> لا تستخدم أي بطاقة بنكية حقيقية أثناء Sandbox UAT.

---

# 7. Tabby — KSA Test Credentials

بيانات Tabby التالية مخصصة لاختبار السعودية **KSA / SAR**.

## 7.1 Payment Success

استخدم:

```text
Email: otp.success@tabby.ai
Phone: +966500000001
OTP: 8888
Expected Result: Successful Payment
```

بعد نجاح الرحلة يجب التحقق حسب التكامل من:

```text
Merchant Dashboard: CAPTURED
Retrieve Payment API: CLOSED
Captured Amount: Present
```

---

## 7.2 Background Pre-scoring Reject

استخدم:

```text
Email: otp.success@tabby.ai
Phone: +966500000002
```

النتيجة المتوقعة:

```text
Tabby is unavailable / rejected during eligibility check
```

ويجب ألا يتم التعامل مع الطلب باعتباره مدفوعًا.

---

## 7.3 Payment Failure / Rejected

استخدم:

```text
Email: otp.rejected@tabby.ai
Phone: +966500000001
OTP: 8888
Expected Result: REJECTED
```

النتيجة المتوقعة:

- تظهر شاشة رفض Tabby.
- يعود المستخدم إلى Failure URL أو Checkout حسب التكامل.
- لا يعتبر الطلب مدفوعًا.
- لا ينشأ Enrollment مدفوع.
- Payment status النهائي يكون `REJECTED`.

---

## 7.4 Payment Cancellation

ابدأ ببيانات النجاح:

```text
Email: otp.success@tabby.ai
Phone: +966500000001
OTP: 8888
```

ثم ألغِ العملية من صفحة Tabby قبل إكمالها.

يجب التحقق من:

- العودة إلى Cancel URL.
- عدم اعتبار الطلب مدفوعًا.
- عدم إنشاء Enrollment مدفوع.
- عدم ضياع السلة بصورة غير صحيحة.
- إمكانية إعادة المحاولة.

---

# 8. سيناريو الدفع الإلكتروني المطلوب في UAT

للدفع الإلكتروني، لا يكفي ظهور صفحة "تم الدفع".

يجب التحقق من الرحلة كاملة:

```text
Checkout
→ Create Order
→ Open Payment Provider
→ Complete Payment
→ Provider Callback / Redirect
→ Webhook
→ Payment Status Update
→ Order Status Update
→ Enrollment
→ Invoice
→ My Programs
```

---

# 9. اختبار نجاح Tap / Tabby

بعد الدفع الناجح تحقق من:

- [ ] تم إنشاء Order واحد فقط.
- [ ] المبلغ صحيح.
- [ ] العملة صحيحة.
- [ ] Payment Provider صحيح.
- [ ] Payment Status صحيحة.
- [ ] Order Status صحيحة.
- [ ] Enrollment تم إنشاؤه مرة واحدة.
- [ ] البرنامج ظهر في My Programs.
- [ ] Invoice تم إنشاؤها مرة واحدة.
- [ ] Coupon/Discount محفوظ بالقيمة الصحيحة إن استخدم.
- [ ] Refresh لا ينشئ Order جديدًا.
- [ ] Refresh لا ينشئ Payment جديدًا.
- [ ] Refresh لا ينشئ Enrollment جديدًا.
- [ ] Webhook المتكرر لا يسبب Duplicate.
- [ ] المستخدم يستطيع بدء رحلة التعلم بعد نجاح الدفع.

---

# 10. اختبار فشل / رفض الدفع

عند محاكاة Declined أو Rejected:

- [ ] لا تتحول العملية إلى Paid.
- [ ] لا يعتبر Order ناجحًا بشكل خاطئ.
- [ ] لا ينشأ Enrollment مدفوع.
- [ ] لا تنشأ Invoice مدفوعة بشكل خاطئ.
- [ ] تظهر رسالة مفهومة للمستخدم.
- [ ] يستطيع المستخدم إعادة المحاولة.
- [ ] لا يؤدي Retry إلى ازدواجية غير صحيحة.
- [ ] لا يؤدي Refresh إلى تحويل العملية الفاشلة إلى ناجحة.

---

# 11. اختبار الإلغاء

عند إلغاء الدفع:

- [ ] يعود المستخدم إلى المنصة بصورة صحيحة.
- [ ] لا يسجل Payment Successful.
- [ ] لا ينشأ Enrollment بسبب العملية الملغاة.
- [ ] السلة لا تضيع بصورة غير متوقعة.
- [ ] توجد CTA واضحة لإعادة المحاولة.
- [ ] لا يوجد Server Error / 500.

---

# 12. اختبار Idempotency

هذا الاختبار مهم جدًا قبل الإطلاق التجاري.

كرر أو نفذ ما يلي:

```text
Double click on payment CTA
Refresh return page
Browser Back
Repeated webhook
Retry payment
```

ثم تحقق:

- [ ] Order لا يتكرر للمحاولة نفسها.
- [ ] Payment لا يتكرر بصورة غير صحيحة.
- [ ] Enrollment لا يتكرر.
- [ ] Invoice لا تتكرر.
- [ ] المبلغ لا يحتسب مرتين.
- [ ] حالة النظام النهائية متسقة.

أي ازدواجية مالية أو Enrollment غير صحيح تعتبر مشكلة عالية الخطورة ويجب إغلاقها قبل Public Launch.

---

# 13. التحويل البنكي

إضافة إلى Tap وTabby، يجب تنفيذ رحلة Bank Transfer الموجودة ضمن V1:

```text
Checkout
→ Bank Transfer
→ Order Pending
→ Transfer Number
→ Receipt Upload
→ Admin Review
→ Approve / Reject
→ Payment / Order Update
→ Enrollment on Approval
→ Invoice
```

ويجب اختبار:

- رفع Receipt.
- عزل الملف عن Public access.
- ظهور Receipt الصحيح للإدارة.
- Approve.
- Reject.
- عدم إنشاء Enrollment قبل الاعتماد.
- عدم تكرار Enrollment/Invoice عند إعادة الإجراء.

---

# 14. ملاحظات أمنية

- لا تضع `sk_test_*` أو `sk_live_*` أو أي Secret Key داخل هذه الوثيقة.
- لا تضع مفاتيح Production في GitHub.
- لا تضع كلمة مرور Admin في مستودع عام.
- البطاقات المذكورة هنا Test Cards وليست بيانات بطاقات حقيقية.
- بيانات Tabby المذكورة Test Credentials رسمية.
- عند الانتقال إلى Production يجب استبدال مفاتيح Test بالمفاتيح الحية فقط عبر Secret/Environment Management.
- لا يتم اختبار Production ببطاقات Sandbox.

---

# 15. المصادر الرسمية لبيانات الدفع

## Tap Payments

Test Cards:

https://developers.tap.company/reference/testing-cards

API / Test Keys overview:

https://developers.tap.company/docs/get-started

## Tabby

Testing Credentials:

https://docs.tabby.ai/testing-guidelines/testing-credentials

---

# 16. ترتيب الاختبار المقترح

نفّذ الاختبارات بهذا الترتيب:

```text
1. Launch V1 Scope Gate
2. Public Website
3. Registration / Login
4. Catalog
5. Cart
6. Coupon
7. Checkout
8. Tap Success
9. Tap Failure
10. Tabby Success
11. Tabby Pre-scoring Reject
12. Tabby Payment Reject
13. Payment Idempotency
14. Orders / Payments / Invoices
15. Enrollment
16. Learning
17. Quiz
18. Assignment
19. Completion
20. Certificate / QR Verification
21. Notifications
22. Support
23. Bank Transfer
24. Permissions / Isolation
25. Responsive / RTL
26. Final Golden E2E
```

---

# 17. معيار الإغلاق

لا يعتبر اختبار الدفع مغلقًا بمجرد نجاح Tap أو Tabby.

يجب نجاح الترابط التالي بالكامل:

```text
Payment Provider
+
Order
+
Payment
+
Invoice
+
Enrollment
+
Student Access
```

ويجب التأكد من:

- لا توجد Duplicate Records.
- لا يوجد Enrollment بعد Failed/Rejected/Cancelled Payment.
- لا يوجد Paid Order بسبب Return URL فقط دون تحقق آمن.
- Webhook يعالج بصورة Idempotent.
- المبلغ والعملة والخصم والفاتورة متطابقة.
- الطالب يحصل على البرنامج الصحيح فقط.

---

# 18. ملاحظة نهائية للمختبر

المرجع الوظيفي الأساسي أثناء UAT هو:

1. `03-Wamadat-Launch-V1-Manual-UAT-Checklist.md`
2. `01-Wamadat-Launch-V1-Features-and-Operations.md`
3. `02-Wamadat-Product-Development-Roadmap.md`

وعند وجود اختلاف في فهم Feature:

- وثيقة **Features and Operations** تحدد ما يدخل V1.
- وثيقة **Manual UAT** تحدد كيف يتم قبوله واختباره.
- وثيقة **Roadmap** تحدد ما تم تأجيله وما يأتي بعد الإطلاق.

---

**End of Document**
