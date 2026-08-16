# خطة بيئة Staging + ضبط النشر — 2026-07-22

> **الحالة:** مسوّدة معتمَدة للتنفيذ. تُنفَّذ على مراحل؛ بنود «مالك» موسومة صراحةً.
> **المحرّك:** اكتُشف أن `main` يُنشر تلقائيًّا إلى الإنتاج (`api.wmt.sa`) عبر
> تطبيق Laravel Cloud، **ويُطبّق هجرات المستأجرين على Supabase الحيّة تلقائيًّا**.
> لا يوجد حاجز بين الدمج والإنتاج سوى حماية `main` التي فُعّلت اليوم.

---

## 1. الواقع المُثبَت (2026-07-22)

| الحقيقة | الدليل |
|---|---|
| الخلفية تُنشر تلقائيًّا من `main` | `course-builder.js` المنشور مطابق بايتيًّا لالتزام دُمج 08:36؛ CSP `/instructor` يعكس `a36818c` |
| النشر يُطبّق هجرات المستأجر على الإنتاج | `tenant_wamadat.migrations` batch 81 = `add_optimistic_lock_to_curriculum` (من #60) |
| الإنتاج **سليم ومتّسق حاليًّا** | P1 نزل كاملًا: `idempotency_keys` + `lock_version`/`updated_by_user_id` على ٥ جداول في سكيما المستأجر |
| لا بيئة staging | `docs/07-deployment.md` يصف `staging.wamadat.dev` على Railway = وثيقة لاغية لبنية سابقة |
| الواجهة يدويّة | لا نشرة Vercel Preview واحدة رغم عشرات الـPRs؛ كلها Production يدويّة |
| `deploy/README.md:51` قديم | يوثّق `migrate --database=landlord` فقط، بينما هجرات المستأجر تُطبَّق فعلًا ⇒ الأمر الفعلي في اللوحة يشمل `tenants:artisan migrate` |

**الخلاصة:** لا حريق — الإنتاج صحّي. لكن خطّ النشر يطبّق DDL على قاعدة حيّة بلا
بروفة ولا حاجز يدوي. هذه الخطة تبني الحاجز.

---

## 2. المرحلة ٠ — إيقاف النشر التلقائي فورًا (مالك · ~٥ دقائق)

النشر التلقائي **لا يُوقَف من الكود** — مفتاحه في لوحة Laravel Cloud (لا CLI/MCP
لها هنا؛ وتوكن `gh` الحالي لا يملك نطاق تطبيقات GitHub). الإجراء:

1. **Laravel Cloud → المشروع → Settings → Deployments** (أو Environment → `production`).
2. أطفئ **"Auto deploy on push"** / **"Deploy when code is pushed to `main`"**.
3. النتيجة: الدمج إلى `main` يعود بلا أثر على الإنتاج؛ النشر يصير بضغطة **Deploy** يدويّة.
4. تحقّق: ادمج تغييرًا وثائقيًّا صغيرًا وتأكّد أن `api.wmt.sa` **لم** يُعِد النشر.

> حتى تُطفأ: **لا تدمج أي شيء وظيفي إلى `main`** — أي دمج يصل الإنتاج مباشرة.
> بنود الأمان المؤجّلة (Dependabot، الفحص المجدول) تنتظر هذه المرحلة.

**بند تحقّق مرافق (مالك):** افتح **Deploy Command** في اللوحة وانسخه حرفيًّا إلى
`deploy/README.md` (الموثّق قديم). تأكّد أنه يشمل — بهذا الترتيب:
```
php artisan migrate --database=landlord --force
php artisan tenants:artisan 'migrate --force'      # ← لا بدّ منه؛ الإثبات أعلاه أنه يعمل
php artisan config:cache && route:cache && view:cache
```

---

## 3. المعمارية المستهدفة

```
        دمج PR  ─────────────►  main (محميّة، ٥ فحوص خضراء)
                                  │  (auto)
                                  ▼
                          ┌───────────────┐   بيانات اصطناعية (Seeders)
                          │   STAGING      │   لا PII إنتاجيّة إطلاقًا
                          │  Laravel Cloud │
                          │ api-staging.…  │◄─ Supabase staging (فرع أو مشروع منفصل)
                          │  Vercel staging│
                          │ staging.wmt.sa │
                          └───────┬───────┘
                                  │  Smoke E2E آليًّا (المنشئ + الطالب)
                                  ▼  ✅ إن نجح
                       ┌─────────────────────┐
                       │  ترقية يدويّة للمالك  │  ← الحاجز الوحيد للإنتاج
                       │  (Deploy على prod أو │
                       │  fast-forward production)
                       └──────────┬──────────┘
                                  ▼
                          الإنتاج api.wmt.sa + beta.wmt.sa
```

**القرار الجوهري — كيف يُنشر الإنتاج بعد بناء staging؟** خياران:

| # | الآلية | كيف تعمل | ملاءمة |
|---|---|---|---|
| **أ (موصى به)** | فرع `production` | Laravel Cloud الإنتاجي يراقب `production` لا `main`. `main`→staging تلقائيًّا. الترقية = `git fast-forward production ← main` بعد نجاح staging. | يجعل «الترقية» فعلًا صريحًا في git، مدقّقًا، وقابلًا للرجوع بـrevert. لا لبس. |
| ب | Deploy يدوي على prod | الإنتاج يبقى على `main` لكن Auto-deploy مطفأ؛ المالك يضغط Deploy لـSHA بعينه. | أبسط إعدادًا، لكن «أي SHA نُشر» يعيش في اللوحة لا في git. |

الخطة تفترض **(أ)**. أُجهّز فرع `production` عند `a2fb975` (وهو المنشور حرفيًّا الآن)
فتصير إعادة توجيه اللوحة إليه بلا فجوة.

---

## 4. المراحل

### المرحلة ١ — تجهيز بنية staging (مالك يُنشئ · أنا أُعدّ)
- **Supabase staging:** فرع من مشروع `wamadat-platform` (فرانكفورت) — الأرخص والأنسب —
  أو مشروع منفصل لعزل كامل. القرار للمالك حسب الكلفة. أُجهّز سكربت الهجرة + البذر.
- **Laravel Cloud staging environment:** بيئة ثانية في نفس المشروع بدومين
  `api-staging.wmt.sa`. أُجهّز مانيفست متغيّرات البيئة (نسخة من الإنتاج مع أسرار
  staging + `APP_ENV=staging` + `PANEL_2FA_DISABLE=true` لتمكين E2E).
- **Vercel staging:** دومين `staging.wmt.sa` على نفس المشروع، Environment=Preview
  مربوط بـ`main`، `NEXT_PUBLIC_API_BASE_URL=https://api-staging.wmt.sa`.
- **Cloudflare DNS:** سجلّا `staging` و`api-staging` تحت `wmt.sa`.

### المرحلة ٢ — وصل نشر staging (أنا)
- staging يراقب `main` (Auto-deploy مُبقًى هنا — آمن لأنه ليس الإنتاج).
- **Deploy command لـstaging** يشمل هجرات landlord + المستأجر + **بذر بيانات اصطناعية**:
  `TestAccountsSeeder` + `BigDemoSeeder` + `E2eSeeder` (لا لقطة إنتاج — PDPL).
- عامل طابور + Scheduler على staging (مطابقان للإنتاج ليُختبرا فعلًا).
- تحقّق: دمج تجريبي → staging يبني ويهاجر ويبذر → فحص دخان يدوي.

### المرحلة ٣ — أتمتة فحص الدخان (أنا · يُغلق بند BACKLOG)
- وظيفة CI جديدة `E2E (Course Builder)` تُشغّل حزمة `--project=builder` (٣٦ اختبارًا)
  ضد staging: `PANEL_2FA_DISABLE` متاح، وحسابات الاختبار مبذورة، وتهيئة الكورس عبر
  `builder:e2e-fixture`. يُغلق البند المسجّل في `BACKLOG.md`.
- فحص دخان بعد كل نشر staging لأدوار Admin/Instructor/Student (المنشئ، Autosave،
  Draft Recovery، Student Player).

### المرحلة ٤ — خطّ الترقية للإنتاج (أنا أُجهّز · مالك يُنفّذ)
- إنشاء فرع `production` عند SHA الإنتاج الحالي `a2fb975`.
- إعادة توجيه Laravel Cloud الإنتاجي: يراقب `production` بدل `main` (مالك، لوحة).
- Runbook الترقية: بعد اخضرار staging → `git checkout production && git merge --ff-only main && git push`
  → Laravel Cloud ينشر → تحقّق إنتاج → عند الخلل: `git revert` + إعادة نشر.
- الواجهة: `vercel --prod` يدويًّا من `frontend/` بعد ترقية الخلفية (كما هو).

### المرحلة ٥ — أتمتة الأمان (أنا · بعد وجود staging)
الآن آمنة لأن كل تغيير يمرّ staging أولًا:
- `Dependabot` (composer + pnpm) — PRs ترقية استباقيّة بدل تحمير CI المفاجئ.
- نقل فحص الثغرات المتغيّرة (`composer audit` / `pnpm audit`) إلى workflow **مجدول يومي**
  يفتح Issue/PR، وإخراجه من بوابة الدمج.
- إبقاء فحوص الجودة الأساسية (Pint، PHPStan، Pest، البناء، حارس المونوريبو) في بوابة الدمج.

---

## 5. تقسيم المسؤوليّات

| مالك (لوحات لا أصلها) | أنا (كل ما حولها في الكود) |
|---|---|
| إطفاء Auto-deploy على الإنتاج | مانيفست متغيّرات بيئة staging |
| إنشاء Supabase staging (فرع/مشروع) | Deploy commands (staging + prod) |
| إنشاء Laravel Cloud staging env + دومين | وصل seeders الاصطناعية |
| أسرار staging (مفاتيح، DSN، بوابات test) | وظيفة CI للمنشئ + فحص الدخان |
| Cloudflare DNS (staging, api-staging) | Runbook الترقية + تحديث الوثائق |
| إعادة توجيه prod إلى `production` | إنشاء فرع `production` عند `a2fb975` |
| نسخ Deploy Command الفعلي للتوثيق | Dependabot + الفحص المجدول |

---

## 6. الكلفة والمخاطر

- **كلفة:** بيئة Laravel Cloud ثانية + Supabase staging = كلفة شهرية إضافية.
  فرع Supabase أرخص من مشروع منفصل. القرار للمالك.
- **PDPL:** staging لا يحمل بيانات إنتاج — بذر اصطناعي فقط. لا نسخ لقطة PII.
- **الأسرار:** بوابات دفع staging على **وضع الاختبار** فقط (لا مفاتيح إنتاج حيّة).
- **الرجوع:** آلية `production` fast-forward تجعل أي نشر قابلًا للـrevert بأمر git واحد.

---

## 7. أول ما يُنفَّذ فور الموافقة

1. **مالك:** المرحلة ٠ (إطفاء Auto-deploy) — يحرّر `main` للعمل الآمن.
2. **أنا:** فرع `production` عند `a2fb975` + مانيفست بيئة staging + Deploy commands (بلا لمس لوحة).
3. **مالك:** المرحلة ١ (بنية staging) بالمانيفست الذي أُسلّمه.
4. **أنا:** المرحلتان ٢-٣ (وصل + أتمتة) والتحقّق.
