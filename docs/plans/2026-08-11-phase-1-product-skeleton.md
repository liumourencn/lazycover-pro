# LazyCover Pro Phase 1 Product Skeleton Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a minimal Next.js + Supabase application in which an authenticated test user can securely upload optional reference assets, submit a cover request with one of four logo corners, and see the persisted request in private history.

**Architecture:** Keep the Next.js App Router application in `apps/web` and Supabase migrations/tests at the repository root. Use Supabase SSR cookie clients for authentication, server actions for mutations, private Storage for user assets, and Postgres RLS as the final authorization boundary.

**Tech Stack:** Next.js App Router with TypeScript and Tailwind CSS, React Server Components, Supabase Postgres/Auth/Storage, `@supabase/ssr`, `@supabase/supabase-js`, Zod, Vitest, Testing Library, Playwright, Supabase CLI/pgTAP.

---

## Before implementation

Read `AGENTS.md`, `docs/PRODUCT_V2.md`, `docs/DATABASE.md`, `docs/PROMPT_ENGINE.md`, `docs/ROADMAP.md`, and the companion Phase 1 design document.

Use the current official conventions:

- create the app with `create-next-app`;
- use `src/proxy.ts` rather than deprecated `middleware.ts` on Next.js 16+;
- use `@supabase/ssr` for cookie-backed SSR auth;
- manage database changes through committed Supabase migrations;
- never run destructive linked-database reset commands against production.

### Task 1: Scaffold the web application

**Files:**

- Create: `apps/web/**` through `create-next-app`
- Modify: `apps/web/package.json`
- Modify: `apps/web/src/app/page.tsx`
- Create: `apps/web/src/app/(app)/layout.tsx`

**Step 1: Verify toolchain**

Run:

```bash
node --version
npm --version
```

Expected: supported Node.js and npm versions print successfully.

**Step 2: Scaffold without touching the documentation root**

Run:

```bash
npx create-next-app@latest apps/web --ts --eslint --tailwind --app --src-dir --import-alias "@/*" --use-npm
```

Expected: `apps/web/src/app`, `apps/web/package.json`, and configuration files are created.

**Step 3: Install runtime dependencies**

Run:

```bash
npm --prefix apps/web install @supabase/supabase-js @supabase/ssr zod
```

Expected: dependencies appear in `apps/web/package.json`.

**Step 4: Install test dependencies**

Run:

```bash
npm --prefix apps/web install -D vitest @vitest/coverage-v8 jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @playwright/test
```

Expected: dev dependencies appear in `apps/web/package.json`.

**Step 5: Add scripts**

Add these scripts to `apps/web/package.json`:

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "typecheck": "tsc --noEmit"
  }
}
```

Preserve the scaffolded `dev`, `build`, `start`, and `lint` scripts.

**Step 6: Replace the starter page**

Make `/` render a small LazyCover Pro introduction with links to `/login` and `/generate`. Do not build marketing sections outside Phase 1.

**Step 7: Run baseline checks**

Run:

```bash
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
npm --prefix apps/web run build
```

Expected: all commands pass.

**Step 8: Commit**

```bash
git add apps/web
git commit -m "scaffold phase 1 web app"
```

### Task 2: Define environment and Supabase clients

**Files:**

- Create: `apps/web/.env.example`
- Create: `apps/web/src/lib/env.ts`
- Create: `apps/web/src/lib/supabase/client.ts`
- Create: `apps/web/src/lib/supabase/server.ts`
- Create: `apps/web/src/lib/supabase/proxy.ts`
- Create: `apps/web/src/proxy.ts`
- Test: `apps/web/src/lib/env.test.ts`

**Step 1: Write a failing environment test**

Test that missing required values produce a clear validation error and valid values parse successfully:

```ts
import { describe, expect, it } from "vitest";
import { parsePublicEnv } from "./env";

