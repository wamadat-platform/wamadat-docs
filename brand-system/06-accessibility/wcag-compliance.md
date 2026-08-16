# 06.1 — WCAG 2.1 AA Compliance

> الوُصوليّة حَقّ، ليس إضافَة. كلّ صَفحَة، كلّ مُكوّن، يَجب أن يَخدُم كلّ المُستَخدِمين.

---

## المَعايير الأَربَع (POUR)

### 1. Perceivable (يُدرَك)
المُحتوى يَجب أن يَكون مُدرَكاً للحَواسّ.

**Checklist:**
- ✅ Color contrast 4.5:1 للنَصّ العاديّ، 3:1 للنَصّ الكَبير (18px+ bold)
- ✅ Alt text لكلّ صورَة (`<img alt="...">`)
- ✅ Captions للفيديوهات
- ✅ Transcripts للأصوات
- ✅ لا تَعتَمِد على اللون وَحده للمعنى (icon + color + text)
- ✅ Text resizable حتّى 200% بدون فُقدان مَعلومات

### 2. Operable (قابِل للتَشغيل)
كلّ شَيء قابِل للتَنفيذ بالكِيبورد.

**Checklist:**
- ✅ كلّ الـ interactive elements مَوصول لها بـ Tab
- ✅ ترتيب الـ Tab مَنطِقيّ (يَتبَع الـ visual order)
- ✅ Focus indicators واضِحَة (`ring-2 ring-orange-500`)
- ✅ Skip links متاحة (`<a href="#main">Skip to content</a>`)
- ✅ لا keyboard traps
- ✅ Auto-play موقوف افتراضيّاً
- ✅ Timeouts قابِلَة للتَمديد

### 3. Understandable (مَفهوم)
لُغَة الصَفحَة + الإرشادات واضِحَة.

**Checklist:**
- ✅ `<html lang="ar" dir="rtl">` على كلّ صَفحَة عَربيّة
- ✅ Labels واضِحَة لكلّ form field
- ✅ Error messages مَفهومَة وإنسانيّة
- ✅ Navigation مُتَّسِق بَين الصَفحات
- ✅ Behavior مُتَوَقَّع (لا surprises)

### 4. Robust (مَتين)
يَعمَل مع كلّ التِقنيّات المُساعِدَة.

**Checklist:**
- ✅ HTML semantic صَحيح (`<button>` ليس `<div onClick>`)
- ✅ ARIA attributes عند الحاجَة فقط
- ✅ Status messages عَبر `aria-live`
- ✅ يَعمَل مع NVDA, JAWS, VoiceOver

---

## الأَدوات

### Automated
- **axe DevTools** (Chrome extension)
- **Lighthouse** (Chrome built-in)
- **WAVE** (https://wave.webaim.org/)

### Manual
- اختَبر بـ Tab keyboard فقط (بدون mouse)
- اختَبر مع screen reader (NVDA مَجّاناً)
- اختَبر بِـ 200% zoom

---

## Common Mistakes

| الخَطَأ | الإصلاح |
|---|---|
| `<div onClick={...}>` | `<button onClick={...}>` |
| `<img>` بدون alt | أَضِف `alt="وَصف"` أو `alt=""` للزَخرَفَة |
| `<a href="#">` placeholder | استَخدِم `<button>` بَدَلاً |
| Color-only states (e.g., "العُنصُر الأَحمَر error") | أَضِف icon + text |
| Focus بدون رؤيَة | أَضِف `focus-visible:ring-2 ring-orange-500` |
| Modal بدون trap | استَخدِم Radix Dialog أو focus-trap-react |
| `placeholder` كَـ label | استَخدِم `<label>` صَريح |

---

## المعايير الخاصّة بـ ومضات

1. **RTL أَوَّلاً** — كلّ مُكوّن مُختَبَر في RTL
2. **Arabic screen reader compatible** — اختَبر مع NVDA + Arabic
3. **Touch targets ≥ 44×44px** — للجَوّال
4. **Reduced motion respected** — `prefers-reduced-motion`
5. **High contrast mode** — يَعمَل في Windows High Contrast

---

## Compliance Tests Run Quarterly

Q1: Lighthouse audit على homepage + programs + dashboard
Q2: Manual keyboard navigation test
Q3: Screen reader test (NVDA + Arabic)
Q4: External a11y consultant audit
