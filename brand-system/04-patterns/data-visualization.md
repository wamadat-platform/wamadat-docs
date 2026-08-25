# 04.10 — Data Visualization

> الأرقام تَكلَّم. لكن لا تَكذِب بـ الـ visualizations.

---

## أنواع نُحَبِّذها

### 1. Stat Cards
```tsx
<div className="bg-white rounded-2xl border border-jet-100 p-6">
    <div className="size-12 rounded-xl bg-orange-100 flex items-center justify-center mb-4">
        <TrendingUp className="size-6 text-orange-700" />
    </div>
    <div className="text-3xl font-black text-jet-900 tabular-nums">١,٥٠٠+</div>
    <div className="text-sm text-jet-500 mt-1">مُتدَرِّب نَشِط</div>
</div>
```

### 2. Progress Bars (مَع labels)
```tsx
<div className="space-y-2">
    <div className="flex justify-between text-sm">
        <span>تَقدّمك</span>
        <span className="font-bold tabular-nums">٧٥%</span>
    </div>
    <div className="h-2 bg-jet-100 rounded-full">
        <div className="h-full bg-orange-500 rounded-full" style={{ width: '75%' }} />
    </div>
</div>
```

### 3. Line / Bar Charts (مَع Recharts)
```bash
pnpm add recharts
```

```tsx
<LineChart data={revenue30Days} width={600} height={300}>
    <CartesianGrid strokeDasharray="3 3" stroke="#F0F0F0" />
    <XAxis dataKey="date" tick={{ fontSize: 12, fill: '#707070' }} />
    <YAxis tick={{ fontSize: 12, fill: '#707070' }} />
    <Tooltip
        contentStyle={{ backgroundColor: '#343333', borderRadius: '8px', border: 'none' }}
        labelStyle={{ color: '#FAAF3C' }}
        itemStyle={{ color: '#FFFFFF' }}
    />
    <Line dataKey="revenue" stroke="#FAAF3C" strokeWidth={2} dot={false} />
</LineChart>
```

---

## ألوان للـ Charts

| الاستخدام | اللون |
|---|---|
| سلسلة أَساسيّة | `#FAAF3C` (orange-500) |
| سلسلة مُقارَنَة | `#343333` (jet-900) |
| Positive trend | `#16A34A` (success) |
| Negative trend | `#DC2626` (danger) |
| Neutral / Grid | `#F0F0F0` (jet-100) |

---

## القَواعِد

✅ **Tabular nums** على كلّ الأرقام في الـ chart.
✅ **Y-axis يَبدأ من 0** افتراضيّاً (لا تَكذِب بـ scaling).
✅ **Tooltips واضِحَة** — تاريخ + قِيمَة + سلسلة.
✅ **Legend اختياريّ** لو فيه 2+ سلاسِل.
✅ **Mobile**: قَلِّل tick labels أو agg لأسبوعيّ بدل يَوميّ.
❌ **لا 3D charts** — مُضِلَّلَة.
❌ **لا pie chart مَع 7+ شَرائح** — استَخدِم bar chart.
❌ **لا تَستَخدِم ألوان rainbow عَشوائيّة** — التَزِم بِسُلَّمنا.
