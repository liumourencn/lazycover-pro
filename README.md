# LazyCover Pro

LazyCover Pro is the proprietary product layer for turning a title, script, or topic into finished social-media cover assets.

This repository is private. It contains product architecture, database design, prompt-engine design, implementation plans, and—once development starts—the paid application source code.

## Repository boundary

- Public/free Skill: [liumourencn/lazyc-cover-](https://github.com/liumourencn/lazyc-cover-)
- Private/paid product: this repository

The public Skill remains an acquisition and education layer that can produce useful prompts. Proprietary prompt strategies, ranking logic, experiment results, user data, private industry packs, and paid application code belong here.

## Confirmed paid-delivery requirements

- Every paid generation returns exactly three meaningfully different candidates.
- Preview images contain server-baked watermarks and never expose private master URLs.
- Customer logos use only four supported positions: top-left, top-right, bottom-left, and bottom-right.
- The server reserves a safe logo zone and composites the original logo asset after image generation.
- Master downloads require an authenticated output entitlement and a short-lived signed URL.

## Current status

The repository currently contains the approved V2 product documents and the Phase 1 implementation plan. Application code has not been implemented yet. The migrated specification snapshot comes from public branch `agent/lazycover-v2-product-spec` at commit `6423873`.

Read in this order:

1. `AGENTS.md`
2. `docs/PRODUCT_V2.md`
3. `docs/DATABASE.md`
4. `docs/PROMPT_ENGINE.md`
5. `docs/ROADMAP.md`
6. `docs/plans/2026-08-11-phase-1-product-skeleton-design.md`
7. `docs/plans/2026-08-11-phase-1-product-skeleton.md`

## Phase 1 boundary

Phase 1 builds the smallest secure product skeleton:

- Next.js App Router application
- Supabase Auth, Postgres, and private Storage
- authenticated profile
- cover request form
- platform and aspect ratio selection
- optional person/product reference upload
- optional customer logo upload with four-corner selection
- persisted generation record and history

Phase 1 does **not** implement image generation, three-candidate ranking, watermark rendering, master unlocks, payment, or autonomous prompt learning. Those remain later roadmap phases.

## Development handoff

Follow `docs/plans/2026-08-11-phase-1-product-skeleton.md` task by task. Keep secrets in local environment files, keep all user assets private, and verify Row Level Security before connecting a production Supabase project.
