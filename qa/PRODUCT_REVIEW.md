# مراجعة المنتج — جاهزية البيتا (2026-07-17)

منهج: مراجعة على مستوى الشيفرة (خرائط المسارات + موارد الإدارة + مصادر البيانات). ما لم يُشغَّل في متصفّح حيّ مُعلَّم «يحتاج تحقّق runtime». المنصّة عربية-RTL فقط، الهوية Jet/Orange/Tajawal، النموذج: برامج مفردة + هدايا + استشارات + B2B + أفلييت (لا اشتراكات، لا حزم).

## 1. رحلات المستخدم (62 مسارًا تحت `app/[locale]`)

| الشخصية | الرحلة | المسار → API | الحالة |
|---|---|---|---|
| زائر | تصفّح/بحث | `/programs` `/programs/[slug]` `/search` `/categories` `/instructors` `/consultations` `/plus` `/for-business` → catalog | موصولة (server + ISR) |
| زائر | صفحات ثابتة | `/about` `/contact` `/help` `/privacy` `/terms` `/refund-policy` `/security` | موصولة |
| زائر | هبوط | `/[slug]` (pages) · `/lp/[slug]` (موحّد-أولًا + fallback بعد 4د) · `/lp/preview/[token]` | موصولة |
| طالب | تسجيل/دخول/MFA | `/sign-up` `/sign-in` `/forgot-password` `/reset-password` → auth (bearer+cookie) | موصولة |
| طالب | سلّة→دفع | `/cart` `/checkout` `/checkout/return|success|bank-transfer` `/cart/restore/[token]` | موصولة (idempotency مؤكّد) |
| طالب | تعلّم | `/learn/[slug]` `/learn/quizzes/[id]` → curriculum/progress/quiz | موصولة |
| طالب | لوحة | `/dashboard/{today,my-programs,orders,wallet,certificates,achievements,assignments,live-sessions,messages,notifications,profile,settings,support}` | موصولة (17 صفحة) |
| طالب | استشارة | `/consultations` `/consultations/track` → submit/pay/track (IDOR مغلق بالتوكن) | موصولة (+ إصلاح FIN-1) |
| طالب | بلس | `/plus` `/plus/services/[slug]` `/plus/sellers/[id]` `/plus/orders/return` `/dashboard/plus/*` | موصولة |
| مدرّب | بوّابة ويب | `/trainer` `/trainer/attendance` `/trainer/messages` + لوحة `/instructor` Filament | موصولة |
| إدارة | لوحة `/admin` | 50 موردًا، 12 مجموعة | موصولة (RBAC للطاقم) |

**ملاحظة:** الرحلات موصولة على مستوى الشيفرة (المسار موجود + الـAPI مُعرَّف). يبقى **تحقّق runtime** (متصفّح حيّ ضدّ api.wmt.sa) لكل رحلة كبند قبل الإطلاق العام — أوصى به المدقّق ولم يُجرَ في هذه الجولة.

## 2. تصنيف موارد الإدارة (50 موردًا / 12 مجموعة)

اللوحة منظّمة جيّدًا (دفعة 3 رتّبتها). المجموعات: المستخدمون · المحتوى (8) · المبيعات (5) · المالية (5) · التسويق (5) · التعلّم والتقييم (5) · الموقع (6) · الدعم (2) · المجتمع (1) · الحوكمة (3) · ومضات بلس (4) · Super (4).

| تصنيف | موارد | ملاحظة |
|---|---|---|
| **أساسية** | Program, Order, Invoice, User, Lesson, Quiz, Certificate*, Payout, Coupon, SupportTicket, Consultation... | جوهر التشغيل |
| **تحتاج تحسين** | كل موارد الجداول: أضِف eager-load (N+1، انظر PERFORMANCE_AUDIT) | سطر واحد لكلٍّ |
| **تداخل/تقارب** | **`PageResource` + `LandingPageResource` + `LandingPagesRelationManager`** = 3 أنظمة صفحات | التقارب في 4هـ (لا حذف قبل الترحيل) |
| **يمكن إخفاؤه للإطلاق** | `FailedJobResource`, `EmailOutboxResource` (تشغيليّة داخلية) — أبقِها للمالك فقط | موجودة بمجموعة «التشغيل» |
| **لا حذف** | لا مورد مرشّح للحذف الآمن الآن — كلّها موصولة | تحقّق الاعتماديات قبل أي حذف |

## 3. فجوات المنتج للإطلاق
- **تحقّق runtime للرحلات** غير مُجرى (الأهمّ — بند قبل الإطلاق العام).
- `/design-system` (484 سطرًا) **غير محصور بالإنتاج** — يجب `notFound()` في production.
- Bulk actions محدودة في بعض الجداول (تصدير/تعليم-مقروء) — تحسين ما بعد الإطلاق.
- تقارير المالك: widgets موجودة (Revenue/Enrollments/TopPrograms/Health) — جيّدة للبيتا.

## 4. أهمّ 20 تحسينًا قبل الإطلاق (أثر/تكلفة)
1. تحقّق runtime لكل رحلة أساسية ضدّ api.wmt.sa (H/M)
2. حصر `/design-system` بغير الإنتاج (H/L)
3. مفاتيح الإنتاج الحقيقية (Tap/Resend/Sentry DSN) — إجراء مالك (H/L)
4. تأكيد Filament RBAC لكل مورد (H/M) — أمن
5. حارس مبلغ PlusPayout + مسار الوسائط الموقّع (H/M) — أمن
6. توحيد أنظمة الصفحات (4هـ) قبل ازدواج المحتوى (M/M)
7. حالات فارغة/خطأ لكل شاشة لوحة (M/M)
8. تحقّق شكل payload استرداد Tap الجزئي (M/L) — مالك
9. نسخ احتياطي خارج الخادم مُختبَر (H/M) — مالك
10-20. راجع PERFORMANCE_AUDIT (فهرس البحث، N+1، فهارس الترتيب، كاش الشهادة) + OWNER_ACTIONS.

## 5. أهمّ 10 تحسينات بعد الإطلاق
Push notifications للجوّال · بحث full-text مُفهرَس · bulk actions موسّعة · كاش الشهادات · لوحة تحليلات أعمق · تقارير مالية مجدولة · A/B لصفحات الهبوط · قوالب/مراجعات الهبوط · تحسين رحلة الدفع (deep-link) · اختبارات E2E موسّعة.

## 6. صفحات: حذف/دمج/إخفاء
- **حذف:** لا شيء الآن (كلّها مستخدمة).
- **دمج/تقارب:** 3 أنظمة صفحات → `landing_pages` (4هـ، بعد ترحيل آمن).
- **إخفاء:** `/design-system` في الإنتاج؛ موارد التشغيل الداخلية للمالك فقط.
