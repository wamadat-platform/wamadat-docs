# 05.8 — عَناوين الـ SEO

> العُنوان في Google = أَوَّل لُقاء. اجعَله يَدُلّ على القيمَة.

---

## الـ Anatomy

```
{Specific Title} | {Brand}
```

- **Specific Title**: 30-60 حَرف (يَظهَر كامِلاً في Google)
- **Brand**: "ومضات أكاديمي" أو "Wamadat Academy"

---

## أنماط

### Homepage
```
ومضات أكاديمي — مَهارَة جاهِزَة من اليوم الأَوَّل
```

### Programs Listing
```
كلّ البَرامج | ومضات أكاديمي
```

### Single Program
```
{Program Title} | ومضات أكاديمي
```
مَثَل: `مُهارَات التَعليق الصَوتيّ | ومضات أكاديمي`

### Instructors
```
المُدرِّبون المُعتَمَدون | ومضات أكاديمي
```

### Instructor Profile
```
{Instructor Name} - مُمارِس {Specialty} | ومضات
```

### Categories
```
بَرامج {Category} | ومضات أكاديمي
```

### About
```
من نَحن — قِصَّة ومضات أكاديمي
```

### Pricing
```
الباقات والأسعار | ومضات أكاديمي
```

### Contact
```
تَواصَل مَعَنا | ومضات أكاديمي
```

### Blog Post (Future)
```
{Article Title} | بلوغ ومضات
```

---

## Meta Descriptions

### الـ Anatomy
- 130-160 حَرف
- يَتَضَمَّن CTA خَفيف ("اكتَشِف"، "ابدأ")
- يَحوي keywords أَساسيّة

### أمثلة

**Homepage**:
> ومضات أكاديمي — مَنصّة التَدريب التَطبيقيّ في الـ MENA. 30+ بَرنامج، 15 مُدرِّب مُمارِس، شَهادات مُعتَمَدَة. ابدأ مَهارَتك اليوم.

**Program (Voice-over)**:
> تَعَلَّم التَعليق الصَوتيّ من الصِفر للاحتِراف. 40 ساعَة تَدريب تَطبيقيّ مَع وائل الحبال + شَهادَة مُعتَمَدَة.

**About**:
> 1500+ مُتدَرِّب، 50+ مُدرِّب، شَراكات مع وَزارَة الإعلام وشَركات كُبرى. اكتَشِف كَيف نَصنَع كَفاءات قابِلَة للتَطبيق.

---

## القَواعِد

✅ **Brand دائماً في آخِر الـ title** (بَعد |).
✅ **Specific قَبل Generic** — "التَعليق الصَوتيّ" قَبل "البَرامج".
✅ **Keywords طَبيعيّة** — لا حَشو ("بَرامج تَدريب تَدريبيّ تَدريب").
✅ **Meta description كـ promise** — ماذا سَيَجِد في الصَفحَة.
❌ **لا CAPS مُبالَغَة** — العَربيّة لا تَملِك caps أصلاً، لكن لا تَفعَل "!!" بَدلاً.
❌ **لا تَكرار للـ brand** في كلّ كَلِمَة.
❌ **لا keywords stuffing**.

---

## Implementation

```tsx
// app/[locale]/programs/[slug]/page.tsx
export async function generateMetadata({ params }: { params: Promise<{ slug: string }> }) {
    const { slug } = await params;
    const program = await fetchProgramBySlug(slug);
    if (!program) return { title: 'البَرنامج غَير مَوجود | ومضات' };

    return {
        title: `${program.title_ar} | ومضات أكاديمي`,
        description: program.subtitle_ar?.slice(0, 160),
        openGraph: {
            title: program.title_ar,
            description: program.subtitle_ar,
            images: [program.cover_image_url],
        },
    };
}
```
