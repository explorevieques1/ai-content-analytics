# CLAUDE.md — AI Content Analytics

Operating manual for Claude working in this repo. Read this before every task.
Step-by-step build order: `docs/DEVELOPMENT_PLAN.md`. Original product brief: `docs/original-brief.md`.

## Product

A web app (SaaS) for creators who run AI-generated social media characters to make money. Users log in, create
characters, connect each character's social accounts (YouTube, Instagram, Facebook, TikTok), store the
character's identity and media, track analytics, and track revenue and expenses per character.

- **North star:** "Which of my AI characters are actually becoming successful businesses, and why?"
- **Principle:** Character first. Analytics second. Content third. The character is the central object. This is
  not a generic social dashboard with characters bolted on.
- **Data flow:** `User → Character → Platform Accounts → Content → Metric Snapshots → Revenue/Expenses → Insights`
- A user may run 1 character or 20. Every screen can be filtered to "All characters" or one character.
- App name "AI Content Analytics" is a working name. Read it from `NEXT_PUBLIC_APP_NAME` and never hard-code it.

## Stack (fixed — do not swap without asking)

| Concern            | Choice                                                                 |
| ------------------ | ---------------------------------------------------------------------- |
| Framework          | Next.js (App Router), React, TypeScript `strict`                       |
| Styling / UI       | Tailwind CSS + shadcn/ui, `next-themes` for dark/light                 |
| Auth / DB / Files  | Supabase (Auth, Postgres, Storage) with Row Level Security everywhere  |
| Charts             | Recharts                                                               |
| Validation         | Zod (forms, server actions, route handlers, API responses)             |
| Client caching     | TanStack Query only where server components aren't enough             |
| Tests              | Vitest (analytics + adapters), Playwright (smoke E2E, later)           |
| Hosting / jobs     | Vercel; Vercel Cron for scheduled metric syncs                         |
| AI                 | Anthropic Claude API, server-side only (`claude-sonnet-5` default)     |

Next.js was chosen over Vite because OAuth callbacks and platform tokens must stay server-side. Social tokens
never reach the browser.

## Commands

```bash
npm run dev            # local dev server
npm run build          # production build (must pass before commit)
npm run lint
npm run typecheck      # tsc --noEmit
npm run test           # vitest

supabase start                     # local Supabase stack (Docker)
supabase db reset                  # re-run migrations + seed locally
supabase migration new <name>      # new SQL migration
supabase gen types typescript --local > src/types/database.ts
```

## Folder structure

```
src/
  app/
    (auth)/login, signup, callback
    (dashboard)/
      page.tsx                    # portfolio overview (command center)
      characters/[slug]/          # character profile: overview, content, platforms, media, finance, bible
      content/  analytics/  finance/  media/  insights/  integrations/  settings/
    api/
      integrations/[platform]/connect|callback
      cron/sync                   # protected by CRON_SECRET
  components/ui/                  # shadcn primitives (don't hand-edit heavily)
  components/                     # shared app components (CharacterCard, CharacterSelector, KpiCard, ...)
  features/<domain>/              # characters, content, analytics, finance, media, insights, integrations
    components/  actions.ts  queries.ts  schemas.ts
  lib/
    supabase/server.ts | browser.ts | admin.ts   # admin = service role, server-only
    analytics/                    # pure metric functions + tests
    integrations/
      types.ts                    # SocialPlatformAdapter
      youtube/  meta/  tiktok/    # one folder per platform, nothing platform-specific outside it
    crypto.ts                     # token encryption helpers
  types/database.ts               # generated — never hand-edit
supabase/
  migrations/                     # all schema changes go here, never through the dashboard
  seed.sql                        # demo data (is_demo = true)
docs/
```

Separate UI, business logic, data access, analytics calculations, and integrations. No giant components.

## Data model

Every user-owned table has `user_id uuid references auth.users` (or reaches it through `character_id`), **RLS
enabled**, and owner-only policies. Money is stored as **integer cents** plus a `currency` code. Timestamps are
`timestamptz`. Index every FK.

