# `packages/team-client`

<!-- Earned: score 17 (package root, 44 source files, 34 subpath exports, cross-platform boundary) -->

## OVERVIEW
Cross-platform Team client connection, WebRTC framing, RPC dispatch, and shared presentation coordination for desktop, mobile, and web.

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| WebRTC peer lifecycle & channels | `src/remote-peer.ts` | 2.1k LOC; connection state machine, ICE negotiation, data-channel multiplexing |
| Admin & API RPC requests | `src/team-admin-requests.ts`, `src/team-api-requests.ts` | Connector management, skill installs, routines, context resets |
| Host directory & pairing | `src/remote-directory.ts` | Host discovery, pairing invitations, key pinning, ticket validation |
| Wire framing protocol | `src/webrtc-framing.ts` | Team protocol v2 binary frame packing and unpacking |
| Dynamic Island & Live Activities | `src/dynamic-island-coordinator.ts`, `src/live-activity-props.ts` | macOS status item and iOS widget state reconciliation |
| Routine scheduling & calendar | `src/routine-schedule.ts`, `src/routine-calendar.ts` | Routine recurrence calculation, UTC civil date handling |
| Network recovery & retry | `src/remote-recovery.ts` | Exponential backoff coordination; never runs in background |
| Security sealing & crypto | `src/live-activity-seal.ts`, `src/push-notification-seal.ts`, `src/ed25519.ts` | HMAC/Ed25519 payload signing and verification |

## CONVENTIONS
- Cross-platform portability: Code MUST stay independent of Electron, Node, and DOM APIs (outside standard WebRTC and `fetch`). Mobile runs it directly in React Native.
- Dependency injection: Platform-specific globals passed via function arguments (e.g. `TeamClientFetch = typeof globalThis.fetch`).
- Subpath isolation: Consumers import granular subpaths (`@openbot/team-client/<module>`) rather than a single massive barrel to keep bundle sizes minimal.
- Structured concurrency: Use `effect` (`runTeamEffect`, `Effect.Effect`) for typed errors and resource lifecycle.
- Civil date arithmetic: Always use UTC day arithmetic (`routine-calendar-dates.ts`) to avoid DST skew.
- Route template sanitization: Record route templates with non-word segments replaced by `:id` and query params stripped before storing trace records.

## ANTI-PATTERNS
- Do not change the meaning of a shipped wire format. `webrtc-framing.ts` encodes Team protocol v2 frames. Put new behavior behind protocol versions in `packages/contracts/src/team-protocol`.
- Do not re-export `@openbot/contracts/team-protocol` from `index.ts`. Each module has its own subpath in `package.json`.
- Do not log tokens, tickets, or message bodies.
- Never store raw request paths, tokens, query parameters, or server error text in trace files or telemetry (`sync-profile.ts`); error codes must be fixed identifier strings.
- Never populate `reference` with raw server error text in `remote-peer.ts`; use user-copyable error codes only (e.g. `signal/host_busy`).
- Background time must never trigger network operations in `remote-recovery.ts`.
- Missed routine occurrences are collapsed, never replayed (`routine-schedule.ts`).

## COMMANDS
```bash
bun run --cwd packages/team-client typecheck     # Package typecheck
bun run typecheck:team-client                   # Root shortcut
bun run perf:remote                             # Remote sync performance E2E
bun run test:host-key-rotation                  # Host key rotation E2E
bun run test:routine-overnight-window           # Overnight routine window E2E
```

