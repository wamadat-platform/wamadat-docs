# 04.2 — Hero Sections

> الـ Hero هي أوّل 3 ثَوانٍ. يُحَدِّد ما إذا كان المُستَخدِم سيَستَمِرّ.

---

## Variants

### 1. Centered Hero (الافتراضيّ)

```
┌──────────────────────────────────────┐
│                                      │
│        [Tagline pill]                │
│                                      │
│   Big Bold Title (6xl, black)        │
│                                      │
│   Subtitle (lg, regular, jet-700)    │
│                                      │
│       [Primary]  [Secondary]         │
│                                      │
│  ↓ Stats row (optional, small)       │
└──────────────────────────────────────┘
```

**استخدام:** Homepage، About، For Business

```tsx
<section className="bg-gradient-to-b from-orange-50 to-white py-24 lg:py-32">
    <div className="container mx-auto px-4 text-center max-w-4xl">
        <Badge variant="accent" className="mb-6">جَديد — موسم 2026</Badge>
        <h1 className="text-4xl md:text-6xl font-black text-jet-900 leading-tight mb-6">
            مَهارَة جاهِزَة من اليوم الأوّل
        </h1>
        <p className="text-lg md:text-xl text-jet-700 leading-relaxed mb-8">
            لا المُحتوى النَظَريّ — بل مُهارَة قابِلَة للتَطبيق فوراً مع مُمارِسين خَبراء.
        </p>
        <div className="flex flex-col sm:flex-row gap-3 justify-center">
            <Button size="lg" asChild>
                <Link href="/programs">تَصَفَّح البَرامج</Link>
            </Button>
            <Button size="lg" variant="outline" asChild>
                <Link href="/about">من نَحن</Link>
            </Button>
        </div>
    </div>
</section>
```

---

### 2. Split Hero (50/50)

```
┌──────────────────────────────────────┐
│  ┌──────────────┐  ┌──────────────┐  │
│  │              │  │              │  │
│  │   Content    │  │    Image     │  │
│  │   (Right)    │  │   (Left)     │  │
│  │              │  │              │  │
│  └──────────────┘  └──────────────┘  │
└──────────────────────────────────────┘
```

**استخدام:** Feature pages، instructor profiles

```tsx
<section className="bg-white py-20 lg:py-32">
    <div className="container mx-auto px-4">
        <div className="grid gap-12 lg:grid-cols-2 items-center max-w-6xl mx-auto">
            <div>
                <h1 className="text-4xl md:text-5xl font-black text-jet-900 mb-6">
                    تَدريب الشَركات
                </h1>
                <p className="text-lg text-jet-700 mb-8">
                    حَلّ تَدريبيّ مُتَكامِل لِفِرَقك مع تَقارير ROI.
                </p>
                <Button size="lg">احجِز عَرض</Button>
            </div>
            <div className="aspect-square bg-orange-100 rounded-3xl">
                <Image src="/hero-business.webp" alt="..." />
            </div>
        </div>
    </div>
</section>
```

---

### 3. Dark Hero (مَع overlay)

```
┌──────────────────────────────────────┐
│ ████ Image background ████████████   │
│ ████ + dark overlay ██████████████   │
│                                      │
│   White Title                        │
│   White subtitle                     │
│                                      │
│   [Orange CTA]                       │
└──────────────────────────────────────┘
```

**استخدام:** Campaign pages، video hero

```tsx
<section className="relative py-24 lg:py-32 overflow-hidden">
    <Image src="/hero-bg.webp" alt="" fill className="object-cover" />
    <div className="absolute inset-0 bg-gradient-to-b from-jet-950/60 to-jet-950/85" />
    <div className="relative container mx-auto px-4 text-center max-w-4xl">
        <h1 className="text-4xl md:text-6xl font-black text-white mb-6">
            ابدأ رِحلَتَك الآن
        </h1>
        <p className="text-lg text-white/90 mb-8">
            انضم لـ ١٥٠٠+ مُتَدَرِّب يُطَبِّقون ما تَعَلَّموه.
        </p>
        <Button size="lg">سَجِّل مَجّاناً</Button>
    </div>
</section>
```

---

## القَواعِد

✅ **عُنوان واحِد بارِز** (لا "Big text + Big text").
✅ **CTA الأَهَمّ أَوَّلاً**، الثانويّ بَعده.
✅ **Padding سَخيّ** (`py-24+`).
✅ **Center alignment** للـ centered hero — لا تَنحِية يَمين/يَسار عَشوائيّ.
❌ **لا تَنشُر كلّ الـ value props** في Hero — احفِظ لِسكشن لاحِق.
❌ **لا فيديو auto-play** — يُبَطِّئ الـ first paint.
❌ **لا 3+ CTAs** — التَشَتُّت يُقَلِّل التَحويل.
