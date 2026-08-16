# 08.3 — الإصدارات (Versioning)

> Semantic Versioning. كلّ رَقَم يَحمِل معنى.

---

## التَنسيق

```
MAJOR.MINOR.PATCH
1.0.0
```

### MAJOR (X.0.0)
**Breaking changes** — تَتَطَلَّب تَغيير في الكود المُستَخدِم.

أمثِلَة:
- إزالَة prop من component
- تَغيير اسم Tailwind token
- تَغيير سُلوك مَعروف للمُكوّن
- حَذف component مَهجور

### MINOR (1.X.0)
**Backward-compatible additions** — إضافات لا تَكسِر شَيئاً.

أمثِلَة:
- إضافَة variant جَديد لمُكوّن
- إضافَة token جَديد
- إضافَة component جَديد
- تَوسيع الـ API بـ optional props

### PATCH (1.0.X)
**Backward-compatible fixes** — إصلاحات صَغيرَة.

أمثِلَة:
- تَصحيح typo في الوَثائق
- إصلاح bug في component
- تَحديث Microcopy لتَحسين الوُضوح

---

## الـ Bumping

```bash
# في فَريقَنا، manual في CHANGELOG.md
# لاحِقاً: nuxt automation عَبر changesets أو semantic-release
```

---

## Pre-release Tags

للـ experimental work:
- `1.1.0-alpha.1` — للتَجريب الداخليّ
- `1.1.0-beta.1` — للتَجريب الخارجيّ
- `1.1.0-rc.1` — مُرَشَّح للإصدار

---

## كَيف نَعرِف الإصدار الحاليّ؟

```bash
# في docs/brand-system/CHANGELOG.md
## [1.0.0] — 2026-05-12
```

---

## Communication

عند إصدار جَديد:
1. Update `CHANGELOG.md` بتَفاصيل
2. Tag في Git: `git tag wds-v1.1.0`
3. Notify في #design-system channel
4. اذا major: blog post + migration guide

---

## Migration Guides

للـ MAJOR releases، اكتُب migration guide:

```markdown
# Migrating from WDS v1 to v2

## Breaking Changes

### Button: `variant="default"` removed
**Before:**
\`\`\`tsx
<Button variant="default">Click</Button>
\`\`\`

**After:**
\`\`\`tsx
<Button variant="primary">Click</Button>
\`\`\`

Find + replace:
\`\`\`bash
sed -i 's/variant="default"/variant="primary"/g' src/**/*.tsx
\`\`\`
```
