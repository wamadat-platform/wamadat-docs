# 02.2 — الطِباعَة (Typography)

> الخَطّ صَوت العَلامَة على الشاشة. خَطّ واحد، 4 أوزان، 10 أحجام، قَواعِد صارِمَة.

---

## العائلة (Family)

**Tajawal** — مَصمَّم من قِبَل Boutros Type عام 2017، يَدعَم العَربيّة واللاتينيّة بِنفس النَفَس البَصريّ.

```css
font-family: 'Tajawal', 'Tahoma', 'Segoe UI', sans-serif;
```

**لماذا Tajawal؟**
- أصيل للعَربيّة (لا تَزويق غَريب)
- يَتَعامل مع المسافات العَريضَة بَين الكَلِمات
- مَفتوح ومَجّاني (Google Fonts)
- مَدعوم بـ 4 أوزان فقط — يَفرِض الحَزْم

---

## الأوزان (Weights) — 4 فقط

| Weight | Value | الاستخدام |
|---|---|---|
| Regular | 400 | النُصوص العاديّة (body, captions) |
| Medium | 500 | تَأكيد خَفيف (labels، meta) |
| Bold | 700 | العَناوين، الـ CTAs |
| Black | 900 | Hero titles، Display text |

> **ممنوع:** Thin (100)، ExtraBold (800). **Light (300)** مسموح **للـ Captions والتواقيع فقط** (لا نصوص متن — رفيع على العربية)؛ راجع `docs/00-brand-identity.md` §4.2. لو احتجت غيرها، الـ hierarchy لديك مَكسور — أَعِد التَفكير.

---

## Type Scale — Major Third (1.250)

```
xs:   12px (0.75rem)   ← السطور الصَغيرة، fine print
sm:   14px (0.875rem)  ← Captions، النصوص الثانوية
base: 16px (1rem)      ← النَصّ الأساسيّ ⭐
lg:   18px (1.125rem)  ← النَصّ البارِز
xl:   20px (1.25rem)   ← عَناوين فَرعيّة
2xl:  24px (1.5rem)    ← عَناوين أقسام
3xl:  30px (1.875rem)  ← عَناوين صَفحات
4xl:  36px (2.25rem)   ← عَناوين بارِزَة
5xl:  48px (3rem)      ← Hero secondary
6xl:  60px (3.75rem)   ← Hero primary
7xl:  72px (4.5rem)    ← Display (نادر)
```

> **لا نَستَخدِم أحجام خارِج هذا السُلَّم.** لو احتَجت 22px، فأنت بَين 20 و24 — اختَر واحِداً.

---

## Line Height (ارتِفاع السَطر)

| Token | Value | استخدام |
|---|---|---|
| `tight` | 1.25 | العَناوين الكَبيرَة (5xl+) |
| `snug` | 1.375 | العَناوين المُتَوَسِّطَة (3xl-4xl) |
| `normal` | 1.5 | النُصوص الإنجليزيّة |
| `relaxed` | 1.75 | **النُصوص العَربيّة الافتراضيّة** ⭐ |
| `loose` | 2 | الفقرات الطَويلَة (مَقالات) |

> العَربيّة تَحتاج line-height أعلى من الإنجليزيّة — الحُروف فيها تَشكيل وذُيول.

---

## Letter Spacing (المَسافَة بَين الحُروف)

| Token | Value | استخدام |
|---|---|---|
| `tighter` | -0.05em | Display خاصّة |
| `tight` | -0.025em | العَناوين الكَبيرَة الإنجليزيّة |
| `normal` | 0 | **الافتراضيّ — العَربيّة دائماً** ⭐ |
| `wide` | 0.025em | All-caps، الأرقام (نادراً) |

> **العَربيّة:** لا تَستَخدِم letter-spacing أبداً. الحُروف تُكتَب مُتَّصِلَة، أيّ تَباعُد يَكسِر القِراءَة.

---

## أنماط الاستخدام (Type Patterns)

### Hero Title (الصَفحة الرَئيسيّة)
```tsx
<h1 className="text-4xl md:text-6xl font-black tracking-tight leading-tight text-jet-900">
    مَهارَة جاهِزَة من اليوم الأوّل
</h1>
```

### Page Title (H1)
```tsx
<h1 className="text-3xl md:text-4xl font-bold leading-tight text-jet-900">
    البَرامج التَدريبيّة
</h1>
```

