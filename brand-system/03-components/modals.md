# 03.5 — Modals (Dialogs)

> الـ Modal مُقاطَعَة. استَخدِمه عند الحاجَة الحَقيقيّة فقط — تَأكيد إجراء، نَموذج مُهمّ، تَعليمات حَرِجَة.

---

## Anatomy

```
┌─────────────────── overlay (jet-950/60 + blur) ───┐
│                                                    │
│       ┌───── Modal ─────┐                          │
│       │  [Header]    [X]│                          │
│       │  Title          │                          │
│       │  Description    │                          │
│       │                 │                          │
│       │  [Content]      │                          │
│       │                 │                          │
│       │  [Footer]       │                          │
│       │  Cancel  Confirm│                          │
│       └─────────────────┘                          │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Sizes

| Size | Max-width |
|---|---|
| `sm` | `max-w-sm` (384px) — تأكيد سريع |
| `md` | `max-w-md` (448px) — الافتراضيّ ⭐ |
| `lg` | `max-w-lg` (512px) — نَماذِج مُتَوَسِّطَة |
| `xl` | `max-w-2xl` (672px) — نَماذِج طَويلَة |

---

## Component (Radix UI)

المنصة تستخدم `dialog.tsx` المبني على `@radix-ui/react-dialog` لضمان الوصولية (Accessibility) والتركيز التلقائي.

```tsx
// @/components/ui/dialog.tsx
import * as DialogPrimitive from '@radix-ui/react-dialog';
// ... (exporting Dialog, DialogTrigger, DialogContent, DialogHeader, etc.)
```

---

## Usage

```tsx
import { useState } from 'react';
import {
    Dialog,
    DialogContent,
    DialogDescription,
    DialogFooter,
    DialogHeader,
    DialogTitle,
    DialogTrigger,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

export function DeleteAccountModal() {
    const [open, setOpen] = useState(false);

    return (
        <Dialog open={open} onOpenChange={setOpen}>
            <DialogTrigger asChild>
                <Button variant="danger">حذف الحساب</Button>
            </DialogTrigger>
            <DialogContent className="sm:max-w-md">
                <DialogHeader>
                    <DialogTitle>حَذف الحِساب</DialogTitle>
                    <DialogDescription>
                        هذا الإجراء لا يُمكن التَراجُع عنه.
                    </DialogDescription>
                </DialogHeader>
                <div className="space-y-4">
                    <Input placeholder="اكتُب 'حذف' لِتَأكيد" />
                </div>
                <DialogFooter>
                    <Button variant="ghost" onClick={() => setOpen(false)}>إلغاء</Button>
                    <Button variant="danger">تَأكيد الحَذف</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    );
}
```

---

## القَواعِد

✅ **Esc يُغلِق** الـ modal دائماً.
✅ **Click خارج** يُغلِق (إلّا modals حَرِجَة كَحَذف).
✅ **Focus trap** — التَنَقُّل بـ Tab داخل الـ modal فقط.
✅ **Body scroll lock** عند الفَتح.
❌ **لا modals مُتَتالِيَة** (modal فوق modal).
❌ **لا تَستَخدِم لِتَنبيهات بَسيطَة** — استَخدِم Toast.
❌ **لا تَستَخدِم لِنَماذِج طَويلَة** — استَخدِم صَفحَة مُستَقِلَّة.
