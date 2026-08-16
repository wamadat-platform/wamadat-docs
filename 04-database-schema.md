# 04 — Database Schema

> **مخطّط قاعدة البيانات الكامل.** PostgreSQL 16 مع pgvector. يحتوي ~85 جدولاً موزَّعة على schemas:
> - `landlord` — مشترك بين الجميع (tenants, plans, system_users, audit logs المركزية)
> - `tenant_<slug>` — schema منفصل لكلّ مستأجر (يحتوي 80+ جدولاً)

> **معايير التسمية:**
> - أسماء الجداول: `snake_case`, **plural**, English (e.g., `users`, `program_enrollments`).
> - الـ Primary Key: `id` (BIGSERIAL) أو UUID v7 حيث الـ ordering مطلوب.
> - الـ FKs: `<table_singular>_id` (e.g., `user_id`, `program_id`).
> - Timestamps: `created_at`, `updated_at`, `deleted_at` (soft delete حيث منطقيّ).
> - JSONB للـ flexible payloads (preferences, metadata).
> - Boolean: `is_<flag>` أو `has_<flag>`.

---

## 📑 الفهرس

- [Landlord Schema](#landlord-schema)
- [Tenant Schema — Identity & Access](#tenant--identity--access)
- [Tenant Schema — Catalog](#tenant--catalog)
- [Tenant Schema — Learning](#tenant--learning)
- [Tenant Schema — Live Sessions & Attendance](#tenant--live-sessions--attendance)
- [Tenant Schema — Assessment](#tenant--assessment)
- [Tenant Schema — Certification](#tenant--certification)
- [Tenant Schema — Commerce](#tenant--commerce)
- [Tenant Schema — Billing & Subscriptions](#tenant--billing--subscriptions)
- [Tenant Schema — Affiliates](#tenant--affiliates)
- [Tenant Schema — Engagement](#tenant--engagement)
- [Tenant Schema — Communication](#tenant--communication)
- [Tenant Schema — Support / CRM](#tenant--support--crm)
- [Tenant Schema — Analytics](#tenant--analytics)
- [Tenant Schema — Marketing](#tenant--marketing)
- [Tenant Schema — Content / CMS](#tenant--content--cms)
- [Tenant Schema — AI Assistant](#tenant--ai-assistant)
- [Cross-cutting](#cross-cutting)

---

## Landlord Schema

> مشترك. يُعرَف بـ `landlord.*`. لا يحتوي بيانات مستأجر فردية.

### `tenants`
| العمود | النوع | Notes |
|---|---|---|
| `id` | UUID v7 PK | |
| `name` | VARCHAR(120) | اسم العرض |
| `slug` | VARCHAR(60) UNIQUE | للـ subdomain |
| `status` | tenant_status ENUM | provisioning/active/suspended/archived/deleted |
| `plan_id` | UUID FK plans | |
| `owner_user_id` | UUID FK system_users | |
| `db_schema_name` | VARCHAR(80) UNIQUE | `tenant_<slug>` |
| `custom_domain` | VARCHAR(255) NULL UNIQUE | Pro+ tiers |
| `data_residency` | VARCHAR(20) DEFAULT 'sa' | للـ compliance |
| `metadata` | JSONB DEFAULT '{}' | |
| `created_at`, `updated_at`, `deleted_at` | TIMESTAMPTZ | |

**Indexes**: `(slug)`, `(custom_domain)`, `(status)`.

### `plans`
| العمود | النوع | |
|---|---|---|
| `id` | UUID PK | |
| `code` | VARCHAR(40) UNIQUE | `free`/`starter`/`pro`/`business`/`enterprise` |
| `name_ar`, `name_en` | VARCHAR(120) | |
| `monthly_price_sar` | INTEGER | بالـ halalas (cents) |
| `yearly_price_sar` | INTEGER | |
| `commission_percent` | NUMERIC(5,2) | |
| `quotas` | JSONB | `{max_learners, max_courses, max_storage_gb, ...}` |
| `features` | JSONB | feature flags |
| `is_public` | BOOLEAN | enterprise قد لا يكون public |
| `sort_order` | INTEGER | |

### `tenant_subscriptions`
| العمود | النوع | |
|---|---|---|
| `id` | UUID PK | |
| `tenant_id` | UUID FK tenants | |
| `plan_id` | UUID FK plans | |
| `status` | subscription_status ENUM | |
| `billing_interval` | ENUM monthly/yearly | |
| `started_at` | TIMESTAMPTZ | |
| `current_period_start`, `current_period_end` | TIMESTAMPTZ | |
| `cancelled_at` | TIMESTAMPTZ NULL | |
| `trial_ends_at` | TIMESTAMPTZ NULL | |
| `created_at`, `updated_at` | | |

**Indexes**: `(tenant_id)`, `(status, current_period_end)`.

### `system_users`
المستخدمون الإداريّون (فريق وامضات الداخليّ — Super Admins).

| العمود | النوع | |
|---|---|---|
| `id` | UUID PK | |
| `email` | VARCHAR(255) UNIQUE | |
| `password_hash` | VARCHAR(255) | argon2id |
| `name` | VARCHAR(120) | |
| `mfa_enabled` | BOOLEAN | إلزامي = true |
| `mfa_secret_encrypted` | TEXT | |
| `last_login_at` | TIMESTAMPTZ | |

### `audit_logs_central` (immutable, append-only)
سجلّات تدقيق لإجراءات Super Admin على المستأجرين.

| العمود | النوع | |
|---|---|---|
| `id` | BIGSERIAL PK | |
| `actor_id` | UUID FK system_users | |
| `tenant_id` | UUID NULL FK tenants | |
| `action` | VARCHAR(120) | dot.notation e.g. `tenant.suspended` |
| `subject_type`, `subject_id` | VARCHAR/TEXT | |
| `changes` | JSONB | |
| `ip`, `user_agent` | TEXT | |
| `created_at` | TIMESTAMPTZ | |

**Indexes**: `(tenant_id, created_at DESC)`, `(actor_id, created_at DESC)`, `(action)`.

---

## Tenant — Identity & Access

> كلّ ما تالٍ يعيش في `tenant_<slug>` schema.

### `users`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `email` | VARCHAR(255) | unique within tenant |
| `phone_e164` | VARCHAR(20) NULL | |
| `password_hash` | VARCHAR(255) | argon2id |
| `first_name`, `last_name` | VARCHAR(80) | |
| `display_name` | VARCHAR(160) | |
| `avatar_url` | TEXT | |
| `locale` | VARCHAR(8) DEFAULT 'ar' | |
| `timezone` | VARCHAR(40) DEFAULT 'Asia/Riyadh' | |
| `email_verified_at` | TIMESTAMPTZ NULL | |
| `phone_verified_at` | TIMESTAMPTZ NULL | |
| `mfa_enabled` | BOOLEAN | |
| `mfa_secret_encrypted` | TEXT NULL | |
| `last_login_at` | TIMESTAMPTZ | |
| `marketing_opt_in` | BOOLEAN DEFAULT false | |
| `metadata` | JSONB | |
| `created_at`, `updated_at`, `deleted_at` | | |

**Indexes**: `UNIQUE(email)`, `UNIQUE(phone_e164) WHERE phone_e164 IS NOT NULL`.

### `roles`
| `id` UUID PK, `code` VARCHAR(40) UNIQUE, `name_ar`, `name_en`, `is_system` BOOLEAN |

### `permissions`
| `id` UUID PK, `code` VARCHAR(100) UNIQUE (e.g. `programs.publish`), `description` |

### `role_user`, `permission_role`, `permission_user`
جداول الـ pivot (Spatie/laravel-permission).

### `user_devices`
لإدارة الـ MFA recovery + معرفة الـ logged-in devices.

| `id`, `user_id`, `device_name`, `last_ip`, `last_seen_at`, `revoked_at`, `user_agent` |

### `password_reset_tokens`
| `email`, `token_hash`, `created_at` |

### `oauth_clients`, `oauth_access_tokens`
Sanctum/Passport tables (موجودة افتراضياً).

### `personal_access_tokens` (Sanctum)
| `id`, `tokenable_type`, `tokenable_id`, `name`, `token`, `abilities`, `last_used_at`, ... |

---

## Tenant — Catalog

### `programs`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `instructor_id` | UUID FK users | المدرّب الأساسيّ |
| `category_id` | UUID FK categories | |
| `branch_id` | UUID FK branches NULL | إذا برنامج بفرع محدّد |
| `slug` | VARCHAR(160) UNIQUE | |
| `title_ar`, `title_en` | VARCHAR(220) | |
| `subtitle_ar`, `subtitle_en` | VARCHAR(255) | |
| `description_ar`, `description_en` | TEXT | |
| `level` | ENUM beginner/intermediate/advanced | |
| `language` | VARCHAR(8) DEFAULT 'ar' | |
| `mode` | ENUM self_paced/cohort/live/hybrid/workshop | |
| `duration_minutes` | INTEGER | |
| `price_sar_halalas` | INTEGER | |
| `compare_price_halalas` | INTEGER NULL | |
| `cover_image_url`, `trailer_video_url` | TEXT | |
| `learning_outcomes` | JSONB | array of strings |
| `prerequisites` | JSONB | |
| `tags` | JSONB | array |
| `status` | ENUM draft/pending_review/published/archived | |
| `published_at`, `archived_at` | TIMESTAMPTZ NULL | |
| `seo` | JSONB | `{title, description, og_image, canonical}` |
| `created_at`, `updated_at`, `deleted_at` | | |

**Indexes**: `(status, published_at DESC)`, `(category_id)`, `(instructor_id)`, `UNIQUE(slug)`.

### `categories`
| `id`, `parent_id` FK self NULL, `slug`, `name_ar`, `name_en`, `icon`, `sort_order`, ... |

### `tags`
| `id`, `slug`, `name_ar`, `name_en`, `usage_count` |

### `program_co_instructors`
Pivot للـ multi-instructor.

### `instructor_profiles`
| `user_id` PK FK users, `bio_ar`, `bio_en`, `specializations` JSONB, `social_links` JSONB, `awards` JSONB, `years_experience`, `is_public` |

### `branches`
| `id`, `name_ar`, `name_en`, `address`, `city`, `country`, `lat`, `lng`, `phone`, `manager_user_id`, `opening_hours` JSONB, `is_active` |

---

## Tenant — Learning

### `program_modules`
| `id`, `program_id`, `title_ar/en`, `description`, `sort_order`, `is_published` |

### `lessons`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `module_id` | UUID FK program_modules | |
| `type` | ENUM video/text/pdf/audio/live/quiz/assignment | |
| `title_ar`, `title_en` | VARCHAR(220) | |
| `content` | JSONB | type-dependent payload |
| `video_provider_id` | VARCHAR(120) NULL | Bunny Stream ID |
| `duration_seconds` | INTEGER | |
| `is_preview` | BOOLEAN | free preview |
| `sort_order` | INTEGER | |
| `drip_rule` | JSONB NULL | unlock condition |

### `lesson_resources`
| `id`, `lesson_id`, `name`, `file_url`, `size_bytes`, `mime_type` |

### `enrollments`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `user_id` | FK users | |
| `program_id` | FK programs | |
| `cohort_batch_id` | FK cohort_batches NULL | |
| `enrolled_at` | TIMESTAMPTZ | |
| `expires_at` | TIMESTAMPTZ NULL | |
| `progress_percent` | NUMERIC(5,2) DEFAULT 0 | |
| `completed_at` | TIMESTAMPTZ NULL | |
| `source` | VARCHAR(40) | direct/affiliate/coupon/campaign |
| `metadata` | JSONB | |

**Indexes**: `UNIQUE(user_id, program_id)`, `(program_id, completed_at)`.

### `lesson_progress`
| `id`, `enrollment_id`, `lesson_id`, `started_at`, `last_position_seconds`, `completed_at`, `watch_time_seconds` |

**Index**: `UNIQUE(enrollment_id, lesson_id)`.

### `cohort_batches`
| `id`, `program_id`, `name`, `starts_at`, `ends_at`, `capacity`, `seats_filled`, `status` |

---

## Tenant — Live Sessions & Attendance

### `live_sessions`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `program_id` | FK programs NULL | |
| `lesson_id` | FK lessons NULL | |
| `branch_id` | FK branches NULL | للحضوريّ |
| `instructor_id` | FK users | |
| `title_ar/en` | VARCHAR(220) | |
| `mode` | ENUM online/in_person/hybrid | |
| `room_provider` | ENUM `100ms`/none | |
| `room_external_id` | VARCHAR(200) | 100ms room ID |
| `starts_at`, `ends_at` | TIMESTAMPTZ | |
| `status` | ENUM scheduled/live/ended/cancelled | |
| `recording_url` | TEXT NULL | |
| `recording_ready_at` | TIMESTAMPTZ NULL | |
| `max_capacity` | INTEGER | |

### `live_session_polls`
| `id`, `session_id`, `question`, `options` JSONB, `correct_option`, `released_at`, `closed_at` |

### `live_session_chat_messages`
| `id`, `session_id`, `user_id`, `message`, `posted_at`, `is_pinned`, `is_removed` |

### `attendance_records`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `session_id` | FK live_sessions | |
| `user_id` | FK users | |
| `status` | ENUM present/late/absent/excused | |
| `check_in_at` | TIMESTAMPTZ NULL | |
| `check_in_method` | ENUM qr_scan/manual/auto_live | |
| `checked_in_by` | FK users NULL | للـ manual |
| `notes` | TEXT NULL | |

**Indexes**: `UNIQUE(session_id, user_id)`.

### `qr_codes`
| `id`, `session_id`, `code` UNIQUE, `expires_at`, `used_count`, `max_uses` |

---

## Tenant — Assessment

### `quizzes`
| `id`, `program_id`, `lesson_id` NULL, `title_ar/en`, `description`, `passing_score_percent`, `max_attempts`, `time_limit_minutes` NULL, `randomize_questions`, `is_published` |

### `quiz_questions`
| `id`, `quiz_id`, `type` ENUM, `prompt_ar/en`, `points` NUMERIC, `correct_answer` JSONB, `explanation`, `sort_order` |

### `quiz_choices`
| `id`, `question_id`, `text_ar/en`, `is_correct`, `sort_order` |

### `quiz_attempts`
| `id`, `quiz_id`, `user_id`, `started_at`, `submitted_at` NULL, `score` NUMERIC, `passing` BOOLEAN, `attempt_number` |

### `quiz_attempt_answers`
| `id`, `attempt_id`, `question_id`, `answer` JSONB, `is_correct`, `score` |

### `assignments`
| `id`, `program_id`, `title_ar/en`, `instructions`, `due_at` NULL, `max_score`, `submission_type` ENUM (text/file/url), `rubric` JSONB |

### `assignment_submissions`
| `id`, `assignment_id`, `user_id`, `submitted_at`, `content`, `files` JSONB, `score` NULL, `feedback`, `graded_by`, `graded_at` |

---

## Tenant — Certification

### `certificate_templates`
| `id`, `name`, `background_image_url`, `fields` JSONB (positions/styles), `is_default`, `created_by` |

### `issued_certificates`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `user_id` | FK users | |
| `program_id` | FK programs | |
| `enrollment_id` | FK enrollments | |
| `certificate_number` | VARCHAR(40) UNIQUE | `WMD-2026-0000001` |
| `verification_code` | UUID UNIQUE | public |
| `template_id` | FK certificate_templates | |
| `pdf_url` | TEXT | Bunny Storage |
| `issued_at` | TIMESTAMPTZ | |
| `expires_at` | TIMESTAMPTZ NULL | |
| `revoked_at` | TIMESTAMPTZ NULL | |
| `revoked_reason` | TEXT NULL | |

### `certificate_verifications`
| `id`, `verification_code`, `verified_by_ip`, `user_agent`, `verified_at` |

---

## Tenant — Commerce

### `carts`
| `id`, `user_id` NULL, `session_id` NULL, `currency`, `created_at`, `updated_at`, `abandoned_at` NULL |

### `cart_items`
| `id`, `cart_id`, `program_id` NULL, `subscription_plan_id` NULL, `qty`, `unit_price_halalas`, `metadata` JSONB |

### `orders`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `order_number` | VARCHAR(40) UNIQUE | `ORD-2026-0000123` |
| `user_id` | FK users | |
| `cart_id` | FK carts NULL | |
| `status` | ENUM | pending/awaiting_payment/paid/processing/completed/cancelled/refunded |
| `currency` | VARCHAR(3) | SAR |
| `subtotal_halalas` | BIGINT | |
| `discount_halalas` | BIGINT | |
| `tax_halalas` | BIGINT | |
| `total_halalas` | BIGINT | |
| `applied_coupon_id` | FK coupons NULL | |
| `affiliate_partner_id` | FK affiliate_partners NULL | |
| `billing_address` | JSONB | |
| `placed_at`, `completed_at`, `cancelled_at` | TIMESTAMPTZ | |

**Indexes**: `(user_id, placed_at DESC)`, `(status)`.

### `order_items`
| `id`, `order_id`, `product_type` ENUM (program/subscription/membership), `product_id`, `qty`, `unit_price`, `subtotal`, `metadata` JSONB |

### `payments`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `order_id` | FK orders | |
| `gateway` | VARCHAR(40) | tap/tamara/mada/bank |
| `method` | VARCHAR(40) | card/apple_pay/tamara/transfer |
| `external_id` | VARCHAR(200) | gateway-side ID |
| `status` | ENUM | initiated/pending/captured/failed/refunded |
| `amount_halalas` | BIGINT | |
| `currency` | VARCHAR(3) | |
| `gateway_response` | JSONB | raw |
| `captured_at`, `failed_at` | TIMESTAMPTZ | |

**Indexes**: `(order_id)`, `(gateway, external_id)`.

### `payment_webhooks`
لـ idempotency.

| `id`, `gateway`, `external_id`, `event_type`, `raw_payload` JSONB, `processed_at`, `result` ENUM (ok/duplicate/failed) |

**Index**: `UNIQUE(gateway, external_id, event_type)`.

### `coupons`
| `id`, `code` UNIQUE, `name`, `discount_type` ENUM (percent/fixed), `discount_value`, `min_order_halalas`, `max_redemptions`, `used_count`, `applies_to` JSONB (programs/categories), `starts_at`, `expires_at`, `is_active` |

### `coupon_redemptions`
| `id`, `coupon_id`, `user_id`, `order_id`, `redeemed_at` |

### `refunds`
| `id`, `order_id`, `payment_id`, `amount_halalas`, `reason`, `requested_by`, `approved_by` NULL, `processed_at`, `gateway_refund_id`, `status` ENUM |

---

## Tenant — Billing & Subscriptions

### `subscriptions`
اشتراكات المتعلِّمين على البرامج/العضويات (داخل tenant).

| `id`, `user_id`, `subscribable_type`, `subscribable_id`, `plan_interval` ENUM, `status` ENUM, `current_period_start/end`, `cancelled_at`, `trial_ends_at`, `metadata` |

### `invoices`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `invoice_number` | VARCHAR(40) UNIQUE | per-tenant per-year sequential |
| `user_id` | FK users | |
| `order_id` | FK orders NULL | |
| `subscription_id` | FK subscriptions NULL | |
| `issued_at` | TIMESTAMPTZ | |
| `due_at` | TIMESTAMPTZ | |
| `paid_at` | TIMESTAMPTZ NULL | |
| `subtotal_halalas`, `tax_halalas`, `total_halalas` | BIGINT | |
| `tax_rate` | NUMERIC(5,4) | 0.15 = 15% |
| `pdf_url` | TEXT | |
| `zatca_uuid` | UUID NULL | |
| `zatca_status` | ENUM pending/submitted/cleared/rejected | |
| `zatca_xml_url` | TEXT NULL | |

### `invoice_items`
| `id`, `invoice_id`, `description`, `qty`, `unit_price`, `tax_amount`, `total` |

### `zatca_submissions`
| `id`, `invoice_id`, `xml_payload`, `signature`, `submitted_at`, `response_code`, `response_payload` JSONB |

---

## Tenant — Affiliates

### `affiliate_partners`
| `id`, `user_id` FK, `referral_code` UNIQUE, `commission_rate` NUMERIC(5,2), `payout_method` ENUM (bank/stc_pay/wallet), `bank_details` JSONB encrypted, `status` ENUM (pending/approved/suspended), `joined_at` |

### `affiliate_clicks`
| `id`, `referral_code`, `ip`, `user_agent`, `referrer`, `landing_url`, `clicked_at` |

**Index partition**: monthly by `clicked_at` (high volume).

### `affiliate_attributions`
| `id`, `order_id`, `partner_id`, `commission_halalas`, `attributed_at`, `paid_at` NULL |

### `affiliate_payouts`
| `id`, `partner_id`, `period_start/end`, `amount_halalas`, `status` ENUM, `processed_at`, `payment_reference` |

### `affiliate_payout_line_items`
| `id`, `payout_id`, `attribution_id` |

---

## Tenant — Engagement

### `discussions`
| `id`, `program_id` NULL, `lesson_id` NULL, `user_id`, `title`, `body`, `is_pinned`, `replies_count`, `last_reply_at`, `is_closed` |

### `discussion_replies`
| `id`, `discussion_id`, `parent_reply_id` NULL (threaded), `user_id`, `body`, `created_at`, `is_removed`, `removed_by`, `removed_reason` |

### `reactions`
| `id`, `reactable_type`, `reactable_id`, `user_id`, `type` ENUM (like/helpful/insightful/disagree), `created_at` |

**Index**: `UNIQUE(reactable_type, reactable_id, user_id, type)`.

### `reviews`
| `id`, `program_id`, `user_id`, `rating` (1-5), `body`, `is_visible`, `created_at`, `instructor_response`, `response_at` |

**Index**: `UNIQUE(program_id, user_id)`.

### `comments`
| `id`, `commentable_type`, `commentable_id`, `user_id`, `body`, `parent_comment_id` NULL, `status` ENUM (safe/flagged/removed), `flagged_count`, `created_at` |

---

## Tenant — Communication

### `notification_templates`
| `id`, `code` UNIQUE, `name`, `channels` JSONB (array of email/sms/whatsapp/push), `variables` JSONB, `translations` JSONB (per-locale subject+body), `category` ENUM, `is_active` |

### `notifications`
| العمود | النوع | |
|---|---|---|
| `id` | UUID v7 PK | |
| `user_id` | FK users | |
| `template_code` | FK notification_templates.code | |
| `channel` | ENUM email/sms/whatsapp/push/in_app | |
| `payload` | JSONB | rendered subject+body+data |
| `status` | ENUM queued/sent/delivered/failed/bounced | |
| `provider_message_id` | VARCHAR(200) NULL | |
| `sent_at`, `delivered_at`, `opened_at`, `failed_at` | TIMESTAMPTZ | |
| `error` | TEXT NULL | |

**Indexes**: `(user_id, sent_at DESC)`, `(status)`.

### `notification_clicks`
| `id`, `notification_id`, `link`, `clicked_at`, `ip` |

### `user_communication_preferences`
| `user_id` PK, `email_marketing` BOOL, `sms_marketing` BOOL, `whatsapp_marketing` BOOL, `push_enabled` BOOL, `quiet_hours_start`, `quiet_hours_end` |

---

## Tenant — Support / CRM

### `support_tickets`
| `id`, `number` UNIQUE, `user_id`, `subject`, `priority` ENUM, `status` ENUM, `assigned_agent_id` FK users NULL, `category`, `last_activity_at`, `resolved_at`, `closed_at`, `first_response_at`, `csat_score` NULL, `csat_comment` |

### `support_ticket_messages`
| `id`, `ticket_id`, `user_id`, `body`, `is_internal_note` BOOL, `attachments` JSONB, `posted_at` |

### `support_pipelines`
| `id`, `name`, `is_default` |

### `support_pipeline_stages`
| `id`, `pipeline_id`, `name`, `sort_order`, `is_won`, `is_lost` |

### `support_pipeline_cards`
| `id`, `pipeline_id`, `stage_id`, `subject`, `lead_user_id` FK users NULL, `lead_email`, `lead_phone`, `value_halalas`, `assigned_to`, `moved_to_stage_at` |

### `kb_articles`
| `id`, `slug` UNIQUE, `title_ar/en`, `body_ar/en`, `category_id`, `views_count`, `helpful_yes_count`, `helpful_no_count`, `published_at`, `updated_at` |

### `kb_categories`
| `id`, `name_ar/en`, `slug`, `icon`, `parent_id`, `sort_order` |

---

## Tenant — Analytics

### `event_logs` (append-only)
| `id` BIGSERIAL PK, `event_name` VARCHAR(120), `aggregate_type`, `aggregate_id`, `payload` JSONB, `user_id` NULL, `occurred_at` TIMESTAMPTZ, `recorded_at` |

**Partition**: monthly. **Index**: `(event_name, occurred_at DESC)`, `(aggregate_type, aggregate_id)`.

### `metric_snapshots`
| `id`, `metric_name`, `dimension_type` NULL, `dimension_id` NULL, `window_start`, `window_end`, `granularity` ENUM, `value` NUMERIC, `computed_at` |

**Index**: `UNIQUE(metric_name, dimension_type, dimension_id, window_start, granularity)`.

### `saved_reports`
| `id`, `user_id`, `name`, `query_definition` JSONB, `last_run_at`, `is_shared`, `schedule` JSONB NULL |

### `report_export_jobs`
| `id`, `report_id`, `requested_by`, `format` ENUM (csv/xlsx/pdf), `status`, `file_url`, `expires_at` |

---

## Tenant — Marketing

### `campaigns`
| `id`, `name`, `type` ENUM (email/sms/whatsapp/push), `status` ENUM, `audience_definition` JSONB, `template_code`, `scheduled_at`, `started_at`, `completed_at`, `sent_count`, `opened_count`, `clicked_count`, `converted_count`, `goal_type` ENUM |

### `campaign_variants` (A/B)
| `id`, `campaign_id`, `name`, `traffic_percent`, `template_payload` JSONB |

### `email_sequences`
| `id`, `name`, `trigger_event`, `is_active`, `audience_filter` JSONB |

### `email_sequence_steps`
| `id`, `sequence_id`, `step_order`, `delay_minutes`, `template_code`, `conditions` JSONB |

### `email_sequence_runs`
| `id`, `sequence_id`, `user_id`, `current_step`, `started_at`, `completed_at`, `cancelled_at` |

### `abandoned_cart_recoveries`
| `id`, `cart_id`, `user_id`, `value_halalas`, `attempts_made`, `last_attempt_at`, `recovered_at`, `recovery_order_id` |

### `landing_pages`
| `id`, `slug` UNIQUE, `title`, `template`, `content` JSONB, `published_at`, `views`, `conversions`, `seo` JSONB |

---

## Tenant — Content / CMS

### `posts` (blog)
| `id`, `slug` UNIQUE, `title_ar/en`, `excerpt`, `body` JSONB (rich), `cover_image_url`, `author_id`, `category_id`, `tags` JSONB, `status` ENUM, `published_at`, `seo` JSONB, `views_count` |

### `post_categories` (CMS-side)
| `id`, `name_ar/en`, `slug`, `parent_id` |

### `pages`
| `id`, `slug` UNIQUE, `title_ar/en`, `content` JSONB, `status`, `seo` JSONB |

### `podcast_episodes`
| `id`, `season`, `episode_number`, `title_ar/en`, `audio_url`, `duration_seconds`, `transcript`, `show_notes`, `published_at`, `cover_image_url`, `plays_count` |

### `faqs`
| `id`, `category` VARCHAR, `question_ar/en`, `answer_ar/en`, `sort_order`, `is_published` |

### `banners`
| `id`, `placement` ENUM (homepage_hero/sidebar/footer), `image_url`, `link_url`, `headline`, `subtitle`, `starts_at`, `ends_at`, `priority` |

---

## Tenant — AI Assistant

### `ai_conversations`
| `id`, `user_id`, `title`, `context_type` (course/general), `context_id`, `started_at`, `last_message_at`, `archived_at`, `message_count`, `total_tokens` |

### `ai_messages`
| `id`, `conversation_id`, `role` ENUM (user/assistant/tool/system), `content`, `tokens_prompt`, `tokens_completion`, `model`, `cost_usd_micro` INTEGER, `created_at` |

### `ai_recommendations`
| `user_id` PK, `recommendations` JSONB (top-N programs with scores), `computed_at`, `expires_at` |

### `ai_embeddings`
| `id`, `content_type`, `content_id`, `embedding` vector(1536), `content_hash`, `model`, `created_at` |

**Index**: `USING hnsw (embedding vector_cosine_ops)`, `(content_type, content_id) UNIQUE`.

### `ai_usage_quotas`
| `user_id` PK, `period_start`, `tokens_used`, `quota_limit`, `messages_count` |

---

## Cross-cutting

### `audit_logs` (per-tenant)
| `id` BIGSERIAL, `actor_id` FK users, `action`, `subject_type`, `subject_id`, `changes` JSONB, `ip`, `user_agent`, `created_at` |

**Index**: `(actor_id, created_at DESC)`, `(subject_type, subject_id)`.

### `media_files`
مخزّن مركزيّ لأيّ ملف.

| `id`, `disk` ENUM, `path`, `original_name`, `mime_type`, `size_bytes`, `uploaded_by`, `metadata` JSONB |

### `feature_flags`
| `key` PK, `is_enabled`, `description`, `rollout_percent`, `targets` JSONB |

### `settings`
| `key` PK, `value` JSONB, `updated_at`, `updated_by` |

### `jobs`, `failed_jobs`
جداول Laravel queue standard.

### `cache`, `sessions`
لو لم نستخدم Redis لها (نستخدم Redis، فقط للـ fallback).

---

## ملحق — Indexing Strategy

### Indexes العامّة
- كلّ FK له index (Laravel default).
- timestamps الـ "DESC" الشائعة: `(created_at DESC)`, `(completed_at DESC)`.
- Composite indexes للـ filters الشائعة: `(status, created_at)`, `(tenant scope columns, ...)`.

### Partitioning
- `event_logs` — partition by `occurred_at` monthly.
- `affiliate_clicks` — partition by `clicked_at` monthly.
- `notifications` — partition by `sent_at` monthly (في Scale phase).

### Full-text & Vector
- Meilisearch للـ full-text (مكرّر، خارج DB).
- pgvector على `ai_embeddings.embedding` بـ HNSW index.

### Materialized Views (Phase 2+)
- `mv_program_stats` — احصاءات الكورس.
- `mv_instructor_revenue` — إيراد المدرّب الشهريّ.

---

<sub>**النسخة**: 1.0 · **عدد الجداول**: ~85 · **عدد الـ ENUMs المُعرَّفة**: ~30</sub>
