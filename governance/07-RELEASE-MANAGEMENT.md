# منصة ومضات التعليمية — إدارة الإصدارات
## RELEASE MANAGEMENT STANDARD

**الإصدار:** 1.0  
**الحالة:** `LIVING RELEASE POLICY`  
**المراجع التقنية:** `../technical/04-Production-Deployment-and-Operations.md` + `../technical/05-Backup-Restore-and-Rollback.md`  

---

# 1. الهدف

جعل كل تغيير Production قابلًا للإجابة عن:

- ماذا تغير؟
- لماذا؟
- ما SHA/images المستخدمة؟
- هل توجد Migration؟
- كيف تم التحقق؟
- ما خطة التراجع؟
- ما النتيجة بعد النشر؟

---

# 2. نموذج الإصدار

نستخدم:

```text
MAJOR.MINOR.PATCH
```

## PATCH — `1.0.x`

- Bug fixes.
- Security/operational corrections متوافقة.
- UX fixes صغيرة لا تضيف Product capability كبيرة.

## MINOR — `1.x.0`

- Feature/Capability جديدة متوافقة.
- حزمة Product واضحة لها Scope/UAT.

## MAJOR — `2.0.0`

- تغيير جوهري في حدود المنتج/العقود/النموذج قد يتطلب Migration/transition واسع.

Semantic versioning هنا يستخدم **لإدارة المنتج والإصدارات الداخلية**؛ التوافق الدقيق للـpublic API يوثق في Contract/API versioning منفصل عند الحاجة.

---

# 3. Release Types

| النوع | مثال | Gate |
|---|---|---|
| `HOTFIX` | P0/P1 correction | مختصر لكن موثق |
| `PATCH` | مجموعة fixes | Standard |
| `MINOR` | Feature package | Full Feature + UAT Gate |
| `MAJOR` | توسع/تغيير حدود | Full + migration/strategy plan |
| `CONFIG` | تغير environment/provider/flag | Change record + verification |

حتى Config-only production change يحتاج سجلًا إذا كان يؤثر على سلوك المستخدم.

---

# 4. Release Manifest Contract

يمتلك كل Release سجلًا مثل:

```yaml
release_id: WAMADAT-1.1.0
release_type: MINOR
release_date:
owner:
backend_git_sha:
backend_image:
web_git_sha:
web_image:
changes:
  - WAM-0123
  - WAM-0140
migrations:
feature_flags:
config_changes:
external_dependencies:
backup_reference:
rollback_reference:
uat_reference:
release_notes:
status:
```

التفاصيل التنفيذية للصور وCoolify والـpreflight تبقى في `technical/04` ولا تكرر هنا.

---

# 5. Release States

```text
DRAFT
→ CODE_COMPLETE
→ VERIFYING
→ UAT
→ RELEASE_READY
→ APPROVED
→ DEPLOYING
→ VERIFYING_PRODUCTION
→ RELEASED
→ MONITORING
→ CLOSED
```

حالات استثنائية:

- `NO_GO`
- `ROLLED_BACK`
- `FORWARD_FIX_REQUIRED`
- `PARTIALLY_ENABLED`

---

# 6. Pre-Release Gate

قبل Go:

- [ ] Scope مرتبط بـBacklog IDs.
- [ ] كل Feature Done وفق معيارها.
- [ ] Release notes draft.
- [ ] Config/flags معروفة.
- [ ] Migration reviewed إن وجدت.
- [ ] Backup requirement fulfilled وفق technical policy.
- [ ] Rollback/forward-fix path مفهوم.
- [ ] External provider readiness confirmed إن وجد.
- [ ] Monitoring queries/dashboards/log points معروفة.
- [ ] Product/QA approval حسب نوع Release.

---

# 7. Deployment Rule

المرجع التقني الحالي يعتمد Build/Images immutable وRelease manifest. هذه الوثيقة لا تغيّر ذلك.

القاعدة التنظيمية:

> لا يصح أن يقال «نشرنا آخر نسخة»؛ يجب ذكر `release_id` ومرجع build/image/sha وفق المنصة التقنية.

---

# 8. Production Verification

بعد النشر:

## Minimum Smoke

- Public site reachable.
- Authentication critical path.
- API health.
- No obvious 5xx spike.
- Queue/scheduler health relevant to change.

## Change-Specific Smoke

يُشتق من العناصر المنشورة، مثل:

- Payment when payment code changed.
- Enrollment when access changed.
- Certificate when completion/certificate changed.
- Permission checks when role/access changed.

لا نعيد full regression يدويًا لكل Patch إذا لم يبرر Risk، لكن نختار Regression based on impact.

---

# 9. Release Notes

كل Release Notes يجب أن يفصل:

```text
Added
Changed
Fixed
Security
Operational
Known Notes
```

ولا يذكر Feature خلف Flag كأنها منشورة للمستخدم.

مثال:

```markdown
# WAMADAT 1.1.0

## Added
- ...

## Fixed
- ...

## Operational
- ...

## Known Notes
- ...
```

---

# 10. Rollback Decision

استخدم `technical/05` كمرجع التنفيذ.

القرار العام:

| الحالة | القرار المحتمل |
|---|---|
| Application defect + DB compatible | redeploy previous immutable images |
| Safe rapid correction | forward-fix new immutable release |
| incompatible migration/data impact | manual incident decision / restore/PITR path |
| corruption/data loss | stop writes + recovery procedure |

لا يتم `migrate:rollback` تلقائيًا كحل عام.

---

# 11. Hotfix Lane

P0/P1 Hotfix:

1. Incident/Change ID.
2. Reproduce/contain if possible.
3. Smallest safe fix.
4. Focused verification.
5. Deploy with release identity.
6. Production verify.
7. Monitor.
8. Add regression prevention/postmortem if material.

Hotfix لا يعني commit مجهول على Production.

---

# 12. Release Calendar

لا يلزم يوم نشر ثابت، لكن يفضل:

- تجنب تغييرات عالية المخاطر دون توفر الفريق للمراقبة.
- عدم دمج عدة تغييرات مستقلة عالية المخاطر في Release واحد.
- فصل migration-heavy change عن Product launch الكبير عند الحاجة.
- استخدام controlled rollout/feature flag للقدرات المناسبة.

---

# 13. Release Closure Record

بعد نافذة المراقبة:

```text
Release ID:
Status: CLOSED / ROLLED BACK / FORWARD FIX
Production Verification: PASS/FAIL
Incidents: none / references
Success Metric Early Signal:
Known Follow-ups:
Closed by:
Date:
```

---

# 14. Changelog

| التاريخ | الإصدار | التغيير |
|---|---|---|
| 2026-09-09 | 1.0 | توحيد Versioning وRelease states والـmanifest والـgo/no-go لما بعد V1. |
