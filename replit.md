# YT Creator AI

An AI-powered YouTube creator platform that helps content creators grow their channels with AI-generated titles, scripts, hooks, viral scoring, analytics insights, trend forecasting, content repurposing, A/B testing, and a full creator workspace.

## Run & Operate

- `pnpm --filter @workspace/ytcreator run dev` — run the frontend (port auto-assigned)
- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string (auto-provisioned)
- Required env: `OPENAI_API_KEY` — OpenAI API key for all AI tools
- Required auth env: `SESSION_SECRET` (32+ characters), `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REDIRECT_URI`, and `APP_ORIGIN`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS + shadcn/ui + wouter + TanStack Query
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- AI: OpenRouter through the OpenAI SDK (`openai/gpt-4o-mini`) for all generation endpoints
- Authentication: Google OAuth authorization code flow, verified with `google-auth-library`
- Sessions: opaque random tokens stored as SHA-256 hashes in PostgreSQL; HttpOnly cookies
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — Single source of truth for all API contracts
- `lib/db/src/schema/` — Database schemas (users, sessions, projects, trends, notifications, activity_log)
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/api-server/src/lib/ai.ts` — OpenRouter AI helper
- `artifacts/api-server/src/lib/auth.ts` — Google OAuth config, user upsert, and DB sessions
- `artifacts/api-server/src/middlewares/authMiddleware.ts` — session loading and protected-route middleware
- `artifacts/ytcreator/src/lib/auth-context.tsx` — browser auth state and logout
- `artifacts/ytcreator/src/pages/` — Frontend pages
- `artifacts/ytcreator/src/components/layout/AppLayout.tsx` — Sidebar navigation shell

## Product

**YT Creator AI** gives YouTube creators a full AI toolkit:
- **Dashboard** — project stats, viral scores, recent activity
- **Creator Workspace** — manage projects with status, tags, bookmarks, notes
- **AI Title Generator** — 5 title variants with CTR/SEO/emotional/curiosity scores
- **AI Hook Generator** — 3s/5s/15s hooks in 9 styles (curiosity, shock, storytelling, etc.)
- **Viral Score Engine** — score any content across 6 dimensions
- **Audience Retention Predictor** — drop-off analysis + retention curve chart
- **Content Repurposer** — convert video to Shorts, blog, X thread, LinkedIn, email, etc.
- **Description Generator** — SEO-optimized with hashtags, timestamps, keywords
- **Channel Branding Studio** — names, colors, typography, logo ideas
- **Copyright Checker** — risk level, flagged issues, safe rewrites
- **Sponsor Assistant** — email templates, pitch deck, pricing tiers
- **Smart Publishing Assistant** — best days/times, hashtag recommendations
- **Trend Radar** — trending topics with growth/competition/upload window analysis
- **Notifications** — alert center for trending topics, opportunities, reminders

## Architecture decisions

- All AI features are thin wrappers around OpenAI gpt-4.1-mini with `response_format: { type: "json_object" }` — structured JSON output avoids fragile parsing
- `type: integer` fields were changed to `type: number` in the OpenAPI spec to avoid Orval v8 generating `zod.int()` which is incompatible with Zod v3
- AI routes return HTTP 503 with a clear message if `OPENAI_API_KEY` is not configured
- All AI results are ephemeral (not stored); "Save to Project" is a proposed follow-up task
- Personal project, notification, and activity records are scoped by `user_id`; legacy unowned records are assigned to the first verified Google account that signs in. Null-owner seeded trends remain shared read-only reference data.
- Google OAuth uses `GET /api/auth/google`, callback `GET /api/auth/google/callback`, session check `GET /api/auth/me`, and logout `POST /api/auth/logout`

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- After any `lib/*` change, run `pnpm run typecheck:libs` before checking artifact packages
- After OpenAPI spec changes, always run `pnpm --filter @workspace/api-spec run codegen`
- Never use `type: integer` in openapi.yaml — use `type: number` instead (Orval v8 / Zod v3 incompatibility)
- Do not use `console.log` in API server — use `req.log` in handlers or the `logger` singleton

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
