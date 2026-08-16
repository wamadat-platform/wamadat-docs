# 03.1 — Buttons

> الزِرّ عَقد بَين المُستَخدِم والمنصّة. كلّ زِرّ يَعد بشَيء — وفِّ بالوَعد.

---

## Variants

| Variant | الاستخدام | المَظهَر |
|---|---|---|
| `primary` | الإجراء الأهَمّ في الصَفحة (**واحد فقط**) | `bg-orange-500 text-jet-900` |
| `secondary` | إجراء ثانويّ هامّ | `bg-white border border-jet-200 text-jet-900` |
| `outline` | بَديل قَويّ بدون مِلء | `bg-transparent border-2 border-jet-900 text-jet-900` |
| `ghost` | إجراء خَفيف (links داخل قَوائم) | `bg-transparent text-jet-700 hover:bg-jet-50` |
| `danger` | حَذف، إلغاء، إجراء لا رَجعَة فيه | `bg-red-600 text-white` |
| `link` | يَبدو كَرابِط لكن سُلوكه زِرّ | `text-orange-700 underline` |

---

## Sizes

| Size | Padding | Text | استخدام |
|---|---|---|---|
| `xs` | `px-2 py-1` | `text-xs` | داخل tables، tags |
| `sm` | `px-3 py-1.5` | `text-sm` | داخل cards، forms compact |
| `md` | `px-4 py-2` | `text-base` | **الافتراضيّ** ⭐ |
| `lg` | `px-6 py-3` | `text-lg` | Hero CTAs، formal forms |
| `xl` | `px-8 py-4` | `text-xl` | Landing page main CTA |

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
// frontend/components/ui/button.tsx
import * as React from 'react';
import { Slot } from '@radix-ui/react-slot';
import { Loader2 } from 'lucide-react';
import { cn } from '@/lib/utils';

const variants = {
    primary: 'bg-orange-500 text-jet-900 hover:bg-orange-600 active:bg-orange-700',
    secondary: 'bg-white border border-jet-200 text-jet-900 hover:bg-jet-50',
    outline: 'bg-transparent border-2 border-jet-900 text-jet-900 hover:bg-jet-900 hover:text-white',
    ghost: 'bg-transparent text-jet-700 hover:bg-jet-50',
    danger: 'bg-red-600 text-white hover:bg-red-700',
    link: 'text-orange-700 underline underline-offset-4 hover:text-orange-800',
};

const sizes = {
    xs: 'px-2 py-1 text-xs h-7',
    sm: 'px-3 py-1.5 text-sm h-9',
    md: 'px-4 py-2 text-base h-10',
    lg: 'px-6 py-3 text-lg h-12',
    xl: 'px-8 py-4 text-xl h-14',
};

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
    variant?: keyof typeof variants;
    size?: keyof typeof sizes;
    asChild?: boolean;
    loading?: boolean;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
    ({ variant = 'primary', size = 'md', asChild = false, loading = false, className, children, disabled, ...props }, ref) => {
        const Comp = asChild ? Slot : 'button';
        return (
            <Comp
                ref={ref}
                disabled={disabled || loading}
                className={cn(
                    'inline-flex items-center justify-center gap-2 rounded-lg font-bold',
                    'transition-all duration-200 ease-out',
                    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-500 focus-visible:ring-offset-2',
                    'active:scale-[0.98]',
                    'disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:bg-current',
                    variants[variant],
                    sizes[size],
                    className,
                )}
                {...props}
            >
                {loading && <Loader2 className="size-4 animate-spin" />}
                {children}
            </Comp>
        );
    },
);
Button.displayName = 'Button';
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
