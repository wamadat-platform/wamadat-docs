# 04.6 — Success States

> النَجاح يَستَحقّ احتِفال. لكن قاس — لا تُبالِغ.

---

## Levels

| Level | متى | المَظهَر |
|---|---|---|
| `Subtle` | حِفظ تِلقائيّ، إعادَة | Toast بسيط، 2 ثانية |
| `Standard` | إرسال نَموذج، إكمال مَهَمَّة | Inline alert + رَسالَة |
| `Celebratory` | شَهادَة، أوّل شِراء، إكمال بَرنامج | Modal/page مَع رَمز نَجاح + CTA |

---

## Pattern: Subtle (Toast)
```tsx
toast.success('تَمّ الحِفظ');
```

---

## Pattern: Standard
```tsx
<div className="rounded-2xl bg-green-50 border border-green-200 p-6">
    <div className="flex items-start gap-3">
        <div className="size-10 rounded-full bg-green-500 text-white flex items-center justify-center flex-shrink-0">
            <Check className="size-5" />
        </div>
        <div>
            <h3 className="font-bold text-jet-900 mb-1">تَمّ الإرسال بنَجاح</h3>
            <p className="text-sm text-jet-700">سَنَتَواصَل مَعك خِلال يوم عَمَل.</p>
        </div>
    </div>
</div>
```

---

## Pattern: Celebratory
```tsx
<div className="text-center max-w-md mx-auto py-12">
    <div className="size-20 mx-auto rounded-full bg-gradient-to-br from-orange-400 to-orange-600 flex items-center justify-center mb-6">
        <Award className="size-10 text-white" />
    </div>
    <h1 className="text-3xl font-black text-jet-900 mb-3">
        مَبروك! 🎉
    </h1>
    <p className="text-lg text-jet-700 mb-8">
        أَنجَزت بَرنامج التَعليق الصَوتيّ بنَجاح. شَهادَتك جاهِزَة.
    </p>
    <div className="flex flex-col sm:flex-row gap-3 justify-center">
        <Button asChild>
            <Link href="/dashboard/certificates">تَنزيل الشَهادَة</Link>
        </Button>
        <Button variant="outline" asChild>
            <Link href="/programs">بَرنامج آخَر</Link>
        </Button>
    </div>
</div>
```

---

## النَبرَة

✅ **فَخور بإنجاز المُستَخدِم** — "أَنجَزت"، "حَقَّقت"، "وَصَلت"
✅ **اعرِض الخَطوَة التالِيَة** — لا تَترُكه يَنظُر للنَجاح
✅ **مَتى يَستَحقّ celebratory؟** — أوّل شَهادَة، أوّل شراء، إكمال مَسار طَويل
❌ **لا "Congratulations!!!" مَع 3 نَجمات** — مُبَالَغ
❌ **لا تَستَخدِم celebratory في كلّ نَجاح صَغير** — يَفقِد قِيمَتَه
