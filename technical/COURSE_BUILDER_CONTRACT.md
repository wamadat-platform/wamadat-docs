# عقد منشئ الكورس المجمّد — Course Builder Contract (P1·3)

> **مجمّد:** الواجهة (Livewire/JS) مكتوبة على هذه المفاتيح بالضبط. لا يُغيَّر إلا **إضافةً** (حقل جديد)، ولا يُعاد تسمية/توظيف مفتاح قائم — حتى لا تُعاد كتابة الواجهة.

## استجابة موحّدة لكل Action (`BuilderActionResponse`)
كل عملية منشئ (create/update/delete/reorder/move/saveSettings/restoreDraft) تُنفَّذ عبر `BuilderAction::run()` وتُعيد **نفس الشكل**:

```json
{
  "success": true,
  "operation_id": "<uuid>",
  "entity_id": "<id|null>",
  "version": 4,
  "saved_at": "2026-07-21T10:00:00+00:00",
  "retryable": false,
  "validation_errors": { "field": ["message"] },
  "conflict": null,
  "error_code": null,
  "message": ""
}
```

## رموز الأخطاء المميَّزة (`BuilderErrorCode`) — 419 ليست شمّاعة
| الرمز | HTTP | سياسة إعادة المحاولة |
|---|---|---|
| `csrf_expired` | 419 | **refresh_once** (تجديد ثم إعادة مرة) |
| `session_expired` | 401 | none (شاشة دخول) |
| `permission_denied` | 403 | none |
| `resource_gone` | 410 | none (حُذف العنصر) |
| `version_conflict` | 409 | none (تعارض نافذتين) |
| `validation_error` | 422 | none |
| `operation_in_progress` | 409 | **auto** (نفس المفتاح قيد التنفيذ — انتظر ثم أعد) |
| `idempotency_conflict` | 409 | none (نفس المفتاح ببيانات مختلفة — خطأ عميل) |
| `network_error` | — | **auto** (إعادة تلقائية بتراجع) |

## تعارض النسخة (Optimistic Lock)
- كل كيان قابل للتعديل يحمل `lock_version` (يزداد مع كل تحديث ناجح) + `updated_by_user_id`.
- كل تحديث يُرسل النسخة التي حمّلها العميل عبر `ConcurrencyGuard::checkedUpdate($model, $clientVersion, $attrs)`.
- عند الاختلاف → `409 version_conflict`، **بلا استبدال صامت**، مع حمولة `conflict`:
```json
{
  "deleted": false,
  "client_version": 3,
  "current_version": 4,
  "conflicting_fields": ["title_ar"],
  "current_updated_at": "2026-07-21T09:59:00+00:00",
  "current_updated_by": "<user id>"
}
```

## تجديد الجلسة/CSRF
`POST /builder/session/refresh` — معفى من CSRF (سبب استدعائه توكن قديم) لكنه يتحقّق من الجلسة:
- جلسة صالحة → `200 { session_valid: true, csrf_token }` → أعِد المحاولة **مرة واحدة** بنفس idempotency key.
- جلسة منتهية → `401 { error_code: session_expired }` → شاشة الدخول/الاستعادة.

## Idempotency
- الإنشاء/النسخ/النقل/إعادة الترتيب/إعادة محاولة 419 تمرّ عبر `IdempotencyService::execute(user, operation, key, payload, work)`.
- نفس المفتاح+البيانات → النتيجة الأولى بلا تنفيذ · بيانات مختلفة → رفض · متزامن → in_progress.

## نقطة الدخول الخلفية (`BuilderActionService`)
كل طفرة في المنشئ تمرّ عبر خدمة واحدة تُركّب العقد كاملًا (استجابة موحّدة + idempotency + تفاوض النسخة + عقد الأدوار + **نفس** دماغ الحفظ `CurriculumDetailSync` المستخدم في مسار Filament — بلا مسار كتابة موازٍ):

| العملية | التوقيع | التفاوض/الحماية |
|---|---|---|
| إنشاء عنصر | `createItem(actor, program, module, data, key)` | idempotent · النشر للمشرف فقط |
| تحديث عنصر | `updateItem(actor, program, lesson, clientVersion, data, key)` | تفاوض النسخة (409) + مزامنة التفصيل ذريًّا |
| حذف عنصر | `deleteItem(actor, program, lesson, clientVersion, key)` | تفاوض النسخة + حذف قسريّ للتفصيل المرتبط |
| حفظ وحدة | `saveModule(actor, program, module, clientVersion, data, key)` | تفاوض النسخة |
| ترتيب الوحدات | `reorderModules(actor, program, orderedIds, key)` | idempotent · رفض مجموعة غريبة/ناقصة |
| ترتيب الدروس | `reorderLessons(actor, program, module, orderedIds, key)` | idempotent |
| نقل درس | `moveLesson(actor, program, lesson, targetModule, position, expectedVersion, key)` | **تفاوض النسخة** + رفض النقل بين كورسات (IDOR/عبر مستأجر) + رفض وحدة محذوفة |

> **تغيير عقد موثَّق (2026-07-21، زيادة الواجهة ٣):** أُضيف `expectedVersion` إلى `moveLesson`.
> **السبب:** النقل كان الطفرة الوحيدة بلا تفاوض نسخة، فكان ممكنًا أن يُنقل عنصر عُدّل أو نُقل من نافذة أخرى منذ أن بدأ السحب. الآن يُعاد قراءة الصفّ تحت قفل: نسخة مختلفة → `409 version_conflict`، والصفّ محذوف → `410 resource_gone`، والوحدة المقصودة محذوفة → `validation_error` برسالة صريحة. مُغطّى باختبارات في `CourseBuilderDragDropTest`.

- **حارس الانتماء (IDOR):** كل عملية تتحقّق أن `lesson/module.program_id == program.getKey()` قبل أي كتابة — مدرّب مُسنَد لكورس لا يصل درس كورس آخر بربطه بكورسٍ يملكه (يُردّ `permission_denied`).
- **بِتّة النشر:** المدرّب المُسنَد يؤلّف لكن لا يغيّر حالة النشر — تغيير `is_published` وحده يتطلّب `assertCanPublish` (مشرف فقط)؛ الحفظ دون تغييرها مسموح.

## سياسة إعادة المحاولة (الواجهة)
- **شبكة / عملية-قيد-التنفيذ:** تلقائيّ (backoff). · **CSRF:** جدّد ثم أعِد مرة. · **تعارض/تحقّق/صلاحية/جلسة/تضارب-مفتاح:** لا إعادة — يقرّر المستخدم.
