# 07.3 — استراتيجيّة التَرجَمَة

> ومضات Arabic-first. الإنجليزيّة ثَانويّة. الانتِشار الإقليميّ مَخطَّط.

---

## اللُغات المَدعومَة

### v1.0 (الآن)
- **العَربيّة (`ar`)** — الافتراضيّ ⭐
- **الإنجليزيّة (`en`)** — للزُوّار الأَجانب + B2B partners

### v1.x (مَخطَّط — راجع `01-strategy/narrative.md`)
- v1.5 (2028): الإمارات، الكُويت، مِصر، الأُردُن — نَفس العَربيّة، تَكييف خَفيف
- v2.0 (2029): التُركيّة، الفَرَنسيّة (شَمال أفريقيا)
- v3.0 (2030+): الأُرديّة، الفارسيّة

---

## الـ Locale System

نَستَخدِم `next-intl`:

```ts
// next-intl.config.ts
export default {
    locales: ['ar', 'en'],
    defaultLocale: 'ar',
    localePrefix: 'as-needed', // /ar/programs = /programs، /en/programs explicit
};
```

### Routing
- `wamadat.academy/programs` → العَربيّة (default)
- `wamadat.academy/en/programs` → الإنجليزيّة
- `wamadat.academy/ar/programs` → العَربيّة explicit

---

## Translation Files

```
frontend/messages/
├── ar.json  ← المَصدَر، 100% complete
└── en.json  ← المُتَرجَم
```

### Structure
```json
{
    "nav": {
        "home": "الرئيسيّة",
        "programs": "البرامج",
        "instructors": "المدرّبون"
    },
    "buttons": {
        "start": "ابدأ",
        "continue": "تابع",
        "cancel": "إلغاء"
    }
}
```

### Naming Convention
- **Namespaces** متَوافِقَة مع section/component
- **Keys snake_case**
- **No deep nesting** (عُمق 2 maximum)

---

## Translation Workflow

### 1. كَتابَة بالعَربيّة
كلّ نَصّ يَبدأ بالعَربيّة في `messages/ar.json`.

### 2. مُراجَعَة من Brand Voice
نَقارِن النَصّ مع `05-voice-and-tone/microcopy-library.md`.

### 3. تَرجَمَة للإنجليزيّة
- مُتَرجِم بَشَريّ (ليس Google Translate)
- يَعرِف ومضات brand voice
- يَفهَم السياق الإقليميّ

### 4. مُراجَعَة QA
- اختَبر في الـ UI الفِعليّ (text overflow، RTL/LTR شَكل)
- اقرَأ مع native English speaker

---

## القَواعِد

### ✅ نَفعَل
- **اكتُب نَصّاً قابِلاً للتَرجَمَة** — لا تَفترِض ترتيب الكَلِمات
- **استَخدِم placeholders** للقِيَم الديناميكيّة: `"hello": "أهلاً {name}"`
- **اختَبر تَوَسُّع النَصّ** — الإنجليزيّة عادَة أطوَل من العَربيّة بـ 30%
- **استَخدِم plural forms** عند الحاجَة: `"{count, plural, =1 {درس واحد} other {# درس}}"`

### ❌ لا نَفعَل
- **لا تَضَع نَصّاً مُباشِراً في JSX** — كلّ نَصّ من i18n
- **لا تَستَخدِم Google Translate للنَصّ النِهائيّ** — للتَجريب فَقَط
- **لا تَفترِض RTL = mirror** — بَعض العَناصِر لا تَنعَكِس
- **لا تَنسَ صياغَة الأسعار** — `4,195 ر.س` (عَربيّة) vs `SAR 4,195` (English)

---

## Date & Number Formatting

استَخدِم `Intl` API دائماً:

```tsx
// التَواريخ
new Date().toLocaleDateString('ar-SA', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
});
// → "١٢ مايو ٢٠٢٦"

new Date().toLocaleDateString('en-US', { ... });
// → "May 12, 2026"

// الأرقام
(4195).toLocaleString('en-US');
// → "4,195"

(4195).toLocaleString('ar-SA');
// → "٤٬١٩٥"

// الـ UI نَستَخدِم en-US للأرقام (مَنطِق الأرقام اللاتينيّة)
```

---

## Currency

```tsx
new Intl.NumberFormat('ar-SA', {
    style: 'currency',
    currency: 'SAR',
}).format(4195);
// → "٤٬١٩٥٫٠٠ ر.س"
```

في الـ UI نَستَخدِم نَموذَجاً مُبَسَّطاً:
```tsx
<span className="tabular-nums">{price.toLocaleString('en-US')} ر.س</span>
```

---

## Pluralization

```tsx
// next-intl supports ICU MessageFormat
{
    "lessons_count": "{count, plural, =0 {لا دُروس} =1 {درس واحد} =2 {درسان} other {# دُروس}}"
}

useTranslations('lessons_count', { count: 5 });
// → "5 دُروس"
```

---

## Quality Checklist

قَبل deploy:
- [ ] كلّ keys في `ar.json` تَوجَد في `en.json` (و العَكس)
- [ ] لا hardcoded strings في `.tsx` files
- [ ] التَواريخ + الأرقام مُنَسَّقَة بالـ locale
- [ ] لا overflow في الـ UI بَعد تَرجَمَة (English text أَطوَل)
- [ ] RTL/LTR mirroring يَعمَل بشَكل صَحيح
- [ ] الأَيقونات الاتِّجاهيّة تَنعَكِس
