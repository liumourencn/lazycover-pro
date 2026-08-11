# LazyCover V2 Database Plan

Initial target: Supabase Postgres. Use UUID primary keys, timestamps, foreign keys, indexes for common filtering, and Row Level Security for user-owned tables.

This document describes logical tables; implementation migrations may refine types and constraints.

## profiles

Application profile linked to the auth user.

- `id` uuid PK, same identity as auth user where appropriate
- `display_name`
- `credits_balance` optional cached balance; ledger remains source of truth
- `created_at`
- `updated_at`

## user_preferences

Persistent personalization, not global industry truth.

- `user_id` uuid PK/FK
- `primary_industries` array/json
- `preferred_platforms` array/json
- `preferred_styles` array/json
- `disliked_styles` array/json
- `preferred_layouts` jsonb
- `preferred_palettes` jsonb
- `subject_preferences` jsonb
- `preference_json` jsonb for forward-compatible attributes
- `updated_at`

Prefer structured columns for stable/high-value dimensions and JSONB for evolving detail.

## series_profiles

- `id` uuid PK
- `user_id` uuid FK
- `name`
- `industry`
- `sub_industry`
- `platform`
- `aspect_ratio`
- `lock_level` enum: light / series / template
- `style_family`
- `style_variant`
- `palette` jsonb
- `typography_rules` jsonb
- `layout_rules` jsonb
- `subject_rules` jsonb
- `decoration_rules` jsonb
- `fixed_elements` jsonb
- `variable_elements` jsonb
- `episode_rules` jsonb
- `reference_assets` jsonb
- `logo_asset_id` nullable FK to reference_assets
- `logo_corner` nullable enum: top_left / top_right / bottom_left / bottom_right
- `prompt_strategy_id` nullable FK
- `created_at`
- `updated_at`

## generations

One user generation request / decision context.

- `id` uuid PK
- `user_id` uuid FK
- `series_id` nullable uuid FK
- `episode_no` nullable text/int
- `title`
- `content`
- `industry`
- `sub_industry`
- `platform`
- `aspect_ratio`
- `content_type`
- `target_audience`
- `user_psychology`
- `click_driver`
- `visual_anchor`
- `cover_structure`
- `style_family`
- `style_variant`
- `analysis_json` jsonb
- `decision_json` jsonb
- `prompt_strategy_id` FK
- `llm_provider`
- `image_provider`
- `logo_asset_id` nullable FK
- `logo_corner` nullable enum: top_left / top_right / bottom_left / bottom_right
- `logo_config` jsonb for server-controlled safe-margin, scale, and contrast treatment
- `status`
- `created_at`

Do not rely only on a final prompt string; retain structured decisions so experiments can be analyzed later.

## generation_outputs

One candidate image from a generation.

- `id` uuid PK
- `generation_id` uuid FK
- `master_storage_path` private original/base image
- `logo_master_storage_path` nullable private customer-logo composite
- `preview_storage_path` watermarked preview derivative
- `provider_asset_id` nullable
- `provider_metadata` jsonb
- `seed` nullable
- `model_params` jsonb
- `candidate_index` constrained to 1..3
- `visual_plan_json` jsonb
- `style_family`
- `style_variant`
- `created_at`

Avoid making public image URLs the canonical stored value. Store controlled private paths and generate authorized, short-lived URLs when needed. The client must never receive a master URL merely to display a watermarked preview.

## feedback_events

Append-only behavioral/explicit feedback.

- `id` uuid PK
- `user_id` uuid FK
- `generation_id` uuid FK
- `output_id` nullable uuid FK
- `event_type`
- `reason` nullable text
- `metadata` jsonb
- `created_at`

Initial event types:

- selected
- downloaded
- liked
- disliked
- regenerated
- changed_style
- changed_title
- rejected
- published

Prefer events over overwriting a single rating because the sequence contains useful information.

## output_entitlements

Server-side authorization for downloading an unlocked candidate.

- `id` uuid PK
- `user_id` uuid FK
- `output_id` uuid FK
- `grant_type` manual / credit / payment / package
- `credit_ledger_id` nullable FK
- `external_payment_id` nullable
- `created_at`
- `revoked_at` nullable

Unique active entitlement per (`user_id`, `output_id`). Do not trust an unlock flag supplied by the browser. Unlocking one candidate does not imply access to sibling candidates unless the package grants them explicitly.

## prompt_strategies

Versioned production strategies. Never overwrite a released strategy in place.

- `id` uuid PK
- `strategy_key` e.g. `ai.xhs.tutorial`
- `version` integer
- `status` draft / experiment / production / retired
- `industry`
- `sub_industry`
- `platform` nullable
- `content_type` nullable
- `strategy_config` jsonb
- `prompt_template` private text or encrypted/private server-managed reference
- `parent_strategy_id` nullable FK
- `created_at`
- `promoted_at` nullable

Unique: (`strategy_key`, `version`).

## industry_strategy_stats

Derived/aggregated metrics, not raw user preference.

- `strategy_id`
- `segment_key`
- `sample_size`
- `selection_rate`
- `download_rate`
- `regeneration_rate`
- `negative_rate`
- `metric_json`
- `calculated_at`

Do not overfit tiny samples. Product logic should enforce minimum evidence before historical statistics meaningfully influence ranking.

## credit_ledger

Prefer an append-only ledger over only mutating a balance.

- `id` uuid PK
- `user_id` uuid FK
- `amount` integer (positive grant/purchase, negative usage)
- `reason`
- `generation_id` nullable FK
- `external_payment_id` nullable
- `created_at`

## reference_assets

Private reusable person/product/brand references.

- `id` uuid PK
- `user_id` uuid FK
- `asset_type` person / product / logo / style_reference
- `storage_path`
- `label`
- `metadata` jsonb
- `created_at`

## Privacy / security requirements

- Enable RLS for every user-owned table.
- Never trust `user_id` from client request bodies; derive authenticated identity server-side.
- Keep production prompts and provider API keys off the client.
- Keep private reference images, logos, and master outputs in non-public storage.
- Generate preview derivatives server-side with watermarks baked into pixels; never rely on a removable CSS overlay.
- Authorize master downloads through output_entitlements and short-lived signed URLs.
- Restrict logo placement values to the four supported corner enums and calculate actual coordinates server-side.
- Define deletion/export flows before collecting sensitive creator assets at scale.
- Do not use private user data to create public examples without explicit permission.

## Analytics separation

Always distinguish:

1. raw user-owned events;
2. anonymized/aggregated industry statistics;
3. personal preference profile;
4. series-specific identity.

This prevents one user's taste from being mistaken for global performance.