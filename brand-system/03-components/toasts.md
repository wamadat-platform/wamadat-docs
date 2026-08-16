# 03.15 — Toasts (Transient Notifications)

> رَسائِل قَصيرَة تَختَفي تلقائيّاً. للتَأكيدات والتَنبيهات غَير الحَرِجَة.

---

## Variants

| Variant | استخدام |
|---|---|
| `success` | "تَمّ الحِفظ"، "تَمّ الإرسال" |
| `info` | "نُسَخَة جَديدَة مُتاحَة" |
| `warning` | "اتِّصالَك بَطيء" |
| `error` | "فَشِل الحِفظ — حاوِل مرّة أخرى" |

---

## Behavior

- يَظهَر من **أعلى الصَفحَة** (`top-right` في LTR، `top-left` في RTL — أو centered top)
- يَختَفي بَعد **3-5 ثواني** تلقائيّاً
- يُمكِن **إغلاقه يَدويّاً** (X button)
- **يَتَكَدَّس** لو وَصَلَت رَسائل مُتَعَدِّدَة (max 3)

---

## Pattern (Sonner library)

```bash
npm install sonner
```

```tsx
// app/layout.tsx
import { Toaster } from 'sonner';

<Toaster
    position="top-center"
    dir="rtl"
    toastOptions={{
        className: 'rtl text-right',
        style: { fontFamily: 'Tajawal, sans-serif' },
    }}
/>
```

```tsx
import { toast } from 'sonner';

toast.success('تَمّ حِفظ الإعدادات');
toast.error('فَشِل الحِفظ — حاوِل مرّة أخرى', {
    action: { label: 'إعادة', onClick: () => retry() },
});
```

---

## Anatomy

```
┌── Toast ─────────────────────────────────┐
│ [Icon]  Title                       [X]  │
│         Optional description             │
│         [Optional action button]         │
└──────────────────────────────────────────┘
```

---

## القَواعِد

✅ **محتوى قَصير** (5-15 كَلِمَة).
✅ **Action واحِد فَقَط** (لا 3 أزرار).
✅ **Auto-dismiss** بَعد 3-5 ثواني (errors قد تَكون longer).
❌ **لا Toasts للأخطاء الحَرِجَة** — استَخدِم Modal.
❌ **لا 5+ toasts في وقت واحِد** — مُربِك.
