# LazyCover V2 Roadmap

The goal is to validate a repeatable paid outcome before building a large SaaS product.

## Phase 0 — Preserve current Skill

- Keep existing public Skill functional.
- Treat it as free/open-source acquisition layer.
- Stop placing proprietary V2 experiment intelligence into the public Skill.
- Use the current Skill as source material when extracting structured design rules.

## Phase 1 — Product skeleton

Build the smallest end-to-end application.

- Next.js project
- Supabase project
- authentication
- private user profile
- basic generation page
- title/script input
- platform and aspect ratio
- optional person/product reference asset upload
- optional customer logo upload
- four-corner logo preference selector
- private storage for customer assets
- generation history
- server-side secrets

Acceptance criterion: a signed-in test user can submit a cover request, optionally attach a customer logo with one of four corner presets, and see a persisted generation record without exposing private asset URLs.

## Phase 2 — Structured Prompt Engine

- define `ContentAnalysis`
- define `VisualPlan`
- extract the current Content Analyzer rules
- extract cover structures
- extract initial style/variant rules
- introduce `prompt_strategies` versions
- implement one image-provider adapter
- generate exactly 3 meaningfully distinct candidate covers
- validate candidate diversity beyond color-only variation
- reserve one of four logo corner safe zones
- composite original customer logos after image generation
- generate lower-resolution server-baked watermarked previews
- keep master outputs private and gate downloads through output entitlements

Start with a narrow set of high-value industries rather than migrating every rule at once. Suggested first segment: AI/technology creator content because the existing Skill already contains strong rules there.

Acceptance criterion: every A/B/C candidate can be traced to structured analysis + visual plan + prompt strategy version; the browser sees only watermarked previews, and an entitled user can download only the authorized private master.

## Phase 3 — Feedback and personalization

- selected/downloaded/regenerated/disliked events
- user preference profile
- basic preference update logic
- do not feed personal preference into global stats directly
- admin/debug view showing why a strategy was selected

Acceptance criterion: returning users receive meaningfully personalized defaults while incompatible choices are still prevented by content-fit logic.

## Phase 4 — Series Style Profiles

- create/edit series
- light / series / template lock levels
- fixed vs variable visual rules
- episode number support
- reusable person/product/brand reference assets
- generate next episode from a series

Acceptance criterion: 10 generated episodes look recognizably like one series while preserving content-specific variation.

## Phase 5 — Industry learning

- aggregate comparable generation segments
- minimum sample thresholds
- strategy performance dashboard
- candidate prompt strategy creation
- controlled A/B assignment
- strategy promotion/rollback

Acceptance criterion: a new strategy can be compared to production using recorded outcomes without rewriting historical data.

## Phase 6 — Monetization

After repeat usage is validated:

- credit packages / subscription decision
- payment provider
- credit ledger integration
- output entitlement grants for one-candidate and all-candidate packages
- idempotent payment webhooks
- generation cost controls
- rate limits / abuse protection
- failed-generation refund rules

Before this phase, manually granting credits to a small test cohort is acceptable.

## Phase 7 — Stronger moat

Only after enough clean data:

- similar-case retrieval
- industry/sub-industry expansion
- stronger evaluation models
- opt-in publishing performance integrations
- CTR-aware strategy evaluation
- creator/team brand systems
- batch campaign generation

## First Codex implementation task

When development begins, ask Codex to read `AGENTS.md` and `docs/*.md`, then implement **Phase 1 only**. Do not ask it to build all phases in one pass.

Suggested instruction:

> Read AGENTS.md and the relevant docs. Implement Phase 1 of LazyCover V2 as a minimal Next.js + Supabase application. Preserve the existing public Skill. Before coding, propose the exact file plan, environment variables, database migration plan, and acceptance tests. Do not implement payments or autonomous prompt optimization yet.

## Validation questions for early testers

Track evidence for:

1. Is the generated result better/faster than writing prompts manually?
2. Do creators return to generate covers repeatedly?
3. Does series consistency solve a recurring pain point?
4. Which feedback action best predicts a genuinely useful cover?
5. Will users pay for repeated finished-cover generation rather than prompt access?

These answers should determine the next investment, not feature count.