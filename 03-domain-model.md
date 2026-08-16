# 03 — Domain Model

> **نمذجة الـ 18 Bounded Context.** لكلّ context: Aggregates، Entities، Value Objects، Domain Events، Invariants. هذا المستند يقود تصميم الـ Eloquent models والـ services في الـ Application layer.

> **مبادئ النمذجة:**
> - Aggregate Root هو نقطة الدخول الوحيدة لتعديل الـ aggregate.
> - Value Objects immutable، لها equality بالقيمة لا بالـ identity.
> - Domain Events تُصاغ بصيغة الماضي (`OrderPlaced`, `PaymentCaptured`).
> - Invariants تُحرَس داخل الـ aggregate (لا في الـ controller).

---

## 📑 الفهرس

01. [Tenancy](#01-tenancy) · 02. [Identity & Access](#02-identity--access) · 03. [Catalog](#03-catalog) · 04. [Learning](#04-learning) · 05. [Live Sessions](#05-live-sessions) · 06. [Attendance](#06-attendance) · 07. [Assessment](#07-assessment) · 08. [Certification](#08-certification) · 09. [Commerce](#09-commerce) · 10. [Billing & Subscriptions](#10-billing--subscriptions) · 11. [Affiliates](#11-affiliates) · 12. [Engagement](#12-engagement) · 13. [Communication](#13-communication) · 14. [Support / CRM](#14-support--crm) · 15. [Analytics](#15-analytics) · 16. [Marketing](#16-marketing) · 17. [Content (CMS)](#17-content-cms) · 18. [AI Assistant](#18-ai-assistant)

---

## 01. Tenancy

**Purpose**: إدارة المستأجرين، الفروع، خطط الاشتراك (Wamadat→Tenant).

### Aggregates
- **Tenant** *(root)* → Branches, TenantSettings, TenantSubscription
- **Plan** *(root)* → PlanFeatures (catalog of Wamadat plans)

### Value Objects
- `Subdomain` — يتحقّق من الـ format والـ uniqueness
- `TenantStatus` — enum: `provisioning`, `active`, `suspended`, `archived`, `deleted`
- `PlanQuotas` — حدود (max_learners, max_courses, max_storage_gb, ...)
- `BillingCycle` — `monthly`, `yearly`

### Domain Events
- `TenantProvisioned(tenant_id, plan_id, owner_user_id)`
- `TenantActivated(tenant_id)`
- `TenantSuspended(tenant_id, reason)`
- `TenantPlanChanged(tenant_id, from_plan, to_plan)`
- `BranchCreated(tenant_id, branch_id)`

### Invariants
- لا يمكن لمستأجر تجاوز `PlanQuotas` المتعلّقة بخطّته.
- Tenant `archived` للقراءة فقط — لا تعديلات.
- Subdomain فريد عبر كلّ المنصّة (case-insensitive).

---

## 02. Identity & Access

**Purpose**: المستخدمون، الأدوار، الصلاحيات، Authentication.

### Aggregates
- **User** *(root)* → UserCredentials, UserProfile, UserDevices, MfaSettings
- **Role** *(root)* → Permissions (RBAC)

### Value Objects
- `Email` — validated، normalized lowercase
- `PhoneNumber` — E.164 format (KSA validation)
- `HashedPassword` — Argon2id
- `MfaSecret` — encrypted TOTP secret
- `LocaleCode` — `ar`, `en`

### Domain Events
- `UserRegistered(user_id, email, tenant_id, source)`
- `UserEmailVerified(user_id)`
- `UserPhoneVerified(user_id)`
- `UserRoleAssigned(user_id, role)`
- `MfaEnabled(user_id, method)`
- `UserPasswordChanged(user_id)`
- `UserLockedOut(user_id, reason)`

### Invariants
- Email فريد ضمن الـ tenant (ليس عبر المنصّة).
- Super Admin له role في landlord DB، ليس في tenant schema.
- MFA إلزاميّ لكلّ من له role `Academy Owner`/`Finance`/`Super Admin`.

---

## 03. Catalog

**Purpose**: عرض البرامج للسوق، التصنيفات، صفحات المدرّبين.

### Aggregates
- **Program** *(root)* → ProgramSEO, PricingTiers, Prerequisites, LearningOutcomes
- **Category** *(root)* → CategoryHierarchy
- **InstructorProfile** *(root)* → Specializations, Awards, SocialLinks

### Value Objects
- `Money` — `{amount: int, currency: 'SAR'}` (cents-based)
- `Slug` — URL-safe, unique within tenant
- `Duration` — `{minutes: int}`
- `Level` — `beginner`, `intermediate`, `advanced`
- `ProgramStatus` — `draft`, `pending_review`, `published`, `archived`

### Domain Events
- `ProgramDrafted(program_id, instructor_id)`
- `ProgramSubmittedForReview(program_id)`
- `ProgramPublished(program_id)`
- `ProgramArchived(program_id, reason)`
- `ProgramPriceChanged(program_id, old, new)`

### Invariants
- لا يُنشَر برنامج بدون: title، description، instructor، at least 1 module، price.
- `Money.amount` لا يكون سالباً.
- `Program.slug` فريد ضمن الـ tenant.

---

## 04. Learning

**Purpose**: الدروس، الفصول، التقدّم، Drip release، Resources.

### Aggregates
- **Course** *(root)* → Modules → Lessons → Resources
- **Enrollment** *(root)* → LessonProgress, Certificates link
- **CohortBatch** *(root)* → CohortSchedule, CohortMembers (for cohort-based)

### Value Objects
- `LessonType` — `video`, `text`, `pdf`, `audio`, `live`, `quiz`
- `Progress` — `{completed_lessons: int, total: int, percent: float}`
- `DripRule` — `{type: 'days_after_enroll'|'date'|'after_lesson', value: ...}`
- `VideoSource` — `{provider: 'bunny', stream_id: string, duration_sec: int}`

### Domain Events
- `LearnerEnrolled(enrollment_id, course_id, user_id, source)`
- `LessonStarted(enrollment_id, lesson_id, at)`
- `LessonCompleted(enrollment_id, lesson_id, at, duration)`
- `CourseCompleted(enrollment_id, course_id, at)`
- `DripContentUnlocked(enrollment_id, lesson_id)`

### Invariants
- Enrollment فريد لكلّ (user, course) — لا تكرار.
- Lesson لا تُحسب مكتملة إلا بـ watched_percent >= 90% (للفيديو) أو attempt successful (للـ quiz).
- Drip rules تُحترَم: lesson مغلقة لا تُفتَح إلا بعد شرطها.

---

## 05. Live Sessions

**Purpose**: الجلسات المباشرة، التسجيلات، Polls، Chat.

### Aggregates
- **LiveSession** *(root)* → Polls, ChatMessages, Recordings, SessionParticipants

### Value Objects
- `SessionStatus` — `scheduled`, `live`, `ended`, `cancelled`
- `Room100msId` — معرّف الغرفة في 100ms
- `RecordingUrl` — مع expiry للـ signed URL

### Domain Events
- `LiveSessionScheduled(session_id, starts_at)`
- `LiveSessionStarted(session_id, at)`
- `LiveSessionEnded(session_id, at, duration_min)`
- `LiveSessionRecordingReady(session_id, recording_url)`
- `ParticipantJoined(session_id, user_id, at)`
- `ParticipantLeft(session_id, user_id, at, duration_min)`

### Invariants
- لا يمكن تشغيل session قبل `starts_at - 15min`.
- `ended_at` > `started_at`.
- التسجيل لا يُحفَظ إلا لـ tenants على Plan ≥ Pro.

---

## 06. Attendance

**Purpose**: الحضور (Live + In-person)، QR check-in.

### Aggregates
- **AttendanceRecord** *(root)* → CheckInEvents
- **QrCode** *(root)* — auto-generated per session, expires

### Value Objects
- `CheckInMethod` — `qr_scan`, `manual_admin`, `auto_live` (presence detection)
- `AttendanceStatus` — `present`, `late`, `absent`, `excused`
- `QrToken` — short-lived JWT (5min validity)

### Domain Events
- `CheckedIn(record_id, user_id, session_id, method, at)`
- `MarkedAbsent(record_id, user_id, session_id, by)`
- `AttendanceCorrected(record_id, by, reason)`

### Invariants
- لا يمكن check-in قبل `session.starts_at - 30min` أو بعد `session.ends_at + 30min`.
- QR token صالح مرّة واحدة، لمدة 5 دقائق.
- Manual override يتطلّب role `Instructor`/`Support`/`Owner`.

---

## 07. Assessment

**Purpose**: الاختبارات، الواجبات، التقييم، Grading.

### Aggregates
- **Quiz** *(root)* → Questions → Choices, ScoringRules
- **Submission** *(root)* → Answers, Grade, GradingHistory
- **Assignment** *(root)* → AssignmentSubmission → Feedback, Files

### Value Objects
- `QuestionType` — `mcq_single`, `mcq_multi`, `true_false`, `short_answer`, `essay`, `file_upload`
- `Score` — `{points: float, max: float, percent: float}`
- `GradingRubric` — array of criteria with weights

### Domain Events
- `QuizPublished(quiz_id, course_id)`
- `QuizAttemptStarted(attempt_id, user_id, quiz_id)`
- `QuizAttemptSubmitted(attempt_id, score)`
- `QuizAttemptGraded(attempt_id, final_score, graded_by)`
- `AssignmentSubmitted(submission_id, user_id, assignment_id)`
- `AssignmentGraded(submission_id, score, feedback)`

### Invariants
- المحاولات (attempts) لا تتجاوز `quiz.max_attempts`.
- لا تُعدَّل الإجابات بعد submission.
- درجة الـ MCQ تُحسَب تلقائياً، الـ essay يدوياً.

---

## 08. Certification

**Purpose**: توليد الشهادات، التحقّق العامّ، Templates.

### Aggregates
- **CertificateTemplate** *(root)* → DesignFields, Variables
- **IssuedCertificate** *(root)* → VerificationCode, AuditTrail

### Value Objects
- `CertificateNumber` — `{prefix: 'WMD', year: 2026, serial: 0000001}` → `WMD-2026-0000001`
- `VerificationCode` — UUID v4، public-facing
- `IssuedAt`, `ExpiresAt` (optional)

### Domain Events
- `CertificateIssued(certificate_id, user_id, course_id, number)`
- `CertificateRevoked(certificate_id, reason, by)`
- `CertificateVerified(verification_code, by_ip, at)` (audit)

### Invariants
- شهادة تُصدَر فقط بعد `CourseCompleted` لذلك الـ enrollment.
- `CertificateNumber` فريد عبر المنصّة بأكملها.
- شهادة `revoked` تظهر للجمهور كـ "ملغاة" مع سبب.

---

## 09. Commerce

**Purpose**: السلّة، الطلبات، الكوبونات، Refunds.

### Aggregates
- **Cart** *(root)* → CartItems
- **Order** *(root)* → OrderItems, Payments, AppliedCoupons, OrderHistory
- **Coupon** *(root)* → CouponRules, RedemptionLog
- **Refund** *(root)* → RefundReason, ApprovedBy

### Value Objects
- `Money`, `Discount` (`{type: 'percent'|'fixed', value: int}`)
- `OrderStatus` — `pending`, `awaiting_payment`, `paid`, `processing`, `completed`, `cancelled`, `refunded`
- `PaymentMethod` — `card`, `apple_pay`, `mada`, `tamara`, `bank_transfer`

### Domain Events
- `CartItemAdded(cart_id, item)`
- `OrderPlaced(order_id, user_id, total)`
- `PaymentInitiated(order_id, gateway, amount)`
- `PaymentCaptured(order_id, payment_id, amount)`
- `PaymentFailed(order_id, gateway, reason)`
- `OrderCompleted(order_id)` *(critical event — Learning/Billing/Comm/Affiliates listen)*
- `OrderCancelled(order_id, reason)`
- `RefundIssued(refund_id, order_id, amount)`

### Invariants
- Order total = sum(items) - discounts + tax. لا يُسمَح بـ off-by-fils.
- لا يكتمل `OrderCompleted` إلا بـ `Payment.captured`.
- Coupon لا يُطبَّق مرّتين على نفس الـ order.
- Refund لا يتجاوز `Payment.captured_amount`.

---

## 10. Billing & Subscriptions

**Purpose**: الاشتراكات المتجدّدة، الفواتير، ZATCA.

### Aggregates
- **Subscription** *(root)* → BillingCycles, Renewals, GracePeriod
- **Invoice** *(root)* → InvoiceItems, ZatcaSubmission, PdfLocation

### Value Objects
- `BillingInterval` — `monthly`, `yearly`
- `SubscriptionStatus` — `trialing`, `active`, `past_due`, `cancelled`, `expired`
- `TaxRate` — `{rate: 0.15, jurisdiction: 'KSA'}`
- `ZatcaUuid` — invoice UUID from ZATCA
- `InvoiceNumber` — sequential per tenant per year

### Domain Events
- `SubscriptionCreated(subscription_id, user_id, plan)`
- `SubscriptionRenewed(subscription_id, period)`
- `SubscriptionPaymentFailed(subscription_id, attempt)`
- `SubscriptionCancelled(subscription_id, effective_at)`
- `InvoiceGenerated(invoice_id, amount, due_at)`
- `InvoiceSubmittedToZatca(invoice_id, zatca_uuid)`
- `InvoicePaid(invoice_id)`

### Invariants
- Subscription `past_due` يحصل على grace period 3 أيام قبل `expired`.
- كلّ فاتورة B2C >= 1000 SAR تتطلّب ZATCA submission.
- InvoiceNumber لا يُعاد استخدامه أبداً.

---

## 11. Affiliates

**Purpose**: برنامج التسويق بالعمولة.

### Aggregates
- **AffiliatePartner** *(root)* → ReferralLinks, PayoutMethod, BankDetails
- **Attribution** *(root)* → ClickLog, AttributedOrder
- **Payout** *(root)* → PayoutLineItems, PayoutStatus

### Value Objects
- `CommissionRate` — `{percent: float}` أو `{fixed: Money}`
- `AttributionWindow` — TTL by days (default 30)
- `ReferralCode` — short URL token (6-8 chars)

### Domain Events
- `AffiliateApplied(partner_id)`
- `AffiliateApproved(partner_id)`
- `ReferralClickRecorded(code, ip, ua, at)`
- `SaleAttributed(order_id, partner_id, commission_amount)`
- `PayoutRequested(partner_id, amount)`
- `PayoutCompleted(payout_id, method, reference)`

### Invariants
- نسبة العمولة لا تتجاوز config tenant (default 30%).
- Attribution يستخدم last-touch within `AttributionWindow`.
- Payout لا يصرف إلا بعد `MinimumPayoutAmount` (default 200 SAR).

---

## 12. Engagement

**Purpose**: النقاشات، التعليقات، المراجعات، Reactions.

### Aggregates
- **Discussion** *(root)* → Replies (threaded), Reactions
- **Review** *(root)* → ReviewResponse (instructor)
- **Comment** *(root)* → CommentVotes

### Value Objects
- `Rating` — int 1–5
- `Reaction` — `like`, `helpful`, `insightful`, `disagree`
- `ContentSeverity` — for moderation: `safe`, `flagged`, `removed`

### Domain Events
- `DiscussionStarted(discussion_id, course_id, by_user)`
- `ReplyPosted(reply_id, discussion_id)`
- `ReviewPublished(review_id, course_id, rating)`
- `CommentFlagged(comment_id, by_user, reason)`
- `CommentRemoved(comment_id, by_moderator, reason)`

### Invariants
- Review مسموح فقط للمتعلّمين الذين أكملوا >= 50% من الكورس.
- Comment على درس مغلق (لم يفتح drip) غير مسموح.
- Auto-moderation: AI flag > threshold → status `flagged` للمراجعة.

---

## 13. Communication

**Purpose**: البريد، SMS، WhatsApp، Push، In-app notifications.

### Aggregates
- **NotificationTemplate** *(root)* → TranslationsByLocale, Variables
- **Notification** *(root)* → DeliveryAttempts, OpenedAt, ClickedLinks

### Value Objects
- `Channel` — `email`, `sms`, `whatsapp`, `push`, `in_app`
- `DeliveryStatus` — `queued`, `sent`, `delivered`, `failed`, `bounced`
- `NotificationCategory` — `transactional`, `marketing`, `system`

### Domain Events
- `NotificationQueued(notification_id, channel, recipient)`
- `NotificationSent(notification_id, provider_id)`
- `NotificationDelivered(notification_id, at)`
- `NotificationOpened(notification_id, at)`
- `NotificationClicked(notification_id, link, at)`
- `NotificationFailed(notification_id, error)`

### Invariants
- Marketing notifications تحترم `user.marketing_opt_in`.
- Transactional لا تحترم opt-in (إلزامية).
- Rate limit: max 5 marketing emails per week per user.
- Quiet hours: 22:00–07:00 KSA لـ SMS/WhatsApp (إلا critical).

---

## 14. Support / CRM

**Purpose**: التذاكر، الـ pipelines، KB.

### Aggregates
- **Ticket** *(root)* → Messages, Attachments, StatusHistory, AssignedAgent
- **Pipeline** *(root)* → Stages, Cards (deals)
- **KbArticle** *(root)* → Translations, Helpfulness

### Value Objects
- `TicketStatus` — `new`, `open`, `pending_user`, `pending_agent`, `resolved`, `closed`
- `Priority` — `low`, `medium`, `high`, `urgent`
- `CsatScore` — 1–5

### Domain Events
- `TicketCreated(ticket_id, by_user, subject)`
- `TicketAssigned(ticket_id, agent_id)`
- `TicketMessagePosted(ticket_id, message_id, by)`
- `TicketResolved(ticket_id, by)`
- `CsatSubmitted(ticket_id, score, comment)`
- `KbArticleHelpfulVoted(article_id, vote)`

### Invariants
- Ticket في `urgent` priority يُعَيَّن agent خلال 15 دقيقة (SLA).
- CSAT يُطلب بعد resolved مرّة واحدة فقط.

---

## 15. Analytics

**Purpose**: التقارير، Dashboards، Aggregations.

### Aggregates
- **MetricSnapshot** *(root)* — pre-aggregated KPIs daily
- **SavedReport** *(root)* — user-defined queries
- **EventLog** — append-only stream of all domain events (للـ analytical replay)

### Value Objects
- `TimeWindow` — `{from: date, to: date, granularity: 'day'|'week'|'month'}`
- `MetricName` — enum of all defined KPIs
- `Dimension` — `tenant`, `instructor`, `course`, `category`, `country`

### Domain Events
- `MetricSnapshotComputed(metric, window, value)`
- `ReportSaved(report_id, user_id)`
- `ReportExported(report_id, format)`

### Invariants
- Read-only context (لا يعدّل بيانات أخرى).
- يَستهلك events من جميع الـ contexts عبر event log.
- Snapshots تحسب خلال nightly job (3 AM KSA).

---

## 16. Marketing

**Purpose**: الحملات، Coupons، Email sequences، Abandoned cart.

### Aggregates
- **Campaign** *(root)* → Audiences, Schedule, Variants (A/B), Performance
- **EmailSequence** *(root)* → Steps, TriggerConditions
- **AbandonedCartRecovery** *(root)* → RecoveryAttempts

### Value Objects
- `CampaignStatus` — `draft`, `scheduled`, `running`, `paused`, `completed`
- `AudienceSegment` — `{filters: [...]}` (compiled to SQL)
- `ConversionGoal` — `signup`, `purchase`, `enrollment`

### Domain Events
- `CampaignLaunched(campaign_id)`
- `CampaignTargetReached(campaign_id, sent_count)`
- `EmailSequenceStarted(sequence_id, user_id)`
- `EmailSequenceStepFired(sequence_id, user_id, step)`
- `AbandonedCartDetected(cart_id, user_id, value)`
- `CartRecovered(cart_id, by_attempt)`

### Invariants
- Email sequence step لا يُكرَّر لنفس المستخدم.
- Campaign على audience > 10k يتطلّب approval من Owner.

---

## 17. Content (CMS)

**Purpose**: المدوّنة، الصفحات الثابتة، البودكاست، FAQ، Banners.

### Aggregates
- **Post** *(root)* → Tags, SEO, Translations, CommentsAllowed
- **Page** *(root)* → SeoFields, Variants
- **PodcastEpisode** *(root)* → AudioSource, Transcript, ShowNotes
- **FaqEntry** *(root)* → Category

### Value Objects
- `PublishStatus` — `draft`, `scheduled`, `published`, `archived`
- `SeoMetadata` — title, description, og_image, canonical
- `ContentLocale` — `ar`, `en`

### Domain Events
- `PostPublished(post_id)`, `PostScheduled(post_id, at)`
- `PodcastEpisodeReleased(episode_id)`
- `FaqUpdated(faq_id, by)`

### Invariants
- Slug فريد ضمن النوع داخل الـ tenant.
- Published post لا يُعدَّل title/slug بدون redirect.

---

## 18. AI Assistant

**Purpose**: المساعد الذكي، التوصيات، Embeddings.

### Aggregates
- **Conversation** *(root)* → Messages, UsedTools
- **Recommendation** *(root)* — cached top-N for user
- **EmbeddingRecord** *(root)* — per content piece

### Value Objects
- `MessageRole` — `user`, `assistant`, `tool`, `system`
- `TokenUsage` — `{prompt: int, completion: int, cost_usd: float}`
- `VectorDimension` — 1536 (OpenAI text-embedding-3-small)

### Domain Events
- `ConversationStarted(conversation_id, user_id)`
- `AiMessageGenerated(message_id, tokens, model)`
- `RecommendationsRefreshed(user_id, count)`
- `EmbeddingGenerated(content_type, content_id)`

### Invariants
- Token usage لكلّ مستخدم لا يتجاوز quota الـ plan (50/day Free، 200/day Pro).
- Embedding يُحدَّث عند تغيير المحتوى المصدر.
- المحادثات بعد 30 يوماً تُؤرشَف.

---

## 📌 التالي

- [`04-database-schema.md`](04-database-schema.md) — تحويل هذه الـ aggregates إلى جداول PostgreSQL.
- [`05-api-design.md`](05-api-design.md) — كيف تظهر هذه الـ domain events كـ API endpoints.

---

<sub>**النسخة**: 1.0 · **عدد الـ Aggregates المحدَّدة**: ~50 · **عدد Domain Events**: ~90</sub>
