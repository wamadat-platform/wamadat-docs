# منصة ومضات التعليمية
## خطة التصفية النهائية لميزات ما بعد Launch V1
### V2 / V1.1 / Future Features Exposure Cleanup Plan

**نوع الوثيقة:** خطة تنفيذ تقنية نهائية  
**الإصدار:** 1.0  
**تاريخ الإعداد:** 21 أغسطس 2026  
**المشروع:** منصة ومضات التعليمية  
**المستودعات المشمولة:** `wamadat-web` + `wamadat-backend`  
**الهدف:** جعل نسخة Launch V1 تعرض فقط الوظائف المعتمدة في المرحلة الأولى، وإخفاء كل ميزة مؤجلة من جميع الأسطح القابلة للوصول أو الاكتشاف، بدون حذف الكود أو البيانات أو الجداول.

---

# 1. الهدف التنفيذي

المطلوب هو تنفيذ **Launch V1 Surface Lockdown** كامل.

بعد التنفيذ يجب أن يرى:

- الزائر.
- الطالب.
- المدرب.
- الإدارة.

فقط الوظائف المعتمدة للمرحلة الأولى.

أي ميزة مقررة لـ:

- V1.1.
- Phase 2.
- Phase 3.
- أي مرحلة مستقبلية.

يجب ألا:

- تظهر في Landing Page.
- تظهر في Navbar.
- تظهر في Footer.
- تظهر في Mega Menu.
- تظهر في Dashboard.
- تظهر كـQuick Action.
- تظهر داخل Program Details.
- تظهر داخل Profile/Settings إذا كانت مؤجلة.
- تظهر داخل Admin Panel.
- تظهر داخل Instructor Panel.
- تظهر داخل Relation Managers.
- تظهر في Sitemap.
- تظهر في SEO Structured Data.
- تظهر عبر رابط مباشر.
- تعود من CMS/Nav/Footer configuration.
- تنفذ Background Jobs أو Emails خاصة بها.
- تعاد بالخطأ إلى الواجهة من Feature Flag غير مضبوط.

---

# 2. مبدأ التنفيذ

## لا نحذف الكود

ممنوع في هذه المهمة:

```text
DROP TABLE
DELETE migrations
حذف Modules
حذف Models
حذف Controllers
حذف بيانات Production
Refactor جذري
```

الكود المستقبلي يبقى محفوظًا.

---

## المطلوب هو الإخفاء المحكم

النموذج الصحيح:

```text
Feature code remains
+
Feature data remains
+
Feature migrations remain
+
Feature disabled by release scope
+
No public/admin/instructor exposure
```

وبذلك يمكن إعادة تشغيل الميزة في إصدار لاحق بصورة مدروسة.

---

# 3. القاعدة الأساسية: V1 Allowlist

بدل بناء قائمة Blacklist فقط، يجب اعتماد:

> **كل شيء مخفي ما لم يكن ضمن Launch V1 Allowlist.**

هذه الطريقة أكثر أمانًا من مطاردة كل Feature جديدة قد يضيفها الكود مستقبلًا.

---

# 4. Launch V1 Allowlist المعتمد

## Public / Visitor

المسموح:

```text
Home
Programs
Program Details
Categories
Search
Instructors
About
Contact
Help
Legal Pages
Sign Up
Sign In
Password Reset
Certificate Verification
```

---

## Commerce

المسموح:

```text
Cart
Coupons
Checkout
Electronic Payment Methods المعتمدة
Bank Transfer
Orders
Payments
Invoices
```

---

## Student

المسموح:

```text
Dashboard / Today — بعد تنظيفه
My Programs
Assignments
Orders & Payments
Certificates
Support
Notifications الأساسية
Profile الأساسي
Settings الأساسية والحقوق النظامية
Learning
Lesson Notes
Lesson Q&A
Quiz
Assignment Submission
Progress
Completion
Attendance V1
Certificate PDF
Certificate Verification
```

---

## Instructor

المسموح:

```text
Instructor Home
Programs
Lessons
Students / Trainees
Quiz
Assignments
Grading
Lesson Questions
Attendance V1
Certificates ضمن البرنامج إذا كانت ضمن الصلاحيات
```

---

## Admin

المسموح:

```text
Dashboard
Programs
Lessons
Students
Enrollments
Instructors
Quizzes
Assignments
Orders
Payments
Bank Transfers
Invoices
Coupons
Issued Certificates
Support Tickets
Site Settings
```

---

# 5. الميزات التي يجب إخفاؤها بالكامل

تشمل التصفية كل ما يلي.

## V1.1 Deferred

```text
Direct Student ↔ Instructor Messaging
Program Reviews / Ratings
Streak
Achievements
Gamification
Program Gifts
Gift Redemption
Program Interest / Waitlist
Advanced Student Profile
Advanced Notification Preferences
Advanced Account Sessions UI
2FA UI إن لم يكن جزءًا من سياسة الدخول الإلزامية
Student Calendar enhancements
Wishlist UI
Wallet UI
```

---

## Phase 2 Deferred

```text
Live Sessions
Community / General Forum
Program Discussions خارج Lesson Q&A
Program Announcements المتقدمة
Consultations
Advanced Analytics surfaces
Push Notification UI
SMS
Marketing Automation
CRM / Leads
Advanced Attendance
Newsletter / Marketing Subscription UI
```

---

## Phase 3 / Future Deferred

```text
Flutter Mobile Product references
AI Assistant
Wamadat Plus / Marketplace
Subscriptions
Bundles
Affiliate
Alumni
B2B Portal
Multi-Academy / SaaS UI
Tenant Management
Academy Plans
Academy Subscriptions
```

---

# 6. Master Release Gate — Frontend

يجب إضافة Master Gate أعلى جميع Feature Flags.

اقتراح:

```env
NEXT_PUBLIC_RELEASE_SCOPE=v1
```

وفي `lib/features.ts`:

```ts
const IS_LAUNCH_V1 = process.env.NEXT_PUBLIC_RELEASE_SCOPE === 'v1';
```

ويجب أن يكون سلوك أي Feature مستقبلية:

```ts
enabled = !IS_LAUNCH_V1 && normalFeatureFlag
```

أي أن وجود:

```env
NEXT_PUBLIC_FEATURE_PLUS=true
```

بالخطأ لا يكفي لإظهار Plus إذا:

```env
NEXT_PUBLIC_RELEASE_SCOPE=v1
```

---

# 7. Master Release Gate — Backend

إضافة:

```env
RELEASE_SCOPE=v1
```

ويتم إنشاء مصدر مركزي مثل:

```text
config/release.php
```

أو توسيع:

```text
config/features.php
```

بحيث تصبح كل Feature مستقبلية مغلقة بقيدين:

```text
RELEASE_SCOPE != v1
AND
FEATURE_X=true
```

لا تعتمد على Feature Flag منفردة.

---

# 8. Feature Flags المطلوبة

الموجود حاليًا في Frontend يشمل:

```text
subscriptions
bundles
consultations
plus
business
aiAssistant
sms
affiliate
alumni
```

لكن هناك Features مؤجلة **غير محمية حاليًا بFeature Flags**.

أضف Gate مركزيًا لـ:

```env
NEXT_PUBLIC_FEATURE_DIRECT_MESSAGING=false
NEXT_PUBLIC_FEATURE_REVIEWS=false
NEXT_PUBLIC_FEATURE_GAMIFICATION=false
NEXT_PUBLIC_FEATURE_GIFTS=false
NEXT_PUBLIC_FEATURE_PROGRAM_INTEREST=false
NEXT_PUBLIC_FEATURE_LIVE_SESSIONS=false
NEXT_PUBLIC_FEATURE_COMMUNITY=false
NEXT_PUBLIC_FEATURE_NEWSLETTER=false
NEXT_PUBLIC_FEATURE_ADVANCED_PROFILE=false
NEXT_PUBLIC_FEATURE_ADVANCED_NOTIFICATION_SETTINGS=false
NEXT_PUBLIC_FEATURE_CALENDAR=false
NEXT_PUBLIC_FEATURE_WALLET=false
NEXT_PUBLIC_FEATURE_WISHLIST=false
```

وفي Backend:

