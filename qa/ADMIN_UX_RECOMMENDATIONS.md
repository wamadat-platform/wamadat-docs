# ADMIN_UX_RECOMMENDATIONS — لوحة /admin

> مبني على الجرد الفعلي (ADMIN_PAGE_INVENTORY.md) وبوّابة الوصول المُتحقَّقة
> (`UserModel::canAccessPanel` + `AdminRoleGate`). 2026-07-16.

## الهيكل الحالي (فعلي — 12 مجموعة)
المحتوى · المبيعات · المالية · التعلّم والتقييم · ومضات بلس · الحوكمة · التسويق ·
المجتمع · الدعم · التشغيل · الموقع · المستخدمون.

**التقييم:** بنية **جيدة أصلًا** — مجموعات عربية منطقية، ووصول مُقيَّد بالأدوار
(`academy_owner/admin` كامل، `finance` مالية، `marketing` تسويق، `support` دعم،
`instructor` محتوى، `plus_supervisor` بلس). التحسينات أدناه **صقل**، لا إعادة هيكلة.

## المشكلات المرصودة
1. **تشتّت وظيفي بسيط**: `LessonResource` تحت «المحتوى» بينما رحلة العمل تضعها مع
   «التعلّم». `AbandonedCart` و`ProgramInterest` (Leads) تحت «المبيعات» بينما هما تسويقيان.
   `Partner` (شعارات موثوق-من) تحت «المجتمع» بينما هو محتوى موقع.
2. **موردان للأفلييت منفصلان** (Partner + Conversion) — يمكن دمجهما بصريًا في تبويبين.
3. **محتوى الرئيسية مبعثر**: `HomeCard`/`HomeFaq`/`Testimonial` بلا مجموعة موحّدة
   (يظهر بعضها خارج «الموقع»).
4. **الموارد التقنية/الحسّاسة** ظاهرة في التنقّل — مقبول للموظّف المخوّل، لكن يجب
   تأكيد أنها **خلف صلاحية صريحة** لا مجرّد وجود: MediaAsset, PaymentWebhook,
   EmailOutbox, FailedJob, TenantAuditLog, UserConsent.

## الهيكل المقترح (صقل — بلا حذف)
```
نظرة عامة        → Dashboard (widgets)
المحتوى والتعلّم   → Program · Lesson(⬅نقل) · CohortBatch · LearningPath ·
                    Category · Instructor · Quiz · Assignment · LiveSession
المبيعات والمالية  → Order · Coupon · Gift · Invoice · Wallet · BankTransfer ·
                    ConsultationRequest · B2bAccount
ومضات بلس         → PlusService · PlusSeller · PlusCategory · PlusDispute · PlusPayout(مرآة)
الطلاب والمستخدمون → User(الأدوار) · TeamMember
التسويق          → Affiliate(مدمج) · Newsletter · AbandonedCart(⬅نقل) · ProgramInterest(⬅نقل)
التواصل والمجتمع   → Review · SupportTicket · ContactMessage · Partner(⬅نقل)
الموقع           → Page · HomeCard · HomeFaq · Testimonial
الحوكمة          → PolicyVersion · UserConsent · TenantAuditLog
التشغيل (Admin)   → EmailOutbox · FailedJob · PaymentWebhook · MediaAsset
```

## الصفحات المرشّحة للدمج (بصريًا فقط)
- **الأفلييت**: `AffiliatePartnerResource` + `AffiliateConversionResource` → مورد واحد بتبويبين.
- **محتوى الرئيسية**: `HomeCard`/`HomeFaq`/`Testimonial` → تحت مجموعة «الموقع» بجوار `Page`.

## الصفحات المرشّحة للإخفاء خلف صلاحية صريحة
(تبقى موجودة ومتاحة للمخوّل — تُخفى من التنقّل لمن لا يملك الصلاحية عبر `shouldRegisterNavigation`)
- `MediaAssetResource` → `media.manage`
- `PaymentWebhookResource` → `payments.webhooks.view`
- `EmailOutboxResource` · `FailedJobResource` → `ops.view` (Admin فقط)
- `TenantAuditLogResource` · `UserConsentResource` → `governance.view` (للقراءة)

## الصفحات التي يُمنع حذفها (كلها)
لا مورد بلا استخدام. كل مورد يعكس جدولًا حيًّا ورحلة عمل. أي تغيير = **نقل/تسمية/صلاحية**،
مع **redirect** لأي مسار تغيّر، وتحديث الاختبارات.

## التنفيذ (منخفض المخاطر — دفعة UX منفصلة)
1. تعديل `navigationGroup`/`navigationSort` على الموارد المنقولة (سطر واحد لكل مورد).
2. إضافة `shouldRegisterNavigation()` يفحص صلاحية على الموارد التقنية.
3. اختبار `AdminListSmokeTest` الموجود يضمن أن كل مورد لا يزال يُقلع بعد النقل.
4. لا migrations، لا تغيير بيانات، لا تغيير حارس.

> ملاحظة: بوّابة الوصول (`canAccessPanel`) **مُختبَرة** في `AdminSecurityTest` — أي تغيير
> على الأدوار يُمسك بالاختبار.

---

## ✅ المُطبَّق — الدفعة 3 (2026-07-16)
نقلات مجموعات تنقّل فقط (نصّية، صفر مخاطر وظيفية)، محروسة بـ`AdminListSmokeTest`
(«كل مورد لا يزال يُقلع» — أخضر بعد التغيير):
- `AbandonedCartResource`: المبيعات → **التسويق** (استرداد السلال شأن تسويقي).
- `ProgramInterestResource`: المبيعات → **التسويق** (Leads).
- `PartnerResource`: المجتمع → **الموقع** (شعارات «موثوق من» = محتوى موقع).
- `HomeCardResource` · `HomeFaqResource` · `TestimonialResource`: **بلا مجموعة → الموقع**
  (كانت تظهر خارج أي مجموعة؛ صارت بجوار `PageResource`).

**مُؤجَّل عمدًا** (يحتاج قرارًا/تحقّقًا أعمق، ليس آمنًا كنقلة نصّية):
- نقل `LessonResource` (جدل: الدروس محتوى تابع للبرنامج — يُبقى تحت «المحتوى»).
- تسييج الموارد التقنية بـ`shouldRegisterNavigation` على صلاحية — يتطلّب التأكّد أن
  الصلاحية موجودة وأن الأدوار الصحيحة تملكها (خطر إخفاء مورد عن مستخدم مخوّل). دفعة لاحقة.
- دمج موردَي الأفلييت بصريًا (تبويبات) — تغيير أبنية، ليس نقلة تنقّل.

**لا حذف. لا تغيير حارس. لا migration.**
