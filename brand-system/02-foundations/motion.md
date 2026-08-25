# 02.7 — الحَرَكَة (Motion)

> الحَرَكَة لُغَة. كلّ حَرَكَة تَقول شَيئاً. السُرعَة، الـ easing، والمَدى كلّها رَسائِل.

---

## الـ Durations

| Token | Value | استخدام |
|---|---|---|
| `instant` | 100ms | hover، tap (تَجاوُب فَوريّ) |
| `fast` | 200ms | Transitions الافتراضيّة ⭐ |
| `normal` | 300ms | Modals، drawers، tabs |
| `slow` | 500ms | Page transitions |
| `glacial` | 1000ms | Storytelling، loading spinners |

> **قاعِدَة:** إذا حَرَكَة تَأخُذ أكثر من 500ms، أَعِد التَفكير. سُرعَة المُستَخدِم > جَمال الانتِقال.

---

## Easing Functions

| الـ Easing | متى | الشَخصيّة |
|---|---|---|
| `ease-out` | الدُخول (مَرحَبَة) | يَأتي بسُرعَة، يَستَقِرّ بهُدوء |
| `ease-in` | الخُروج (مُوَدِّعَة) | يَبدأ بهُدوء، يَنطَلِق |
| `ease-in-out` | التَنَقُّل | متوازِن، الافتراضيّ |
| `linear` | Loading، Progress | لا تَسارُع — مُنتَظِم |
| `spring` (Framer Motion) | Playful تَفاعلات | يَرتَدّ قَليلاً |

CSS:
```css
.element {
    transition-timing-function: cubic-bezier(0.16, 1, 0.3, 1); /* ease-out spring-like */
}
```

Tailwind:
```tsx
<div className="transition duration-200 ease-out hover:scale-105">
```

---

## الأنماط الشائِعَة

### Hover على Card
```tsx
<article className="
    transition-all duration-200 ease-out
    hover:shadow-lg hover:-translate-y-0.5
    cursor-pointer
">
```

**القَواعِد:**
- `translateY(-2px)` (لا أَكثَر — يَبدو "يَطيرُ")
- `shadow-sm → shadow-lg`
- 200ms (سَريع كَفاية لِيَكون فَوريّاً، بَطيء كَفاية لِيَكون مَرئيّاً)

### Button Click
```tsx
<button className="
    transition transform duration-100
    active:scale-[0.98]
">
```

**القاعِدَة:** Scale بَسيط (0.98) — يَشعُر المُستَخدِم بـ "نَقَر".

### Modal Entry
```tsx
{/* Tailwind classes إن استَخدَمت headlessui */}
<Transition
    enter="ease-out duration-300"
    enterFrom="opacity-0 scale-95"
    enterTo="opacity-100 scale-100"
    leave="ease-in duration-200"
    leaveFrom="opacity-100 scale-100"
    leaveTo="opacity-0 scale-95"
>
```

**القاعِدَة:**
- دُخول 300ms ease-out (مَرحَبَة)
- خُروج 200ms ease-in (أَسرَع)
- Scale 0.95 → 1.0 (لا flip أو slide مَلَّ)

### Page Transition
```tsx
{/* Framer Motion */}
<motion.div
    initial={{ opacity: 0, y: 8 }}
    animate={{ opacity: 1, y: 0 }}
    transition={{ duration: 0.3, ease: 'easeOut' }}
>
```

**القاعِدَة:** opacity + translateY صَغير (8px). لا "explosion" حَرَكات.

### Loading Spinner
```tsx
<div className="animate-spin size-5 border-2 border-orange-500 border-t-transparent rounded-full" />
```

**القاعِدَة:**
- 1000ms full rotation
- Linear (لا تَسارُع)
- Infinite

### Toast Entry
```tsx
{/* slide من الأعلى */}
<motion.div
    initial={{ opacity: 0, y: -20 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -20 }}
    transition={{ duration: 0.25 }}
>
```

### Accordion Expand
```css
.content {
    overflow: hidden;
    transition: max-height 0.3s ease-in-out;
}
.content.collapsed { max-height: 0; }
.content.expanded { max-height: 500px; }
```

---

## Tailwind Animation Tokens

في Tailwind v4، تم نقل التكوين من `tailwind.config.ts` إلى المتغيرات في `styles/tokens.css` والـ `@keyframes` في `styles/globals.css`.

جميع الحركات المخصصة تبدأ بـ `wmt-` لتجنب التعارض (Namespaced)، مثل:
- `wmt-marquee` و `wmt-marquee-reverse` (للشرائط المتحركة)
- `wmt-shimmer` (لتحميل الـ Skeletons)
- `wmt-sweep`
- `wmt-blob` و `wmt-blob-slow` (للأشكال العضوية)
- `wmt-float`
- `wmt-pulse-ring` (للتنبيهات)
- `wmt-slider-fade` و `wmt-slider-pager` (للـ Banners)

