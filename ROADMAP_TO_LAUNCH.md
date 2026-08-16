# 🗺️ خارطة الطريق إلى الإطلاق الكامل

**التاريخ:** 2026-05-12
**الإصدار:** v1.0
**المالك:** مشعل العسيري
**المنصة:** ومضات أكاديمي — منصة تعلم عربية متعددة المستأجرين

---

## 📐 الفلسفة

هذي الخارطة مبنية على **3 بوابات إطلاق**:

| البوابة | الجمهور | متى |
|---|---|---|
| 🟡 **G1 — Closed Beta** | 50 طالب + 5 مدربين بدعوة فقط | بعد أسبوعين من اليوم |
| 🟢 **G2 — Soft Launch** | عام بدون تسويق مدفوع | بعد 4 أسابيع |
| 🚀 **G3 — Full Launch** | تسويق + علاقات عامة + بيع نشط | بعد 8 أسابيع |

كل بوابة لها **شروط لازمة** ولا تُفتح إلا لما كل بنودها تنجلق.

---

## ✅ القرارات المالية الحاسمة (مُقفلة)

| القرار | الحالة |
|---|---|
| لا اشتراكات شهرية/سنوية للطلاب | ✅ مُقفل — feature flag معطل |
| لا باقات (Bundles) | ✅ مُقفل — feature flag معطل |
| المنصة عربية فقط، بدون إنجليزي | ✅ مُقفل — /en يُعاد توجيهه |
| كتابة عربية سادة بدون تشكيل | ✅ مُقفل — سُحبت من 78 ملف اليوم |
| لا تكامل مع TVTC | ✅ مُقفل — شهادات ومضات الخاصة |
| إيرادات ومضات = برامج فردية + هدايا + استشارات + B2B + Affiliate | ✅ مُقفل |

---

## 🚧 الوضع الحالي (اليوم)

### ما يعمل ✅
- صفحة هبوط احترافية مع 5 أقسام مُتحركة
- كتالوج البرامج (`/programs`) + البحث + الفلترة
- صفحة المدرب الواحد (`/instructors/[id]`)
- صفحة فريق العمل + المدربين + انضم إلينا (`/instructors`)
- صفحة الاستشارات (`/consultations`) — 3 باقات + نموذج حجز
- صفحة الشركات (`/for-business`) — تيير + CTA
- التسجيل + تسجيل الدخول (مُصلح في هذا الشوط)
- السلة → التسجيل → الدفع (المنطق موجود، اختبار الـ checkout ناقص)
- التعلم: تشغيل الدروس + التقدم + الاختبارات + الشهادات
- الإشعارات + قائمة الأمنيات + الطلبات + المنجزات + سلسلة التعلم
- الهدايا (إنشاء + تفعيل بكود + لوحة هدايا مرسلة)
- البرنامج المرافق (Affiliate) — backend جاهز
- 8 حسابات تجريبية شغّالة (`<role>@wamadat.test` / `Wamadat@2026`)
- لوحة Filament أدمن — Orders / Invoices / Coupons
- security headers مُفعلة (HSTS + X-Frame-Options + CSP أساسي)

### ما لا يعمل ❌ أو ناقص ⚠️
- **❌ نسيت كلمة المرور** — لا يوجد endpoint
- **❌ قفل الحساب بعد محاولات فاشلة** — لا يوجد
- **❌ 2FA للحسابات الحساسة** — موجود في الـ super أدمن فقط
- **❌ لوحة المدرب الذاتية** — لا توجد UI
- **❌ نموذج التقديم كمدرب** — صفحة `/become-instructor` ماركتنج فقط
- **❌ Filament moderation** — مراجعات / استرداد / إيقاف مستخدم
- **❌ تحليلات (Analytics)** — لا Plausible ولا أحداث
- **❌ Bunny Stream wiring للفيديو** — كل البرامج تستخدم HTML5 player الحالي
- **❌ بريد إشعارات** — الأحداث تنطلق لكن لا listeners
- **❌ Captions للفيديوهات** — WCAG fail
- **⚠️ غلاف PWA** — manifest موجود، service worker جزئي
- **⚠️ /pricing redirects إلى /programs** — مقبول لكن يحتاج صفحة "كيف نُسعّر" بديلة
- **⚠️ تأكيد البريد + الجوّال** — موجود لكن غير مُفعّل في رحلة التسجيل

---

# 🟡 G1 — Closed Beta (الأسبوع 1-2)

**الهدف:** التحقق من رحلة المشتري الكاملة مع 50 مستخدم حقيقي ومنح ثقة استثمارية.

