# 02.1 — نظام الألوان (Colors)

> الألوان أداة لا زِينَة. كلّ لون له وَظيفَة. الـ Orange ليس "جَميل" — هو إشارَة فِعل.

---

## فَلسَفَة 60–30–10

| النِسبة | اللون | الدَور |
|---|---|---|
| 60% | **White** `#FFFFFF` | الخَلفيّات، المَساحات السَلبيّة |
| 30% | **Jet** `#343333` | النُصوص، الـ borders الثَقيلَة، Footer |
| 10% | **Orange** `#FAAF3C` | CTAs، التَأكيدات، الحالات النَشِطَة |

> القاعِدَة الذَهَبيّة: إذا رأيت Orange يَأخُذ أكثر من 10% من شاشة، **هذا خَطَأ**.

---

## Primary Palette

```css
--wamadat-white: #FFFFFF;
--wamadat-jet:   #343333;  /* Brand */
--wamadat-orange: #FAAF3C; /* Brand */
```

---

## Jet Scale — للنُصوص والـ borders والـ surfaces الداكِنَة

| Token | Hex | الاستخدام |
|---|---|---|
| `jet-50`  | `#F7F7F7` | Page background (subtle) |
| `jet-100` | `#F0F0F0` | Card borders، dividers |
| `jet-200` | `#E0E0E0` | Input borders، dividers ثَقيلَة |
| `jet-300` | `#C2C2C2` | Disabled text، placeholders |
| `jet-400` | `#999999` | Muted text، secondary labels |
| `jet-500` | `#707070` | Caption text، meta |
| `jet-600` | `#555555` | Body text (relaxed) |
| `jet-700` | `#444444` | Body text (default) |
| `jet-800` | `#383838` | Headings (alt) |
| `jet-900` | `#343333` | **Brand. Headings الأساسيّة.** |
| `jet-950` | `#1A1A1A` | Footer، Hero overlays الداكِنَة |

---

## Orange Scale — للـ accents فقط

| Token | Hex | الاستخدام |
|---|---|---|
| `orange-50`  | `#FFF7E6` | Coupon banner، subtle highlight |
| `orange-100` | `#FFE8B8` | Active link background |
| `orange-200` | `#FFD68A` | Pill backgrounds |
| `orange-300` | `#FFC55C` | Hover states (very light) |
| `orange-400` | `#FFB54A` | Light accents |
| `orange-500` | `#FAAF3C` | **Brand. CTAs الأساسيّة.** |
| `orange-600` | `#E89B2A` | CTA hover |
| `orange-700` | `#C9821C` | CTA active، link text |
| `orange-800` | `#A66A14` | Dark text on light orange |
| `orange-900` | `#82530E` | Gold ribbon stops |

---

## Semantic Colors — للحالات فقط

```css
--success: #16A34A;  /* نَجاح. Confirmations، إكمال. */
--warning: #EAB308;  /* تَحذير. تَأخير، تَنبيه غير حَرِج. */
--danger:  #DC2626;  /* خَطَر. خَطَأ، حَذف. */
--info:    #2563EB;  /* مَعلومة. تَلميحات، توضيحات. */
```

**قاعِدَة:** لا تَستَخدِم Semantic كَلَون أساسيّ في تَصميم. هي **حَصراً** لحالات.

---

## التَباين (Contrast) — WCAG AA

كل تَركيبَة هنا تَجتاز WCAG AA (4.5:1 للنَصّ العاديّ، 3:1 للنَصّ الكَبير).

| Background | Foreground | Ratio | حُكم |
|---|---|---|---|
| `bg-white` | `text-jet-900` | 12.6:1 | ✅ Excellent |
| `bg-white` | `text-jet-700` | 7.5:1 | ✅ AA |
| `bg-white` | `text-jet-500` | 4.6:1 | ✅ AA (للنَصّ العاديّ) |
| `bg-white` | `text-jet-400` | 3.4:1 | ⚠️ AA Large فقط (18px+) |
| `bg-white` | `text-orange-700` | 4.6:1 | ✅ AA |
| `bg-white` | `text-orange-500` | 2.5:1 | ❌ لا تَستَخدِم للنَصّ |
| `bg-jet-900` | `text-white` | 12.6:1 | ✅ Excellent |
| `bg-jet-900` | `text-orange-500` | 5.0:1 | ✅ AA |
| `bg-orange-500` | `text-jet-900` | 8.4:1 | ✅ Excellent |
| `bg-orange-500` | `text-white` | 2.6:1 | ❌ نَصّ كَبير فقط، Bold |

> **القاعِدَة:** نَصّ أبيض على Orange ممنوع للأحجام الصَغيرة. استَخدِم نَصّ Jet على Orange.

---

## القَواعِد الصارِمَة

### ✅ نَفعَل
- **Cards**: `bg-white` + `border-jet-100`.
- **Subtle sections**: `bg-jet-50` (نادراً).
- **Hero text**: `text-jet-900` على `bg-white`.
- **Primary buttons**: `bg-orange-500` + `text-jet-900` (ليس أبيض).
- **Footer**: `bg-jet-900` + `text-white` + `text-orange-500` accents.

### ❌ لا نَفعَل
- لا نَستَخدِم `bg-orange-500` كَخَلفيّة قِسم كامِل (إلّا Hero CTA bar نادراً).
- لا نَخلِط ألواناً من خارِج النِظام أبداً — لا `bg-blue-500`، لا `bg-pink-300`.
- لا نَستَخدِم Gradients مُتَعَدِّدَة الألوان (Gradient واحد مَسموح: jet-800 → jet-900 للـ Hero الداكِن).
- لا نَستَخدِم Orange فاتح (`orange-100`/`200`) كَنَصّ على أبيض — التَباين فاشِل.
- لا نَستَخدِم Semantic colors لِأشياء غَير دَلاليّة (مَثَل green ≠ "خَيار جَيِّد"، green = "نَجاح حَصَل").

---

## دالّة "أيّ لون أَستَخدِم؟"

```
هل هذا CTA رَئيسيّ؟
├── نَعَم → bg-orange-500 text-jet-900 hover:bg-orange-600
└── لا
    │
    هل هو إجراء ثانويّ؟
    ├── نَعَم → bg-white border-jet-200 text-jet-900
    │
    هل هو نَصّ؟
    ├── عُنوان → text-jet-900
    ├── جِسم → text-jet-700
    ├── مُساعِد → text-jet-500
    │
    هل هو حالة؟
    ├── نَجاح → text-success bg-green-50
    ├── تَحذير → text-warning bg-amber-50
    └── خَطَأ → text-danger bg-red-50
```

---

## التَطبيق في Tailwind

كلّ هذا مَوجود في `frontend/tailwind.config.ts`. **لا تُضِف لَوناً خارجَ النِظام.** لو احتَجت لَوناً جَديداً، افتَح PR في `08-governance/`.

---

## Dark Mode (Roadmap v1.1)

Dark mode سَيُضاف في v1.1. خَريطة التَحويل المَبدئيّة:

| Light | Dark |
|---|---|
| `bg-white` | `bg-jet-950` |
| `text-jet-900` | `text-jet-50` |
| `bg-jet-50` | `bg-jet-900` |
| `border-jet-100` | `border-jet-700` |

Orange يَبقى كَما هو في كلا الوَضعَين (الـ accent).
