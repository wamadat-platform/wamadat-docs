# منصة ومضات التعليمية — أول Product Backlog بعد V1
## INITIAL PRODUCT BACKLOG — CYCLE 01

**Baseline المنتج:** `WAMADAT-V1.0.0`  
**Baseline Date:** `2026-09-09`  
**Backlog Snapshot:** `2026-09-13`  
**الحالة:** `PRE-TRIAGE / NOT COMMITTED`  
**المرجع الحاكم:** `04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md`

---

# 1. الهدف

هذه الوثيقة هي **أول Backlog موحد** بعد إغلاق V1. لا تعني أن العناصر الواردة فيها معتمدة للتنفيذ، ولا أن ترتيبها الحالي هو Release Order.

تم جمع العناصر من:

1. قرارات Product Governance المعتمدة بعد V1.
2. `03-PRODUCT-ROADMAP.md`.
3. `04-Wamadat-Launch-V1-Deferred-Features-Registry.md`.
4. `09-TECHNICAL-DEBT-AND-RISK-REGISTER.md`.
5. خارطة التطوير التاريخية المؤرشفة كمصدر Candidate/Effort فقط.
6. فحص السورس الحالي للـBackend/Web، مع تسجيل قرار إيقاف الكود Mobile legacy كأثر تاريخي فقط.

**قاعدة:** لا نعطي RICE Score رقميًا بدون Reach/Confidence حقيقيين. أي عنصر تنقصه البيانات يبقى `NEEDS_EVIDENCE` أو `NEEDS_DECISION`.

---

# 2. نتائج الفحص التي تؤثر على الـBacklog

## 2.1 Mobile App — التقنية محسومة Flutter

القرار النهائي المعتمد:

- التطبيق الجديد يبنى بـ**Flutter** لـAndroid وiOS.
- يبدأ من قاعدة نظيفة (Clean implementation).
- مستودع React Native/Expo القديم **متروك/Retired** ولا يعاد استخدامه في التنفيذ الجديد.
- لا نحتاج Technology Discovery أو مقارنة Frameworks.
- المطلوب قبل Coding هو فقط: MVP scope، personas، API readiness/gaps، release/store ownership، effort/risk، وDefinition of Ready.

## 2.2 Program Interest / Waitlist — حُسم ضمن V1

تم اعتماد Program Interest / Waitlist رسميًا كجزء من `WAMADAT-V1.0.0` لأنه موجود ومسلّم ضمن رحلة المنتج الحالية:

- Public interest form.
- Program linkage.
- Admin lead visibility/export.
- Marketing event `program_interest`.
- ضوابط التكرار/الإدخال وفق التنفيذ الحالي.

لذلك **لا يظهر كـFuture Feature ولا كعنصر Triage**. تم تحديث Frozen Baseline وDeferred Registry وفق ذلك.

## 2.3 Payment recovery موجود بالفعل

السورس/الوثائق الحالية تتضمن `orders:reconcile` و`webhooks:replay-queued` وIdempotency/Webhook safeguards. لذلك لا ننشئ Feature باسم "Payment Reconciliation" من الصفر؛ الحاجة المستقبلية هنا هي **Observability/Provider Health/Alerting** فقط ما لم يظهر Incident جديد.

---

# 3. Backlog Lanes

- `NOW-ENABLER`: عمل ضروري لسلامة القرار/الـRelease القادم، وليس Feature تجارية بحد ذاته.
- `NEXT-CANDIDATE`: مرشح قريب يحتاج Triage/Scoring/Ready.
- `LATER`: Opportunity حقيقية لكن لا يوجد سبب لفتحها الآن.
- `HOLD`: لا تتحرك حتى قرار/دليل محدد.

---

# 4. Initial Backlog Register

