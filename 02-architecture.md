# 02 — البنية المعمارية (Architecture)

> **مرجع تقنيّ شامل.** كلّ ADR في هذا المستند **ملزم** — التعديل يتطلّب فتح ADR جديد. هذا المستند يُجيب على: "ما الذي نبنيه؟ ولماذا اخترنا كلّ خيار؟"

> الصياغة استراتيجية، التفاصيل المعمّقة موزَّعة على:
> - بنية البيانات → [`04-database-schema.md`](04-database-schema.md)
> - تصميم الـ APIs → [`05-api-design.md`](05-api-design.md)
> - الأمان → [`06-security.md`](06-security.md)
> - النشر → [`07-deployment.md`](07-deployment.md)

---

## 📑 الفهرس

1. [الملخّص التنفيذي](#1-الملخص-التنفيذي)
2. [نمط المعمارية](#2-نمط-المعمارية)
3. [C4 — السياق العام (System Context)](#3-c4--السياق-العام)
4. [C4 — الحاويات (Containers)](#4-c4--الحاويات)
5. [الطبقات الأربع (Layered Architecture)](#5-الطبقات-الأربع)
6. [الـ 18 Bounded Contexts](#6-الـ-18-bounded-contexts)
7. [التواصل بين الـ Contexts](#7-التواصل-بين-الـ-contexts)
8. [Multi-tenancy](#8-multi-tenancy)
9. [Identity & Access — RBAC + ABAC](#9-identity--access)
10. [Payment Gateway Abstraction](#10-payment-gateway-abstraction)
11. [Video & Live Sessions](#11-video--live-sessions)
12. [AI Microservice](#12-ai-microservice)
13. [Search Strategy](#13-search-strategy)
14. [Caching Strategy](#14-caching-strategy)
15. [Background Jobs](#15-background-jobs)
16. [Real-time Layer](#16-real-time-layer)
17. [Storage & CDN](#17-storage--cdn)
18. [Observability](#18-observability)
19. [Scalability Plan](#19-scalability-plan)
20. [ADR Index](#20-adr-index)

---

## 1. الملخّص التنفيذي

**ومضات أكاديمي** منصّة LMS متعدّدة المستأجرين، مبنيّة بـ **Modular Monolith** على Laravel 11، مع microservice واحد بـ Python/FastAPI للذكاء الاصطناعي، وطبقة Real-time بـ Laravel Reverb، وواجهة Next.js 15 منفصلة.

نتبنّى نهج **Pragmatic Microservices**: نبدأ مونوليث، نَفصِل خدمة عند ظهور حاجة واضحة (الأداء، فريق مستقلّ، تكنولوجيا مختلفة). الذكاء الاصطناعي مفصول من اليوم الأوّل لأنّه Python-only ولا فائدة من حشره في PHP.

| الجانب | الخيار | المبرّر |
|---|---|---|
| **النمط الأساسي** | Modular Monolith | تعقيد منخفض، نشر بسيط، تطوّر تدريجي |
| **Backend** | Laravel 11 + Filament 3 (PHP 8.3+) | pool المطوّرين، ecosystem ناضج، Admin جاهز |
| **Frontend** | Next.js 15 + TypeScript 5.5 | RSC للأداء، RTL، SEO |
| **Database** | PostgreSQL 16 + pgvector | موثوقية، JSONB، Vector search في نفس المحرّك |
| **Cache/Queue** | Redis 7 (Cluster mode للإنتاج) | متعدّد الأغراض، إدارة موحَّدة |
| **Search** | Meilisearch | بحث عربيّ ممتاز، خفيف |
| **Live** | **100ms** (انظر §11) | White-label، MENA latency، تكلفة معقولة |
| **Recorded Video** | Bunny Stream | DRM، adaptive bitrate، PoP سعودي |
| **AI** | FastAPI + Claude API + pgvector | Python للـ ML، Claude للعربية |
| **Tenancy** | Schema-per-tenant (Spatie) | عزل قوي، نسخ احتياطي مرن |
| **Hosting** | Railway (BE) + Vercel (FE) + Bunny (CDN/Stream) | DX ممتاز، تكلفة معقولة، PoP سعودي |

---

## 2. نمط المعمارية

### 2.1 الاختيار: Modular Monolith + خدمة مساعدة

```
┌────────────────────────────────────────────────────────────┐
│                    Wamadat Platform                         │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │       Laravel Monolith (PHP 8.3, Octane)             │  │
│  │                                                       │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │  │
│  │  │ Tenancy │ │ Identity│ │ Catalog │ │Learning │ ...│  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘    │  │
│  │                  18 Bounded Contexts                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↕ HTTP / Events                    │
│  ┌──────────────────────┐  ┌───────────────────────────┐   │
│  │  AI Service (Python) │  │  Reverb (Laravel WebSock.)│   │
│  │  FastAPI + Claude    │  │  Live presence, chat      │   │
│  └──────────────────────┘  └───────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### 2.2 لماذا Modular Monolith (وليس Microservices من اليوم الأوّل)

| العامل | Monolith | Microservices |
|---|---|---|
| تعقيد البنية التحتية | ✅ منخفض | ❌ عالٍ جداً (k8s, service mesh, observability) |
| سرعة التطوير الأولى | ✅ 2–3× أسرع | ❌ بطيء، كلّ تغيير عابر للخدمات |
| المعاملات (Transactions) | ✅ ACID native | ❌ saga pattern مُتطلَّب |
| فريق صغير (1–5 مطوّرين) | ✅ مناسب | ❌ مُهلِك |
| توسّع لـ 1M user | ✅ ممكن (مع caching/scaling) | ✅ متفوّق |
| فصل خدمة لاحقاً | ✅ سهل لو modules نظيفة | — |

**ADR-009**: نبدأ Modular Monolith. نَفصِل بوّابة عند ظهور حاجة قياسية (RPS عالية على module معيّن، فريق مستقلّ، technology mismatch).

### 2.3 الخدمات المفصولة من اليوم الأوّل

| الخدمة | السبب |
|---|---|
| **AI Service (Python)** | Python-only ecosystem لـ ML/embeddings. حشرها في PHP خسارة. |
| **Reverb WebSockets** | عملية طويلة العمر، تحتاج connection persistence مختلفة عن HTTP. |

---

## 3. C4 — السياق العام (System Context)

```
                    ┌──────────────────┐
                    │   Tenant Admin   │ (Filament Panel)
                    └────────┬─────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
   ┌────────▼────────┐   ┌───▼────┐   ┌──────▼──────┐
   │    Learner      │   │Instructor│   │ Super Admin │
   │  (Web + PWA)    │   │  (Web)  │   │  (Filament) │
   └────────┬────────┘   └───┬─────┘   └──────┬──────┘
            │                │                 │
            └────────────────┼─────────────────┘
                             ▼
                    ╔════════════════╗
                    ║                ║
                    ║    WAMADAT     ║
                    ║   PLATFORM     ║
                    ║                ║
                    ╚═══════╤════════╝
                            │
   ┌─────────┬──────────────┼──────────────┬─────────────┐
   ▼         ▼              ▼              ▼             ▼
┌──────┐ ┌──────┐    ┌─────────────┐  ┌───────┐   ┌──────────┐
│ Tap  │ │Tamara│    │ Bunny.net   │  │ 100ms │   │  Resend  │
│Pay   │ │Pay   │    │ Stream/CDN/ │  │ Live  │   │  Email   │
│      │ │      │    │  Storage    │  │  SDK  │   │          │
└──────┘ └──────┘    └─────────────┘  └───────┘   └──────────┘
                                                  
   ┌─────────┬──────────────┬──────────────┬─────────────┐
   ▼         ▼              ▼              ▼             ▼
┌──────┐ ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐
│Unifo-│ │ WhatsApp │  │ Claude   │  │ Sentry   │  │ ZATCA  │
│ nic  │ │   API    │  │   API    │  │  / BL    │  │ E-inv. │
│ SMS  │ │          │  │  (AI)    │  │          │  │        │
└──────┘ └──────────┘  └──────────┘  └──────────┘  └────────┘
```

### الجهات الخارجية

| النوع | الجهة | الغرض |
|---|---|---|
| **Payment** | Tap, Tamara, Mada, Apple Pay | معالجة الدفع |
| **Media** | Bunny.net (Stream + CDN + Storage) | فيديو، CDN، تخزين |
| **Live** | 100ms | جلسات مباشرة |
| **Messaging** | Resend, Unifonic, WhatsApp Cloud API | بريد، SMS، رسائل |
| **AI** | Anthropic Claude, OpenAI Embeddings | المساعد، التضمينات |
| **Observability** | Sentry, Better Stack | Errors, Logs, Uptime |
| **Compliance** | ZATCA Phase 2 | الفوترة الإلكترونية |

---

## 4. C4 — الحاويات (Containers)

```
┌──────────────────────────────────────────────────────────────┐
│                    Vercel (Frontend Edge)                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │    Next.js 15 App Router (RSC + Client Components)     │  │
│  │    - Tenant subdomain routing                           │  │
│  │    - i18n (ar/en), RTL                                  │  │
│  │    - shadcn/ui + Tailwind 4                             │  │
│  └────────────────────────────────────────────────────────┘  │
└────────────────────────┬─────────────────────────────────────┘
                         │ HTTPS (REST + GraphQL where needed)
                         │ WebSocket (subscriptions)
┌────────────────────────▼─────────────────────────────────────┐
│           Railway/Fly.io (Backend, KSA region preferred)      │
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │           Laravel 11 + Octane (Swoole)               │    │
│  │  ┌──────────────────┐  ┌──────────────────────────┐  │    │
│  │  │  HTTP API Layer  │  │  Filament Admin Panels   │  │    │
│  │  │  (Sanctum tokens)│  │  (8 dashboards by role)  │  │    │
│  │  └──────────────────┘  └──────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────┐    │    │
│  │  │     18 Bounded Context Modules                │    │    │
│  │  └──────────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  ┌─────────────────────┐  ┌────────────────────────────┐    │
│  │  Horizon (Queues)   │  │  Reverb (WebSocket Server) │    │
│  │  on Redis           │  │  ports broadcasting        │    │
│  └─────────────────────┘  └────────────────────────────┘    │
│                                                                │
│  ┌────────────────────────────────────────────────────┐      │
│  │      AI Service (Python FastAPI sidecar)           │      │
│  │      pgvector queries + Claude calls + caching     │      │
│  └────────────────────────────────────────────────────┘      │
└─────────┬──────────────────┬────────────────┬─────────────────┘
          │                  │                │
   ┌──────▼──────┐    ┌──────▼──────┐  ┌─────▼─────┐
   │ PostgreSQL  │    │   Redis 7   │  │Meilisearch│
   │ 16 + pgvec  │    │  Cluster    │  │           │
   │ (Tenant DB) │    │  cache/queue│  │ ar-tuned  │
   └─────────────┘    └─────────────┘  └───────────┘
```

---

## 5. الطبقات الأربع (Layered Architecture)

داخل كلّ Bounded Context نطبّق نفس المعمارية:

```
┌─────────────────────────────────────────────────────────┐
│  📡 Presentation Layer                                  │
│     HTTP Controllers · Filament Resources · GraphQL     │
│     WebSocket Channels · Console Commands               │
└────────────────────────┬────────────────────────────────┘
                         │ DTOs (input/output)
┌────────────────────────▼────────────────────────────────┐
│  🎯 Application Layer                                   │
│     Use Cases (single-purpose handlers)                 │
│     Application Services (orchestration)                │
│     Event Dispatchers · Saga coordinators (later)       │
└────────────────────────┬────────────────────────────────┘
                         │ Domain calls
┌────────────────────────▼────────────────────────────────┐
│  🧠 Domain Layer                                        │
│     Entities · Value Objects · Aggregates               │
│     Domain Services · Domain Events                     │
│     Specifications (business rules)                     │
└────────────────────────┬────────────────────────────────┘
                         │ Repository interfaces
┌────────────────────────▼────────────────────────────────┐
│  🔧 Infrastructure Layer                                │
│     Eloquent Repositories · External API Clients        │
│     Cache Adapters · Queue Adapters · Storage Adapters  │
└─────────────────────────────────────────────────────────┘
```

### قواعد العبور بين الطبقات

| من | إلى | مسموح؟ |
|---|---|---|
| Presentation | Application | ✅ |
| Application | Domain | ✅ |
| Application | Infrastructure (عبر interfaces) | ✅ |
| Domain | Infrastructure | ❌ (Domain لا يعرف Eloquent) |
| Infrastructure | Domain | ✅ (implements domain interfaces) |
| Bounded Context A → Context B (direct) | ❌ ممنوع — تواصُل عبر Events فقط |

---

## 6. الـ 18 Bounded Contexts

| # | الـ Context | المسؤولية الجوهرية | يَملك (Owns) |
|---|---|---|---|
| 01 | **Tenancy** | المستأجرون، الفروع، خطط الاشتراك، الـ subdomain routing | Tenants, Branches, Plans, Subscriptions (Wamadat→Tenant) |
| 02 | **Identity & Access** | المستخدمون، الأدوار، الصلاحيات، Authentication | Users, Roles, Permissions, Sessions, Tokens |
| 03 | **Catalog** | البرامج، التصنيفات، صفحات المدرّبين، عرض السوق | Programs (catalog view), Categories, Tags, Instructors profile |
| 04 | **Learning** | الدروس، الفصول، التقدّم، الـ drip release | Modules, Lessons, Progress, Resources |
| 05 | **Live Sessions** | الجلسات المباشرة، Zoom/100ms integration | Live Sessions, Recordings, Polls, Chat |
| 06 | **Attendance** | تسجيل الحضور (Live + In-person)، QR check-in | Attendance records, QR codes, Check-in events |
| 07 | **Assessment** | الاختبارات، الواجبات، التقييم، Auto-grading | Quizzes, Questions, Submissions, Grades |
| 08 | **Certification** | توليد الشهادات، التحقّق العامّ | Certificate templates, Issued certificates, Verifications |
| 09 | **Commerce** | السلّة، الطلبات، الكوبونات، Order lifecycle | Carts, Orders, Coupons, Refunds |
| 10 | **Billing & Subscriptions** | الاشتراكات المتجدّدة (Wamadat→Tenant و Tenant→Learner) | Subscription cycles, Invoices, ZATCA filings |
| 11 | **Affiliates** | برنامج التسويق بالعمولة | Affiliate partners, Tracking links, Commissions, Payouts |
| 12 | **Engagement** | النقاشات، التعليقات، المراجعات | Comments, Reviews, Discussions, Reactions |
| 13 | **Communication** | البريد، SMS، WhatsApp، Push، In-app | Notifications, Templates, Delivery logs |
| 14 | **Support / CRM** | تذاكر الدعم، الـ pipelines، الـ KB | Tickets, Pipelines, KB articles, CSAT |
| 15 | **Analytics** | التقارير، الـ dashboards، الـ exports | Aggregations, Reports, Saved queries |
| 16 | **Marketing** | الحملات، Landing pages، الإيميل ماركتنغ، Abandoned cart | Campaigns, Funnels, Email sequences, Recovery flows |
| 17 | **Content (CMS)** | المدوّنة، الصفحات الثابتة، البودكاست، FAQ، Banners | Posts, Pages, Podcast episodes, FAQ entries |
| 18 | **AI Assistant** | المساعد الذكي، التوصيات، Embeddings | Conversations, Recommendations, Embeddings index |

### خريطة العلاقات (Bounded Context Map)

```
                      ┌──────────────┐
                      │   Tenancy    │ ← (root context — everyone depends on tenant scope)
                      └──────┬───────┘
                             │
       ┌─────────────────────┼─────────────────────┐
       ▼                     ▼                     ▼
┌──────────────┐      ┌─────────────┐      ┌──────────────┐
│Identity&Access│     │  Catalog    │      │  Marketing   │
└──────┬───────┘      └──────┬──────┘      └──────────────┘
       │                     │
       │                     ▼
       │            ┌────────────────┐
       │            │   Learning     │ ←─── Live Sessions
       │            │                │ ←─── Attendance
       │            └────────┬───────┘ ←─── Assessment ←─── Certification
       │                     │
       │                     ▼
       │            ┌────────────────┐
       └──────────▶│   Commerce     │◀── Affiliates
                    └────────┬───────┘
                             │
                    ┌────────▼────────┐
                    │Billing & Subs.  │ ← Communication
                    └─────────────────┘ ← Analytics ← AI Assistant
                                        ← Support/CRM ← Engagement
                                        ← Content (CMS)
```

> **ملاحظة**: كلّ النقاط في الخريطة تمثّل **Domain Events**, لا direct calls. السهم يعني "يُنصِت لأحداث".

---

## 7. التواصل بين الـ Contexts

### قاعدة جوهرية
> **Bounded Contexts لا تستدعي بعضها مباشرة.** التواصُل عبر **Domain Events فقط**.

### النمط

```
Context A                                    Context B
─────────                                    ─────────
                                                  
[UseCase]                              [Event Listener]
   │                                        ▲
   ├── publishes ─►  Domain Event  ─────────┤
   │                                        │
   ▼                                        ▼
[Repository]                          [UseCase B]
```

### مثال: شراء برنامج

```
1. Commerce.OrderCompleted
        │
        ├──► Learning.GrantAccessListener         (يفتح الكورس للطالب)
        ├──► Communication.SendReceiptListener    (يُرسل إيصال)
        ├──► Analytics.RecordSaleListener         (يحدّث المقاييس)
        ├──► Affiliates.AttributeSaleListener     (يحسب العمولة لو وُجد affiliate)
        ├──► Billing.GenerateInvoiceListener      (ينشئ فاتورة ZATCA)
        └──► AI.UpdateRecommendationsListener     (يحدّث توصيات الطالب)
```

### Implementation
- Laravel Events + Listeners
- Async listeners عبر Redis queue (Horizon)
- Critical listeners معاملاتياً (database transaction) — مثل GrantAccess
- Idempotency keys لمنع التكرار عند retry

### ADR-010 — Event Bus داخلي قبل Message Broker خارجي
نستخدم Laravel Events فوق Redis كـ event bus داخليّ. عند الحاجة لـ cross-service messaging (مثلاً AI service)، نُضيف **Redis Streams** أو **NATS** (ليس RabbitMQ/Kafka — overkill حالياً).

---

## 8. Multi-tenancy

### 8.1 الاستراتيجية: Schema-per-Tenant

```
PostgreSQL Cluster
├── landlord database (shared)
│   ├── tenants                  (Wamadat-owned)
│   ├── tenant_subscriptions     (Wamadat→Tenant billing)
│   └── system_users (Super Admins)
│
├── tenant_wamadat               (flagship academy, schema)
│   ├── users
│   ├── programs
│   ├── orders
│   └── ... (~80 tables)
│
├── tenant_<academy_2>           (future tenant, schema)
│   └── ...
│
└── tenant_<academy_N>
    └── ...
```

### 8.2 المزايا والتحدّيات

| الجانب | Schema-per-tenant | Row-level (tenant_id) | DB-per-tenant |
|---|---|---|---|
| **عزل البيانات** | ✅ قوي | ⚠️ يعتمد على الـ queries | ✅✅ أقصى |
| **النسخ الاحتياطي لمستأجر واحد** | ✅ ممكن | ❌ صعب جداً | ✅ |
| **التكلفة (1000 مستأجر)** | ✅ معقولة | ✅ الأرخص | ❌ غالٍ جداً |
| **Migration عبر المستأجرين** | ⚠️ تتطلّب أتمتة | ✅ سهلة | ❌ معقّدة |
| **Cross-tenant queries (analytics)** | ⚠️ تتطلّب federation | ✅ سهلة | ❌ شبه مستحيلة |
| **الحدّ الأعلى المُجرَّب** | ~10,000 مستأجر | غير محدود | ~100 مستأجر |

**ADR-011**: نختار Schema-per-tenant لأنّ:
- عزل البيانات قويّ بشكل دفاعيّ (RLS + Schema isolation = طبقتان).
- النسخ الاحتياطي لمستأجر مهمّ تجارياً (Right to Be Forgotten + Tenant Export).
- 10,000 مستأجر يكفي للسنوات الـ 3 الأولى.

### 8.3 الأدوات
- **Spatie/laravel-multitenancy** — اختيار schema تلقائياً حسب الـ subdomain
- **stancl/tenancy** (بديل، أوسع ميزات لكن أبطأ) — لو احتجنا multi-database لاحقاً
- **PostgreSQL search_path** — التبديل بين schemas

### 8.4 Tenant Identification

```
Request: https://wamadat.platform.io/learn/123
                │
                ▼
       ┌────────────────────┐
       │ Subdomain Resolver │
       │ "wamadat" → tenant │
       └─────────┬──────────┘
                 │
                 ▼
       ┌────────────────────────┐
       │ Set search_path        │
       │ to "tenant_wamadat"    │
       └─────────┬──────────────┘
                 │
                 ▼
            [Application]
```

### 8.5 Tenant Lifecycle

```
[Provision]  →  [Active]  →  [Suspended]  →  [Archived]  →  [Deleted]
    │              │             │              │            │
    │              │             │              │            └─ بيانات تُحذف 30 يوماً بعد
    │              │             │              └─ بيانات للقراءة فقط
    │              │             └─ subscription لم يُجدَّد
    │              └─ تشغيل عادي
    └─ Tenant جديد: إنشاء schema + seed data
```

---

## 9. Identity & Access

### 9.1 طبقتا الصلاحيات: RBAC + ABAC

| النمط | الغرض | المكتبة |
|---|---|---|
| **RBAC** (Role-Based) | الصلاحيات حسب الأدوار الثابتة | Spatie/laravel-permission |
| **ABAC** (Attribute-Based) | قواعد ديناميكية (مثل: المدرّب يرى طلابه فقط) | Laravel Policies + Gates |

### 9.2 الأدوار الافتراضية (8 roles per tenant)

| الدور | النطاق | أمثلة على الصلاحيات |
|---|---|---|
| **Super Admin** | عابر للمستأجرين (Wamadat فريق) | إدارة tenants، billing، فحص system |
| **Academy Owner** | المستأجر | إدارة كاملة داخل tenant |
| **Instructor** | المستأجر | إنشاء برامج، تعديل كورساته، عرض طلابه |
| **Marketing** | المستأجر | حملات، كوبونات، تحليلات تحويل |
| **Finance** | المستأجر | فواتير، عمولات، تقارير مالية |
| **Support** | المستأجر | تذاكر، KB، نَفاذ قراءة لبيانات الطلاب |
| **Affiliate** | المستأجر | لوحة الشريك، تتبّع روابطه، عمولاته |
| **Student** | المستأجر | تعلّم، تفاعل، حساب شخصي |

### 9.3 Authentication

| القناة | الآلية |
|---|---|
| **Web (Frontend Next.js)** | Sanctum SPA tokens (HTTP-only cookies) |
| **Mobile (PWA)** | Sanctum API tokens (Bearer) |
| **Filament Admin** | Session-based (Laravel native) |
| **API (3rd-party)** | Personal Access Tokens (Sanctum) |
| **SSO (Enterprise tenants)** | OIDC (Filament has built-in support) |
| **OAuth (Google, Apple)** | Laravel Socialite |

### 9.4 MFA
- TOTP (Google Authenticator/Authy) لكلّ Admin roles (إلزامي)
- SMS OTP عبر Unifonic للـ students (اختياري بضغطة)
- Recovery codes (10 codes per user)

---

## 10. Payment Gateway Abstraction

### 10.1 المشكلة
نحتاج دعم متعدّد البوّابات (Tap, Tamara, Mada, Apple Pay) دون أن تتلوّث domain logic. أيّ بوّابة قابلة للإضافة/الاستبدال خلال أسبوع.

### 10.2 التصميم

```
       Application Layer (Commerce)
              │
              │   $gateway->charge($order, $method);
              ▼
    ┌──────────────────────────┐
    │  PaymentGateway          │  ← interface
    │  Interface               │
    └────────┬─────────────────┘
             │  implements
   ┌─────────┼─────────┬───────────┬──────────────┐
   ▼         ▼         ▼           ▼              ▼
┌─────┐  ┌──────┐  ┌──────┐  ┌─────────┐  ┌────────────┐
│ Tap │  │Tamara│  │ Mada │  │Apple Pay│  │Bank Transfer│
│Adapter│ │Adapter│ │Adapter│ │ Adapter │ │  Adapter    │
└─────┘  └──────┘  └──────┘  └─────────┘  └────────────┘
```

### 10.3 PaymentGateway Interface (مرجعيّ)

```php
interface PaymentGateway
{
    public function charge(Order $order, array $context): PaymentResult;
    public function refund(Payment $payment, Money $amount): RefundResult;
    public function verifyWebhook(Request $request): bool;
    public function handleWebhook(Request $request): WebhookHandled;
    public function supportsInstallments(): bool;
    public function supportedMethods(): array;
}
```

### 10.4 Tap Payment Flow

```
[Frontend Checkout]
        │
        │  POST /api/orders { items, gateway:"tap" }
        ▼
[Commerce.PlaceOrder UseCase]
        │
        ├─► Create Order (status: pending)
        │
        │  $tap->charge($order, ['method' => 'card'])
        ▼
[TapAdapter]
        │
        │  HTTP POST https://api.tap.company/v2/charges
        │  Headers: Authorization: Bearer $secret_key
        │  Body: { source, customer, redirect, amount, currency }
        │
        ▼
[Tap returns charge with redirect URL]
        │
        │  PaymentResult { redirectUrl, externalId }
        ▼
[Frontend: redirect user to Tap hosted page]
        │
        │  ... user completes payment ...
        ▼
[Tap webhook → POST /webhooks/tap]
        │
        │  Verify signature (HMAC-SHA256)
        │  Idempotency check (charge_id used?)
        │
        ▼
[TapAdapter.handleWebhook]
        │
        ├─► Order.markPaid → Domain Event: OrderCompleted
        │   └─► Listeners: GrantAccess, SendReceipt, RecordSale, ...
        │
        ▼
[200 OK to Tap]
```

### 10.5 Tamara (BNPL — Buy Now Pay Later)

Tamara تختلف في 3 نواحٍ:
1. **Pre-authorization**: تحقّق من تمويل العميل قبل إكمال الطلب.
2. **Multi-stage status**: `created → approved → captured → declined`.
3. **Late status updates**: قد تتأخّر الموافقة 1–2 ساعة.

```
[Checkout: user selects Tamara]
        │
        ▼
[Commerce.PlaceOrder]
        │
        ├─► Create Order (status: pending_authorization)
        │
        │  $tamara->charge($order) → POST /v2/checkout/order
        ▼
[Tamara returns checkout_url + order_id]
        │
        │  Frontend redirects to Tamara hosted flow
        ▼
[User completes Tamara onboarding]
        │
        ▼
[Tamara webhook: order_approved | order_declined]
        │
        ├─► approved → Order.markAuthorized → wait for capture
        │
        │   (auto-capture after delivery confirmation)
        │
        ├─► Tamara webhook: order_captured
        │   └─► Order.markPaid → Event: OrderCompleted
        │       └─► Listeners as above
        │
        └─► declined → Order.fail → User notified, allow retry
```

### 10.6 Mada / Apple Pay
- Mada يمرّ عبر **Tap** (Tap هو acquirer متوافق مع Mada local cards).
- Apple Pay كذلك عبر Tap (Apple Pay = tokenized card → Tap → Mada/Visa).
- لا حاجة لتكامل مستقلّ مع Mada Network.

### 10.7 ADRs الخاصّة بالدفع

- **ADR-012** — `PaymentGateway` interface + adapter pattern إلزاميّ. ممنوع استدعاء SDK البوّابة مباشرة من Commerce module.
- **ADR-013** — Idempotency: كلّ webhook handler يتحقّق من `external_id` قبل التطبيق. الجدول `payment_webhooks` يحفظ سجلّ التنفيذ.
- **ADR-014** — Webhook security: signature verification إلزاميّ. Webhooks بدون signature صحيح ترفض بـ 401.
- **ADR-015** — Currency: SAR هو الافتراضي. كلّ Money value object يحمل العملة دائماً (لا نخزّن decimals عائمة).

---

## 11. Video & Live Sessions

### 11.1 الفيديو المُسجَّل — **Bunny Stream**

| المعيار | الاختيار |
|---|---|
| التحويل (Transcoding) | تلقائي (multi-bitrate adaptive) |
| التوصيل | Bunny CDN (PoP في الرياض، جدّة، الدمام) |
| DRM | DRM standard (Widevine/PlayReady) |
| Watermarking | ديناميكي بـ user_id + email خفيف |
| الأسعار | $0.005/GB transcoding + $0.005/GB delivery |
| البديل | Mux (أغلى)، AWS Elemental (أعقد) |

### 11.2 الجلسات المباشرة — **100ms**

**ADR-016** — نختار **100ms** كـ Live SDK الأساسيّ. القرار النهائي.

#### المقارنة

| البعد | Zoom Meeting SDK | 100ms | Daily.co | LiveKit |
|---|---|---|---|---|
| White-label | ❌ Zoom branding | ✅ | ✅ | ✅ |
| تكلفة (1000 سا/شهر) | ~$1,500 | ~$600 | ~$400 | ~$200 (self-host) |
| MENA latency | جيّد | ✅ ممتاز (PoP UAE) | متوسّط | حسب الـ host |
| Recording مدمج | ✅ | ✅ | ✅ | يدوي |
| WebRTC native | جزئي | ✅ | ✅ | ✅ |
| Mobile SDK | ✅ | ✅ | ✅ | ✅ |
| إدارة المشغّلين | ❌ معقّدة | ✅ سهلة | ✅ | يدوي |
| سمعة في MENA | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |

#### لماذا 100ms (وليس الآخرين)
- **White-label** ضروريّ — كلّ tenant يبيع لطلابه تحت علامته، لا "Zoom".
- **MENA latency** — PoP في دبي يضع UAE/KSA على < 80ms.
- **تكلفة معقولة** — 60% أقلّ من Zoom Meeting SDK.
- **Recording + Composition** مدمج (لا حاجة لخدمة ثانية).
- **JS/React SDK نظيف** — تكامل مع Next.js مباشر.

#### المخاطرة
100ms شركة أحدث من Zoom (تأسّست 2021، Series A). نخفّف بـ:
- **Adapter pattern** على Live SDK (مثل PaymentGateway) — استبدال 100ms بـ Zoom/Daily خلال أسبوع لو احتجنا.

---

## 12. AI Microservice

### 12.1 الفصل

**ADR-017** — AI service مفصول كـ Python FastAPI من اليوم الأوّل.

```
[Laravel Monolith]
       │
       │  HTTP/JSON
       ▼
[AI Service — FastAPI, Python 3.12]
       │
       ├─► Anthropic Claude API (text/chat)
       ├─► OpenAI Embeddings (vector generation)
       └─► PostgreSQL pgvector (similarity search)
```

### 12.2 الميزات

| الميزة | الـ Endpoint | الـ TTI المستهدف |
|---|---|---|
| المساعد الذكي (Chat) | `POST /ai/chat` | < 800ms first token |
| توصيات الكورسات | `GET /ai/recommendations/{user}` | < 200ms (cached) |
| البحث الدلاليّ | `POST /ai/semantic-search` | < 300ms |
| توليد ملخّصات الدروس | `POST /ai/summarize` | < 3s |
| فحص جودة المحتوى (للمدرّب) | `POST /ai/quality-check` | < 5s |

### 12.3 Embeddings Strategy
- نموذج: OpenAI `text-embedding-3-small` (1536 بعد).
- التخزين: `pgvector` في PostgreSQL (لا حاجة لـ Pinecone/Weaviate).
- التحديث: عند نشر/تعديل كورس → event → embedding job → upsert.

### 12.4 Cost Control
- **Caching agressive** — Redis 24h لكلّ توصية، 7 days لكلّ embedding.
- **Rate limiting per user** — 50 chat msgs/day على Free، 200 على Pro.
- **Token budget per request** — clamp max_tokens، use Haiku for cheap ops.
- **Async by default** — توصيات تُحدَّث في background, ليس عند كلّ request.

---

## 13. Search Strategy

### 13.1 Meilisearch لأنّ
- **بحث عربيّ ممتاز** افتراضياً (يفهم الـ stemming العربيّ، يتجاهل الـ tashkeel).
- **Typo-tolerant** بدون إعدادات.
- **سريع جداً** (< 50ms لـ 1M وثيقة).
- **Faceted search** out-of-the-box (تصفية بالـ category, price, level).
- **Self-hosted أو cloud** — مرونة.

### 13.2 ما يُفهرَس
| المؤشّر (Index) | الحقول |
|---|---|
| `programs` | title_ar, title_en, description, instructor, category, tags |
| `instructors` | name_ar, name_en, bio, specializations |
| `posts` (Blog) | title, content, tags |
| `faq` | question, answer, category |
| `users` (Admin only) | name, email, phone (للـ super admin search) |

### 13.3 Synchronization
- Eloquent Observer على Model events → Async queue job → Meilisearch upsert.
- مكتبة: Laravel Scout + meilisearch driver.

### 13.4 Multi-tenancy في Search
لكلّ مستأجر **Meilisearch index منفصل** (نمط: `tenant_{id}_programs`). يحقّق العزل ويسمح بإعدادات مخصَّصة لكلّ tenant.

---

## 14. Caching Strategy

### 14.1 طبقات الـ Caching

```
┌─────────────────────────────────────────────────────────┐
│  L1: Vercel Edge Cache (Next.js)                        │
│      Static pages, ISR pages, API responses (public)    │
│      TTL: 1m–1h                                          │
├─────────────────────────────────────────────────────────┤
│  L2: Laravel Cache (Redis)                              │
│      Computed data, query results, rate limits          │
│      TTL: 5m–24h                                         │
├─────────────────────────────────────────────────────────┤
│  L3: Database Query Cache (PostgreSQL)                  │
│      Implicit via shared_buffers                        │
└─────────────────────────────────────────────────────────┘
```

### 14.2 ما يُكاش وأين

| البيانات | الموقع | TTL | Invalidation |
|---|---|---|---|
| Program catalog page | Edge ISR | 5 min | Tag-based on publish |
| User dashboard data | Redis | 10 min | Event-based |
| Tenant config | Redis | 24h | Event-based |
| Rate limit counters | Redis | rolling | TTL |
| Session data | Redis | 7 days | Login/logout |
| AI recommendations | Redis | 24h | On new enrollment |
| Search results (top queries) | Meilisearch internal | dynamic | dynamic |

### 14.3 Redis Topology

```
Redis Cluster (3 nodes minimum للإنتاج)
├── DB 0: Cache (Laravel default)
├── DB 1: Sessions
├── DB 2: Queues (Horizon)
├── DB 3: Broadcasting (Reverb pub/sub)
└── DB 4: Rate Limiting
```

---

## 15. Background Jobs

### 15.1 Laravel Horizon
- Dashboard مدمج لمراقبة الـ jobs.
- Auto-scaling workers حسب الحمل.
- Failed jobs دائماً مع retry policy.

### 15.2 Queue Priorities

| Queue | الـ Workers | أمثلة |
|---|---|---|
| `high` | 8 | Payment webhooks, OTP SMS, certificate generation |
| `default` | 16 | Email send, search index update |
| `low` | 4 | Analytics aggregation, embedding generation |
| `notifications` | 8 | Push, WhatsApp |
| `media` | 2 | Video processing callbacks |

### 15.3 Cron Jobs (الـ Schedule)

| المهمّة | التكرار |
|---|---|
| Renew tenant subscriptions | كل ساعة |
| Issue invoices (ZATCA) | يومياً 02:00 KSA |
| Aggregate daily analytics | يومياً 03:00 KSA |
| Cleanup expired tokens | يومياً 04:00 KSA |
| Send drip content emails | كل 15 دقيقة |
| Send live session reminders | كل 10 دقائق |
| Refresh AI recommendations | يومياً 05:00 KSA |
| Backup verification | يومياً 06:00 KSA |

---

## 16. Real-time Layer

### 16.1 Laravel Reverb
- خادم WebSocket مدمج مع Laravel (لا Pusher SaaS).
- Pub/sub عبر Redis.
- يدعم Laravel Echo client (المتوافق مع Next.js).

### 16.2 الاستخدامات

| الميزة | القناة | الـ Event |
|---|---|---|
| Live session presence | `presence.session.{id}` | join/leave |
| Live chat | `private.session.{id}.chat` | message |
| Live poll | `private.session.{id}.poll` | new/result |
| Attendance check-in | `private.session.{id}.attendance` | check-in |
| Notifications inbox | `private.user.{id}.notifications` | new |
| Order status (للمتعلِّم) | `private.user.{id}.orders` | status_changed |
| Admin dashboard updates | `private.tenant.{id}.metrics` | tick |

---

## 17. Storage & CDN

### 17.1 Bunny Topology

```
                  ┌────────────────┐
                  │ Bunny.net      │
                  ├────────────────┤
                  │ Storage Zones  │  → S3-compatible API
                  │ (Riyadh PoP)   │
                  ├────────────────┤
                  │ Stream Library │  → Video transcoding + delivery
                  │ (auto-CDN)     │
                  ├────────────────┤
                  │ CDN            │  → Static assets, images, PDFs
                  │ (global PoPs)  │
                  └────────────────┘
```

### 17.2 ما يُخزَّن أين

| النوع | الموقع | TTL Cache | Access |
|---|---|---|---|
| Course videos | Bunny Stream | dynamic | Signed URLs (1h) |
| Course materials (PDF/ZIP) | Bunny Storage + CDN | 1d | Signed URLs |
| Brand assets, UI images | Bunny CDN | 30d | Public |
| User avatars | Bunny Storage + CDN | 7d | Public |
| Certificate PDFs | Bunny Storage | dynamic | Signed URLs |
| Tenant logos/banners | Bunny CDN | 30d | Public |
| Backup files | Bunny Storage (cold) | — | Internal only |

### 17.3 Asset URLs (multi-tenant aware)

```
https://cdn.wamadat.io/t/{tenant_slug}/avatars/{user_id}.webp
https://stream.wamadat.io/t/{tenant_slug}/videos/{program_id}/{lesson_id}.m3u8?token=...
```

---

## 18. Observability

### 18.1 Tools

| الحاجة | الأداة |
|---|---|
| Error tracking | Sentry (BE + FE) |
| Uptime monitoring | Better Stack |
| Logs aggregation | Better Stack Logs |
| Metrics (custom KPIs) | Prometheus (self-hosted، optional) |
| APM | Sentry Performance |
| Real User Monitoring | Vercel Speed Insights + Sentry |

### 18.2 ما يُسجَّل دائماً

- كلّ request ID (`X-Request-ID`).
- كلّ tenant context (`tenant_id` في كلّ log entry).
- كلّ payment webhook (idempotency log).
- كلّ admin action (audit log منفصل، immutable).
- كلّ failed job مع stack trace كامل.

### 18.3 Alerts
- p95 latency > 500ms (warn) / > 1s (page).
- Error rate > 1% (warn) / > 5% (page).
- Queue depth > 1000 (warn) / > 5000 (page).
- DB connections > 80% (warn).
- Webhook signature failures > 10/min (security alert).

---

## 19. Scalability Plan

### Phase 1 — MVP (0–1,000 active users)
- Single Railway/Fly.io instance (4 vCPU, 8GB RAM).
- PostgreSQL single primary (managed Railway/Neon).
- Redis single instance.
- Bunny.net default tier.
- **Cost estimate**: ~$300/شهر.

### Phase 2 — Growth (1k–100k active users)
- Horizontal scaling Laravel Octane (3–5 instances behind load balancer).
- PostgreSQL primary + 2 read replicas.
- Redis Cluster (3 nodes).
- Bunny Premium tier.
- AI service خلف Cloudflare workers أو مستقل.
- **Cost estimate**: ~$2,500/شهر.

### Phase 3 — Scale (100k–1M active users)
- Kubernetes (EKS/GKE) أو Railway Pro.
- PostgreSQL partitioning (by tenant_id ranges) أو Citus.
- Redis Enterprise أو AWS ElastiCache.
- مفصول AI + Search إلى خدمات مستقلّة.
- Multi-region active-passive.
- **Cost estimate**: ~$15,000/شهر.

### Phase 4 — Hyper-scale (1M+)
- نَفصِل Bounded Contexts الأثقل (Learning, Commerce) إلى خدمات مستقلّة.
- Event-driven architecture كامل (NATS أو Kafka).
- CDN-cached APIs (Vercel + Edge functions).
- إعادة تقييم.

---

## 20. ADR Index

تلخيص لكلّ ADRs المُتَّخَذة (الإضافة الجديدة في هذا المستند مُعَلَّمة 🆕):

| # | القرار | الحالة |
|---|---|---|
| ADR-001 | Multi-tenant SaaS, Scenario B phased rollout | ✅ مُقفَل |
| ADR-002 | Backend: Laravel 11 + Filament 3 | ✅ مُقفَل |
| ADR-003 | Modular Monolith + AI microservice | ✅ مُقفَل |
| ADR-004 | Payment Gateway Abstraction | ✅ مُقفَل |
| ADR-005 | Bunny Stream للفيديو المُسجَّل | ✅ مُقفَل |
| ADR-006 | RBAC + ABAC، 8 أدوار افتراضية | ✅ مُقفَل |
| ADR-007 | Frontend: Next.js 15 + TS + Tailwind 4 | ✅ مُقفَل |
| ADR-008 | Infra: Postgres 16 + Redis 7 + Meilisearch + Bunny | ✅ مُقفَل |
| ADR-009 🆕 | Modular Monolith قبل Microservices | ✅ مُقفَل |
| ADR-010 🆕 | Event bus داخلي (Laravel Events + Redis) قبل Message Broker | ✅ مُقفَل |
| ADR-011 🆕 | Schema-per-tenant (ليس row-level ولا DB-per-tenant) | ✅ مُقفَل |
| ADR-012 🆕 | PaymentGateway interface + Adapter pattern إلزاميّ | ✅ مُقفَل |
| ADR-013 🆕 | Webhook idempotency عبر `payment_webhooks` table | ✅ مُقفَل |
| ADR-014 🆕 | Webhook signature verification إلزاميّ | ✅ مُقفَل |
| ADR-015 🆕 | Money value object يحمل العملة دائماً (SAR default) | ✅ مُقفَل |
| ADR-016 🆕 | **100ms** كـ Live SDK الأساسيّ (وليس Zoom) | ✅ مُقفَل |
| ADR-017 🆕 | AI service Python/FastAPI مفصول من اليوم الأوّل | ✅ مُقفَل |

---

## 📌 التالي

- [`03-domain-model.md`](03-domain-model.md) — نمذجة كلّ Bounded Context (entities، VOs، aggregates).
- [`04-database-schema.md`](04-database-schema.md) — مخطّط قاعدة البيانات الكامل (~80 جدول).

---

<sub>**النسخة**: 1.0 · **آخر تحديث**: 2026-05-11 · **حالة**: مُعتمَدة معمارياً · **عدد الـ ADRs النشطة**: 17</sub>
