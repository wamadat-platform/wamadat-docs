# API v1 — Signed media URLs

**Authority:** how the API serves access-controlled file downloads — specifically `lesson_resources.file_url` (lesson attachments PDFs/decks).

**Captured:** 2026-05-13 (CP4-11). Commit anchor: `<filled at commit time>`.

---

## The problem before CP4-11

`LessonResource::toArray()` used to emit:

```jsonc
{
  "resources": [
    { "id": "...", "name": "Slides", "file_url": "lesson-resources/abc.pdf", ... }
  ]
}
```

That `file_url` was a path on `disk('public')`, which Laravel resolves to a publicly-addressable URL via the `storage/` symlink. Three problems:
1. **No auth gate** — anyone with the URL could download forever.
2. **No expiry** — links shared via screenshot or forwarded never died.
3. **No un-enrollment defense** — a student who un-enrolled (or had their account suspended) could still hit URLs they previously saw.

This is a real leak. Curriculum is enrollment-gated, but the URLs it exposes weren't.

---

## The shape after CP4-11

`LessonResource::toArray()` now emits:

```jsonc
{
  "resources": [
    {
      "id": "...",
      "name": "Slides",
      "download_url": "https://api.wmt.sa/api/v1/media/lesson-resources/<id>?expires=...&user=<userId>&signature=<hex>",
      ...
    }
  ]
}
```

- `file_url` is GONE (BREAKING for any client reading it). Clients use `download_url` instead.
- The URL is a Laravel temporarySignedRoute — 15-minute expiry, HMAC-signed against `APP_KEY`.
- The URL embeds the requesting user's id as a query parameter (`user=`) which is part of the signature.

External-link resources (admin-entered full `http(s)://` URLs in the Filament resource form) pass through unchanged — they were never on our storage and shouldn't be wrapped.

---

## How the gate works

`MediaController::lessonResource()` enforces, in order, before streaming the file:

1. **`signed` route middleware** validates the HMAC + expiry. Tampered signatures or expired URLs throw `InvalidSignatureException`. `ApiErrorRenderer` maps these to:
    ```json
    { "error": { "code": "PERMISSION_DENIED", "message": "...", "status": 403, "details": null } }
    ```

2. **`auth:sanctum` route middleware** requires a valid bearer token. No Authorization header → 401 UNAUTHENTICATED.

3. **`active.account` middleware** rejects suspended/locked/deleted users.

4. **Controller-level user binding** — `$request->query('user')` must equal `$request->user()->id`. A signed URL minted for User A forwarded to User B's session → 403 PERMISSION_DENIED.

5. **Controller-level enrollment re-check** — `EnrollmentService::findForUser($user, $lesson->program_id)` must return non-null. Un-enrollment mid-window → 403 NOT_ENROLLED.

6. **File existence on disk** — `Storage::disk('public')->exists($row->file_url)` must be true. Otherwise 404 RESOURCE_NOT_FOUND.

7. **Stream the file** with:
    - `Content-Type: <stored mime>`
    - `Content-Disposition: inline; filename="..."`
    - `Cache-Control: private, no-store, max-age=0` — no proxy or browser may cache.

The default URL TTL is `MediaController::SIGNATURE_TTL_MINUTES = 15`. Short enough that a leaked URL expires fast; long enough that a slow mobile connection can complete the download.

The endpoint is throttled at `throttle:60,1` per user.

---

## Live proof bundle (CP4-11 baseline)

Seven scenarios were run against `127.0.0.1:8765`:

| # | Scenario | URL | Auth header | Expected | Result |
|---|---|---|---|---|---|
| 1 | valid signed URL + enrolled user token | fresh | enrolled | 200 + `application/pdf` body | ✓ |
| 2 | valid signed URL + NO header | fresh | _none_ | 401 UNAUTHENTICATED | ✓ |
| 3 | valid signed URL + DIFFERENT user's token | fresh | other | 403 PERMISSION_DENIED (user mismatch) | ✓ |
| 4 | EXPIRED signed URL | tampered `expires=` | enrolled | 403 PERMISSION_DENIED (invalid sig) | ✓ |
| 5 | TAMPERED signature | last hex digit flipped | enrolled | 403 PERMISSION_DENIED | ✓ |
| 6 | URL signed for User B + User A's token | minted for B | A | 403 PERMISSION_DENIED (user mismatch) | ✓ |
| 7 | URL signed for User B (un-enrolled) + User B's token | minted for B | B | 403 NOT_ENROLLED | ✓ |

The 4 + 5 responses carry the AR-localized `error.message` via `Accept-Language`. Sending `Accept-Language: en` returns `"You do not have permission for this action."` instead.

These scenarios are encoded as Pest tests in `backend/tests/Feature/Tenant/MediaSignedUrlTest.php` for regression protection (will execute once the tenant test bootstrap regression — R-OPEN-31 — is fixed in CP4-12).