**المعيار:** يستطيع 50 طالب التسجيل + الشراء + التعلم + استلام الشهادة بدون تدخل يدوي.

## 🔴 الحرج — لا إطلاق بدونها

| # | البند | المُلكية | الجهد | الاعتمادية |
|---|---|---|---|---|
| 1 | **قفل الحساب بعد 5 محاولات فاشلة** — عمود `users.locked_until` + خدمة | Backend | 1d | — |
| 2 | **اختبار `/checkout` Pest كامل** (5 سيناريوهات) | QA | 1d | factories |
| 3 | **اختبار `/webhooks/{gateway}` idempotency** | QA | 1d | — |
| 4 | **اختبار `RefundService`** (داخل/خارج نافذة 14 يوم) | QA | 1d | factories |
| 5 | **اختبار `CouponService`** (تطبيق + انتهاء + min/max) | QA | 0.5d | factories |
| 6 | **اختبار `GiftService` end-to-end** | QA | 0.5d | factories |
| 7 | **Factories: User/Program/Order/Quiz** | QA | 1d | — |
| 8 | **رحلة Visitor → Buyer E2E بـ Playwright** | QA + Frontend | 2d | Playwright config |
| 9 | **رحلة Student → Certificate E2E** | QA + Frontend | 2d | — |
| 10 | **`generateMetadata` على كل صفحة عامة** | Frontend | 1d | — |

## 🟠 مهم — لا يُمنع لكن يُضعف التجربة

| # | البند | المُلكية | الجهد |
|---|---|---|---|
| 11 | فحص Dashboard sidebar — كل العناصر تظهر بالعربي بدون فجوات | Frontend | 0.5d ✅ مُنجز |
| 12 | إصلاح كل `href="#"` (apps، contact، help) | Frontend | 0.5d ✅ مُنجز جزئياً |
| 13 | "تابع من حيث توقفت" — بطاقة في `/dashboard` | Frontend | 0.5d |
| 14 | "إعادة محاولة الاختبار" CTA من شاشة الفشل | Frontend | 0.3d |
| 15 | تجميل empty states عبر كل لوحات | Frontend | 1d |
| 16 | تحويل `<img>` إلى `next/image` على Hero + بطاقات البرامج | Frontend | 0.5d |
| 17 | تأكيد البريد عند التسجيل (link في الإيميل) | Backend + Ops | 1d (بعد بنية الإيميل) |
| 18 | إصلاح A11Y-001 (touch target الإشعارات) + A11Y-003 (aria-label الـ icon buttons) | Frontend | 0.5d |

## 🎯 معيار قبول G1

- [ ] 50 مدعو يستطيعون التسجيل من رابط دعوة
- [ ] كل طالب يستطيع شراء برنامج واحد على الأقل (Mock gateway مقبول للـ beta)
- [ ] كل طالب يستطيع إكمال درس + اجتياز اختبار + تحميل الشهادة
- [ ] Critical 1-10 منجزة
- [ ] لا أخطاء JS في console عبر الرحلات الـ 5 الرئيسية
- [ ] Lighthouse ≥ 85 على الصفحة الرئيسية

**التقدير:** 12-14 يوم عمل = **أسبوعان** بفريق واحد.

---

# 🟢 G2 — Soft Launch (الأسبوع 3-4)

**الهدف:** فتح المنصة للعموم بدون حملة. الاعتماد على SEO + word-of-mouth + إعلان واحد في X.

## 🔴 الحرج

| # | البند | المُلكية | الجهد |
|---|---|---|---|
| 19 | **مسار "نسيت كلمة المرور"** — `/auth/forgot-password` + `/auth/reset-password` + إيميل | Backend + Frontend + Ops | 2d (يعتمد على بنية الإيميل) |
| 20 | **بنية البريد** — Mailgun/SES + قالب welcome/confirm/reset/refund/cert | Ops + Backend | 2d |
| 21 | **MFA لـ admin / instructor / finance** — `/me/2fa/enroll` + UI | Backend + Frontend | 2d |
| 22 | **Filament admin: Reviews moderation** | Backend | 1d |
| 23 | **Filament admin: Refunds queue** | Backend | 1d |
| 24 | **Filament admin: User management (suspend/ban)** | Backend | 1d |
| 25 | **Tap recurring/one-time integration** | Backend + Ops | 3d |
| 26 | **Tamara BNPL integration (اختياري لكن مُهم للسوق السعودي)** | Backend | 2d |
| 27 | **Plausible Analytics** + لوحة الأحداث الرئيسية | Frontend + Ops | 1d |

## 🟠 مهم

