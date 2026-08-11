# LazyCover Pro Phase 1 Product Skeleton Design

## Status

Approved from the prior product discussion and frozen for Phase 1 implementation.

## Goal

Build the smallest secure end-to-end LazyCover Pro application in which a signed-in test user can submit a cover request, optionally upload private reference assets and a customer logo, choose one of four logo corners, and view a persisted request in their own generation history.

## Scope boundary

Phase 1 proves identity, private data ownership, request persistence, and asset handling. It deliberately stops before expensive or proprietary generation work.

Included:

- email/password authentication;
- private user profile;
- title/script input;
- platform and aspect ratio;
- optional person/product reference asset;
- optional customer logo asset;
- four-corner logo selector;
- private storage paths;
- generation request persistence;
- user-owned generation history;
- server-side environment configuration;
- RLS and storage authorization tests.

Excluded:

- image-provider calls;
- prompt compilation;
- three-candidate generation;
- logo compositing;
- watermarked preview rendering;
- output entitlements;
- credits deduction and payment;
- feedback learning;
- Series Style Profiles.

The excluded behavior remains specified in the product, prompt-engine, database, and roadmap documents and must not be partially simulated in Phase 1.

## Architecture

Use a small repository with the web application under `apps/web` and Supabase configuration/migrations under `supabase`. The existing `docs` directory remains the durable product and engineering context.

The Next.js App Router application uses Server Components by default. Supabase SSR clients live in `apps/web/src/lib/supabase`; the browser client is used only where a Client Component genuinely needs it. `apps/web/src/proxy.ts` refreshes authentication cookies, while authorization is enforced again in server actions and—most importantly—by Postgres Row Level Security.

All mutations run through server actions. The browser never supplies a trusted `user_id`; the server resolves the authenticated user and writes that identity. Stable request input is validated with Zod before database or storage access.

## Data model

Phase 1 implements only the tables needed for the skeleton:

- `profiles`: one row per authenticated user;
- `reference_assets`: private person, product, or logo metadata and storage path;
- `generations`: one persisted cover request, optionally referencing person/product and logo assets.

`logo_corner` is a database enum limited to `top_left`, `top_right`, `bottom_left`, and `bottom_right`. The generation status is limited to `draft`, `submitted`, and `failed` in this phase. Later migrations may add outputs, feedback, entitlements, strategies, series, and credits without rewriting Phase 1 history.

## Storage design

Create one private bucket named `reference-assets`. Store objects under a user-prefixed path:

`<authenticated-user-id>/<asset-id>/<sanitized-filename>`

The upload action creates the asset UUID and derives the path server-side. Storage policies allow authenticated users to read, insert, update, and delete only objects whose first folder equals their user ID. The database stores only the private storage path, never a permanent public URL.

Allowed Phase 1 types:

- person/product: PNG, JPEG, or WebP;
- logo: PNG or SVG;
- maximum file size: 10 MiB.

SVG logos remain private and are not rendered as trusted inline HTML. Phase 2 will decide the sanitized rasterization/compositing path.

## Request flow

1. An unauthenticated visitor is redirected to `/login`.
2. A signed-in user opens `/generate`.
3. The form validates title, optional script, platform, ratio, asset files, and optional logo corner.
4. Optional files are uploaded to the private bucket and recorded in `reference_assets`.
5. The server inserts one `generations` row with `status = 'submitted'`.
6. The user is redirected to `/history/<generation-id>`.
7. The detail page reads the row under RLS and displays metadata only; it does not pretend that images were generated.

If an upload succeeds but the generation insert fails, the action removes newly created storage objects and asset rows on a best-effort basis and returns a generic failure message. Detailed provider/database errors stay in server logs.

## UI design

Keep Phase 1 visually credible but compact:

- a simple product shell with navigation for Generate, History, and Sign out;
- a single-column request form with clear sections;
- four explicit logo-corner radio cards rather than arbitrary coordinates;
- an upload summary showing filename/type/size, not a public asset URL;
- history cards showing title, platform, ratio, logo corner, status, and timestamp;
- empty, loading, validation, and failure states.

The four-corner selector is shown only when a logo is attached. Removing the logo clears `logo_corner` before submit.

## Security and privacy

- RLS is enabled on every user-owned table.
- Authenticated identity is derived server-side.
- Service-role credentials are never used in browser code.
- The public/publishable key is allowed in the browser; the secret/service-role key is not.
- All uploaded assets remain in a private bucket.
- Signed URLs are not needed for the Phase 1 form/history acceptance path.
- Validation rejects unsupported MIME types, oversized files, and invalid enum values.
- Tests prove that one user cannot read or mutate another user's profile, assets, or generations.

## Testing strategy

- Unit tests: Zod schemas, ratio/platform compatibility, filename sanitization, and logo-corner normalization.
- Database tests: constraints, RLS isolation, and storage path policies against local Supabase.
- Component tests: form behavior and conditional four-corner selector.
- End-to-end tests: login, submit without assets, submit with a logo, view history, and cross-user denial.
- Build checks: TypeScript, ESLint, unit tests, and production build.

## Acceptance criteria

Phase 1 is complete only when:

1. a signed-in local test user can create a persisted generation request;
2. an optional logo can be uploaded privately and associated with exactly one supported corner;
3. no browser-visible response contains a public or permanent private asset URL;
4. the user sees the new request in history;
5. a second user cannot read the first user's rows or objects;
6. all migration, unit, component, end-to-end, lint, type, and build checks pass.
