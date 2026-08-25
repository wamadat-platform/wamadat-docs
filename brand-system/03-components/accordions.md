# 03.12 — Accordions (Expandable)

> تَوسيع/طَيّ مُحتوى. للـ FAQs، curriculum، تَفاصيل اختياريّة.

---

## Pattern

```tsx
import {
    Accordion,
    AccordionContent,
    AccordionItem,
    AccordionTrigger,
} from '@/components/ui/accordion';

// ...

<Accordion type="single" collapsible className="w-full space-y-2">
    {items.map((item, idx) => (
        <AccordionItem key={idx} value={`item-${idx}`} className="bg-white rounded-xl border border-jet-100 overflow-hidden px-4">
            <AccordionTrigger>
                {item.question}
            </AccordionTrigger>
            <AccordionContent>
                {item.answer}
            </AccordionContent>
        </AccordionItem>
    ))}
</Accordion>
```

---

## Variants

### Default (Closed)
- خَلفيّة `bg-white` + border خَفيف `border-jet-100`
- الـ `AccordionTrigger` يدير حالة الـ Chevron داخلياً.

### Open
- نَفس الخَلفيّة + المحتوى ظاهِر
- الـ Chevron المدمج في `AccordionTrigger` يَنقَلِب تلقائياً بناءً على الحالة.

---

## القَواعِد

✅ **استخدم Radix Accordion (`@/components/ui/accordion`)** — يضمن Keyboard Navigation و Accessibility (مثل `aria-controls` و `aria-expanded`) بشكل مجاني.
✅ **Chevron يَدور للإشارَة** — مدمج تلقائياً في `AccordionTrigger`.
✅ **محتوى يَتَوَسَّع بحَرَكَة سَلِسَة** — Radix يتعامل مع الـ Animation بالتعاون مع Tailwind.
❌ **لا accordion داخل accordion** (nested) — مُربِك ويصعب التنقل فيه بلوحة المفاتيح.
❌ **تجنب HTML `<details>` المباشر** — استخدم المكون الجاهز للحصول على الـ Accessibility الكاملة.
