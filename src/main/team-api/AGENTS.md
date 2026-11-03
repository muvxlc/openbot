# `src/main/team-api`

HTTP and WebSocket route handlers for OpenBot Team API services (score 10, distinct domain).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Route dispatch & lifecycle | `../team-api-server.ts` | Server class, WebSocket handling, HTTP dispatcher |
| Shared dependencies | `dependencies.ts` | Narrow `TeamApi*` service projections |
| Request parsing & auth | `request-helpers.ts` | Request decoders, capability gates, auth helpers |
| Error classification | `http-error.ts` | Canonical `HttpError` definition |
| Request context | `request-context.ts` | `TeamApiRequestContext` definitions |
| Team & session routes | `route-team.ts` | Members, invitations, sessions, presence |
| Agent routes | `route-agents.ts` | Agent collection & `/v1/agents/:agentId` regex |
| Screen & remote routes | `route-remote-screen.ts`, `route-direct.ts` | Remote screen delegation, direct calls |

## CONVENTIONS

- One file per domain exporting `route*(context, deps)` with narrow `*RouteDependencies`.
- Explicit `Promise<RouteOutcome>` return type; every route must finish with `return "unmatched"`.
- Dispatcher handles 404 and top-level errors; domain routes never return 404 directly.
- Gate order is frozen: compatibility -> remote-screen delegation -> protocol gate -> path-independent 401.
- Unsupported HTTP methods on known endpoints must return `"unmatched"` (yielding 404, not 405).
- Error messages reaching users must use `sourceText(key, params)` from `@openbot/i18n/source`.

## ANTI-PATTERNS

- Never define `HttpError` outside `http-error.ts`; check errors strictly via `instanceof HttpError`.
- Never return 405 for wrong method or custom 404 in route modules.
- Never catch unexpected exceptions inside route handlers; let them bubble to dispatcher.
- Never log passwords or session tokens in plain text.
- Never alter released wire protocol formats or reorder frozen security gates.
