# 04.5 — Loading States

> الانتِظار حَتميّ. اجعَله مُحتَرَماً.

---

## Strategy

| الـ Wait Time | الـ Pattern |
|---|---|
| < 100ms | لا شَيء (المُستَخدِم لا يَشعُر) |
| 100ms-1s | Spinner خَفيف |
| 1s-3s | Skeleton أو Progress |
| > 3s | Progress + رَسالَة "هذا يَأخذ وَقتاً عادَة" |

---

## Patterns

### Initial Page Load
```tsx
{loading ? (
    <div className="space-y-4 animate-pulse">
        <div className="h-8 bg-jet-100 rounded w-1/2" />
        <div className="h-4 bg-jet-100 rounded w-3/4" />
        <div className="grid grid-cols-3 gap-4">
            <div className="h-48 bg-jet-100 rounded-2xl" />
            <div className="h-48 bg-jet-100 rounded-2xl" />
            <div className="h-48 bg-jet-100 rounded-2xl" />
        </div>
    </div>
) : (
    <RealContent />
)}
```

### Button Submit
```tsx
<Button loading={isSubmitting}>
    {isSubmitting ? 'جارٍ الإرسال…' : 'إرسال'}
</Button>
```

### Polling
```tsx
{polling && (
    <div className="text-center py-8 text-jet-500">
        <Loader2 className="size-6 animate-spin mx-auto mb-2" />
        <p>نَنتَظِر تَأكيد الدَفع…</p>
        <p className="text-xs text-jet-400 mt-1">عادَة يَأخذ ثوانٍ مَعدودَة</p>
    </div>
)}
```

### List Loading More
```tsx
{loadingMore && (
    <div className="col-span-full flex items-center justify-center py-8">
        <Loader2 className="size-5 animate-spin text-orange-500" />
    </div>
)}
```

---

## القَواعِد

✅ **Skeleton أَفضَل من Spinner** لِلتَحميل الأَوَّليّ (يُعطي شَكل المحتوى).
✅ **Spinner للأَفعال** (button submit، fetch trigger).
✅ **رَسالَة تَوضيحيّة بَعد 3 ثَوانٍ** ("هذا يَستَغرِق وَقتاً أَحياناً").
✅ **`aria-busy="true"`** على الـ region.
❌ **لا spinner كَبير في وَسط الشاشَة** كلّ مَرّة — يُبَطِّئ شُعور السُرعَة.
❌ **لا تَستَخدِم loading state غامض** بدون تَوضيح ماذا يَحدُث.
