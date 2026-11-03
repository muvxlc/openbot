<!-- Score: 15 (distinct domain: backend domain services, Effect-TS runtime, and D1 data access) -->
# `apps/auth-api/src/server`

## OVERVIEW
Core backend service layer providing Effect-TS runtime execution, D1 repository access, Stripe billing, and server orchestration.

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Request auth & DI | `request-auth.ts` | Service container (`createRequestAuthServices`), token auth |
| Effect bridge | `effect-runtime.ts` | `runApiEffect`, `runApiResponse` execution bridge |
| D1 SQL queries | `d1-auth-repository.ts` | Direct SQLite queries for `openbot-auth` D1 database |
| Remote control plane | `remote-control-plane.ts` | Machine tokens, pairing, signaling, and device sessions |
| Server provisioning | `hosted-server-service.ts` | Server lifecycle and boat container orchestration |
| Hosted sites | `hosted-site-service.ts` | Site deployments, custom domain routing, report handling |
| Billing & Stripe | `billing-service.ts`, `stripe-client.ts` | Subscriptions, checkout analytics, sales ledger |
| Skill marketplace | `skill-marketplace.ts` | Skill publishing, ZIP unpack verification, moderation |
| Worker pipeline | `worker-handler.ts` | `createWorkerHandler`, scheduled cron jobs (`handleScheduled`) |

## CONVENTIONS
- Functional Effect runtime: Business logic encapsulates in `Effect.Effect<A, E>` and executes via `runApiEffect` or `runApiResponse`.
- Request service container: Instantiate services per request via `createRequestAuthServices(env)` without global singleton mutation.
- Typed domain errors: Errors extend structured classes (`AuthServiceError`, `BillingError`, `AgentMarketplaceError`) with explicit HTTP mapping.
- Platform bindings: Access D1, KV, and R2 through Worker `env` bindings passed into service constructors.
- In-memory service assembly: Assemble repositories and clients without heavyweight DI frameworks.

## ANTI-PATTERNS
- Never expose private keys or secrets in public endpoints (e.g. JWK RFC 7518 in `remote-control-plane.ts`).
- Never leak database or gateway payload details in financial and USD conversion failures (`billing-usd.ts`).
- Never trust `X-Forwarded-For` without Cloudflare IP validation (`source-ip.ts`).
- Never unpack untrusted skill ZIPs without validating file paths against directory traversal (`skill-marketplace.ts`).
- Never route browser chat traffic through account API endpoints (`browser-api.ts`).
