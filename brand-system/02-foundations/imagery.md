# 02.6 — الصور (Imagery)

> الصور تَحكي قِصّة. صورنا أصيلَة، سُعوديّة، احتِرافيّة دافِئَة.

---

## الفِئات (Categories)

### 1. Hero Images (للأقسام البَطَلَة)
**الأبعاد:** 1920×1080 (16:9) أو 1440×800
**الأسلوب:** Lifestyle، سياق سُعوديّ حَقيقيّ
**النَبرة:** cinematic، احتِرافيّة، دافِئَة

**أمثلة سياقات مَناسِبَة:**
- صورَة مَكتَب حَديث في الرياض
- مُتدَرِّب يَستَخدِم لابتوب في كافيه
- مَجموعَة شَباب في ورشَة عَمَل
- مُدرّب يُحاضِر أمام قاعَة

---

### 2. Course Covers (لِكروت البرامج)
**الأبعاد:** 1280×720 (16:9)
**الأسلوب:** نَظيف، مَوضوعيّ
**المُعالَجَة:** Subtle gradient overlay (إذا الصورَة فاتِحَة جدّاً)

**عَوامِل النَجاح:**
- صورَة مُرتَبِطَة بمَجال البَرنامج (مايكروفون لِبَرنامج التَعليق الصَوتيّ)
- لا نَصّ في الصورَة (النَصّ يُضاف بـ CSS)
- جَودَة Retina (2x)

**الـ Fallback:**
لو لم تُرفَع صورَة، النِظام يُولِّد SVG cover مَوصوف في `02-foundations/colors.md` — gradient + الحَرف الأوّل من العُنوان.

---

### 3. Instructor Portraits
**الأبعاد:** 800×800 (square)
**الأسلوب:** Professional headshot
**الخَلفيّة:** orange (`#FAAF3C`) bokeh أو neutral cream

**القَواعِد:**
- وَجه واضِح، عَيْنان للكاميرا (أو 3/4)
- ابتِسامَة طَبيعيّة، لا مَزعومَة
- مَلابس مُهَنيّة (شَماغ + ثوب أو ثَوب أبيض، بِزَّة، blazer)
- إضاءَة طَبيعيّة، لا flash قاسٍ
- لا فِلتَر مُبالَغ (Instagram filter ممنوع)

---

### 4. Empty State Illustrations
**الأبعاد:** 400×400
**الأسلوب:** Line drawings + orange accent
**النَبرة:** صَديقَة، غَير عِيادِيّة

**أمثلَة:**
- "لا برامج" → رَسم كِتاب مَفتوح بَخَطّ
- "لا إشعارات" → رَسم جَرَس صَغير
- "لا نَتائج بَحث" → رَسم عَدَسَة مُكَبِّرَة

> النِظام الحاليّ يَستَخدِم Lucide icons في empty states (مَوصوف في `04-patterns/empty-states.md`). الـ illustrations المُخَصَّصَة مَحجوزَة لِـ v1.2.

---

## أسلوب التَصوير (Photography Style)

### ✅ نَفعَل
- **سياق سُعوديّ حَقيقيّ** — مَلامِح، مَلابِس، أَماكِن مَعروفَة (مَكاتِب الرياض، شَوارِع جدّة)
- **تَنَوّع تَمثيليّ** — رِجال، نِساء، أَعمار مُختَلِفَة، مَلابِس مُحَلّيّة وعالميّة
- **إضاءَة طَبيعيّة** — ضوء النَهار مُفَضَّل
- **مَوقف فِعليّ** — شَخص يَعمَل، يُحاضِر، يَدرُس — لا "يَنظُر للكاميرا بابتِسامَة"
- **دِفء** — ألوان دافِئَة (لا cold blues)

### ❌ لا نَفعَل
- ❌ صور **Stock أَجنَبيّة كليشيه** (الفِريق الأمريكيّ المُبتَسِم في مَكتَب)
- ❌ صور **مَوضوعَة بشَكل مُتكَلَّف** (شَخص يَنظُر لـ "اللاشَيء" بثَوب أبيض في صَحراء)
- ❌ صور **مَلوَّنَة فوق المَعقول** (HDR مُبالَغ)
- ❌ صور **خاوِيَة من النَاس** للقَطاعات التي تَخصّ النَاس (لا تَستَخدِم صورَة لابتوب فاضي لِبَرنامج "التَواصُل")
- ❌ صور **AI-generated** (لا Midjourney، لا DALL-E لِشَخصيّات)

---

## التَقنيّات

### Formats
- **WebP** (الأَوَّل) — أَخَفّ، أَسرَع
- **JPEG** كـ fallback (للمُتَصَفِّحات القَديمَة)
- **PNG** فقط لِما يَحتاج transparency
- **SVG** للأيقونات والـ illustrations

