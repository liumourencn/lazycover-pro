# LazyCover V2 Product Specification

## Product thesis

LazyCover should move from "generate a good cover prompt" to "give me a topic and return covers I can publish".

The internal prompt remains important, but becomes invisible infrastructure. The user pays for useful outcomes: fast cover creation, consistent series branding, personalization, and progressively better industry-specific decisions.

## Free vs Pro

### Public Skill / Free layer

- Existing prompt-generation workflow
- Content Analyzer
- Public style demonstrations
- Useful prompt output for users who want to use external image tools
- Acquisition and education channel

### LazyCover Pro

- Direct image generation
- Private production prompt engine
- User accounts and credits
- Persistent preferences
- Generation history
- Series Style Profiles
- Private industry strategies / Prompt Packs
- Feedback learning and prompt evaluation
- Later: payment, teams, performance analytics, batch workflows

## Primary user flow

1. User enters a title, script, topic, or keywords.
2. User chooses platform; aspect ratio is inferred or selected.
3. User may upload a person/product reference image.
4. User may select an existing series profile.
5. Content Analyzer produces structured attributes.
6. The decision engine chooses a cover structure and candidate visual strategies.
7. Global industry intelligence, personal preference, and series constraints influence ranking.
8. Prompt Engine creates internal model instructions.
9. The engine produces exactly 3 intentionally distinct VisualPlans; differences must be meaningful in structure, style family/variant, or click-driver expression rather than color-only changes.
10. The image provider generates private master images for all 3 candidates.
11. When a customer logo is present, the server reserves a safe zone and composites the original logo asset into one of four supported corners.
12. The server creates separate lower-resolution preview derivatives with baked-in watermarks.
13. The user compares the 3 watermarked previews, then selects, unlocks, downloads, regenerates, changes style, or rejects outputs.
14. Those actions become feedback events for later evaluation.

## Structured analysis

At minimum preserve:

- industry
- sub_industry
- content_type
- target_audience
- user_psychology
- click_driver
- visual_anchor
- platform
- aspect_ratio
- cover_structure
- style_family
- style_variant

Avoid reducing the decision to rules such as `AI -> tech style`. For example, an AI product launch, AI tutorial, AI money-making topic, and AI controversy can require different structures despite sharing an industry.

## Intelligence layers

### Content fit

The needs of the current topic. This should have the strongest influence when there is little historical evidence.

### Global industry intelligence

Aggregated performance/feedback patterns across comparable generations. Examples: which structures or variants receive more selections for AI tutorials on Xiaohongshu.

### Personal preference

A user's persistent preferences: platforms, industries, palettes, density, layouts, preferred/disliked styles, subject placement, etc.

Personal preference must not contaminate global conclusions. A user who always selects purple tech covers does not prove that all AI creators prefer purple.

### Series identity

A named recurring content series can lock selected visual rules across episodes.

## Series Style Profiles

A series profile should support three useful consistency levels:

1. **Light lock** — preserve palette and general visual language; layout can vary.
2. **Series lock** — preserve layout zones, typography hierarchy, subject zone, badges, and key identity elements while episode content varies.
3. **Template lock** — preserve almost the entire composition; replace title, episode number, subject/product, and controlled content elements.

A profile can include:

- series name
- platform / ratio
- industry
- style family / variant
- palette
- typography rules
- layout rules
- subject rules
- decoration rules
- fixed elements
- variable elements
- reference assets
- episode numbering rules
- current prompt strategy version

The goal is recognizable continuity without making every episode identical.

## Customer logo placement

Paid delivery supports customer logos as controlled brand assets.

- Supported positions are limited to four presets: top-left, top-right, bottom-left, and bottom-right.
- Do not offer arbitrary drag/drop coordinates in the MVP.
- The decision engine should recommend the least-conflicted corner; the user may switch among the four presets.
- Reserve a platform-aware safe zone before image generation so the logo does not collide with titles, faces, products, badges, or cropping areas.
- Default safe margin is approximately 5% of canvas dimensions; default maximum logo width is approximately 12% of canvas width. Exact values remain server-controlled and may vary by platform.
- Use the original transparent PNG/SVG logo as a post-generation compositing asset. Do not ask an image model to redraw logos.
- Automatically choose an approved full-color, light, or dark logo variant when available; otherwise add a restrained contrast plate.
- Series Style Profiles may hard-lock the logo corner for consistent recurring content.
- Paid exports may include both a customer-logo master and an optional no-logo master when the product/package permits.

## Paid preview and delivery protection

- Master images and customer assets remain in private storage.
- Browsers receive only dedicated preview derivatives; never load a master image and cover it with a CSS watermark.
- Preview watermarks are baked server-side, repeated diagonally, and use restrained opacity so users can still judge design quality.
- Preview derivatives may include a non-sensitive account/order reference to discourage redistribution.
- Only an authenticated user with a valid output entitlement may receive a short-lived signed URL for an unlocked master.
- Unlocking one candidate does not automatically expose the other two unless the purchased package explicitly includes all candidates.
- Download and unlock actions are recorded as feedback/audit events.

## Feedback signals

Capture explicit and implicit product actions, including:

- selected
- downloaded
- liked
- disliked
- regenerated
- changed_style
- changed_title
- rejected
- published (when explicitly supplied)
- performance metrics such as CTR only when the user intentionally connects/provides them

Do not assume a view equals a positive signal.

## Learning loop

Production prompts should not self-modify.

Use:

`collect feedback -> segment comparable cases -> identify patterns -> propose candidate strategy -> assign version -> evaluate/A-B test -> promote winner`

Every output must be traceable to the strategy/prompt version that produced it.

## Product moat

The long-term defensibility is not a secret single prompt. It is the combination of:

- structured cover-design decision system
- industry/sub-industry strategy library
- user preference memory
- series visual identity
- generation/feedback dataset
- versioned prompt experiments and measured outcomes
- eventually, connected publishing-performance data where users explicitly opt in

## MVP scope

The first paid-testable version should focus on:

- authentication
- topic/script input
- platform/ratio
- optional person/product reference image
- optional customer logo upload and four-corner placement
- optional series selection
- direct generation of exactly 3 meaningfully distinct candidates
- server-baked watermarked previews with private master storage
- entitlement-controlled, short-lived download access
- generation history
- basic preference persistence
- feedback events
- credits ledger/basic quota
- admin-visible prompt versions

Formal payment integration can follow initial usage validation; credits can initially be granted manually.

## Non-goals for first release

- autonomous prompt self-training
- complex ML recommender
- dozens of image providers
- full creator analytics suite
- team collaboration
- mobile app
- fully automated publishing

Collect clean data first.