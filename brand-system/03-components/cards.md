# 03.3 — Cards

> الكارت وِحدَة عرض المُحتوى. كلّ كارت يَحكي قِصّة كامِلَة في 200 بِكسل.

---

## Anatomy

```
┌─────────────────────────┐
│  [Cover / Header]       │  ← اختياريّ (16:9)
├─────────────────────────┤
│  [Category badge]       │  ← خَفيف
│                         │
│  [Title]                │  ← أَهَمّ شَيء
│  [Subtitle / Excerpt]   │
│                         │
│  [Meta row]             │  ← duration, lessons, rating
│                         │
│  [Footer: price + CTA]  │
└─────────────────────────┘
```

---

## Variants

| Variant | الاستخدام |
|---|---|
| `default` | للقَوائم العامّة (programs, instructors) |
| `compact` | للـ list views (dashboard items) |
| `feature` | لـ Hero/Featured items |
| `interactive` | يَتَوَسَّع على Hover (programs grid) |
| `flat` | بدون border/shadow (داخل cards أخرى) |

---

## Base Structure

```tsx
// @/components/ui/card.tsx
import { forwardRef, type HTMLAttributes } from 'react';
import { cn } from '@/lib/utils';

export const Card = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div
            ref={ref}
            className={cn(
                'rounded-xl border border-jet-100 bg-white text-jet-800 shadow-sm transition-shadow hover:shadow-md',
                className,
            )}
            {...props}
        />
    ),
);
Card.displayName = 'Card';

export const CardHeader = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('flex flex-col gap-1.5 p-6 pb-3', className)} {...props} />
    ),
);
CardHeader.displayName = 'CardHeader';

export const CardTitle = forwardRef<HTMLHeadingElement, HTMLAttributes<HTMLHeadingElement>>(
    ({ className, ...props }, ref) => (
        <h3 ref={ref} className={cn('text-xl leading-tight font-bold', className)} {...props} />
    ),
);
CardTitle.displayName = 'CardTitle';

export const CardDescription = forwardRef<HTMLParagraphElement, HTMLAttributes<HTMLParagraphElement>>(
    ({ className, ...props }, ref) => (
        <p ref={ref} className={cn('text-sm text-jet-400', className)} {...props} />
    )
);
CardDescription.displayName = 'CardDescription';

export const CardContent = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('p-6 pt-0', className)} {...props} />
    ),
);
CardContent.displayName = 'CardContent';

export const CardFooter = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('flex items-center gap-3 p-6 pt-0', className)} {...props} />
    ),
);
CardFooter.displayName = 'CardFooter';
```

---

## Usage

```tsx
<Card>
    <CardHeader>
        <CardTitle>مُهارَات التَعليق الصَوتيّ الاحتِرافيّ</CardTitle>
        <CardDescription>تَعَلَّم أساليب الإلقاء من الصِفر للاحتِراف</CardDescription>
    </CardHeader>
    <CardContent>
        <div className="flex items-center gap-3 text-sm text-jet-500">
            <Clock className="size-4" />
            <span>40 ساعَة</span>
        </div>
    </CardContent>
    <CardFooter>
        <span className="text-2xl font-bold text-orange-600">٤,١٩٥ ر.س</span>
        <Button>ابدأ</Button>
    </CardFooter>
</Card>
```

---

## Interactive Cards (Hover)

```tsx
<Card className="
    transition-all duration-200 ease-out
    hover:shadow-xl hover:-translate-y-1 cursor-pointer
    group
">
    <img className="transition-transform duration-300 group-hover:scale-105" />
    <CardContent>...</CardContent>
</Card>
```

---

## القَواعِد

✅ **Cards دائماً rounded-xl** (12px). الافتراضي في المنصة.
✅ **shadow-sm افتراضيّاً**، hover:shadow-md مبني داخل المكون الأساسي (لا تحتاج لإضافته للـ Interactive إلا لمزيد من التخصيص).
✅ **Padding 24px (`p-6`)** افتراضيّ، 32px للـ feature cards.
✅ **Border `border-jet-100`** — لون خَفيف يَفصِل بدون ضَجيج.
❌ **لا nested cards** — لا تَضَع كارت داخِل كارت.
❌ **لا backgrounds مُلَوَّنَة** — `bg-white` فقط.
