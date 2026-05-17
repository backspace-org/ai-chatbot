# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is Vercel's Chat SDK — a Next.js 15 AI chatbot template using the AI SDK, PostgreSQL (Drizzle ORM), and Auth.js. It is a single Next.js application (not a monorepo).

### Services

| Service | Required | Notes |
|---------|----------|-------|
| Next.js App | Yes | `pnpm dev` on port 3000 (Turbopack) |
| PostgreSQL | Yes | Local instance on port 5432; user `chatuser`, password `chatpass`, database `chatdb` |
| Redis | No | Enables resumable streams; app gracefully degrades without it |
| xAI API | No | Needed for real AI chat; without it, registration/auth/UI still work |

### Key commands

See `package.json` scripts. The important ones:

- **Dev server**: `pnpm dev`
- **Lint**: `pnpm lint` (runs `next lint` + `biome lint`)
- **Format**: `pnpm format`
- **Tests**: `pnpm test` (sets `PLAYWRIGHT=True` and runs Playwright)
- **DB migrate**: `pnpm db:migrate`

### Environment

The `.env.local` file must contain at minimum `AUTH_SECRET` and `POSTGRES_URL`. See `.env.example` for all variables.

### Testing caveats

- When `PLAYWRIGHT=True` is set (either on the dev server or via the test script), the app uses **mock AI models** (`lib/ai/models.test.ts`) instead of real providers. This means Playwright tests do NOT require an `XAI_API_KEY`.
- The test script (`pnpm test`) sets `PLAYWRIGHT=True` which only affects the test runner process. The **dev server must also have `PLAYWRIGHT=True`** in its environment for mock models to activate. Playwright's config starts its own dev server with the correct env when no server is already running, or reuses an existing one.
- Route tests (`tests/routes/`) pass without Redis. Stream resumption tests (`Ada can resume chat generation`, etc.) require Redis to be available — without it, the stream endpoint returns 204 instead of the expected responses.
- Playwright requires Chromium: `pnpm exec playwright install --with-deps chromium`

### PostgreSQL setup

PostgreSQL 16 is installed locally. To start it:
```bash
sudo pg_ctlcluster 16 main start
```

The database is already migrated. If schema changes occur, run `pnpm db:migrate`.

### Non-obvious gotchas

- The app redirects unauthenticated users to `/login`. Guest mode is also available via `/api/auth/guest`.
- `next.config.ts` enables experimental PPR (Partial Pre-Rendering).
- Biome fixed 10 files on first lint run — this is expected behavior from `--write --unsafe` flag in the lint script.
