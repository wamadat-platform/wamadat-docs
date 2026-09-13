# منصة ومضات التعليمية — Authoritative Documentation Baseline

**تاريخ آخر مراجعة:** 13 سبتمبر 2026  
**Baseline Product Release:** `WAMADAT-V1.0.0`  
**حالة المنتج:** `V1 CLOSED BASELINE / PRODUCTION OPERATIONS ACTIVE / V1.x DEVELOPMENT GOVERNED`  
**حالة المستودع:** `FINAL APPROVED AUTHORITATIVE DOCUMENTATION BASELINE`  

---

# 1. نقطة البدء الرسمية

منذ 9 سبتمبر 2026، يبدأ أي عمل Product/Development من:

**[governance/00-PRODUCT-GOVERNANCE-README.md](./governance/00-PRODUCT-GOVERNANCE-README.md)**

هذا المجلد يحدد إغلاق V1، Product Baseline، الـRoadmap الحية، Backlog، Change Control، Feature Delivery، Release Management، KPIs، المخاطر وسجل القرارات.

> عند التعارض حول ترتيب التطوير بعد V1، تكون وثائق `governance/` أحدث من الترتيب الزمني التاريخي داخل Roadmap الإطلاق القديمة.

---

# 2. Product Governance — المرجع الحي بعد V1

1. [00-PRODUCT-GOVERNANCE-README.md](./governance/00-PRODUCT-GOVERNANCE-README.md) — الفهرس، هرم مصادر الحقيقة، الأدوار والقواعد.
2. [01-V1-RELEASE-CLOSURE-AND-ACCEPTANCE.md](./governance/01-V1-RELEASE-CLOSURE-AND-ACCEPTANCE.md) — إغلاق وقبول V1.
3. [02-CURRENT-PRODUCT-BASELINE.md](./governance/02-CURRENT-PRODUCT-BASELINE.md) — Product Capability Baseline للإصدار المغلق.
4. [03-PRODUCT-ROADMAP.md](./governance/03-PRODUCT-ROADMAP.md) — Now / Next / Later الحية.
5. [04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md](./governance/04-PRODUCT-BACKLOG-AND-PRIORITIZATION-POLICY.md) — Intake/Triage/P0-P3/RICE-lite.
6. [05-CHANGE-REQUEST-PROCESS.md](./governance/05-CHANGE-REQUEST-PROCESS.md) — Change Control.
7. [06-FEATURE-DELIVERY-LIFECYCLE.md](./governance/06-FEATURE-DELIVERY-LIFECYCLE.md) — Ready/Done/UAT lifecycle.
8. [07-RELEASE-MANAGEMENT.md](./governance/07-RELEASE-MANAGEMENT.md) — Versioning/manifest/go-no-go/rollback.
9. [08-PRODUCT-KPI-AND-PLATFORM-HEALTH.md](./governance/08-PRODUCT-KPI-AND-PLATFORM-HEALTH.md) — KPI/Health/Error-budget-lite.
10. [09-TECHNICAL-DEBT-AND-RISK-REGISTER.md](./governance/09-TECHNICAL-DEBT-AND-RISK-REGISTER.md) — Risk/Debt register.
11. [10-DECISION-LOG.md](./governance/10-DECISION-LOG.md) — Product/Technical decision log.

---

# 3. Launch V1 — Frozen / Historical Acceptance Baseline

1. [qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md](./qa-final/01-Wamadat-Launch-V1-Features-and-Operations.md) — `FROZEN`: النطاق الوظيفي المسلم في V1.
2. [qa-final/02-Wamadat-Product-Development-Roadmap.md](./qa-final/02-Wamadat-Product-Development-Roadmap.md) — `ARCHIVED CANDIDATE ROADMAP`: الرؤية السابقة ومخزون المرشحين؛ ليست ترتيب التنفيذ الحي بعد الإغلاق.
3. [qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md](./qa-final/03-Wamadat-Launch-V1-Manual-UAT-Checklist.md) — `FROZEN EVIDENCE / REUSABLE REGRESSION SOURCE`.
4. [qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md](./qa-final/04-Wamadat-Launch-V1-Deferred-Features-Registry.md) — `LIVING REGISTER`: المرجع الرسمي لما هو Hidden/Blocked/Disabled/Dormant.
5. [qa-final/05-Wamadat-Launch-V1-UAT-Links-and-Test-Payment-Data.md](./qa-final/05-Wamadat-Launch-V1-UAT-Links-and-Test-Payment-Data.md) — `ARCHIVED TEST REFERENCE`: لا يستخدم لتقرير حالة Production الحالية.

---

# 4. الوثائق التقنية والتشغيلية — Living Technical Baseline

1. [technical/01-Current-Technical-Architecture.md](./technical/01-Current-Technical-Architecture.md) — المعمارية الحالية.
2. [technical/02-Security-and-Access-Control.md](./technical/02-Security-and-Access-Control.md) — الأمان والتحكم بالوصول.
3. [technical/03-Payments-and-External-Integrations.md](./technical/03-Payments-and-External-Integrations.md) — عقود الدفع والتكاملات.
4. [technical/04-Production-Deployment-and-Operations.md](./technical/04-Production-Deployment-and-Operations.md) — النشر والتشغيل.
5. [technical/05-Backup-Restore-and-Rollback.md](./technical/05-Backup-Restore-and-Rollback.md) — النسخ والاستعادة والتراجع.
6. [technical/COURSE_BUILDER_CONTRACT.md](./technical/COURSE_BUILDER_CONTRACT.md) — عقد Course Builder.

> القيم المتغيرة بيئيًا مثل تفعيل مزود دفع معين أو Topology فعلية يجب التحقق منها من Production configuration/runbook الحالي، لا من بيانات UAT التاريخية.

---

# 5. Brand & Engineering

- [engineering/coding-standards.md](./engineering/coding-standards.md) — معايير الكود.
- [brand-system/README.md](./brand-system/README.md) — نظام الهوية والتصميم.

---

# 6. Incident History

- [incidents/](./incidents/) — سجلات Postmortem تاريخية. لا تعد Incident مفتوحة ما لم يوجد سجل حالي يقول ذلك.

---

# 7. Documentation Status Policy

- `FROZEN`: لا يعاد كتابة التاريخ؛ التغيير في Release لاحق.
- `LIVING`: يحدث مع Changelog.
- `ARCHIVED`: مرجع تاريخي لا يقود القرارات الحالية.
- `REGISTER`: سجل حي.

Git History يحتفظ بمراحل التدقيق والإغلاق السابقة التي أزيلت من HEAD لتقليل التعارض.

---

### Product Cycle 01 — Post-V1

- [`governance/11-INITIAL-PRODUCT-BACKLOG-2026-09-13.md`](governance/11-INITIAL-PRODUCT-BACKLOG-2026-09-13.md) — أول Backlog موحد بعد V1 (Pre-Triage).
- [`governance/12-TRIAGE-01-PREPARATION-2026-09-13.md`](governance/12-TRIAGE-01-PREPARATION-2026-09-13.md) — حزمة أول جلسة Triage مع العميل.
- [`governance/13-MOBILE-FLUTTER-FOUNDATION-BRIEF.md`](governance/13-MOBILE-FLUTTER-FOUNDATION-BRIEF.md) — قرار Flutter المعتمد، حدود الـMVP، ومتطلبات تأسيس التطبيق من الصفر.

