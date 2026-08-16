# 06.4 — تَبايُن الألوان (Color Contrast)

> القاعِدَة: 4.5:1 للنَصّ، 3:1 للنَصّ الكَبير، 3:1 لِعَناصِر UI.

---

## WCAG Levels

| Level | Standard text | Large text (18px+ أو 14px+ bold) |
|---|---|---|
| AA (الحَدّ الأَدنى) | 4.5:1 | 3:1 |
| AAA (الأَعلى) | 7:1 | 4.5:1 |

> ومضات تَلتَزِم بـ **AA**.

---

## تَركيبات مُعتَمَدَة (راجع `02-foundations/colors.md`)

### على bg-white
- ✅ `text-jet-900` — 12.6:1 (Excellent)
- ✅ `text-jet-800` — 11.4:1
- ✅ `text-jet-700` — 7.5:1 (AA)
- ✅ `text-jet-600` — 6.0:1 (AA)
- ✅ `text-jet-500` — 4.6:1 (AA حَدّ)
- ⚠️ `text-jet-400` — 3.4:1 (Large text فَقَط)
- ❌ `text-jet-300` — 2.5:1 (لا تَستَخدِم للنَصّ)

### على bg-jet-900
- ✅ `text-white` — 12.6:1
- ✅ `text-orange-500` — 5.0:1 (AA)
- ✅ `text-orange-400` — 4.5:1

### على bg-orange-500
- ✅ `text-jet-900` — 8.4:1 (Excellent)
- ⚠️ `text-white` — 2.6:1 (Large bold فَقَط)
- ❌ `text-orange-700` — 2.0:1

---

## Tools للفَحص

### Online
- https://webaim.org/resources/contrastchecker/
- https://contrast-ratio.com/

### Browser
- Chrome DevTools → Lighthouse → Accessibility
- axe DevTools extension

### Figma
- Figma Color Contrast plugin

---

## المَناطِق الحَرِجَة

### Buttons
- Primary (`bg-orange-500 text-jet-900`) — ✅ 8.4:1
- Secondary (`bg-white text-jet-900 border-jet-200`) — ✅ 12.6:1
- Danger (`bg-red-600 text-white`) — ✅ 5.3:1

### Form Fields
- Label (`text-jet-700`) — ✅ 7.5:1
- Placeholder (`text-jet-400`) — ⚠️ نَصّ كَبير فَقَط (placeholder عاديّ في الـ inputs ≥16px)
- Error text (`text-red-600`) — ✅ 5.7:1

### Cards
- Title (`text-jet-900`) — ✅ 12.6:1
- Body (`text-jet-700`) — ✅ 7.5:1
- Meta (`text-jet-500`) — ✅ 4.6:1

### Disabled States
- `opacity-50` على نَصّ مُعَتَم أَصلاً قَد يَكسِر AA
- إذا اضطُرِرت، استَخدِم `text-jet-400` بدل `opacity-50`

---

## Color-Only Communication (لا)

**لا تَعتَمِد على اللون وَحده** للمعنى:

```tsx
{/* خَطَأ: لون فقط */}
<span className="text-red-600">قَيد المُراجَعَة</span>

{/* صَحيح: لون + icon + text */}
<span className="text-red-600 inline-flex items-center gap-1">
    <AlertCircle className="size-4" />
    قَيد المُراجَعَة
</span>
```

> يَحمي ضُعَفاء الرُؤيَة وعَمى الألوان.

---

## High Contrast Mode

في Windows، المُستَخدِم قَد يُفَعِّل "High Contrast Mode". تَأَكَّد من أنّ:

- لا تَعتَمِد على `background-image` للـ icons (تَختَفي في HC mode)
- استَخدِم `currentColor` على SVGs (تَتَكَيَّف)
- اختَبر يَدويّاً في Windows Settings → Accessibility → Contrast themes

---

## Quarterly Audit

كلّ 3 شُهور، شَغِّل Lighthouse على:
1. Homepage
2. Programs listing
3. Program detail
4. Sign-up form
5. Dashboard
6. Checkout

هَدَف: **Accessibility score ≥ 95** على كلّ صَفحَة.
