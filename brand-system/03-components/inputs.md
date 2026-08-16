# 03.2 — Inputs

> الـ Input صَدفَة الثِقَة بين المُستَخدِم والمنصّة. كلّ خانة تَطلُب بَيانات يَجب أن تُبَرِّر طَلَبها.

---

## Variants

- `text` — الافتراضيّ
- `email`
- `tel` (مَع validation عَربيّ/سعوديّ)
- `password` (مَع reveal toggle)
- `number` (مَع tabular-nums)
- `textarea` (مُتَعَدِّد الأسطر)
- `select` (قائِمَة منسدِلَة)

---

## States

| State | Border | Text |
|---|---|---|
| Default | `border-jet-200` | `text-jet-900` |
| Focus | `border-orange-500 ring-2 ring-orange-200` | `text-jet-900` |
| Filled | `border-jet-300` | `text-jet-900` |
| Error | `border-red-500 ring-2 ring-red-100` | `text-jet-900` + error msg |
| Disabled | `border-jet-200 bg-jet-50` | `text-jet-400` |
| Read-only | `border-jet-100 bg-jet-50` | `text-jet-700` |

---

## Anatomy

```
[Label *required]              ← font-medium، text-sm، text-jet-700
[============ Input ============]
[ ↳ Helper text or error]      ← text-xs، text-jet-500 (or text-red-600)
```

---

## React Example

```tsx
// frontend/components/ui/input.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';

export interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
    error?: boolean;
}

export const Input = React.forwardRef<HTMLInputElement, InputProps>(
    ({ className, error, ...props }, ref) => (
        <input
            ref={ref}
            className={cn(
                'h-12 w-full rounded-lg px-4 text-base text-jet-900 bg-white',
                'border border-jet-200 placeholder:text-jet-400',
                'transition-colors duration-200',
                'focus:outline-none focus:border-orange-500 focus:ring-2 focus:ring-orange-100',
                'disabled:bg-jet-50 disabled:text-jet-400 disabled:cursor-not-allowed',
                error && 'border-red-500 ring-2 ring-red-100 focus:border-red-500',
                className,
            )}
            {...props}
        />
    ),
);
Input.displayName = 'Input';

export const Label = React.forwardRef<HTMLLabelElement, React.LabelHTMLAttributes<HTMLLabelElement> & { required?: boolean }>(
    ({ className, children, required, ...props }, ref) => (
        <label
            ref={ref}
            className={cn('block text-sm font-medium text-jet-700 mb-1.5', className)}
            {...props}
        >
            {children}
            {required && <span className="text-red-600 ms-1">*</span>}
        </label>
    ),
);
Label.displayName = 'Label';
```

---

## Usage

```tsx
<div>
    <Label htmlFor="email" required>البريد الإلكترونيّ</Label>
    <Input id="email" type="email" placeholder="you@example.com" dir="ltr" />
    <p className="mt-1.5 text-xs text-jet-500">لن نُشارك بَريدك مع أيّ جِهَة.</p>
</div>

{/* مع خَطَأ */}
<div>
    <Label htmlFor="password" required>كلمة المرور</Label>
    <Input id="password" type="password" error />
    <p className="mt-1.5 text-xs text-red-600">كلمة المرور قَصيرَة جدّاً.</p>
</div>
```

---

## RTL & العَربيّة

- **العَربيّة**: `dir="rtl"` (الافتراضيّ على الصَفحَة)
- **اللاتينيّة المَطلوبَة**: emails, URLs, phone — `dir="ltr"` على الـ input نَفسه
- **placeholder عَربيّ**: مَسموح (لا يَنعَكِس على المُحاذاة)

```tsx
{/* بَريد إلكترونيّ — يَبقى LTR */}
<Input type="email" dir="ltr" placeholder="you@example.com" />

{/* اسم عَربيّ — RTL */}
<Input type="text" placeholder="الاسم الكامل" />
```

---

## Validation Patterns

كلّ خَطَأ يَتَبَع نَمَط:
1. لا يَظهَر قَبل الـ blur الأوّل
2. يَختَفي حَين يُصَحِّح المُستَخدِم
3. لُغَتُه إنسانيّة (لا "Error 422")

```tsx
{form.formState.errors.email && (
    <p className="mt-1.5 text-xs text-red-600 flex items-center gap-1">
        <AlertCircle className="size-3.5" />
        {form.formState.errors.email.message}
    </p>
)}
```

---

## Accessibility

- ✅ `<label for="x">` مُرتَبِط بـ `id="x"`
- ✅ `aria-invalid={!!error}` على error
- ✅ `aria-describedby={helperId}` للـ helper text
- ✅ `aria-required="true"` على المَطلوب
- ✅ Visible focus state — لا تَخفيه أبداً
