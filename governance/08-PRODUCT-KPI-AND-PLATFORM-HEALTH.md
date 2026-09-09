# منصة ومضات التعليمية — مؤشرات المنتج وصحة المنصة
## PRODUCT KPI & PLATFORM HEALTH

**الإصدار:** 1.0  
**الحالة:** `LIVING MEASUREMENT STANDARD`  
**Cadence:** Weekly health + Monthly product review  

---

# 1. الهدف

الانتقال من:

> «نعتقد أن Feature مهمة»

إلى:

> «لدينا Outcome ومؤشر Baseline ونقيس هل التغيير حسّنه أم لا».

هذه الوثيقة **لا تضع أهدافًا رقمية مخترعة**. أول 2–4 أسابيع من القياس تستخدم لتثبيت Baseline موثوق، ثم يعتمد Product Owner أهداف الدورة.

---

# 2. KPI Layers

نفرق بين:

1. **Business/Product KPIs** — هل المنتج يحقق قيمة؟
2. **Journey KPIs** — أين يسقط المستخدم؟
3. **Operational KPIs** — هل الفريق يستطيع تشغيل المنتج؟
4. **Reliability/Platform Health** — هل النظام مستقر؟

لا نعوض ضعف reliability بارتفاع visits.

---

# 3. Acquisition

مؤشرات سبق تعريفها في Roadmap الأساسية:

- Visitors.
- Program Views.
- Registration Conversion.

إضافيًا، عند توفر analytics الموثوقة:

- Landing → Program Detail rate.
- Source/UTM conversion segmentation.

---

# 4. Commerce

- Cart → Checkout conversion.
- Checkout → Payment success.
- Payment failure rate.
- Payment failure reasons by gateway where available safely.
- Coupon usage.
- Bank Transfer approval rate.
- Time from order creation to successful enrollment.

يجب تجنب احتساب test/internal traffic ضمن KPIs التجارية حيث أمكن.

---

# 5. Learning

- Enrollment activation.
- Learners starting first lesson.
- Lesson engagement.
- Quiz participation.
- Quiz pass rate.
- Assignment submission rate.
- Program completion rate.
- Time to completion حسب نوع البرنامج عند فائدته.

لا يجب تفسير completion المنخفض وحده كعيب Product دون مراعاة مدة البرنامج ونوعه.

---

# 6. Certification

- Certificates issued.
- Certificate PDF downloads.
- Public verification usage.
- Certificate issuance failures.

---

# 7. Support & Operations

- Tickets created.
- Average first response time.
- Resolution time.
- Reopen rate إن أمكن.
- Top recurring issue categories.
- Manual interventions in payment/enrollment/certificate flows.

الهدف ليس فقط خفض التذاكر؛ بعض الانخفاض قد يعني صعوبة الوصول للدعم. ننظر إلى السبب.

---

# 8. Platform Health

المؤشرات الأساسية المذكورة في Roadmap:

- Error rate.
- Failed jobs.
- Queue delay.
- Webhook failures.
- Email delivery failures.

ويضاف حسب التشغيل:

- HTTP 5xx rate.
- Critical endpoint latency where measured.
- Scheduler heartbeat/health.
- Database/resource saturation signals.
- Backup freshness/monitor result.
- External provider error spikes.

---

# 9. Core Journey Health Board

اللوحة التنفيذية الشهرية يجب أن توضح على الأقل:

```text
Registration
→ Checkout
→ Payment
→ Order
→ Enrollment
→ Learning Start
→ Assessment
→ Completion
→ Certificate
```

لكل خطوة:

- volume.
- success rate.
- failure reason if known.
- trend versus previous period.

---

# 10. Reliability Guardrail / Error Budget Lite

لا نحتاج SRE ثقيلًا لفريق صغير. نطبق قاعدة عملية:

```text
إذا كانت مؤشرات reliability ضمن المستوى المتفق عليه
→ يستمر Feature delivery

إذا ظهرت P0 أو تكررت P1/critical failures أو تدهورت core journey
→ تخفض/توقف non-critical feature releases مؤقتًا
→ Stability work
→ Health restored
→ Resume roadmap
```

يحدد الفريق thresholds الرقمية بعد Baseline؛ لا تترك القاعدة مبهمة للأبد.

---

# 11. Metric Definition Contract

كل KPI يعتمد يجب أن يوثق:

```text
Metric Name:
Business Meaning:
Numerator:
Denominator:
Source:
Filters/Exclusions:
Timezone:
Refresh Frequency:
Owner:
Known Limitations:
```

هذا يمنع اختلاف الأرقام بين Analytics/Admin/DB.

---

# 12. Feature Success Measurement

قبل Release لـFeature Product:

```text
Primary Outcome:
Primary Metric:
Baseline:
Expected Direction:
Observation Window:
Guardrail Metrics:
Decision after window: Keep / Iterate / Disable
```

لا يشترط A/B testing لكل Feature، لكن يجب معرفة ما الذي سنراقبه.

---

# 13. Dashboard Tiers

## Executive/Product — شهري

- 8–12 مؤشرًا فقط.
- Trends + decisions.
- لا يغرق في logs.

## Operations — يومي/أسبوعي

- payments/webhooks/jobs/email/errors/support.

## Engineering — on demand/realtime

- traces/logs/latency/resources/database/queues.

---

# 14. Monthly Product Review Template

```text
Period:

1. Business/Product Outcomes
2. Core Journey Funnel
3. Learning Outcomes
4. Support Themes
5. Platform Health
6. Incidents / Risks
7. What shipped
8. Did shipped items move target metrics?
9. Roadmap decisions
10. Actions / Owners
```

---

# 15. Data Quality Rules

- استبعاد test data حيث أمكن.
- توثيق أي تغيير في event/schema يقطع المقارنة التاريخية.
- عدم جمع PII غير ضرورية للقياس.
- عدم تحويل metric غير موثوقة إلى قرار مالي كبير.
- مراجعة timezone والعملات/بوابات الدفع عند تفسير التجارة.

---

# 16. أول Baseline مطلوب بعد اعتماد الحزمة

خلال أول نافذة قياس موحدة، يثبت الفريق على الأقل:

- Payment success/failure.
- Enrollment success after eligible purchase.
- Completion trend.
- Ticket themes.
- Failed jobs/queue health.
- Webhook failures.
- Email delivery failures.
- Top production errors.

بعدها فقط تحدد Targets للدورة التالية.

---

# 17. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | تحويل KPI list القديمة إلى Measurement/Health operating standard. |
