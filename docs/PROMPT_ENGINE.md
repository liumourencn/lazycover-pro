# LazyCover V2 Prompt Engine

## Goal

Turn the current large natural-language Skill into a controlled decision and rendering pipeline without losing its useful design knowledge.

The Prompt Engine is proprietary server-side product logic. The public Skill can remain a useful free prompt generator, but should not become the storage location for private experiment results or production strategy data.

## Pipeline

1. Normalize input.
2. Analyze content into structured fields.
3. Classify industry and sub-industry.
4. Select click driver and cover structure.
5. Retrieve compatible industry strategies.
6. Apply evidence from global strategy performance when sample size is sufficient.
7. Apply user preference as personalization, not as global truth.
8. Apply Series Style Profile constraints when present.
9. Rank exactly 3 meaningfully distinct candidate visual plans.
10. When a customer logo exists, reserve one of four supported corner safe zones in each plan.
11. Compile each plan into an internal image-model prompt.
12. Generate private base images through a provider adapter.
13. Composite the original customer logo server-side; do not redraw it with the image model.
14. Create separate lower-resolution preview derivatives with server-baked watermarks.
15. Persist decisions, strategy version, master/preview paths, entitlements, and feedback.

## Suggested internal objects

### ContentAnalysis

```json
{
  "industry": "ai",
  "sub_industry": "tools",
  "content_type": "tutorial",
  "target_audience": "creators who want to automate repetitive work",
  "user_psychology": ["curiosity", "efficiency"],
  "click_driver": ["numeric_impact", "benefit"],
  "visual_anchor": "5个"
}
```

### VisualPlan

```json
{
  "cover_structure": "data_analysis",
  "style_family": "tech",
  "style_variant": "glassmorphism",
  "layout": {},
  "palette": {},
  "typography": {},
  "subject": {},
  "decorations": {},
  "fixed_series_constraints": {},
  "brand_logo": {
    "asset_id": null,
    "corner": "top_right",
    "safe_margin_ratio": 0.05,
    "max_width_ratio": 0.12,
    "contrast_treatment": "auto"
  },
  "reason_codes": []
}
```

Reason codes are useful for debugging and evaluation. The system should be able to explain internally why a plan was selected without exposing proprietary prompts to end users.

## Strategy selection

Do not use one universal prompt. Maintain strategy families such as:

- `ai.news`
- `ai.tools.tutorial`
- `ai.product_launch`
- `ai.money`
- `ai.comparison`
- `finance.company_analysis`
- `beauty.review`
- `career.growth`

A strategy defines compatible structures, style priors, layout constraints, visual emphasis, typography rules, negative constraints, and model-specific compilation hints.

## Ranking factors

Start simple and deterministic. Conceptually:

- content fit: strongest baseline signal
- industry historical evidence: increases with sufficient samples
- personal preference: increases after enough user history
- series constraints: hard or soft constraints depending on lock level

Do not hard-code an assumed final weighting as product truth. Log component scores and tune from evidence.

For a brand-new user, content fit and industry defaults dominate. As personal history grows, personalization can have more influence without overriding obviously incompatible content requirements.

## Three-candidate diversity

Every paid generation returns exactly three candidates. They must represent distinct viable decisions, not cosmetic recolors.

- Prefer differences in cover structure, style family/variant, composition, evidence presentation, or click-driver expression.
- Preserve the same factual title/content requirements across all candidates.
- Log diversity reason codes so the system can explain why A/B/C differ.
- Reject or repair a candidate set when two plans are substantially equivalent.
- Candidate indices remain stable as A/B/C for preview, feedback, entitlement, and evaluation.

## Customer logo handling

- Logo position is limited to top-left, top-right, bottom-left, or bottom-right.
- The engine recommends the least-conflicted corner based on title, face, product, badge, and platform crop zones.
- The user may switch among the four presets; arbitrary coordinates are not supported in the MVP.
- Reserve the logo safe zone in VisualPlan before prompt compilation.
- Treat logo corner, margin, scale ceiling, and series lock as structured fields.
- Generate the scene with intentional negative space in the reserved zone.
- Composite the original transparent logo asset after image generation.
- Use approved light/dark/full-color variants or a restrained contrast plate rather than allowing the image model to reinterpret the logo.
- Under Series or Template lock, logo corner may be a hard constraint.

## Series handling

Series constraints should be explicit.

### Hard constraints examples

- aspect ratio
- series badge position
- logo placement
- title zone
- fixed palette under template lock
- customer logo corner and safe zone when locked

### Soft constraints examples

- preferred subject position
- background treatment
- decorative motif
- lighting style

Generate controlled novelty by varying episode-specific subject matter, visual anchor, supporting assets, expression/pose, and selected secondary composition elements.

## Prompt compilation

Keep model-neutral VisualPlan separate from provider-specific prompt syntax.

Suggested interface:

```ts
interface ImageProvider {
  generate(plan: VisualPlan, assets: ReferenceAsset[]): Promise<GeneratedAsset[]>;
}

interface DeliveryRenderer {
  compositeCustomerLogo(asset: GeneratedAsset, logo: ReferenceAsset, rules: LogoRules): Promise<GeneratedAsset>;
  createWatermarkedPreview(asset: GeneratedAsset, watermark: WatermarkRules): Promise<GeneratedAsset>;
}
```

Provider adapters may compile the plan differently for different image models. This prevents the core decision engine from becoming dependent on one provider.

## Preview security and unlock delivery

- Keep provider originals and logo-composited masters in private storage.
- Create preview derivatives on the server and bake a repeated diagonal watermark into image pixels.
- Never send an unwatermarked master to the browser and rely on CSS/DOM overlays for protection.
- Preview resolution should be sufficient for design evaluation but not substitute for the paid master.
- Resolve output entitlement server-side before issuing a short-lived signed master URL.
- Record selected, unlocked, and downloaded events separately.
- Access to candidate A must not expose B/C unless the user's package includes all three.

## Versioning

Every production strategy is immutable once used.

Example:

- `ai.xhs.tutorial:v1`
- `ai.xhs.tutorial:v2`
- `ai.xhs.tutorial:v3`

A generation records exactly which strategy version was used.

New improvements create a candidate version. They do not mutate V1/V2 history.

## Evaluation

Useful early metrics include:

- candidate selection rate
- download rate
- regeneration rate
- style-change rate
- explicit dislike/rejection rate

Later, with user opt-in, include publishing outcomes such as CTR. Treat external performance metrics as stronger evidence than a simple download, but normalize for context before drawing conclusions.

## Improvement loop

AI can help analyze failures and propose a new strategy version, but promotion must be controlled.

Recommended flow:

1. Select a well-defined segment.
2. Compare strong vs weak outputs and their metadata.
3. Ask an evaluator/analyst to identify hypotheses.
4. Create a new candidate strategy version.
5. Test it against current production strategy.
6. Promote only with sufficient evidence.
7. Preserve rollback ability.

## Retrieval later

Once enough data exists, retrieval can surface similar high-performing historical cases by structured filters and/or embeddings. Do not start by building a complex vector recommender. First ensure generation metadata and feedback are clean enough to trust.