describe("parsePublicEnv", () => {
  it("rejects missing Supabase variables", () => {
    expect(() => parsePublicEnv({})).toThrow();
  });

  it("accepts a URL and publishable key", () => {
    expect(
      parsePublicEnv({
        NEXT_PUBLIC_SUPABASE_URL: "https://example.supabase.co",
        NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY: "test-key",
      }),
    ).toEqual({
      NEXT_PUBLIC_SUPABASE_URL: "https://example.supabase.co",
      NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY: "test-key",
    });
  });
});
```

**Step 2: Run the test and verify failure**

```bash
npm --prefix apps/web test -- src/lib/env.test.ts
```

Expected: FAIL because `parsePublicEnv` does not exist.

**Step 3: Implement environment parsing**

Use Zod with exactly these public variables:

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
```

Do not add a service-role key to `.env.example` until a server-only feature actually requires it.

**Step 4: Implement browser/server clients**

- `client.ts` uses `createBrowserClient`.
- `server.ts` uses `createServerClient` and Next.js `cookies()`.
- `lib/supabase/proxy.ts` refreshes auth cookies.
- `src/proxy.ts` exports `proxy(request)` and excludes static assets with a matcher.

On server authorization paths, use `getClaims()` or `getUser()` rather than trusting the user object from `getSession()`.

**Step 5: Run focused and baseline checks**

```bash
npm --prefix apps/web test -- src/lib/env.test.ts
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
```

Expected: PASS.

**Step 6: Commit**

```bash
git add apps/web/.env.example apps/web/src/lib apps/web/src/proxy.ts
git commit -m "add Supabase SSR clients"
```

### Task 3: Create the Phase 1 database schema and RLS

**Files:**

- Create: `supabase/config.toml`
- Create: `supabase/migrations/20260811000100_phase1_schema.sql`
- Create: `supabase/seed.sql`
- Create: `supabase/tests/database/phase1_schema.test.sql`
- Create: `supabase/tests/database/phase1_rls.test.sql`

**Step 1: Initialize Supabase locally**

Run:

```bash
npx supabase init
```

Expected: a committed `supabase/config.toml` is created.

**Step 2: Write failing pgTAP schema tests**

Cover:

- `profiles`, `reference_assets`, and `generations` exist;
- RLS is enabled on all three;
- `logo_corner` accepts only four enum values;
- generations can reference optional person/product and logo assets;
- user IDs reference `auth.users`;
- timestamps have defaults.

Example assertion:

```sql
select plan(6);
select has_table('public', 'profiles');
select has_table('public', 'reference_assets');
select has_table('public', 'generations');
select col_is_fk('public', 'generations', 'user_id');
select col_not_null('public', 'generations', 'title');
select finish();
```

**Step 3: Run and verify failure**

```bash
npx supabase start
npx supabase test db
```

Expected: FAIL because the Phase 1 tables do not exist.

**Step 4: Write the migration**

The migration must create:

```sql
create type public.asset_type as enum ('person', 'product', 'logo');
create type public.logo_corner as enum ('top_left', 'top_right', 'bottom_left', 'bottom_right');
create type public.generation_status as enum ('draft', 'submitted', 'failed');
```

`profiles`:

```sql
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  display_name text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

`reference_assets`:

```sql
create table public.reference_assets (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  asset_type public.asset_type not null,
  storage_path text not null unique,
  original_filename text not null,
  mime_type text not null,
  size_bytes bigint not null check (size_bytes > 0 and size_bytes <= 10485760),
  metadata jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now()
);
```

`generations`:

```sql
create table public.generations (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  title text not null check (char_length(title) between 1 and 160),
  content text,
  platform text not null,
  aspect_ratio text not null,
  reference_asset_id uuid references public.reference_assets(id) on delete set null,
  logo_asset_id uuid references public.reference_assets(id) on delete set null,
  logo_corner public.logo_corner,
  status public.generation_status not null default 'submitted',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  check ((logo_asset_id is null and logo_corner is null) or (logo_asset_id is not null and logo_corner is not null))
);
```

Add indexes on `reference_assets(user_id, created_at desc)` and `generations(user_id, created_at desc)`.

Create a trigger that inserts a `profiles` row after an auth user is created. Add owner-only `select`, `insert`, `update`, and `delete` RLS policies. Do not use browser-supplied `user_id` as a policy bypass.

**Step 5: Add cross-user RLS tests**

Create two fixture users. Prove user A can access A's rows and user B cannot select, update, or delete them. Also prove an invalid logo corner cannot be inserted.

**Step 6: Reset and test**

```bash
npx supabase db reset
npx supabase test db
```

Expected: all pgTAP tests pass.

**Step 7: Commit**

```bash
git add supabase
git commit -m "add phase 1 schema and RLS"
```

### Task 4: Configure private reference-asset storage

**Files:**

- Create: `supabase/migrations/20260811000200_reference_asset_storage.sql`
- Create: `supabase/tests/database/reference_asset_storage.test.sql`
- Create: `apps/web/src/lib/assets.ts`
- Test: `apps/web/src/lib/assets.test.ts`

**Step 1: Write failing filename/path tests**

```ts
import { describe, expect, it } from "vitest";
import { buildPrivateAssetPath, sanitizeFilename } from "./assets";

