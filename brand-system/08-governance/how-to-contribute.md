# 08.1 — كَيف تُشارِك في WDS

> WDS مُلك الفَريق، ليس مُلك شَخص. أيّ تَعديل يَتَّبع البَوّابَة.

---

## مَتى تَفتَح PR على WDS؟

| السيناريو | الإجراء |
|---|---|
| إضافَة مُكوّن جَديد | PR في `03-components/` |
| تَعديل لون أو خَطّ | PR في `02-foundations/` + Tailwind config |
| إضافَة pattern | PR في `04-patterns/` |
| تَحديث microcopy | PR في `05-voice-and-tone/` |
| تَحديث accessibility guide | PR في `06-accessibility/` |
| اقتراح breaking change | RFC أَوَّلاً (راجع `versioning.md`) |

---

## الـ Process

### 1. اقتَرِح في GitHub Issue
- العُنوان: `[WDS] إضافَة مُكوّن X`
- الوَصف: المُشكِلَة، الحَلّ المُقترَح، البَدائل المَرفوضَة
- تَوقَّع نِقاش 2-5 أيّام

### 2. ناقِش مع الفَريق
- ادعُ:
    - Brand owner (يَتأكَّد من توافُق الـ voice)
    - Lead designer (يَتأكَّد من توافُق الـ visual system)
    - Lead engineer (يَتأكَّد من قابليّة التَطبيق)
- اتَّفِق على spec قَبل البَدء

### 3. صَمِّم في Figma
- استَخدِم الـ tokens المَوجودَة
- اعرِض كلّ الـ variants + states
- شارِك الـ link في الـ issue

### 4. وَثِّق في WDS
- أنشِئ ملفّ `.md` بالـ structure المُتَّبَع
- اتَّبِع الـ template (راجع `03-components/buttons.md` كَمَرجِع)
- اذكُر: variants, states, when to use, when not, a11y checklist

### 5. نَفِّذ في الكود
- أَضِف الـ component في `frontend/components/ui/`
- أَضِف الـ tokens في `frontend/tailwind.config.ts` إذا لَزِم
- اكتُب وَحَدَة test (Vitest)

### 6. اختَبر
- A11y: axe DevTools
- Visual: Storybook (مُستَقبَلاً)
- Functional: Vitest unit tests

### 7. افتَح PR
- العُنوان: `feat(wds): add X component`
- الـ body:
    - تَصَوّر للمُكوّن (screenshot)
    - رابط Figma
    - رابط الوَثيقَة
    - Checklist (راجع التَالي)

### 8. Code Review
2 reviewers مَطلوب (راجع `review-process.md`).

### 9. Merge + Changelog
- Update `CHANGELOG.md`
- Bump version (راجع `versioning.md`)

---

## PR Checklist

قَبل ما تَطلُب مُراجَعَة:

- [ ] الوَثيقَة كامِلَة (variants, states, when, when-not, a11y)
- [ ] Tokens مُسَجَّلَة في Tailwind config
- [ ] Component مَكتوب في `components/ui/`
- [ ] Unit tests تَمُرّ
- [ ] A11y audit (axe) clean
- [ ] الـ component يَعمَل في RTL + LTR
- [ ] تَجرِبَة على Mobile + Desktop
- [ ] CHANGELOG مُحَدَّث
- [ ] Version bumped (لو major/minor)

---

## مَن يَقرَأ الـ Final Word؟

| النَوع | المَسؤول |
|---|---|
| Visual decisions (لون، خَطّ) | Brand Owner |
| Component API | Lead Engineer |
| Microcopy | Brand Voice Lead |
| Accessibility | A11y Specialist (أو senior eng) |
| Breaking changes | كلّ الـ leads (consensus) |

---

## الـ Templates

### Component Doc Template
```markdown
# 03.N — {ComponentName}

> {One-line purpose}

---

## Variants
[table]

## Sizes
[table]

## States
[list]

## When to Use
[bullets]

## When NOT to Use
[bullets]

## React Example
[code]

## Accessibility
[checklist]
```

---

## كَيف نُعَلِّم الفَريق الجَديد؟

- **Day 1**: قَراءَة `README.md` + `01-strategy/values.md`
- **Day 2**: قَراءَة `02-foundations/` كاملاً
- **Day 3**: قَراءَة `03-components/buttons.md` + `cards.md` كأمثلَة
- **Day 4**: قَراءَة `05-voice-and-tone/microcopy-library.md`
- **Day 5**: مُشارَكَة في PR صَغير على WDS

---

## ماذا لو وَجَدت "خَطَأ" في WDS؟

1. تَأَكَّد أنّه فِعلاً خَطَأ، ليس قَرار مَقصود
2. ابحَث في الـ Git history — قَد يَكون فيه تَبرير
3. افتَح issue (لا تُغَيِّر مُباشَرَة)
4. ناقِش مع الـ owner

> القَرار القَديم قَد يَكون صَحيحاً في وَقته. التَغيير يَتَطَلَّب context جَديد.
