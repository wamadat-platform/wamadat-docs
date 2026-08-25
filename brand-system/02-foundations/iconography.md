# 02.5 — الأيقونات (Iconography)

> Lucide-react فقط. Stroke 1.5px. Orange افتراضيّاً. لا emojis في الـ UI.

---

## المَكتَبَة (Library)

**Lucide React** — لِسَبَبَين:
1. مَكتَبَة كَبيرَة (~1400 أيقونَة)
2. Stroke-based، نَظيفَة، تَتَناسَب مع الـ DNA البَصريّ لـ ومضات

تَثبيت (مَوجود مُسبَقاً):
```bash
pnpm add lucide-react
```

استخدام:
```tsx
import { Heart, Star, BookOpen } from 'lucide-react';
```

---

## الأنماط (Styling)

### Stroke Width
```css
stroke-width: 1.5;  /* الافتراضيّ في ومضات */
```

> أنحف من stroke=2 (الافتراضيّ في Lucide). يُعطي أَنَاقَة وخِفّة.

### Default Color
```tsx
<Heart className="size-5 text-orange-500" />
```

### Inside Buttons (يَرِث لون الزِرّ)
```tsx
<button className="bg-orange-500 text-jet-900">
    <ArrowLeft className="size-4" />
    التالي
</button>
```

`text-current` تلقائيّاً يَجعَل الأيقونَة تَأخُذ لون النَصّ.

---

## الأحجام (Sizes)

| Token | Pixel | استخدام |
|---|---|---|
| `size-3` | 12px | inline في text صَغير |
| `size-4` | 16px | inside buttons، list items |
| `size-5` | 20px | الأكثَر شِيوعاً ⭐ |
| `size-6` | 24px | Section headers، action items |
| `size-8` | 32px | feature icons في cards |
| `size-12` | 48px | Hero icons، empty states |
| `size-16` | 64px | Large illustrations |

> **استَخدِم `size-*` (Tailwind 3.4+) بدل `w-* h-*` المُنفَصِلَة.**

---

## الألوان السياقيّة (Contextual Colors)

| السياق | اللون |
|---|---|
| تَزييني | `text-orange-500` |
| داخل زِرّ | `text-current` (يَرِث) |
| نَجاح | `text-green-600` |
| تَحذير | `text-amber-600` |
| خَطَأ | `text-red-600` |
| مُعَطَّل (Disabled) | `text-jet-300` |
| Muted | `text-jet-400` |

---

## أيقونات شائِعَة في المنصّة

| الاستخدام | الأيقونَة |
|---|---|
| البرامج | `BookOpen` |
| المُدرِّبون | `GraduationCap` أو `Users` |
| الشَهادات | `Award` |
| السلّة | `ShoppingBag` |
| التَقييم | `Star` |
| الإشعارات | `Bell` |
| الإعدادات | `Settings` |
| البحث | `Search` |
| المُستَخدِم | `User` |
| القائِمَة | `Menu` |
| إغلاق | `X` |
| التالي (RTL يَنعَكِس) | `ArrowLeft` + `rtl:rotate-180` |
| السابق (RTL يَنعَكِس) | `ArrowRight` + `rtl:rotate-180` |
| تَحَقُّق | `Check` |
| تَنبيه | `AlertCircle` |
| حَذف | `Trash2` |
| تَحرير | `Edit` |
| تَحميل | `Download` |
| رَفع | `Upload` |
| ساعَة | `Clock` |
| تاريخ | `Calendar` |
| مَكان | `MapPin` |
| هاتف | `Phone` |
| إيميل | `Mail` |
| واتساب | `MessageCircle` |
| تَشغيل فيديو | `PlayCircle` |
| مَفتاح آمن | `Lock`، `Shield` |

---

## RTL & Iconography

### أيقونات **لا تَنعَكِس** في RTL
- User, Bell, Heart, Star (رُموز عامّة)
- Settings, Mail, Phone (مَفاهيم)
- Lock, Shield (أمن)

### أيقونات **تَنعَكِس** في RTL
- ArrowLeft → يَبدو "السابق" في LTR، "التالي" في RTL
- ArrowRight → عَكس
- ChevronLeft / ChevronRight → نَفسه
- Send (سَهم الإرسال)

```tsx
<ArrowLeft className="size-4 rtl:rotate-180" />
```

> **نَهج:** استَخدِم `rtl:rotate-180` (أو `rtl:scale-x-[-1]` لِلانعكاس) للأيقونات الاتِّجاهيّة.

### استثناءات (تَبقى كَما هي)
- Quote marks (`"...")
- Music notes
- Logo icons تَجاريّة

---

## مَتى لا نَستَخدِم أيقونَة؟

❌ **لا تَستَخدِم أيقونَة "للزَخرَفَة"** — كلّ أيقونَة يَجب أن تُضيف معنى.
❌ **لا تَخلِط مَكتَبَتَين** — Lucide فقط. لا Heroicons، لا FontAwesome.
❌ **لا تَستَخدِم Filled icons** — كلّ أيقوناتنا stroke (Lucide stroke variant، لا "Lucide-fill").
❌ **لا emojis في الـ UI الأساسيّ** — مَسموح فقط في:
- مَحادَثات (chat messages)
- مُحتوى المُستَخدِم (تَقييم 5/5 ⭐)
- البلوغ (المَحتوى التَسويقيّ الجانبيّ)

---

## أنماط شائِعَة

### Icon + Text
```tsx
<button className="flex items-center gap-2">
    <Heart className="size-4" />
    إضافة لِقائمَة الأمنيات
</button>
```

### Icon داخل بَطاقَة الحال
```tsx
<div className="size-12 rounded-xl bg-orange-100 flex items-center justify-center">
    <Award className="size-6 text-orange-700" />
</div>
```

### Icon أساسيّ مع Badge
```tsx
<button className="relative">
    <Bell className="size-6" />
    {unread > 0 && (
        <span className="absolute -top-1 -end-1 bg-orange-500 text-jet-900 size-5 text-xs font-bold rounded-full flex items-center justify-center">
            {unread}
        </span>
    )}
</button>
```

### Empty State Icon
```tsx
<div className="text-center py-12">
    <BookOpen className="size-16 text-jet-300 mx-auto mb-4" />
    <h2 className="text-xl font-bold text-jet-800 mb-2">لم تُسَجِّل في بَرنامج بَعد</h2>
</div>
```

---

## الـ Accessibility

كلّ أيقونَة وَحدها (بدون نَصّ مُجاوِر) **يَجب** أن تَملك `aria-label`:

```tsx
{/* صَحيح */}
<button aria-label="إغلاق">
    <X className="size-5" />
</button>

{/* خَطَأ — لا aria-label */}
<button>
    <X className="size-5" />
</button>
```

الأيقونات مع نَصّ مُجاوِر **لا تَحتاج** aria-label (الزِرّ نَفسه له نَصّ):

```tsx
<button>
    <Heart className="size-4" />
    قائِمَة الأمنيات
</button>
```

---

## التَخصيص (Customization)

لو احتَجت أيقونَة غَير مَوجودَة في Lucide:
1. ابحَث في Lucide أوّلاً (1400+ أيقونَة، الاحتمال كَبير مَوجودَة).
2. لو غَير مَوجودَة، استَخدِم SVG inline.
3. لا تَستَورِد مَكتَبَة جَديدَة.

```tsx
<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="1.5"
     className="size-5 text-orange-500" aria-hidden="true">
    <path d="..." />
</svg>
```

> اتَّبع نَفس قَواعِد stroke=1.5 و currentColor.
