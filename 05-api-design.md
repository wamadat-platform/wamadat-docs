# 05 — API Design

> **مبادئ تصميم الـ API** للمنصّة. يَحكم كلّ endpoint سيُكتَب في الـ Laravel backend.

---

## 📑 الفهرس

1. [الفلسفة العامّة](#1-الفلسفة-العامة)
2. [نمط الـ API: REST + GraphQL مُختار](#2-نمط-الـ-api-rest--graphql-مختار)
3. [Versioning](#3-versioning)
4. [Authentication & Authorization](#4-authentication--authorization)
5. [Tenant Routing](#5-tenant-routing)
6. [Resource Naming & URL Structure](#6-resource-naming--url-structure)
7. [HTTP Methods & Status Codes](#7-http-methods--status-codes)
8. [Request/Response Format](#8-requestresponse-format)
9. [Error Format](#9-error-format)
10. [Pagination & Filtering](#10-pagination--filtering)
11. [Rate Limiting](#11-rate-limiting)
12. [Idempotency](#12-idempotency)
13. [Webhooks](#13-webhooks)
14. [API Documentation](#14-api-documentation)
15. [Deprecation Policy](#15-deprecation-policy)

---

## 1. الفلسفة العامّة

- **Consistency > Cleverness** — endpoint واحد متوقَّع أحسن من ميزة متطوّرة.
- **Self-documenting** — JSON يكشف عن نفسه (no cryptic codes).
- **Stable by default** — تغيير breaking في v2 ليس v1.
- **Tenant-aware always** — كلّ endpoint يعرف tenant context.
- **Failures are first-class** — error format موحَّد، مفصَّل، مفيد.

---

## 2. نمط الـ API: REST + GraphQL مُختار

### القرار

**ADR-018**: REST كأساس لـ 95% من الـ API. GraphQL **فقط** للـ Frontend dashboards التي تجمع بيانات من 3+ resources.

### المبرّر

| المتطلّب | الأنسب |
|---|---|
| Public API (3rd-party developers) | REST |
| Mobile/PWA simple resource access | REST |
| Webhooks (out) | REST POST |
| Dashboard analytics (multi-resource fetch) | GraphQL |
| Instructor Studio overview | GraphQL |
| Real-time subscriptions | WebSocket (Reverb) — ليس GraphQL subscription |

### Implementation
- REST: Laravel Controllers + Form Requests + API Resources
- GraphQL: Lighthouse PHP (schema-first)
- WebSocket: Reverb (Laravel Broadcasting)

---

## 3. Versioning

### URL versioning
```
https://api.wamadat.io/v1/programs
https://api.wamadat.io/v2/programs
```

### قواعد
- **Major version bump** عند breaking changes.
- **Minor changes** (إضافة حقل، endpoint جديد) لا تتطلّب نسخة.
- نُبقي v(N-1) لمدّة **12 شهراً** بعد إصدار vN.
- في كلّ response: `X-API-Version` header.

---

## 4. Authentication & Authorization

### Authentication Methods

| القناة | الآلية | Header |
|---|---|---|
| Frontend SPA | Sanctum cookies (HTTP-only) | تلقائي |
| Mobile/PWA | Bearer token | `Authorization: Bearer <token>` |
| 3rd-party | Personal Access Token | `Authorization: Bearer <token>` |
| Webhooks (in) | HMAC signature | `X-Wamadat-Signature: <hmac>` |
| Service-to-service (AI svc) | mTLS أو shared secret | TLS cert |

### Authorization
- Policies على كلّ Eloquent model.
- Gates للـ ad-hoc rules.
- Permissions check قبل entering UseCase: `Gate::authorize('programs.publish', $program)`.

### Token Scopes (Personal Access Tokens)
```
read:programs
write:programs
read:learners
write:enrollments
admin:tenant
```

---

## 5. Tenant Routing

### Strategy
كلّ request يجب أن يَعرف الـ tenant. ثلاث طرق:

| الأولوية | المصدر | متى |
|---|---|---|
| 1 | Subdomain | الحالة العامّة: `wamadat.platform.io` |
| 2 | Custom domain | Pro+ tenants: `learn.example.com` |
| 3 | `X-Tenant-Id` header | API calls من خارج، super-admin tools |

### Middleware
```php
Route::middleware(['tenant.resolve'])->group(function () {
    // كلّ الـ endpoints الـ tenant-scoped
});
```

---

## 6. Resource Naming & URL Structure

### قواعد
- Plural nouns: `/programs`, `/users`, `/enrollments`.
- Lowercase, kebab-case للأسماء المركّبة: `/live-sessions`, `/coupon-redemptions`.
- Nested resources حدّ أقصى مستويين: `/programs/{id}/modules/{moduleId}/lessons`.
- لا فعل في الـ URL — استخدم HTTP method. (استثناء: actions غير CRUD مثل `POST /programs/{id}/publish`).

### أمثلة
```
GET    /v1/programs                          → list
POST   /v1/programs                          → create
GET    /v1/programs/{id}                     → read
PATCH  /v1/programs/{id}                     → partial update
PUT    /v1/programs/{id}                     → full replace (نادر، PATCH مفضّل)
DELETE /v1/programs/{id}                     → soft delete

POST   /v1/programs/{id}/publish             → action
POST   /v1/programs/{id}/archive             → action
POST   /v1/programs/{id}/duplicate           → action

GET    /v1/programs/{id}/enrollments         → nested list
POST   /v1/programs/{id}/enrollments         → enroll
```

---

## 7. HTTP Methods & Status Codes

| الحالة | الكود | متى |
|---|---|---|
| ✅ Success | 200 OK | GET, PATCH success |
| ✅ Created | 201 Created | POST success مع موارد جديد |
| ✅ Accepted | 202 Accepted | Async action queued |
| ✅ No Content | 204 No Content | DELETE success |
| ⚠️ Bad Request | 400 | Malformed request |
| ⚠️ Unauthorized | 401 | لا token / invalid token |
| ⚠️ Forbidden | 403 | token صحيح، لكن لا صلاحية |
| ⚠️ Not Found | 404 | resource غير موجود |
| ⚠️ Conflict | 409 | duplicate, version conflict |
| ⚠️ Unprocessable | 422 | validation failed |
| ⚠️ Too Many | 429 | rate limit |
| ❌ Server Error | 500 | unexpected |
| ❌ Bad Gateway | 502 | upstream فشل (Tap, AI) |
| ❌ Service Unavailable | 503 | maintenance، الحدّ تجاوز |

---

## 8. Request/Response Format

### Content-Type
- Request: `application/json` (default).
- File uploads: `multipart/form-data` فقط حيث ضروريّ.
- Response: `application/json` دائماً.

### Standard Response (single resource)

```json
{
  "data": {
    "id": "01HX7G3K...",
    "type": "program",
    "attributes": {
      "title_ar": "أساسيات البرمجة",
      "price_sar": 49900,
      "status": "published"
    },
    "relationships": {
      "instructor": { "id": "...", "type": "user" },
      "category": { "id": "...", "type": "category" }
    },
    "links": {
      "self": "/v1/programs/01HX7G3K..."
    }
  },
  "meta": {
    "request_id": "req_..."
  }
}
```

### Standard Response (collection)

```json
{
  "data": [
    { "id": "...", "type": "program", "attributes": { ... } },
    ...
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 145,
      "total_pages": 8
    },
    "request_id": "req_..."
  },
  "links": {
    "first": "/v1/programs?page=1",
    "last": "/v1/programs?page=8",
    "next": "/v1/programs?page=2",
    "prev": null
  }
}
```

### Field Casing
- **snake_case** في JSON (يطابق DB).
- Frontend يحوّل لـ camelCase عند الحاجة.

### Timestamps
- ISO 8601 مع timezone: `2026-05-11T14:30:00+03:00`.
- في DB: TIMESTAMPTZ (UTC) — التحويل لـ user timezone في Resource layer.

### Money
```json
{
  "price": {
    "amount": 49900,
    "currency": "SAR",
    "formatted": "499.00 ر.س"
  }
}
```

Backend دائماً يَستقبل/يُرسل `amount` بـ halalas (cents). `formatted` للـ display only.

---

## 9. Error Format

### Single Error

```json
{
  "errors": [
    {
      "id": "err_01HX...",
      "status": "422",
      "code": "VALIDATION_FAILED",
      "title": "بيانات الطلب غير صالحة",
      "detail": "حقل البريد الإلكتروني مطلوب",
      "source": { "pointer": "/data/attributes/email" },
      "meta": {
        "field_errors": {
          "email": ["مطلوب", "صيغة غير صحيحة"],
          "price": ["يجب أن يكون أكبر من صفر"]
        }
      }
    }
  ],
  "meta": {
    "request_id": "req_..."
  }
}
```

### Error Codes (subset)

| Code | معنى |
|---|---|
| `VALIDATION_FAILED` | 422 — input validation فشل |
| `RESOURCE_NOT_FOUND` | 404 — المورد غير موجود |
| `UNAUTHORIZED` | 401 — لا token صحيح |
| `FORBIDDEN` | 403 — لا صلاحية |
| `RATE_LIMITED` | 429 — تجاوز الحدّ |
| `QUOTA_EXCEEDED` | 402 — تجاوز quota الـ plan |
| `PAYMENT_FAILED` | 402 — رفض البوّابة |
| `WEBHOOK_INVALID_SIGNATURE` | 401 — توقيع غير صحيح |
| `TENANT_SUSPENDED` | 403 — المستأجر معلَّق |
| `IDEMPOTENCY_CONFLICT` | 409 — request مُكرَّر مع payload مختلف |
| `INTERNAL_ERROR` | 500 — لم نتوقّعه (شُحِنَ لـ Sentry) |

### Localization
رسائل الخطأ بـ Accept-Language:
```
Accept-Language: ar
→ "title": "بيانات الطلب غير صالحة"

Accept-Language: en
→ "title": "Invalid request data"
```

---

## 10. Pagination & Filtering

### Pagination
```
GET /v1/programs?page=2&per_page=20
```

| Param | Default | Max |
|---|---|---|
| `page` | 1 | — |
| `per_page` | 20 | 100 |

للـ feeds الطويلة (event logs): **cursor-based pagination**:
```
GET /v1/event-logs?cursor=eyJpZCI6IjEyMyJ9&limit=50
```

### Filtering
```
GET /v1/programs?filter[status]=published&filter[category]=tech&filter[price][gte]=100
```

عوامل المقارنة: `eq` (default), `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `like`.

### Sorting
```
GET /v1/programs?sort=-published_at,title_ar
```
`-` للتنازليّ.

### Sparse Fieldsets
```
GET /v1/programs?fields[program]=title_ar,price,status
```

### Including Relationships
```
GET /v1/programs?include=instructor,category,modules.lessons
```

---

## 11. Rate Limiting

### Default limits

| Endpoint type | Limit | Window |
|---|---|---|
| Public (unauthenticated) | 60 req | 1 minute |
| Authenticated (regular) | 600 req | 1 minute |
| Admin endpoints | 1200 req | 1 minute |
| Auth endpoints (login, signup) | 5 req | 1 minute per IP |
| OTP send (SMS) | 3 req | 1 hour per phone |
| Payment endpoints | 30 req | 1 minute per user |
| AI chat | حسب الـ plan | 1 day |

### Headers في كلّ response
```
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 547
X-RateLimit-Reset: 1717100000
Retry-After: 23 (في 429 فقط)
```

---

## 12. Idempotency

### للـ POST mutations

كلّ POST عمليّاً sensitive (مدفوعات، enrollments) يدعم:

```
POST /v1/orders
Idempotency-Key: ord_attempt_8f3a2b1c
```

- يُخزَّن المفتاح + hash للـ body في Redis لـ 24h.
- إذا تكرّر مع نفس الـ payload → يُعاد الـ response السابق.
- إذا تكرّر مع payload مختلف → 409 `IDEMPOTENCY_CONFLICT`.

---

## 13. Webhooks

### Outbound (نُرسل للمستأجرين/3rd-party)

```
POST <tenant_webhook_url>
Headers:
  X-Wamadat-Event: order.completed
  X-Wamadat-Signature: t=<ts>,v1=<hmac_sha256>
  X-Wamadat-Delivery: <delivery_id>
Body:
  {
    "event": "order.completed",
    "occurred_at": "2026-05-11T...",
    "data": { ... }
  }
```

- **Signature**: `HMAC-SHA256(secret, timestamp + "." + body)`.
- **Retries**: exponential backoff (1m, 5m, 30m, 2h, 1d) — 5 محاولات.
- **Delivery log**: استعلام عبر `GET /v1/webhooks/deliveries`.

### Inbound (Tap, Tamara webhooks)
انظر `02-architecture.md §10`.

---

## 14. API Documentation

### Tools
- **OpenAPI 3.1** spec auto-generated من Laravel routes + Form Requests.
- **Swagger UI** على `/api/docs` (admin only).
- **Redoc** على `/api/docs/public` (للـ partners).
- **Postman Collection** مُصدَّر تلقائياً.

### Per-endpoint توثيق إلزاميّ
- Description (ar/en).
- Auth requirements.
- Request body schema.
- Response schema.
- Possible error codes.
- Example request + response.

---

## 15. Deprecation Policy

### Lifecycle
```
Active → Deprecated → Sunset
  │          │           │
  │          │           └─ Returns 410 Gone
  │          └─ يُرجع X-Deprecation header
  └─ الإصدار الحاليّ
```

### Timing
- إعلان deprecation: 6 أشهر قبل sunset.
- Sunset: لا أقلّ من 12 شهراً بعد إصدار v(N+1).

### Communication
- Headers: `X-Deprecation: This endpoint is deprecated. Use /v2/programs.`
- Email لكلّ tenant يَستخدم الـ endpoint.
- Changelog entries.

---

<sub>**النسخة**: 1.0 · **ADR-018** هو الـ ADR الوحيد الجديد · **OpenAPI Spec** تُولَّد آلياً في PHASE 3</sub>
