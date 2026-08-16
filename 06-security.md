# 06 — Security

> **الأمان غير قابل للمساومة.** هذا المستند يَحكم كلّ قرار أمنيّ في المنصّة. نلتزم بـ OWASP Top 10 (2021) + PDPL (السعودية) + ZATCA Phase 2 + GDPR (للمستخدمين خارج KSA).

---

## 📑 الفهرس

1. [مبادئ الأمان](#1-مبادئ-الأمان)
2. [Threat Model](#2-threat-model)
3. [OWASP Top 10 Mapping](#3-owasp-top-10-mapping)
4. [Authentication & MFA](#4-authentication--mfa)
5. [Authorization (RBAC + ABAC)](#5-authorization-rbac--abac)
6. [Multi-tenancy Isolation](#6-multi-tenancy-isolation)
7. [Encryption (At-Rest + In-Transit)](#7-encryption-at-rest--in-transit)
8. [Secret Management](#8-secret-management)
9. [Input Validation & Output Encoding](#9-input-validation--output-encoding)
10. [Audit Logs](#10-audit-logs)
11. [Rate Limiting & Anti-Abuse](#11-rate-limiting--anti-abuse)
12. [Payment Security (PCI-DSS scope)](#12-payment-security-pci-dss-scope)
13. [PDPL Compliance (السعودية)](#13-pdpl-compliance)
14. [ZATCA Phase 2 Compliance](#14-zatca-phase-2-compliance)
15. [GDPR (للمستخدمين الأوروبيين)](#15-gdpr)
16. [Vulnerability Management](#16-vulnerability-management)
17. [Incident Response](#17-incident-response)
18. [Security Testing](#18-security-testing)

---

## 1. مبادئ الأمان

1. **Defense in depth** — طبقات متعدّدة، لا نقطة فشل واحدة.
2. **Least privilege** — كلّ كيان يحصل على أقلّ صلاحية ممكنة.
3. **Secure by default** — الإعدادات الافتراضية آمنة، التراخي يتطلّب explicit opt-in.
4. **Fail closed** — عند الشكّ، نَرفض.
5. **Trust no input** — كلّ data من خارج النظام مُلوّثة حتى تثبت نظافتها.
6. **Audit everything that matters** — كلّ تغيير حسّاس يُسجَّل immutable.
7. **No security through obscurity** — الـ algorithms معروفة، السرّ في الـ keys.

---

## 2. Threat Model

### Actors
| Threat Actor | Motivation | Capability |
|---|---|---|
| Script kiddie | للتجربة | منخفضة |
| Competitor | تجسّس صناعي | متوسّطة |
| Disgruntled tenant | انتقام | متوسّطة (يعرف الـ flow) |
| Insider (مطوّر) | تسريب بيانات | عالية (نخفّف بـ audit logs + LP) |
| Organized cybercrime | مال (مدفوعات، بيانات) | عالية |
| Nation-state | بيانات بنية تحتية | حرجة (لا نحمي ضدّ هذا بعمق) |

### Assets الأكثر قيمة
1. بيانات المتعلِّمين (PII).
2. بيانات الدفع (في الـ gateway، ليس عندنا — انظر §12).
3. محتوى المدرّبين (IP).
4. أسرار النظام (DB، API keys).
5. سمعة البرند.

### Attack Surfaces
- Public REST API.
- Tenant subdomains.
- Filament admin panels.
- Webhook endpoints (Tap, Tamara, AI).
- File uploads.
- Email/SMS bounce handlers.

---

## 3. OWASP Top 10 Mapping

| OWASP | Threat | كيف نخفّف |
|---|---|---|
| **A01** Broken Access Control | Tenant cross-access، privilege escalation | RBAC + ABAC + tenant middleware في كلّ route + Policies على كلّ Model |
| **A02** Cryptographic Failures | Weak crypto, plaintext secrets | argon2id للـ passwords، AES-256-GCM للـ data at rest، TLS 1.3 |
| **A03** Injection | SQL, NoSQL, OS command | Eloquent (parameterized)، لا `DB::raw($input)`، Pint/PHPStan في CI |
| **A04** Insecure Design | Logic flaws | Threat modeling لكلّ feature جديد، security review قبل merge |
| **A05** Security Misconfig | Debug mode, default creds | `.env` strict checklist، `php artisan config:cache` في prod، scanners CI |
| **A06** Vulnerable Components | CVEs في dependencies | Dependabot + composer audit + npm audit في CI |
| **A07** Identification & Auth Failures | Weak passwords, session hijack | argon2id + MFA + Sanctum + secure cookies |
| **A08** Software & Data Integrity | Supply chain | Lockfiles، SRI للـ scripts، signed releases |
| **A09** Security Logging Failures | لا audit | audit_logs immutable + structured logs + Sentry alerts |
| **A10** SSRF | Server-side request forgery | URL allow-list لكلّ outbound HTTP، block private IPs |

---

## 4. Authentication & MFA

### Password Policy
- **Min length**: 10.
- **Required**: حرف، رقم.
- **Optional**: رمز (موصى به).
- **Blacklist**: top-10k common passwords (Have I Been Pwned).
- **Hashing**: argon2id (m=65536, t=4, p=1).
- **Reset**: token expires in 60 min, one-time use.

### MFA
| الدور | إلزامي؟ |
|---|---|
| Super Admin | ✅ TOTP |
| Academy Owner | ✅ TOTP |
| Finance | ✅ TOTP |
| Support | اختياري |
| Instructor | اختياري |
| Student | اختياري |

- TOTP: RFC 6238، 30-second window.
- Recovery codes: 10، single-use، hashed.
- SMS OTP **ليس** MFA — للـ phone verification فقط.

### Session
- Sanctum HTTP-only cookies (frontend SPA).
- Cookie flags: `Secure`, `HttpOnly`, `SameSite=Lax`.
- Session timeout: 7 days للـ student، 24 hours للـ admin.
- Concurrent sessions: مسموح، لكن قابل للـ revoke من device list.

### Brute-Force Protection
- 5 failed attempts → 15 min lockout على الـ account.
- 20 failed attempts من نفس IP → 1h block.
- CAPTCHA بعد 3 failed attempts.

---

## 5. Authorization (RBAC + ABAC)

### RBAC (Spatie/laravel-permission)
- الأدوار الـ 8 معرّفة في `02-architecture.md §9`.
- Permissions موزَّعة على dot-notation: `programs.publish`, `users.impersonate`.
- Roles تُسنَد للـ User per tenant.

### ABAC (Laravel Policies)
- Tenant scoping: كلّ Query يَحضر `tenant_id` تلقائياً عبر global scope.
- Ownership: Instructor يرى برامجه فقط (`$user->id === $program->instructor_id`).
- Contextual: Support يرى ticket الطالب الذي يدعمه فقط.

### Authorization Flow
```
Request →
  Auth Middleware (تحقق token) →
    Tenant Middleware (تحقق tenant) →
      Permission Middleware (RBAC check) →
        Controller →
          Policy::authorize() (ABAC check) →
            UseCase
```

كلّ طبقة تَفشل تَنتج 401/403 صريحاً.

---

## 6. Multi-tenancy Isolation

### الطبقة 1 — Schema-per-tenant
- PostgreSQL `search_path` يُحدَّد بالـ middleware.
- لا يمكن لـ tenant A الـ query على schema tenant B بـ Eloquent.

### الطبقة 2 — Application checks
- كلّ Eloquent model له trait `BelongsToTenant` يَتحقّق أنّ الـ query ضمن الـ tenant الحاليّ.
- إذا حاول query على ID ينتمي لـ tenant آخر → ModelNotFoundException.

### الطبقة 3 — Audit
- كلّ cross-tenant access (Super Admin فقط) يُسجَّل في `audit_logs_central`.

### Subdomain Hijacking
- Subdomain حُجِز يبقى حتى لو الـ tenant `archived`.
- `wamadat.platform.io` reserved permanently.

---

## 7. Encryption (At-Rest + In-Transit)

### At-Rest
- **PostgreSQL**: Transparent Data Encryption عبر cloud provider (Railway/AWS).
- **Backups**: AES-256 encrypted.
- **Bunny Storage**: encrypted at rest (provider default).
- **Sensitive columns**: `mfa_secret_encrypted`, `bank_details` → app-level AES-256-GCM بمفتاح من Vault.

### In-Transit
- **TLS 1.3** فقط (لا 1.2 ولا 1.1).
- HSTS: `max-age=63072000; includeSubDomains; preload`.
- Certificates: Let's Encrypt + auto-renewal.
- Internal service-to-service: mTLS بين Laravel ↔ AI service.

### Application-level Crypto
- Laravel `encrypt()`/`decrypt()` للحقول الحسّاسة (AES-256-GCM).
- Sodium للـ TOTP secrets.
- Keys في environment variables، مع rotation سنوياً.

---

## 8. Secret Management

### Storage
- **Development**: `.env` (git-ignored)، sample في `.env.example`.
- **Staging/Production**: 
  - Railway: encrypted env vars.
  - Vercel: encrypted env vars.
  - Sensitive (DB, payment keys): HashiCorp Vault أو AWS Secrets Manager (Phase 2+).

### Rotation
- **API keys** للـ third-party: كل 90 يوماً.
- **DB passwords**: كل 180 يوماً.
- **JWT signing key**: كل سنة.
- **Webhook secrets**: عند تغيير عقد مع المزوِّد فقط.

### قواعد
- ❌ ممنوع committing secrets to Git (Husky pre-commit hook + GitGuardian).
- ❌ ممنوع logging secrets (filtered في Laravel `logging.php`).
- ❌ ممنوع secrets في URLs (use Authorization header).

---

## 9. Input Validation & Output Encoding

### Validation
- **Form Requests** لكلّ controller action.
- **Strict types**: integers مُتحقَّق منها، dates parsed بـ Carbon، URLs validated.
- **Whitelist > Blacklist**: السمح بقائمة محدّدة من الـ values.

### Output Encoding
- **JSON**: Laravel auto-escapes.
- **HTML emails**: Blade `{{ }}` (auto-escape).
- **PDF certificates**: Whitelist للحروف، لا inline HTML من user input.
- **File names**: sanitized قبل الحفظ (no `../`, no nulls).

### File Uploads
- **Max size**: 10MB افتراضي، 500MB للفيديو، 5MB للصور.
- **Mime sniffing**: استخدام file content، ليس extension.
- **AV scanning**: ClamAV على ملفات user-uploaded في Phase 2+.
- **Storage**: Bunny Storage مع random file names، لا direct execution.

---

## 10. Audit Logs

### ما يُسجَّل دائماً

| Category | Examples |
|---|---|
| **Auth events** | login, logout, MFA setup/disable, password change |
| **User changes** | role assignment, permission grant |
| **Sensitive data** | view of student PII by admin, export operations |
| **Money** | order, payment, refund, payout |
| **Tenant lifecycle** | provision, suspend, plan change |
| **Configuration** | settings change, feature flag toggle |

### Schema
موثَّق في `04-database-schema.md §audit_logs`.

### Immutability
- لا UPDATE/DELETE على `audit_logs` و `audit_logs_central`.
- Postgres triggers تمنع المباشر.
- Backup منفصل لـ audit logs بـ retention 7 سنوات.

### Access
- View: Super Admin + Owner (per-tenant).
- Export: Owner فقط، مُسجَّل كـ audit entry.

---

## 11. Rate Limiting & Anti-Abuse

### Limits معرّفة في [`05-api-design.md §11`](05-api-design.md).

### Anti-Abuse Patterns
- **Signup throttle**: 3 signups/hour per IP.
- **OTP throttle**: 3 OTP/hour per phone، 30/day.
- **Password reset throttle**: 5/hour per email.
- **Coupon brute-force**: 10 attempts/hour per user.
- **Free plan abuse**: max 2 tenants per email/phone.

### Bot Detection
- **CAPTCHA** على signup + login بعد 3 failed.
- **Suspicious patterns** → review queue:
  - Many enrollments من same IP.
  - Rapid-fire API calls.
  - User-Agent anomalies.

### Vercel BotID (Phase 2)
لو احتجنا حماية أعمق على marketing pages.

---

## 12. Payment Security (PCI-DSS scope)

### ما نتجنّبه
- ❌ ما نخزّن أرقام بطاقات.
- ❌ ما نَستلم CVV.
- ❌ ما نعالج بطاقات server-side.

### كيف نقلّل scope
- **Tap Hosted Page**: المستخدم يدخل بطاقته في صفحة Tap، نَستلم token فقط.
- **Tamara**: redirect kompletely، nothing touches our server.
- **Apple Pay**: tokenized، يمرّ عبر Tap.

### نتيجة
PCI-DSS scope = **SAQ A** (الأبسط) — نملأ self-assessment سنوياً، لا audit ميداني.

### حقول البنوك للـ instructor payouts
- مُشفَّرة بـ AES-256-GCM في `affiliate_partners.bank_details_encrypted`.
- مفاتيح في Vault.
- يُكشَف فقط عند payout processing.

---

## 13. PDPL Compliance

### المعايير الأساسية (السعودية، Saudi Personal Data Protection Law)

| المطلَب | كيف نلبّيه |
|---|---|
| **Consent** للـ marketing | `user.marketing_opt_in` صريح، opt-in default false |
| **Data minimization** | نَجمع فقط الضروريّ |
| **Purpose limitation** | كلّ بيانات لها purpose موثَّق في privacy policy |
| **Right to access** | `GET /v1/me/data-export` يَنتج ZIP بكلّ بياناتك |
| **Right to rectify** | UI لتعديل بياناتك |
| **Right to erase** | `DELETE /v1/me/account` → 30-day grace → hard delete |
| **Data residency** | KSA tenants: بياناتهم في KSA PoP حصراً (Bunny KSA + Railway region) |
| **Cross-border transfer** | اتفاقية SCC مع Anthropic/OpenAI، disclosed في privacy policy |
| **DPO** | Data Protection Officer مُعيَّن، contact في footer |
| **Breach notification** | 72 ساعة للسلطة + للمتأثّرين |

### Privacy Policy
- موجود في `wamadat.io/privacy` بـ ar/en.
- يَذكر: ما نَجمع، لماذا، مدة الحفظ، حقوقك.
- آخر تحديث + version.

### Children
- منع التسجيل للأقلّ من 13 سنة.
- 13–18: تتطلّب موافقة ولي الأمر (Phase 2).

---

## 14. ZATCA Phase 2 Compliance

### المتطلَّبات
1. كلّ فاتورة B2C/B2B يُولَّد لها XML بـ UBL 2.1 format.
2. Cryptographic stamp بـ certificate من ZATCA.
3. QR code على PDF يحوي tax data.
4. إرسال للـ ZATCA portal خلال 24h.

### Integration
- مكتبة: `salla/zatca-php` أو custom.
- Test endpoint قبل go-live.
- شهادة Production certificate من ZATCA portal.

### Failure Modes
- ZATCA رفض → invoice في حالة `zatca_rejected` → manual review + resubmit.
- ZATCA down → queue retry مع backoff.
- Invoice غير قابل لـ submission → flag للـ finance team.

---

## 15. GDPR

للمستأجرين/المتعلِّمين خارج KSA (مصر، الأردن، الإمارات لها قوانين مشابهة).

### أساسيّاً مماثل لـ PDPL مع:
- **Cookie consent banner** للزائرين من EU/UK.
- **Data Processing Agreement (DPA)** مع كلّ tenant Enterprise.
- **Sub-processors list** publicly disclosed.

---

## 16. Vulnerability Management

### Dependency Scanning
- **Dependabot** على GitHub: PR تلقائياً لكلّ vulnerability.
- **composer audit** + **npm audit** في CI.
- **Snyk** أو **GitGuardian** للـ secrets في commits.

### CVE Response SLA
| Severity | SLA |
|---|---|
| Critical (CVSS 9–10) | 24 ساعة |
| High (CVSS 7–8.9) | 7 أيام |
| Medium (CVSS 4–6.9) | 30 يوماً |
| Low | next minor release |

### Penetration Testing
- **Yearly** external pentest (Phase 2+).
- **Quarterly** internal scan (OWASP ZAP، Burp).
- **Bug bounty** على HackerOne (Phase 3).

---

## 17. Incident Response

### Severity Levels
| Sev | معنى | Response Time |
|---|---|---|
| Sev-1 | data breach، production down | < 15 min |
| Sev-2 | major feature broken | < 1 hour |
| Sev-3 | minor issue، workaround موجود | < 24 hour |
| Sev-4 | improvement | sprint planning |

### Process
1. **Detect** — alerts من Sentry, Better Stack, user reports.
2. **Triage** — على-call engineer يَفحص خلال 15 min.
3. **Contain** — يحدّ من الانتشار (rotate keys, block IPs).
4. **Eradicate** — الجذر، deploy fix.
5. **Recover** — يَستعيد الخدمة.
6. **Post-mortem** — blameless writeup خلال 5 أيام.

### Communication
- Status page: `status.wamadat.io`.
- Tenant notification بـ email + in-app banner.
- في حالة Data Breach: 72h notification لكلّ متأثِّر.

---

## 18. Security Testing

### في الـ CI/CD
- **Static Analysis**: PHPStan level 8، ESLint strict، Pint.
- **Dependency Scan**: composer + npm audit.
- **Secret Scan**: GitGuardian/TruffleHog.
- **SAST**: Semgrep rules.

### Manual
- **Code review** بـ checklist (انظر `08-coding-standards.md`).
- **Security review** للـ PRs على auth/payment/tenancy.

### Continuous
- **OWASP ZAP** scheduled scans أسبوعياً.
- **Lighthouse Security** على frontend.

---

## ADRs الأمنية في هذا المستند

| # | القرار |
|---|---|
| **ADR-019** | argon2id فقط للـ password hashing |
| **ADR-020** | PCI scope: SAQ A عبر hosted pages (Tap/Tamara) |
| **ADR-021** | Tenant isolation: schema + app-level + audit (3 طبقات) |
| **ADR-022** | DPO + 72h breach notification + data export endpoint مطبَّق من اليوم الأوّل |

---

<sub>**النسخة**: 1.0 · **آخر مراجعة**: 2026-05-11 · **التالي**: pentest في Phase 2</sub>
