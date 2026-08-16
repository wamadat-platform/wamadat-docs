# 08.2 — عَمَليّة المُراجَعَة

> المُراجَعَة ليست عَقَبَة — هي جَودَة.

---

## مَن يُراجِع؟

### للـ PRs العاديّة (component جَديد، microcopy)
**2 reviewers مَطلوب:**
- 1 من نَفس الـ discipline (Design للـ design changes، Eng للـ code)
- 1 من discipline مُختَلِف (cross-perspective)

### للـ Breaking Changes
**3 reviewers مَطلوب:**
- Brand Owner
- Lead Engineer
- Product Owner

---

## ماذا يُراجَع؟

### Brand Voice
- ✅ النَصّ يَتَّبع `05-voice-and-tone/voice-principles.md`
- ✅ النَبرَة مُناسِبَة للسياق
- ✅ الكَلِمات من المَكتَبَة (`microcopy-library.md`)

### Visual
- ✅ يَستَخدِم tokens (ألوان، خُطوط، مَسافات) من النِظام
- ✅ يَتَّبع spacing rules (8pt grid)
- ✅ Shadows + borders مُتَّسِقَة

### Code
- ✅ TypeScript clean
- ✅ Tailwind classes منَظَّمَة
- ✅ لا inline styles
- ✅ Component يَستَخدِم `cn()` utility

### Accessibility
- ✅ Semantic HTML
- ✅ ARIA labels عند الحاجَة
- ✅ Keyboard navigation works
- ✅ Color contrast AA passing

### Documentation
- ✅ ملفّ `.md` كامِل (variants, when to use, etc.)
- ✅ React examples تَعمَل
- ✅ يَذكُر anti-patterns

---

## SLAs

| نَوع الـ PR | الـ First Review | الـ Merge |
|---|---|---|
| Bug fix | < 1 يوم عَمَل | < 2 يوم |
| New component | < 2 يوم | < 5 يوم |
| Breaking change | < 1 أُسبوع | < 2 أُسابيع |

---

## Review Comments

### النَوع
| Prefix | المعنى |
|---|---|
| `nit:` | اقتِراح صَغير، اختياريّ |
| `q:` | سؤال للتَوضيح |
| `suggestion:` | اقتِراح، الكاتِب يُقَرِّر |
| `blocking:` | يَجب حَلّه قَبل الـ merge |

### النَبرَة
- ✅ "ما رَأيك لو استَخدَمنا X هنا؟"
- ❌ "هذا خَطَأ، غَيِّره"

> المُراجَعَة عَن الكود، ليس عَن المُؤَلِّف.

---

## بَعد الـ Merge

1. Update CHANGELOG.md
2. Bump version (لو needed)
3. Notify team في channel #design-system
4. Update Figma library (لو visual change)
5. Migration notes للـ breaking changes