| Table                      | Purpose / key columns                                                                                      |
| -------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `profiles`                 | 1:1 with `auth.users`; display name, avatar, plan. Created by trigger on signup                            |
| `characters`               | `user_id, name, slug (unique per user), tagline, description, niche, status (active/paused/archived), avatar_url, is_demo, profile jsonb` |
| `platform_accounts`        | `character_id, platform (youtube/instagram/facebook/tiktok/other), external_account_id, handle, status (connected/manual/error/revoked), last_synced_at` |
| `platform_tokens`          | **Server-only.** `platform_account_id, access_token_enc, refresh_token_enc, expires_at, scopes`. RLS on, no client policies; accessed only via service role |
| `content`                  | `character_id, platform_account_id, external_id, url, caption, thumbnail_url, published_at, duration_sec, status (draft/scheduled/published/archived), is_ai_disclosed, source (manual/csv/api), topic, hook, extra jsonb` |
| `content_metric_snapshots` | `content_id, captured_at, views, likes, comments, shares, saves, watch_time_sec, avg_pct_watched`. Append-only time series. Index `(content_id, captured_at desc)` |
| `account_metric_snapshots` | `platform_account_id, captured_at, followers, total_views`. Append-only                                   |
| `revenue_entries`          | `character_id, content_id?, platform?, category (platform_payout/sponsorship/affiliate/product/subscription/other), amount_cents, currency, occurred_on, note` |
| `expense_entries`          | `character_id, content_id?, category (ai_video/ai_image/ai_voice/editing/software/ads/contractor/other), amount_cents, currency, occurred_on, vendor, note` |
| `media_assets`             | `character_id, storage_path, kind (image/video/reference/3d_model/other), mime_type, size_bytes, tags text[], is_reference` |
| `insights`                 | `character_id?, kind (observed/calculated/ai_interpretation/ai_recommendation), title, body, evidence jsonb, model, created_at, dismissed_at` |
| `sync_runs`                | `platform_account_id, started_at, finished_at, status (running/success/error), items_synced, error`        |

- `characters.profile` is the extensible "character bible": personality, age/persona, interests, visual identity,
  voice, target audience, brand guidelines, common topics, prompt templates, negative prompts. Validate its shape
  with Zod. Don't create columns for these until something needs to query them.
- Nullable metrics mean "not provided by the platform", never 0.
- Storage: private bucket `character-media`, path `{user_id}/{character_id}/{uuid}.{ext}`, storage RLS checks the
  first path segment equals `auth.uid()`. Serve via signed URLs.

## Social integrations

All platforms implement one interface in `src/lib/integrations/types.ts`:

```ts
interface SocialPlatformAdapter {
  platform: Platform;
  getAuthUrl(state: string): string;
  exchangeCode(code: string): Promise<TokenSet>;
  refreshToken(refreshToken: string): Promise<TokenSet>;
  getAccount(tokens: TokenSet): Promise<ExternalAccount>;
  listContent(tokens: TokenSet, since?: Date): Promise<ExternalContent[]>;
  getContentMetrics(tokens: TokenSet, externalIds: string[]): Promise<ContentMetrics[]>;
  getAccountMetrics(tokens: TokenSet): Promise<AccountMetrics>;
  publish?(tokens: TokenSet, input: PublishInput): Promise<PublishResult>;
}
```

| Platform   | API                                                            | Status / notes                                                     |
| ---------- | -------------------------------------------------------------- | ------------------------------------------------------------------ |
| YouTube    | Google OAuth + YouTube Data API v3 + YouTube Analytics API     | **Build first.** Works in "testing" mode with up to 100 test users |
| Instagram  | Meta Graph API (Instagram API with Facebook/Instagram Login)   | Needs a Business/Creator account. Meta App Review before public use |
| Facebook   | Meta Graph API (Pages)                                         | Page insights only; shares the Meta app with Instagram             |
| TikTok     | TikTok Login Kit + Display API                                 | Sandbox first; app audit before public use; limited metrics        |

Rules:
- **Never fake an integration.** If an adapter isn't implemented, the UI shows "Not connected — manual / CSV
  entry" and `platform_accounts.status = 'manual'`.
- OAuth `state` is signed and tied to the user + character. Verify it in the callback.
- Tokens are encrypted with `TOKEN_ENCRYPTION_KEY` (AES-256-GCM in `lib/crypto.ts`) before insert and are only
  read with the admin client inside server code.