### Section Title (H2)
```tsx
<h2 className="text-2xl font-bold text-jet-900 mb-4">
    المُدرِّبون المُمَيَّزون
</h2>
```

### Subsection Title (H3)
```tsx
<h3 className="text-xl font-bold text-jet-900 mb-2">
    ماذا ستُنجِز؟
</h3>
```

### Body
```tsx
<p className="text-base leading-relaxed text-jet-700">
    تَخرُج من ومضات وأنت قادر على التَطبيق فوراً.
</p>
```

### Caption / Meta
```tsx
<span className="text-sm text-jet-500">
    آخر تَحديث: ١٢ مايو ٢٠٢٦
</span>
```

### Fine Print
```tsx
<small className="text-xs text-jet-400">
    تَنطَبِق الشُروط والأحكام.
</small>
```

---

## Headlines vs. Body — قاعِدَة الأَلوان

| العُنصُر | اللون | الـ Weight |
|---|---|---|
| H1–H3 | `text-jet-900` | 700 (bold) أو 900 (black) |
| H4–H6 | `text-jet-800` | 700 |
| Body default | `text-jet-700` | 400 |
| Body emphasis | `text-jet-900` | 500 |
| Meta / Caption | `text-jet-500` | 400 |
| Fine print | `text-jet-400` | 400 |

---

## الأرقام (Numbers)

```css
.tabular-nums {
    font-variant-numeric: tabular-nums;
}
```

**استخدم `tabular-nums` دائماً للأرقام في:**
- الأسعار
- الإحصاءات
- العَدّاد (Counter)
- التَواريخ والأوقات
- أيّ أرقام تَتَغَيَّر (لو لم تَستَخدمها، الأرقام "تَرقُص" عند التَحديث)

```tsx
<span className="text-2xl font-bold tabular-nums">٤,١٩٥</span>
<span className="text-sm text-jet-500"> ر.س</span>
```

---

## الأرقام العَربيّة (Arabic-Indic) ضد اللاتينيّة

| السياق | الاستخدام |
|---|---|
| النَصّ الأدَبي / الرَواية | عَربيّة `٠ ١ ٢ ٣` |
| التَواريخ في المَقالات | عَربيّة |
| الـ UI (أسعار، إحصاءات، مَسلَسَلات) | لاتينيّة `0 1 2 3` |
| النَماذِج (Forms) | لاتينيّة (دائماً) |
| كود المُنتَج، أرقام الطُلبات | لاتينيّة |

> **سَبَب:** اللاتينيّة أَسهَل قِراءَة بسُرعَة، والمُستَخدِم العَربيّ مُعتاد عليها في الـ UI.

---

## الـ Anti-patterns

❌ **العَناوين المُتَدَرِّجَة:** `bg-gradient-to-r from-orange-500 to-purple-500` — لا.
❌ **All-caps العَربيّة:** العَربيّة ليس فيها حُروف كَبيرَة وصَغيرَة.
❌ **خُطوط متعدِّدَة:** Tajawal فقط، لا ندخل Cairo أو Almarai.
❌ **Italic للعَربيّة:** الخَطّ المائل غَريب على القارئ العَربيّ.
❌ **Underline تَزييني:** الخَطّ تحت النَصّ مَحجوز للـ links فقط.
❌ **أحجام صَغيرَة جدّاً:** لا تَنزِل تحت `text-xs` (12px) أبداً للنَصّ القابِل للقراءَة.

---

## Hierarchy Test

عند فَتح أيّ صَفحة، يَجب أن يُلاحِظ القارِئ التَدرّج خِلال 3 ثَوانٍ:

1. **العُنوان** يَقفِز فوراً (Hero أو Page Title).
2. **Subtitle/Tagline** يَأخذ المَكان الثاني.
3. **Body content** يَنتَظِر.

إذا كلّ شَيء بَدا بنَفس الأهَمّيّة، الـ hierarchy مَكسور.

---

## التَطبيق في Tailwind

كلّ القِيَم مُسَجَّلة في `frontend/tailwind.config.ts` ضِمن `extend.fontSize` و `extend.fontFamily`. النُمَيِّجات الجاهِزَة (presets) مُتَاحَة في `frontend/components/ui/typography.tsx` (مَحجوز للـ v1.1).