| ID | Title | Type | Source | Initial Priority | Lane | Status | Evidence / Reason | Rough Effort | Target |
|---|---|---|---|---|---|---|---|---|---|
| **WAM-0001** | Flutter Mobile MVP Scope & Delivery Foundation | `DISCOVERY` / `PLANNING` | Business + technical decision | P1 candidate | NEXT-CANDIDATE | `NEEDS_PRODUCT_SCOPE` | Flutter محسوم؛ المطلوب MVP/API gaps/effort/release readiness فقط، والكود RN/Expo القديم Retired | 2–4 days planning/audit | Mobile Track |
| **WAM-0002** | Product KPI Baseline & Core Journey Health Board | `ENHANCEMENT` | Governance 08 + RISK-009 | P1 | NOW-ENABLER | `READY_FOR_SCORING` | لا يمكن ترتيب Roadmap باحتراف بدون baseline موحد للـcommerce/learning/support/health | 3–7 days initial baseline | Ops/Product |
| **WAM-0003** | Backup Restore Drill + Backup Monitoring | `TECH_DEBT` | RISK-008 + technical/05 | P1 | NOW-ENABLER | `READY` | Backup دون restore confidence مخاطرة تشغيلية عالية | 1–3 days first drill | Operations |
| **WAM-0004** | External Provider Health & Alerting | `TECH_DEBT` | RISK-003/RISK-004 | P1 | NOW-ENABLER | `NEEDS_SCOPING` | Payment/email/external providers تحتاج visibility؛ reconcile موجود بالفعل | 3–7 days | Operations |
| **WAM-0006** | Single-Academy Legacy Architecture Cleanup Assessment | `DISCOVERY` | DEC-013 + RISK-007 | P2 | HOLD | `NEEDS_EVIDENCE` | المنتج Single-Academy نهائيًا، لكن tenant-aware foundation ما تزال داخلية؛ لا داعي لإعادة بناء بلا Business/Engineering case | 2–4 days assessment | Technical Roadmap |
| **WAM-0010** | Account Security Enhancements | `SECURITY` / `ENHANCEMENT` | V1.1 Candidate | P1 | NEXT-CANDIDATE | `READY_FOR_SCORING` | Active sessions/devices/logout others/security events/2FA candidate | 3–7 days | V1.x |
| **WAM-0011** | Direct Messaging — Student ↔ Instructor | `NEW_FEATURE` | Deferred Registry / V1.1 | P1 | NEXT-CANDIDATE | `NEEDS_EVIDENCE` | أساس موجود في السورس؛ القيمة تعتمد على حاجة التواصل الفعلية؛ permissions/isolation critical | 1–2 weeks | V1.x |
| **WAM-0012** | Advanced Notification Preferences | `ENHANCEMENT` | Deferred Registry / V1.1 | P1/P2 | NEXT-CANDIDATE | `NEEDS_EVIDENCE` | يفصل transactional preferences عن marketing؛ يمكن أن يخدم Mobile/Push لاحقًا | 1–2 weeks | V1.x |
| **WAM-0013** | Program Reviews / Ratings | `NEW_FEATURE` | Deferred Registry / V1.1 | P1/P2 | NEXT-CANDIDATE | `NEEDS_EVIDENCE` | أساس جزئي موجود؛ يحتاج eligibility + moderation policy | 3–5 days + moderation closure | V1.x |
| **WAM-0014** | Student Calendar & Upcoming Activities | `NEW_FEATURE` | Deferred Registry / V1.1 | P2 | NEXT-CANDIDATE | `NEEDS_EVIDENCE` | deadlines/milestones/ICS؛ القيمة تعتمد على دعم البرامج طويلة المدة | 3–7 days | V1.x |
| **WAM-0015** | Gamification — Streak & Achievements | `NEW_FEATURE` | Deferred Registry / V1.1 | P2 | LATER | `NEEDS_EVIDENCE` | لا يسبق completion/reliability issues؛ يحتاج تعريف Activity/Timezone | 1–2 weeks | Later V1.x |
| **WAM-0016** | Program Gifts + Redemption | `NEW_FEATURE` | Deferred Registry / V1.1 | P2 | LATER | `NEEDS_DECISION` | يحتاج refund/expiry/redeem rules قبل Ready | 1–2 weeks | Later V1.x |
| **WAM-0017** | Advanced Student Profile | `ENHANCEMENT` | Deferred Registry / V1.1 | P2 | LATER | `NEEDS_EVIDENCE` | لا قيمة مباشرة مثبتة حاليًا للـbio/skills/portfolio | 3–5 days | Later V1.x |
| **WAM-0018** | Wallet | `NEW_FEATURE` | Deferred Registry | P2/P3 | LATER | `NEEDS_BUSINESS_CASE` | Financial contract واضح مطلوب قبل التنفيذ | TBD | Later |
| **WAM-0019** | Wishlist | `NEW_FEATURE` | Deferred Registry | P2 | LATER | `NEEDS_EVIDENCE` | يجب إثبات قيمة retention/discovery | TBD | Later |
| **WAM-0020** | Advanced Reporting & Analytics | `ENHANCEMENT` | Historical roadmap / P1 | P1 | NEXT-CANDIDATE | `NEEDS_SCOPING` | Admin/instructor/student analytics؛ يجب فصل Product KPI board الداخلي عن customer-facing analytics | 3–5 weeks full scope | V1.x / Phase 2 |
| **WAM-0021** | Basic Live Sessions | `NEW_FEATURE` | Deferred Registry / Phase 2 | P1 | NEXT-CANDIDATE | `NEEDS_BUSINESS_CASE` | قيمة تعليمية محتملة عالية؛ يحتاج provider/operating model وسلوك attendance | 2–4 weeks basic | Phase 2 |
| **WAM-0022** | Push Notifications | `INTEGRATION` / `ENHANCEMENT` | Deferred Registry / Phase 2 | P2 | NEXT-CANDIDATE | `NEEDS_SCOPING` | يرتبط مباشرة بـFlutter MVP؛ يحدد هل يدخل الإصدار الأول أم مرحلة لاحقة | 1–2 weeks baseline | Mobile/Phase 2 |
| **WAM-0023** | Advanced Attendance | `ENHANCEMENT` | Phase 2 | P2 | LATER | `NEEDS_EVIDENCE` | V1 QR/attendance core موجود؛ المتبقي late/excused/bulk/audit/analytics | 2–4 weeks | Phase 2 |
| **WAM-0024** | Community / Forum | `NEW_FEATURE` | Deferred Registry / Phase 2 | P2 | LATER | `NEEDS_OPERATING_MODEL` | لا يطلق بدون moderation/reporting/ownership | 3–5 weeks | Phase 2 |
| **WAM-0025** | Consultations | `NEW_FEATURE` | Deferred Registry / Phase 2 | P2 | LATER | `NEEDS_BUSINESS_RULES` | أساس موجود لكن SLA/pricing/delivery/refund/owner غير محسومة | 2–4 weeks | Phase 2 |
| **WAM-0026** | CRM / Leads | `NEW_FEATURE` | Phase 2 | P2 | LATER | `NEEDS_BUSINESS_CASE` | يجمع contact/interest/consultation/corporate leads؛ لا يبدأ دون process owner | 2–3 weeks | Phase 2 |
| **WAM-0027** | Marketing Automation | `NEW_FEATURE` | Deferred Registry / Phase 2 | P2 | LATER | `NEEDS_BUSINESS_CASE` | campaigns/abandoned cart/re-engagement؛ يجب فصل transactional عن marketing | 2–4 weeks | Phase 2 |
| **WAM-0028** | SMS Integration | `INTEGRATION` | Deferred Registry / Phase 2 | P2 | LATER | `BLOCKED_BY_PROVIDER` | provider/sender ID/cost/templates/regulations مطلوبة | 1–2 weeks after provider | Phase 2 |
| **WAM-0029** | Newsletter Management | `NEW_FEATURE` | Deferred Registry | P2 | LATER | `NEEDS_BUSINESS_CASE` | يجب أن يرتبط باستراتيجية marketing consent/campaigns | TBD | Phase 2 |
| **WAM-0030** | AI Learning Assistant | `NEW_FEATURE` / `DISCOVERY` | Strategic pool | P3 | LATER | `NEEDS_USE_CASE` | لا يبدأ كـAI عام؛ يحتاج use case/cost/privacy/human review | 3–6 weeks first useful feature | Strategic |
| **WAM-0031** | Wamadat Plus / Marketplace | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_BUSINESS_CASE` | Product مستقل تقريبًا: sellers/commission/payout/disputes | 8–12+ weeks | Strategic |
| **WAM-0032** | Subscriptions | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_BUSINESS_MODEL` | recurring billing/entitlements/refunds lifecycle | TBD | Strategic |
| **WAM-0033** | Bundles | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_BUSINESS_CASE` | pricing/entitlements/refunds/discount interaction | TBD | Strategic |
| **WAM-0034** | Affiliate | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_BUSINESS_MODEL` | attribution/commission/payout/fraud rules | TBD | Strategic |
| **WAM-0035** | Alumni | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_PROBLEM_STATEMENT` | لا توجد Outcome محددة بعد | TBD | Strategic |
| **WAM-0036** | B2B / Corporate Training | `NEW_FEATURE` | Strategic pool | P3 | LATER | `NEEDS_BUSINESS_CASE` | يحتاج corporate buyer/admin/contracts/reporting model | TBD | Strategic |

**مستبعد عمدًا:** Multi-Academy / SaaS ليس Candidate Product وفق قرار Single-Academy المعتمد.

---

# 5. ماذا لا يدخل الـBacklog كـFeature جديدة؟

القدرات التالية جزء من V1 Baseline ولا تعاد تسميتها كـRoadmap Features إلا إذا ظهر Bug/Enhancement محدد:

- Tap / Tabby / Tamara integration contracts حسب التفعيل التشغيلي.
- Bank Transfer.
- Orders / Payments / Invoices.
- Enrollment / Learning / Quiz / Assignment / Certificates / Attendance V1.
- GTM / GA4 / Meta Pixel / TikTok Pixel / Snapchat Pixel.
- Payment webhook idempotency / reconciliation foundation.
- Core support / notifications.

---

# 6. First Triage Shortlist

لا ينصح بمناقشة 30 عنصرًا مع العميل في الجلسة الأولى. نبدأ فقط بهذه المجموعة:

| ID | لماذا يدخل أول Triage؟ | القرار المطلوب |
|---|---|---|
| WAM-0001 | Mobile + Flutter قراران معتمدان | Outcome + personas + MVP + API/release readiness |
| WAM-0002 | يمنحنا Evidence لبقية القرارات | Baseline metrics + dashboard owner |
| WAM-0010 | Security مرشح P1 | هل هناك Pain/Risk يبرر إدخاله مباشرة؟ |
| WAM-0011 | Communication P1 candidate | هل التواصل خارج Lesson Q&A مشكلة فعلية؟ |
| WAM-0012 | Notifications يخدم engagement وMobile | ما الأنواع المزعجة/المهمة؟ |
| WAM-0020 | Analytics/Reporting مرشح P1 | من يحتاج أي قرار لا يستطيع أخذه اليوم؟ |
| WAM-0021 | Live Sessions مرشح تعليمي كبير | هل هو requirement تجاري فعلي أم nice-to-have؟ |
| WAM-0013 | Reviews قد تؤثر على trust/conversion | هل هناك طلب/أثر واضح؟ |

الأعمال WAM-0003/0004 تناقش داخليًا مع Smart Agency كـReliability/Governance enablers ولا تحتاج مفاضلة تجارية مع العميل مثل Feature. WAM-0005 أُغلق توثيقيًا بعد اعتماد Program Interest ضمن V1.

---

# 7. عناصر يجب حسمها داخليًا قبل Scoring

## WAM-0001 Flutter Mobile

لا Score نهائي قبل:

- تحديد Objective: retention؟ push؟ سرعة الوصول؟ Store presence؟ offline؟
- تحديد Personas: Student فقط أم Instructor أيضًا؟
- تحديد MVP journeys.
- API readiness/gap assessment مع Backend الحالي.
- تحديد Push/Deep Links/Payments/Media requirements للـMVP.
- App Store / Play Store operational ownership.
- Rough effort + risks + Definition of Ready.

**غير مطلوب:** مقارنة Flutter مع React Native/Expo؛ القرار محسوم Flutter والكود القديم Retired.

## WAM-0005 Program Interest — CLOSED BASELINE CORRECTION

تم إغلاق هذا العنصر دون Development Release: Program Interest / Waitlist مؤكدة كـV1 Delivered Capability، وتمت مواءمة وثائق Baseline وDeferred Registry. لا تدخل Scoring أو Triage.

---

# 8. RICE — الحالة الحالية

لا توجد الآن بيانات كافية لإعطاء Scores رقمية موثوقة لمعظم Features. لذلك لا نستخدم أرقامًا تخمينية.

بعد تجميع أول KPI baseline وTriage business evidence:

```text
RICE-lite = (Reach × Impact × Confidence) / Effort
```

ويتم Scoring فقط للعناصر التي أصبحت `READY_FOR_SCORING`.

---

# 9. Definition of Ready للـV1.1 Candidate

لا يدخل أي Candidate إلى Release Scope حتى يحتوي:

- Problem Statement.
- Business/User Outcome.
- Evidence.
- Affected personas.
- In Scope / Out of Scope.
- Acceptance Criteria.
- Dependencies.
- Risks.
- Rough Effort.
- Success Metric.
- Owner.
- Target Release.

---

# 10. النتيجة المطلوبة من Cycle 01

بنهاية أول Triage نريد فقط:

1. تثبيت 3 Outcomes للـ90 يوم القادمة.
2. حسم WAM-0001 Flutter Mobile MVP scope (وليس التقنية).
3. اختيار 3–5 Product Candidates فقط للـScoring التفصيلي.
4. اختيار 1–3 Initiatives فقط كـ`NEXT COMMITTED`.
5. بعد ذلك فقط تعريف Scope لأول Release بعد V1.

---

# 11. Change Log

| Date | Version | Change |
|---|---|---|
| 2026-09-13 | 1.0 | إنشاء أول Backlog موحد بعد V1 من Governance/Deferred/Risk/source audit. |
| 2026-09-13 | 1.1 | حسم Flutter كتقنية التطبيق الجديد وإغلاق WAM-0005 بعد اعتماد Program Interest / Waitlist ضمن V1. |
