# ADMIN_PAGE_INVENTORY — لوحة إدارة الأكاديمية (/admin)

> مصدر البيانات: فحص فعلي لكود Filament (`app/**/Filament/**/*Resource.php`)، 2026-07-16.
> اللوحة تحوي **50 موردًا (Resource)** موزّعة على **3 لوحات**: `/admin` (guard `web`)،
> `/instructor` (guard `web`)، `/super` (guard `system`). الأدمن **منظّم مسبقًا** إلى
> **12 مجموعة تنقّل عربية** — البنية الحالية جيدة، والتحسينات أدناه منخفضة المخاطر.

## اللوحات
| اللوحة | المسار | الحارس | العلامة |
|---|---|---|---|
| إدارة الأكاديمية | `/admin` | `web` | ومضات — لوحة التحكّم |
| بوابة المدرّب | `/instructor` | `web` | ومضات — بوابة المدرّب |
| السوبر (مركزية) | `/super` | `system` | Wamadat — Super Admin |

## مجموعات تنقّل /admin الحالية (فعلية)
المحتوى (8) · المبيعات (7) · المالية (5) · التعلّم والتقييم (5) · ومضات بلس (4) ·
الحوكمة (3) · التسويق (3) · المجتمع (2) · الدعم (2) · التشغيل (2) · الموقع (1) · المستخدمون (1).

---

## الجرد الكامل

الأعمدة: المورد · المجموعة · النموذج · الدور المستهدف · التصنيف المقترح · الأولوية.
التصنيفات: **أساسي** · **تقني-أخفِ خلف صلاحية** · **مرشّح للدمج** · **إعادة تسمية/نقل**.

### مجموعة: المحتوى (المدير + المدرب)
| المورد | النموذج | الدور | التصنيف | ملاحظة |
|---|---|---|---|---|
| ProgramResource | ProgramModel | admin/instructor | **أساسي** | العمود الفقري للكتالوج |
| LessonResource | LessonModel | admin/instructor | **أساسي** | مرشّح للنقل إلى «التعلّم» (أقرب لرحلة العمل) |
| CohortBatchResource | CohortBatchModel | admin | **أساسي** | الدفعات |
| LearningPathResource | LearningPathModel | admin | **أساسي** | المسارات |
| CategoryResource | CategoryModel | admin | **أساسي** | |
| InstructorResource | InstructorProfileModel | admin | **أساسي** | ملفات المدربين |
| TeamMemberResource | TeamMemberModel | admin | أساسي | فريق المنصّة الظاهر |
| MediaAssetResource | MediaAssetModel | admin | **تقني** | مكتبة الأصول — أبقِها لكن خلف صلاحية `media.manage` |

### مجموعة: المبيعات
| المورد | النموذج | التصنيف | ملاحظة |
|---|---|---|---|
| OrderResource | OrderModel | **أساسي** | الطلبات |
| CouponResource | CouponModel | **أساسي** | |
| GiftResource | ProgramGiftModel | **أساسي** | الهدايا |
| ConsultationRequestResource | ConsultationRequestModel | **أساسي** | الاستشارات |
| AbandonedCartResource | CartModel | تقني/تسويقي | **مرشّح للنقل** إلى «التسويق» (استرداد السلال) |
| ProgramInterestResource | ProgramInterestLeadModel | تسويقي | **مرشّح للنقل** إلى «التسويق» (Leads) |
| B2bAccountResource | (B2bAccountModel) | أساسي | تدريب الشركات — يستحق مجموعة/تسمية أوضح |

### مجموعة: المالية (دور Finance)
| المورد | النموذج | التصنيف | ملاحظة |
|---|---|---|---|
| InvoiceResource | InvoiceModel | **أساسي** | فواتير ZATCA |
| WalletResource | WalletModel | **أساسي** | المحافظ |
| BankTransferResource | OrderBankTransferModel | **أساسي حسّاس** | يحوي IBAN/أسماء — تأكّد الإخفاء في الجداول/الروابط |
| PlusPayoutResource | PlusPayoutRequestModel | **أساسي** | سحوبات بلس |
| PaymentWebhookResource | PaymentWebhookModel | **تقني** | أبقِها خلف صلاحية `payments.*`؛ لا تعرض الحمولة الخام |