```env
FEATURE_DIRECT_MESSAGING=false
FEATURE_REVIEWS=false
FEATURE_GAMIFICATION=false
FEATURE_GIFTS=false
FEATURE_PROGRAM_INTEREST=false
FEATURE_LIVE_SESSIONS=false
FEATURE_COMMUNITY=false
FEATURE_NEWSLETTER=false
FEATURE_MARKETING_AUTOMATION=false
FEATURE_PUSH=false
FEATURE_CALENDAR=false
```

> يمكن تبسيط عدد المتغيرات داخليًا باستخدام `RELEASE_SCOPE=v1` ومصفوفة Feature Map بدل الاعتماد على عدد كبير من ENV variables، لكن يجب أن يوجد مصدر حقيقة واحد.

---

# 9. تحديث `lib/features.ts`

الملف الحالي يحمي:

- Plus.
- Consultations.
- Business.
- Subscriptions.
- Bundles.
- AI.
- Affiliate.
- Alumni.
- SMS.

لكنه لا يحمي:

- `/dashboard/live-sessions`
- `/dashboard/messages`
- `/trainer/messages`
- `/dashboard/achievements`
- `/redeem-gift`
- Reviews.
- Waitlist.
- Gamification.
- Newsletter.
- Calendar.
- Wallet.

يجب توسيع `isFeaturePathEnabled()` بحيث يمنع جميع المسارات المؤجلة.

---

# 10. قائمة المسارات المحجوبة في V1

يجب أن يرجع الوصول المباشر لها 404 أو Not Found آمن.

## Student

```text
/dashboard/live-sessions
/dashboard/live-sessions/*
/dashboard/messages
/dashboard/achievements
/dashboard/wallet
```

إذا وجدت صفحات إضافية مستقبلية:

```text
/dashboard/reviews
/dashboard/gifts
/dashboard/calendar
/dashboard/wishlist
```

تحجب أيضًا.

---

## Instructor

```text
/trainer/messages
```

وأي واجهة Frontend مستقبلية لـ:

```text
/trainer/live-sessions
/trainer/community
```

---

## Public

```text
/consultations
/consultations/*
/plus
/plus/*
/redeem-gift
/for-business
/subscriptions
/subscriptions/*
/bundles
/bundles/*
/affiliate
/affiliate/*
/alumni
/alumni/*
/ai-assistant
/ai-assistant/*
```

---

# 11. منع الوصول المباشر — لا يكفي حذف الرابط

بعض الميزات الحالية لها Layout Gate مثل:

```text
/consultations
/plus
/for-business
/dashboard/plus
```

وهذا جيد.

لكن Features مثل:

```text
/dashboard/live-sessions
/dashboard/messages
/dashboard/achievements
/trainer/messages
/redeem-gift
```

يمكن الوصول لها مباشرة حاليًا.

المطلوب:

## الخيار المفضل

إضافة Release Route Gate مركزي في:

```text
middleware.ts
```

بعد إزالة locale prefix.

مثال منطقي:

```ts
if (isReleaseBlockedPath(strippedPath)) {
    return NextResponse.rewrite(new URL('/ar/not-found', request.url));
}
```

أو استخدام بنية Next الصحيحة لإرجاع 404.

---

## دفاع إضافي

يفضل أيضًا وضع:

```ts
notFound()
```

في Layouts الخاصة بالمجموعات المؤجلة.

الهدف:

```text
Navigation hide
+
Middleware gate
+
Page/Layout gate
```

Defense in Depth.

---

# 12. Navbar — تصفية كاملة

الملف:

```text
components/layouts/navbar.tsx
```

يوجد حاليًا في `NAV_LINKS`:

```text
/plus
/consultations
/for-business
```

وهي تمر على Feature Filter، لكن يجب الإبقاء على الحماية.

---

## مشكلة مهمة في `SHOWCASE_CARDS`

المصدر الحالي يحتوي بطاقات ثابتة:

```text
ومضات بلس
الاستشارات والشركات
```

ويتم عمل:

```ts
SHOWCASE_CARDS.map(...)
```

مباشرة بدون `isFeaturePathEnabled()`.

المطلوب:

```ts
SHOWCASE_CARDS
  .filter(card => isFeaturePathEnabled(card.href))
  .map(...)
```

أو إزالة بطاقات V2 بالكامل في `RELEASE_SCOPE=v1`.

---

## مشكلة مهمة في `QUICK_PILLS`

المصدر يحتوي:

```text
الورش المباشرة → /programs?mode=live
ومضات بلس
الاستشارات
للشركات
```

ويتم عرضها مباشرة.

المطلوب:

- حذف `الورش المباشرة` من V1.
- فلترة Plus.
- فلترة Consultations.
- فلترة B2B.
- منع أي dynamic pill مستقبلية غير V1.

---

# 13. Footer — تصفية كاملة

الملف:

```text
components/layouts/footer.tsx
```

الروابط الحالية لـ:

```text
Consultations
Plus
For Business
```

تمر على Feature Filter.

يجب الحفاظ على ذلك.

---

## Newsletter

الـFooter يعرض حاليًا:

```tsx
<NewsletterSignup source="footer" variant="light" />
```

دائمًا.

بما أن Newsletter / Marketing Automation مؤجلان:

- لا يظهر Newsletter Signup في V1.
- يحاط بـFeature Gate.
- لا تذكر "اشترك في النشرة" في Footer أو أي صفحة.

---

# 14. Dynamic CMS Navigation

الموقع يستطيع استقبال:

```text
nav_items
footer_links
operator pages
```

من Site Settings / CMS.

الموجود حاليًا يستخدم:

```text
isFeaturePathEnabled()
```

وهذا ممتاز.

المطلوب توسيع `isFeaturePathEnabled()` لكل V2 path حتى لا يستطيع Admin أو بيانات قديمة إعادة Feature مؤجلة إلى Navbar/Footer.

---

# 15. Announcement Bar

الملف:

```text
components/layouts/announcement-bar.tsx
```

يستخدم Feature Path Filter.

بعد توسيع الـFilter:

- أي Promo يؤدي إلى V2 يجب ألا يظهر CTA الخاص به.
- الأفضل إخفاء الإعلان كله إذا كان موضوعه Feature مخفية وليس الرابط فقط.

---

# 16. CMS Page Sections

الملف:

```text
components/cms/page-sections.tsx
```

حاليًا يمنع CTA URLs المعطلة.

المطلوب:

- الحفاظ على الفلترة.
- عدم عرض Button أو Link إلى V2.
- فحص النصوص نفسها؛ لأن الـCMS قد يحتوي نصًا يروج للاستشارات أو الجلسات حتى بدون رابط.

---

# 17. Landing Page — إزالة أي ذكر لميزات V2

الملف:

```text
app/[locale]/page.tsx
```

حاليًا Consultations وBusiness محميان بFeature Flags.

يجب إبقاء:

```tsx
FEATURES.consultations
FEATURES.business
```

مغلقين في V1.

لكن هناك نصوص ومكونات أخرى تذكر Features مؤجلة بصورة غير مباشرة.

---

# 18. Landing — `messages/ar.json`

يجب تنظيف النصوص التالية.

## `meta.site_description`

الحالي يذكر:

```text
لقاءات مباشرة
ورش حضورية
```

استبداله بنص V1 يركز على:

```text
البرامج
التعلم المرن
الاختبارات
الواجبات
الشهادات الموثقة
```

---

## `hero.subhead`

الحالي يذكر:

```text
لقاءات مباشرة
ورش حضورية
```

يستبدل بنص Launch V1 فقط.

---

## `features`

الحالي يحتوي:

```text
لقاءات مباشرة
ورش حضورية
```

يتم إعادة بناء Features V1 إلى شيء مثل:

```text
تعلم على وقتك
محتوى منظم وموارد
اختبارات وواجبات
شهادات موثقة
```

---

## `how_it_works.s3_body`

الحالي:

```text
فيديو وصوت وملفات ولقاءات مباشرة
```

يعدل إلى:

```text
فيديو وملفات ومحتوى تعليمي منظم واختبارات وواجبات
```

---

## FAQ

احذف سؤال:

```text
كيف تجري اللقاءات المباشرة؟
```

واستبدله بسؤال V1 مثل:

```text
كيف أتابع تقدمي في البرنامج؟
```

أو:

```text
متى أحصل على الشهادة؟
```

---

## Refund FAQ

راجع النص الذي يذكر:

```text
حضورية أو مباشرة عبر الإنترنت
```

