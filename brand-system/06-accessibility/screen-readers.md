# 06.3 — قَوارئ الشاشَة (Screen Readers)

> مَن لا يَرى الشاشَة يَستَحقّ تَجرِبَة مُتَكامِلَة. semantic HTML أَوَّلاً، ARIA ثانياً.

---

## Semantic HTML

استَخدِم العَلامات الصَحيحَة دائماً:

| الاستخدام | العَلامَة الصَحيحَة |
|---|---|
| زِرّ | `<button>` |
| رابِط | `<a href="...">` |
| نَموذَج | `<form>` |
| Field label | `<label for="id">` |
| Section heading | `<h1>` - `<h6>` (بالترتيب) |
| Navigation | `<nav>` |
| Main content | `<main>` |
| Article | `<article>` |
| Aside | `<aside>` |
| Footer | `<footer>` |
| Header | `<header>` |
| List | `<ul>` / `<ol>` / `<dl>` |

---

## ARIA Attributes (عند الحاجَة فقط)

### الأَكثَر شِيوعاً
| Attribute | متى | مَثَل |
|---|---|---|
| `aria-label` | عُنصُر بدون نَصّ مَرئيّ | `<button aria-label="إغلاق"><X /></button>` |
| `aria-labelledby` | عُنصُر مُرتَبِط بِنَصّ آخَر | `<section aria-labelledby="h1-id">` |
| `aria-describedby` | إضافَة وَصف | `<input aria-describedby="help-text">` |
| `aria-expanded` | accordion/menu state | `<button aria-expanded="true">` |
| `aria-hidden="true"` | إخفاء عن screen reader | `<svg aria-hidden="true">` (decorative icons) |
| `aria-live="polite"` | تَحديثات ديناميكيّة | toasts، notifications |
| `aria-current="page"` | الرابط النَشِط | `<a aria-current="page">` |
| `aria-required="true"` | حَقل مَطلوب | `<input aria-required="true">` |
| `aria-invalid="true"` | حَقل به خَطَأ | `<input aria-invalid="true">` |
| `role="alert"` | إعلان عاجِل | `<div role="alert">` |
| `role="status"` | تَحديث غَير عاجِل | `<div role="status">` |

---

## أنماط شائِعَة

### زِرّ بدون نَصّ
```tsx
<button aria-label="إغلاق القائِمَة">
    <X className="size-5" aria-hidden="true" />
</button>
```

### Toast (live region)
```tsx
<div role="status" aria-live="polite" className="sr-only">
    {message}
</div>
```

### Tabs
```tsx
<div role="tablist">
    <button role="tab" aria-selected={isActive} aria-controls="panel-1">
        تَبويب 1
    </button>
</div>
<div role="tabpanel" id="panel-1">...</div>
```

### Form with error
```tsx
<label htmlFor="email">البَريد الإلكترونيّ</label>
<input
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={!!error}
    aria-describedby="email-error"
/>
{error && (
    <p id="email-error" role="alert" className="text-red-600 text-xs">
        {error.message}
    </p>
)}
```

### Loading state
```tsx
<div aria-busy="true" aria-live="polite">
    <Spinner />
    <span className="sr-only">جارٍ التَحميل…</span>
</div>
```

---

## Visually Hidden (sr-only)

نَصّ مَرئيّ لِقَارئ الشاشَة فَقَط:

```tsx
<span className="sr-only">تَحَقَّق من تَأكيد الـ 2FA</span>
```

Tailwind utility `sr-only`:
```css
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}
```

---

## Common Mistakes

❌ `<div role="button" onClick={...}>` — استَخدِم `<button>`
❌ `aria-label` على عُنصُر له نَصّ — تَكرار
❌ `aria-hidden="true"` على عُنصُر focusable — يُسَبِّب bug
❌ نَسيان `role="alert"` على إشعارات حَرِجَة

---

## Test

### اختَبر مع NVDA (مَجّاناً، Windows)
1. تَحميل من https://www.nvaccess.org/
2. شَغِّل + افتَح المَوقع
3. Tab خِلال كلّ شَيء
4. تَأَكَّد من أنّ كلّ عُنصُر يُقرَأ بشَكل واضِح

### اختَبر مع VoiceOver (Mac)
1. Cmd+F5 لِتَفعيل
2. Control+Option+Arrows للتَنَقُّل