describe("private asset paths", () => {
  it("removes unsafe filename characters", () => {
    expect(sanitizeFilename("../我的 logo (1).PNG")).toBe("logo-1.png");
  });

  it("prefixes paths with the authenticated user and asset id", () => {
    expect(buildPrivateAssetPath("user-1", "asset-1", "logo.png"))
      .toBe("user-1/asset-1/logo.png");
  });
});
```

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- src/lib/assets.test.ts
```

Expected: FAIL because the helpers do not exist.

**Step 3: Implement deterministic helpers**

The helpers must:

- lowercase extensions;
- remove path segments and unsafe characters;
- use a fallback filename when the sanitized name is empty;
- never accept `user_id` from a form field;
- validate allowed MIME types and the 10 MiB limit.

**Step 4: Create a private bucket migration**

Insert bucket `reference-assets` with `public = false`, 10 MiB file limit, and allowed MIME types:

- `image/png`
- `image/jpeg`
- `image/webp`
- `image/svg+xml`

Add `storage.objects` policies that compare `(storage.foldername(name))[1]` to `auth.uid()::text` for owner-only access.

**Step 5: Add storage policy tests**

Prove user A can write under `A/...`, cannot write under `B/...`, and user B cannot read A's object metadata.

**Step 6: Run checks**

```bash
npx supabase db reset
npx supabase test db
npm --prefix apps/web test -- src/lib/assets.test.ts
```

Expected: PASS.

**Step 7: Commit**

```bash
git add supabase apps/web/src/lib/assets.ts apps/web/src/lib/assets.test.ts
git commit -m "secure private reference assets"
```

### Task 5: Implement authentication screens and protected shell

**Files:**

- Create: `apps/web/src/app/(auth)/login/page.tsx`
- Create: `apps/web/src/app/(auth)/signup/page.tsx`
- Create: `apps/web/src/app/(auth)/actions.ts`
- Create: `apps/web/src/app/auth/callback/route.ts`
- Create: `apps/web/src/app/(app)/layout.tsx`
- Create: `apps/web/src/components/app-nav.tsx`
- Test: `apps/web/src/app/(auth)/actions.test.ts`

**Step 1: Write failing action tests**

Mock the Supabase server client and test:

- empty/invalid email is rejected before provider calls;
- password shorter than the configured minimum is rejected;
- provider errors map to a generic form error;
- successful login redirects to `/generate`;
- sign out redirects to `/login`.

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- "src/app/(auth)/actions.test.ts"
```

Expected: FAIL because actions do not exist.

**Step 3: Implement auth actions and pages**

Use email/password for the Phase 1 test cohort. Keep provider error detail server-side. Add accessible labels and disabled/pending submit state.

**Step 4: Protect the app layout**

Resolve the current user server-side. Redirect unauthenticated visitors to `/login`. Render navigation for Generate, History, and Sign out.

**Step 5: Run tests and checks**

```bash
npm --prefix apps/web test -- "src/app/(auth)/actions.test.ts"
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
```

Expected: PASS.

**Step 6: Commit**

```bash
git add apps/web/src/app apps/web/src/components
git commit -m "add authenticated application shell"
```

### Task 6: Implement the generation request contract

**Files:**

- Create: `apps/web/src/features/generations/schema.ts`
- Create: `apps/web/src/features/generations/platforms.ts`
- Test: `apps/web/src/features/generations/schema.test.ts`

**Step 1: Write failing contract tests**

Test:

- title length 1–160;
- optional content length limit;
- allowed platforms and ratios;
- platform/ratio compatibility;
- logo corner accepts exactly four values;
- a logo requires a corner;
- no logo clears the corner;
- unsupported files and files over 10 MiB fail.

Use this stable corner list:

```ts
export const LOGO_CORNERS = [
  "top_left",
  "top_right",
  "bottom_left",
  "bottom_right",
] as const;
```

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- src/features/generations/schema.test.ts
```

