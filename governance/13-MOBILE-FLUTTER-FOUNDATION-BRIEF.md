# منصة ومضات التعليمية — Flutter Mobile Foundation Brief
## WAM-0001 — FLUTTER MOBILE MVP SCOPE & DELIVERY FOUNDATION

**الحالة:** `FRAMEWORK DECIDED / MVP SCOPING REQUIRED`  
**الأولوية:** Early post-V1 priority candidate  
**القرار التجاري:** التطبيق مهم ضمن خطة التطوير الأولية  
**القرار التقني:** `FLUTTER — CLEAN IMPLEMENTATION`  
**Legacy Decision:** `REACT NATIVE / EXPO RETIRED — DO NOT REUSE`

---

# 1. القرار النهائي

اعتمدت ومضات وSmart Agency المسار التالي:

- تطبيق واحد جديد بـFlutter لـAndroid وiOS.
- يبدأ من مشروع نظيف ومعمارية حديثة مناسبة للعقد الحالي للـBackend.
- الكود القديم React Native/Expo لا يُرحّل ولا يُحدّث ولا يستخدم كأساس للتطبيق الجديد.
- يمكن الرجوع للكود القديم فقط كمرجع سلوكي غير ملزم عند الحاجة لفهم رحلة سابقة؛ لا تنقل منه Architecture أو dependencies أو implementation افتراضيًا.
- لا يوجد Technology Discovery أو Framework comparison بعد هذا القرار.

---

# 2. ما الذي يجب حسمه قبل Coding؟

## Product Scope

- Primary personas: Student فقط أم Student + Instructor.
- أهم 3–5 MVP journeys.
- Push requirement.
- Offline/download requirement.
- Checkout/payment strategy داخل التطبيق.
- Certificates/support requirements.
- Arabic/English scope.
- Store launch target.
- Minimum Android/iOS versions.

## API Readiness

- Auth/session/refresh contract.
- Program/catalog/details APIs.
- Enrollment/My Programs.
- Learning/content/media APIs.
- Quiz/assignment/progress.
- Orders/payments/bank transfer.
- Certificates/verification.
- Notifications/push registration.
- Support.
- Program Interest / Waitlist where the mobile MVP exposes it.
- Feature-scope compatibility so Mobile does not reveal deferred capabilities.

---

# 3. Flutter Foundation Requirements

الحد الأدنى المعماري المطلوب قبل بناء Features:

- واضح `app / domain / data / presentation` separation أو بديل موثق بنفس الصرامة.
- Typed API client/contracts حيث يمكن.
- Secure token storage.
- Refresh/session lifecycle.
- Central error mapping بالعربية/الإنجليزية حسب الـscope.
- Deep links.
- Push registration abstraction.
- Sentry/crash reporting.
- Analytics/event layer متوافقة مع Product measurement.
- RTL + accessibility baseline.
- Media/video/PDF strategy.
- Environment separation (dev/staging/prod).
- CI build/signing/release ownership.
- App Store / Play Store operational checklist.

---

# 4. MVP Candidate — ليس Commitment نهائيًا

المرشح الأولي الذي يدخل Triage/DoR:

1. Authentication / account.
2. Home / Today.
3. Program catalog/details.
4. My Programs.
5. Learning player (video/PDF/text/resources).
6. Progress.
7. Quiz.
8. Assignment.
9. Certificates.
10. Notifications.
11. Support.

عناصر مثل native checkout، offline downloads، biometrics، advanced push، instructor surfaces وlive sessions تحدد حسب الـOutcome والجهد والاعتماديات، ولا تضاف تلقائيًا.

---

# 5. Non-Goals

المسار الجديد لا يعني:

- إعادة استخدام React Native/Expo القديم.
- إعادة بناء Backend لمجرد وجود Mobile.
- فتح Deferred Features على API/Web.
- نقل كل شاشات أو أفكار التطبيق القديم إلى Flutter.
- إدخال Wallet/Community/Consultations/Gamification وغيرها بدون Scope مستقل.

---

# 6. Required Output قبل أن تصبح WAM-0001 READY

1. `Flutter Mobile MVP Scope`.
2. Persona/Journey map.
3. API Gap List.
4. Flutter architecture foundation decision/ADR.
5. Push/Deep-link/Payment strategy.
6. Store & signing ownership checklist.
7. Rough effort range.
8. Risk/dependency list.
9. Acceptance Criteria للـMVP.
10. Definition of Ready.

---

# 7. Release Positioning

Mobile Track يمكن أن يسير بإصدار مستقل عن Web V1.1، لكنه يخضع لنفس الحوكمة:

```text
Approved Outcome
→ MVP Scope
→ API Gap Assessment
→ Flutter Foundation
→ Definition of Ready
→ Delivery
→ QA/UAT
→ Store Release Gate
→ Production Monitoring
```

---

# 8. القرار الحاكم

> **التقنية محسومة؛ ما نحتاج اكتشافه الآن هو المنتج الصحيح الذي نبنيه بـFlutter، وليس أي Framework نستخدمه.**
