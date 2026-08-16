# 08.4 — سياسَة الاستِبعاد (Deprecation)

> نَموت بَطيئاً، لا فَجأَة. الاستِبعاد عَمَليّة، ليس قَرار يَوم واحِد.

---

## مَتى نَستَبعِد؟

| الحالَة | الإجراء |
|---|---|
| Component لم يُستَخدَم 6 شُهور | Mark deprecated |
| Token استُبدِل بأَفضَل | Soft deprecation |
| Pattern ثَبَت أنّه anti-pattern | Hard deprecation |
| API security flaw | Immediate removal (سَريع) |

---

## مَراحِل الاستِبعاد

### المَرحَلَة 1: التَحذير (Warn) — 3 شُهور
- أَضِف JSDoc deprecation:
```tsx
/**
 * @deprecated استَخدِم `<Button variant="primary">` بَدَلاً. سيُحذَف في v2.0.
 */
export const PrimaryButton = ...
```

- Console warning في dev mode:
```tsx
if (process.env.NODE_ENV === 'development') {
    console.warn('[WDS] <PrimaryButton> is deprecated. Use <Button variant="primary">.');
}
```

- Update docs مع banner "Deprecated"

### المَرحَلَة 2: التَوقُّف عن الاستِخدام (Discourage) — 3 شُهور
- ESLint rule يَحَذِّر عند الاستِخدام
- لا PRs جَديدَة تَستَخدِم العُنصُر المُستَبعَد
- Migration guide مَوجود ومُحَدَّث

### المَرحَلَة 3: الإزالَة (Remove)
- في الـ MAJOR version التالي
- إعلام الفَريق قَبل أُسبوع
- PR إزالَة يَحتوي:
    - الـ deletion
    - codemod للـ migration
    - Updated CHANGELOG

---

## أنواع الاستِبعاد

### Soft Deprecation
- العُنصُر يَعمَل، لكن يُحَذَّر
- يَستَخدِمه الكود القَديم بأمان
- يَحَذَّر للجَديد

### Hard Deprecation
- العُنصُر سَيُحذَف قَريباً
- لا PRs جَديدَة تُقبَل تَستَخدِمه
- يَجب migration

### Immediate Removal
- security issue أو bug critical
- يَحدُث بدون warning period
- يَحدُث في PATCH version (1.0.X)

---

## مَكتَبَة الـ Deprecated

| العُنصُر | السَبَب | البَديل | تاريخ الإزالَة |
|---|---|---|---|
| (لا شَيء بَعد — WDS v1.0 جَديد) | — | — | — |

---

## Migration Codemods

لِكلّ deprecation، اكتُب codemod (لو ممكِن):

```bash
# مَثَل: استَبدِل <PrimaryButton> بـ <Button variant="primary">
npx jscodeshift -t wds-migrate-primary-button.js src/
```

> يَجعَل الـ migration آليّ، ليس يَدويّ.

---

## التَواصُل

عند بَدء استِبعاد:
1. Post في #design-system: "FYI: X سيُستَبعَد، migration guide هنا"
2. Email للـ leads
3. Update CHANGELOG.md
4. Update الـ component doc مَع warning banner

عَند الإزالَة:
1. Final announcement أُسبوع قَبل
2. Confirm لا code يَستَخدِمه (`git grep`)
3. PR الإزالَة
4. Tag major version

---

## مَبادِئ

✅ **لا تَستَبعِد بدون بَديل أَفضَل**
✅ **3 شُهور warning minimum** قَبل أيّ إزالَة
✅ **codemods حَيث يُمكِن**
✅ **migration guides لكلّ deprecation**
❌ **لا تَستَبعِد لِأَسباب جَماليّة فَقَط** — يَجب تَوفير قِيمَة حَقيقيّة