- Syncs append snapshots, log to `sync_runs`, respect rate limits/quotas, and are idempotent (upsert `content` on
  `(platform_account_id, external_id)`).
- Only show a metric in the UI if that platform's API actually provides it.

## Analytics layer

- All metrics live in `src/lib/analytics/` as **pure functions**. Components never compute metrics.
- Every metric returns `MetricResult<T> = { status: 'ok'; value: T } | { status: 'unavailable'; reason: string }`.
  Missing data results in `unavailable`, never an invented number or silent 0.
- Examples: `engagementRate`, `followerGrowth`, `retentionRate`, `revenuePerVideo`, `costPerVideo`, `profit`,
  `roi`, `rpm` (revenue per 1k views), `cpm` (cost per 1k views), `viewsPerFollower`, `postingFrequency`.
- Each function has Vitest tests, including the missing-data cases.

## AI insights

- Generated only from stored data. Never fabricate analytics.
- Every insight has a `kind`: **observed** (raw fact), **calculated** (from the analytics layer),
  **ai_interpretation**, or **ai_recommendation**, and `evidence` (the numbers/ids it's based on). The UI labels
  each kind distinctly.
- Compute rule-based insights first. Then send the computed metrics (not raw PII) to Claude server-side for
  interpretation.

## Security

- `SUPABASE_SERVICE_ROLE_KEY`, platform secrets, `ANTHROPIC_API_KEY` are server-only. Never prefix them with
  `NEXT_PUBLIC_`. Never import `lib/supabase/admin.ts` into client code (use `import 'server-only'`).
- Zod-validate every form, server action, route handler input, and external API response.
- `/api/cron/*` requires `Authorization: Bearer ${CRON_SECRET}`.
- After schema changes, run the Supabase security advisor and fix any RLS warnings.

## Environment variables

Keep `.env.example` in sync. Never commit `.env*` files except `.env.example`.

```
NEXT_PUBLIC_APP_NAME=AI Content Analytics
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
TOKEN_ENCRYPTION_KEY=          # 32-byte base64
CRON_SECRET=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
META_APP_ID=
META_APP_SECRET=
TIKTOK_CLIENT_KEY=
TIKTOK_CLIENT_SECRET=
ANTHROPIC_API_KEY=
```

## UI direction

- Should feel like "I'm managing a portfolio of digital characters", not a spreadsheet. Premium, modern, slightly
  futuristic.
- Large character imagery, premium character cards, a prominent game-like **character selector** in the top bar
  that filters the whole dashboard.
- Sidebar: Overview, Characters, Content, Analytics, Finance, Media, Insights, Integrations, Settings.
- Dark and light themes, subtle motion, strong typography, generous spacing, mobile-first responsive.
- Every data view has loading (skeleton), empty (with a clear next action), and error states.
- Demo data is visibly badged "Demo".

## Development rules

1. Build the smallest useful version first. Follow the phase order in `docs/DEVELOPMENT_PLAN.md`.
2. Inspect existing code before adding anything. Reuse components and helpers before creating new ones.
3. Schema changes go through migrations only. Regenerate `src/types/database.ts` afterwards.
4. Use TypeScript types throughout. No `any` without a comment explaining why.
5. Keep analytics, integrations, data access, and UI separate.
6. Don't fabricate data. Mock data exists only in `seed.sql` with `is_demo = true`.
7. Don't over-engineer. No features beyond the current phase unless asked.
8. Make incremental changes. Don't rewrite working features.
9. After each feature: `npm run typecheck && npm run lint && npm run test && npm run build`, then run the app and
   click through the affected flow. Fix regressions before moving on.
10. Commit per feature with clear messages. Work on a branch, not `main`.

## Future (do not build yet)

- **3D characters:** generate a stylized 3D avatar ("bitmoji-like") from a character's reference images/video via
  an image-to-3D service, store the `.glb` as a `media_assets` row with `kind = '3d_model'`, render with
  react-three-fiber. To support this later, keep reference media tagged (`is_reference`) and organized now.
- Content scheduling/publishing, prompt management, AI content ideation, predictions, team accounts, billing.