يجب أن يطابق طرق البرامج المعروضة فعليًا في V1.

---

# 19. Landing — `HowItWorks`

الملف:

```text
components/marketing/how-it-works.tsx
```

النص الثابت حاليًا يذكر:

```text
لقاءات مباشرة تتفاعل فيها
ورش حضورية
```

يجب تحديثه مباشرة.

لا تعتمد فقط على Translation file لأن هذا النص Hardcoded في المكون.

---

# 20. Landing — `Features`

الملف:

```text
components/marketing/features.tsx
```

المشكلة ليست فقط Fallback translations.

الـFeatures قد تأتي من:

```text
home_cards
```

من قاعدة البيانات.

لذلك يجب:

1. تعديل fallback.
2. مراجعة `home_cards` الحالية في DB.
3. حذف/تعطيل Cards التي تروج لـ:
   - Live.
   - Consultations.
   - B2B.
   - Community.
   - Plus.
4. عدم حذف السجلات؛ يكفي `is_visible=false` أو استبدال محتواها حسب النظام.

---

# 21. Landing — `StartHere`

راجع:

```text
components/marketing/start-here.tsx
```

والـCMS Cards.

الموجود في النصوص الحالية:

```text
أمثّل جهة أو فريق
حلول تدريب لمؤسستك
```

هذا B2B.

في V1:

- أخفِ Card الخاصة بالجهات.
- لا توجه إلى `/for-business`.

---

# 22. Landing — `Worlds`

الملف:

```text
components/marketing/worlds.tsx
```

يحتوي:

```text
program
path
lab
initiative
forum
```

### قرار V1

`Community / General Forum` مؤجل.

لكن `ProgramType=forum` في السورس هو **نوع Offering** وليس بالضرورة المنتدى الاجتماعي نفسه.

لذلك:

- لا تحذف Enum أو بيانات `forum`.
- لا تحول هذا التعديل إلى Migration.
- إذا كان العميل لا يستخدم ملتقيات كبرامج في Launch V1، أخفِ Card `forum` من Landing وفلتر Public Catalog.
- إذا توجد برامج V1 فعلية من النوع `forum`، تعامل معها كبرنامج فقط ولا تعرض أي Community functionality.

الهدف منع الخلط بين:

```text
Offering type: ملتقى
```

و:

```text
Community Forum feature
```

---

# 23. Landing — Programs Preview

الملف:

```text
components/marketing/programs-preview.tsx
```

الحالي يحتوي Tab:

```text
المباشرة والورش
```

ويصنف:

```text
live
workshop
cohort
in_person
```

المطلوب في V1:

- إزالة Tab `live`.
- عدم الترويج لـLive Sessions.
- فحص البرامج Featured القادمة من API.
- أي برنامج `mode=live` يجب ألا يظهر في V1 إلا إذا تم اعتماد طريقة تشغيل غير Live Session ضمن النطاق.

---

# 24. Landing — تقييمات البرامج

`ProgramsPreview` يعرض:

```text
average_rating
reviews_count
```

بما أن Reviews مؤجلة:

- لا تعرض Stars.
- لا تعرض Review Count.
- لا تستخدم User-generated rating كSocial Proof في V1.

---

# 25. Program Cards

الملف:

```text
components/programs/program-card.tsx
```

أخفِ في V1:

```text
average_rating
reviews_count
```

لا تترك:

```text
4.8 (27)
```

في البطاقة بينما صفحة Reviews نفسها مخفية.

---

# 26. Instructor Cards

الملف:

```text
components/marketing/instructors-preview.tsx
```

يوجد عرض لـ:

```text
average_rating
```

إذا هذا الرقم مبني على Review System المؤجل:

- أخفه في V1.

إذا كان Rating مستقلاً ومدارًا من الإدارة وليس Reviews، وثق ذلك قبل إبقائه.

الافتراضي في هذه الخطة: **إخفاؤه** لتجنب التناقض.

---

# 27. Program Details — إزالة Reviews بالكامل

الملف:

```text
app/[locale]/programs/[slug]/page.tsx
```

الحالي:

```text
fetchProgramReviews()
TestimonialsMarquee
ProgramReviewForm
average_rating
reviews_count
AggregateRating JSON-LD
```

المطلوب في V1:

- لا تنفذ `fetchProgramReviews`.
- لا تعرض `TestimonialsMarquee`.
- لا تعرض `ProgramReviewForm`.
- لا تعرض average rating.
- لا تعرض reviews count.
- لا تضف `AggregateRating` إلى JSON-LD.
- لا تضف `Review` schema إلى SEO.

هذا مهم حتى لا يبقى Review System ظاهرًا لمحركات البحث رغم اختفائه بصريًا.

---

# 28. Program Details — Gifts

الملف:

```text
components/programs/program-detail-cta.tsx
```

حاليًا يعرض:

```tsx
<GiftProgramButton />
```

المطلوب:

- عدم Render في V1.
- عدم Import إذا لم يكن مطلوبًا.
- لا يظهر CTA:
  - أهدِ البرنامج.
  - شراء كهدية.
  - إرسال هدية.

---

# 29. `/redeem-gift`

حاليًا المسار يعمل مباشرة ولا يملك Feature Gate.

الملفات:

```text
app/[locale]/redeem-gift/page.tsx
app/[locale]/redeem-gift/layout.tsx
```

المطلوب:

- Feature Gate.
- `notFound()` في V1.
- Middleware block.
- يبقى `robots: noindex`.
- لا يظهر في أي Email جديد أثناء V1.
- لا يظهر في Navigation.

---

# 30. Program Interest / Waitlist

الملف:

```text
app/[locale]/programs/[slug]/page.tsx
```

حاليًا عند:

```text
interest_capture=true
```

يعرض:

```text
ProgramInterestForm
```

وهذا Feature مؤجل.

المطلوب:

## في V1

إذا التسجيل مغلق أو البرنامج ممتلئ:

اعرض حالة بسيطة فقط:

```text
التسجيل مغلق
```

أو:

```text
اكتمل العدد
```

بدون:

```text
سجل اهتمامك
أدخل بريدك
انتظر الدفعة القادمة
```

---

# 31. Backend Program Interest

المسار الحالي مفتوح:

```text
POST /catalog/programs/{slug}/interest
```

المطلوب:

- Gate بـ`FEATURE_PROGRAM_INTEREST`.
- في `RELEASE_SCOPE=v1` لا يتم تسجيل Route أصلًا، أو يرجع 404.
- لا ينشئ Leads جديدة أثناء V1.

---

# 32. Student Dashboard Navigation

الملف:

```text
app/[locale]/dashboard/layout.tsx
```

الحالي يحتوي:

```text
/dashboard/live-sessions
/dashboard/messages
```

يجب حذفهما من `NAV_ITEMS` في V1.

المسموح:

```text
اليوم
برامجي
الواجبات
الطلبات والمدفوعات
شهاداتي
الدعم والمساعدة
```

والإشعارات تبقى عبر Notification Bell.

---

# 33. Student Account Menu

في `dashboard/layout.tsx` يوجد رابط:

```text
/dashboard/achievements
إنجازاتي
```

يجب إخفاؤه.

راجع أيضًا:

```text
Wallet
Wishlist
Calendar
```

وأي روابط Account Menu غير V1.

---

# 34. Today Dashboard — تنظيف جذري

الملف:

```text
app/[locale]/dashboard/today/page.tsx
```

الحالي يعرض:

```text
Streak
Upcoming Live Sessions
Pending Assignments
```

وفيه Links إلى:

```text
/dashboard/achievements
/dashboard/live-sessions
```

المطلوب:

## إزالة

```text
سلسلة التعلم
عدد أيام Streak
Achievements CTA
جلسات قادمة
Upcoming Sessions list
Live Session CTAs
```

---

## إبقاء Today بسيطًا

مثال:

```text
متابعة البرنامج
التقدم الحالي
الدرس التالي
الواجبات المعلقة
الشهادة/الإكمال عند الاقتراب
```

لا توسع المهمة إلى إعادة تصميم كبيرة؛ فقط نظفها إلى V1.

---

# 35. Today API Payload

Backend endpoint:

```text
GET /me/today
```

قد يعيد:

```text
streak
upcoming sessions
```

هناك خياران:

## المفضل

