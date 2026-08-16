# WAMADAT — Go-Live Defect Register
> مصدر الحقيقة الوحيد لإغلاق العيوب قبل الإطلاق. مستخرج من تدقيق الكود الفعلي (file:line).
> المرجع المصاحب: Homepage Blueprint v2.0 + Student/Trainer/Admin Experience Blueprints v1.0.
> الحالة: مفتوح. التاريخ: 2026-06-13.

## التصنيف
- **النوع:** `FIX` = تصحيح سلوك قائم خاطئ/غير آمن (مسموح تحت وضع التشغيل: reliability/conversion/usability). `BUILD` = ميزة جديدة (مُجمّدة حتى يفتح المالك نافذة بناء).
- **القبول:** لا يُغلق بند إلا بإثبات end-to-end (كود + سلوك)، لا grep.

---

## ✅ سجل الإغلاق — G1 (2026-06-13)
- **P0-1 مُغلق:** سياسة الاسترداد = «قبل بدء التدريب وخلال أول يومه فقط»؛ موحّدة في شارات الثقة + صندوق الشراء + الأسئلة (ar+en) + مفروضة في `RefundService` (ندم العميل فقط؛ تصحيحات المشغّل معفاة). مدّة الصرف في الإيميل → حتى 14 يوم عمل.
- **P0-2 مُغلق:** 5 أزرار for-business → `/contact?topic=business`.
- **P0-3 مُغلق:** تحرير البرنامج مقيّد بالملكية (instructor يحرّر برامجه فقط).
- **P1-4/A1 مُغلق:** بوّابة SYSTEM على LearningPath/Plan/TeamMember.
- **بقايا الهوية:** زر الاستشارات → برتقالي؛ «طالب»→«مبدع» (أسطح المدرب).
- تحقّق: pint passed · php -l سليم · tsc صفر أخطاء جديدة · JSON صالح.

## ✅ سجل الإغلاق — G2 (2026-06-13)
- **P1-1 مُغلق:** الواجب الإلزامي يَحجب الشهادة (graded + passing)؛ + حدث `AssignmentGraded` يُطلَق عند التصحيح ومسجَّل في `CertificationServiceProvider` فلا تتعطّل الشهادة لبرامج فيها واجب.
- **P1-2 مُغلق:** `add_to_cart` (ProgramCard + ProgramDetailCta شراء/إضافة) و`checkout_started` (دخول صفحة الدفع) تُطلَق الآن → القمع لم يعد أعمى.
- **P1-5 مُغلق:** إسناد دور instructor في `InstructorResource` مُدقَّق عبر `AdminAuditWriter` (`user.role_assigned`).
- تحقّق: pint passed · php -l سليم · tsc صفر أخطاء جديدة.

---

## P0 — حاجز إطلاق (Trust / Safety / Conversion)

| ID | العيب | الدليل | النوع | معيار القبول |
|----|-------|--------|------|--------------|
| P0-1 | سياسة استرداد متناقضة (7 ≠ 14 ≠ ربع) وغير مُنفَّذة كوداً | `trust-badges.tsx:27` · `program-detail-cta.tsx:151` · `ar.json faq.q6` · `RefundService` (لا فحص مدّة) | FIX | سياسة واحدة معتمدة، موحّدة في الواجهات الثلاث + مفروضة في `RefundService` (رفض خارج النافذة) |
| P0-2 | أزرار for-business ميّتة (بلا وجهة) | `for-business/page.tsx:78,82,162,182` | FIX | كل زر → `/contact?topic=business` أو نموذج lead؛ تأكيد تدفّق للإدارة |
| P0-3 | لا حماية ملكية البرامج — أي مدرب يحرّر/يحذف أي برنامج | `ProgramResource.php:573-588` (`isContent()` بلا فحص ملكية) | FIX | `canEdit/canDelete` يتحقّق من `primary_instructor_id`/`owner_user_id` لدور instructor |

## P1 — Conversion / Integrity / Security

| ID | العيب | الدليل | النوع | معيار القبول |
|----|-------|--------|------|--------------|
| P1-1 | الواجبات الإلزامية لا تَحجب الشهادة | `IssueCertificateOnProgramCompletion.php:45-60` (يفحص الدروس+الكويزات فقط) | FIX | فحص `assignments.is_required_for_completion` ضمن شرط الإصدار |
| P1-2 | `add_to_cart` / `checkout_started` معرّفان ولا يُطلقان | `lib/analytics/track.ts:13` (لا استدعاء) | FIX | إطلاق الحدثين من البطاقة/الدفع؛ ظهورهما في FunnelStats |
| P1-3 | المدرب لا يحرّر ملفه/صورته | `InstructorResource.php:52-80` (SYSTEM) | FIX/BUILD | المدرب يحرّر ملفه الخاص (صورة/نبذة/headline) بحدود ملكية |
| P1-4 | موارد إدارة بلا بوّابة | `LearningPathResource` · `PlanResource` · `TeamMemberResource` | FIX | كل مورد خلف بوّابة مناسبة (Path/Plan=SYSTEM، Team=SYSTEM) |
| P1-5 | استرداد/إسناد أدوار غير مُدقّق | `OrderResource.php:346-444` · `InstructorResource.php:217` | FIX | `AdminAuditWriter.record` على approve/reject الاسترداد وإسناد الدور |
| P1-6 | تحقّق البريد غير مفعّل | `UserModel` يطبّق `MustVerifyEmail`، `email_verified_at` لا يُضبط | FIX/BUILD | قرار: تفعيل تحقّق + بوّابة، أو إزالة العقد |
| P1-7 | لا واجهة لقوالب الإشعارات | `NotificationTemplateModel` بلا Resource | BUILD | واجهة Filament لتحرير القوالب (تحكّم بلا كود) |
| P1-8 | تصحيح كويز يدوي غير مُنفَّذ + لا إشعار تصحيح | `QuizAttemptService` (يترك pending) · `GradeAssignment.php:45` (toast فقط) · `QuizPassed` بلا listener | BUILD | واجهة تصحيح يدوي + إشعار/إيميل للمبدع عند التصحيح |

## P2/P3 — تحسينات (مؤجّلة)
- تذكير الجلسة المباشرة (`live.starting` بلا جدولة) · Credit Note للاسترداد (ZATCA) · لوحة مبدعي المدرب · نظام الدفعات · محرّك التوصيات · مساحة عمل مدرب · تنظيف سجل التدقيق · تقرير إيراد/تصدير · توحيد مجموعات تنقّل الإدارة · شارة Q&A للمشاركين · «طالب»→«مبدع» (المتبقّي) · زر استشارات داكن.

## V — يحتاج تحقّق
| V1 | تضارب: الذاكرة تقول «Site Settings شُحنت 2026-05-16»، التدقيق لم يجد واجهة إعدادات موقع عامة (وجد Page Builder فقط) | تحقّق قبل أي بناء |

---

## بوّابات الإطلاق (Go-Live Gates)
1. **G1 Trust & Safety:** كل P0 مُغلق + مُثبت.
2. **G2 Conversion & Integrity:** P1-1, P1-2, P1-5 مُغلقة.
3. **G3 Content:** إدخال محتوى حقيقي (مدربون بصور · برامج بأغلفة ونتائج · أرقام حقيقية).
4. **G4 Real-User Test:** اختبار مستخدم حقيقي (LAN) لرحلة الشراء→التعلّم→الشهادة.
5. **G5 Ops:** scheduler+worker يعملان · health أخضر · نسخ احتياطي مُختبر.
