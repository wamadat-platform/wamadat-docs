# 08 — Coding Standards

> **معايير الكود الإلزامية.** كلّ PR يُراجَع وفق هذا المستند. الانحراف يَتطلَّب exception موثَّق.

---

## 📑 الفهرس

1. [مبادئ عامة](#1-مبادئ-عامة)
2. [PHP / Laravel](#2-php--laravel)
3. [TypeScript / Next.js](#3-typescript--nextjs)
4. [SQL / Migrations](#4-sql--migrations)
5. [Testing](#5-testing)
6. [Git / PR](#6-git--pr)
7. [Documentation](#7-documentation)
8. [Localization (i18n)](#8-localization-i18n)
9. [Performance](#9-performance)
10. [Tooling](#10-tooling)

---

## 1. مبادئ عامة

1. **Readability > Cleverness**. كود يُقرَأ مرّة، يُعدَّل عشرات المرّات.
2. **Explicit > Implicit**. type hints دائماً، magic لا.
3. **Single Responsibility**. function/class تَفعل شيئاً واحداً.
4. **Fail fast**. validate إدخالات على الحدود (boundaries) لا في الـ deep logic.
5. **No dead code**. غير المُستخدَم يُحذَف.
6. **Comments explain WHY, not WHAT**. الكود يَكفي للـ WHAT.

---

## 2. PHP / Laravel

### Style
- **PSR-12** + Laravel Pint preset.
- **PHP 8.3+** features: readonly properties, enums, first-class callable syntax.
- **strict_types** declared في كلّ file: `declare(strict_types=1);`.

### Naming
| Element | Convention | Example |
|---|---|---|
| Classes | PascalCase | `EnrollmentService` |
| Methods | camelCase | `markAsCompleted()` |
| Properties | camelCase | `$enrolledAt` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_ATTEMPTS` |
| Database tables | snake_case plural | `program_enrollments` |
| Routes | kebab-case | `/live-sessions` |
| Config keys | snake_case | `payment.tap_secret` |

### Module Structure (per Bounded Context)

```
app/
  Modules/
    Learning/
      Application/
        UseCases/
          EnrollLearner.php
        Services/
        DTOs/
      Domain/
        Entities/
          Enrollment.php
        ValueObjects/
          Progress.php
        Events/
          LearnerEnrolled.php
        Repositories/
          EnrollmentRepository.php (interface)
      Infrastructure/
        Persistence/
          EloquentEnrollmentRepository.php
        Listeners/
        Filament/
        Http/
          Controllers/
          Requests/
          Resources/
      Tests/
        Unit/
        Feature/
```

### Eloquent Rules
- Models امتداد `app/Modules/{Context}/Infrastructure/Persistence/Models/`.
- **لا** business logic في Models — فقط relations، casts، scopes.
- **لا** `DB::raw($input)` مع user input.
- Global scopes للـ tenant + soft delete + active.

### Service Pattern
```php
final readonly class EnrollLearner
{
    public function __construct(
        private EnrollmentRepository $enrollments,
        private EventDispatcher $events,
    ) {}

    public function __invoke(EnrollLearnerCommand $command): EnrollmentId
    {
        // 1. validate domain invariants
        // 2. create domain entity
        // 3. persist via repository
        // 4. dispatch domain event
        // 5. return ID
    }
}
```

### Anti-patterns (ممنوعة)
- ❌ Fat controllers (> 30 lines per action).
- ❌ Service classes استدعاء eloquent مباشرة (use repositories).
- ❌ Static facades في Domain layer (allowed في Infra فقط).
- ❌ N+1 queries — Laravel debugbar في dev يَكشفها.

---

## 3. TypeScript / Next.js

### Style
- **TypeScript 5.5 strict mode**: `strict: true`, `noUncheckedIndexedAccess: true`.
- **Prettier** + ESLint (Next.js preset + custom rules).
- **No `any`** — استخدم `unknown` ثمّ narrowing.

### File Organization
```
frontend/
  app/                          # App Router
    (marketing)/                # route groups
    (dashboard)/
    [locale]/                   # i18n routes
  components/
    ui/                         # shadcn primitives
    features/                   # composed feature components
    layouts/
  lib/
    api/                        # API client (typed)
    auth/
    utils/
  hooks/
  types/                        # shared types
  styles/
```

### Component Rules
- **Server Components by default** (Next.js 15 App Router).
- `'use client'` فقط حيث ضروري (state, effects, browser APIs).
- Props دائماً typed بـ interface محدّد.
- لا inline objects كـ props (memory churn).

### Naming
| Element | Convention | Example |
|---|---|---|
| Components | PascalCase | `ProgramCard.tsx` |
| Hooks | camelCase with `use` | `useEnrollment.ts` |
| Utilities | camelCase | `formatCurrency.ts` |
| Constants | SCREAMING_SNAKE | `MAX_PAGE_SIZE` |
| Types/Interfaces | PascalCase | `ProgramDetails` |

### State Management
- **Local UI state**: `useState`.
- **Server state**: TanStack Query.
- **Global UI state**: Zustand (very few stores).
- **Forms**: React Hook Form + Zod.

### Anti-patterns
- ❌ Prop drilling > 2 levels — use composition or context.
- ❌ Effects للـ data fetching في Client Components — use Server Components or TanStack Query.
- ❌ Mixing `'use client'` and heavy data fetching في نفس component.

---

## 4. SQL / Migrations

### Style
- **Lowercase keywords**: `SELECT`, `FROM` لكن في الكود `select`, `from`.
- **Indent consistent**:
```sql
SELECT
    u.id,
    u.email,
    COUNT(e.id) AS enrollments_count
FROM users u
LEFT JOIN enrollments e ON e.user_id = u.id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.email
ORDER BY enrollments_count DESC
LIMIT 10;
```

### Migration Rules
- اسم descriptive: `2026_05_11_120000_add_drip_rule_to_lessons.php`.
- **Reversible** عند الإمكان (down method).
- Comments إذا الـ migration معقّدة.
- **No data manipulation** في schema migration — use separate seeder/job.

### Indexing
- كلّ FK له index (Laravel `foreignId()` تَفعل تلقائياً).
- Composite indexes لـ query patterns الشائعة.
- لا تَزِد indexes "احتياطاً" — كلّ index = كلفة writes.

---

## 5. Testing

### Pyramid
```
        / e2e \                  ~5%  Playwright
       /-------\
      /feature  \                ~25% Pest feature
     /-----------\
    /    unit     \              ~70% Pest unit
   /---------------\
```

### Coverage Targets
- **Domain layer**: 95%+ unit coverage.
- **Application/UseCases**: 90%+ unit coverage.
- **HTTP endpoints**: feature tests للـ happy path + 1–2 edge cases.
- **E2E**: critical user flows (signup, enroll, pay, certificate).

### Test Naming
```php
// Pest style
it('grants course access when payment is captured')
    ->expect(...)
    ->toBe(...);

// أو PHPUnit
public function test_grants_course_access_when_payment_is_captured(): void
```

### Test Data
- **Factories** لكل model.
- لا hard-coded fixtures.
- Tenant context يُهيَّأ في setup.

### CI Requirements
- All tests pass.
- Coverage لا يَنخفض > 2%.
- Mutation testing (Infection) على critical modules — Phase 2.

---

## 6. Git / PR

### Branches
- `main` — production.
- `feature/<short-name>` — features.
- `fix/<short-name>` — bug fixes.
- `chore/<short-name>` — tooling, deps.

### Commits — Conventional Commits
```
feat(learning): add drip content unlock
fix(commerce): handle tamara declined webhook
chore(deps): bump laravel from 11.2 to 11.5
docs(arch): clarify multi-tenancy strategy
test(payment): cover tap signature verification edge case
refactor(catalog): extract program publisher service
```

Types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `security`.

### PR Requirements
- **Title**: Conventional Commit format.
- **Description**: WHAT + WHY (ليس فقط WHAT).
- **Linked issue** أو Linear ticket.
- **Tests** أُضيفت/مُحدَّثة.
- **Docs** مُحدَّثة لو واجهة عامّة تتغيّر.
- **Screenshots** للـ UI changes.
- **Migration plan** لو schema يتغيّر.

### Code Review Checklist
- [ ] الكود يَتبع المعايير في هذا المستند.
- [ ] Tests adequate و pass.
- [ ] لا secrets في الكود.
- [ ] لا N+1 queries.
- [ ] Tenant scoping صحيح.
- [ ] Authorization checks موجودة.
- [ ] Error handling sensible.
- [ ] Logs/metrics adequate.
- [ ] Backward compatible (أو migration plan موثَّق).

### Approval Rules
- **Min 1 reviewer** للـ regular changes.
- **Min 2 reviewers** للـ security/payment/tenancy changes.
- **Author لا يَoperationمoperationerge own PR** (إلا للـ docs).

---

## 7. Documentation

### Code Documentation
- **Doc blocks** للـ public methods (PHPDoc / TSDoc).
- **README.md** في كلّ Module folder يَشرح: purpose، public API، examples.
- **ADRs** في `docs/adrs/` للقرارات الكبرى.

### Comments
- اللغة: **English** (universal).
- ✅ Why مذكور: `// Use ULID for natural ordering across tenants`
- ❌ What مذكور (overkill): `// increment counter`

---

## 8. Localization (i18n)

### Strings
- **لا** hard-coded user-facing strings.
- Laravel: `__('messages.welcome', ['name' => $name])`.
- Next.js: `t('welcome', { name })` عبر next-intl.

### Files
```
backend/lang/
  ar/
    messages.php
    validation.php
  en/
    messages.php
    validation.php

frontend/messages/
  ar.json
  en.json
```

### قواعد
- Keys بـ dot.notation: `programs.created.success`.
- لا تَدمج HTML في translations.
- Plurals صحيحة (العربية لها 6 صيغ).
- Number/date formatting عبر Intl API (لا inline).

---

## 9. Performance

### Database
- لا N+1 (use eager loading).
- Pagination على كلّ list endpoint.
- Cache للـ queries المُكلفة (مع invalidation strategy).

### HTTP
- ETags على responses القابلة للـ cache.
- Compression: Brotli + gzip.
- Pagination: 20 افتراضي، 100 max.

### Frontend
- Image: Next.js `<Image>` فقط، WebP/AVIF.
- Font: Tajawal محلي مع `font-display: swap`.
- Bundle: code-split على route boundaries.
- LCP target: < 2s.
- CLS target: < 0.1.
- INP target: < 200ms.

### Background Jobs
- Long-running operations (> 500ms) → queue.
- Idempotent jobs دائماً.

---

## 10. Tooling

### Backend
- **Laravel Pint** — formatter.
- **PHPStan level 8** + **larastan** — static analysis.
- **Pest** — testing.
- **Rector** — auto refactoring (used carefully).
- **PHP CS Fixer** — رديف لـ Pint للقواعد المتقدّمة.

### Frontend
- **Prettier** + **ESLint** (Next.js + Tailwind).
- **TypeScript** strict.
- **Vitest** — unit tests.
- **Playwright** — e2e.
- **Storybook** للـ component library (Phase 2).

### Cross-cutting
- **EditorConfig** للـ tab/space/line endings.
- **Husky** + **lint-staged** للـ pre-commit hooks.
- **CommitLint** للـ Conventional Commits enforcement.

---

## ADRs الجديدة

| # | القرار |
|---|---|
| **ADR-027** | PHP `declare(strict_types=1)` إلزامي في كلّ file |
| **ADR-028** | TypeScript strict mode + noUncheckedIndexedAccess إلزامي |
| **ADR-029** | Conventional Commits + Husky pre-commit hook |
| **ADR-030** | Min 2 reviewers على security/payment/tenancy changes |

---

<sub>**النسخة**: 1.0 · **Pint preset**: Laravel · **ESLint preset**: next + tailwind + simple-import-sort</sub>
