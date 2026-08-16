# قائمة المراقبة اليومية — 5 دقائق

> **متى**: كلّ يوم بين 8:00 و 9:00 صباحاً (بعد ما الـ daily-summary task يولّد ملف الصباح في 08:00).
> **المدّة**: ≤5 دقائق لو كلّ شيء أخضر. ≤30 دقيقة لو ظهر شي.
> **الهدف**: التقاط أيّ مؤشّر سلبي قبل ما المستخدم يلاحظه.

---

## الرتم اليومي — 5 خطوات بالترتيب

### 1 — افتح ملخّص اليوم (دقيقة واحدة)

```
File Explorer → C:\Users\U\wamadat-platform\docs\operations\summaries\
→ افتح أحدث ملفّ summary-YYYYMMDD.txt
```

تحقّق من هذه السطور 4:

| السطر | الشرط الأخضر |
|---|---|
| `Errors 24h:` | ≤5 (أو رقم تعرفه ومتقبّله) |
| `Failed jobs 24h:` | 0 |
| `Active programs:` | 5/5 (at cap) |
| `Latest offsite bk:` | < 30h ago |

أيّ سطر برتقالي/أحمر → انتقل لـ `05-incident-escalation.md`.

---

### 2 — افتح Sentry (دقيقتان)

```
https://sentry.io/issues/
```

تحقّق:

| العنصر | الشرط الأخضر |
|---|---|
| **Inbox** | لا issues جديدة "first seen" خلال آخر 24 ساعة |
| **Total events 24h** | <10 (مع 1-10 مستخدمين، أيّ شيء فوق 10 يستحقّ نظرة) |
| **أحداث new releases** | لو في، تأكّد إنّك تتذكّر إصدار حديث |

أيّ issue جديد → اقرأ الـ stack trace، حدّد الخطورة من قاعدة `03-feedback-form.md §ب`، اتّبع `05-incident-escalation.md` لو P0 / P1.

---

### 3 — افحص alerts log (30 ثانية)

```powershell
Get-Content C:\Users\U\wamadat-platform\docs\operations\alerts-log.txt -Tail 10
```

الشرط الأخضر: كلّ السطور الأخيرة `OK failed_jobs=0`. أيّ سطر `ALERT` → افتح Filament `/admin → Failed Jobs`.

---

### 4 — افحص backup (15 ثانية)

```powershell
Get-ChildItem C:\Users\U\OneDrive\Wamadat-Backups -Filter "wamadat-backup-*.zip" | Sort-Object LastWriteTime -Descending | Select-Object -First 2
```

الشرط الأخضر: أحدث ملفّ من اليوم نفسه (أو ليلة أمس بعد 03:00).

---

### 5 — افحص رسائل WhatsApp/Telegram (دقيقتان)

- هل في رسالة دعم من مدعو بيتا لم تردّ عليها بعد؟ → ردّ الآن بسطر واحد على الأقل
- هل في ردّ جديد على استمارة Tally؟ → افتح `https://tally.so` وراجع

---

## شجرة قرار سريعة

```
كلّ السطور الـ 4 خضراء + Sentry هادي + لا رسائل معلّقة؟
   └─► ✅ يوم عادي. ابعث دعوة جديدة لو ضمن الـ cap. اشتغل في عمل آخر.

سطر واحد ≤ orange (warning، near cap، stale)؟
   └─► راقب اليوم القادم. لو استمر، اتّخذ قرار.

سطر واحد أحمر (over cap، drift، DOWN)؟
   └─► ⛔ توقّف عن إرسال دعوات جديدة. افتح `05-incident-escalation.md`.

Sentry فيه issue P0/P1 جديد؟
   └─► ⛔ توقّف. تابع escalation.
```

---

## ما لا تفعله صباحاً

- ❌ لا تشغّل `restore-drill.bat` يومياً — مرّة في الأسبوع كافية (الأحد مثلاً)
- ❌ لا تعدّل `.env` أو تشغّل migrations في الصباح — الصباح مراقبة، التعديل بعد الظهر بعد ما تتأكّد من الحالة
- ❌ لا تستجب لـ Sentry alerts من جوّالك في 2:00 ليلاً — حدّد ساعة المراقبة، التزم فيها
- ❌ لا تَفعل أيّ هندسة قبل ما تنهي الـ 5 خطوات أعلاه

---

## أسبوعي (الأحد)

بعد الـ checklist اليومية:

1. شغّل `restore-drill.bat` → آخر سطر في `restore-drill-log.txt` لازم PASS
2. تحقّق نموّ DB من الـ `[Storage]` block على مدى آخر 7 أيام
3. راجع backlog "Beta feedback" — هل في P2/P3 يستحقّ أن يصبح P1؟
4. لو الأسبوع 7 أيام خضراء كاملة → اعتبر زيادة الـ cap درجة واحدة (5→7 برامج، 50→60 مستخدم) **بعد** قراءة `docs/operations/06-closed-beta.md §exit criteria`

---

## شهرياً (أول أحد من الشهر)

- راجع نموّ Sentry events: هل المعدّل اليومي يرتفع؟ سبب؟
- راجع OneDrive backup retention: هل >30 ملفّ؟ احذف الأقدم يدوياً
- راجع Tally responses: هل في pattern؟ موضوع يتكرّر؟
- قرار: استمرار البيتا، توسيع البيتا، أو الانتقال للـ Open Cohort (راجع `06-closed-beta.md` §exit criteria)