أبقِ API backward-compatible لكن Frontend لا يستخدم الحقول المؤجلة.

## الأكثر صرامة

في `RELEASE_SCOPE=v1` لا يتم حساب بيانات Streak/Live التي لا تستخدم، لتقليل Queries.

لكن لا تكسر Contract بلا حاجة.

---

# 36. Achievements Route

المسار:

```text
/dashboard/achievements
```

المطلوب:

- لا Navigation.
- Middleware block.
- Page `notFound()` في V1.
- لا API calls.

---

# 37. Streak API

المسار Backend:

```text
GET /me/streak
```

المطلوب:

- Gate بالـRelease Scope.
- أو يبقى داخليًا بدون استهلاك Frontend.

الأفضل لإغلاق صارم:

```text
FEATURE_GAMIFICATION=false
```

ويتم عدم تسجيل Routes:

```text
me/streak
me/achievements
```

في V1.

---

# 38. Direct Messaging — Student

المسار:

```text
/dashboard/messages
```

والـAPI:

```text
GET  /me/conversations
POST /me/conversations
GET  /me/conversations/{conversation}
POST /me/conversations/{conversation}/messages
```

المطلوب:

- إزالة من Dashboard Nav.
- Route Not Found.
- Gate Backend routes.
- عدم توليد Notifications خاصة برسائل Direct Messaging أثناء V1.
- لا CTA "راسل المدرب".

---

# 39. Direct Messaging — Instructor

الملفات:

```text
app/[locale]/trainer/messages/page.tsx
app/[locale]/trainer/layout.tsx
app/[locale]/trainer/page.tsx
```

المطلوب:

- حذف `رسائل الطلاب` من `TRAINER_NAV`.
- حذف Quick Action `الرسائل`.
- حجب `/trainer/messages`.
- لا Text يشير إلى "الرد على استفسارات الطلاب" كرسائل خاصة.

Lesson Q&A يبقى؛ لأنه جزء V1.

---

# 40. Live Sessions — Student

المسارات:

```text
/dashboard/live-sessions
/dashboard/live-sessions/{id}
```

المطلوب:

- لا Navigation.
- لا Today card.
- لا Upcoming sessions section.
- Route Not Found.
- لا Email CTA يؤدي إليها.
- لا Analytics event يروج لها.

---

# 41. Live Sessions — Instructor Frontend

`app/[locale]/trainer/page.tsx` يحتوي Quick Action:

```text
الجلسات
إدارة الجلسات المباشرة والحضور
```

احذفه في V1.

---

# 42. Live Sessions — Instructor Filament

الملف:

```text
app/Providers/Filament/InstructorPanelProvider.php
```

حاليًا يسجل:

```php
LiveSessionResource::class
```

المطلوب:

```text
REMOVE from ->resources() in Launch V1
```

لا تحذف Resource file.

---

# 43. Live Sessions — Lesson Authoring

المشكلة الأكبر:

`LessonResource` يسمح بنوع:

```text
LessonType::Live
```

ويوجد Helper:

```text
الجلسات المباشرة تُدار من قسم الجلسات المباشرة
/admin/live-sessions
```

هذا يعرّض V2 من داخل Core V1.

المطلوب:

- حذف `Live` من Select options في Launch V1.
- لا يظهر Helper الخاص بالجلسات.
- لا يظهر رابط `/admin/live-sessions`.
- لا يمكن إنشاء Lesson جديدة من نوع Live.
- لا تحذف LessonType enum.
- لا تحذف بيانات Live القديمة.

---

# 44. Live داخل Curriculum Builder

الملف:

```text
ProgramResource/RelationManagers/LessonsRelationManager.php
```

حاليًا:

```text
LessonType::Live
LiveSessionResource::detailFormComponents()
notify_trainees
```

المطلوب:

- عدم تمرير `LessonType::Live` ضمن inlineTypes في V1.
- عدم Render `LiveSessionResource::detailFormComponents()`.
- عدم عرض `notify_trainees` المرتبط بالجلسة.
- منع إنشاء Live node في V1.

---

# 45. فحص بيانات Live قبل الإطلاق

قبل تعطيل الواجهة:

نفذ Audit على Production/Staging لمعرفة:

```text
published programs
with active live lessons
```

إذا وجدت:

- لا تحذفها.
- حدد هل هي مستخدمة في البرنامج الحالي.
- إما تحول الدرس إلى نوع V1.
- أو تزيله من البرنامج المنشور.
- أو تعتبر البرنامج خارج Launch V1.

**وجود Live Lesson ضرورية داخل برنامج سيتم إطلاقه = Launch Blocker.**

---

# 46. Program Discussions / Community

Lesson Q&A يبقى.

لكن Program-wide:

```text
DiscussionsRelationManager
AnnouncementsRelationManager
```

ليسا ضمن Launch V1.

في:

```text
ProgramResource::getRelations()
```

### Instructor

احذف من V1:

```text
DiscussionsRelationManager
AnnouncementsRelationManager
```

### Admin

احذف من V1:

```text
DiscussionsRelationManager
AnnouncementsRelationManager
```

لا تحذف الملفات.

---

# 47. Advanced Program Relation Managers

الـAdmin ProgramResource الحالي يحتوي ميزات كثيرة لا تدخل V1:

```text
InterestLeadsRelationManager
DiscussionsRelationManager
GlossaryRelationManager
CaseStudiesRelationManager
AnnouncementsRelationManager
EncyclopediaRelationManager
VolunteerOpportunitiesRelationManager
FaqsRelationManager
RevenueRelationManager — إذا اعتبر خارج التشغيل الأساسي
```

يجب تحويل `getRelations()` إلى **V1 explicit allowlist**.

---

# 48. Admin Program Relations — Allowlist المقترحة

## Admin V1

احتفظ فقط بـ:

```text
TrainersRelationManager
CertificatesRelationManager
AttendanceRelationManager
ModulesRelationManager
LessonsRelationManager
EnrollmentsRelationManager
QuestionsRelationManager
ReferencesRelationManager
CouponsRelationManager
```

### ملاحظة Revenue

إذا يحتاج Admin Revenue tab فعليًا في Launch V1 ويخدم المدفوعات، يمكن إبقاؤه.

وإلا:

```text
RevenueRelationManager → hide
```

لأن Orders/Payments/Invoices موجودة أصلًا كمصادر مستقلة.

---

## Instructor V1

احتفظ:

```text
ModulesRelationManager
LessonsRelationManager
MyTraineesRelationManager
QuestionsRelationManager
CertificatesRelationManager
AttendanceRelationManager
```

أخفِ:

```text
DiscussionsRelationManager
AnnouncementsRelationManager
```

---

# 49. Reviews — Backend

المسارات الحالية غير محمية:

```text
GET    /catalog/programs/{slug}/reviews
POST   /catalog/programs/{slug}/reviews
DELETE /me/reviews/{id}
```

المطلوب:

- Gate بـ`FEATURE_REVIEWS`.
- لا تسجل routes في V1.
- لا تقبل Review submission.
- لا ترجع Public review list.

---

# 50. Reviews — Admin Widgets

`NeedsActionWidget` يحتوي:

```text
تقييمات بانتظار الإشراف
```

ويشير إلى:

```text
ReviewResource
```

رغم أن ReviewResource غير مسجل في Admin V1.

احذف هذا Definition من V1.

---

# 51. Reviews — Recent Activity

إذا كان:

```text
RecentActivityFeedWidget
```

مستخدمًا أو سيعاد لاحقًا، فهو يجلب Reviews ويعرض:

```text
فلان قيّم البرنامج
```

في V1:

- لا Query للReviews.
- لا تعرض Rating activity.

---

# 52. Gifts — Backend

المسارات:

```text
POST /me/gifts
GET  /me/gifts/sent
POST /me/gifts/redeem
```

المطلوب:

- Gate بـ`FEATURE_GIFTS=false`.
- لا Route registration في V1.
- لا Gift emails/events جديدة.

---

# 53. Gift Admin

`GiftResource` موجود في الكود لكنه غير مسجل حاليًا.

تأكد:

- لا يتم إضافته إلى AdminPanelProvider.
- لا Widgets تشير له.
- لا Quick links.

---

# 54. Advanced Profile — Student

الملف:

```text
app/[locale]/dashboard/profile/page.tsx
```

الحالي يحتوي:

