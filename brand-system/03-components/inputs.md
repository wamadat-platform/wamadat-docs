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
// @/components/ui/input.tsx
import { forwardRef, type InputHTMLAttributes, type TextareaHTMLAttributes } from 'react';
import { cn } from '@/lib/utils';

export const Input = forwardRef<HTMLInputElement, InputHTMLAttributes<HTMLInputElement>>(
    ({ className, type = 'text', ...props }, ref) => (
        <input
            ref={ref}
            type={type}
            className={cn(
                'flex h-12 w-full rounded-lg border border-jet-200 bg-white px-4 py-2 text-base text-jet-800 transition-colors',
                'placeholder:text-jet-400',
                'focus:border-orange-500 focus:ring-2 focus:ring-orange-500 focus:outline-none',
                'disabled:cursor-not-allowed disabled:opacity-50',
                className,
            )}
            {...props}
        />
    ),
);
Input.displayName = 'Input';

export const Textarea = forwardRef<HTMLTextAreaElement, TextareaHTMLAttributes<HTMLTextAreaElement>>(
    ({ className, rows = 5, ...props }, ref) => (
        <textarea
            ref={ref}
            rows={rows}
            className={cn(
                'flex w-full rounded-lg border border-jet-200 bg-white px-4 py-3 text-base text-jet-800 transition-colors',
                'placeholder:text-jet-400',
                'focus:border-orange-500 focus:ring-2 focus:ring-orange-500 focus:outline-none',
                'resize-y disabled:cursor-not-allowed disabled:opacity-50',
                className,
            )}
            {...props}
        />
    )
);
Textarea.displayName = 'Textarea';

export function Label({
    htmlFor,
    children,
    className,
    required,
}: {
    htmlFor?: string;
    children: React.ReactNode;
    className?: string;
    required?: boolean;
}) {
    return (
        <label
            htmlFor={htmlFor}
            className={cn('mb-2 block text-sm font-medium text-jet-800', className)}
        >
            {children}
            {required && <span className="me-1 text-orange-900">*</span>}
        </label>
    );
}
```

---

## Usage

```tsx
<div>
    <Label htmlFor="email" required>البريد الإلكترونيّ</Label>
    <Input id="email" type="email" placeholder="you@example.com" dir="ltr" />
    <p className="mt-1.5 text-xs text-jet-500">لن نُشارك بَريدك مع أيّ جِهَة.</p>
</div>

{/* مع خَطَأ (استخدم FormField و FormMessage من react-hook-form عوضاً عن تمرير error) */}
<FormField
    control={form.control}
    name="password"
    render={({ field }) => (
        <FormItem>
            <FormLabel required>كلمة المرور</FormLabel>
            <FormControl>
                <Input type="password" {...field} />
            </FormControl>
            <FormMessage />
        </FormItem>
    )}
/>
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
