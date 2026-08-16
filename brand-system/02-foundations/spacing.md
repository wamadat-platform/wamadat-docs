# 02.3 — المَسافات (Spacing)

> 8pt grid. كلّ مَسافَة في المنصّة مُضاعَف لـ 8 (أو 4 في الحالات النادِرَة).

---

## السُلَّم (Scale)

```
0:    0px
0.5:  2px    ← border refinements
1:    4px    ← أصغر فَجوَة بَين عَناصر مُتقارِبَة
2:    8px    ← base unit ⭐
3:    12px
4:    16px   ← الفَجوَة الافتراضيّة بَين عَناصر
5:    20px
6:    24px   ← Card padding، Section gap الصَغير
8:    32px   ← Section gap المُتَوَسِّط ⭐
10:   40px
12:   48px   ← Section gap الكَبير
16:   64px   ← فَجوَة بَين أقسام رَئيسيّة
20:   80px
24:   96px   ← Hero padding-y
32:   128px
40:   160px
48:   192px
```

---

## الـ Building Blocks (الكُتَل البِنائيّة)

### Padding داخل البِطاقات (Cards)

| الحجم | Padding | استخدام |
|---|---|---|
| Compact | `p-4` (16px) | كروت صَغيرَة، list items |
| Default | `p-6` (24px) | كروت عاديّة ⭐ |
| Spacious | `p-8` (32px) | كروت مُمَيَّزَة، Hero cards |
| Generous | `p-10` lg:`p-12` | Hero sections |

### Gap بَين العَناصر (Stacks)

| الـ Gap | Class | استخدام |
|---|---|---|
| Tight | `gap-2` (8px) | عَناصر مُتقارِبَة (icon + text، badges) |
| Default | `gap-4` (16px) | الافتراضيّ ⭐ |
| Comfortable | `gap-6` (24px) | بَين بُطاقات في Grid |
| Wide | `gap-8` (32px) | فَواصِل عَريضَة |

### Margin بَين الأقسام

```
y-spacing بَين الأقسام: 48px → 64px → 96px
```

```tsx
<section className="py-16 md:py-24 lg:py-32">
    {/* محتوى */}
</section>
```

---

## القَواعِد الصارِمَة

### ✅ نَفعَل
- استَخدِم **مُضاعَفات 8** دائماً (8، 16، 24، 32، 48، 64، 96، 128).
- استَخدِم 4 للـ refinements الصَغيرَة فقط (icon spacing، border offsets).
- استَخدِم `space-y-*` للـ stacks العَموديّة بدل margin يَدويّ.
- استَخدِم `gap-*` في Grid/Flex بدل margin.

### ❌ لا نَفعَل
- لا نَستَخدِم `p-5` (20px) أو `p-7` (28px) — هذه أرقام "بَين بَين".
- لا نَستَخدِم `mt-3` متَبوع بـ `mt-5` — اختار سَلَم.
- لا نَستَخدِم margin عَشوائيّ (`margin-top: 17px`) أبداً.

---

## أنماط المَسافات في الصَفحات

### Hero Section
```tsx
<section className="py-16 md:py-24 lg:py-32 px-4 md:px-8">
    <div className="container max-w-5xl">
        <h1 className="mb-6 ...">...</h1>
        <p className="mb-8 ...">...</p>
        <div className="flex gap-3">
            <Button>...</Button>
            <Button>...</Button>
        </div>
    </div>
</section>
```

### Card
```tsx
<article className="bg-white rounded-2xl border border-jet-100 p-6">
    <header className="mb-4 pb-4 border-b border-jet-100">...</header>
    <div className="space-y-3">
        <p>...</p>
        <p>...</p>
    </div>
    <footer className="mt-6 pt-4 border-t border-jet-100">...</footer>
</article>
```

### Form
```tsx
<form className="space-y-5">
    <div className="grid sm:grid-cols-2 gap-4">
        <Field />
        <Field />
    </div>
    <Field />
    <Button>...</Button>
</form>
```

---

## مَسافات على الجَوّال

عُد إلى **مَقياس أَصغَر** على الجَوّال:

| الـ desktop | الـ mobile |
|---|---|
| `py-32` | `py-16` |
| `px-12` | `px-4` |
| `gap-8` | `gap-4` |
| `p-10` | `p-6` |

استَخدِم `md:` و `lg:` للترقيَة:
```tsx
<section className="py-16 md:py-24 lg:py-32">
<div className="p-6 lg:p-10">
```

---

## مَسافات في الـ Forms

| المَسافَة | القيمَة |
|---|---|
| بَين label والـ input | `mb-1` (4px) |
| بَين input والمساعِد (help text) | `mt-1` (4px) |
| بَين Fields المُختَلِفَة | `space-y-5` (20px) |
| بَين Form section والـ next | `space-y-8` (32px) |

---

## مَسافات في الـ Typography

```css
/* بَين h1 والـ paragraph */
h1 + p { margin-top: 16px; }

/* بَين paragraphs */
p + p { margin-top: 16px; }

/* بَين h2 والمحتوى التالي */
h2 + * { margin-top: 16px; }
section + section { margin-top: 64px; }
```

> Tailwind utility `space-y-4` يَتولّى هذا لك. استَخدِمها بدل margin مُتَفَرِّق.

---

## مَسافات أُفُقيّة للقَوائم

```tsx
{/* قَوائم بَطيئَة (relaxed) */}
<ul className="space-y-3">
    <li>...</li>
    <li>...</li>
</ul>

{/* قَوائم مُكَدَّسَة (compact) */}
<ul className="space-y-1">
    <li>...</li>
    <li>...</li>
</ul>
```

---

## المَسافَة كَأَدَاة Hierarchy

> **الفَراغ يَتَكَلَّم.** كَثرَة المَسافَة = أهَمّيّة. قِلَّتها = تَدفَّق سَريع.

- **Hero**: مَسافات سَخيّة (`py-24+`) — الإعلان مُهمّ.
- **Listings**: مَسافات مُتَوَسِّطَة (`gap-6`) — سَريعَة التَصَفُّح.
- **Dashboard tables**: مَسافات مُحكَمَة (`gap-2`) — كَثافَة معلومات.

استَخدِم المَسافات لتَقول للعَين: "هذا مُهمّ، خُذ وَقت" أو "هذا قائمة، اقرأ بسُرعَة".

---

## التَطبيق في Tailwind

كلّ القِيَم مُسَجَّلة في الـ `spacing` الافتراضيّ لـ Tailwind. لا نَحتاج تَخصيص — السُلَّم الأَصليّ كامِل.

استَخدِم:
- `p-{n}`, `px-{n}`, `py-{n}` للـ padding
- `m-{n}`, `mt-{n}`, `space-y-{n}` للـ margin
- `gap-{n}` في Flex/Grid

ابتَعِد عن `style={{ marginTop: ... }}` نِهائيّاً.
