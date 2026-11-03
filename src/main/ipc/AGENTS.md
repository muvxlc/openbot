# `src/main/ipc`

Typed IPC handlers exposing main and backend services to renderer windows (score 13, distinct domain).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Binding primitives | `define-ipc-group.ts` | `handler`, `payloadHandler`, `eventHandler`, `eventPayloadHandler`, `authorizedHandler` |
| Local vs remote routing | `route-to-server.ts`, `scoped-handler.ts` | `routeToServer`, `scopedHandler`, `scopedQueryHandler` |
| Input decoding & validation | `validation.ts`, `*-inputs.ts` | `isObject`, `requireString`, domain input decoders |
| Domain handler definitions | `*-handlers.ts` | Export `*IpcHandlers` returning `Pick<IpcGroupHandlers, ...>` |
| Channel definitions | `packages/contracts/src/ipc-endpoints.ts` | Contract channel declarations and payload shapes |

## CONVENTIONS

- One file per domain exporting `*IpcHandlers`. `index.ts` spreads them into `registerIpcGroups`.
- Register IPC strictly via `define-ipc-group.ts` primitives; handlers never bind channels directly.
- Decode every payload before handler use (`payloadHandler`). Sender authorization must precede decoding (`authorizedHandler`).
- For server-scoped endpoints, use `routeToServer` or `scopedHandler` rather than manual ternaries.
- Send to renderer only via `sendToRenderer` using direct `IPC_ENDPOINTS.group.name` references.
- Adding channels requires matching updates in contracts, main handlers, preload, and preview mocks.

## ANTI-PATTERNS

- Never add new `untypedRequest(...)` channels (only exception is `browserInput.sendLiveViewInput`).
- Never bind raw `unknown` as no-payload handler; never duplicate channels across groups.
- Never use `eventPayloadHandler` when sender identity must be verified.
- Never decode payloads before authorization checks succeed.
- Never hand-write duplicated local/remote server branch ternaries.

