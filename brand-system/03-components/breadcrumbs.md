# 03.9 — Breadcrumbs

> مَسار التَنَقُّل. يُعطي المُستَخدِم خَريطَة "وَين أنا، وَين كنت".

---

## Structure

```
الرَئيسيّة › البَرامج › التَعليق الصَوتيّ
```

### في RTL
- Separator: `‹` (chevron-left) — يَنعَكِس بـ `rtl:rotate-180`
- التَدفُّق من **اليَمين إلى اليَسار**

---

## Anatomy

| Item | Style |
|---|---|
| Items سابِقَة | `text-jet-500 hover:text-orange-700` |
| Current page (آخِر) | `text-jet-700 font-medium` (لا link) |
| Separator | `ChevronLeft className="rtl:rotate-180 text-jet-300 size-3.5"` |

---

## Component

```tsx
// frontend/components/layouts/breadcrumbs.tsx
import Link from 'next/link';
import { ChevronLeft } from 'lucide-react';

export interface BreadcrumbItem {
    label: string;
    href?: string;
}

export function Breadcrumbs({ items }: { items: BreadcrumbItem[] }) {
    if (items.length === 0) return null;
    return (
        <nav aria-label="مسار التَنَقُّل" className="text-sm">
            <ol className="flex items-center gap-1.5 flex-wrap">
                {items.map((item, idx) => {
                    const isLast = idx === items.length - 1;
                    return (
                        <li key={idx} className="flex items-center gap-1.5">
                            {idx > 0 && (
                                <ChevronLeft className="size-3.5 text-jet-300 rtl:rotate-180" aria-hidden />
                            )}
                            {isLast || !item.href ? (
                                <span className="text-jet-700 font-medium" aria-current="page">
                                    {item.label}
                                </span>
                            ) : (
                                <Link href={item.href as never} className="text-jet-500 hover:text-orange-700 transition-colors">
                                    {item.label}
                                </Link>
                            )}
                        </li>
                    );
                })}
            </ol>
        </nav>
    );
}
```

---

## Usage

```tsx
<Breadcrumbs items={[
    { label: 'الرَئيسيّة', href: '/' },
    { label: 'البَرامج', href: '/programs' },
    { label: 'الإعلام والصَوت', href: '/programs?category=voice-and-media' },
    { label: 'مُهارَات التَعليق الصَوتيّ' }, // current page — no href
]} />
```

---

## القَواعِد

✅ **آخِر item بدون link** — هي الصَفحَة الحاليّة.
✅ **`aria-current="page"`** على الآخِر.
✅ **Chevron يَنعَكِس في RTL** بـ `rtl:rotate-180`.
✅ **3-5 levels قَصوى** — أكثَر يَعني هَيكَل URL مَكسور.
❌ **لا تَستَخدِمها على home page** — لا فائدة.
❌ **لا تَستَخدِمها كـ navigation primary** — هي مَسار، ليس قائِمَة.