| # | البند | المُلكية | الجهد |
|---|---|---|---|
| 28 | تطبيق tashkeel sweep على ملفات الـ admin/Filament (بعد الفرونت ✅ منجز) | Backend | 0.5d |
| 29 | لوحة المدرب الذاتية: نموذج التقديم + admin queue للموافقة | Backend + Frontend | 3d |
| 30 | كتالوج محتوى المدرب: قائمة برامجه + إحصائيات | Frontend | 2d |
| 31 | Captions/transcripts لكل درس مُنشور (WCAG 1.2.2) | Content + Backend | 2d (مستمر) |
| 32 | Sitemap.xml + robots.txt محدّثة لكل برنامج + مدرب | Frontend | 0.3d |
| 33 | Open Graph images ديناميكية لكل برنامج + شهادة | Frontend | 1d |
| 34 | تجربة طلب استرداد للطالب (form من dashboard/orders) | Frontend | 1d |
| 35 | لوحة الـ super admin: tenants list + plan management | Backend + Frontend | 2d |

## 🎯 معيار قبول G2

- [ ] أي زائر يستطيع التسجيل والشراء بدون دعوة
- [ ] بنية البريد ترسل: welcome + order confirmation + cert + refund
- [ ] الـ MFA متاح وإلزامي لكل دور غير الطالب
- [ ] كل بنود G1 + Critical 19-27 منجزة
- [ ] لوحة Filament تغطي 80% من العمليات اليومية للأدمن
- [ ] أول 100 طلب تمر بدون تدخل دعم

**التقدير:** 16-20 يوم عمل = **3-4 أسابيع**.

---

# 🚀 G3 — Full Launch (الأسبوع 5-8)

**الهدف:** حملة إطلاق رسمية، علاقات عامة، إعلانات مدفوعة، بداية الـ B2B.

## 🔴 الحرج

| # | البند | المُلكية | الجهد |
|---|---|---|---|
| 36 | **Bunny Stream wiring** — كل البرامج المنشورة تنتقل من HTML5 → Bunny | Backend + Ops | 3d |
| 37 | **PWA كامل** — service worker offline + push notifications | Frontend | 2d |
| 38 | **تسريع Lighthouse** — ≥ 95 على Home + Programs (image opt + bundle split + font subset) | Frontend | 2d |
| 39 | **PDPL register published** + breach notification SOP | Ops + Legal | 1d |
| 40 | **Penetration test خارجي** — OWASP ZAP + SAR pentester | Ops + Security | 5d |
| 41 | **CI/CD: tests + coverage > 70% + lint + Lighthouse budgets** | DevOps | 2d |
| 42 | **Backup + DR runbook** — landlord + tenants + S3 attachments | Ops | 1d |
| 43 | **B2B sales flow** — request quote → signed contract → tenant provisioning | Sales + Backend | 4d |
| 44 | **Affiliate dashboard للطرف الآخر** — payout history + W-9-equivalent | Backend + Frontend | 3d |

## 🟠 مهم

| # | البند | المُلكية | الجهد |
|---|---|---|---|
| 45 | برامج تعلم متعدد المستأجرين (B2B onboarding wizard) | Backend + Frontend | 4d |
| 46 | لوحة instructor analytics (enrollments / completion / earnings) | Frontend + Backend | 3d |
| 47 | Live sessions UI كامل (Zoom/Teams integration via API) | Backend + Frontend | 4d |
| 48 | Quiz timer countdown UI | Frontend | 0.5d |
| 49 | Email digests أسبوعية للطلاب النشطين | Backend + Ops | 2d |
| 50 | RSS feed للبرامج الجديدة | Backend | 0.5d |
| 51 | webhooks خارجة (لشركاء B2B) | Backend | 2d |
| 52 | تطبيقات موبايل (RN أو PWA-as-app) | Frontend | 3 weeks (مرحلة منفصلة) |

## 🎯 معيار قبول G3

- [ ] PR launch مع 5 شركاء محتوى ومدربين معروفين
- [ ] أول 10 عملاء B2B موقّعين
- [ ] Lighthouse ≥ 95
- [ ] WCAG 2.1 AA: pass على axe scan
- [ ] Penetration test: clean (أو ثغرات معلومة موثقة + معالَجة)
- [ ] Uptime SLO 99.9% خلال أول 30 يوم
- [ ] الـ Critical 36-44 منجزة

**التقدير:** 25-30 يوم عمل = **4 أسابيع** بعد G2.

---

# 📊 ما بعد الإطلاق (الشهر 3+)

