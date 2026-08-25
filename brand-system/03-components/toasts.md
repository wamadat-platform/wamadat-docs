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
pnpm add sonner
```

```tsx
// @/components/ui/toaster.tsx (يتم وضعه في app/layout.tsx)
import { Toaster as SonnerToaster } from 'sonner';

export function Toaster() {
    return (
        <SonnerToaster
            position="top-center"
            richColors
            closeButton
            duration={4000}
            toastOptions={{
                classNames: {
                    toast: 'font-sans',
                    title: 'font-bold',
                    description: 'text-jet-600',
                },
            }}
        />
    );
}

// app/layout.tsx
import { Toaster } from '@/components/ui/toaster';

export default function RootLayout({ children }) {
    return (
        <html dir="rtl" lang="ar">
            <body>
                {children}
                <Toaster />
            </body>
        </html>
    );
}
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
