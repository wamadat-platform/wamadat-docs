# 03.12 — Accordions (Expandable)

> تَوسيع/طَيّ مُحتوى. للـ FAQs، curriculum، تَفاصيل اختياريّة.

---

## Pattern

```tsx
<div className="space-y-2">
    {items.map((item, idx) => (
        <details key={idx} className="group bg-white rounded-xl border border-jet-100">
            <summary className="
                flex items-center justify-between p-4 cursor-pointer
                list-none [&::-webkit-details-marker]:hidden
            ">
                <h3 className="font-bold text-jet-900">{item.question}</h3>
                <ChevronDown className="size-5 text-jet-400 transition-transform group-open:rotate-180" />
            </summary>
            <div className="px-4 pb-4 text-jet-700 leading-relaxed">
                {item.answer}
            </div>
        </details>
    ))}
</div>
```

---

## Variants

### Default (Closed)
- خَلفيّة `bg-white` + border خَفيف
- العُنوان مَع chevron يَدُلّ "اضغُط لتَفتَح"

### Open
- نَفس الخَلفيّة + المحتوى ظاهِر
- Chevron يَنقَلِب 180° (`group-open:rotate-180`)

---

## القَواعِد

✅ **Native `<details>/<summary>`** — accessibility مَجّاناً، يَعمَل بدون JS.
✅ **Chevron يَدور للإشارَة** (لا "+/-").
✅ **محتوى يَتَوَسَّع بحَرَكَة سَلِسَة** (CSS transition).
✅ **`group-open:` للـ Tailwind** يُتيح تَخصيص ابن العُنصُر المَفتوح.
❌ **لا accordion داخل accordion** (nested) — مُربِك.
❌ **لا تَفتَح كلّ الـ items بشَكل افتراضيّ** — يَفقِد الـ progressive disclosure.
