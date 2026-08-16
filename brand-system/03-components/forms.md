# 03.7 — Forms

> النَموذج عَقد. كلّ خانة تَطلُب بَيانات يَجب أن تُبَرِّر طَلَبها للمُستَخدِم.

---

## Anatomy

```
┌── Form ────────────────────────────────────┐
│                                            │
│  [Label *required]                         │
│  [Input ============]                      │
│  [↳ Helper text]                           │
│                                            │
│  [Label *required]    [Label]              │
│  [Input ==========]   [Input ==========]   │
│                                            │
│  [Textarea (full width)]                   │
│  [============================]            │
│                                            │
│  [Checkbox] أُوافِق على الشُروط           │
│                                            │
│  [Cancel]                  [Submit]        │
└────────────────────────────────────────────┘
```

---

## Layout

### Single Column
```tsx
<form className="space-y-5 max-w-md">
    <Field />
    <Field />
    <Button type="submit">إرسال</Button>
</form>
```

### Two Column (Responsive)
```tsx
<form className="space-y-5 max-w-2xl">
    <div className="grid sm:grid-cols-2 gap-4">
        <Field />
        <Field />
    </div>
    <Field className="col-span-2" />
    <Button>إرسال</Button>
</form>
```

---

## Validation Patterns

### الـ Library: React Hook Form

```tsx
import { useForm } from 'react-hook-form';

const form = useForm<FormFields>({
    defaultValues: { email: '', name: '' },
});

async function onSubmit(values: FormFields) {
    const res = await api.post('endpoint', values);
    if (!res.ok) {
        // Map server errors to fields
        for (const [field, msgs] of Object.entries(res.error.errors)) {
            form.setError(field as keyof FormFields, { message: msgs[0] });
        }
        return;
    }
    // Success
}
```

### When to Show Errors
- ❌ **عَلى كلّ keystroke** — مُتَوَتِّر، يَفقِد المُستَخدِم الثِقَة
- ✅ **على blur الأوّل** — بَعد ما المُستَخدِم انتَهى من الخانَة
- ✅ **على submit** — قَبل الإرسال للخادِم
- ✅ **يَختَفي عند التَصحيح** — تَدفُّق إيجابيّ

---

## CTA Patterns

### Single Action
```tsx
<Button type="submit" disabled={form.formState.isSubmitting}>
    {form.formState.isSubmitting ? 'جارٍ الإرسال...' : 'إرسال'}
</Button>
```

### Cancel + Submit
```tsx
<div className="flex gap-3 justify-end">
    <Button variant="ghost" type="button" onClick={onCancel}>إلغاء</Button>
    <Button type="submit">حِفظ</Button>
</div>
```

---

## Field Types

| Type | الاستخدام |
|---|---|
| `Input type="text"` | أسماء، عُناوين قَصيرَة |
| `Input type="email"` | بَريد إلكترونيّ + `dir="ltr"` |
| `Input type="tel"` | هاتف + validation سُعوديّ |
| `Input type="password"` + reveal toggle | كَلِمَة المرور |
| `Input type="number"` | الأرقام + `tabular-nums` |
| `Textarea` | نَصّ طَويل |
| `Select` | اختيار من قائِمَة |
| `Checkbox` | نَعَم/لا |
| `Radio group` | اختيار من 2-5 خيارات |
| `Switch` | تَفعيل/إيقاف |

---

## Loading & Success States

### Loading (أثناء الإرسال)
```tsx
<Button loading={form.formState.isSubmitting}>
    إرسال
</Button>
```

### Success (بعد الإرسال)
```tsx
{submitted ? (
    <div className="rounded-2xl bg-green-50 border border-green-200 p-8 text-center">
        <Check className="size-12 mx-auto text-green-600 mb-3" />
        <h3 className="text-xl font-bold text-jet-900 mb-2">تَمّ الإرسال بنَجاح</h3>
        <p className="text-sm text-jet-600">سَنَتَواصَل معك خِلال يوم عَمَل واحِد.</p>
    </div>
) : (
    <form>...</form>
)}
```

---

## القَواعِد

✅ **Label يَسبق Input دائماً** (لا placeholder كَـ label).
✅ **`required` ظاهِر** بنَجمَة حَمراء.
✅ **Helper text** اختياريّ، يُفَسِّر "لماذا نَطلُب هذا".
✅ **Submit button** آخِر شَيء في الـ form (و enter يُفَعِّله).
❌ **لا تَطلُب بَيانات لا تَحتاجها** — كلّ خانة تَزيد abandonment.
❌ **لا تَستَخدِم Modals لـ forms طَويلَة** — صَفحَة مُستَقِلَّة.
