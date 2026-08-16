# 03.8 — Navigation (Navbar)

> Navbar هو "Home base" للمُستَخدِم. سَهل، ثابِت، يَعرِف فيه أين هو دائماً.

---

## Structure

```
[Logo (start)] [Nav links (center)] [Search + Lang + CTAs (end)]
```

### على Mobile
```
[Logo] [Hamburger (toggles drawer)]
```

---

## Behavior

- **Sticky على scroll** (`sticky top-0 z-50`)
- **خَلفيّة شَبه شَفّافَة** مَع backdrop-blur (`bg-white/85 backdrop-blur`)
- **يَنكَمِش قَليلاً على scroll** (اختياريّ)
- **يُخفي تَكويني على scroll السُفلي** (اختياريّ، نادِر)

---

## Active State

الرابط النَشِط (للصَفحَة الحاليّة):
```tsx
{active ? 'bg-orange-50 text-orange-700' : 'text-jet-700 hover:bg-jet-50'}
```

> **`aria-current="page"`** على الرابط النَشِط للوصوليّة.

---

## Implementation

راجع `frontend/components/layouts/navbar.tsx` للتَطبيق الحاليّ.

---

## القَواعِد

✅ **Logo دائماً في البداية** (start).
✅ **CTAs دائماً في النِهايَة** (end).
✅ **Nav links 4-6 maximum** — أكثر، يَصير فوضى.
✅ **Mobile breakpoint عند `lg:`** — التَكوين الكامل يَختَفي.
✅ **Hamburger menu يَدفَع للأسفل** (لا overlay overlay) في drawer mobile.
❌ **لا navbar متَعَدِّد الصُفوف** — صَفّ واحِد.
❌ **لا Mega menus** — keep it simple.