### Resolution
- **2x مُلزِم** لكلّ صورَة (Retina-ready)
- **3x اختياريّ** للـ Hero images الكَبيرَة

### Optimization
- **Bunny CDN** (مُحَدَّد في ADRs) للتَوصيل
- **Lazy loading** على كلّ صورَة غَير مَرئيّة في viewport البِدائيّ
- **Blur placeholder** أو dominant color placeholder

```tsx
<Image
    src="/programs/voice-over.webp"
    alt="بَرنامج التَعليق الصَوتيّ"
    width={1280}
    height={720}
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,..."
    loading="lazy"
/>
```

---

## Alt Text (مَطلوب دائماً)

كلّ صورَة **يَجب** أن تَملك `alt` يَشرَح المحتوى:

### ✅ صَحيح
```html
<img src="/instructor.webp" alt="الأستاذ وائل الحبال يَشرَح تِقنيات التَعليق الصَوتيّ في الاستوديو" />
```

### ❌ خَطَأ
```html
<img src="/instructor.webp" alt="صورَة" />
<img src="/instructor.webp" alt="" />  /* فارِغ، يُسبِّب مَشاكِل لِقارئ الشاشَة */
<img src="/instructor.webp" />  /* مَفقود نِهائيّاً */
```

### استَثناء: الصور الزَخرَفيّة فَقَط
لو الصورَة زَخرَفيّة 100% (لا تُضيف معنى)، استَخدِم `alt=""` + `role="presentation"`:

```html
<img src="/decorative-pattern.svg" alt="" role="presentation" />
```

---

## التَطبيق في المنصّة

### Program Cover
```tsx
<div className="relative aspect-video rounded-2xl overflow-hidden bg-jet-100">
    {program.cover_image_url ? (
        <Image
            src={program.cover_image_url}
            alt={program.title_ar}
            width={1280}
            height={720}
            className="w-full h-full object-cover"
        />
    ) : (
        <SvgFallback slug={program.slug} title={program.title_ar} />
    )}
</div>
```

### Instructor Portrait
```tsx
<div className="size-32 rounded-full overflow-hidden bg-orange-100">
    {instructor.avatar_url ? (
        <Image
            src={instructor.avatar_url}
            alt={`الأستاذ ${instructor.full_name_ar}`}
            width={400}
            height={400}
            className="object-cover w-full h-full"
        />
    ) : (
        <InitialAvatar name={instructor.full_name_ar} />
    )}
</div>
```

---

## مَكتَبَة المَصادِر (Sources)

عند الحاجَة لِصور Stock، استَخدِم:
1. **Unsplash** — مَجّاني، جَودَة جَيِّدَة (ابحَث "Saudi Arabia office" / "Arabic professional")
2. **Pexels** — مَجّاني، تَنَوّع أَوسَع
3. **Saudi Tourism Authority** للصور الرَسميّة عن المَلامِح
4. **شُركاء مَحَلّيّون** للصور المُخَصَّصَة (الأولويّة دائماً)

> **حَذار:** Stock photos، حَتّى من الـ MENA، قد تَكون مَستَخدَمَة في إعلانات مُنافِسة. حَلّ: الـ DALL-E ممنوع، لكن شَراء صور مُخَصَّصَة من مُصَوِّرين سُعوديّين مُحَلِّيّين خَيار مُمتاز.

---

## المُعالَجَة (Treatment)

### Hero Overlay
```css
/* لِنَضمَن قَراءَة النَصّ فوق صورَة */
background-image:
    linear-gradient(to bottom, rgba(26, 26, 26, 0.6), rgba(26, 26, 26, 0.85)),
    url('/hero.webp');
```

```tsx
<section className="relative">
    <Image src="/hero.webp" alt="" fill className="object-cover" />
    <div className="absolute inset-0 bg-gradient-to-b from-jet-950/60 to-jet-950/85" />
    <div className="relative z-10 container py-32">
        <h1 className="text-white">...</h1>
    </div>
</section>
```

### Color Treatment for Brand Consistency
```css
/* لتَوحيد صور مُخَتَلِفَة الألوان */
img.brand-treated {
    filter: contrast(1.05) saturate(0.95);
}
```

> استَخدِم فِلتَر خَفيف فقط. لا "Instagram look".

---

## الـ Anti-patterns

❌ **خَلط أساليب تَصوير** في نَفس الصَفحَة (صورَة طَبيعيّة + صورَة AI + رَسم vector)
❌ **حَجم صورَة أكبَر من اللازِم** (10MB صورَة لِـ thumbnail)
❌ **صور مَع نَصّ مَكتوب فيها** (يَصعُب تَرجَمَتها، حَلّ: استَخدِم CSS overlay)
❌ **صور بحَقوق نَشر غَير واضِحَة** (Always credit + license)