Expected: FAIL.

**Step 3: Implement the Zod schema**

Define platform/ratio pairs explicitly. Do not support arbitrary logo coordinates. Normalize empty file inputs to `undefined` before validation.

**Step 4: Run and verify pass**

```bash
npm --prefix apps/web test -- src/features/generations/schema.test.ts
```

Expected: PASS.

**Step 5: Commit**

```bash
git add apps/web/src/features/generations
git commit -m "define generation request contract"
```

### Task 7: Build the generation form

**Files:**

- Create: `apps/web/src/app/(app)/generate/page.tsx`
- Create: `apps/web/src/features/generations/generation-form.tsx`
- Create: `apps/web/src/features/generations/logo-corner-picker.tsx`
- Test: `apps/web/src/features/generations/generation-form.test.tsx`

**Step 1: Write failing component tests**

Test that:

- title, content, platform, and ratio are present;
- person/product and logo inputs are separate;
- corner choices are hidden without a logo;
- attaching a logo reveals exactly four labeled choices;
- removing the logo clears the selected corner;
- invalid input renders inline errors;
- submit enters a pending/disabled state.

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- src/features/generations/generation-form.test.tsx
```

Expected: FAIL.

**Step 3: Implement the form**

Use semantic HTML and progressive enhancement around a server action. The four labels shown to users are:

- 左上
- 右上
- 左下
- 右下

Keep stored values as the four English enum identifiers.

**Step 4: Run focused tests**

```bash
npm --prefix apps/web test -- src/features/generations/generation-form.test.tsx
```

Expected: PASS.

**Step 5: Commit**

```bash
git add apps/web/src/app apps/web/src/features/generations
git commit -m "build generation request form"
```

### Task 8: Persist uploads and generation requests

**Files:**

- Create: `apps/web/src/features/generations/actions.ts`
- Create: `apps/web/src/features/generations/repository.ts`
- Test: `apps/web/src/features/generations/actions.test.ts`
- Test: `apps/web/src/features/generations/repository.test.ts`

**Step 1: Write failing server-action tests**

Cover:

- unauthenticated requests are rejected;
- authenticated identity overrides any forged form identity;
- a request without files inserts one generation;
- reference and logo files create private objects and asset rows;
- logo assets require one valid corner;
- database failure after upload triggers best-effort cleanup;
- returned errors do not include storage paths or provider details;
- success redirects to `/history/<id>`.

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- src/features/generations/actions.test.ts
```

Expected: FAIL.

**Step 3: Implement the repository boundary**

Define a small repository interface so action tests do not require real Supabase:

```ts
export interface GenerationRepository {
  uploadReference(input: UploadReferenceInput): Promise<ReferenceAsset>;
  createGeneration(input: CreateGenerationInput): Promise<{ id: string }>;
  removeReference(asset: ReferenceAsset): Promise<void>;
}
```

The production implementation derives user ID from verified auth and writes private paths only.

**Step 4: Implement the server action**

Order:

1. verify authenticated user;
2. validate form;
3. upload optional reference;
4. upload optional logo;
5. insert generation;
6. redirect;
7. on failure, remove assets created by this attempt.

**Step 5: Run tests and checks**

```bash
npm --prefix apps/web test -- src/features/generations/actions.test.ts src/features/generations/repository.test.ts
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
```

Expected: PASS.

**Step 6: Commit**

```bash
git add apps/web/src/features/generations
git commit -m "persist private generation requests"
```

### Task 9: Add generation history and detail

**Files:**

