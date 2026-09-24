# Development Plan — AI Content Analytics

The step-by-step build order. Work through phases in order. Each phase ends in a working, deployed app.
Architecture, rules and the data model live in `/CLAUDE.md`, so this file only covers **what to build, when, and
how to know it's done**.

**First test user:** you. Create your own AI character in Phase 2 and use it as the real test case the whole way
through.

---

## Platform reality check (read before Phase 7)

| Platform  | What you need                                                                                   | Approval before other users can connect                         | Metrics the API actually gives                                                                  |
| --------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| YouTube   | Google Cloud project, OAuth consent screen, YouTube Data API v3 + YouTube Analytics API enabled | Testing mode: up to 100 listed test users. Public: Google OAuth verification for YouTube scopes (can take weeks) | Views, likes, comments, shares, subscribers gained/lost, watch time, avg view duration, avg % viewed, retention curve |
| Instagram | Meta developer app. Account must be **Business or Creator** (personal accounts have no API)     | Meta App Review + business verification                          | Reach, views/plays, likes, comments, shares, saves, followers. Limited watch-time data           |
| Facebook  | Same Meta app. A Facebook **Page** (not a personal profile)                                      | Meta App Review                                                  | Page/post/video insights: views, reactions, comments, shares, followers                         |
| TikTok    | TikTok for Developers app, Login Kit + Display API                                              | Sandbox for testing, then app audit                              | Per video: views, likes, comments, shares. Per account: followers. **No retention/watch time**  |

- **Quotas:** YouTube Data API allows 10,000 units/day by default. Batch calls, and don't sync more often than
  needed.
- **AI disclosure:** YouTube, TikTok, Instagram and Facebook all require or encourage labeling realistic
  AI-generated content. The app tracks this per post (`content.is_ai_disclosed`) and should warn when it's off.
- Google, Meta and TikTok all require a public **privacy policy** and **terms** URL for review, and Meta requires
  a **data deletion** callback. These are in Phase 11, but add simple pages early if you apply for review sooner.
- Hide or mark "unavailable" any metric a platform doesn't provide. Don't show a 0 in its place.

---

## Phase 0 — Accounts & project setup

**Goal:** an empty Next.js app deployed on Vercel, connected to Supabase and GitHub.

- [ ] Install CLIs: `npm i -g supabase vercel` (or the Supabase CLI via its installer), GitHub CLI `gh`, and
      Docker (for local Supabase)
- [ ] Create the Supabase project (region near you). Save the URL, anon key and service role key
- [ ] Scaffold: `npx create-next-app@latest . --ts --tailwind --eslint --app --src-dir --import-alias "@/*"`
- [ ] `npx shadcn@latest init`, then add button, card, input, form, dialog, dropdown-menu, tabs, table, badge,
      skeleton, sonner, avatar, select, sheet
- [ ] Add dependencies: `@supabase/supabase-js @supabase/ssr zod react-hook-form @hookform/resolvers recharts
      next-themes lucide-react server-only`
- [ ] Add dev dependencies: `vitest @vitest/coverage-v8 prettier prettier-plugin-tailwindcss`. Add `typecheck` and
      `test` npm scripts
- [ ] `supabase init` and `supabase link --project-ref <ref>`
- [ ] Create `.env.example` (see CLAUDE.md) and `.env.local`
- [ ] Push to GitHub, import the repo in Vercel, and add env vars in Vercel (Production + Preview)
- [ ] Google Cloud: create the project, configure the OAuth consent screen (External, Testing, add yourself as a test
      user), and enable **YouTube Data API v3** and **YouTube Analytics API**. Create an OAuth client (Web) with
      redirect URIs for localhost and the Vercel domain

**Done when:** the Vercel URL shows the app's landing page, and `npm run build` and `npm run typecheck` pass
locally.

---

## Phase 1 — Auth & app shell

**Goal:** users can sign up, log in, and see an empty protected dashboard.

- [ ] Add `lib/supabase/server.ts`, `browser.ts` and `admin.ts` (the last with `import 'server-only'`)
- [ ] Add `middleware.ts` to refresh the Supabase session and redirect unauthenticated users away from
      `(dashboard)`
- [ ] Build `(auth)` pages: login, signup, forgot password, and the `/auth/callback` route. Email+password plus
      "Continue with Google"
- [ ] Migration `profiles` with a trigger on `auth.users` insert, and RLS
- [ ] Build the dashboard layout: sidebar (all nav items; unbuilt ones show "Coming soon"), top bar with the
      character selector placeholder, user menu, and theme toggle
- [ ] Add a landing page at `/` with a sign-in call to action

**Done when:** you can sign up, log out, log back in, and a logged-out user can't reach `/characters`.

---

## Phase 2 — Characters (the core object)

**Goal:** create and manage characters, and see them in a portfolio.

