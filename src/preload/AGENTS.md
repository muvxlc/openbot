# PRELOAD KNOWLEDGE BASE

## OVERVIEW
Electron renderer-to-main trust boundary exposing validated IPC bridges on `window.openbot` (Score: 15, distinct domain).

## WHERE TO LOOK
| File | Role |
|------|------|
| `src/preload/index.ts` | Main preload entry point; binds IPC bridges and decoders to `window.openbot`. |
| `src/preload/*-decoding.ts` | Domain runtime decoders validating raw payloads arriving from main process. |
| `src/preload/team-webrtc.ts` | Isolated preload script forwarding a single `MessagePort` to hidden WebRTC window. |
| `src/preload/clipboard-files.ts` | File extraction utility handling renderer clipboard paste actions. |

## CONVENTIONS
- Pure forwarding: `index.ts` only maps `OpenBotDesktopApi` methods to IPC channels; add zero business logic here.
- Boundary decoders: decode all main-to-renderer payloads in domain `*-decoding.ts` before renderer consumption.
- Domain separation: keep preload `FromMain` decoders strictly isolated from main-process `FromHost` decoders.
- Group bridging: pass whole groups via `bridgeGroup(IPC_ENDPOINTS.group, decoders)` without explicit type args to enforce endpoint checks.
- Payload limits: bridged methods forward at most one payload; scoped endpoints send `AgentIpcRequest` with target server.
- Hand-written endpoints: only `attachmentImports` and untyped `browserInput` are manually wired; use `listen` helper for events.
- Synchronization: update IPC channel, handler, preview mock, and preload method in the same change.
- Verification: bundle is checked against `IPC_ENDPOINTS` by `scripts/verify-preload-bundle.ts`.

## ANTI-PATTERNS
- Never expose `ipcRenderer`, Electron objects, Node APIs, or raw credentials to the page context.
- Do not define business logic or decoders inside `index.ts`.
- Do not mix preload `FromMain` decoders with main-process `FromHost` decoders.
- Do not supply explicit type arguments to `bridgeGroup(...)`.
- Do not invoke single endpoints directly if their group is bridged.
- Do not pass bridged methods by reference into DOM event listeners (e.g. `onClick={api.update.check}`).
- Do not call `ipcRenderer.on` directly; always use the `listen(...)` helper.
- In `team-webrtc.ts`, never expose Node or Electron APIs; pass only the single `MessagePort`.

## COMMANDS
```bash
bun run test:desktop -- src/preload/<name>.test.ts   # Run single preload test
bun run test:desktop -- src/main/ipc-channel-coverage.test.ts # Verify channel coverage
bun run verify:preload                               # Verify preload bundle contracts
```

