# 06.2 — التَنَقُّل بالكِيبورد (Keyboard Navigation)

> كلّ شَيء يُمكِن فِعله بالـ mouse، يَجب أن يَكون مُمكِناً بالكِيبورد.

---

## Tab Order

الـ Tab يَتبَع الـ **visual reading order** (يَمين → يَسار → أَسفَل في RTL).

```tsx
{/* خَطَأ: الـ Tab يَقفِز بَين عَناصِر مُبَعثَرَة */}
<div className="flex">
    <Sidebar /> {/* tab=1 */}
    <Main /> {/* tab=2 */}
    <Header className="absolute top-0" /> {/* tab=3 — مَكسور! */}
</div>

{/* صَحيح: Header قَبل Sidebar/Main في الـ DOM */}
<div>
    <Header /> {/* tab=1 */}
    <Sidebar /> {/* tab=2 */}
    <Main /> {/* tab=3 */}
</div>
```

---

## Focus Indicators

كلّ عُنصُر focusable يَجب أن يُظهِر focus state واضِح:

```css
*:focus-visible {
    outline: 2px solid #FAAF3C;
    outline-offset: 2px;
    border-radius: 4px;
}
```

أو في Tailwind:
```tsx
className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-500 focus-visible:ring-offset-2"
```

---

## Shortcuts

### Global
- `Tab` — التالي
- `Shift+Tab` — السابِق
- `Enter` — تَفعيل الزِرّ / إرسال نَموذج
- `Space` — تَفعيل الـ checkbox / radio
- `Esc` — إغلاق modal / dropdown
- `Cmd/Ctrl+K` — فَتح البَحث (Future)

### Modals
- `Tab` — التَنَقُّل داخل الـ modal فقط (focus trap)
- `Esc` — إغلاق

### Menus
- `↑` / `↓` — التَنَقُّل بَين العَناصِر
- `Home` / `End` — الذِهاب للأَوَّل / الآخِر
- `Enter` — تَحديد

### Tables
- `Tab` — الانتِقال للزِرّ التالي
- `↑↓←→` — التَنَقُّل بَين الخَلايا (لو الـ table interactive)

---

## Skip Links

أَوَّل عُنصُر في الـ `<body>`:

```tsx
<a
    href="#main"
    className="
        sr-only focus:not-sr-only
        focus:absolute focus:top-2 focus:start-2 focus:z-50
        focus:px-4 focus:py-2 focus:bg-jet-900 focus:text-white focus:rounded-lg
    "
>
    تَخَطّى إلى المُحتوى الرَئيسيّ
</a>
```

> يَختَفي افتراضيّاً (sr-only) ويَظهَر فقط عند الـ focus بـ Tab.

---

## Focus Trap (في Modals)

```tsx
import { FocusTrap } from '@headlessui/react';

<FocusTrap active={open}>
    <Modal>...</Modal>
</FocusTrap>
```

Headless UI Dialog يُطَبِّق الـ trap تلقائيّاً.

---

## القَواعِد

✅ **`tabindex="0"`** على custom interactive elements (نادراً)
✅ **`tabindex="-1"`** على elements تُريد تَركيزها برمجيّاً فَقَط
✅ **لا تَستَخدِم `tabindex="1"` أو أَعلى** — يَكسِر الترتيب الطَبيعيّ
❌ **لا تُخفي focus state** أبداً (`outline: none` بدون بَديل = خَطَأ)
❌ **لا تَستَخدِم `onClick` على `<div>`** — استَخدِم `<button>`