```text
bio
city
website
interests
skills
Instagram
LinkedIn
X
YouTube
Behance
Portfolio
Portfolio image URL
```

المرحلة الأولى تحتاج Profile أساسي فقط.

---

# 55. Profile V1 المقترح

أظهر فقط ما يلزم فعليًا:

```text
الاسم
الصورة
البريد — read only حسب النظام
الجوال — حسب القواعد
المدينة إذا لازمة
بيانات الشهادة اللازمة
```

ويمكن إبقاء:

```text
bio
```

فقط إذا كانت هناك حاجة فعلية.

أخفِ:

```text
skills
interests
social links
portfolio
website_url
advanced professional fields
```

لا تحذف البيانات المخزنة.

---

# 56. Settings — Marketing Preferences

الملف:

```text
app/[locale]/dashboard/settings/page.tsx
```

حاليًا يعرض:

```text
إيميل تسويقي
WhatsApp تسويقي
SMS تسويقي عند Flag
```

وبما أن Marketing Automation مؤجل:

- لا تظهر Marketing Preferences في V1.
- لا تعرض WhatsApp marketing toggle.
- SMS مغلق أصلًا.
- لا تعرض واجهة توحي بوجود حملات تسويقية.

يمكن الحفاظ على Consents في قاعدة البيانات.

---

# 57. Settings — Sessions Manager

حاليًا:

```tsx
<SessionsManager />
```

Advanced Account Sessions UI مؤجل إلى V1.1.

المطلوب:

- أخفه من V1 UI.
- لا تحذف Backend session security.
- لا تمنع النظام من إبطال الجلسات أمنيًا.
- هذا إخفاء UX وليس تعطيل Security.

---

# 58. PDPL — لا تخفِ الحقوق النظامية

لا تعتبر هذه V2 features:

```text
تنزيل نسخة من بياناتي
حذف الحساب
```

إذا كانت جزءًا من التزام PDPL الحالي.

**يجب إبقاؤها.**

أي Cleanup يجب ألا يحذف:

```text
Data Export
Account Deletion
Privacy page
Terms
Refund Policy
```

---

# 59. 2FA

إذا 2FA جزء من حماية إلزامية للAdmin:

```text
لا تعطله
```

أما Tenant User self-service 2FA UI إن كان غير مستخدم ضمن V1:

- لا تضع CTA له.
- يمكن إبقاء Backend endpoints.

الأمن ليس Feature تجميلية؛ لا تخفض مستوى الحماية بسبب Scope Cleanup.

---

# 60. Notifications

Basic Transactional Notifications تبقى.

المسموح:

```text
Notification Bell
Unread count
Notification list
Mark read
Mark all read
Action URL للميزات V1 فقط
```

---

# 61. Advanced Notifications المؤجلة

أخفِ:

```text
Notification channel preferences المتقدمة
Push setup
Marketing notification preferences
Segments
Campaign preferences
```

ويجب التأكد أن Action URL لأي Notification قديمة تشير إلى Feature مؤجلة:

- لا تظهر CTA غير صالح.
- أو يعاد توجيهها إلى صفحة V1 مناسبة.

---

# 62. Push Devices

Backend يحتوي:

```text
POST /me/devices
DELETE /me/devices
```

وهي استعداد لـPush/Mobile.

إذا Web V1 لا يستخدمها:

- Gate بـ`FEATURE_PUSH`.
- لا تستدعي من Frontend.
- لا تسجل Device tokens دون حاجة.

---

# 63. Calendar

Backend:

```text
GET /me/calendar.ics
```

إذا Calendar enhancements مؤجلة:

- لا يظهر CTA Export Calendar في V1.
- Gate route إذا لم يوجد استخدام V1.
- لا تروج لCalendar integration.

---

# 64. Wallet

يوجد:

```text
/dashboard/wallet
GET /me/wallet
```

والـDashboard comment يقول إنه خارج Daily IA.

لإغلاق كامل:

- Middleware block `/dashboard/wallet`.
- لا Link.
- Gate API أو اتركه dormant إذا لا يستدعى.
- لا يظهر Balance/Wallet لأي مستخدم.

---

# 65. Wishlist

Backend يدعم:

```text
GET /me/wishlist
POST /me/wishlist
DELETE /me/wishlist/{program_slug}
```

وTranslations تحتوي:

```text
أضف للمفضلة
```

إذا Wishlist ليست ضمن وثيقة V1:

- لا CTA.
- لا Heart icon.
- لا Profile link.
- Gate API أو اتركه dormant بلا استهلاك.
- افحص Program Details وCards لأي Wishlist button.

---

# 66. Consultations — Frontend

الحماية الحالية جيدة:

```text
FEATURES.consultations=false
Consultations layout → notFound()
```

تأكد من:

```env
NEXT_PUBLIC_FEATURE_CONSULTATIONS=false
```

ولا تعتمد على default فقط في Production؛ عرّف القيمة صراحة.

---

# 67. Consultations — Backend

تأكد:

```env
FEATURE_CONSULTATIONS=false
```

والRoutes لا تسجل.

---

# 68. Consultations — Admin

حتى لو Resource غير مسجل، `NeedsActionWidget` الحالي يحتوي:

```text
طلبات استشارة جديدة
```

يجب إخفاؤها في V1.

وأي:

```text
ConsultationsContentPage
ConsultationRequestResource
```

لا يسجل في Admin V1.

---

# 69. B2B

تأكد:

```env
NEXT_PUBLIC_FEATURE_BUSINESS=false
FEATURE_BUSINESS=false
```

لا يظهر:

```text
للشركات
حلول الجهات
أمثّل جهة أو فريق
تدريب الفرق
Corporate CTA
```

---

# 70. B2B Admin Widget

`NeedsActionWidget` يحتوي:

```text
حسابات أعمال (B2B) جديدة
```

هذا يجب إزالته من V1.

---

# 71. Wamadat Plus

تأكد:

```env
NEXT_PUBLIC_FEATURE_PLUS=false
FEATURE_PLUS=false
```

ويجب ألا يظهر:

```text
Plus Storefront
Services
Sellers
Buyer Orders
Seller Orders
Messages
Payouts
Disputes
Seller onboarding
Plus settings
```

---

# 72. Plus Scheduler

في:

```text
routes/console.php
```

حاليًا يعمل دائمًا:

```text
plus:process-sla
```

حتى لو Plus مغلق.

هذا ليس Exposure بصري فقط؛ هو نشاط خلفي لFeature مؤجلة.

المطلوب:

```text
Schedule Plus SLA only when FEATURE_PLUS=true
AND RELEASE_SCOPE != v1
```

مثال:

```php
if (config('features.plus') && ! config('release.v1')) {
    Schedule::command('plus:process-sla')->hourly()...
}
```

---

# 73. Reviews Scheduler

الحالي:

```text
emails:review-requests
```

يعمل يوميًا.

بما أن Reviews مؤجلة:

- لا Schedule في V1.
- لا ترسل Email "قيّم البرنامج".

---

# 74. Live Session Scheduler

الحالي:

```text
emails:live-session-reminders
```

يعمل كل 5 دقائق.

بما أن Live Sessions مؤجلة:

- لا Schedule في V1.

---

# 75. Marketing Automation Scheduler

الحالي:

```text
emails:recover-abandoned-carts
```

هذه Marketing Automation.

حسب Roadmap هي مؤجلة.

في V1:

- عطّل Schedule.
- لا ترسل Cart Recovery email.
- يمكن إبقاء Cart sync إذا له حاجة تشغيلية، لكن لا يستخدم للتسويق.

---

# 76. Attendance Cards Scheduler

الحالي:

```text
emails:attendance-cards
```

قرار التنفيذ:

- إذا Attendance Card جزء من V1 الحالي، أبقه.
- إذا يعتمد على Sessions/Live المؤجلة، عطله.

يجب اختبار الاعتمادية قبل القرار.

---

# 77. Cohort Start Reminders

هذه أقرب إلى Transactional Learning Notification.

يمكن إبقاؤها إذا:

- تستخدمها البرامج الحالية.
- لا تعتمد على Live Sessions.
- الرسالة لا تروج لFeature مؤجلة.

---

# 78. Newsletter API

المسارات الحالية عامة:

```text
POST /newsletter/subscribe
GET  /newsletter/unsubscribe
```

