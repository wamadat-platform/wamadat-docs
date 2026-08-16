# 03.16 — Progress Indicators

> أعطِ المُستَخدِم شُعور بالتَقَدُّم. الانتِظار بدون مُؤَشِّر = قَلَق.

---

## Types

| Type | متى |
|---|---|
| `Spinner` | < 1 ثانية، المُدّة غَير مَعروفَة |
| `Progress Bar` | المُدّة مَعروفَة (file upload، course completion) |
| `Skeleton` | تَحميل مُحتوى — يُحاكي البِنية |
| `Step Indicator` | عَمَليّة مُتَعَدِّدَة الخُطوات (checkout) |

---

## Spinner

```tsx
<Loader2 className="size-5 animate-spin text-orange-500" />
```

### مع نَصّ
```tsx
<div className="flex items-center gap-3 text-jet-500">
    <Loader2 className="size-5 animate-spin" />
    <span>جارٍ التَحميل…</span>
</div>
```

---

## Progress Bar

### Determinate (نِسبَة مَعروفَة)
```tsx
<div className="space-y-2">
    <div className="flex justify-between text-sm">
        <span className="text-jet-700">إجمالي التَقدُّم</span>
        <span className="text-jet-900 font-bold tabular-nums">{percent}%</span>
    </div>
    <div className="h-2 bg-jet-100 rounded-full overflow-hidden">
        <div
            className="h-full bg-orange-500 transition-all duration-500 ease-out"
            style={{ width: `${percent}%` }}
            role="progressbar"
            aria-valuenow={percent}
            aria-valuemin={0}
            aria-valuemax={100}
        />
    </div>
</div>
```

### Indeterminate (مُدّة غَير مَعروفَة)
```tsx
<div className="h-1 bg-jet-100 overflow-hidden">
    <div className="h-full w-1/3 bg-orange-500 animate-progress-indeterminate" />
</div>
```

---

## Skeleton

```tsx
{loading ? (
    <div className="space-y-3 animate-pulse">
        <div className="h-4 bg-jet-100 rounded w-3/4" />
        <div className="h-4 bg-jet-100 rounded w-1/2" />
        <div className="h-32 bg-jet-100 rounded-2xl" />
    </div>
) : (
    <RealContent />
)}
```

---

## Step Indicator (Checkout)

```tsx
<div className="flex items-center justify-center mb-10">
    <Step number={1} label="السلّة" done />
    <span className="flex-1 h-0.5 bg-orange-500" />
    <Step number={2} label="الدَفع" active />
    <span className="flex-1 h-0.5 bg-jet-200" />
    <Step number={3} label="التَأكيد" />
</div>

function Step({ number, label, done, active }) {
    return (
        <div className="flex flex-col items-center gap-2 px-3">
            <div className={cn(
                'size-10 rounded-full font-bold flex items-center justify-center',
                done && 'bg-orange-500 text-white',
                active && 'bg-jet-900 text-orange-500 ring-4 ring-orange-100',
                !done && !active && 'bg-jet-100 text-jet-400',
            )}>
                {done ? '✓' : number}
            </div>
            <span className={cn(
                'text-xs font-bold',
                (done || active) ? 'text-jet-900' : 'text-jet-400'
            )}>
                {label}
            </span>
        </div>
    );
}
```

---

## القَواعِد

✅ **استَخدِم progress bar لو تَعرِف الـ %**.
✅ **استَخدِم spinner لو لا تَعرِف**.
✅ **استَخدِم skeleton لِتَحميل مُحتوى** (يُعطي شُعور بِسُرعَة أكثَر من spinner).
✅ **`aria-busy="true"`** على المَنطِقَة المُحَمَّلَة.
❌ **لا spinner على شيء أَسرَع من 1 ثانية** — يُسَبِّب وَميض.
❌ **لا تَدَع المُستَخدِم بدون مُؤَشِّر** أبداً.
