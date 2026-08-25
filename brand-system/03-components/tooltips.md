# 03.13 — Tooltips

> تَلميحَات قَصيرَة على hover/focus. للمَعلومات الثانويّة فقط — لا تَستَخدِمها لِنَصّ حَرِج.

---

## Pattern (مَع Radix UI)

```tsx
import * as Tooltip from '@radix-ui/react-tooltip';

<Tooltip.Provider>
    <Tooltip.Root>
        <Tooltip.Trigger asChild>
            <button aria-label="مَعلومات إضافيّة">
                <Info className="size-4 text-jet-400" />
            </button>
        </Tooltip.Trigger>
        <Tooltip.Portal>
            <Tooltip.Content
                className="
                    bg-jet-900 text-white text-xs px-3 py-2 rounded-lg shadow-lg
                "
                side="top"
                sideOffset={6}
            >
                هذه شَهادَة مُعتَمَدَة من أكاديميّة ومضات
                <Tooltip.Arrow className="fill-jet-900" />
            </Tooltip.Content>
        </Tooltip.Portal>
    </Tooltip.Root>
</Tooltip.Provider>
```

---

## القَواعِد

✅ **محتوى قَصير** (5-15 كلمة).
✅ **Hover + Focus** يُظهِرها (touch screens لا تَملك hover — استَخدِم Popovers).
✅ **`aria-describedby`** يَربط الـ trigger بالـ content.
❌ **لا تَستَخدِمها لِنَصّ حَيَوي** — قد لا يَراه مَستَخدِم touch.
❌ **لا scroll داخِلها** — قَصيرَة دائماً.
❌ **لا أَزرار داخِلها** — هي للقِراءَة، ليس للتَفاعُل.
