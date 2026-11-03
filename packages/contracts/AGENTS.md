# packages/contracts

## OVERVIEW
Typed IPC catalog, common desktop payload contracts, and multi-protocol wire boundaries (earned: score 15, package root and boundary).

## WHERE TO LOOK
| Path | Purpose |
| --- | --- |
| `src/ipc-endpoints.ts` | Single source of truth for all `IPC_ENDPOINTS`, groups, and request/event typing |
| `src/ipc.ts` | Shared desktop domain interfaces, entities, and event payloads |
| `src/runtime-values.ts` | Zero-dependency runtime validators (`isDynamicRecord`, `isString`, `isOneOf`) |
| `src/team-api-routes.ts` | Canonical HTTP route builder for current team host interactions |
| `src/team-protocol/` | Frozen multi-version wire protocol and codecs (see `src/team-protocol/AGENTS.md`) |
| `src/signal-protocol/` | Ingress signaling, Ed25519 queue keys, JWT tickets, webhook bridges |

## CONVENTIONS
- **One channel source**: `src/ipc-endpoints.ts` defines all channels in `IPC_ENDPOINTS`. Main and preload read `.channel` from endpoint objects, never hardcoded strings. `ipc-channel-coverage.test.ts` enforces wire uniqueness.
- **Typed endpoints**: Use `request<P, R>()` or `event<P>()`. Desktop API signatures use `Invoke<typeof IPC_ENDPOINTS.group.name>` or `Subscribe<...>`.
- **Scoped endpoints**: Use `scopedRequest<I, R>()` or `scopedQuery<R>()` for server-scoped calls; preload automatically attaches selected server context.
- **Preload bridging**: Pass-through groups use `bridgeGroup(IPC_ENDPOINTS.group, decoders)`. Custom preload logic spreads the group outside and writes only stateful methods.
- **Two mirrors**: `src/preload/index.ts` and `src/renderer/src/preview/mock-openbot.ts` mirror `IPC_ENDPOINTS`. Adding a channel requires updating endpoints, main registrar, preload, and mock together.
- **Granular subpaths**: Monorepo packages import explicit subpaths (`@openbot/contracts/ipc`, `@openbot/contracts/runtime-values`) rather than a single monolithic barrel.

## ANTI-PATTERNS
- NEVER use raw string literals for IPC channel names outside `src/ipc-endpoints.ts`.
- NEVER pass workspace paths or raw file bytes directly across renderer boundaries (e.g. `ipc-instance-transfer.ts`, `ipc-attachments.ts`).
- NEVER rename or remove existing identifier/enum members across wire boundaries (e.g. `app-language.ts`, `app-logo-color.ts`).
- NEVER serialize current desktop IPC types across external wire boundaries (`src/team-protocol` or `src/signal-protocol`).
- NEVER "fix" route duplication between `team-api-routes.ts` and `team-protocol/v1.ts`; frozen protocol artifacts keep independent copies by design.