إذا Newsletter مؤجلة:

- Gate بـFeature flag.
- لا تعرض Signup.
- لا تقبل Subscribers جدد في V1.

---

# 79. Sitemap

الملف:

```text
app/sitemap.ts
```

حاليًا يضيف:

```text
/consultations
/plus
/plus/services
/for-business
```

ويفلتر عبر `isFeaturePathEnabled()`.

بعد توسيع الـFilter:

- يجب أن تبقى مستبعدة.

---

## أضف فحصًا صريحًا

تأكد أن Sitemap لا يحتوي:

```text
consultations
plus
for-business
redeem-gift
live-sessions
messages
achievements
subscriptions
bundles
affiliate
alumni
ai-assistant
```

Dashboard routes لا ينبغي أن تكون في Sitemap أصلًا.

---

# 80. Robots

راجع:

```text
app/robots.ts
```

حتى لو Route 404:

- Disallow لمسارات V2 المعروفة.
- `redeem-gift` موجود أصلًا ضمن disallow.
- أضف أي Public future surfaces عند الحاجة.

لكن:

> Robots ليس حماية أمنية ولا بديلًا عن Route Gate.

---

# 81. SEO Metadata

ابحث عن أي Metadata يذكر:

```text
Live Sessions
Community
Reviews
Consultations
Plus
B2B
Marketplace
Gifts
```

في Launch V1.

خصوصًا:

```text
site_description
program JSON-LD
AggregateRating
FAQ schema
OpenGraph descriptions
```

يجب تنظيفها.

---

# 82. Structured Data

Program Detail حاليًا قد يضيف:

```text
aggregateRating
reviewCount
```

أزلهما من V1.

FAQ schema يجب ألا يحتوي أسئلة عن Live.

---

# 83. Backend API — V1 Gate

القاعدة:

> Feature مؤجلة لا يجب أن تكون Public/Authenticated API نشطة بلا داعٍ أثناء V1.

أضف Conditional Route Registration للـFeatures التالية:

```text
reviews
program interest
gifts
direct messaging
gamification
live sessions
newsletter
push devices
calendar
```

الميزات الموجودة أصلًا خلف flags تبقى كذلك.

---

# 84. API Routes التي تبقى

لا تمس:

```text
Auth
Catalog V1
Search
Coupons
Checkout
Payment Methods
Invoices QR
Contact
Analytics الأساسية إذا مطلوبة
Learning
Notes
Lesson Q&A
Quizzes
Assignments
Orders
Bank Transfer
Certificates
Attendance V1
Notifications الأساسية
Support
Profile basic
Password
PDPL
```

---

# 85. Admin Panel — Resources

`AdminPanelProvider.php` الحالي يحتوي Allowlist جيدة:

```text
ProgramResource
LessonResource
StudentResource
EnrollmentResource
InstructorResource
QuizResource
AssignmentResource
OrderResource
PaymentResource
BankTransferResource
InvoiceResource
CouponResource
IssuedCertificateResource
SupportTicketResource
```

**احتفظ بهذه القائمة فقط.**

لا تضف:

```text
ReviewResource
GiftResource
ConsultationRequestResource
ProgramInterestResource
LiveSessionResource
Plus resources
Affiliate resources
B2B resources
Technical resources
```

---

# 86. Admin Dashboard Widgets

هذه نقطة مهمة؛ Resource قد يكون مخفيًا لكن Widget يعيد Feature إلى Dashboard.

`NeedsActionWidget` الحالي يحتوي V2/Technical entries.

أخفِ من V1:

```text
نزاعات ومضات بلس
طلبات سحب بلس
بائعون بانتظار الاعتماد
تقييمات بانتظار الإشراف
طلبات استشارة جديدة
حسابات أعمال B2B جديدة
Failed Jobs إذا قررنا إبقاء الواجهة التقنية مخفية عن العميل
```

---

## أبقِ فقط Queue Actions المناسبة لـV1

مثل:

```text
Bank Transfers
On-hold payments
Refunds إذا كانت عملية V1
Ungraded Assignments
Open Support Tickets
Draft Programs إذا مفيد
```

---

# 87. Admin Notifications

قد توجد Notifications قديمة خاصة بـ:

```text
Plus
Reviews
Consultations
Gifts
```

المطلوب:

- لا تنشئ Notifications جديدة لهذه Features في V1.
- لا تحتاج حذف History من DB.
- إذا القديمة ظاهرة ومربكة، فلترها من Admin Bell في V1.

---

# 88. Instructor Panel

`InstructorPanelProvider.php`

Launch V1 Resources يجب أن تصبح:

```text
ProgramResource
LessonResource
AssignmentResource
QuizResource
```

ولا تسجل:

```text
LiveSessionResource
```

الحضور يبقى عبر العلاقات/واجهة V1 المعتمدة.

---

# 89. Instructor Program Relations

كما سبق:

أخفِ:

```text
Discussions
Announcements
```

وأبقِ:

```text
Modules
Lessons
Trainees
Questions
Certificates
Attendance
```

---

# 90. Audio Content

وثيقة V1 الحالية تركز على:

```text
Video
PDF
Text
Resources
```

بينما السورس يدعم Audio.

لعدم كسر بيانات موجودة:

## السياسة الآمنة

- لا تروج لـAudio في Landing.
- نفذ Data Audit على البرامج المنشورة.
- إذا لا توجد Audio lessons مستخدمة في V1:
  - أخفِ Audio من Authoring الجديد.
- إذا توجد Audio lessons فعلية:
  - أبقِ Playback للLegacy content.
  - امنع إنشاء Audio جديد حتى قرار Phase لاحق.

لا تحذف المحتوى.

---

# 91. Live Program Modes

هناك Public filters على:

```text
mode=live
```

ويجب تنظيفها.

نفذ Audit:

```text
published programs where mode=live
```

إن كانت موجودة:

- لا تختفِ بصمت دون قرار.
- حدد ما إذا كانت ستتحول إلى self-paced / scheduled non-live.
- أو تؤجل من Public Launch.

---

# 92. CMS / Database Content Audit

لأن كثيرًا من النصوص تأتي من DB، Grep السورس وحده غير كافٍ.

افحص:

```text
site_settings.nav_items
site_settings.footer_links
home_content
home_cards
home_faqs
pages
landing_pages
announcement/promo settings
```

عن كلمات وروابط V2.

---

# 93. Forbidden Public Links Audit

ابحث في DB عن:

```text
/plus
/consultations
/for-business
/redeem-gift
/dashboard/live-sessions
/dashboard/messages
/trainer/messages
/dashboard/achievements
/subscriptions
/bundles
/affiliate
/alumni
/ai-assistant
```

أي Link موجود:

- يعطّل.
- أو يعدل إلى V1 destination.

---

# 94. Forbidden Copy Audit

ابحث عن كلمات مثل:

```text
الجلسات المباشرة
لقاءات مباشرة
البث المباشر
ومضات بلس
الاستشارات
للشركات
Marketplace
المجتمع
منتدى
التقييمات
أهدِ
هدية
سلسلة التعلم
الإنجازات
النشرة البريدية
SMS
ذكاء اصطناعي
اشتراكات
باقات
Affiliate
Alumni
```

لا تحذف الكلمات آليًا من DB.

راجع كل نتيجة يدويًا لأنها قد تظهر في:

- سياسة قديمة.
- وصف برنامج.
- محتوى تاريخي.

---

# 95. Search Source Audit — Frontend

بعد التنفيذ يجب تشغيل:

```bash
rg -n -i \
"live.sessions|الجلسات المباشرة|لقاءات مباشرة|/dashboard/messages|رسائل الطلاب|reviews|تقييمات|redeem-gift|GiftProgramButton|streak|achievements|ProgramInterestForm|consultations|ومضات بلس|for-business|affiliate|alumni|ai-assistant|newsletter" \
app components messages lib
```

أي نتيجة يجب تصنيفها:

```text
A) dormant implementation allowed
B) guarded by release scope
C) test only
D) still exposed — BUG
```

لا يجب أن تبقى نتيجة `D`.

---

# 96. Search Source Audit — Backend

شغل:

```bash
rg -n -i \
"LiveSessionResource|DiscussionsRelationManager|AnnouncementsRelationManager|InterestLeadsRelationManager|ReviewResource|GiftResource|ConsultationRequestResource|Plus|Affiliate|B2b|review-requests|live-session-reminders|recover-abandoned-carts" \
app routes config
```

