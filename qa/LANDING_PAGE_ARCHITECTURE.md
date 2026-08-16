# LANDING_PAGE_ARCHITECTURE — منشئ صفحات الهبوط

> **تصميم للاعتماد قبل أي migration.** 2026-07-16. الدفعة 4.
> **مبدأ حاكم (قاعدة 14): توسعة الموجود لا إعادة بنائه.** المنصّة تملك ~80% من
> النظام المطلوب أصلًا — هذه الوثيقة توثّق الموجود، تحلّل الفجوة، وتصمّم التوسعة.

---

## 1. الموجود فعلًا (لا يُبنى من جديد)

| المكوّن | الوصف | الملف |
|---|---|---|
| **صفحات CMS عامة** | `PageModel`: `sections` (jsonb)، `status` (draft/published/archived)، `slug`، `seo_title`، `seo_description`، `show_in_nav`، `sort_order` | Catalog/…/PageModel + PageResource |
| **صفحات هبوط للبرامج** | `ProgramLandingPageModel`: `program_id` FK، `slug` فريد، `sections` jsonb، SEO، `is_active`، `is_default` | ProgramLandingPageModel |
| **إدارة في Filament** | `LandingPagesRelationManager` تحت ProgramResource + `PageResource` مستقلّ | RelationManagers/LandingPagesRelationManager |
| **مُصيِّر الواجهة** | `dynamic-blocks.tsx` — **14 نوع block**: hero, rich_text, image, image_text, video, faq, cta, stats, program_grid, lab_grid, initiative_grid, path_grid, trainer_grid, custom_code | frontend/components/cms/ |
| **العرض** | `PageController` + catch-all `/[slug]` + `/lp/[slug]` | frontend/app/[locale]/[slug] |
| **الأمان** | `custom_code` مُعقَّم بـDOMPurify وقت الحفظ (M-3، يسمح iframe يحذف script) | تدقيق سابق |
| **SEO** | sitemap ديناميكي يمرّ على البرامج/المدربين/خدمات بلس + حقول SEO للصفحة | frontend/app/sitemap.ts |

**الخلاصة:** يستطيع المدير **الآن** إنشاء صفحة هبوط لبرنامج (مخصّصة/افتراضية)، ترتيب
blocks، ضبط SEO، ونشرها. النظام حيّ ومُصان.

---

## 2. تحليل الفجوة (مقابل متطلّباتك الـ30+)

### ✅ موجود
مسودة/منشور · slug فريد داخل المستأجر · SEO title/description · ترتيب الأقسام ·
صفحة افتراضية مقابل مخصّصة (`is_default`) · ربط بالبرنامج · RTL/عربي · sitemap ·
تعقيم HTML/XSS · مُصيِّر allowlist (أنواع معروفة فقط).

### 🟠 ناقص أو جزئي (نطاق الدفعة 4)
| # | الفجوة | الأولوية |
|---|---|---|
| G1 | **ربط بأي كيان** (مدرب، خدمة بلس، استشارة، هدية، حملة، B2B) — حاليًا البرامج فقط | عالية |
| G2 | **معاينة قبل النشر** (preview token آمن دون نشر) | عالية |
| G3 | **versioning للأقسام** (`version` لكل نوع + إصلاح تدريجي دون كسر صفحات قديمة) | عالية |
| G4 | **مراجعات + استعادة** (page revisions + rollback) | متوسطة |
| G5 | **canonical URL + noindex/index** | متوسطة |
| G6 | **جدولة النشر/الإلغاء** (`published_at`/`unpublished_at`) | متوسطة |
| G7 | **قوالب** (حفظ كقالب + إنشاء من قالب) | متوسطة |
| G8 | **نسخ صفحة موجودة** | منخفضة |
| G9 | **رؤية القسم** (إظهار/إخفاء + رؤية حسب الجهاز desktop/mobile) | متوسطة |
| G10 | **tracking id لكل قسم** (تحليلات) | منخفضة |
| G11 | **تحويل لرابط خارجي** (redirect بدل عرض) ضمن allowlist | منخفضة |
| G12 | **أنواع أقسام إضافية**: السعر قبل/بعد · عدّاد زمني · الدفعات · مواعيد الجلسات · الموقع · معرض صور · ملفات للتنزيل · نموذج اهتمام · واتساب · ضمان/استرداد · المتطلّبات · الشهادة/التحقّق · شركاء | متوسطة |

