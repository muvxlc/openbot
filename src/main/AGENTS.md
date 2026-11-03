# `src/main`

Electron main process owning the trust boundary, application lifecycle, windows, and privileged services (score 13, subtree root).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Dispatcher & lifecycle | `index.ts` | App events, IPC group wiring, window activate |
| Service construction | `application-services.ts` | Dependency-ordered creation and explicit teardowns |
| Window management | `main-window.ts`, `main-window-state.ts` | `BrowserWindow` creation, position debounce save |
| Protocol & security | `session-configuration.ts`, `content-security-policy.ts` | CSP, custom protocols, origin permissions |
| Remote server sync | `remote-server-manager.ts` | Team server connection, directory sync, presence |
| IPC endpoints | `ipc/` | Typed IPC handlers; see `ipc/AGENTS.md` |
| Team API routes | `team-api/` | HTTP/WS routes; see `team-api/AGENTS.md` |
| Local server child procs | `local-servers/` | `LocalServers` lifecycle and registry |

## CONVENTIONS

- Keep `index.ts` strictly as dispatcher and lifecycle junction; do not move service construction into it.
- Wire pre-ready module dependencies as call-time getters (`() => Service | null`); wire `application-services.ts` with values.
- Register service teardown beside construction with an explicit ordinal; never infer or reverse order.
- Main process may import `src/backend` and `packages/contracts`, but never `src/renderer`.
- User-facing error or status strings must use `sourceText(key, params)` from `@openbot/i18n/source`.
- Unit tests cover only the trust boundary and redaction; mock Electron with `vi.hoisted` and capture listeners.

## ANTI-PATTERNS

- Never import across the renderer/main boundary.
- Never share `FromHost` and `FromMain` decoders across different trust boundaries.
- Never implement IPC handlers or instantiate services directly inside `index.ts`.
- Never alter released HTTPS V1/V3 or WebRTC V2 wire protocol encodings.
- Never conflate `RemoteTeamDirectory` control-plane methods with host calls.

