# 03.1 — Buttons

> الزِرّ عَقد بَين المُستَخدِم والمنصّة. كلّ زِرّ يَعد بشَيء — وفِّ بالوَعد.

---

## Variants

| Variant | الاستخدام | المَظهَر |
|---|---|---|
| `primary` | الإجراء الأهَمّ في الصَفحة (**واحد فقط**) | `bg-orange-500 text-jet-800` |
| `secondary` | إجراء ثانويّ هامّ | `bg-jet-800 text-white` |
| `outline` | بَديل قَويّ بدون مِلء يعتمد على الحدود | `bg-white border-jet-200 text-jet-800` |
| `ghost` | إجراء خَفيف (links داخل قَوائم) | `text-jet-800 hover:bg-jet-50` |
| `danger` | حَذف، إلغاء، إجراء لا رَجعَة فيه | `bg-danger text-white` |

---

## Sizes

| Size | Padding | استخدام |
|---|---|---|
| `sm` | `px-4 h-9` | داخل cards، forms compact |
| `md` | `px-6 h-11` | **الافتراضيّ** ⭐ |
| `lg` | `px-8 h-14` | Hero CTAs، formal forms |
| `icon` | `w-10 h-10` | أزرار الأيقونات بدون نص |

---

## States

| State | المَظهَر |
|---|---|
| Default | الـ variant base |
| Hover | يَزداد قَتاماً بـ 100 درجة (`orange-500 → orange-600`) |
| Focus | `ring-2 ring-orange-500 ring-offset-2 outline-none` |
| Active | `scale-[0.98]` (نَقرَة خَفيفَة) |
| Disabled | `opacity-50 cursor-not-allowed` |
| Loading | spinner داخل الزِرّ + النَصّ القَديم disabled |

---

## When to Use

### Primary
- ✅ زِرّ التَسجيل، الشراء، الإرسال (واحد لكلّ صَفحة)
- ✅ "ابدأ التَعلّم"، "ادفَع الآن"، "أرسِل"
- ❌ لا تَستَخدِمه مَرَّتَين في نفس القِسم — يَفقِد قُوَّته

### Secondary
- ✅ زِرّ "تَفاصيل أكثَر"، "حِفظ للاحِق"، "إلغاء"
- ✅ يَأتي بِجانب primary كَخيار بَديل

### Outline
- ✅ Alternative قَوي بِدون "ضَوضاء" بَصَريّة (Hero secondary CTA)

### Ghost
- ✅ Navigation links، أزرار "عرض المَزيد"، "إغلاق"

### Danger
- ✅ "حَذف الحِساب"، "إلغاء الاشتِراك"، "حَذف الدَفعَة"
- ⚠️ دائماً مَع `requiresConfirmation` modal

### Link
- ✅ داخل نَصّ، يَبدو كَرابِط

---

## React Example

```tsx
// @/components/ui/button.tsx
import { Slot } from '@radix-ui/react-slot';
import { cva, type VariantProps } from 'class-variance-authority';
import { type ButtonHTMLAttributes, forwardRef } from 'react';

import { cn } from '@/lib/utils';

/**
 * Brand button — Orange primary, Jet secondary, Outline tertiary.
 * No new color tokens; everything resolves to Wamadat palette.
 */
const buttonVariants = cva(
    'inline-flex items-center justify-center gap-2 rounded-lg font-medium transition-colors focus-visible:ring-2 focus-visible:ring-orange-500 focus-visible:ring-offset-2 focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50',
    {
        variants: {
            variant: {
                primary: 'text-jet-800 bg-orange-500 hover:bg-orange-600 active:bg-orange-700',
                secondary: 'bg-jet-800 hover:bg-jet-700 active:bg-jet-900 text-white',
                outline: 'border-jet-200 text-jet-800 hover:bg-jet-50 border bg-white',
                ghost: 'text-jet-800 hover:bg-jet-50',
                danger: 'bg-danger text-white hover:bg-red-700 active:bg-red-800',
            },
            size: {
                sm: 'h-9 px-4 text-sm',
                md: 'h-11 px-6 text-base',
                lg: 'h-14 px-8 text-lg',
                icon: 'h-10 w-10',
            },
        },
        defaultVariants: {
            variant: 'primary',
            size: 'md',
        },
    },
);

export interface ButtonProps
    extends ButtonHTMLAttributes<HTMLButtonElement>, VariantProps<typeof buttonVariants> {
    asChild?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
    ({ className, variant, size, asChild = false, ...props }, ref) => {
        const Comp = asChild ? Slot : 'button';
        return (
            <Comp
                className={cn(buttonVariants({ variant, size, className }))}
                ref={ref}
                {...props}
            />
        );
    },
);
Button.displayName = 'Button';

export { buttonVariants };
```

---

## استخدام

```tsx
<Button variant="primary" size="lg">
    ابدأ رِحلَتَك التَعليميّة
    <ArrowLeft className="size-5 rtl:rotate-180" />
</Button>

<Button variant="outline" size="md">
    تَصَفُّح البَرامج
</Button>

<Button variant="danger" loading={isDeleting}>
    حَذف الحِساب
</Button>

{/* As Link */}
<Button asChild>
    <Link href="/programs">ابدأ الآن</Link>
</Button>
```

---

## Accessibility

- ✅ `<button>` دائماً (ليس `<div onClick>`)
- ✅ Focus ring واضِح (`ring-2 ring-orange-500 ring-offset-2`)
- ✅ Disabled = `disabled` HTML attribute (لا فقط opacity)
- ✅ Loading state = `aria-busy="true"`
- ✅ زِرّ بأيقونَة فَقَط = `aria-label` مَطلوب
- ✅ Min hit target = 44×44px (متَوافِق على mobile)