كل نتيجة يجب أن تكون:

- dormant.
- gated.
- غير مسجلة في Panels.
- غير Scheduled في V1.

---

# 97. حماية من عودة Feature مستقبلًا

أضف Automated Tests.

## Frontend Test

يتأكد أن:

```text
isFeaturePathEnabled('/plus') = false
isFeaturePathEnabled('/consultations') = false
isFeaturePathEnabled('/for-business') = false
isFeaturePathEnabled('/redeem-gift') = false
isFeaturePathEnabled('/dashboard/live-sessions') = false
isFeaturePathEnabled('/dashboard/messages') = false
isFeaturePathEnabled('/dashboard/achievements') = false
isFeaturePathEnabled('/trainer/messages') = false
```

عندما:

```text
RELEASE_SCOPE=v1
```

---

# 98. Route E2E Tests

Direct GET يجب ألا يفتح:

```text
/ar/plus
/ar/consultations
/ar/for-business
/ar/redeem-gift
/ar/dashboard/live-sessions
/ar/dashboard/messages
/ar/dashboard/achievements
/ar/trainer/messages
```

المتوقع:

```text
404 / Not Found
```

وليس:

```text
200 hidden page
500
redirect loop
```

---

# 99. Navigation E2E

Student Nav يجب أن يحتوي فقط على V1.

Instructor Nav يجب أن يحتوي فقط على V1.

Public Navbar/Footer يجب ألا يحتوي V2.

اختبار النص وليس Href فقط.

---

# 100. Admin Panel Tests

تأكد أن Navigation لا يظهر:

```text
Live Sessions
Reviews
Gifts
Consultations
Program Interest
Plus
Affiliate
B2B
Community
Announcements
```

داخل Program edit أيضًا.

---

# 101. Instructor Panel Tests

لا يظهر:

```text
Live Sessions resource
Discussions
Announcements
Direct Messages
```

ويبقى:

```text
Programs
Lessons
Assignments
Quizzes
Students
Questions
Attendance
```

---

# 102. Scheduler Test

شغل:

```bash
php artisan schedule:list
```

في Production V1.

يجب ألا تظهر:

```text
plus:process-sla
emails:recover-abandoned-carts
emails:live-session-reminders
emails:review-requests
```

إذا تم اعتماد تعطيلها حسب هذه الخطة.

---

# 103. API Route Test

شغل:

```bash
php artisan route:list
```

في V1.

يجب ألا تظهر Routes المؤجلة التي اخترنا Gate لها.

خصوصًا:

```text
reviews
program interest
gifts
conversations
live-sessions
plus
consultations
affiliate
```

أو تكون محمية بـRelease middleware بحيث لا يمكن استخدامها.

الأفضل: **عدم تسجيلها في V1**.

---

# 104. Sitemap Test

افتح:

```text
/sitemap.xml
```

وابحث عن:

```text
plus
consultations
for-business
redeem-gift
live-sessions
messages
achievements
affiliate
alumni
```

النتيجة:

```text
0 matches
```

---

# 105. Homepage Copy Test

ابحث بصريًا في Landing عن:

```text
جلسات مباشرة
لقاءات مباشرة
استشارات
ومضات بلس
للشركات
مجتمع
أهدِ
تقييم
```

يجب ألا تظهر كميزات منتج Launch V1.

---

# 106. Program Page Test

صفحة أي برنامج V1:

لا يظهر:

```text
Reviews
Star rating
Review count
Review form
Gift button
Waitlist form
Live Session CTA
Community CTA
```

ويظهر:

```text
Program details
Instructor
Price
Seats/status
Cart/Checkout CTA
Curriculum
V1 trust information
```

---

# 107. Student Dashboard Test

لا يظهر:

```text
Live Sessions
Messages
Achievements
Streak
Wallet
Wishlist
Gifts
Reviews
Calendar advanced
```

ويظهر:

```text
Today V1
My Programs
Assignments
Orders
Certificates
Support
Notifications
Profile basic
Settings basic
```

---

# 108. Instructor Test

لا يظهر:

```text
Live Sessions
Messages
Discussions
Announcements
```

ويبقى Lesson Q&A.

---

# 109. Admin Test

لا يظهر أي Feature خارج Admin V1 Allowlist.

حتى داخل:

```text
Program edit tabs
Dashboard widgets
Notifications
Quick actions
```

---

# 110. ENV Production النهائي

Frontend:

```env
NEXT_PUBLIC_RELEASE_SCOPE=v1

NEXT_PUBLIC_FEATURE_PLUS=false
NEXT_PUBLIC_FEATURE_CONSULTATIONS=false
NEXT_PUBLIC_FEATURE_BUSINESS=false
NEXT_PUBLIC_FEATURE_SUBSCRIPTIONS=false
NEXT_PUBLIC_FEATURE_BUNDLES=false
NEXT_PUBLIC_FEATURE_AI_ASSISTANT=false
NEXT_PUBLIC_FEATURE_SMS=false
NEXT_PUBLIC_FEATURE_AFFILIATE=false
NEXT_PUBLIC_FEATURE_ALUMNI=false

NEXT_PUBLIC_FEATURE_DIRECT_MESSAGING=false
NEXT_PUBLIC_FEATURE_REVIEWS=false
NEXT_PUBLIC_FEATURE_GAMIFICATION=false
NEXT_PUBLIC_FEATURE_GIFTS=false
NEXT_PUBLIC_FEATURE_PROGRAM_INTEREST=false
NEXT_PUBLIC_FEATURE_LIVE_SESSIONS=false
NEXT_PUBLIC_FEATURE_COMMUNITY=false
NEXT_PUBLIC_FEATURE_NEWSLETTER=false
NEXT_PUBLIC_FEATURE_ADVANCED_PROFILE=false
NEXT_PUBLIC_FEATURE_ADVANCED_NOTIFICATION_SETTINGS=false
NEXT_PUBLIC_FEATURE_CALENDAR=false
NEXT_PUBLIC_FEATURE_WALLET=false
NEXT_PUBLIC_FEATURE_WISHLIST=false
```

---

Backend:

```env
RELEASE_SCOPE=v1

FEATURE_PLUS=false
FEATURE_CONSULTATIONS=false
FEATURE_BUSINESS=false
FEATURE_AI_ASSISTANT=false
FEATURE_SMS=false
FEATURE_AFFILIATES=false
FEATURE_ALUMNI=false

FEATURE_DIRECT_MESSAGING=false
FEATURE_REVIEWS=false
FEATURE_GAMIFICATION=false
FEATURE_GIFTS=false
FEATURE_PROGRAM_INTEREST=false
FEATURE_LIVE_SESSIONS=false
FEATURE_COMMUNITY=false
FEATURE_NEWSLETTER=false
FEATURE_MARKETING_AUTOMATION=false
FEATURE_PUSH=false
FEATURE_CALENDAR=false
```

> إذا اختير Model أقل في ENV، يجب أن تبقى النتيجة الوظيفية نفسها عبر `RELEASE_SCOPE=v1`.

---

# 111. لا تثق في ENV وحده

هذه الخطة تتطلب أربع طبقات:

```text
1. Release Scope
2. Feature Flags
3. UI / Navigation Filtering
4. Direct Route / Backend Gate
```

أي طبقة منفردة غير كافية.

---

# 112. ترتيب التنفيذ

## المرحلة A — Central Gates

1. إضافة Release Scope.
2. توسيع Frontend Feature Map.
3. إضافة Backend Feature Map.
4. إضافة blocked route map.
5. اختبارات الوحدة للـflags.

---

## المرحلة B — Public Website

1. Navbar.
2. Mega Menu Showcase.
3. Quick Pills.
4. Footer.
5. Newsletter.
6. Landing Copy.
7. Features.
8. How It Works.
9. FAQ.
10. Start Here.
11. Programs Preview.
12. Program Cards.
13. Instructor rating.
14. CMS dynamic links.
15. Announcement bars.

---

## المرحلة C — Program Details

1. Reviews fetch/UI.
2. Aggregate rating SEO.
3. Gift CTA.
4. Waitlist.
5. V2 copy.
6. Live indicators.

---

## المرحلة D — Student

