# 02.4 — الشَبَكَة (Grid)

> Container واحد، Breakpoints مَدروسَة، أعمِدَة مَرِنَة.

---

## الـ Containers

```
sm:  640px   ← Mobile landscape, tablets vertical
md:  768px   ← Tablets
lg:  1024px  ← Laptops
xl:  1280px  ← Desktops ⭐ (الافتراضيّ)
2xl: 1536px  ← Wide screens
```

```tsx
<div className="container mx-auto px-4 md:px-6 lg:px-8">
    {/* المحتوى */}
</div>
```

> **القاعِدة:** كلّ صَفحة يَجب أن تَكون مَحدودَة بـ `container` — لا نَملأ الشاشة الكامِلَة على 4K بنَصّ يَمتَدّ.

---

## الـ Breakpoints

| Token | Pixel | الجِهاز |
|---|---|---|
| `sm` | 640px | جَوّال (landscape)، تابليت صَغير |
| `md` | 768px | تابليت |
| `lg` | 1024px | لابتوب |
| `xl` | 1280px | ديسكتوب |
| `2xl` | 1536px | شاشات عَريضَة |

---

## الأعمِدَة (Columns)

| الجِهاز | الأعمِدَة الافتراضيّة | Gap |
|---|---|---|
| Mobile (<640px) | 1 col | `gap-4` (16px) |
| Tablet (640-1024px) | 2-3 cols | `gap-4` |
| Desktop (>1024px) | 3-4 cols | `gap-6` (24px) |
| Wide (>1536px) | 4-6 cols | `gap-8` (32px) |

---

## أنماط Grid شائِعَة

### Cards Grid (للبَرامج، المُدرِّبين)
```tsx
<div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
    <Card />
    <Card />
    ...
</div>
```

### 2-Column Layout (Form + Sidebar)
```tsx
<div className="grid gap-6 lg:grid-cols-[1fr_400px]">
    <main>...</main>
    <aside>...</aside>
</div>
```

### 12-Column (مَرَن)
```tsx
<div className="grid grid-cols-12 gap-6">
    <div className="col-span-12 lg:col-span-8">...</div>
    <div className="col-span-12 lg:col-span-4">...</div>
</div>
```

### Hero Split (Content + Image)
```tsx
<div className="grid gap-8 lg:grid-cols-2 items-center">
    <div>{/* content */}</div>
    <div>{/* image */}</div>
</div>
```

### Dashboard (Sidebar + Main)
```tsx
<div className="flex">
    <aside className="hidden lg:flex w-64">...</aside>
    <main className="flex-1 min-w-0">...</main>
</div>
```

---

## القَواعِد

### ✅ نَفعَل
- استَخدِم `grid-cols-{n}` للـ grids المُنتَظِمَة.
- استَخدِم `lg:` و `md:` للـ responsive (mobile-first).
- استَخدِم `min-w-0` على flex items التي قد تَفيض.

### ❌ لا نَفعَل
- لا نَستَخدِم `width: 33.33%` ثابِت — استَخدِم Grid.
- لا نُلصِق `max-width` عَشوائيّ على عَناصر منفَردَة — استَخدِم Container.
- لا نَستَخدِم `<table>` للـ layout (التَّيبل للبَيانات الجَدوَليّة فقط).

---

## RTL Considerations

كلّ شَيء في Grid يَعمل في RTL تلقائيّاً (Tailwind يَدعَم `dir="rtl"` افتراضيّاً).

**استَثناء واحِد:** `space-x-*` و `space-y-*` تَحتاج `rtl:space-x-reverse` للـ horizontal stacks في RTL:

```tsx
<div className="flex space-x-3 rtl:space-x-reverse">
    <Item />
    <Item />
</div>
```

> **حَلّ أبسَط:** استَخدِم `gap-*` بدل `space-x-*` — يَعمل في كلا الاتِّجاهَين بدون reverse.

---

## أمثلة Real-world من المنصّة

### صَفحة البرامج
```tsx
<div className="container mx-auto px-4">
    <header className="py-12">...</header>
    <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
        {programs.map(p => <ProgramCard key={p.id} program={p} />)}
    </div>
</div>
```

### تَفاصيل البرنامج
```tsx
<div className="container mx-auto px-4 lg:px-8 py-8">
    <div className="grid gap-8 lg:grid-cols-[1fr_360px]">
        <main className="space-y-8">
            <Hero />
            <Curriculum />
            <Instructor />
            <Reviews />
        </main>
        <aside className="lg:sticky lg:top-24 h-fit">
            <PriceCard />
        </aside>
    </div>
</div>
```

### Dashboard
```tsx
<div className="flex min-h-screen">
    <Sidebar className="hidden lg:flex w-64" />
    <main className="flex-1 min-w-0 p-6 lg:p-10">
        {children}
    </main>
</div>
```

---

## Breaking out of Container

أحياناً نَحتاج عُنصُر يَمتَدّ على عَرض الشاشة (Hero مَع خَلفيّة لون كامِل):

```tsx
{/* Hero بخَلفيّة كامِلَة */}
<section className="bg-jet-900">
    <div className="container mx-auto px-4 py-24">
        {/* المحتوى داخِل container، الخَلفيّة كامِلَة */}
    </div>
</section>
```

> الـ section يَأخُذ الخَلفيّة كامِلَة، الـ container يَحُدّ المحتوى.
