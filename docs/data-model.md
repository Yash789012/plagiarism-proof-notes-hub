# Data Model Overview

## User & Identity
- `users`
  - `id` (UUID), `email`, `display_name`, `role` (student, teacher, admin), `locale`, `created_at`, `status`.
- `user_profiles`
  - `user_id`, `avatar_url`, `bio`, `institution_id`, `preferences` (JSONB for tone presets, accessibility settings).
- `auth_providers`
  - `user_id`, `provider` (password, google, apple, microsoft, sso), `provider_user_id`, `last_login_at`.
- `devices`
  - `device_id`, `user_id`, `platform`, `last_sync_at`, `push_token`, `biometric_enabled`.

## Notes & Content
- `notebooks`
  - `id`, `owner_id`, `title`, `color`, `icon`, `visibility`, `created_at`, `updated_at`.
- `notes`
  - `id`, `notebook_id`, `title`, `source_type` (scan, text, audio), `language`, `status` (draft, processing, ready), `created_at`, `updated_at`.
- `note_versions`
  - `id`, `note_id`, `version_number`, `content_delta` (Yjs binary), `summary`, `ai_tone`, `created_by`, `created_at`.
- `note_assets`
  - `id`, `note_id`, `asset_type` (scan_image, audio, export), `storage_url`, `checksum`, `watermark_hash`, `created_at`.
- `flashcards`
  - `id`, `note_id`, `front_text`, `back_text`, `difficulty`, `interval`, `next_review_at`, `created_at`.

## Collaboration & Permissions
- `workspaces`
  - `id`, `name`, `type` (classroom, study_group), `owner_id`, `created_at`.
- `workspace_members`
  - `workspace_id`, `user_id`, `role` (viewer, editor, moderator, teacher), `joined_at`.
- `workspace_notes`
  - `workspace_id`, `note_id`, `permission` (view, edit, manage), `shared_by`, `shared_at`.
- `comments`
  - `id`, `note_id`, `version_id`, `author_id`, `body`, `anchor`, `created_at`, `resolved_at`.
- `activities`
  - `id`, `workspace_id`, `entity_type`, `entity_id`, `action`, `actor_id`, `timestamp`, `metadata` (JSONB).

## AI Processing
- `processing_jobs`
  - `id`, `note_id`, `job_type` (ocr, rewrite, plagiarism, flashcard), `status`, `queued_at`, `started_at`, `completed_at`, `error`.
- `ocr_outputs`
  - `id`, `note_id`, `language`, `text`, `confidence`, `structure` (JSONB bounding boxes), `created_at`.
- `rewrite_outputs`
  - `id`, `note_version_id`, `style`, `summary_ratio`, `text`, `explanations` (JSONB), `semantic_score`, `created_at`.
- `plagiarism_reports`
  - `id`, `note_version_id`, `similarity_percentage`, `matched_sources` (JSONB), `verdict`, `generated_at`.

## Billing & Marketplace
- `plans`
  - `id`, `name`, `tier`, `price_monthly`, `price_yearly`, `features` (JSONB caps and entitlements).
- `subscriptions`
  - `id`, `user_id` or `workspace_id`, `plan_id`, `status`, `renewal_date`, `payment_provider`, `trial_end`.
- `transactions`
  - `id`, `subscription_id`, `amount`, `currency`, `provider_reference`, `timestamp`.
- `marketplace_items`
  - `id`, `creator_id`, `title`, `description`, `category`, `price`, `rating`, `metadata` (JSONB), `created_at`.
- `purchases`
  - `id`, `buyer_id`, `item_id`, `license_type`, `expires_at`, `created_at`.

## Compliance & Audit
- `audit_logs`
  - `id`, `user_id`, `action`, `entity_type`, `entity_id`, `timestamp`, `ip_address`, `user_agent`, `details` (JSONB).
- `data_residency_policies`
  - `id`, `region`, `storage_bucket`, `processing_cluster`, `effective_at`.
- `consents`
  - `id`, `user_id`, `policy_version`, `consented_at`, `revoked_at`.