1. Dashboard Nav.
2. Today page.
3. Messages.
4. Live Sessions.
5. Achievements.
6. Streak.
7. Wallet.
8. Wishlist.
9. Advanced Profile.
10. Marketing preferences.
11. Sessions UI.
12. Calendar UI.

---

## المرحلة E — Instructor

1. Trainer Nav.
2. Quick Actions.
3. Messages route.
4. Live Sessions entry.
5. Instructor Panel LiveSessionResource.
6. Program Discussions.
7. Announcements.
8. Live lesson authoring.

---

## المرحلة F — Admin

1. Preserve explicit resource allowlist.
2. Clean Program relation managers.
3. Clean Dashboard widgets.
4. Clean Admin notifications.
5. Remove V2 deep links.
6. Remove Live authoring options.

---

## المرحلة G — Backend Operations

1. Conditional routes.
2. Conditional scheduler entries.
3. Review emails OFF.
4. Live reminders OFF.
5. Plus SLA OFF.
6. Cart recovery OFF.
7. Newsletter intake OFF.
8. Push registration OFF if unused.

---

## المرحلة H — SEO / Discovery

1. Sitemap.
2. Robots.
3. Metadata.
4. JSON-LD.
5. FAQ schema.
6. CMS links.

---

## المرحلة I — Regression

1. Build.
2. Lint.
3. Tests.
4. Route list.
5. Schedule list.
6. Public grep audit.
7. Admin visual audit.
8. Student audit.
9. Instructor audit.
10. UAT Launch Gate.

---

# 113. فحوصات البناء المطلوبة

Frontend:

```bash
pnpm install --frozen-lockfile
pnpm lint
pnpm typecheck
pnpm build
```

استخدم الأوامر الفعلية الموجودة في `package.json` إذا اختلفت الأسماء.

لا تصلح Warnings خارج النطاق إلا إذا كانت تمنع البناء.

---

Backend:

```bash
composer install --no-interaction
php artisan optimize:clear
php artisan route:list
php artisan schedule:list
php artisan test
```

واستخدم Tests المستهدفة أولًا إذا كانت المجموعة الكاملة ثقيلة.

---

# 114. ممنوع أثناء المهمة

لا:

```text
تحذف Modules
تحذف migrations
تغير Database schema بلا حاجة
تمس Payment logic
تمس Enrollment logic
تمس Certificate logic
تمس Bank Transfer logic إلا Regression
تعمل dependency upgrade جماعي
تغير Framework
تقوم Refactor واسع
تحذف بيانات V2 من Production
```

---

# 115. حماية البيانات التاريخية

قد توجد بيانات سابقة لـ:

```text
Reviews
Gifts
Consultations
Live Sessions
Plus
Interest leads
```

المطلوب:

```text
Keep data
Stop new exposure
Stop new creation in V1
```

لا تعمل Purge.

---

# 116. حالة البرامج التي تعتمد على Feature مؤجلة

إذا كشف Audit أن برنامجًا منشورًا يعتمد على:

```text
Live Lesson
Live Session
Community requirement
Feature V2 أخرى
```

لا تخفِ المكون وتترك البرنامج مكسورًا.

يجب تسجيله كـ:

```text
V1 Content Migration Blocker
```

ثم اتخاذ أحد:

```text
Convert to V1 content
Unpublish program
Remove deferred requirement
Move program to future release
```

---

# 117. Definition of Done

تعتبر المهمة CLOSED فقط عندما:

- [ ] Landing لا تذكر أي Feature مؤجلة.
- [ ] Navbar لا يحتوي أي Feature مؤجلة.
- [ ] Mega Menu نظيف.
- [ ] Footer نظيف.
- [ ] Newsletter مخفي.
- [ ] Dynamic CMS links لا تعيد V2.
- [ ] Program pages لا تعرض Reviews/Gifts/Waitlist.
- [ ] SEO لا يحتوي Reviews/V2 claims.
- [ ] Student Dashboard V1 فقط.
- [ ] Today V1 فقط.
- [ ] Instructor UI V1 فقط.
- [ ] Admin UI V1 فقط.
- [ ] Program relation managers V1 فقط.
- [ ] Live type غير قابل للإنشاء في V1.
- [ ] Deferred direct URLs = 404.
- [ ] Deferred Backend routes غير مفعلة.
- [ ] Deferred schedulers غير فعالة.
- [ ] Sitemap نظيف.
- [ ] Production ENV مضبوط صراحة.
- [ ] Build ينجح.
- [ ] Lint/Typecheck ينجحان.
- [ ] Backend tests المستهدفة تمر.
- [ ] لا Regression على V1.
- [ ] UAT-SCOPE-002 من وثيقة UAT يمر بالكامل.

---

# 118. نتيجة Launch V1 بعد التصفية

## Public

```text
Wamadat
Programs
Categories
Search
Instructors
About
Contact
Help
Auth
```

---

## Commerce

```text
Cart
Coupons
Checkout
Payments
Bank Transfer
Orders
Invoices
```

---

## Learning

```text
My Programs
Units
Lessons
Video
PDF
Text
Resources
Notes
Lesson Q&A
Quiz
Assignment
Progress
Completion
Attendance V1
Certificates
Verification
```

---

## Operations

```text
Notifications الأساسية
Support
Instructor teaching tools
Admin V1 resources
Site Settings
```

ولا شيء آخر.

---

# 119. النتيجة المطلوبة للمستخدم

يجب أن يشعر الطالب أن ومضات اليوم هي:

> منصة واضحة لاكتشاف برنامج وشرائه ودراسته وإكمال تقييماته والحصول على شهادة موثقة.

لا يجب أن يرى:

```text
Marketplace
Live platform
Social network
Consulting product
Corporate SaaS
Gamification product
Gift marketplace
Subscription product
AI product
```

قبل موعد إطلاق هذه المنتجات رسميًا.

---

# 120. النتيجة المطلوبة للإدارة

يجب أن يرى Admin فقط ما يحتاجه لتشغيل V1 يوميًا:

```text
Programs
Lessons
Students
Enrollments
Instructors
Quizzes
Assignments
Orders
Payments
Bank Transfers
Invoices
Coupons
Certificates
Support
Settings
```

لا يجب أن يرى قوائم Features مستقبلية تعطي انطباعًا بأنها جاهزة أو مطلوبة الآن.

---

# 121. النتيجة المطلوبة للمدرب

يجب أن تتركز واجهته على:

```text
برامجي
الدروس
طلابي
الاختبارات
الواجبات
التصحيح
أسئلة الدروس
الحضور
```

ولا يرى:

```text
Live Sessions
Direct Messages
Community Discussions
Announcements advanced
```

في Launch V1.

---

# 122. رسالة الوكيل المختصرة

نفّذ هذه الوثيقة باعتبارها **Launch V1 Surface Lockdown**.

المطلوب ليس حذف ميزات V2، بل جعلها Dormant بالكامل:

```text
keep code
keep migrations
keep data
hide UI
block direct routes
gate backend routes
remove admin/instructor exposure
disable deferred schedulers
clean SEO/CMS
```

اعتمد V1 Allowlist كمصدر الحقيقة.

لا تعتبر المهمة مكتملة بمجرد اختفاء الروابط من Navbar؛ يجب إثبات الإخفاء عبر:

```text
Landing
Program pages
Student
Instructor
Admin
Direct URLs
API
Scheduler
Sitemap
SEO
CMS
```

ثم شغّل فحوصات البناء والـRegression وأخرج تقريرًا نهائيًا يحتوي:

```text
Files changed
Features hidden
Routes gated
Schedulers gated
Admin/Instructor surfaces removed
Frontend build result
Backend tests result
Remaining blockers = NONE
```

---

# 123. الخلاصة

هذا التعديل يحول السورس من:

> منصة تحتوي عشرات الوحدات الجاهزة أو شبه الجاهزة والمتداخلة في الواجهة

إلى:

> **Launch V1 واضح ومركز ومستقر، مع احتفاظ السورس بكل استثمارات V2 دون كشفها للمستخدم أو تحميل الإطلاق تكلفة اختبارها الآن.**

المبدأ النهائي:

```text
V1 visible
Future dormant
Code preserved
Data preserved
No accidental exposure
No surprise background behavior
```

وهذا هو الشكل الصحيح قبل بدء الـManual UAT النهائي وتسليم النسخة للعميل.
