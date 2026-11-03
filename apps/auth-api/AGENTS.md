<!-- Score: 12 (distinct domain: Cloudflare Worker backend for accounts, auth, hosting, and migrations) -->
# `apps/auth-api`

## OVERVIEW
Cloudflare Worker backend handling accounts, avatars, host configuration, memberships, invitations, and logical sessions; never holds chats, files, or commands.

## STRUCTURE
```
apps/auth-api/
├── migrations/      # D1 SQLite forward-compatible migrations applied pre-deploy
├── src/server/      # Effect-TS runtime, D1 repository, Stripe and Boat domain services
├── src/routes/      # TanStack Solid Router API and SSR landing endpoints
├── src/lib/         # Shared client and server metadata, catalog, and analytics
├── src/components/  # SolidJS landing and UI presentation components
└── test/            # Integration and unit tests (vitest)
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Database changes | `migrations/` | Expand-contract schema; backward compatible with active Worker |
| Backend services | `src/server/` | Effect-TS service layer, D1 queries, Stripe/Boat clients |
| API endpoints | `src/routes/` | TanStack file routes; server handlers and SSR components |
| Mobile LAN access | `dev-network-access.ts` | Route allowlist for phone testing via Vite dev server |

## CONVENTIONS
- Pre-deploy D1 migrations: CI applies migrations before Worker deployment. All DDL must remain backward compatible with running Worker code (no `NOT NULL` without default, no eager drops/renames).
- Wrap runtime callbacks: Wrap global `fetch` as `(input, init) => fetch(input, init)` in service bags; bare function changes receiver and triggers `Illegal invocation` in workerd.
- Mobile dev LAN access: Update `dev-network-access.ts` and `test/dev-network-access.test.ts` when adding mobile-accessible endpoints.
- Outbox pattern: Profile writes await durable D1 insert; Signal delivery schedules via `waitUntil` with request timeouts; failed delivery stays in retry outbox.

## ANTI-PATTERNS
- Never store chat messages, user files, or shell commands in `auth-api`.
- Never add `NOT NULL` columns without default, rename columns, or drop in-use tables in migrations without two-step release.
- Never pass unwrapped global `fetch` into service dependency bags.
- Never allow desktop-only ticket or sign-in routes through LAN allowlist.
