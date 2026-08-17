# Wamadat Platform — Launch V1 Out of Scope

## 1. الغرض من الوثيقة

تحدد هذه الوثيقة بصورة صريحة كل ما **لا يدخل ضمن Launch V1** لمنصة ومضات.

وجود كود أو Database Tables أو Routes أو واجهات حالية للميزة لا يعني أنها جزء من الإصدار.

قاعدة العمل:

> Out of Scope لا يعني Delete.

في أغلب الحالات:

```text
Keep Code
Keep Data
Keep Tables
Hide / Disable Exposure
Do Not Expand
Do Not Refactor
```

---

# 2. SaaS التجاري

خارج Launch V1 بالكامل:

- بيع ومضات كـSaaS.
- إنشاء أكاديمية جديدة.
- Tenant onboarding.
- Tenant registration.
- SaaS pricing.
- Academy plans.
- Academy subscription billing.
- Multi-academy management.
- Tenant switching.
- Public tenant signup.
- Custom academy onboarding.
- SaaS sales funnels.

---

# 3. إزالة Multi-Tenancy من الباك إند

خارج Launch V1:

- إزالة Spatie Multitenancy.
- دمج landlord وtenant schemas.
- تحويل كل Models إلى single database architecture.
- حذف tenants.
- إعادة كتابة authentication على أساس single tenant.
- إعادة كتابة queues أو storage أو billing بسبب إزالة tenancy.
- إعادة تصميم database بالكامل.

البنية الحالية تبقى داخليًا.

---

# 4. Super Admin التجاري

الـSuper Admin الموجود لإدارة:

- Tenants.
- Plans.
- System users.
- SaaS operations.

ليس جزءًا من الاستخدام اليومي لعميل ومضات في Launch V1.

لا يتم حذفه تقنيًا، لكن لا يتم تقديمه كجزء من المنتج التجاري الحالي.

---

# 5. Wamadat Plus / Marketplace

ومضات بلس خارج Launch V1 بالكامل.

يشمل ذلك:

- Marketplace homepage.
- Services.
- Sellers.
- Seller onboarding.
- Seller KYC.
- Seller dashboard.
- Plus orders.
- Plus messages.
- Packages.
- Reviews الخاصة بالسوق.
- Payouts.
- Seller earnings.
- Plus VAT logic.
- Disputes.
- SLA.
- Service moderation.
- Marketplace payment flows.

يجب إخفاؤه من:

- Navbar.
- Footer.
- Student Dashboard.
- Admin navigation.
- Public navigation.

ولا يتم حذف كوده أو بياناته.

---

# 6. Mobile Application

جميع تطبيقات الهاتف خارج Launch V1.

يشمل ذلك التطبيق الحالي الموجود في المشروع.

لا يدخل:

- إصلاح شامل للتطبيق الحالي.
- Feature parity.
- Android production launch.
- iOS production launch.
- Google Play publishing.
- App Store publishing.

المرحلة التالية المخطط لها:

> تحليل المتطلبات وإعادة بناء التطبيق باستخدام Flutter.

---

# 7. Flutter Rebuild

بناء تطبيق Flutter الجديد بالكامل خارج Launch V1.

يشمل ذلك:

- Architecture.
- UI.
- API integration.
- Notifications.
- Store deployment.
- App review process.

يتم فتح مشروع مستقل له بعد استقرار Web V1.

---

# 8. Python AI Sidecar

خدمة:

```text
ai-service
```

خارج Launch V1.

السورس الحالي يمثل Scaffold أساسيًا، لذلك لا يدخل:

- نشر الخدمة.
- ML pipeline.
- RAG.
- Vector database.
- AI orchestration.
- AI scaling.

---

# 9. Learning AI Assistant

المساعد الذكي داخل Lesson خارج Launch V1 ما لم يتم إصدار قرار Scope Change مستقل.

يشمل ذلك:

- Anthropic integration كمتطلب للإطلاق.
- AI usage limits.
- Prompt architecture.
- AI analytics.
- AI moderation.
- AI billing.

بما أن مكوّن AI يظهر حاليًا داخل Learning UI في السورس، يجب إخفاؤه في Launch V1 بدل الاعتماد على عدم وجود API Key فقط.

---

# 10. Affiliates

نظام Affiliate خارج Launch V1.

