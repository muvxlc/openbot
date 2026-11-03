# PROJECT KNOWLEDGE BASE

**Generated:** 2026-11-03
**Commit:** 21369e1ee
**Branch:** main

## OVERVIEW
Cross-platform agent platform: Electron + SolidJS 2.0 desktop, Expo/React Native mobile, Cloudflare Worker auth/accounts, Effect-TS backend services, and local SQLite persistence.

## STRUCTURE
```
openbot/
├── apps/auth-api/       # Cloudflare Worker for accounts, auth, and D1 migrations
├── apps/mobile/         # Expo/React Native mobile app with local native modules
├── packages/contracts/  # Typed IPC endpoints and frozen wire protocols (v1-v8)
├── packages/i18n/       # 13-locale catalog runtime and sourceText fallback
├── packages/team-client/# Cross-platform WebRTC framing, RPC, and presentation
├── packages/ui/         # Shared SolidJS 2.0 / Kobalte component primitives
├── remote/api/          # Bun WebRTC signaling, tickets, TURN, and webhook ingress
├── src/backend/         # Local SQLite facade, Effect domain services, agent runtime
├── src/main/            # Electron main process, window lifecycle, IPC handlers
├── src/preload/         # Isolated context bridge and FromMain payload decoders
└── src/renderer/        # SolidJS desktop UI shell and domain feature views
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| IPC definitions | packages/contracts/src/ipc-endpoints.ts | Single source of truth for IPC channels |
| Wire protocol | packages/contracts/src/team-protocol/ | Frozen v1-v8 specs; increment version on change |
| SQLite schema & migrations | src/backend/openbot-database.ts | Dual path: LATEST_SCHEMA_SQL vs MIGRATIONS |
| Agent runtime | src/backend/agent-service.ts | Turn lifecycle, attention, and tool catalog |
| Main IPC dispatch | src/main/index.ts | Spreads domain handlers into registerIpcGroups |
| Preload bridge | src/preload/index.ts | window.openbot bridge; zero business logic |
| Shared UI primitives | packages/ui/ | SolidJS 2.0 + Kobalte; no direct app context |
| Mobile native modules | apps/mobile/modules/ | Local Expo modules (sherpa-onnx, app lock) |
| Cloudflare auth API | apps/auth-api/src/server/ | D1 repo, Effect bridge, Stripe/Boat integrations |
| Signaling & ingress | remote/api/ | Ephemeral in-memory queue; constant-time secrets |

## CODE MAP
| Symbol | Type | Location | Refs | Role |
|--------|------|----------|------|------|
| `IPC_ENDPOINTS` | const | packages/contracts/src/ipc-endpoints.ts | >50 | Master IPC catalog for main and preload |
| `OpenBotDatabase` | class | src/backend/openbot-database.ts | >30 | Local SQLite facade and migration dispatcher |
| `AgentService` | class | src/backend/agent-service.ts | >25 | Turn lifecycle and agent tool executor |
| `sourceText` | function | packages/i18n/src/source.ts | >700 | Wire/error text format with localized UI reverse |
| `runApiEffect` | function | apps/auth-api/src/server/bridge.ts | >20 | Effect-TS execution bridge for Cloudflare Workers |

## CONVENTIONS
- **Effect 4**: Use `Effect.fn`, `Effect.gen`, `Context.Service`, `Schema.TaggedError` for backend/worker services. No v3 compatibility aliases.
- **Subpath imports**: `@openbot/contracts/*`, `@openbot/ui/*`, `@openbot/i18n/*`, `@openbot/team-client/*`. Monolithic barrel imports forbidden.
- **I18n on wire**: Error objects, IPC messages, and server responses must carry canonical `sourceText(key, params)`; client UI translates via template matching.
- **SolidJS 2.0 reactivity**: Read props inside `createEffect` compute functions, never effect callbacks (`PRIMITIVE_IN_EFFECT_CALLBACK` prevention).
- **Separated decoders**: Preload `FromMain` decoders and main process `FromHost` decoders must never share implementations.
- **Communication**: ASD-STE100 Simplified Technical English for all documentation, PRs, and messages.

## ANTI-PATTERNS (THIS PROJECT)
- **Never edit or delete released migrations**: `openbot.db` schema upgrades are irreversible; v8 baseline frozen.
- **Never alter released wire protocols**: Team API v1-v8 and WebRTC v2 framing are permanent and frozen.
- **No renderer-to-main leakage**: Renderer must never import `src/main`, `src/backend`, or `src/preload`.
- **No client bundle worker bindings**: `cloudflare:workers` must never leak into client browser bundles.
- **No barrel files in feature slices**: `apps/mobile/src/features/` and `src/backend` forbid root `index.ts` barrels.
- **No manual native edits**: Continuous Native Generation prohibits manual edits in `apps/mobile/ios/` and `android/`.
- **No unit tests or story play functions**: E2E scripts only under `scripts/`; Vitest restricted to migrations, trust boundaries, secrets, and lint rules.
- **No full repository checks locally**: Do not run `bun run typecheck`, full lint, or `bun run format`.
- **No process-pattern killing**: Never `pkill -f electron` or `pkill -f bun`; use worktree dev supervisor commands.

## UNIQUE STYLES
- Pure CSS with design tokens (`primitives.css` / `@openbot/ui/styles.css`); no Tailwind in desktop renderer.
- Native chrome mandatory on mobile (`Stack`, `NativeTabs`) for iOS Liquid Glass / Android platform styling.

## COMMANDS
```bash
bun install --frozen-lockfile                   # Install dependencies
bun run typecheck:node                          # Check main, backend, preload, scripts types
bun run typecheck:renderer                      # Check renderer and ui types
bun run test:changed                            # Run tests for files changed vs origin/main
bun run test:related -- <source>                # Run tests for specific source file
biome check --write --max-diagnostics=none <path> # Format/lint changed files
bun run check:ui                                # Verify UI boundary and tokens
bun run i18n:check                              # Validate localization keys and templates
bun run test:backend -- openbot-database-schema-parity.test.ts # Schema parity test
```

## NOTES
- `openbot.db` SQLite is local source of truth; Cloudflare D1 (`openbot-auth`) handles accounts/sessions only.
- Database host process runs in Electron `utilityProcess` with SIGKILL bounds on runaway queries.
- In Cloudflare Workers, wrap global fetch: `(input, init) => fetch(input, init)` to avoid illegal invocation in workerd.
- Secret redaction is mandatory on every logging, export, and telemetry path.
- Keep PolyForm Noncommercial 1.0.0 license; do not add incompatible dependencies.

