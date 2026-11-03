# `remote/api`

## OVERVIEW
Signal service relaying WebRTC signalling, remote tickets, TURN credentials, and webhook ingress (Slack, Discord, Telegram, GitHub). Earned AGENTS.md: score 16 (distinct domain & service boundary).

## WHERE TO LOOK
| Component | Location | Role |
|-----------|----------|------|
| Server bootstrap | `src/index.ts` | Process entry, Effect runtimes, Bun HTTP/metrics listeners |
| Webhook & HTTP routes | `src/app.ts` | Elysia app, signature checks, event forwarding |
| Signal session state | `src/signal-service.ts` | WebRTC signaling relays, route auth, socket lifetimes |
| Tokens & secrets | `src/tokens.ts` | Constant-time secret checks, token signing, Telegram grants |
| Protocol definitions | `src/protocol.ts` | Server validation re-exporting `@openbot/contracts/signal-protocol` |
| Ingress queue | `src/ingress-queue.ts` | Ephemeral sealed in-memory ingress queue (10m TTL max) |
| Discord integration | `src/discord-gateway.ts`, `src/discord-api.ts`, `src/discord-events.ts` | Discord Gateway connection, scoped API calls, event routing |
| Telegram integration | `src/telegram.ts` | Bot API calls via `telegram-call`, single-use upload tokens |
| TURN relays | `src/turn-relays.ts`, `src/turn-probe.ts` | Coturn allocation probing and credential generation |

## CONVENTIONS
- Bun runtime only: keep Bun types inside `remote/api`; root `scripts/**` and Electron main use Node types.
- Effect 4 manages async service lifecycles, typed errors (`Schema.TaggedError`), and runtime layers.
- Contract shapes live in `@openbot/contracts/signal-protocol`; `src/protocol.ts` provides server validation and re-exports.
- Forward webhook bodies in transit (`POST /v1/slack/events`, `POST /v1/telegram/updates/:botId`, `POST /v1/github/events`); do not read or log payload text.

## ANTI-PATTERNS
- NEVER log webhook bodies, bot or session tokens, route tickets, file/upload tokens, TURN credentials, or config secrets; log error message only.
- NEVER remove signature checks on Slack/GitHub or secret header verification on Telegram.
- NEVER persist ingress queue bodies to disk or keep unsealed payloads; queue is memory-only, sealed to host queue key, max 10 minutes.
- NEVER expose Telegram bot tokens in frames, logs, or results; host calls bot API only through `telegram-call` with strict `TelegramCallParams`.
- NEVER permit Discord calls outside routed guild/channel; never log bot tokens or Discord error text.
- NEVER replace constant-time secret comparison (`timingSafeEqual`) or exceed token TTL limits.
- NEVER run `remote/scripts/update.ts` as check (recreates live coturn container). Run `bun run test:signal` for E2E security tests.

