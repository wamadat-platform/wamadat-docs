# 03.11 — Tabs

> تَبديل بَين أقسام مُحتوى داخِل نَفس الصَفحَة.

---

## Variants

| Variant | متى |
|---|---|
| `Underline` | الافتراضيّ — خَطّ orange تحت النَشِط ⭐ |
| `Pills` | للفِلتَر القَصير |
| `Boxed` | داخِل cards |

---

## Pattern: Underline

```tsx
<div className="border-b border-jet-200">
    <nav className="flex gap-1">
        {tabs.map((tab) => (
            <button
                key={tab.key}
                onClick={() => setActive(tab.key)}
                className={cn(
                    'px-4 py-3 text-sm font-medium border-b-2 -mb-px transition-colors',
                    active === tab.key
                        ? 'border-orange-500 text-orange-700'
                        : 'border-transparent text-jet-500 hover:text-jet-700 hover:border-jet-200'
                )}
                role="tab"
                aria-selected={active === tab.key}
            >
                {tab.label}
            </button>
        ))}
    </nav>
</div>

<div className="mt-6" role="tabpanel">
    {activeContent}
</div>
```

---

## Pattern: Pills

```tsx
<nav className="flex gap-2 flex-wrap">
    {tabs.map((tab) => (
        <button
            key={tab.key}
            className={cn(
                'px-4 py-2 rounded-full text-sm font-medium transition-colors',
                active === tab.key
                    ? 'bg-jet-900 text-white'
                    : 'bg-jet-50 text-jet-700 hover:bg-jet-100'
            )}
        >
            {tab.label}
        </button>
    ))}
</nav>
```

---

## القَواعِد

✅ **3-7 tabs maximum** — أكثَر، استَخدِم dropdown.
✅ **`role="tab"` + `role="tabpanel"`** + `aria-selected`.
✅ **Keyboard navigation**: Arrow keys للتَنَقُّل بَين tabs.
❌ **لا تَستَخدِم tabs للمَسارات المُختَلِفَة** (URL routing) — استَخدِم links.
❌ **لا تُخفي محتوى مُهمّ** خَلف tab — قد لا يَكتَشِفه المُستَخدِم.
