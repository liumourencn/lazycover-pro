# LazyCover — Codex Project Instructions

LazyCover is evolving from an open-source social-media cover prompt Skill into a paid AI cover-generation product.

Before making product, architecture, database, or prompt-engine changes, read the relevant files in `docs/`:

- `docs/PRODUCT_V2.md`
- `docs/DATABASE.md`
- `docs/PROMPT_ENGINE.md`
- `docs/ROADMAP.md`

## Product boundary

The existing `懒人封面/` Skill is the public/free layer. It may continue to generate prompts and demonstrate the methodology.

LazyCover Pro is the proprietary product layer. Users should primarily receive finished cover images rather than internal production prompts. Do not put proprietary V2 intelligence, experiment results, user data, ranking weights, or private industry prompt packs into the public Skill.

## Core principles

1. Treat the internal prompt as an intermediate representation, not the paid product's primary output.
2. Keep API keys, production prompts, ranking logic, and proprietary industry strategies server-side.
3. Preserve structured metadata for every generation so results are measurable and reproducible.
4. Separate global industry intelligence from individual user preferences.
5. Version production prompt strategies. Never let an AI silently overwrite a production prompt.
6. Improve prompts through measured candidates, evaluation, and controlled promotion/A-B tests.
7. Support Series Style Profiles so recurring video series can keep recognizable visual identity while allowing controlled variation.
8. Store explicit user feedback and useful behavioral signals such as selection, download, regeneration, style change, and rejection.
9. Design privacy and authorization from the start. Users must only access their own private data and assets.
10. Prefer a small MVP over a broad SaaS build. Validate repeated use and willingness to pay before adding complexity.

## Recommended initial stack

- Next.js web app
- Server-side API routes / serverless functions
- Supabase Postgres + Auth + Storage
- Pluggable LLM provider for analysis/prompt construction
- Pluggable image-generation provider

The provider interfaces should remain replaceable; do not tightly couple core LazyCover intelligence to one image model.

## Decision pipeline

`user input -> content analysis -> industry/sub-industry -> click driver -> cover structure -> industry strategy -> user preference -> optional series profile -> prompt build -> image generation -> outputs -> feedback -> evaluation`

## Development rule

When implementing a feature, preserve the distinction between:

- content fit: what this specific topic needs;
- global intelligence: what has performed well for similar industry/content cases;
- personal preference: what this specific user tends to prefer;
- series identity: what must remain visually consistent for this recurring series.

Do not collapse these into one mutable prompt.