يشمل:

- Affiliate enrollment.
- Partner dashboard.
- Tracking.
- Click attribution.
- Affiliate conversions.
- Affiliate commissions.
- Affiliate admin resources.

لا يتم إكمال تجربة المستخدم خلال أسبوع الإغلاق.

---

# 11. Alumni

Alumni خارج Launch V1.

لا يدخل:

- Alumni portal.
- Alumni profiles.
- Alumni networking.
- Alumni engagement.

---

# 12. B2B Portal

B2B Platform كاملة خارج Launch V1.

يشمل:

- Corporate accounts.
- Company dashboards.
- Team management.
- Seat management.
- Corporate enrollment.
- Company billing.
- B2B reports.
- Corporate training portal.

التوصية للإطلاق:

إخفاء `/for-business`.

ويمكن الاكتفاء لاحقًا بصفحة تسويقية بسيطة أو CTA إلى Contact دون اعتبارها B2B System.

---

# 13. Consultations

التوصية المعتمدة لـScope Lock:

> Consultations خارج Launch V1.

يشمل:

- Consultation requests.
- Tracking.
- Consultation payment.
- Instructor consultation pricing.
- Consultation admin workflows.

ملاحظة تقنية:

الاستشارات مفعلة افتراضيًا حاليًا في Frontend Feature configuration، كما توجد روابط مباشرة لها في الموقع.

لذلك يجب إخفاؤها فعليًا إذا اعتمد هذا الـScope.

أي قرار لإعادتها إلى V1 يعتبر Scope Change صريحًا.

---

# 14. SMS

خارج Launch V1:

- SMS notifications.
- SMS marketing.
- SMS verification.
- Unifonic production integration.

---

# 15. Phone OTP

خارج Launch V1:

- Login by phone.
- Phone OTP.
- Phone verification.
- OTP through SMS.

Launch V1 يعتمد:

```text
Email + Password
```

---

# 16. Internal 100ms Experience

خارج Launch V1:

- 100ms full integration.
- Native room creation workflows.
- Embedded live classroom.
- Advanced live broadcasting.
- Polls.
- Breakout rooms.
- Recording automation.
- Complex attendance integration with live rooms.

V1 يستخدم روابط الاجتماعات الخارجية فقط.

---

# 17. QR Attendance Scanner & Advanced Workflows

> **تنبيه نطاق العمل (Scope Rule):**
> **QR Attendance الأساسي لتحضير الطلاب = In Scope ✅** (وهو جزء من متطلبات V1 الأساسية).

خارج Launch V1 فقط (ميزات QR المستقبلية والأجهزة غير المتعلقة برحلة التحضير الأساسية):

- ميزات QR المستقبلية غير المتعلقة برحلة التحضير الأساسية للطالب.
- إدارة أجهزة وأجهزة المسح المتقدمة (Scanner device workflows).
- إعادة توليد QR وتخصيصه المتقدم الممتد خارج نطاق التحضير البسيط.


---

# 18. Advanced Attendance System

خارج Launch V1 ما لم يكن برنامج فعلي يعتمد عليه.

يشمل:

- Advanced attendance policies.
- Scanner workflows.
- Automated attendance percentage governance.
- Attendance analytics.

---

# 19. Subscriptions

Student subscription tiers خارج Launch V1.

تبقى:

```text
subscriptions = false
```

ولا يدخل:

- Monthly plans.
- Annual plans.
- Recurring billing.
- Renewal.
- Subscription cancellation.
- Subscription entitlements.

---

# 20. Bundles

Bundles خارج Launch V1.

تبقى:

```text
bundles = false
```

ولا يدخل:

- Program bundles.
- Bundle pricing.
- Bundle discount engine.
- Bundle checkout.

---

# 21. Vimeo

Vimeo خارج قائمة Video Providers المضمونة في Launch V1.

لا يتم:

- تطوير player خاص.
- إصلاح integration.
- اختبار production delivery.

ويفضل إخفاؤه من authoring UI.

---

# 22. Mux

Mux خارج Launch V1.

يتم إخفاؤه من اختيار Video Provider خلال V1.

---

# 23. Advanced Content Types

ليس ضمن التزام V1 ضمان جميع أنواع المحتوى التجريبية أو الأقل اختبارًا.

