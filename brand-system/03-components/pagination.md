# 03.10 — Pagination

> التَنَقُّل بَين صَفحات النَتائج.

---

## Variants

| Variant | متى |
|---|---|
| `Numbered` | للقَوائم القَصيرَة-المُتَوَسِّطَة (< 50 صَفحَة) |
| `Prev/Next only` | الافتراضيّ ⭐ |
| `Infinite scroll` | للموبايل / Feed |
| `Load more button` | بَديل للـ infinite |

---

## Pattern: Prev/Next

```tsx
<div className="flex items-center justify-center gap-2">
    {current > 1 && (
        <a href={pageUrl(current - 1)} className="px-4 py-2 rounded-lg border border-jet-200 hover:bg-jet-50 text-sm">
            <ChevronRight className="size-4 rtl:rotate-180" />
            السابق
        </a>
    )}
    <span className="px-4 py-2 text-sm text-jet-600">
        صَفحَة <span className="font-bold tabular-nums">{current}</span> من <span className="font-bold tabular-nums">{last}</span>
    </span>
    {current < last && (
        <a href={pageUrl(current + 1)} className="px-4 py-2 rounded-lg border border-jet-200 hover:bg-jet-50 text-sm">
            التالي
            <ChevronLeft className="size-4 rtl:rotate-180" />
        </a>
    )}
</div>
```

---

## Pattern: Numbered

```
[‹ السابق]  [1] [2] [3] ... [10]  [التالي ›]
```

- Active page: `bg-jet-900 text-white`
- Inactive: `text-jet-700 hover:bg-jet-50`
- Ellipsis (...): `text-jet-400`

---

## القَواعِد

✅ **Show total** أو **current/total** — المُستَخدِم يَعرِف أين هو.
✅ **Disable الأزرار في الأطراف** (page 1 → no Prev).
✅ **URL يَتَحَدَّث** (`?page=2`) — يُتيح bookmark/share.
✅ **Tabular nums للأرقام**.
❌ **لا تَستَخدِم infinite scroll للبَيانات الحَرِجَة** (orders, financial).