---

## 3. التصميم (توسعة، لا استبدال)

### 3.1 قاعدة البيانات (additive — لا كسر)
**قرار: توحيد على نموذج واحد مرن** `landing_pages` (polymorphic) بدل تشعّب
`pages` + `program_landing_pages`. الترحيل تدريجي (الجدولان يبقيان، نضيف طبقة موحّدة).

جدول `landing_pages` الجديد (كلها إضافية):
```
id (uuid)
linkable_type (nullable)   // App\...\ProgramModel | InstructorProfileModel | PlusServiceModel | ... | null=صفحة حرّة
linkable_id   (nullable)
slug (unique per tenant)
title
sections (jsonb)           // مصفوفة الأقسام (بنية §3.2)
schema_version (int)        // نسخة مخطّط الأقسام (G3)
status (draft|published|archived)
is_default (bool)           // الصفحة الافتراضية للكيان
seo_title, seo_description, canonical_url (nullable), is_indexable (bool default true)  // G5
published_at, unpublished_at (nullable)   // G6 جدولة
external_redirect_url (nullable)          // G11 ضمن allowlist
preview_token (nullable, uuid)            // G2
created_by, updated_by
timestamps, soft_deletes
index (linkable_type, linkable_id, status), unique(slug)
```
جدول `landing_page_revisions` (G4): `id, landing_page_id FK, sections jsonb, schema_version, created_by, created_at` — لقطة عند كل نشر.
جدول `landing_page_templates` (G7): `id, name, sections jsonb, schema_version, is_system` — قوالب قابلة لإعادة الاستخدام.

> **لا حذف** لـ`pages`/`program_landing_pages` — طبقة توافق تقرأ منهما حتى ترحيل البيانات.

### 3.2 مخطّط القسم (JSON schema مُتحقَّق)
كل قسم كائن مُلزَم بالتحقّق (لا بيانات حرّة):
```json
{
  "type": "hero",              // من allowlist معروف فقط
  "version": 1,                 // نسخة هذا النوع (G3)
  "settings": { "bg": "jet", "align": "center" },
  "content": { ... },          // مُتحقَّق حسب النوع + نسخته
  "visibility": { "enabled": true, "devices": ["desktop","mobile"] },  // G9
  "sort": 3,
  "schedule": { "from": null, "to": null },   // اختياري
  "tracking_id": "hero-main"    // G10
}
```
التحقّق: خادميًا عبر **قواعد لكل (type, version)** (rule set في PHP) — يُرفض أي نوع
غير معروف أو محتوى غير مطابق. الأنواع الجديدة (G12) تُضاف كـ(type, version) مع قاعدة تحقّق.

### 3.3 versioning (G3 — لا تنكسر الصفحات القديمة)
- `schema_version` على الصفحة + `version` على كل قسم.
- عند تطوير مكوّن: نضيف `version` جديدة **مع إبقاء المُصيِّر للنسخة القديمة** (migration
  للأقسام كسلة، لا إجباري). مُصيِّر الواجهة يختار المكوّن حسب `type@version`.

### 3.4 API contracts
```
# عام (SSR)
GET  /api/v1/pages/{slug}                  → الصفحة المنشورة (بعد تحقّق)، 404 لغيرها
GET  /api/v1/pages/preview/{token}         → معاينة (G2) — منشورة أو مسودة، لا فهرسة، لا كاش
# إدارة (Filament يستدعيها داخليًا؛ عقود واضحة للاختبار)
POST /admin … (Livewire)                   → إنشاء/تعديل/نشر/جدولة/نسخ/حفظ كقالب/استعادة نسخة
```
الاستجابة العامة تُرجع فقط `sections` بعد التحقّق + بيانات SEO — لا حقول داخلية.

