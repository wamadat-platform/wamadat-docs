# 03.4 — Badges

> الـ Badge شَريط مَعلومات. تَستَخدِمه للتَصنيف، الحالة، أو الكَمّيّة.

---

## Variants

| Variant | Color | استخدام |
|---|---|---|
| `default` / `muted` | `bg-jet-100 text-jet-700` | تَصنيف خَفيف |
| `solid` | `bg-jet-900 text-white` | تَأكيد قَويّ جداً |
| `jet` | `bg-jet-800 text-white` | تأكيد قوي ثانوي |
| `outline` | `border-jet-200 text-jet-700` | شارة مفرغة |
| `accent` | `bg-orange-100 text-orange-800` | تَصنيفات ومضات |
| `success` | `bg-green-100 text-green-800` | "مَدفوع"، "مُكتَمِل" |
| `warning` | `bg-amber-100 text-amber-800` | "بانتِظار"، "قَيد المُراجَعَة" |
| `danger` | `bg-red-100 text-red-700` | "مُلغى"، "فَشِل" |
| `info` | `bg-blue-100 text-blue-800` | "جَديد"، "تَعديل" |

---

## Sizes

| Size | Padding | Text |
|---|---|---|
| `sm` | `px-2 py-0.5` | `text-[10px]` |
| `md` | `px-3 py-1` | `text-xs` |
| `lg` | `px-4 py-1.5` | `text-sm` |

---

## Component

```tsx
// @/components/ui/badge.tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { forwardRef, type HTMLAttributes } from 'react';
import { cn } from '@/lib/utils';

const badgeVariants = cva(
    'inline-flex items-center rounded-full px-3 py-1 text-xs font-medium transition-colors',
    {
        variants: {
            variant: {
                default: 'bg-jet-100 text-jet-700',
                solid: 'bg-jet-900 text-white',
                jet: 'bg-jet-800 text-white',
                outline: 'border-jet-200 text-jet-700 border bg-white',
                muted: 'bg-jet-100 text-jet-700',
                accent: 'bg-orange-100 text-orange-800',
                success: 'bg-green-100 text-green-800',
                warning: 'bg-amber-100 text-amber-800',
                danger: 'bg-red-100 text-red-700',
                info: 'bg-blue-100 text-blue-800',
            },
            size: {
                sm: 'px-2 py-0.5 text-[10px]',
                md: 'px-3 py-1 text-xs',
                lg: 'px-4 py-1.5 text-sm',
            },
        },
        defaultVariants: { variant: 'default', size: 'md' },
    }
);

export interface BadgeProps extends HTMLAttributes<HTMLSpanElement>, VariantProps<typeof badgeVariants> {}

export const Badge = forwardRef<HTMLSpanElement, BadgeProps>(
    ({ className, variant, size, ...props }, ref) => (
        <span ref={ref} className={cn(badgeVariants({ variant, size, className }))} {...props} />
    )
);
Badge.displayName = 'Badge';
```

---

## Usage

```tsx
import { Badge } from '@/components/ui/badge';
import { StatusBadge } from '@/components/ui/status-badge';

<Badge variant="accent">مُمَيَّز</Badge>
<Badge variant="success" size="sm">مَدفوع</Badge>
<Badge variant="outline">تصنيف</Badge>

{/* StatusBadge يستخدم للتعامل مع حالات المنصة بشكل ديناميكي */}
<StatusBadge status="awaiting_payment" label="بانتظار الدفع" />
```

---

## القَواعِد

✅ **`rounded-full`** افتراضيّاً (pills).
✅ **font-medium** ليَكون قابِل للقِراءَة.
✅ **استَخدِم semantic colors بحَكمَة** — لا تَستَخدِم success لِشَيء غَير ناجح.
❌ **لا badge بدون نَصّ** — حَتّى مَع icon، لازِم كَلِمَة.
❌ **لا تَستَخدِم Badge كَزِرّ** — هو شارَة، لا إجراء.
