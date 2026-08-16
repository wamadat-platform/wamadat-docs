# 04.4 — Error States

> الخَطَأ مُؤذٍ نَفسيّاً. خَفِّفه بـ: تَفسير، حَلّ، وَطُمَأنينَة.

---

## Anatomy

```
        [Icon (AlertCircle, red-500)]
        
        [Title — ماذا حَدَث]
        
        [Description — لماذا، أو ماذا تَفعَل]
        
        [Primary CTA — حاوِل ثانية / تَواصَل]
        [Secondary — العودَة]
```

---

## أنواع الأخطاء

### 404 — الصَفحَة غَير مَوجودَة
```tsx
<div className="min-h-[60vh] flex items-center justify-center p-6">
    <div className="text-center max-w-md">
        <div className="text-6xl mb-4">🔍</div>
        <h1 className="text-2xl font-bold text-jet-900 mb-2">
            هذه الصَفحَة غَير مَوجودَة
        </h1>
        <p className="text-jet-500 mb-6">
            قد يَكون الرابط قَديماً أو الصَفحَة تَمّ نَقلها.
        </p>
        <div className="flex gap-3 justify-center">
            <Button asChild>
                <Link href="/">الصَفحَة الرَئيسيّة</Link>
            </Button>
            <Button variant="outline" asChild>
                <Link href="/programs">تَصَفَّح البَرامج</Link>
            </Button>
        </div>
    </div>
</div>
```

### 500 — خَطَأ في الخادِم
```
Title: خَطَأ غَير مُتَوَقَّع
Description: حَدَث خَلَل عِندنا — نَعمَل على إصلاحه. حاوِل بَعد دَقيقَة أو تَواصَل مَعَنا.
CTAs: حاوِل مَرَّة أُخرى / تَواصَل مع الدَعم
```

### Network Error
```
Title: تَعَذَّر الاتِّصال
Description: تَحَقَّق من اتِّصالك بالإنترنت ثم حاوِل مَرَّة أُخرى.
CTAs: إعادة المُحاولَة
```

### Permission Denied (403)
```
Title: لا تَملِك صَلاحيّة الوصول
Description: هذه الصَفحَة مَحجوزَة لِمُستَخدِمي خَطَّة أَعلى.
CTAs: عرض الباقات / العودَة
```

### Payment Failed
```
Title: لم يَكتَمِل الدَفع
Description: رُفض الدَفع من البَنك أو أُلغي يَدويّاً. حاوِل بِبَطاقَة أُخرى.
CTAs: المُحاولَة مَرَّة أُخرى / تَواصَل
```

### Form Validation Error
```
Inline على الحَقل:
- نَصّ صَغير أَحمَر تحت الـ field
- icon AlertCircle (size-3.5)
- نَبرَة شارِحَة، لا لَومِيَّة
```

---

## النَبرَة في Errors

### ✅ نَفعَل
- **اشرَح ما حَدَث** بكَلِمات إنسانيّة
- **اعطِ خَطوَة تالِيَة** واضِحَة
- **خُذ المَسؤوليّة** ("حَدَث خَلَل عِندنا" لا "حَدَث خَطَأ عَندَك")
- **اطمَئِن المُستَخدِم** بأنّ بَياناته آمِنَة

### ❌ لا نَفعَل
- ❌ "Error 422 — Validation Failed" — مَن يَفهَم؟
- ❌ "حَدَث خَطَأ" بدون تَفصيل
- ❌ نَبرَة سَلبيّة ("لا تَستَطيع" → استَبدِل بـ "دَعنا نُساعِدك")
- ❌ Stack traces في الـ UI

---

## النَموذَج

```tsx
// أَيّ Error component يَتَّبع هذا الـ pattern:
<div className="bg-red-50 border border-red-200 rounded-2xl p-6">
    <div className="flex items-start gap-3">
        <AlertCircle className="size-5 text-red-600 flex-shrink-0 mt-0.5" />
        <div className="flex-1">
            <h3 className="font-bold text-jet-900 mb-1">{title}</h3>
            <p className="text-sm text-jet-700 mb-4">{description}</p>
            <div className="flex gap-2">
                <Button size="sm" onClick={retry}>حاوِل مَرَّة أُخرى</Button>
                <Button size="sm" variant="ghost" asChild>
                    <Link href="/contact">تَواصَل</Link>
                </Button>
            </div>
        </div>
    </div>
</div>
```
