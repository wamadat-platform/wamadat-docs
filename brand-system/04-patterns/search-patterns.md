# 04.8 — Search Patterns

> البَحث = نِيَّة المُستَخدِم. اخدِمها بسُرعَة.

---

## أنواع البَحث في ومضات

### 1. Typeahead (في Navbar)
- يَفتَح بـ click على المُدخَل
- يَظهَر نَتائج بَعد كَتابَة **حَرفَين أو أكثَر**
- يُظهِر: برامج (top 5) + مُدرِّبين (top 3)
- "اضغُط Enter لِنَتائج كامِلَة"

### 2. Full Search Page (`/search?q=...`)
- نَتائج كامِلَة مَع filters + sorting
- Grid view
- Pagination

### 3. In-page Filter (داخل /programs)
- يُلغي قائِمَة الفِئَة — لا يَتَنَقَّل لِصَفحَة بَحث

---

## Typeahead Pattern

```tsx
<Combobox>
    <Combobox.Input
        placeholder="ابحَث عن بَرنامج، مُدرِّب…"
        className="h-12 w-full ps-12 rounded-full bg-jet-50 border border-jet-200 focus:bg-white focus:border-orange-500"
    />
    <Search className="absolute start-4 top-1/2 -translate-y-1/2 size-5 text-jet-400" />

    {results.length > 0 && (
        <Combobox.Options className="absolute top-full mt-2 w-full bg-white rounded-2xl shadow-2xl border border-jet-100 max-h-96 overflow-auto">
            {/* Programs */}
            <div className="p-2">
                <h3 className="px-3 py-2 text-xs font-bold text-jet-500 uppercase tracking-wide">البَرامج</h3>
                {programs.map(p => (
                    <Combobox.Option key={p.id} value={p}>
                        {({ active }) => (
                            <div className={cn('flex items-center gap-3 p-3 rounded-lg', active && 'bg-orange-50')}>
                                <BookOpen className="size-5 text-orange-500" />
                                <span className="text-jet-900">{p.title}</span>
                            </div>
                        )}
                    </Combobox.Option>
                ))}
            </div>
        </Combobox.Options>
    )}
</Combobox>
```

---

## Search Results Page

```
┌──────────────────────────────────────┐
│  نَتائج بَحث "تَسويق" (24 نَتيجَة)    │
├──────────────────────────────────────┤
│  Filter Pills (الفِئَة، الـ level)      │
│  Sort: relevance / newest / rating   │
├──────────────────────────────────────┤
│  [Grid of cards]                     │
└──────────────────────────────────────┘
```

---

## Empty Search

```
Icon: Search
Title: لم نَجِد ما يُطابِق بَحثَك
Description: جَرِّب كَلِمات مُختَلِفَة أو تَصَفَّح الفِئات.
CTA: تَصَفَّح كلّ البَرامج / مَسح البَحث
```

---

## القَواعِد

✅ **Debounce 200ms** على الـ keystrokes (لا API call على كلّ حَرف).
✅ **Highlight النَصّ المُطابِق** في النَتيجَة.
✅ **`Cmd/Ctrl+K`** يَفتَح البَحث (keyboard shortcut).
✅ **Loading state** خَفيف أثناء البَحث.
❌ **لا بَحث على المنطِقَة كاملاً** — قَيِّد النِطاق (programs vs all).
❌ **لا تَستَبدِل البَحث بـ AI chatbot** — البَحث الكَلاسيكيّ أَسرَع.
