# 03.14 — Alerts (Inline)

> رَسائِل دائِمَة في سياق الصَفحَة (ليس toast الذي يَختَفي).

---

## Variants

| Variant | الخَلفيّة | الأيقونَة |
|---|---|---|
| `info` | `bg-blue-50 border-blue-200` | `Info` |
| `success` | `bg-green-50 border-green-200` | `Check` |
| `warning` | `bg-amber-50 border-amber-200` | `AlertTriangle` |
| `danger` | `bg-red-50 border-red-200` | `AlertCircle` |

---

## Pattern

```tsx
<div className="
    flex gap-3 items-start
    rounded-lg border bg-amber-50 border-amber-200 px-4 py-3
">
    <AlertTriangle className="size-5 text-amber-700 flex-shrink-0 mt-0.5" />
    <div className="text-sm text-amber-900 leading-relaxed">
        <strong className="block mb-0.5">تَحذير</strong>
        دفعَتك لم تَكتَمِل. يُرجى التَواصُل مَع البَنك.
    </div>
</div>
```

---

## القَواعِد

✅ **Inline في السياق** (داخل صَفحَة الـ checkout، الـ form).
✅ **Icon + Title + Body** — هَرَميّة واضِحَة.
✅ **Action اختياريّ** ضِمن alert (زِرّ "إعادة المُحاولَة").
❌ **لا تَستَخدِمها لِنَجاحات بَسيطَة** — استَخدِم Toast.
❌ **لا تُكَدِّس alerts مُتَتالِيَة** — مُرهِق.
