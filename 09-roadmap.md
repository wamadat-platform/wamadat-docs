# 09 — Roadmap

> **خارطة الطريق المرحلية.** كلّ phase له deliverables واضحة، KPIs، و exit criteria. التعديل يَتطلَّب اجتماع steering مع owner.

---

## 📑 الفهرس

1. [Overview](#1-overview)
2. [Phase 0 — Foundation](#2-phase-0--foundation)
3. [Phase 1 — MVP](#3-phase-1--mvp)
4. [Phase 2 — Growth](#4-phase-2--growth)
5. [Phase 3 — Excellence](#5-phase-3--excellence)
6. [Phase 4 — Scale](#6-phase-4--scale)
7. [Feature Matrix](#7-feature-matrix)
8. [Tenant Onboarding Rollout](#8-tenant-onboarding-rollout)

---

## 1. Overview

| Phase | الفترة | الهدف الأساسيّ | KPI الجوهري |
|---|---|---|---|
| **Phase 0** — Foundation | M0–M1 | بنية تحتية، تأسيس، فريق | جاهزية النشر، CI green |
| **Phase 1** — MVP | M1–M4 | إطلاق أكاديمية ومضات (Flagship Tenant) | أوّل 100 متعلِّم مدفوع |
| **Phase 2** — Growth | M5–M8 | تعميق الميزات + جذب 200+ متعلّم/شهر | 1,000 MAU، 30+ كورس منشور |
| **Phase 3** — Excellence | M9–M12 | فتح المنصّة لمستأجرين آخرين بالدعوة | 5 مستأجرين، 10k MAU |
| **Phase 4** — Scale | M13+ | Public SaaS launch، تحسين الأداء، توسّع | 100 مستأجر، 100k MAU |

---

## 2. Phase 0 — Foundation (M0–M1)

### Deliverables
- **Documentation**: PHASE 1 (هذا الـ docs/ folder) ✅
- **Project skeleton** (PHASE 2): repos، CI، docker-compose
- **Backend boilerplate** (PHASE 3): Laravel + Filament + tenant config
- **Frontend boilerplate** (PHASE 4): Next.js + i18n + RTL + design tokens
- **DevOps** (PHASE 5): docker-compose، scripts، seed
- **Quality gates** (PHASE 6): CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, CHANGELOG

### Team Setup
- **Tech Lead** (1) — أنت أو مَن تَعيِّن
- **Backend Senior** (1) — Laravel/PHP
- **Frontend Senior** (1) — Next.js/React
- **Designer** (0.5 FTE) — UI/UX
- **Owner/PM** — أنت

### Exit Criteria
- ✅ كلّ docs PHASE 1 معتمدة.
- ✅ `setup.ps1` يَعمل من صفر لـ running env خلال 30 دقيقة.
- ✅ Tests CI green على main.
- ✅ Staging environment يُنشَر يدوياً بنجاح.

---

## 3. Phase 1 — MVP (M1–M4)

### Goal
إطلاق **أكاديمية ومضات** كأوّل (والوحيد) tenant على المنصّة. تجربة learner كاملة من signup إلى certificate.

### Modules المطلوبة في MVP

| Module | الحجم | Notes |
|---|---|---|
| **Tenancy** | Core | فقط Wamadat tenant، لا public signup |
| **Identity & Access** | Core | signup، login، MFA للـ admin، password reset |
| **Catalog** | Core | برامج، تصنيفات، صفحات المدرّبين |
| **Learning** | Core | فيديو، PDF، progress، self-paced فقط |
| **Live Sessions** | **Phase 2** | يُؤجَّل |
| **Attendance** | **Phase 2** | يُؤجَّل |
| **Assessment** | Minimal | quizzes MCQ فقط |
| **Certification** | Minimal | template واحد، توليد PDF، verification |
| **Commerce** | Core | cart، order، Tap payment، Tamara |
| **Billing** | Minimal | فاتورة بسيطة، ZATCA submission |
| **Affiliates** | **Phase 2** | يُؤجَّل |
| **Engagement** | Minimal | reviews + comments فقط |
| **Communication** | Core | email transactional فقط (Resend) |
| **Support** | **Phase 2** | يُؤجَّل (نَستخدم email لـ MVP) |
| **Analytics** | Minimal | dashboards الـ owner الأساسية |
| **Marketing** | **Phase 2** | يُؤجَّل |
| **Content (CMS)** | Minimal | blog فقط |
| **AI Assistant** | **Phase 2** | يُؤجَّل |

### Public Pages
- Homepage (hero + featured programs + testimonials + CTA)
- Programs list + Program detail
- About / Instructors
- Blog (4–6 posts للـ launch)
- Pricing (if applicable)
- Privacy + Terms + Contact

### Dashboards
- **Student Dashboard**: my courses، progress، certificates، account
- **Instructor Studio**: create/edit program، lessons، students list
- **Owner Dashboard** (Filament): full admin

### KPIs at end of Phase 1
- ✅ 100 متعلّم مدفوع.
- ✅ 10+ برامج منشورة.
- ✅ Order success rate > 95%.
- ✅ Page LCP < 2s.
- ✅ Zero critical bugs in production.

### Exit Criteria
- ✅ Real users تَكمل journey كامل (signup → buy → learn → certificate).
- ✅ Payments تمرّ عبر Tap + Tamara بنجاح.
- ✅ ZATCA invoices تُقدَّم تلقائياً.
- ✅ NPS > 40 من أوّل 50 متعلّم.

---

## 4. Phase 2 — Growth (M5–M8)

### Goal
تعميق الميزات + جذب 200+ متعلّم/شهر + تحضير المنصّة لـ multi-tenancy.

### New Modules
- **Live Sessions** — 100ms integration، recording، polls.
- **Attendance** — QR check-in للحضوريّ + auto-presence للـ live.
- **Marketing** — campaigns، coupons، abandoned cart، landing pages.
- **Affiliates** — برنامج عمولات كامل.
- **Support / CRM** — ticketing + KB.
- **AI Assistant** — chatbot عربي مع context الكورس.

### Enhancements
- **Catalog**: cohort-based programs، memberships.
- **Assessment**: assignments + grading.
- **Commerce**: Mada، Apple Pay، bank transfer.
- **Communication**: SMS (Unifonic) + WhatsApp.
- **Analytics**: cohort analytics، instructor performance، sales funnel.
- **CMS**: pages، podcast، FAQ، banners، landing pages.

### Tenant Prep
- **Multi-tenancy infra اختبار**: نُنشئ tenant ثانٍ داخلياً (شركة شريكة كـ pilot).
- **Tenant onboarding flow**: نَختبره بدون publish.

### KPIs at end of Phase 2
- 1,000 MAU.
- 30+ برامج منشورة (لـ Wamadat).
- 10+ مدرّبين مفعّلين.
- MRR من المتعلّمين: 75,000 ر.س.
- Course completion rate > 30%.
- p95 API latency < 300ms.

---

## 5. Phase 3 — Excellence (M9–M12)

### Goal
فتح المنصّة لـ **5 مستأجرين بالدعوة** (invite-only)، تعميق الأداء والـ UX.

### New Capabilities
- **Multi-tenant onboarding** (invite link → tenant signup → schema provision → checkout).
- **Tenant subscription billing** (Wamadat → Tenant via Cashier).
- **Custom domains** (Pro+ tenants).
- **White-label** للـ certificates, emails.
- **Tenant analytics**: per-tenant dashboard.

### Enhancements
- **Search**: faceted، فِلتر متقدّم.
- **PWA**: offline support، push notifications.
- **Recommendations**: AI-driven للـ landing.
- **API public**: تَوثيق + dev portal.
- **Webhooks public**: tenants يَستلموا events.

### Tenant Acquisition (invite-only)
- التواصل المباشر مع 20 أكاديمية محلية.
- Onboarding مدفوع بمساعدة فريقنا (high-touch).
- نختار 5 أكاديميات pilot.

### KPIs at end of Phase 3
- 5 مستأجرين نشطين.
- 10,000 MAU عبر كلّ المستأجرين.
- 500+ برامج منشورة.
- ARR: 1.5M+ ر.س.
- NRR > 100%.
- Tenant churn: 0.

---

## 6. Phase 4 — Scale (M13+)

### Goal
Public SaaS launch + توسّع جغرافيّ + بنية تحتية تحتمل 100k+ MAU.

### New Capabilities
- **Public tenant signup** بدون انتظار approval.
- **Self-serve plans** (free → starter → pro → business).
- **Marketplace ميزات** (templates, plugins).
- **Mobile apps** (iOS + Android، native أو RN).
- **Enterprise sales** funnel.
- **Multi-region** (UAE PoP).

### Infrastructure
- Kubernetes أو Railway Pro.
- Postgres partitioning أو Citus.
- Multi-region active-passive.
- Bug bounty program.
- ISO 27001 compliance prep.

### KPIs at end of M18
- 100+ مستأجر نشط.
- 100,000 MAU.
- ARR: 10M+ ر.س.
- LTV/CAC > 3.5x.
- NPS > 50.

---

## 7. Feature Matrix

✅ = موجود · 🔶 = جزئي · ❌ = غير موجود · 🔮 = مخطّط

| Feature | Phase 1 | Phase 2 | Phase 3 | Phase 4 |
|---|---|---|---|---|
| Tenant onboarding | 🔶 (Wamadat فقط) | 🔶 (داخليّ) | ✅ (invite) | ✅ (public) |
| Recorded courses | ✅ | ✅ | ✅ | ✅ |
| Live sessions | ❌ | ✅ (100ms) | ✅ | ✅ |
| In-person workshops + QR | ❌ | ✅ | ✅ | ✅ |
| Hybrid programs | ❌ | ✅ | ✅ | ✅ |
| Memberships | ❌ | ✅ | ✅ | ✅ |
| Consultations | ❌ | ❌ | ✅ | ✅ |
| Webinars | ❌ | ✅ | ✅ | ✅ |
| Podcasts | ❌ | ✅ | ✅ | ✅ |
| Certificates | ✅ (basic) | ✅ (templates) | ✅ | ✅ |
| Assessments — Quizzes | ✅ MCQ | ✅ (all types) | ✅ | ✅ |
| Assessments — Assignments | ❌ | ✅ | ✅ | ✅ |
| Drip content | ❌ | ✅ | ✅ | ✅ |
| Cohort-based | ❌ | ✅ | ✅ | ✅ |
| Gamification (badges/points) | ❌ | 🔶 | ✅ | ✅ |
| Multi-branch | ❌ | ✅ | ✅ | ✅ |
| Tap payment | ✅ | ✅ | ✅ | ✅ |
| Tamara payment | ✅ | ✅ | ✅ | ✅ |
| Mada / Apple Pay | ❌ | ✅ | ✅ | ✅ |
| STC Pay / Bank transfer | ❌ | 🔶 | ✅ | ✅ |
| Subscriptions (tenant→learner) | ❌ | ✅ | ✅ | ✅ |
| ZATCA Phase 2 | ✅ | ✅ | ✅ | ✅ |
| Affiliates program | ❌ | ✅ | ✅ | ✅ |
| Coupons | ❌ | ✅ | ✅ | ✅ |
| Email marketing campaigns | ❌ | ✅ | ✅ | ✅ |
| SMS + WhatsApp | ❌ | ✅ | ✅ | ✅ |
| AI assistant (Claude) | ❌ | ✅ | ✅ | ✅ |
| AI recommendations | ❌ | ✅ | ✅ | ✅ |
| Semantic search | ❌ | 🔶 | ✅ | ✅ |
| Support tickets | ❌ | ✅ | ✅ | ✅ |
| KB articles | ❌ | ✅ | ✅ | ✅ |
| CRM pipelines | ❌ | ❌ | ✅ | ✅ |
| Analytics dashboards | 🔶 | ✅ | ✅ | ✅ |
| Public API + Webhooks | ❌ | 🔶 | ✅ | ✅ |
| Custom domains | ❌ | ❌ | ✅ | ✅ |
| White-label | ❌ | ❌ | ✅ | ✅ |
| PWA / Mobile app | ❌ | 🔶 PWA | ✅ PWA | ✅ Native |

---

## 8. Tenant Onboarding Rollout

### المرحلة الحالية (Phase 1)
- **عدد المستأجرين**: 1 (Wamadat Academy فقط).
- **Public signup**: مغلق.
- **Onboarding**: يدويّاً (شخص في الفريق يُنشئ tenant).

### Phase 2 (M5–M8)
- **عدد المستأجرين**: 2 (Wamadat + pilot واحد).
- **Public signup**: مغلق.
- **Onboarding**: حالياً يدويّاً، نَختبر الـ flow الكامل في staging.

### Phase 3 (M9–M12)
- **عدد المستأجرين**: حتى 5 (invite-only).
- **Public signup**: مغلق، invite link فقط.
- **Onboarding**: self-serve مع high-touch support.

### Phase 4 (M13+)
- **عدد المستأجرين**: غير محدود.
- **Public signup**: ✅ مفتوح.
- **Onboarding**: 100% self-serve.

---

## ADRs الجديدة

| # | القرار |
|---|---|
| **ADR-031** | Phased multi-tenancy exposure: M1–6 single, M7–9 invite-only، M10+ public |
| **ADR-032** | MVP scope limited to self-paced learning فقط (لا live في Phase 1) |
| **ADR-033** | AI features postponed to Phase 2 لتجنّب تأخير MVP |

---

<sub>**النسخة**: 1.0 · **عدد ADRs النشطة** بعد PHASE 1: **33** · **التالي**: PHASE 2 (Project Skeleton)</sub>
