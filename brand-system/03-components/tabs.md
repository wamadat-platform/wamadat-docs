# 03.11 — Tabs

> تَبديل بَين أقسام مُحتوى داخِل نَفس الصَفحَة.

---

## Variants

| Variant | متى |
|---|---|
| `Pills (Segmented)` | الافتراضيّ في المنصة ⭐ (مكون مبني على Radix) |
| `Underline` | غير مدعوم في المكون الافتراضي حالياً (تجنبه إلا لو استدعت الحاجة الماسة). |

---

## Pattern: Pill (Segmented Control)

المنصة تستخدم `@radix-ui/react-tabs` في `@/components/ui/tabs` لضمان إمكانية التنقل بلوحة المفاتيح (Arrows/Home/End) بشكل آلي.

```tsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';

<Tabs defaultValue="tab1" className="w-full">
    <TabsList>
        <TabsTrigger value="tab1">التفاصيل</TabsTrigger>
        <TabsTrigger value="tab2">المنهج</TabsTrigger>
        <TabsTrigger value="tab3">المدربون</TabsTrigger>
    </TabsList>
    
    <TabsContent value="tab1">
        محتوى التفاصيل هنا...
    </TabsContent>
    <TabsContent value="tab2">
        محتوى المنهج هنا...
    </TabsContent>
    <TabsContent value="tab3">
        محتوى المدربين هنا...
    </TabsContent>
</Tabs>
```

---

## القَواعِد

✅ **استخدم Radix Tabs (`@/components/ui/tabs`)** — يضمن `role="tablist"` و keyboard navigation.
✅ **3-7 tabs maximum** — أكثَر، استَخدِم dropdown.
❌ **لا تَستَخدِم tabs للمَسارات المُختَلِفَة** (URL routing) — استَخدِم links/Nav.
❌ **لا تُخفي محتوى مُهمّ** خَلف tab — قد لا يَكتَشِفه المُستَخدِم.