- Create: `apps/web/src/app/(app)/history/page.tsx`
- Create: `apps/web/src/app/(app)/history/[id]/page.tsx`
- Create: `apps/web/src/features/generations/queries.ts`
- Test: `apps/web/src/features/generations/queries.test.ts`

**Step 1: Write failing query tests**

Test:

- list is ordered newest first;
- detail query scopes by authenticated user and generation ID;
- missing/foreign rows return not found;
- query results expose asset metadata only, never public URLs;
- logo corner is presented as a user label.

**Step 2: Run and verify failure**

```bash
npm --prefix apps/web test -- src/features/generations/queries.test.ts
```

Expected: FAIL.

**Step 3: Implement history pages**

History cards show title, platform, ratio, status, logo presence/corner, and creation time. The detail page clearly states that Phase 1 records the request but does not yet generate output images.

**Step 4: Run tests and checks**

```bash
npm --prefix apps/web test -- src/features/generations/queries.test.ts
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
```

Expected: PASS.

**Step 5: Commit**

```bash
git add apps/web/src/app apps/web/src/features/generations
git commit -m "add private generation history"
```

### Task 10: Add end-to-end acceptance tests

**Files:**

- Create: `apps/web/playwright.config.ts`
- Create: `apps/web/e2e/auth.spec.ts`
- Create: `apps/web/e2e/generation.spec.ts`
- Create: `apps/web/e2e/rls-isolation.spec.ts`
- Create: `apps/web/e2e/fixtures/logo.png`
- Create: `apps/web/e2e/fixtures/person.jpg`

**Step 1: Write failing E2E tests**

Scenarios:

1. unauthenticated `/generate` redirects to login;
2. test user signs in and submits a request without files;
3. test user submits with a person image and logo in `bottom_right`;
4. the request appears in history;
5. the detail page shows metadata but no public/master asset URL;
6. user B cannot navigate to user A's generation detail.

**Step 2: Run and verify failure**

```bash
npx supabase start
npm --prefix apps/web run test:e2e
```

Expected: FAIL until fixtures, auth setup, and application behavior are complete.

**Step 3: Add deterministic test setup**

Create two local test users through a server-side setup helper or seed flow. Keep credentials local/test-only and do not commit production secrets.

**Step 4: Run E2E tests**

```bash
npm --prefix apps/web run test:e2e
```

Expected: PASS.

**Step 5: Commit**

```bash
git add apps/web/e2e apps/web/playwright.config.ts
git commit -m "add phase 1 acceptance tests"
```

### Task 11: Final security and documentation verification

**Files:**

- Modify: `README.md`
- Create: `docs/PHASE1_RUNBOOK.md`
- Modify: `docs/ROADMAP.md`

**Step 1: Document the local runbook**

Include:

- prerequisites;
- copying `.env.example` to `.env.local`;
- `npx supabase start`;
- local Supabase URL/key discovery;
- `npm --prefix apps/web run dev`;
- database reset and test commands;
- safe remote linking with `supabase link` and `supabase db push`;
- an explicit warning never to run `supabase db reset --linked` against production.

**Step 2: Mark Phase 1 status accurately**

Only mark Phase 1 implemented after all acceptance checks pass. Do not imply that image generation, previews, logo compositing, or paid unlocks exist.

**Step 3: Run the complete verification suite**

```bash
npx supabase db reset
npx supabase test db
npm --prefix apps/web run test
npm --prefix apps/web run test:e2e
npm --prefix apps/web run lint
npm --prefix apps/web run typecheck
npm --prefix apps/web run build
```

Expected: every command passes.

**Step 4: Inspect the built/client output for secrets**

Search the repository and build output for service-role keys, provider API keys, private storage paths, and hard-coded credentials. Expected: none are present.

**Step 5: Commit**

```bash
git add README.md docs/PHASE1_RUNBOOK.md docs/ROADMAP.md
git commit -m "document phase 1 operations"
```

## Completion gate

Do not start Phase 2 until all six design acceptance criteria pass and an early test user has completed the request flow. The first Phase 2 task should then implement structured `ContentAnalysis` and `VisualPlan` contracts before connecting any paid image provider.
