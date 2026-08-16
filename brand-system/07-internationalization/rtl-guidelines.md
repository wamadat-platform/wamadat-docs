# 07.1 — قَواعِد RTL

> ومضات Arabic-first. كلّ شَيء يَعمَل في RTL أصالةً، لا "مُترجَم".

---

## الأساسيّات

### HTML
```html
<html lang="ar" dir="rtl">
```

### CSS / Tailwind
- استَخدِم **logical properties** (`ms-`, `me-`, `ps-`, `pe-`) بَدَلاً من `ml-`, `mr-`, `pl-`, `pr-`.
- استَخدِم `start`, `end` بَدَلاً من `left`, `right`.

```tsx
{/* خَطَأ */}
<div className="ml-4 pr-2 text-left">...</div>

{/* صَحيح */}
<div className="ms-4 pe-2 text-start">...</div>
```

---

## Tailwind Logical Properties Map

| LTR (تَجَنَّب) | RTL-aware (استَخدِم) |
|---|---|
| `ml-{n}` | `ms-{n}` (margin-inline-start) |
| `mr-{n}` | `me-{n}` (margin-inline-end) |
| `pl-{n}` | `ps-{n}` |
| `pr-{n}` | `pe-{n}` |
| `left-{n}` | `start-{n}` |
| `right-{n}` | `end-{n}` |
| `border-l` | `border-s` |
| `border-r` | `border-e` |
| `rounded-l-{n}` | `rounded-s-{n}` |
| `rounded-r-{n}` | `rounded-e-{n}` |
| `text-left` | `text-start` |
| `text-right` | `text-end` |
| `float-left` | `float-start` |
| `float-right` | `float-end` |

---

## Direction-Aware Modifiers

عند الحاجَة للسُلوك المُختَلِف، استَخدِم `rtl:` و `ltr:`:

```tsx
{/* Arrow يَنعَكِس في RTL */}
<ArrowLeft className="size-4 rtl:rotate-180" />

{/* Margin مُختَلِف */}
<div className="rtl:ml-4 ltr:mr-4">...</div>

{/* Flex direction */}
<div className="flex rtl:flex-row-reverse">...</div>
```

---

## أيقونات الاتِّجاه

راجع `02-foundations/iconography.md`:

### تَنعَكِس
- `ArrowLeft / ArrowRight`
- `ChevronLeft / ChevronRight`
- `Send`
- `Undo / Redo`

### لا تَنعَكِس
- `User`, `Bell`, `Heart`
- `Settings`, `Mail`
- `Lock`, `Shield`

---

## Layout Patterns

### Flexbox
في RTL، `flex-row` يَعكِس تلقائيّاً (Tailwind 3+). لا تَحتاج `flex-row-reverse` عادَةً.

### Grid
يَعمَل تلقائيّاً في RTL — الأعمِدَة تَبدأ من اليَمين.

### Position
استَخدِم `start-0`, `end-0` بَدَلاً من `left-0`, `right-0`.

```tsx
<div className="absolute top-0 end-4">...</div>
```

---

## Spacing بَين العَناصِر

❌ `space-x-3` تَفصل بـ `margin-left/right` ثابِت — يَكسِر في RTL
```tsx
{/* خَطَأ */}
<div className="flex space-x-3">
    <A /> <B /> <C />
</div>
```

✅ استَخدِم `gap-3` (يَعمَل في كِلا الاتِّجاهَين):
```tsx
<div className="flex gap-3">
    <A /> <B /> <C />
</div>
```

أو إذا اضطُرِرت لـ `space-x-*`، أَضِف `rtl:space-x-reverse`:
```tsx
<div className="flex space-x-3 rtl:space-x-reverse">
```

---

## Numbers & Direction

### Numbers Stay LTR
الأرقام دائماً LTR حتّى في نَصّ عَربيّ:
```
الطَلَب رقم 12345 يَتَكَلَّف 4,195 ر.س
```

في الـ CSS، الـ tabular-nums + Unicode bidi يَتولَّى هذا تلقائيّاً.

### Mixed Direction
لو فيك نَصّ مُختَلَط (عَربيّ + إنجليزيّ)، استَخدِم `<bdi>` للعَناصِر الإنجليزيّة:

```tsx
المُدرّب <bdi>John Smith</bdi> يَشرَح اليوم
```

---

## RTL Testing

### Manual
1. افتَح الصَفحَة في `/ar` — يَجب تَكون RTL
2. افتَح في `/en` — يَجب تَكون LTR
3. تَأَكَّد لا overflow على الأطراف
4. تَأَكَّد الأَيقونات الاتِّجاهيّة تَعكِس صَحيح

### Automated
استَخدِم Playwright لِالتِقاط screenshots في كِلا الاتِّجاهَين والمُقارَنَة.

---

## Common Pitfalls

❌ Hardcoded `left/right` في CSS
❌ Absolute positioning بدون `start/end`
❌ Sliders/Carousels لا تَعكِس direction
❌ Drag-and-drop لا يَتَكَيَّف
❌ Number formatting لا يَستَخدِم locale