استخدمها في Tailwind هكذا:
```tsx
<div className="animate-[wmt-blob_4s_infinite_alternate]" />
```

---

## القَواعِد الصارِمَة

### ✅ نَفعَل
- **استَخدِم Motion بهَدَف** — كلّ حَرَكَة تَقول شَيء (تَأكيد، تَوجيه، فَرَح).
- **اختَر duration واحِد** لِكُلّ نَوع تَفاعُل (لا تَخلِط 150ms و220ms عَشوائيّاً).
- **استَخدِم `transform` و `opacity`** فقط لِلأَداء (GPU-accelerated).
- **اخفِف Motion للنّبَر الكَثيف** (loading states).

### ❌ لا نَفعَل
- لا **bounce/spring في كلّ تَفاعُل** — يَبدو طَفوليّاً.
- لا **حَرَكات أَطوَل من 500ms** للتَفاعُلات (مَسموحَة للقِصَص فقط).
- لا **حَرَكَة لا غَرَض لها** (نَجمَة تَدور بدون سَبَب).
- لا **animate كلّ شَيء عند الـ scroll** — لا "AOS-style overload".
- لا **animate `width`, `height`, `top`** (CPU-bound، يَتَكَسَّر).

---

## Reduced Motion (الوصول)

احتَرِم `prefers-reduced-motion` دائماً:

### CSS
```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

### React (مَع Framer Motion)
```tsx
import { useReducedMotion } from 'framer-motion';

const shouldReduceMotion = useReducedMotion();
const variants = shouldReduceMotion
    ? { initial: {}, animate: {} }
    : { initial: { y: 20, opacity: 0 }, animate: { y: 0, opacity: 1 } };
```

---

## Storytelling Motion (نادِر)

أحياناً نَستَخدِم motion لِسَرد قِصّة (Hero الرَئيسيّة، success celebration):

### Success Celebration
```tsx
{/* عند إكمال بَرنامج، شَهادَة، أو شراء */}
<motion.div
    initial={{ scale: 0, rotate: -10 }}
    animate={{ scale: 1, rotate: 0 }}
    transition={{
        type: 'spring',
        stiffness: 300,
        damping: 20,
    }}
>
    <Award className="size-24 text-orange-500" />
</motion.div>
```

> **استَخدِم بحَذَر:** فَرحَة واحِدَة في الرِحلَة الواحِدَة. لو كلّ شَيء "celebratory"، لا شَيء مُمَيَّز.

---

## Performance

| الـ Property | Performance |
|---|---|
| `transform: translate / scale / rotate` | ✅ GPU، ممتاز |
| `opacity` | ✅ GPU، ممتاز |
| `filter` | ⚠️ مُكلِف، استَخدِم بحَذَر |
| `width / height / top / left` | ❌ CPU، يُسَبِّب reflow |
| `box-shadow` | ⚠️ مُكلِف على animate، آمِن على hover |

> **القاعِدَة:** animate `transform` و `opacity` فقط لـ60fps.

---

## أمثِلَة في الـ Real Platform

### Card Hover (programs grid)
```tsx
<article className="
    group bg-white rounded-2xl overflow-hidden border border-jet-100
    transition-all duration-200 ease-out
    hover:shadow-xl hover:-translate-y-1
">
    <img className="transition-transform duration-300 group-hover:scale-105" />
</article>
```

### Skeleton Loader
```tsx
<div className="animate-pulse">
    <div className="h-4 bg-jet-100 rounded w-3/4 mb-2" />
    <div className="h-4 bg-jet-100 rounded w-1/2" />
</div>
```

### Progress Bar
```tsx
<div className="h-2 bg-jet-100 rounded-full overflow-hidden">
    <div
        className="h-full bg-orange-500 transition-all duration-500 ease-out"
        style={{ width: `${percent}%` }}
    />
</div>
```

### Mobile Drawer
```tsx
<aside className={cn(
    "fixed inset-y-0 end-0 w-72 bg-white shadow-2xl",
    "transition-transform duration-300 ease-out",
    isOpen ? "translate-x-0" : "translate-x-full rtl:-translate-x-full"
)}>
```

---

## التَطبيق

كلّ هذه الأنماط مُضَمَّنَة في `styles/globals.css` و `styles/tokens.css`. استَخدِم الـ utility classes — لا تَكتُب CSS مُخَصَّص إلا في أضيق الحدود للحركات المعقدة.
