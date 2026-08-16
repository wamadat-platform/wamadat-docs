# 04.9 — Filter Patterns

> الفِلتَر يُضَيِّق الفَجوة بَين المُستَخدِم والمحتوى. سَهل، مَرئيّ، عَكسيّ.

---

## أنواع

### 1. Filter Pills (الأَكثَر شِيوعاً)
- صَفّ أُفُقيّ، scroll horizontal على mobile
- Active state واضِح
- "كلّ" pill أَوَّلاً

```tsx
<nav className="flex items-center gap-2 overflow-x-auto whitespace-nowrap">
    <FilterPill href="/programs" active={!category}>الكلّ (٣٠)</FilterPill>
    {categories.map(c => (
        <FilterPill key={c.slug} href={`/programs?category=${c.slug}`} active={category === c.slug}>
            {c.name_ar} ({c.count})
        </FilterPill>
    ))}
</nav>

function FilterPill({ href, label, active, count, children }) {
    return (
        <a href={href} className={cn(
            'inline-flex items-center gap-1.5 px-4 py-2 rounded-full text-sm font-medium transition-colors',
            active
                ? 'bg-jet-900 text-white'
                : 'bg-jet-50 text-jet-700 hover:bg-jet-100'
        )}>
            {children ?? label}
        </a>
    );
}
```

### 2. Sidebar Filters (للقَوائم الكَبيرَة)
- Sticky على الجانِب
- Checkboxes للـ multi-select
- Range sliders للأسعار

```tsx
<aside className="w-64 hidden lg:block sticky top-24 self-start">
    <div className="space-y-6">
        <FilterGroup title="المُستَوى">
            <FilterCheckbox label="مُبتَدِئ" count={12} />
            <FilterCheckbox label="مُتَوَسِّط" count={8} />
            <FilterCheckbox label="مُتَقَدِّم" count={4} />
        </FilterGroup>

        <FilterGroup title="السِعر">
            <PriceRange min={0} max={50000} />
        </FilterGroup>
    </div>
</aside>
```

### 3. Bottom Sheet (Mobile)
- يَنبَثِق من الأسفَل
- Filters كامِلَة في sheet
- "تَطبيق" + "مَسح الكلّ" في الـ footer

---

## State Management

```tsx
// URL هو مَصدَر الحَقيقَة — يُتيح bookmark + share
const params = useSearchParams();
const category = params.get('category');
const difficulty = params.get('difficulty');

// Update عَبر router
router.push(`/programs?category=${slug}&difficulty=${level}`);
```

---

## الإلغاء (Clear Filters)

```tsx
{hasActiveFilters && (
    <button onClick={clearAll} className="text-sm text-orange-700 hover:underline">
        مَسح كلّ الفِلاتِر
    </button>
)}
```

---

## القَواعِد

✅ **Counts على كلّ filter** — يُساعِد المُستَخدِم يَتَوَقَّع
✅ **URL state** — مَنتَدى، bookmark، شارَك
✅ **Mobile bottom sheet** — لا sidebar مَضغوط
✅ **مَسح الكلّ زِرّ** ظاهِر عند فِلاتر نَشِطَة
❌ **لا 10+ فِلاتر** على مَرَّة — اكتَشِف الأَهمّ
❌ **لا فِلاتر مَخفيّة افتراضيّاً** — اعرِض الـ top 5