V1 يضمن:

```text
Video
PDF / Resources
Quiz
Assignment
```

أي نوع إضافي لا يدخل Acceptance Criteria إلا بعد اختبار واعتماد مستقل.

---

# 24. Gamification / Achievements

نظام Gamification الكامل خارج Launch V1.

يشمل:

- Achievement expansion.
- Badges redesign.
- Gamification engine.
- Points.
- Leaderboards.
- Rewards.

إذا بقيت صفحة موجودة في السورس فلا تعتبر جزءًا من Acceptance Criteria.

التوصية هي إخفاؤها إن لم تكن مجربة.

---

# 25. Wishlist

Wishlist ليست جزءًا من Launch V1 الأساسي.

لا يتم تطويرها أو إعادة تصميمها خلال أسبوع الإغلاق.

يمكن إبقاء backend الموجود دون توسيع.

---

# 26. Gifts

نظام Gifts خارج Launch V1.

يشمل:

- Gift purchase.
- Gift redemption UX.
- Gift management.
- Gift campaigns.

الكود والجداول يمكن أن تبقى.

---

# 27. Wallet Expansion

تطوير نظام Wallet خارج Launch V1.

لا يدخل:

- Wallet funding.
- Withdrawals.
- Advanced credit management.
- Refund wallet architecture.
- Wallet redesign.

إذا كانت هناك أرصدة Production حالية يعتمد عليها مستخدمون حقيقيون، يجب الحفاظ على Backward Compatibility وعدم حذفها.

لكن Wallet ليست Release Feature جديدة يجب تطويرها هذا الأسبوع.

---

# 28. Advanced Messaging

تطوير Messaging System جديد خارج Launch V1.

لا يدخل:

- Realtime chat.
- Advanced inbox.
- Attachments redesign.
- Presence.
- Read receipts architecture.
- Push integration.

يمكن الاحتفاظ بالوظائف الحالية إذا كانت مستقرة دون إضافة أعمال تطوير إليها.

---

# 29. Advanced Notifications

خارج Launch V1:

- Push notification platform.
- Complex notification preferences.
- Mobile push campaigns.
- Notification automation engine جديد.

يدخل في V1 فقط ما تحتاجه العمليات الأساسية حاليًا.

---

# 30. Accreditation Platform

ميزات الاعتماد المتقدمة خارج Launch V1.

يشمل:

- NELC readiness workflows.
- TVTC workflows.
- CPD tracking.
- HRDF workflows.
- Accreditation management portal.

---

# 31. B2B Accreditation Accounts

أي B2B/Accreditation account workflows خارج Launch V1.

---

# 32. Advanced Analytics

خارج Launch V1:

- Data warehouse.
- BI dashboards.
- Advanced cohort analytics.
- Revenue intelligence.
- Predictive analytics.
- Advanced funnel analysis.
- Dedicated marketing analytics platform.

Basic operational KPIs الموجودة يمكن أن تبقى إذا كانت مستقرة.

---

# 33. CRM Expansion

خارج Launch V1:

- Full CRM.
- Sales pipelines.
- Lead scoring.
- Automated lead nurturing.
- CRM sequences.
- Sales automation.

---

# 34. Abandoned Cart Automation

تطوير Abandoned Cart system جديد خارج Launch V1.

وجود resource حالي لا يجعله شرط إطلاق.

---

# 35. Advanced Marketing Automation

خارج Launch V1:

- Complex email campaigns.
- Drip campaigns.
- Segmentation engine.
- Marketing automation.
- Retargeting engine.

---

# 36. Newsletter Expansion

يمكن بقاء الاشتراك البسيط الموجود إذا كان يعمل دون مشاكل.

لكن خارج Launch V1:

- Newsletter campaign builder.
- Segmentation.
- Analytics.
- Marketing automation.

---

# 37. Advanced Partner Management

تطوير Partner/Sponsor platform خارج Launch V1.

يمكن عرض شركاء ثابتين على الموقع عند الحاجة، لكن إدارة نظام شراكات متكامل ليست هدف هذا الإصدار.

---

# 38. Learning Paths Expansion

تطوير Learning Paths كمنتج مستقل أو محرك تعليمي متقدم خارج Launch V1.

التركيز هو Programs الحالية.

---

# 39. Advanced Landing Page Builder

