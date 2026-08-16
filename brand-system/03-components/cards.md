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
// frontend/components/ui/card.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';

export const Card = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div
            ref={ref}
            className={cn(
                'rounded-2xl bg-white border border-jet-100 shadow-sm overflow-hidden',
                className,
            )}
            {...props}
        />
    ),
);

export const CardHeader = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('p-6 pb-0', className)} {...props} />
    ),
);

export const CardTitle = React.forwardRef<HTMLHeadingElement, React.HTMLAttributes<HTMLHeadingElement>>(
    ({ className, ...props }, ref) => (
        <h3 ref={ref} className={cn('text-xl font-bold text-jet-900 leading-tight', className)} {...props} />
    ),
);

export const CardDescription = React.forwardRef<HTMLParagraphElement, React.HTMLAttributes<HTMLParagraphElement>>(
    ({ className, ...props }, ref) => (
        <p ref={ref} className={cn('text-sm text-jet-500 mt-1.5', className)} {...props} />
    ),
);

export const CardContent = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('p-6', className)} {...props} />
    ),
);

export const CardFooter = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
    ({ className, ...props }, ref) => (
        <div ref={ref} className={cn('p-6 pt-0 flex items-center justify-between', className)} {...props} />
    ),
);
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

✅ **Cards دائماً rounded-2xl** (16px). لا rounded-xl، لا rounded-3xl.
✅ **shadow-sm افتراضيّاً**، shadow-xl على hover (للـ interactive).
✅ **Padding 24px (`p-6`)** افتراضيّ، 32px للـ feature cards.
✅ **Border `border-jet-100`** — لون خَفيف يَفصِل بدون ضَجيج.
❌ **لا nested cards** — لا تَضَع كارت داخِل كارت.
❌ **لا backgrounds مُلَوَّنَة** — `bg-white` فقط.
