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

## Component (Headless UI)

```tsx
import * as React from 'react';
import { Dialog, Transition } from '@headlessui/react';
import { X } from 'lucide-react';
import { Fragment } from 'react';

export function Modal({
    open,
    onClose,
    title,
    description,
    size = 'md',
    children,
}: {
    open: boolean;
    onClose: () => void;
    title?: string;
    description?: string;
    size?: 'sm' | 'md' | 'lg' | 'xl';
    children: React.ReactNode;
}) {
    const sizeClasses = {
        sm: 'max-w-sm',
        md: 'max-w-md',
        lg: 'max-w-lg',
        xl: 'max-w-2xl',
    };

    return (
        <Transition show={open} as={Fragment}>
            <Dialog onClose={onClose} className="relative z-50">
                <Transition.Child
                    as={Fragment}
                    enter="ease-out duration-300"
                    enterFrom="opacity-0"
                    enterTo="opacity-100"
                    leave="ease-in duration-200"
                    leaveFrom="opacity-100"
                    leaveTo="opacity-0"
                >
                    <div className="fixed inset-0 bg-jet-950/60 backdrop-blur-sm" />
                </Transition.Child>

                <div className="fixed inset-0 flex items-center justify-center p-4">
                    <Transition.Child
                        as={Fragment}
                        enter="ease-out duration-300"
                        enterFrom="opacity-0 scale-95"
                        enterTo="opacity-100 scale-100"
                        leave="ease-in duration-200"
                        leaveFrom="opacity-100 scale-100"
                        leaveTo="opacity-0 scale-95"
                    >
                        <Dialog.Panel className={`relative w-full ${sizeClasses[size]} bg-white rounded-2xl shadow-2xl p-6`}>
                            {title && (
                                <header className="mb-4 pe-8">
                                    <Dialog.Title className="text-xl font-bold text-jet-900">{title}</Dialog.Title>
                                    {description && (
                                        <Dialog.Description className="mt-1.5 text-sm text-jet-500">{description}</Dialog.Description>
                                    )}
                                </header>
                            )}

                            <button
                                onClick={onClose}
                                className="absolute top-4 end-4 p-1.5 rounded-lg hover:bg-jet-50"
                                aria-label="إغلاق"
                            >
                                <X className="size-5 text-jet-500" />
                            </button>

                            {children}
                        </Dialog.Panel>
                    </Transition.Child>
                </div>
            </Dialog>
        </Transition>
    );
}
```

---

## Usage

```tsx
const [open, setOpen] = useState(false);

<Modal
    open={open}
    onClose={() => setOpen(false)}
    title="حَذف الحِساب"
    description="هذا الإجراء لا يُمكن التَراجُع عنه."
>
    <div className="space-y-4">
        <Input placeholder="اكتُب 'حذف' لِتَأكيد" />
        <div className="flex gap-3 justify-end">
            <Button variant="ghost" onClick={() => setOpen(false)}>إلغاء</Button>
            <Button variant="danger">تَأكيد الحَذف</Button>
        </div>
    </div>
</Modal>
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
