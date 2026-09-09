# منصة ومضات التعليمية — سياسة الـBacklog وترتيب الأولويات
## PRODUCT BACKLOG & PRIORITIZATION POLICY

**الإصدار:** 1.0  
**الحالة:** `LIVING POLICY + REGISTER CONTRACT`  
**Review:** Weekly Triage  

---

# 1. الهدف

إنشاء **باب دخول واحد** لكل طلب أو مشكلة أو فكرة، بحيث لا تتحول الرسائل أو الملاحظات الشفهية إلى تنفيذ مباشر غير قابل للتتبع.

---

# 2. أنواع عناصر الـBacklog

| Type | التعريف | مثال |
|---|---|---|
| `INCIDENT` | حدث Production يهدد الخدمة/الأمن/البيانات/الإيراد | الدفع متوقف |
| `BUG` | وظيفة ضمن Scope لا تعمل وفق السلوك المعتمد | Enrollment لا ينشأ بعد حالة صحيحة |
| `UX` | الوظيفة تعمل لكن التجربة تحتاج تحسينًا | سبب رفض الدفع غير واضح |
| `ENHANCEMENT` | توسيع وظيفة قائمة | فلاتر إضافية للإدارة |
| `NEW_FEATURE` | Capability جديدة | Direct Messaging |
| `BUSINESS_RULE` | تغيير سياسة/قاعدة | من يحق له التقييم |
| `INTEGRATION` | مزود/نظام خارجي | SMS provider |
| `TECH_DEBT` | Refactor/upgrade/performance/observability | إعادة هيكلة query أو dependency upgrade |
| `SECURITY` | تحسين أو ثغرة/مخاطر وصول | session hardening |
| `DISCOVERY` | بحث/Prototype قبل قرار بناء | دراسة Live provider |

---

# 3. Backlog Item Contract

كل عنصر غير P0 يجب أن يحتوي الحد الأدنى:

```yaml
id: WAM-XXXX
title:
type:
requester/source:
created_at:
problem_statement:
affected_users_or_operations:
current_behavior:
desired_outcome:
evidence:
severity_or_priority_candidate:
business_value:
dependencies:
risks:
rough_effort:
owner:
status:
target_release:
links:
```

للـBug يضاف:

```yaml
expected_behavior:
actual_behavior:
reproduction:
environment:
```

ولـFeature يضاف:

```yaml
success_metric:
out_of_scope:
acceptance_criteria_link:
```

---

# 4. حالات العنصر

```text
NEW
→ TRIAGE
→ NEEDS_EVIDENCE / NEEDS_DECISION / READY_FOR_SCORING
→ BACKLOG
→ NEXT_CANDIDATE
→ READY
→ IN_PROGRESS
→ IN_REVIEW
→ UAT
→ RELEASE_READY
→ RELEASED
→ MEASURED
→ CLOSED
```

حالات جانبية:

- `BLOCKED`
- `DUPLICATE`
- `REJECTED`
- `DEFERRED`
- `CANCELLED`

لا يستخدم `DONE` كمرادف لـ`RELEASED`.

---

# 5. Severity / Priority Classes

نحتفظ بالنظام المعتمد سابقًا:

## P0 — Critical

يشمل مثلًا:

- توقف المنصة أو Journey أساسية.
- مشكلة أمن/تسريب بيانات.
- فقد/فساد بيانات.
- خلل مالي جوهري/دفع/ازدواجية.
- Enrollment/Access/Certificate critical integrity.

**المسار:** Incident lane فوري، لا ينتظر RICE.

## P1 — High

أثر مباشر وعالي على:

- Revenue/Conversion.
- Student/Instructor core experience.
- Operations.
- Completion.
- Support load.
- Reliability risk غير P0.

## P2 — Medium

قيمة جيدة لكن المنتج يعمل دونها.

## P3 — Strategic

استثمار/توسع كبير يحتاج Business Case وCapacity مستقلين.

> **Severity ≠ Roadmap Rank دائمًا.** Bug P1 قد يسبق New Feature P1؛ Security/Financial risk يملك Override.

---

# 6. Triage الأسبوعي

مدة مستهدفة: 30–45 دقيقة.

لكل عنصر جديد يتم قرار واحد فقط من:

1. Incident route.
2. Merge with duplicate.
3. Request evidence.
4. Clarify scope.
5. Reject with reason.
6. Move to backlog.
7. Escalate security/finance/architecture review.

لا نقوم في Triage بتصميم Feature كاملة.

---

# 7. Prioritization — RICE-lite

يستخدم للـEnhancement/New Feature/Discovery عند توفر بيانات كافية.

## Reach

عدد المستخدمين/الطلبات/الحالات/العمليات المتوقع تأثرها خلال نافذة قياس متفق عليها.

## Impact

| الدرجة | المعنى |
|---:|---|
| 3 | أثر جوهري على Outcome رئيسي |
| 2 | أثر كبير |
| 1 | أثر متوسط |
| 0.5 | أثر محدود |
| 0.25 | أثر هامشي |

## Confidence

| القيمة | الوصف |
|---:|---|
| 1.0 | بيانات Production/تكرار مؤكد/طلب تجاري معتمد |
| 0.8 | Evidence جيد لكنه غير كامل |
| 0.5 | فرضية مع إشارات محدودة |

## Effort

تقدير Delivery/Engineering بوحدة Person-Weeks أو وحدة نسبية ثابتة للفريق.

## المعادلة

```text
RICE-lite = (Reach × Impact × Confidence) / Effort
```

لا تستخدم النتيجة كقرار آلي؛ هي أداة مقارنة.

---

# 8. Overrides قبل نتيجة RICE

يمكن تقديم عنصر أقل Score إذا:

- Security/Privacy risk.
- Financial/Data integrity risk.
- Regulatory/contractual deadline.
- Production reliability risk.
- Dependency unlocks several higher-value initiatives.
- Critical external provider deprecation.

ويجب تسجيل سبب الـOverride.

---

# 9. Evidence المقبول

من الأقوى إلى الأضعف تقريبًا:

1. Production incidents/metrics.
2. Revenue/payment/checkout data.
3. Support tickets المتكررة.
4. User behavior analytics.
5. UAT/operations observation.
6. Multiple customer/user interviews.
7. Strategic contractual requirement.
8. Single request/opinion.

لا يمنع المستوى الأخير التنفيذ، لكنه يخفض Confidence ما لم توجد أسباب أخرى.

---

# 10. Backlog Hygiene

مرة شهريًا:

- حذف/دمج duplicates.
- إغلاق ما فقد قيمته.
- إعادة Score للعناصر التي تغيرت بياناتها.
- عدم إبقاء `NEXT CANDIDATE` بلا Owner.
- مراجعة العناصر >90 يومًا دون Evidence.
- التأكد أن Deferred Registry وBacklog لا يتناقضان.

---

# 11. WIP Limits

لفريق صغير:

- لا أكثر من 1–3 مبادرات Product كبيرة `IN_PROGRESS` في نفس الوقت.
- P0 قد يقطع WIP.
- Tech Debt صغير يمكن bundling داخل cycle بشرط عدم إخفائه.

الهدف تقليل work started/not finished.

---

# 12. نموذج قرار الأولوية

```text
ID:
Type:
Priority Class: P0/P1/P2/P3
RICE-lite:
Risk Override: Yes/No
Strategic Fit:
Decision: NOW / NEXT / LATER / REJECT
Decision Reason:
Owner:
Next Review Date:
```

---

# 13. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | اعتماد Backlog موحد + P0–P3 + RICE-lite + Weekly Triage. |
