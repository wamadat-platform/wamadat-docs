# 03.6 — Tables

> الـ Table للبَيانات الجَدوَليّة فقط. ليس لِلتَخطيط (layout).

---

## Anatomy

```
┌── Header (sticky on scroll) ────────────────────┐
│  ID │ الاسم   │ السِعر   │ الحالَة │ إجراءات   │
├─────┼─────────┼─────────┼────────┼────────────┤
│  01 │ بَرنامج │ ٤,١٩٥   │ مَدفوع │ [...]      │
│  02 │ بَرنامج │ ٢,٧٥٠   │ مُعلَّق │ [...]      │
└─────────────────────────────────────────────────┘
[Pagination: 1 2 3 ... »]
```

---

## Styles

```tsx
<table className="w-full text-sm">
    <thead>
        <tr className="border-b border-jet-200">
            <th className="text-start py-3 px-4 font-bold text-jet-700">رقم الطلب</th>
            <th className="text-start py-3 px-4 font-bold text-jet-700">العَميل</th>
            <th className="text-end py-3 px-4 font-bold text-jet-700">الإجمالي</th>
        </tr>
    </thead>
    <tbody>
        <tr className="border-b border-jet-100 hover:bg-jet-50/50 transition-colors">
            <td className="py-3 px-4 font-mono text-jet-900">ORD-2026-0001</td>
            <td className="py-3 px-4 text-jet-700">سارة الأحمد</td>
            <td className="py-3 px-4 text-end tabular-nums font-bold">٤,١٩٥ ر.س</td>
        </tr>
    </tbody>
</table>
```

---

## القَواعِد

✅ **Header row** أَثقَل (font-bold)، خَطّ سُفلي أَكثَف (border-jet-200).
✅ **Rows alternating** اختياريّ — استَخدِم hover state بَدَلاً منه.
✅ **Numeric columns** = `text-end tabular-nums`.
✅ **Long text** = `truncate` أو `line-clamp-1`.
✅ **Sticky header** على scroll: `sticky top-0 bg-white z-10`.
✅ **Responsive**: على mobile، أَخفِ الأعمِدَة الأقَلّ أَهَمّيَّة بـ `hidden md:table-cell`.

❌ **لا shadows على tables** — الـ structure نَفسها كافيَة.
❌ **لا rounded corners على table** نَفسها — استَخدِم `rounded-2xl` على container.

---

## Filament Tables

في Filament admin، استَخدِم الـ utilities المُضَمَّنَة:

```php
Tables\Columns\TextColumn::make('order_number')
    ->fontFamily('mono')
    ->copyable()
    ->searchable()
    ->sortable();

Tables\Columns\TextColumn::make('total_halalas')
    ->formatStateUsing(fn ($state) => number_format($state / 100, 2) . ' ر.س')
    ->alignment('end');

Tables\Columns\BadgeColumn::make('status')
    ->colors(['success' => 'paid', 'warning' => 'pending']);
```