### مجموعة: التعلّم والتقييم
| المورد | النموذج | التصنيف |
|---|---|---|
| QuizResource | QuizModel | **أساسي** |
| AssignmentResource | AssignmentModel | **أساسي** |
| LiveSessionResource | LiveSessionModel | **أساسي** |
| IssuedCertificateResource | IssuedCertificateModel | **أساسي** |
| CertificateTemplateResource | CertificateTemplateModel | أساسي |

### مجموعة: ومضات بلس (دور PlusSupervisor)
PlusServiceResource · PlusSellerResource · PlusCategoryResource · PlusDisputeResource — **كلها أساسية**. ملاحظة: **السحوبات (PlusPayout) في «المالية»** بينما بقية بلس في «ومضات بلس» — تشتيت بسيط؛ خيار: مرآة/رابط تحت بلس أيضًا.

### مجموعة: الحوكمة (دور Owner/Admin)
| المورد | النموذج | التصنيف | ملاحظة |
|---|---|---|---|
| TenantAuditLogResource | AuditLogModel | **تقني حسّاس** | سجل تدقيق المستأجر — للقراءة فقط، خلف صلاحية |
| PolicyVersionResource | PolicyVersionModel | أساسي | نسخ السياسات/الموافقات |
| UserConsentResource | UserConsentModel | تقني | موافقات PDPL — للقراءة فقط |

### مجموعة: التسويق
AffiliatePartnerResource · AffiliateConversionResource · NewsletterSubscriberResource — أساسية. **مرشّح للدمج البصري:** الأفلييت (شريك + تحويلات) في تبويبين تحت مورد واحد.

### مجموعة: المجتمع
ReviewResource (**أساسي**) · PartnerResource (**نقل مقترح** إلى «المحتوى/الموقع» — شعارات «موثوق من»).

### مجموعة: الدعم
SupportTicketResource (التذاكر، **أساسي**) · ContactMessageResource (**أساسي**).

### مجموعة: التشغيل (تقني — دور Admin فقط)
| المورد | النموذج | التصنيف | ملاحظة |
|---|---|---|---|
| EmailOutboxResource | EmailOutboxModel | **تقني** | مخرجات البريد — خلف صلاحية `ops.*`؛ لا تعرض نصّ البريد الحسّاس |
| FailedJobResource | FailedJobModel | **تقني** | مهام فاشلة — دور Admin فقط |

### مجموعة: الموقع / المستخدمون / المحتوى الرئيسي
| المورد | النموذج | التصنيف | ملاحظة |
|---|---|---|---|
| PageResource | PageModel | **أساسي** | باني الصفحات (نقطة انطلاق منشئ صفحات الهبوط — المرحلة 3) |
| UserResource | UserModel | **أساسي حسّاس** | يحوي هوية مشفّرة — تأكّد ألا تظهر `national_id` في الجدول/الفلاتر/التصدير |
| HomeCardResource · HomeFaqResource · TestimonialResource | — | أساسي | محتوى الرئيسية — مرشّح لمجموعة «الموقع» موحّدة |

### لوحة /super فقط (landlord — دور system)
TenantResource · PlanResource · SystemUserResource · AuditLogResource(المركزي) — **أساسية، معزولة عن /admin بحارس مختلف** (`system`). لا يجب أن تظهر أبدًا في /admin.

---

## ملخّص القرارات (منخفضة المخاطر)
1. **نقل** LessonResource → «التعلّم»، AbandonedCart + ProgramInterest → «التسويق»، Partner → «الموقع». (روابط فقط، بلا حذف.)
2. **دمج بصري** لموردَي الأفلييت في تبويبات مورد واحد.
3. **توحيد** HomeCard/HomeFaq/Testimonial تحت مجموعة «الموقع».
4. **تأكيد الصلاحيات** على التقنية/الحسّاسة: MediaAsset, PaymentWebhook, EmailOutbox, FailedJob, TenantAuditLog, UserConsent, BankTransfer.
5. **تدقيق تسريب حسّاس** (P1): UserResource + BankTransferResource — تأكّد ألا تظهر الهوية/IBAN في الأعمدة والفلاتر والتصدير والروابط.

> ⚠️ لا حذف لأي مورد — كلها مستخدمة. التحسينات نقل/تسمية/صلاحية فقط.
> التصنيفات الحسّاسة (5) هي بنود اختبار في `SECURITY_AUDIT.md`.