### 3.5 Renderer في Next (توسعة dynamic-blocks الموجود)
- **يستقبل sections مُتحقَّقة فقط**؛ يختار مكوّنًا حسب `type@version`.
- **يرفض الأنواع غير المدعومة** → fallback واضح (لا تنفيذ).
- **لا JS/HTML غير موثوق**: `custom_code` يبقى مُعقَّمًا خادميًا (DOMPurify)؛ لا `eval`،
  لا سكربت عميل. allowlist صارم.
- **SSR + metadata** (title/description/canonical/noindex من الصفحة).
- **معاينة آمنة** عبر preview token (noindex، `Cache-Control: no-store`).
- **كاش + إبطال عند النشر** (revalidate tag بـslug).
- رؤية الجهاز (G9) تُطبَّق CSS/شرطيًا؛ منع اختلاف hydration.

### 3.6 صلاحيات + تدقيق + عزل
- كل عملية خلف صلاحية (`pages.manage`)؛ العزل بالمستأجر مضمون (كل شيء على اتصال tenant).
- كل تغيير → `audit_logs` (من، متى، المستأجر، قبل/بعد بلا أسرار).
- `slug` فريد **داخل المستأجر** فقط (لا تسريب/تصادم بين المستأجرين).

### 3.7 SEO + sitemap
- الصفحات المنشورة `is_indexable=true` تدخل sitemap تلقائيًا (توسعة `sitemap.ts`).
- `noindex` → تُستبعد + meta robots noindex.
- `canonical_url` يمنع duplicate-content الضارّ (صفحة هبوط مقابل صفحة البرنامج الأصلية).

---

## 4. خطة الترحيل (دفعات صغيرة، لا كسر)
1. **دفعة 4أ** — migration إضافية (`landing_pages` + revisions + templates) + نماذج + سياسات + اختبارات schema. لا تغيير واجهة بعد.
2. **دفعة 4ب** — Renderer موحّد في Next (توسعة dynamic-blocks) + `/pages/{slug}` + preview + الأنواع الحالية الـ14. اختبارات تصيير.
3. **دفعة 4ج** — Filament: builder موحّد (أقسام قابلة للترتيب/الإخفاء/الجدولة + معاينة + نسخ + قالب + مراجعات + SEO). اختبارات Filament.
4. **دفعة 4د** — الأنواع الجديدة (G12) نوعًا نوعًا، كلٌّ (type, version) + قاعدة تحقّق + مكوّن + اختبار.
5. **دفعة 4هـ** — ترحيل بيانات `pages`/`program_landing_pages` → `landing_pages` (سكربت idempotent) + redirects للروابط القديمة. لا حذف حتى التأكّد.

## 5. معايير القبول (E2E — المرحلة 12)
- مدير الأكاديمية ينشئ صفحة هبوط لأي كيان **بلا مطوّر** · يرتّب/يخفي الأقسام · يعاين قبل
  النشر · تظهر على نطاق المستأجر الصحيح فقط · لا JS مخصّص · لا تكسر الصفحة الافتراضية ·
  SEO سليم · تعمل على الجوال · تمرّ الوصول الأساسي + البناء + CI.

---

## قرار مطلوب منك
هذا التصميم **يوسّع نظامك القائم** (Page + ProgramLandingPage + dynamic-blocks) بدل
إعادة بنائه. أعتمده؟ وأي دفعة أبدأ (**أقترح 4أ**: الـmigration الإضافية + النماذج +
اختبارات المخطّط — لا واجهة، لا كسر، قابلة للتراجع)؟

---

## 6. سجل التنفيذ الفعليّ (محدّث)

