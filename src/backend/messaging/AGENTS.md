# `src/backend/messaging`

Chat platform adapters, transport drivers, ingress dispatch, and outbound formatting.
Earned file: score 15, distinct domain (30 files, multi-platform chat integrations).

## OVERVIEW
Platform-agnostic messaging service and specialized drivers for Discord, Slack, and Telegram integrations.

## STRUCTURE
```
messaging/
├── discord/    # Discord Gateway driver, message chunking, upload handling
├── slack/      # Slack Events API, Web API transport, mrkdwn formatting
└── telegram/   # Telegram Bot API polling, HTML formatting, update parsing
```

## WHERE TO LOOK
| Component | Primary Files | Role |
|---|---|---|
| Messaging Service | `messaging-service.ts` | Platform lifecycle, connection manager, turns, approvals |
| Turn Execution | `messaging-threads.ts` | Message threading, turn execution, outbound delivery |
| Shared Contracts | `messaging-types.ts` | `MessagingDriver`, `MessagingTransport`, `InboundMessage` |
| Tools & Schema | `messaging-tools.ts`, `messaging-schema.ts` | Model tool bindings and SQLite projection schemas |
| Platform Drivers | `discord/discord-driver.ts`, etc. | Injected platform driver implementations |

## CONVENTIONS
- **Standard platform structure:** Each platform folder provides `*-driver.ts`, `*-transport.ts`, `*-connect.ts`, and `*-orchestrator.ts`.
- **Injected drivers:** `MessagingService` receives platform drivers via constructor; never imports transport modules directly.
- **Tagged errors:** Public errors extend `Schema.TaggedError` with stable tag identifiers (e.g. `MessagingOperationFailed`).
- **I18n text:** All user-visible status and error strings must use `sourceText` from `@openbot/i18n/source`.
- **Domain prefix naming:** Files repeat domain prefixes (`messaging-*`, `slack-*`, `discord-*`, `telegram-*`); no bare `types.ts`.

## ANTI-PATTERNS
- Do not add `CHECK` constraints to `projection_messaging_connections.platform` (avoids table rebuilds on new platforms).
- Do not log raw message text, user tokens, or platform secrets; redact before transmission.
- Do not re-post failed chunks of a multi-chunk message automatically.
- Do not block outbound message delivery on slow mention-name lookups.