- [ ] Migration `characters` (see CLAUDE.md) with RLS, unique `(user_id, slug)`, and `is_demo`
- [ ] `features/characters/schemas.ts` with Zod schemas for the character and the `profile` "bible" jsonb
- [ ] Server actions: create, update, archive/unarchive, delete (with confirmation)
- [ ] **Portfolio page** `/characters`: grid of premium character cards (avatar, name, tagline, status, platform
      icons, and KPI slots that show "—" until data exists), plus an empty state that says "Create your first
      character"
- [ ] **Character selector** in the top bar: "All characters" plus each character with an avatar. The selection
      persists in the URL (`?c=slug`) or a cookie
- [ ] **Character profile** `/characters/[slug]` with tabs: Overview, Content, Platforms, Media, Finance, Bible.
      Only Overview and Bible have content for now
- [ ] Bible tab: form for personality, persona/age, interests, niche, target audience, visual identity, voice,
      brand guidelines, common topics, prompt templates, negative prompts
- [ ] Avatar upload (a simple version is fine; it's reused in Phase 3)

**Done when:** you've created **your own character** with a full bible and avatar, it shows on the portfolio, and
edit/archive/delete all work.

---

## Phase 3 — Media library

**Goal:** store and organize images and videos for each character.

- [ ] Create the private bucket `character-media` with storage RLS policies (path prefix = `auth.uid()`)
- [ ] Migration `media_assets`
- [ ] Upload UI with drag & drop and multiple files, a progress indicator, and client + server type/size
      validation
- [ ] Gallery grid on the character's Media tab plus a global `/media` page with a character filter, tags, the
      "reference" toggle, a lightbox preview for images, and video playback
- [ ] Serve files through signed URLs. Deleting an asset removes both the storage object and the row
- [ ] Let the user pick the character avatar from the character's media

**Done when:** you've uploaded reference images/videos for your character, tagged some as reference, and they
survive a reload and are invisible to a second test account.

---

## Phase 4 — Content & manual metrics

**Goal:** track posts per character before any API integration exists.

- [ ] Migrations `platform_accounts`, `content`, `content_metric_snapshots`, `account_metric_snapshots`
- [ ] Platforms tab: add an account manually (platform + handle), `status = 'manual'`, and enter follower counts
      over time
- [ ] Content tab and global `/content` library: list/grid view, filters (character, platform, status, date),
      and sort by any metric
- [ ] Add/edit content form: URL, caption, thumbnail, platform account, publish date, status, topic, hook,
      `is_ai_disclosed`
- [ ] "Update metrics" dialog that appends a new snapshot (never overwrites)
- [ ] CSV import: upload, map columns, preview, validate, import. Keep a template CSV for download
- [ ] `supabase/seed.sql` demo data with `is_demo = true`: 5 characters with different profiles (high views/low
      conversion, low views/high engagement, fast growth/low monetization, etc.), 3–4 platforms, 50+ posts and
      several months of snapshots. Add a "Load demo data" / "Remove demo data" option in Settings

**Done when:** your character has manually entered content with metric history, and demo data can be loaded and
removed cleanly.

---

## Phase 5 — Analytics engine & dashboards

**Goal:** the analytics make the product worth using.

- [ ] `lib/analytics/` with a `MetricResult` type and pure functions: engagement rate, follower growth (absolute
      and %), average views, retention/avg % watched, posting frequency, views per follower, plus helpers to
      group by platform and by period. Vitest tests for each, including missing data
- [ ] `features/analytics/queries.ts`: SQL/views that fetch the latest snapshot per content item and time series
      by day/week (add Postgres views or RPCs if queries get heavy)
- [ ] **Overview (command center)**: portfolio KPIs (characters, followers, views, revenue, expenses, profit),
      character performance cards, recent content, and trend charts
- [ ] **Character Overview tab**: KPI row plus charts for followers, views, engagement and revenue over time.
      Date-range picker
- [ ] **Platform comparison** for one character: views, engagement and follower growth per platform, side by side
- [ ] **Content leaderboards**: best/worst by views, engagement, retention and revenue, fastest growing, best
      topics, best hooks, best posting day/hour
- [ ] Every chart and KPI handles loading, empty and `unavailable` states

**Done when:** the dashboards tell the demo characters apart at a glance, and your own character's real numbers
show correctly.

---

## Phase 6 — Revenue & expenses

**Goal:** treat each character as a business.

- [ ] Migrations `revenue_entries`, `expense_entries` (integer cents, category enums)
- [ ] Finance tab per character plus a global `/finance` page: add/edit/delete entries, optionally linked to a
      content item, filters, and CSV import
- [ ] Add analytics functions (with tests): profit, revenue/cost/profit per video, RPM, cost per 1k views, monthly
      profit, ROI
- [ ] Charts: monthly P&L, revenue by source, expenses by category, and ROI per character
- [ ] Fill in the revenue/profit numbers on portfolio cards

**Done when:** you've logged your real tool costs (Higgsfield, voice, editing, subscriptions) and any revenue,
and profit/ROI calculate correctly.

---

## Phase 7 — YouTube integration (first real API)

**Goal:** connect a real YouTube channel to a character and sync real data automatically.

- [ ] `lib/integrations/types.ts`: the `SocialPlatformAdapter` interface and shared types
- [ ] `lib/crypto.ts`: AES-256-GCM encrypt/decrypt using `TOKEN_ENCRYPTION_KEY`, with tests
- [ ] Migrations `platform_tokens` (no client policies) and `sync_runs`
- [ ] `lib/integrations/youtube/`: OAuth URL (scopes `youtube.readonly`, `yt-analytics.readonly`, offline access),
      code exchange, token refresh, channel info, upload list, video stats (Data API), and per-video analytics
      (Analytics API: watch time, avg view duration, avg % viewed, subscribers gained). Zod-parse every response
- [ ] Routes: `/api/integrations/youtube/connect?character=<id>` (signed state) and `/callback` (verify state,
      store the encrypted tokens, create a `platform_accounts` row with `status = 'connected'`)
- [ ] Sync service: upsert content by `(platform_account_id, external_id)`, append metric snapshots, append an
      account snapshot, and log to `sync_runs`. Idempotent and quota-aware
- [ ] Add a "Sync now" button on the Platforms tab, and show last-synced time and error state
- [ ] Add `/api/cron/sync` protected by `CRON_SECRET`, with a daily schedule in `vercel.json`
- [ ] Disconnect: revoke the token, delete the tokens, set status `revoked`, and keep the historical data
- [ ] Integrations page: YouTube "Connect". Instagram/Facebook/TikTok show "Coming soon — use manual/CSV entry"

**Done when:** your character's real YouTube channel is connected, videos and metrics appear, the daily cron adds
new snapshots, and disconnecting works.

---

## Phase 8 — Instagram & Facebook (Meta)

- [ ] Create the Meta developer app with the Instagram and Facebook Login products, and add yourself as a
      tester
- [ ] Decide on the login path: *Instagram API with Instagram Login* (Business/Creator account, no Page needed)
      and/or *Facebook Login* for Pages + linked IG accounts
- [ ] `lib/integrations/meta/`: shared Graph client plus `instagram` and `facebook` adapters (long-lived token
      exchange and refresh, media list, media insights, account insights)
- [ ] Reuse the Phase 7 connect/callback/sync plumbing without changes. If it needs changes, generalize it first
- [ ] Before submitting for App Review: privacy policy, terms, data deletion callback, screencast of each
      permission's use, and business verification

**Done when:** a test IG Business/Creator account and a FB Page sync for your character.

---

## Phase 9 — TikTok

- [ ] Create the TikTok for Developers app with Login Kit + Display API, and add yourself to the sandbox
- [ ] `lib/integrations/tiktok/`: OAuth (PKCE), token refresh, user info (followers), and video list + stats
- [ ] Mark retention/watch time as `unavailable` for TikTok content in the UI
- [ ] Submit the app for audit when ready for other users

**Done when:** your TikTok account syncs in sandbox.

---

## Phase 10 — AI insights

- [ ] Migration `insights`
- [ ] Rule-based **calculated** insights from the analytics layer (e.g., retention by video length bucket,
      engagement by topic, best posting window, cost-per-video trend vs revenue-per-video trend). Each includes
      its evidence
- [ ] Claude step (server-side, `claude-sonnet-5`): send the computed metrics + character bible and receive
      structured JSON **ai_interpretation** / **ai_recommendation** items. Validate with Zod and store the
      evidence and model name
- [ ] Insights page plus an insights widget on the Overview, with distinct badges per kind and a dismiss option
- [ ] Generate on demand and weekly via cron. Cap cost per user

**Done when:** every insight shown can be traced back to stored numbers, and AI output is clearly labelled.

---

## Phase 11 — Polish & public launch

- [ ] Responsive pass (phone first), accessibility pass, and a consistent loading/empty/error state check
- [ ] Playwright smoke tests: sign up, create character, add content, view dashboards
- [ ] Public pages: landing, pricing (if charging), privacy policy, terms, and the data deletion endpoint
- [ ] Account deletion (removes all user data + storage)
- [ ] Error monitoring (e.g., Sentry) and Vercel Analytics
- [ ] Billing with Stripe if going public (plans limited by number of characters/connected accounts)
- [ ] Submit Google OAuth verification, Meta App Review and the TikTok audit

**Done when:** a stranger can sign up on the production URL and use the app end-to-end.

---

## Phase 12 — Later: character intelligence & 3D characters

- [ ] Prompt library per character built from the bible (templates, negative prompts, reference image sets) that
      can be copied into Higgsfield or other generators
- [ ] Character consistency checker (compare new media against reference media)
- [ ] **3D avatar:** choose an image-to-3D provider, send tagged reference images, store the resulting `.glb` in
      `media_assets` (`kind = '3d_model'`), and render it on the character profile with react-three-fiber (orbit,
      lighting, simple poses)
- [ ] Later: video-to-3D, rigging/animation, and exporting the avatar as a stylized "bitmoji" sticker set
- [ ] Content scheduling/publishing where APIs allow it, and AI content ideation from top performers
