# API v1 — Machine-readable docs (Scribe + Postman)

**Authority:** how the OpenAPI YAML, Postman collection, and HTML reference for `/api/v1/*` are generated, consumed, and refreshed.

**Captured:** 2026-05-13 (CP4-7 + CP4-8). Commit anchor: `<filled at commit time>`.

---

## What's in the box

Running `php artisan scribe:generate` from `backend/` produces three artifacts under `backend/public/docs/`:

| File | Format | Use it for |
|---|---|---|
| `openapi.yaml` | OpenAPI 3.0.3 | Mobile SDK codegen (openapi-generator, swagger-codegen), CI contract validators, type-safe client generation |
| `collection.json` | Postman Collection v2.1.0 | Postman / Insomnia import; manual API exploration; QA scripts |
| `index.html` (+ `css/`, `js/`, `images/`) | Static HTML reference | Human-readable browsable docs |

The first two are the **machine-readable contract**. Everything mobile/SDK clients need is in `openapi.yaml`.

The artifact count today: **90 operations** across **55 paths** under `/api/v1` (`/health` + `/webhooks/*` are intentionally excluded — see `config/scribe.php`).

---

## How to regenerate

```bash
cd backend
php artisan scribe:generate
```

Re-run after **any** of:
- A new route is added to `routes/api.php`.
- A FormRequest's `rules()` change (the request schema in the spec lives there).
- A controller's docblock summary/description changes.
- The error-codes catalogue (`docs/api/04-error-codes.md`) changes (the canonical envelope is documented in `intro_text` and referenced from there).

The generation is fast (~5 seconds for the full surface). Commit the diff alongside the code change.

> **Cache:** Scribe writes intermediate extraction data to `backend/.scribe/`. That directory is gitignored; the committed outputs are the three files above only.

---

## Importing the Postman collection

1. Open Postman → **Import** → **File** → select `backend/public/docs/collection.json`.
2. After import, set the collection variable `baseUrl` to one of:
    - `https://api.wmt.sa` (prod)
    - `https://api.staging.wmt.sa` (staging)
    - `http://127.0.0.1:8765` (local dev — Laravel artisan serve)
3. Set the collection variable `Authorization` to `Bearer <your-token>` (obtain via `POST /api/v1/auth/login`).
4. Every request already carries the `X-Tenant-Slug: wamadat` and `Accept-Language: ar` headers — change either per request as needed.

The collection is a **single tag group** (`Endpoints`) because controllers don't yet carry `@group` PHPDoc annotations. The OpenAPI spec has the same flat shape. Mobile SDK consumers should branch on the path / `operationId`, not on grouping.

---

## Importing the OpenAPI spec

Pick whichever generator matches the client:

```bash
# TypeScript fetch (e.g. for the Next.js frontend)
npx openapi-typescript backend/public/docs/openapi.yaml -o frontend/src/lib/api/types.gen.ts

# Dart / Flutter (mobile)
openapi-generator-cli generate -i backend/public/docs/openapi.yaml -g dart -o mobile/lib/api

# Swift (iOS)
openapi-generator-cli generate -i backend/public/docs/openapi.yaml -g swift5 -o mobile-ios/Sources/API
```

The base URL in the spec is `https://api.wmt.sa`. Override per-environment in the generator settings.

---

## Today's limitations (light-touch tier, deferred)

The first generation pass focused on capturing the **endpoint surface + request schemas + auth model**. The following are known weaknesses, tracked as follow-ups rather than blocking soft launch:

1. **`responses: {}` is empty on every operation.** Scribe couldn't auto-extract response shapes without `@response` PHPDoc annotations or API Resource classes used everywhere. The canonical envelope is documented at the meta level in `intro_text`, plus in:
    - `docs/api/02-response-shape-rebuild.md` — success + error envelope.
    - `docs/api/04-error-codes.md` — closed `code` vocabulary.
    - `docs/api/06-pagination.md` — list-envelope tiers.
   Filling per-endpoint `@response` blocks is **R-OPEN-30** (see `docs/audit/OPEN_RISKS.md`).

2. **Empty `summary` / `description` on each operation.** Controllers don't carry `@summary` / `@description` docblocks. Same follow-up sweep as R-OPEN-30.

3. **One default `Endpoints` group.** Controllers don't carry `@group`. Adding `@group Auth`, `@group Catalog`, `@group Learn`, etc. would improve the HTML reference but doesn't change the contract.

These three don't affect the contract that mobile SDKs consume — request paths, methods, URL/header/body parameters, and auth scheme are all captured correctly.

---

## Smoke check (post-regeneration)

```bash
cd backend
php artisan scribe:generate

# 1. Confirm the OpenAPI YAML parses + count operations
php -r '
  $d = file_get_contents("public/docs/openapi.yaml");
  echo "title: " . (preg_match("/title: (.+)/", $d, $m) ? $m[1] : "MISSING") . PHP_EOL;
  preg_match_all("/operationId:/", $d, $m);
  echo "operations: " . count($m[0]) . PHP_EOL;
'
# Expected:
#   title: \"Wamadat Academy API v1\"
#   operations: 90

# 2. Confirm the Postman collection imports
php -r '
  $d = json_decode(file_get_contents("public/docs/collection.json"), true);
  echo "name: " . $d["info"]["name"] . PHP_EOL;
  function count_all($arr){$n=0;foreach($arr as $i){if(isset($i["request"]))$n++; if(isset($i["item"])) $n+=count_all($i["item"]);} return $n;}
  echo "operations: " . count_all($d["item"]) . PHP_EOL;
'
# Expected:
#   name: Wamadat Academy API v1
#   operations: 90
```

If `operations` drifts from 90 without a corresponding change in `routes/api.php`, the contract has regressed. Investigate.
