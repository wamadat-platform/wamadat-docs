# API v1 — Pagination Contract

**Authority:** This document describes the response envelope shape for every list-returning endpoint in `/api/v1/*`. Mobile/SDK clients should read this to know what `meta` to expect per endpoint.

**Captured:** 2026-05-14 (CP4-4). Commit anchor: `<filled at commit time>`.

---

## Three tiers (light-touch by design)

Forcing every endpoint to use the same pagination shape would balloon the API surface and produce gratuitous churn. Instead, list endpoints fall into one of three tiers based on whether they NEED pagination:

### Tier 1 — Offset paginated

Endpoints that return potentially unbounded data sets. Use the `PaginatedCollection` envelope.

| Endpoint | Per-page default | Per-page max |
|---|---|---|
| `GET /api/v1/catalog/programs` | 12 | 48 |
| `GET /api/v1/catalog/instructors` | 24 | 48 |

**Response shape (frozen for v1):**
```json
{
  "data": [ /* item objects */ ],
  "meta": {
    "current_page": 1,
    "per_page": 12,
    "total": 30,
    "last_page": 3,
    "has_more": true
  }
}
```

**Client contract:**
- `data[]` is the items on the current page.
- `current_page` and `last_page` are 1-indexed.
- `has_more` is the boolean derivative of `current_page < last_page` — the only thing infinite-scroll clients need.
- `?page=N` and `?per_page=N` are the query parameters. `per_page` is clamped server-side to the endpoint's max.
- **No URL fields.** Laravel's default `links` block + `meta.links[]` + `meta.path` are stripped to avoid leaking server URLs and HTML entities into the JSON.

### Tier 2 — Full collection with informational meta

User-scoped lists where the total set is naturally bounded (one user has ~5-20 orders, ~30 notifications, etc.). The endpoint returns the COMPLETE collection. `meta` carries domain-specific counters that the UI uses for badges or filters — NOT pagination cursors.

| Endpoint | `meta` shape |
|---|---|
| `GET /api/v1/me/notifications` | `{ unread_count }` |
| `GET /api/v1/me/orders` | `{ total, paid_count }` |
| `GET /api/v1/me/wishlist` | `{ total }` |
| `GET /api/v1/me/sessions` | `{ total, current_session_id }` |
| `GET /api/v1/me/assignments` | `{ open, submitted, graded }` |
| `GET /api/v1/me/live-sessions` | `{ upcoming, ended }` |
| `GET /api/v1/me/enrollments` | `{ ... }` (counts) |
| `GET /api/v1/me/certificates` | (no meta yet — see Tier 3 note) |

**Client contract:**
- `data[]` is the complete collection. No client-side concatenation across pages required.
- `meta.*` keys vary per endpoint. Each endpoint documents its `meta` shape in `docs/api/openapi.yaml` (CP4-7 deliverable).
- The CLIENT does NOT page-paginate Tier 2 — if it ever needs to, the endpoint graduates to Tier 1 in v2 (NEVER retrofitted on v1).

### Tier 3 — Bounded list, no meta

Small, naturally-bounded lists where neither pagination nor counters add value.

| Endpoint | Notes |
|---|---|
| `GET /api/v1/catalog/categories` | ~6-15 categories per tenant; full list always returned. |
| `GET /api/v1/me/certificates` | A user's own certificates; bounded by their enrollment count. |

Shape: `{ "data": [...] }` only.

---

## Future graduation rule

If a Tier 2 endpoint ever needs pagination (e.g. `/me/notifications` crosses 100 rows per user), it ships as **cursor-paginated** in `v2`, never offset-paginated in `v1`. The `meta` shape would then become `{ next_cursor, has_more }` and the catalogue would have a new tier.

Why cursor over offset for `/me/*`: those lists order by `created_at DESC` and grow continuously. Offset pagination on a growing list produces duplicate-or-missing items as new rows arrive between pages (the classic "infinite scroll feed bug"). Cursor avoids this.

---

## Smoke check (light-touch contract proof)

```bash
$ host=http://127.0.0.1:8765
$ tenant='-H "X-Tenant-Slug: wamadat"'

# Tier 1 — programs (paginated)
curl -s "$host/api/v1/catalog/programs?per_page=2&page=2" $tenant \
  | node -e "let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const j=JSON.parse(d);console.log('keys:',Object.keys(j));console.log('meta:',JSON.stringify(j.meta));})"
# Expected:
#   keys: ['data', 'meta']
#   meta: {"current_page":2,"per_page":2,"total":30,"last_page":15,"has_more":true}

# Tier 1 — last page sets has_more=false
curl -s "$host/api/v1/catalog/programs?per_page=10&page=3" $tenant | jq '.meta'
# Expected: { "current_page": 3, "per_page": 10, "total": 30, "last_page": 3, "has_more": false }

# Tier 2 — /me/notifications carries its domain-specific meta
# (login first to get an access token, then…)
curl -s "$host/api/v1/me/notifications" -H "Authorization: Bearer $ACCESS" $tenant | jq '.meta'
# Expected: { "unread_count": <N> }

# Tier 3 — categories has no meta
curl -s "$host/api/v1/catalog/categories" $tenant | jq 'keys'
# Expected: ["data"]
```

If any of these drift unexpectedly, the contract has changed — open a CHANGELOG-v1.md entry.

---

## Add a new list endpoint

1. Decide which tier it belongs to:
    - Unbounded (catalog, public listings, audit logs) → Tier 1.
    - User-scoped + bounded by domain → Tier 2.
    - Small fixed set → Tier 3.
2. **Tier 1:** call `->paginate()` and return `PaginatedCollection::ofItem(ItemResource::class, $paginator)`.
3. **Tier 2:** return `{ data: [...], meta: { /* domain-specific */ } }` via a direct `response()->json()` or a custom `Resource`.
4. **Tier 3:** return `{ data: [...] }` only.
5. Add the endpoint to the table in this doc before merging.

Don't invent a fourth shape.