---

## Client integration

```ts
// 1. Fetch curriculum
const res = await fetch('/api/v1/learn/programs/<slug>', {
  headers: { 'Authorization': `Bearer ${access}`, 'X-Tenant-Slug': 'wamadat' }
});
const { data } = await res.json();

// 2. For a lesson resource, the URL is ready to use as-is
const resource = data.modules[0].lessons[0].resources[0];
// resource.download_url is a fully-formed URL with expiry + signature

// 3. Download — MUST include the SAME bearer token (user binding is verified)
const file = await fetch(resource.download_url, {
  headers: { 'Authorization': `Bearer ${access}`, 'X-Tenant-Slug': 'wamadat' }
});
const blob = await file.blob();
```

Notes:
- The download_url has a 15-minute lifetime from when the curriculum was fetched. Cache aware clients refresh by re-fetching the curriculum.
- Don't share `download_url` across users — the receiving user has a DIFFERENT bearer token, and the user-binding check will reject (403 PERMISSION_DENIED).

---

## What's NOT covered by CP4-11

Deliberately out-of-scope (deferred or N/A):

| Surface | Status | Why |
|---|---|---|
| **Program covers** (`cover_image_url`) | Public, no signing needed | Intentional marketing asset. |
| **Lesson videos** (`media.provider` + `media.external_id`) | Not URLs — opaque IDs | Streaming handled by Bunny/Mux. Their own signed-token endpoint is a separate CP. |
| **Avatar URLs** | External (Gravatar etc.) | User-supplied full URLs, never our storage. |
| **Certificate PDFs** | Controller-served only | `/me/certificates/{id}/download` is already auth + ownership-gated. Shareable signed-URL flow (employer verification) is a future CP. |
| **Assignment submission files** | Frontend → S3 direct | Backend stores URLs in JSON; not yet emitted in API response. Will be tackled when the grader view ships. |
| **Filament admin uploads** | Behind admin auth | Internal staff URLs are outside the public API contract. |

---

## Smoke check

Steps to verify CP4-11 hasn't regressed:

```bash
host=http://127.0.0.1:8765
tenant='X-Tenant-Slug: wamadat'

# 1. Get the curriculum — confirm download_url is in the resource shape and file_url is NOT.
ACCESS=...   # bearer for an enrolled user
curl -s "$host/api/v1/learn/programs/<slug>" \
  -H "Authorization: Bearer $ACCESS" -H "$tenant" \
  | jq '.data.modules[0].lessons[0].resources[0] | keys'
# Expected to include:  "download_url"
# Expected to NOT include:  "file_url"

# 2. Hit the URL with the user's token → 200 + PDF.
URL=$(curl -s "$host/api/v1/learn/programs/<slug>" -H "Authorization: Bearer $ACCESS" -H "$tenant" \
       | jq -r '.data.modules[0].lessons[0].resources[0].download_url')
curl -s -o /tmp/pdf -w "%{http_code} %{content_type}\n" \
  "$URL" -H "Authorization: Bearer $ACCESS" -H "$tenant"
# Expected: 200 application/pdf

# 3. Hit the URL without auth → 401.
curl -s -w "%{http_code}\n" "$URL" -H "$tenant" -o /dev/null
# Expected: 401

# 4. Tamper the signature → 403.
TAMPERED=$(echo "$URL" | sed -E 's/(signature=[a-f0-9]+)[a-f0-9]/\1z/')
curl -s -w "%{http_code}\n" "$TAMPERED" -H "Authorization: Bearer $ACCESS" -H "$tenant" -o /dev/null
# Expected: 403
```

If any step regresses, the contract has broken — open a CHANGELOG-v1.md entry.

---

## Adding signed URLs to a new media surface

When a future endpoint exposes another disk-backed media field:

1. Decide the scope: is it owner-only (certificates) or enrollee-only (lesson resources)?
2. Add a new route `GET /api/v1/media/<entity>/{id}` with `['signed', 'auth:sanctum', 'active.account', 'throttle:N,M']`.
3. Build a sibling to `MediaController::lessonResource()` that:
    - Re-validates the signature.
    - Verifies the `user=` query param matches `$request->user()->id`.
    - Re-checks the scope (ownership / enrollment / …).
    - Re-checks the disk file exists.
    - Streams with `Cache-Control: private, no-store`.
4. Replace the bare URL in the JSON resource with a call to a static `signedUrlForX($model, $userId)` helper on the controller.
5. Add a row to `docs/api/CHANGELOG-v1.md`.
6. Add a Pest test case to `MediaSignedUrlTest.php`.

Never expose a raw `disk('public')` path for content that's supposed to be access-controlled. The storage symlink is permanent and the file is reachable forever.