| الدفعة | ما سُلّم | PR | الحالة |
|---|---|---|---|
| 4-تصميم | هذا المستند | #32 | ✅ مدموج |
| 4أ | migration `landing_pages`(+revisions+templates) + `LandingPageModel` + عزل + اختبارات schema | #33 | ✅ مدموج |
| 4ب | خدمة عامّة `/catalog/landing-pages/{slug}` + `/preview/{token}` + `LandingSectionPresenter` (allowlist) | #34 | ✅ مدموج |
| 4ج | تصيير Next عبر المُصيِّر المشترك + مسار المعاينة `/lp/preview/{token}` + `resolve.ts` | #35 | ✅ مدموج |
| **4د** | **لوحة Filament: `LandingPageResource` (builder موحّد + جدولة + معاينة + نسخ + نشر/إيقاف + SEO)** | — | ⏳ هذه الدفعة |

### 6.1 قرار حاسم — مصالحة شكل الأقسام إلى `{type, data}`
أثناء 4د اكتُشف **عدم تطابق عقد** كان سيُصيِّر الصفحات فارغة: مُقدِّم 4ب كان يُخرج
`{type, settings, content, version}`، بينما (أ) المُصيِّر المشترك `page-sections.tsx`
يقرأ **`section.data`** المسطّح، و(ب) حقل Filament `Builder` يكتب أصلًا `{type, data}`.
لو رُبط الـBuilder بالجدول مباشرةً لخرجت كل الأقسام فارغة.

**القرار (الأفضل للمنصّة):** توحيد خطّ الأنابيب كلّه على الشكل **المُثبَت الذي يعمل
طرفًا-لطرف**: `{type, data}` — نفس ما يكتبه Page Builder القديم ويقرأه المُصيِّر.
- `LandingSectionPresenter` الآن يُخرج `{type, data, visibility, sort}`؛ ويَطوي أي صفّ
  قديم `settings/content` داخل `data` تلقائيًا (توافق خلفيّ، `content` يفوز عند التعارض).
- محوّل الواجهة `landingSections()` صار تمريرًا مطابقًا للنوع (لا دمج).
- الفائدة الجانبية: ترحيل 4هـ يصير **نسخًا مباشرًا** بلا تحويل شكل (المصدر أصلًا `{type,data}`).

### 6.2 ما يضمنه 4د باختبارات فعليّة
- المورد **مُسجّل** في لوحة /admin ومرئيّ لمالك الأكاديمية فقط (بقيّة الطاقم لا يرونه).
- **حارس عدم الانحراف:** الكتل القابلة للتأليف (`PageResource::blockTypes()`) == `ALLOWED_TYPES`
  بالضبط — فلا كتلة تُصيَّر فارغة ولا نوع مسموح بلا أداة تأليف.
- `preview_token` يُولَّد تلقائيًا عند الإنشاء؛ زر «معاينة» يفتح `/lp/preview/{token}` (يعمل اليوم).
- الخدمة تُرجع `data.sections[].data.*` المسطّح (14 اختبار Pest، 37 تأكيدًا).
- **المسار الحيّ `/lp/{slug}` موصول**: موحّد-أولًا (`fetchUnifiedLandingPage`) مع fallback للنظام
  القديم — فزرّ «نشر» يُنتج صفحة **يفتحها الزائر فعلًا** (أُغلقت ثغرة راجعها الناقد العدائي:
  المسار كان يخدم القديم فقط و`fetchUnifiedLandingPage` بلا مستدعٍ).
- **إصلاحان من المراجعة العدائية:** (P2) النسخ يفحص بـ`withTrashed()` فلا يصطدم بـslug محذوف ناعمًا؛
  (P3) الصفحة المستقلّة تُطبّع `is_default=false`.

### 6.3 ما تبقّى بعد 4د
- **4هـ** — ترحيل بيانات `pages` + `program_landing_pages` → `landing_pages` (سكربت idempotent)
  + redirects + إزالة الـfallback القديم بعد التأكّد. لا حذف للجداول القديمة حتى التأكّد (القاعدة 5).
- إثراء اختياريّ: صفحة موحّدة مرتبطة ببرنامج تعرض CTA التسجيل اللاصق (تحتاج توسعة payload الخدمة).
- قوالب + مراجعات (الجدولان جاهزان من 4أ) كواجهة إدارية — دفعة لاحقة.
