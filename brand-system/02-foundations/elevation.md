# 02.8 — الارتِفاع (Elevation / Shadows)

> الظِلال تَخلُق هَرَميّة. كَتَب أعلى في الـ z-axis = أعلى أهَمّيّة.

---

## السُلَّم (Scale)

| Token | Value | الاستخدام |
|---|---|---|
| `shadow-none` | none | Flat sections، dense tables |
| `shadow-xs` | `0 1px 2px rgba(0,0,0,0.05)` | الـ Buttons، subtle outlines |
| `shadow-sm` | `0 1px 3px rgba(0,0,0,0.1)` | Cards (default) ⭐ |
| `shadow-md` | `0 4px 6px rgba(0,0,0,0.1)` | Cards (hover)، popovers |
| `shadow-lg` | `0 10px 15px rgba(0,0,0,0.1)` | Dropdowns، floating panels |
| `shadow-xl` | `0 20px 25px rgba(0,0,0,0.1)` | Modals (medium) |
| `shadow-2xl` | `0 25px 50px rgba(0,0,0,0.25)` | Modals (large)، Dialogs |

---

## القَواعِد

### ✅ نَفعَل
- استَخدِم **shadow-sm افتراضيّاً** للـ Cards.
- زِد إلى **shadow-md/lg عند الـ hover**.
- استَخدِم **shadow-xl/2xl للـ modals + dialogs** فقط.
- استَخدِم **shadow + border** أحياناً للوُضوح:
  ```tsx
  className="bg-white border border-jet-100 shadow-sm"
  ```

### ❌ لا نَفعَل
- لا **shadow ثابِت + colored** (لا "orange shadow") — يَبدو رَخيصاً.
- لا **shadow على flat sections** (Hero مع خَلفيّة لون كامِل لا يَحتاج shadow).
- لا **shadow على footer** (هو في القاع، لا شَيء يَطفو فوقه).

---

## مَتى تَستَخدِم shadow vs. border؟

| الحالَة | shadow أم border؟ |
|---|---|
| Card في Grid | shadow-sm + border-jet-100 |
| Modal | shadow-2xl + no border |
| Input | border-jet-200 + no shadow |
| Sticky header | shadow-sm + no border |
| Footer | no shadow + no border |
| Dropdown menu | shadow-lg + border-jet-100 (نادِر) |

---

## أنماط في المنصّة

### Card Default
```tsx
<article className="bg-white rounded-2xl border border-jet-100 shadow-sm">
```

### Card Hover
```tsx
<article className="
    bg-white rounded-2xl border border-jet-100 shadow-sm
    transition-shadow hover:shadow-lg
">
```

### Modal
```tsx
<div className="bg-white rounded-2xl shadow-2xl max-w-md p-8">
```

### Sticky Top Bar
```tsx
<header className="sticky top-0 z-50 bg-white/95 backdrop-blur shadow-sm">
```

### Floating Action Button (FAB)
```tsx
<button className="
    fixed bottom-6 end-6 size-14 rounded-full
    bg-orange-500 shadow-xl
    hover:shadow-2xl
">
```

---

## Z-Index Scale

لِنَضمَن طَبَقات مَنطِقيّة:

| Token | Value | استخدام |
|---|---|---|
| `z-0` | 0 | الافتراضيّ |
| `z-10` | 10 | Slight elevation (sticky elements داخل cards) |
| `z-20` | 20 | Dropdowns داخل sections |
| `z-30` | 30 | Sticky filters |
| `z-40` | 40 | Sticky headers |
| `z-50` | 50 | Navbar، toasts |
| `z-[60]` | 60 | Modals overlay |
| `z-[70]` | 70 | Modal content |
| `z-[100]` | 100 | Critical alerts (system errors) |

> لا تَستَخدِم `z-9999`. إذا احتَجت قِيمَة بِهذا الـ insanity، الـ stacking مَكسور.