- **A/B testing infrastructure** — قياس تحسينات الصفحة الرئيسية
- **Recommendation engine** — "برامج قد تعجبك" مبنية على enrollments
- **AI study assistant pro** — توسيع `StudyAssistantService` لخطط دراسية مخصصة
- **Group learning / cohorts** — طلاب نفس البرنامج يدرسون مع بعض
- **Mobile native apps** (iOS + Android) — إذا تأكدنا من الـ retention على الويب
- **Affiliate marketplace** — مدربون يجلبون مدربين
- **Partnership API** — منصات أخرى يمكنها sell ومضات programs
- **Multi-language back-office** — للموظفين الذين يفضلون الإنجليزية في الـ admin (الواجهة العامة تبقى عربية)
- **PDPL deep audit** + شهادة Cyber Trust Mark سعودي

---

# 🧮 الجدول الزمني الإجمالي

```
الأسبوع 1-2  →  G1 Closed Beta        (50 طالب + 5 مدربين)
الأسبوع 3-4  →  G2 Soft Launch        (عام بدون تسويق)
الأسبوع 5-8  →  G3 Full Launch        (PR + ads + B2B)
الشهر 3+    →  Growth & retention features
```

**التاريخ المتوقع لـ Full Launch (G3):** **2026-07-07** (8 أسابيع من اليوم).

---

# 💰 ميزانية تقديرية (engineering + ops)

| البند | شهرياً |
|---|---|
| استضافة (Laravel + Postgres + Redis + S3) | 800 SAR |
| Bunny Stream (1TB bandwidth + 100GB storage) | 200 SAR |
| Mailgun / SES (50k emails/month) | 150 SAR |
| Plausible / analytics | 80 SAR |
| Tap merchant fees (~2.5% من المبيعات) | متغيّر |
| Domain + SSL + monitoring (Sentry / Better Stack) | 200 SAR |
| Backups (offsite + S3 versioning) | 100 SAR |
| **المجموع الثابت** | ~1530 SAR/شهر |

(لا تشمل: مرتبات الفريق، أتعاب المدربين، تسويق مدفوع)

---

# 🚦 المخاطر العليا

| المخاطرة | الأثر | التخفيف |
|---|---|---|
| اعتماد على Bunny Stream قبل التحقق من تكلفة الـ bandwidth | تكلفة مفاجئة | مراقبة شهرية + تنبيه على Y%+ |
| Tap recurring API integration معقدة | تأخير G2 | تكامل واحد فقط (one-time) أولاً، recurring مؤجل |
| PDPL audit يكشف مشاكل تخزين بيانات | تأخير G3 | مراجعة قانونية مبكرة (الأسبوع 5) |
| Pen test يكشف ثغرة Critical | تأخير G3 | جلب pentester في G2 ليس G3 |
| Network drop mid-lesson يخسر progress | UX سيئ | حفظ كل 10 ثوان + offline buffer (PWA) |
| فقدان مدرب رئيسي قبل الإطلاق | محتوى أقل | عقود نهائية مع 8+ مدربين قبل G2 |

---

# ✋ قرارات بانتظارك (المالك)

| القرار | لماذا مهم | الأثر إذا تأخر |
|---|---|---|
| **هل المدرب يقدم بنفسه أم نُختاره يدوياً؟** | يحدد ضرورة بناء `/become-instructor` flow كامل | تأجيل بضعة أسابيع |
| **Gift recipient كيف يفعّل** — رابط في الإيميل أم كود يدخله؟ | يحدد بنية الـ email | متوفر اليوم: الاثنين معاً |
| **سعر الشهادة المؤقت إذا الـ Tap recurring اتأخر** | one-time pricing model فقط حتى التكامل | لا أثر — هذا النموذج الحالي |
| **B2B Pricing عام أم على الطلب فقط؟** | يحدد ضرورة بناء سيلف-سيرفس تسعير | الافتراض: عند الطلب |
| **أول 3 شركاء محتوى/مدربين معروفين** | قصة الإطلاق | بدونهم → soft launch فقط |
| **الإطلاق بالـ Mock gateway أم لازم Tap real قبل G1?** | يحدد سرعة الـ Beta | الافتراض: Mock في G1، Tap في G2 |

---

# 📁 المراجع

- `docs/audit/EXECUTIVE_SUMMARY.md` — ملخص المراجعة الـ 8-reviewer
- `docs/audit/01-backend.md` → `08-product.md` — التقارير المفصلة
- `BACKLOG.md` — السجل الزمني لكل المهام المؤجلة
- `docs/00-brand-identity.md` — هوية ومضات + توجيهات بصرية
- `memory/MEMORY.md` — قرارات المالك المُقفلة

---

— نهاية خارطة الطريق. مكتوبة بعربية سادة بدون تشكيل (`feedback_arabic_plain.md`). راجعها وأخبرني بأي تعديل.
