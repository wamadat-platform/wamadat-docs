# 04.7 — Onboarding Flows

> اللحظَة الأهَمّ في عَلاقَة المُستَخدِم. أَوَّل 60 ثانية تُحَدِّد البَقاء أو الرَحيل.

---

## Wamadat Onboarding Strategy

عَلَى عَكس مَنصّات كَثيرَة، ومضات **لا تَطلُب onboarding مُطَوَّل قَبل الدُخول**. فَلسَفَتنا:

> ادخُل، تَصَفَّح، اشتَرِ. وَقت تُريد المَزيد، نَطلُب المَزيد.

### المَراحِل

#### 1. Browse (مَجهول)
- لا يَحتاج حِساب
- يَتَصَفَّح كلّ البَرامج
- يَرى التَفاصيل + المُدرّب
- يُمكِن إضافَة للسلّة (تُحفَظ في localStorage)

#### 2. Sign-up (عند الشراء)
- يُطلَب الحِساب فَقَط عند الـ checkout
- نَموذج **10 حُقول إلزاميّة** (الاسم، الإيميل، الجوّال، الهويّة، تاريخ الميلاد، إلخ — PDPL)
- بَعد الـ submit: cart يَنتَقِل تلقائيّاً

#### 3. First Purchase (الـ Aha moment)
- الشَهادَة الرَقميّة الأولى
- الدُخول للبرنامج فوراً
- إيميل تَأكيد مَع رابط

#### 4. Engagement
- نُذَكِّر بالـ progress عَبر notifications
- نَدعو لِجَلسات مُباشِرَة لو البَرنامج cohort
- بعد الإكمال → CTA لِبَرنامج تالٍ

---

## Welcome Screen (لِلمُسَجَّلين الجُدُد)

عند أَوَّل دُخول للـ dashboard:

```tsx
<div className="bg-gradient-to-br from-orange-50 to-white rounded-3xl p-8 lg:p-12">
    <Sparkles className="size-10 text-orange-500 mb-4" />
    <h1 className="text-3xl font-black text-jet-900 mb-3">
        أهلاً بك، {firstName}! 👋
    </h1>
    <p className="text-lg text-jet-700 mb-6 max-w-2xl">
        ابدأ رِحلَتَك بِخَطوَة واحِدَة: اختَر بَرنامِجك الأَوَّل، أو حَدِّث ملفّك ليَعرِفَك المُدرِّبون.
    </p>
    <div className="flex flex-wrap gap-3">
        <Button asChild>
            <Link href="/programs">اكتَشِف البَرامج</Link>
        </Button>
        <Button variant="outline" asChild>
            <Link href="/dashboard/profile">أكمِل ملفّي</Link>
        </Button>
    </div>
</div>
```

---

## Progress Checklist (للـ Engagement)

```tsx
<Card className="p-6">
    <h3 className="font-bold text-jet-900 mb-4">ابدأ رِحلَتَك في ومضات</h3>
    <ul className="space-y-3">
        <ChecklistItem done>أَنشأت الحِساب</ChecklistItem>
        <ChecklistItem done>اشتَريت أَوَّل بَرنامج</ChecklistItem>
        <ChecklistItem>أَنجَزت أَوَّل درس</ChecklistItem>
        <ChecklistItem>حَصَلت على أَوَّل شَهادَة</ChecklistItem>
    </ul>
</Card>

function ChecklistItem({ done, children }) {
    return (
        <li className="flex items-center gap-3">
            <div className={cn(
                'size-6 rounded-full flex items-center justify-center',
                done ? 'bg-green-500 text-white' : 'bg-jet-100 text-jet-300'
            )}>
                {done && <Check className="size-3.5" />}
            </div>
            <span className={done ? 'text-jet-400 line-through' : 'text-jet-700'}>
                {children}
            </span>
        </li>
    );
}
```

---

## القَواعِد

✅ **لا modal تَدخُّل قَسريّ** على أَوَّل دُخول.
✅ **اعرِض القِيمَة قَبل طَلَب الالتِزام**.
✅ **يَدوَيّ، مُنزَّل بسَخاء** — مُستَخدِم يَختار، نَحن لا نَفرِض.
❌ **لا 10-step tour** بدون استئذان.
❌ **لا تَطلُب بَيانات لَن تَستَخدِمها** "for personalization".