إعادة تصميم Landing Page الرئيسية تدخل V1.

لكن خارج النطاق:

- بناء Visual Website Builder جديد.
- Drag & Drop editor.
- Template marketplace.
- Multi-site landing page platform.

---

# 40. Advanced Media Platform

خارج Launch V1:

- Media transcoding platform.
- Video processing architecture جديدة.
- Bunny Stream migration شامل.
- Media DAM متقدم.
- Automatic optimization pipeline جديد.

المطلوب فقط جعل Media المستخدم فعليًا في V1 يعمل بثبات.

---

# 41. Framework Upgrades

خارج Launch V1:

- Laravel major upgrade.
- Next.js major upgrade.
- Filament major upgrade.
- React migration.
- PHP major migration.
- Node ecosystem migration.
- Large dependency upgrades.

---

# 42. Dependency Mass Updates

Dependabot أو تحديثات الحزم غير الضرورية للإطلاق خارج Scope.

لا يتم دمجها لمجرد أنها متاحة.

---

# 43. Major Refactoring

خارج Launch V1:

- إعادة هيكلة Architecture.
- إعادة بناء Modules.
- تغيير Domain boundaries.
- إعادة تسمية واسعة.
- Massive cleanup.
- Rewriting existing working modules.

---

# 44. Database Redesign

خارج Launch V1:

- إزالة tenant schema.
- تغيير جميع العلاقات.
- إعادة بناء قاعدة البيانات.
- تحويل UUIDs.
- إعادة كتابة migration history.

---

# 45. Future Migrations غير المعتمدة

أي Migration موجود في Repository لكنه غير معتمد ضمن Migration Allowlist الخاص بـV1 يعتبر Out of Scope.

خصوصًا أن السورس الحالي يحتوي Migrations مؤرخة بعد تاريخ الإغلاق الحالي حتى:

```text
August
September
October
November 2026
```

لا يجوز اعتبار وجودها في Repository إذنًا لتطبيقها.

---

# 46. Blanket Production Migration

خارج Scope وممنوع:

```text
php artisan migrate --force
```

أو Tenant migration شامل إذا كان سيشغّل جميع Pending Migrations دون مراجعة.

يجب تطبيق قائمة Release-approved فقط.

---

# 47. Experimental Features

أي Feature:

- Experimental.
- Partial.
- Scaffold.
- Skeleton.
- Unverified.
- غير مستخدمة من العميل حاليًا.

تعتبر Out of Scope افتراضيًا.

---

# 48. Features غير الموجودة في Acceptance Criteria

وجود صفحة أو API أو Resource في Repository لا يجعلها ضمن V1.

القاعدة:

> إذا لم تكن الميزة مذكورة في In Scope ولم تكن ضرورية لرحلة إطلاق معتمدة، فلا يتم تخصيص وقت تطوير لها خلال Closure Week.

---

# 49. Out-of-Scope Bug Policy

إذا ظهر Bug في Feature خارج Scope:

### إذا كان لا يؤثر على V1

لا يتم إصلاحه الآن.

يوضع في Backlog.

### إذا كان يؤثر على V1 بصورة غير مباشرة

يتم إصلاح الجزء الذي يمنع V1 فقط.

ولا يتم تحويل ذلك إلى مشروع تطوير كامل للميزة.

---

# 50. Scope Change Policy

أي طلب بإعادة ميزة من هذه الوثيقة إلى Launch V1 يجب أن يمر عبر:

1. تحديد الميزة.
2. سبب إدخالها.
3. أثرها على الجدول.
4. أثرها على الاختبارات.
5. أثرها على Production.
6. تحديد ما سيتم إخراجه مقابلها إذا لزم.
7. اعتماد القرار قبل بدء التطوير.

---

# 51. قاعدة الإطلاق النهائية

لا يتم تأخير إطلاق ومضات لأن Repository يحتوي على Features إضافية غير مكتملة.

الإطلاق يعتمد فقط على:

```text
Core Education
Core Commerce
Core Administration
Production Reliability
```

والهدف هو:

> إطلاق منصة ومضات الأساسية بصورة مستقرة ومهنية، ثم فتح المراحل التالية للMarketplace والموبايل والذكاء الاصطناعي وSaaS وغيرها بعد استقرار V1.