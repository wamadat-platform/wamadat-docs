# 03.4 — Badges

> الـ Badge شَريط مَعلومات. تَستَخدِمه للتَصنيف، الحالة، أو الكَمّيّة.

---

## Variants

| Variant | Color | استخدام |
|---|---|---|
| `default` | `bg-jet-100 text-jet-700` | تَصنيف خَفيف |
| `solid` | `bg-jet-900 text-white` | تَأكيد قَويّ |
| `accent` | `bg-orange-100 text-orange-800` | تَصنيفات ومضات |
| `success` | `bg-green-100 text-green-800` | "مَدفوع"، "مُكتَمِل" |
| `warning` | `bg-amber-100 text-amber-800` | "بانتِظار"، "قَيد المُراجَعَة" |
| `danger` | `bg-red-100 text-red-700` | "مُلغى"، "فَشِل" |
| `info` | `bg-blue-100 text-blue-800` | "جَديد"، "تَعديل" |

---

## Sizes

| Size | Padding | Text |
|---|---|---|
| `sm` | `px-2 py-0.5` | `text-xs` |
| `md` | `px-3 py-1` | `text-sm` |
| `lg` | `px-4 py-1.5` | `text-base` |

---

## Component

```tsx
// frontend/components/ui/badge.tsx
import { cn } from '@/lib/utils';

const variants = {
    default: 'bg-jet-100 text-jet-700',
    solid: 'bg-jet-900 text-white',
    accent: 'bg-orange-100 text-orange-800',
    success: 'bg-green-100 text-green-800',
    warning: 'bg-amber-100 text-amber-800',
    danger: 'bg-red-100 text-red-700',
    info: 'bg-blue-100 text-blue-800',
};

const sizes = {
    sm: 'px-2 py-0.5 text-xs',
    md: 'px-3 py-1 text-sm',
    lg: 'px-4 py-1.5 text-base',
};

export function Badge({
    variant = 'default',
    size = 'md',
    className,
    children,
    ...props
}: {
    variant?: keyof typeof variants;
    size?: keyof typeof sizes;
    className?: string;
    children: React.ReactNode;
} & React.HTMLAttributes<HTMLSpanElement>) {
    return (
        <span
            className={cn(
                'inline-flex items-center gap-1 rounded-full font-medium',
                variants[variant],
                sizes[size],
                className,
            )}
            {...props}
        >
            {children}
        </span>
    );
}
```

---

## Usage

```tsx
<Badge variant="accent">مُمَيَّز</Badge>
<Badge variant="success" size="sm">مَدفوع</Badge>
<Badge variant="warning">بانتظار التَأكيد</Badge>

{/* مع أيقونَة */}
<Badge variant="info">
    <Sparkles className="size-3" />
    جَديد
</Badge>
```

---

## القَواعِد

✅ **`rounded-full`** افتراضيّاً (pills).
✅ **font-medium** ليَكون قابِل للقِراءَة.
✅ **استَخدِم semantic colors بحَكمَة** — لا تَستَخدِم success لِشَيء غَير ناجح.
❌ **لا badge بدون نَصّ** — حَتّى مَع icon، لازِم كَلِمَة.
❌ **لا تَستَخدِم Badge كَزِرّ** — هو شارَة، لا إجراء.
